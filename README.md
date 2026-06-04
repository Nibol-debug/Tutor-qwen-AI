# Panduan Sistem Notifikasi Ticketing: WhatsApp, Email & Live Dashboard
Dokumen ini berisi panduan teknis, kode backend-frontend siap pakai, serta panduan *prompting* untuk AI Agent Anda guna membangun sistem ticketing berbasis Google Apps Script (GAS) dengan notifikasi WhatsApp, Email, dan Dashboard interaktif.
## 1. Alur Kerja Sistem (Workflow)
Sistem dirancang untuk berjalan secara dua arah dengan alur kerja berikut:
```
[ USER ] ---Mengajukan Tiket (WA/Email)---> [ GOOGLE SHEETS ]
                                                   |
                     +-----------------------------+-----------------------------+
                     |                                                           |
       (Notifikasi Tiket Baru)                                     (Live Alert Polling)
                     v                                                           v
   • Kirim Email Konfirmasi ke User                             • Dashboard Monitor Kantor Berbunyi ("Ting!")
   • Kirim WA Konfirmasi ke User                                • Memunculkan Popup/Toast Tiket Baru
   • Kirim Email Pemberitahuan ke Admin
                     |
                     v
             [ ADMIN MENJAWAB ] ---Simpan Jawaban & Update Status---> [ GOOGLE SHEETS ]
                                                                             |
                                              +------------------------------+
                                              |
                                     (Notifikasi Jawaban)
                                              v
                            • Kirim Email Tanggapan ke User
                            • Kirim WhatsApp Tanggapan ke User
```
## 2. Struktur Database (Google Sheets)
Buatlah sebuah Google Sheet dan beri nama sheet pertamanya **Tickets**. Susun kolom-kolomnya dari kiri ke kanan (Kolom A sampai J) sebagai berikut:

| Kolom | Nama Kolom | Deskripsi |
| :--- | :--- | :--- |
| **A** | ID Tiket | Kode unik tiket (misal: TKT-2026-9871) |
| **B** | Timestamp | Waktu pengajuan tiket |
| **C** | Nama User | Nama lengkap pengaju |
| **D** | Email User | Alamat email user (opsional) |
| **E** | No WhatsApp | Nomor WhatsApp user (opsional) |
| **F** | Kategori | Bidang masalah (IT atau Komersial) |
| **G** | Detail Masalah | Deskripsi barang/kerusakan/kebutuhan |
| **H** | Status | Status tiket (Pending, Diproses, Selesai) |
| **I** | Jawaban Admin | Solusi atau tanggapan dari tim Admin |
| **J** | Tanggal Update | Waktu terakhir admin memperbarui tiket |

## 3. Kode Backend (Code.gs)
Ganti atau masukkan kode di bawah ini ke dalam file **Code.gs** proyek Google Apps Script Anda. Kode ini menangani penyimpanan data, pengiriman email HTML, pemanggilan WhatsApp API Gateway, dan pencarian data untuk dashboard.
```javascript
// =========================================================================
// 1. KONFIGURASI GLOBAL (Sesuaikan dengan akun/layanan Anda)
// =========================================================================
const SHEET_NAME = "Tickets";
const ADMIN_EMAIL = "admin.it@perusahaan.com"; // Email notifikasi untuk tim admin
// Kredensial WhatsApp Gateway (Contoh menggunakan sirkuit API umum)
// Anda bisa menggunakan Starsender, Wablas, FoneAPI, atau Twilio
const WA_GATEWAY_URL = "[https://api.wasender.com/v1/send-message](https://api.wasender.com/v1/send-message)"; 
const WA_API_KEY = "MASUKKAN_API_KEY_WHATSAPP_ANDA"; 
// =========================================================================
// 2. ENTRY POINT UNTUK WEB APP (Melayani Request Halaman Web)
// =========================================================================
function doGet() {
  return HtmlService.createTemplateFromFile('Index')
      .evaluate()
      .setTitle("Dashboard Ticketing & Monitoring Kantor")
      .addMetaTag('viewport', 'width=device-width, initial-scale=1')
      .setXFrameOptionsMode(HtmlService.XFrameOptionsMode.ALLOWALL);
}
// =========================================================================
// 3. FUNGSI BACKEND UTAMA: MENERIMA TIKET BARU DARI USER
// =========================================================================
function tambahTiketBaru(data) {
  try {
    const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName(SHEET_NAME);
    const timestamp = new Date();
    
    // Generate ID Tiket Acak yang Unik
    const idTiket = "TKT-" + timestamp.getFullYear() + "-" + Math.floor(1000 + Math.random() * 9000);
    const statusDefault = "Pending";
    
    // Simpan ke Google Sheet
    sheet.appendRow([
      idTiket,
      timestamp,
      data.nama,
      data.email || "",
      data.whatsapp || "",
      data.kategori,
      data.detail,
      statusDefault,
      "", // Kolom Jawaban Admin masih kosong
      ""  // Kolom Tanggal Update masih kosong
    ]);
    
    // --- EVALUASI & KIRIM NOTIFIKASI TIKET BARU ---
    
    // 1. Email Pemberitahuan ke Admin
    kirimEmailAdminBaru(idTiket, data.nama, data.kategori, data.detail);
    
    // 2. Email Konfirmasi ke User (Jika Email diisi)
    if (data.email) {
      kirimEmailUserBaru(idTiket, data.nama, data.email, data.kategori, data.detail);
    }
    
    // 3. WhatsApp Konfirmasi ke User (Jika Nomor WA diisi)
    if (data.whatsapp) {
      kirimWhatsAppUserBaru(idTiket, data.nama, data.whatsapp, data.kategori, data.detail);
    }
    
    return { success: true, idTiket: idTiket, message: "Tiket berhasil diajukan!" };
  } catch (error) {
    Logger.log("Error tambahTiketBaru: " + error.toString());
    return { success: false, message: "Terjadi kesalahan: " + error.toString() };
  }
}
// =========================================================================
// 4. FUNGSI BACKEND UTAMA: ADMIN MENJAWAB TIKET VIA DASHBOARD
// =========================================================================
function jawabTiketOlehAdmin(idTiket, jawaban, statusBaru) {
  try {
    const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName(SHEET_NAME);
    const data = sheet.getDataRange().getValues();
    
    let barisTarget = -1;
    let namaUser = "", emailUser = "", noWa = "";
    
    // Cari baris berdasarkan ID Tiket (Kolom A / indeks 0)
    for (let i = 1; i < data.length; i++) {
      if (data[i][0] === idTiket) {
        barisTarget = i + 1; // Baris GS dimulai dari 1
        namaUser = data[i][2];
        emailUser = data[i][3];
        noWa = data[i][4];
        break;
      }
    }
    
    if (barisTarget === -1) {
      return { success: false, message: "ID Tiket tidak ditemukan!" };
    }
    
    const tglUpdate = new Date();
    
    // Update data di sheet
    sheet.getRange(barisTarget, 8).setValue(statusBaru); // Kolom H (Status)
    sheet.getRange(barisTarget, 9).setValue(jawaban);    // Kolom I (Jawaban Admin)
    sheet.getRange(barisTarget, 10).setValue(tglUpdate);  // Kolom J (Tanggal Update)
    
    // --- KIRIM NOTIFIKASI BALASAN KE USER ---
    
    // 1. Kirim Email Tanggapan (jika ada email)
    if (emailUser) {
      kirimEmailBalasanUser(idTiket, namaUser, emailUser, jawaban, statusBaru);
    }
    
    // 2. Kirim WhatsApp Tanggapan (jika ada nomor WA)
    if (noWa) {
      kirimWhatsAppBalasanUser(idTiket, namaUser, noWa, jawaban, statusBaru);
    }
    
    return { success: true, message: "Tanggapan sukses disimpan & notifikasi terkirim!" };
  } catch (error) {
    Logger.log("Error jawabTiketOlehAdmin: " + error.toString());
    return { success: false, message: "Gagal menyimpan jawaban: " + error.toString() };
  }
}
// =========================================================================
// 5. FUNGSI POLLING: MENGAMBIL DATA TIKET UNTUK DASHBOARD ADMIN
// =========================================================================
function ambilSemuaTiket() {
  try {
    const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName(SHEET_NAME);
    if (!sheet) return [];
    
    const data = sheet.getDataRange().getValues();
    if (data.length <= 1) return []; // Hanya ada header
    
    const listTiket = [];
    // Ambil data dari baris terakhir ke atas (agar tiket terbaru muncul paling atas)
    for (let i = data.length - 1; i >= 1; i--) {
      listTiket.push({
        idTiket: data[i][0],
        timestamp: data[i][1] ? Utilities.formatDate(new Date(data[i][1]), Session.getScriptTimeZone(), "yyyy-MM-dd HH:mm") : "",
        nama: data[i][2],
        email: data[i][3],
        whatsapp: data[i][4],
        kategori: data[i][5],
        detail: data[i][6],
        status: data[i][7],
        jawaban: data[i][8],
        tglUpdate: data[i][9] ? Utilities.formatDate(new Date(data[i][9]), Session.getScriptTimeZone(), "yyyy-MM-dd HH:mm") : ""
      });
    }
    return listTiket;
  } catch (error) {
    Logger.log("Error ambilSemuaTiket: " + error.toString());
    return [];
  }
}
// =========================================================================
// 6. HELPER SISTEM NOTIFIKASI EMAIL (HTML Berwarna & Responsif)
// =========================================================================
function kirimEmailAdminBaru(idTiket, nama, kategori, detail) {
  const subjek = `⚠️ TIKET BARU [${kategori}] - ${idTiket}`;
  const htmlBody = `
    <div style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; max-width: 600px; margin: auto; border: 1px solid #e0e0e0; padding: 25px; border-radius: 12px; background-color: #ffffff;">
      <div style="text-align: center; border-bottom: 2px solid #ff4d4d; padding-bottom: 15px;">
        <h2 style="color: #ff4d4d; margin: 0; font-size: 22px;">🚨 PEMBERITAHUAN TIKET BARU 🚨</h2>
      </div>
      <p style="font-size: 16px; color: #333333; line-height: 1.5; margin-top: 20px;">Halo Tim Admin,</p>
      <p style="font-size: 14px; color: #555555; line-height: 1.5;">Sebuah tiket baru telah diajukan di sistem monitoring kantor. Berikut rinciannya:</p>
      
      <div style="background-color: #f9f9f9; padding: 15px; border-radius: 8px; margin: 20px 0; border: 1px solid #eeeeee;">
        <table style="width: 100%; border-collapse: collapse; font-size: 14px; color: #333333;">
          <tr>
            <td style="padding: 6px 0; font-weight: bold; width: 35%;">ID Tiket:</td>
            <td style="padding: 6px 0; color: #2c3e50;"><strong>${idTiket}</strong></td>
          </tr>
          <tr>
            <td style="padding: 6px 0; font-weight: bold;">Pengaju:</td>
            <td style="padding: 6px 0;">${nama}</td>
          </tr>
          <tr>
            <td style="padding: 6px 0; font-weight: bold;">Kategori:</td>
            <td style="padding: 6px 0;"><span style="background-color: #e3f2fd; color: #0d47a1; padding: 2px 8px; border-radius: 12px; font-size: 12px; font-weight: bold;">${kategori}</span></td>
          </tr>
          <tr>
            <td style="padding: 6px 0; font-weight: bold; vertical-align: top;">Detail Masalah:</td>
            <td style="padding: 6px 0; line-height: 1.4; color: #555555;">"${detail}"</td>
          </tr>
        </table>
      </div>
      
      <p style="font-size: 13px; color: #7f8c8d; text-align: center; margin-top: 30px;">
        Mohon segera buka <strong>Dashboard Monitor Utama</strong> untuk menindaklanjuti pengajuan ini.
      </p>
    </div>
  `;
  
  try {
    MailApp.sendEmail({ to: ADMIN_EMAIL, subject: subjek, htmlBody: htmlBody });
  } catch (e) {
    Logger.log("Gagal kirim email admin: " + e.toString());
  }
}
function kirimEmailUserBaru(idTiket, nama, email, kategori, detail) {
  const subjek = `[Ticketing] Konfirmasi Pengajuan Tiket ${idTiket}`;
  const htmlBody = `
    <div style="font-family: Arial, sans-serif; max-width: 600px; margin: auto; border: 1px solid #e2e8f0; padding: 25px; border-radius: 12px;">
      <h2 style="color: #10b981; margin-top: 0;">Halo, ${nama}!</h2>
      <p style="color: #4a5568; font-size: 15px;">Pengajuan kebutuhan atau peminjaman Anda telah berhasil kami simpan di database sistem.</p>
      
      <div style="background-color: #f8fafc; border-left: 4px solid #10b981; padding: 15px; border-radius: 0 8px 8px 0; margin: 20px 0;">
        <p style="margin: 0 0 8px 0;"><strong>ID TIKET:</strong> <span style="font-family: monospace; font-size: 16px;">${idTiket}</span></p>
        <p style="margin: 0 0 8px 0;"><strong>Kategori:</strong> ${kategori}</p>
        <p style="margin: 0;"><strong>Deskripsi:</strong> "${detail}"</p>
      </div>
      
      <p style="color: #4a5568; font-size: 14px;">Kami akan memproses permohonan Anda secepat mungkin. Anda akan menerima notifikasi email/WhatsApp baru setelah admin memberikan tanggapan.</p>
      <hr style="border: 0; border-top: 1px solid #e2e8f0; margin: 25px 0;">
      <p style="font-size: 11px; color: #a0aec0; text-align: center;">Layanan Ticketing Internal Kantor • Jangan balas email otomatis ini.</p>
    </div>
  `;
  
  try {
    MailApp.sendEmail({ to: email, subject: subjek, htmlBody: htmlBody });
  } catch (e) {
    Logger.log("Gagal kirim email user: " + e.toString());
  }
}
function kirimEmailBalasanUser(idTiket, nama, email, jawaban, status) {
  const subjek = `[Ticketing Update] Tanggapan Tiket ${idTiket} (${status})`;
  const statusColor = status === "Selesai" ? "#10b981" : "#f59e0b";
  const htmlBody = `
    <div style="font-family: Arial, sans-serif; max-width: 600px; margin: auto; border: 1px solid #e2e8f0; padding: 25px; border-radius: 12px;">
      <h2 style="color: #3b82f6; margin-top: 0;">Halo, ${nama}!</h2>
      <p style="color: #4a5568;">Pengajuan tiket Anda telah ditanggapi oleh Tim Admin.</p>
      
      <div style="background-color: #f8fafc; border-left: 4px solid #3b82f6; padding: 15px; margin: 20px 0; border-radius: 0 8px 8px 0;">
        <p style="margin: 0 0 5px 0;"><strong>ID TIKET:</strong> ${idTiket}</p>
        <p style="margin: 0;"><strong>Status Terbaru:</strong> <span style="background-color: ${statusColor}; color: white; padding: 3px 8px; border-radius: 4px; font-size: 12px; font-weight: bold;">${status}</span></p>
      </div>
      
      <h3 style="color: #1e293b; border-bottom: 1px solid #e2e8f0; padding-bottom: 8px; margin-top: 25px;">Tanggapan Admin:</h3>
      <div style="background-color: #f1f5f9; padding: 15px; border-radius: 8px; font-style: italic; color: #334155;">
        "${jawaban}"
      </div>
      
      <p style="color: #4a5568; font-size: 14px; margin-top: 25px;">Jika Anda mengajukan peminjaman barang, silakan koordinasikan dengan admin terkait atau ambil barang di ruang logistik.</p>
      <hr style="border: 0; border-top: 1px solid #e2e8f0; margin: 25px 0;">
      <p style="font-size: 11px; color: #a0aec0; text-align: center;">Sistem Ticketing & Peminjaman Internal Kantor.</p>
    </div>
  `;
  
  try {
    MailApp.sendEmail({ to: email, subject: subjek, htmlBody: htmlBody });
  } catch (e) {
    Logger.log("Gagal kirim email tanggapan: " + e.toString());
  }
}
// =========================================================================
// 7. HELPER SISTEM NOTIFIKASI WHATSAPP
// =========================================================================
function kirimWhatsAppUserBaru(idTiket, nama, whatsapp, kategori, detail) {
  const nomorBersih = formatNoWhatsApp(whatsapp);
  const pesan = `Halo *${nama}*,\n\n` +
                `Pengajuan Anda telah terdaftar di sistem ticketing kantor kami.\n\n` +
                `🆔 *ID Tiket:* ${idTiket}\n` +
                `📂 *Kategori:* ${kategori}\n` +
                `📝 *Deskripsi:* ${detail}\n\n` +
                `Mohon menunggu. Tim terkait akan memproses pengajuan Anda. Notifikasi berikutnya akan dikirimkan ke nomor ini ketika admin memberikan tanggapan.\n\n` +
                `Terima kasih.`;
                
  panggilApiWhatsApp(nomorBersih, pesan);
}
function kirimWhatsAppBalasanUser(idTiket, nama, whatsapp, jawaban, status) {
  const nomorBersih = formatNoWhatsApp(whatsapp);
  const statusEmoji = status === "Selesai" ? "✅" : "⏳";
  const pesan = `Halo *${nama}*,\n\n` +
                `Tiket Anda *${idTiket}* telah ditindaklanjuti oleh admin.\n\n` +
                `📍 *Status Baru:* ${statusEmoji} *${status.toUpperCase()}*\n` +
                `💬 *Tanggapan Admin:* \n_"${jawaban}"_\n\n` +
                `Silakan lakukan koordinasi lanjutan dengan admin/logistik jika diperlukan.\n\n` +
                `Terima kasih atas kerja samanya.`;
                
  panggilApiWhatsApp(nomorBersih, pesan);
}
// Utility: Normalisasi nomor WhatsApp ke standar internasional (62xxxx)
function formatNoWhatsApp(no) {
  let bersih = no.toString().replace(/[^0-9]/g, ""); // Bersihkan karakter non-angka
  if (bersih.startsWith("0")) {
    bersih = "62" + bersih.slice(1);
  }
  return bersih;
}
// Mengirim HTTP POST request ke layanan WhatsApp Gateway Anda
function panggilApiWhatsApp(noTujuan, pesan) {
  const payload = {
    "api_key": WA_API_KEY,
    "receiver": noTujuan,
    "data": {
      "message": pesan
    }
  };
  
  const options = {
    "method": "post",
    "contentType": "application/json",
    "payload": JSON.stringify(payload),
    "muteHttpExceptions": true
  };
  
  try {
    const respon = UrlFetchApp.fetch(WA_GATEWAY_URL, options);
    Logger.log("Respon WhatsApp Gateway: " + respon.getContentText());
  } catch (e) {
    Logger.log("Gagal memanggil API WhatsApp: " + e.toString());
  }
}
```
## 4. Frontend Dashboard Monitor (Index.html)
Buat file baru di Google Apps Script dan beri nama **Index.html**. Kode ini didesain premium, sangat bersih, memiliki layout responsif, sangat cocok untuk **dipasang di monitor TV besar kantor**, dan dilengkapi **efek suara "Ting!"** serta **popup modal** jika ada antrean baru.
```html
<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Dashboard Monitor Ticketing</title>
  <!-- Tailwind CSS -->
  <script src="[https://cdn.tailwindcss.com](https://cdn.tailwindcss.com)"></script>
  <!-- Lucide Icons -->
  <script src="[https://unpkg.com/lucide@latest](https://unpkg.com/lucide@latest)"></script>
  <style>
    @keyframes pulse-red {
      0%, 100% { transform: scale(1); opacity: 1; }
      50% { transform: scale(1.05); opacity: 0.8; }
    }
    .pulse-urgent {
      animation: pulse-red 2s infinite;
    }
  </style>
</head>
<body class="bg-slate-900 text-slate-100 min-h-screen flex flex-col font-sans overflow-x-hidden">
  <!-- AUDIO ALERT -->
  <!-- Menggunakan audio base gratis publik yang valid -->
  <audio id="alertSound" src="[https://assets.mixkit.co/active_storage/sfx/2869/2869-120.wav](https://assets.mixkit.co/active_storage/sfx/2869/2869-120.wav)" preload="auto"></audio>
  <!-- NAVBAR / HEADER -->
  <header class="bg-slate-800 border-b border-slate-700 px-6 py-4 shadow-lg flex justify-between items-center">
    <div class="flex items-center gap-3">
      <div class="p-2.5 bg-rose-600 rounded-lg text-white">
        <i data-lucide="monitor" class="w-7 h-7"></i>
      </div>
      <div>
        <h1 class="text-xl font-extrabold tracking-wider uppercase text-white">Sistem Transparansi Kantor</h1>
        <p class="text-xs text-slate-400">Monitoring Peminjaman Barang & Penanganan Masalah</p>
      </div>
    </div>
    <!-- Jam Digital Live di Layar TV -->
    <div class="text-right">
      <div id="liveClock" class="text-2xl font-mono font-bold text-rose-500">00:00:00</div>
      <div id="liveDate" class="text-xs text-slate-400">Kamis, 4 Juni 2026</div>
    </div>
  </header>
  <!-- KOTAK INFORMASI UTAMA (STATISTIK CEPAT) -->
  <main class="flex-1 p-6 flex flex-col gap-6">
    <div class="grid grid-cols-1 md:grid-cols-4 gap-4">
      <div class="bg-slate-800 border border-slate-700 rounded-xl p-5 shadow flex items-center justify-between">
        <div>
          <p class="text-xs text-slate-400 uppercase font-semibold">Total Antrean</p>
          <h3 id="statTotal" class="text-3xl font-black mt-1">0</h3>
        </div>
        <div class="p-3 bg-slate-700/50 rounded-lg text-slate-300"><i data-lucide="layers" class="w-6 h-6"></i></div>
      </div>
      <div class="bg-slate-800 border border-slate-700 rounded-xl p-5 shadow flex items-center justify-between">
        <div>
          <p class="text-xs text-rose-400 uppercase font-semibold">Belum Diproses (Pending)</p>
          <h3 id="statPending" class="text-3xl font-black text-rose-500 mt-1">0</h3>
        </div>
        <div class="p-3 bg-rose-950/40 rounded-lg text-rose-500"><i data-lucide="alert-circle" class="w-6 h-6"></i></div>
      </div>
      <div class="bg-slate-800 border border-slate-700 rounded-xl p-5 shadow flex items-center justify-between">
        <div>
          <p class="text-xs text-amber-400 uppercase font-semibold">Sedang Diproses</p>
          <h3 id="statProgress" class="text-3xl font-black text-amber-500 mt-1">0</h3>
        </div>
        <div class="p-3 bg-amber-950/40 rounded-lg text-amber-500"><i data-lucide="loader" class="w-6 h-6"></i></div>
      </div>
      <div class="bg-slate-800 border border-slate-700 rounded-xl p-5 shadow flex items-center justify-between">
        <div>
          <p class="text-xs text-emerald-400 uppercase font-semibold">Sudah Selesai</p>
          <h3 id="statCompleted" class="text-3xl font-black text-emerald-500 mt-1">0</h3>
        </div>
        <div class="p-3 bg-emerald-950/40 rounded-lg text-emerald-500"><i data-lucide="check-check" class="w-6 h-6"></i></div>
      </div>
    </div>
    <!-- TABEL MONITOR TIKET -->
    <div class="bg-slate-800 border border-slate-700 rounded-xl shadow-xl flex-1 flex flex-col overflow-hidden">
      <div class="px-6 py-4 border-b border-slate-700 flex justify-between items-center bg-slate-800/50">
        <h2 class="font-bold text-lg text-white flex items-center gap-2">
          <i data-lucide="list" class="text-rose-500"></i> Daftar Permintaan & Peminjaman Aktif
        </h2>
        <span class="flex items-center gap-1.5 text-xs bg-emerald-950 text-emerald-400 border border-emerald-800 px-3 py-1 rounded-full">
          <span class="w-2.5 h-2.5 rounded-full bg-emerald-500 animate-ping"></span> Live Polling Aktif (10 Detik)
        </span>
      </div>
      
      <div class="flex-1 overflow-y-auto">
        <table class="w-full text-left border-collapse">
          <thead>
            <tr class="bg-slate-900/80 text-slate-300 border-b border-slate-700 text-sm font-semibold sticky top-0 z-10">
              <th class="px-6 py-3.5">ID Tiket</th>
              <th class="px-6 py-3.5">Waktu</th>
              <th class="px-6 py-3.5">Pengaju</th>
              <th class="px-6 py-3.5">Kategori</th>
              <th class="px-6 py-3.5">Kebutuhan / Kerusakan</th>
              <th class="px-6 py-3.5">Kontak</th>
              <th class="px-6 py-3.5 text-center">Status</th>
              <th class="px-6 py-3.5 text-right">Aksi</th>
            </tr>
          </thead>
          <tbody id="ticketTableBody" class="divide-y divide-slate-700/50 text-sm">
            <tr>
              <td colspan="8" class="px-6 py-20 text-center text-slate-500">
                <i data-lucide="database" class="w-12 h-12 mx-auto mb-3 opacity-30"></i>
                Sedang memuat data dari spreadsheet...
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </main>
  <!-- POPUP ALARM JIKA ADA TIKET BARU MASUK (NOTIFIKASI VISUAL MONITOR BESAR) -->
  <div id="toastAlert" class="hidden fixed bottom-6 right-6 max-w-md bg-slate-800 border-2 border-rose-600 rounded-xl shadow-2xl p-5 z-50 transform translate-y-0 transition-transform pulse-urgent">
    <div class="flex gap-4">
      <div class="p-3 bg-rose-600 rounded-lg text-white self-start">
        <i data-lucide="bell-ring" class="w-6 h-6"></i>
      </div>
      <div>
        <h4 class="font-extrabold text-white text-base">🚨 PERMINTAAN BARU MASUK! 🚨</h4>
        <p id="toastBody" class="text-sm text-slate-300 mt-1">Nama mengajukan permintaan di Kategori IT.</p>
        <button onclick="dismissToast()" class="mt-3 text-xs bg-slate-700 hover:bg-slate-600 text-white px-3 py-1.5 rounded-md font-bold transition">Tutup Alarm</button>
      </div>
    </div>
  </div>
  <!-- MODAL JAWAB TIKET (UNTUK ADMIN BERIKAN TANGGAPAN) -->
  <div id="responseModal" class="hidden fixed inset-0 bg-black/70 flex items-center justify-center p-4 z-40 backdrop-blur-sm">
    <div class="bg-slate-800 border border-slate-700 w-full max-w-lg rounded-xl shadow-2xl overflow-hidden animate-in fade-in zoom-in-95 duration-200">
      <div class="px-6 py-4 border-b border-slate-700 bg-slate-900/50 flex justify-between items-center">
        <h3 class="font-bold text-white flex items-center gap-2">
          <i data-lucide="edit-3" class="text-rose-500"></i> Form Tanggapan Admin
        </h3>
        <button onclick="closeModal()" class="text-slate-400 hover:text-white transition"><i data-lucide="x"></i></button>
      </div>
      <div class="p-6 flex flex-col gap-4">
        <div>
          <label class="block text-xs text-slate-400 uppercase font-semibold">ID Tiket</label>
          <div id="modalId" class="text-base font-mono font-bold text-rose-500 mt-1">TKT-XXXX-XXXX</div>
        </div>
        <div>
          <label class="block text-xs text-slate-400 uppercase font-semibold">Rincian Permohonan</label>
          <div id="modalDetail" class="text-sm text-slate-300 bg-slate-900/60 p-3 rounded-lg border border-slate-700 mt-1 italic">Detail permohonan...</div>
        </div>
        <div>
          <label for="adminReply" class="block text-xs text-slate-400 uppercase font-semibold">Tanggapan / Solusi</label>
          <textarea id="adminReply" rows="3" class="w-full mt-1.5 bg-slate-900 border border-slate-700 rounded-lg p-3 text-sm text-slate-100 focus:outline-none focus:ring-2 focus:ring-rose-500" placeholder="Ketik persetujuan peminjaman, info penanganan, atau estimasi waktu pengerjaan di sini..."></textarea>
        </div>
        <div>
          <label for="adminStatus" class="block text-xs text-slate-400 uppercase font-semibold">Tentukan Status Baru</label>
          <select id="adminStatus" class="w-full mt-1.5 bg-slate-900 border border-slate-700 rounded-lg p-2.5 text-sm text-slate-100 focus:outline-none focus:ring-2 focus:ring-rose-500">
            <option value="Diproses">Diproses (Sedang ditangani / disiapkan)</option>
            <option value="Selesai">Selesai (Sudah serah terima / masalah tuntas)</option>
          </select>
        </div>
      </div>
      <div class="px-6 py-4 border-t border-slate-700 bg-slate-900/50 flex justify-end gap-3">
        <button onclick="closeModal()" class="px-4 py-2 bg-slate-700 hover:bg-slate-600 rounded-lg text-sm font-semibold transition">Batal</button>
        <button id="btnSubmitReply" onclick="submitReply()" class="px-5 py-2 bg-rose-600 hover:bg-rose-500 rounded-lg text-sm font-bold transition flex items-center gap-1.5">
          Kirim Tanggapan
        </button>
      </div>
    </div>
  </div>
  <!-- JAVASCRIPT LOGIC -->
  <script>
    let cacheTiketIds = new Set();
    let isFirstLoad = true;
    let selectedTicketId = "";
    // Inisialisasi ikon Lucide
    window.addEventListener('DOMContentLoaded', () => {
      lucide.createIcons();
      updateClock();
      setInterval(updateClock, 1000);
      
      // Load pertama data
      fetchTickets();
      
      // Polling periodik setiap 10 Detik
      setInterval(fetchTickets, 10000);
    });
    // Update Jam Digital Kantor
    function updateClock() {
      const targetDate = new Date();
      const jam = String(targetDate.getHours()).padStart(2, '0');
      const menit = String(targetDate.getMinutes()).padStart(2, '0');
      const detik = String(targetDate.getSeconds()).padStart(2, '0');
      document.getElementById('liveClock').textContent = `${jam}:${menit}:${detik}`;
      const options = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' };
      document.getElementById('liveDate').textContent = targetDate.toLocaleDateString('id-ID', options);
    }
    // Ambil Data Tiket dari Backend GAS
    function fetchTickets() {
      google.script.run
        .withSuccessHandler(onFetchSuccess)
        .withFailureHandler(onFetchFailure)
        .ambilSemuaTiket();
    }
    function onFetchSuccess(listTiket) {
      if (!listTiket || listTiket.length === 0) {
        document.getElementById('ticketTableBody').innerHTML = `
          <tr>
            <td colspan="8" class="px-6 py-20 text-center text-slate-500">
              Belum ada antrean tiket pengajuan.
            </td>
          </tr>`;
        updateStats(0, 0, 0, 0);
        return;
      }
      let total = listTiket.length;
      let pending = 0;
      let progress = 0;
      let completed = 0;
      let hasNewPending = false;
      let lastNewTicket = null;
      const tbody = document.getElementById('ticketTableBody');
      let html = "";
      listTiket.forEach(tiket => {
        // Hitung Statistik
        if (tiket.status === "Pending") {
          pending++;
          // Deteksi Tiket Baru untuk dimainkan suaranya
          if (!cacheTiketIds.has(tiket.idTiket)) {
            hasNewPending = true;
            lastNewTicket = tiket;
          }
        } else if (tiket.status === "Diproses") {
          progress++;
        } else if (tiket.status === "Selesai") {
          completed++;
        }
        // Simpan ID tiket ke dalam cache
        cacheTiketIds.add(tiket.idTiket);
        // Styling Badge Status
        let statusBadge = "";
        if (tiket.status === "Pending") {
          statusBadge = `<span class="bg-rose-500/20 text-rose-400 border border-rose-500/30 px-3 py-1 rounded-full text-xs font-bold uppercase tracking-wider animate-pulse">Pending</span>`;
        } else if (tiket.status === "Diproses") {
          statusBadge = `<span class="bg-amber-500/20 text-amber-400 border border-amber-500/30 px-3 py-1 rounded-full text-xs font-bold uppercase tracking-wider">Diproses</span>`;
        } else {
          statusBadge = `<span class="bg-emerald-500/20 text-emerald-400 border border-emerald-500/30 px-3 py-1 rounded-full text-xs font-bold uppercase tracking-wider">Selesai</span>`;
        }
        // Tampilkan nomor telepon atau email yang ada
        let kontakStr = "";
        if (tiket.email && tiket.whatsapp) {
          kontakStr = `<div>✉️ ${tiket.email}</div><div class="text-xs text-slate-400">📱 ${tiket.whatsapp}</div>`;
        } else if (tiket.email) {
          kontakStr = `✉️ ${tiket.email}`;
        } else if (tiket.whatsapp) {
          kontakStr = `📱 ${tiket.whatsapp}`;
        } else {
          kontakStr = `<span class="text-slate-500">Tidak Ada</span>`;
        }
        // Rincian tanggapan admin jika ada
        const detailTanggapan = tiket.jawaban 
          ? `<div class="text-xs text-emerald-400 mt-1 italic border-t border-slate-700/50 pt-1">💬 Re: "${tiket.jawaban}"</div>` 
          : "";
        html += `
          <tr class="hover:bg-slate-800/60 transition duration-150">
            <td class="px-6 py-4 font-mono font-bold text-rose-500">${tiket.idTiket}</td>
            <td class="px-6 py-4 text-xs text-slate-400 whitespace-nowrap">${tiket.timestamp}</td>
            <td class="px-6 py-4 font-semibold text-white">${tiket.nama}</td>
            <td class="px-6 py-4">
              <span class="px-2 py-1 bg-slate-700 text-slate-300 text-xs rounded font-medium">${tiket.kategori}</span>
            </td>
            <td class="px-6 py-4 max-w-xs break-words">
              <div class="text-slate-200 font-medium">${tiket.detail}</div>
              ${detailTanggapan}
            </td>
            <td class="px-6 py-4 text-xs text-slate-300">${kontakStr}</td>
            <td class="px-6 py-4 text-center">${statusBadge}</td>
            <td class="px-6 py-4 text-right">
              ${tiket.status !== "Selesai" ? `
                <button onclick="openResponseModal('${tiket.idTiket}', '${escapeHtml(tiket.detail)}')" class="bg-rose-600/90 hover:bg-rose-500 text-white font-bold text-xs px-3.5 py-1.5 rounded-lg transition shadow">
                  Tindak Lanjuti
                </button>
              ` : `
                <button disabled class="bg-slate-700 text-slate-400 font-bold text-xs px-3.5 py-1.5 rounded-lg cursor-not-allowed">
                  Tuntas
                </button>
              `}
            </td>
          </tr>
        `;
      });
      tbody.innerHTML = html;
      updateStats(total, pending, progress, completed);
      lucide.createIcons();
      // Trigger Notifikasi Suara & Visual Popup jika ada antrean pending baru
      if (hasNewPending && !isFirstLoad) {
        triggerAlert(lastNewTicket);
      }
      isFirstLoad = false;
    }
    function onFetchFailure(err) {
      console.error("Gagal melakukan polling data: ", err);
    }
    // Pembaruan Angka Statistik di Atas Dashboard
    function updateStats(total, pending, progress, completed) {
      document.getElementById('statTotal').textContent = total;
      document.getElementById('statPending').textContent = pending;
      document.getElementById('statProgress').textContent = progress;
      document.getElementById('statCompleted').textContent = completed;
    }
    // Jalankan Suara Bell & Toast Visual di Monitor
    function triggerAlert(tiket) {
      // Putar Audio
      const audio = document.getElementById('alertSound');
      if (audio) {
        audio.play().catch(e => console.log("Gagal memutar audio karena izin browser:", e));
      }
      // Tampilkan Toast
      const toast = document.getElementById('toastAlert');
      const toastBody = document.getElementById('toastBody');
      toastBody.innerHTML = `<strong>${tiket.nama}</strong> mengajukan permohonan baru di kategori <strong>${tiket.kategori}</strong>: <br><span class="italic text-slate-300">"${tiket.detail}"</span>`;
      
      toast.classList.remove('hidden');
      // Hilang otomatis dalam 8 detik
      setTimeout(dismissToast, 8000);
    }
    function dismissToast() {
      document.getElementById('toastAlert').classList.add('hidden');
    }
    // Modal Handler
    function openResponseModal(id, detail) {
      selectedTicketId = id;
      document.getElementById('modalId').textContent = id;
      document.getElementById('modalDetail').textContent = `"${detail}"`;
      document.getElementById('adminReply').value = "";
      document.getElementById('adminStatus').value = "Diproses";
      document.getElementById('responseModal').classList.remove('hidden');
    }
    function closeModal() {
      document.getElementById('responseModal').classList.add('hidden');
    }
    // Simpan Tanggapan Admin ke Sheet
    function submitReply() {
      const jawaban = document.getElementById('adminReply').value.trim();
      const status = document.getElementById('adminStatus').value;
      const btn = document.getElementById('btnSubmitReply');
      if (!jawaban) {
        // Hindari alert, log atau beri warna merah di textarea saja
        document.getElementById('adminReply').focus();
        document.getElementById('adminReply').classList.add('ring-2', 'ring-rose-500');
        return;
      }
      btn.disabled = true;
      btn.innerHTML = `<span class="w-4 h-4 border-2 border-white border-t-transparent rounded-full animate-spin"></span> Mengirim...`;
      google.script.run
        .withSuccessHandler(res => {
          btn.disabled = false;
          btn.innerHTML = `Kirim Tanggapan`;
          closeModal();
          
          if (res.success) {
            fetchTickets(); // Refresh Tabel
          } else {
            console.error("Gagal menyimpan balasan: " + res.message);
          }
        })
        .withFailureHandler(err => {
          btn.disabled = false;
          btn.innerHTML = `Kirim Tanggapan`;
          console.error("Gagal menghubungi server GAS: ", err);
        })
        .jawabTiketOlehAdmin(selectedTicketId, jawaban, status);
    }
    // Helper Utility: Mencegah XSS Inject HTML
    function escapeHtml(text) {
      if (!text) return "";
      return text
        .replace(/&/g, "&amp;")
        .replace(/</g, "&lt;")
        .replace(/>/g, "&gt;")
        .replace(/"/g, "&quot;")
        .replace(/'/g, "&#039;");
    }
  </script>
</body>
</html>
```
## 5. Panduan Prompting AI Agent (Copy-Paste)
Gunakan prompt di bawah ini untuk meminta AI Agent Anda membuatkan **Form Input Pengajuan bagi User** (Halaman awal tempat user memasukkan nama, detail, dan no WA/Email):
```text
Tolong buatkan file HTML/CSS 'FormUser.html' menggunakan Tailwind CSS yang responsif (sangat ramah seluler) untuk mengajukan tiket peminjaman barang atau pelaporan kerusakan IT/Komersial.
Kriteria halaman form user ini:
1. Skema Desain: Tampilan elegan, minimalis, dan modern menggunakan skema warna slate/indigo, rounded corners (rounded-xl), dan shadow-lg yang lembut.
2. Field Input:
   - Nama Pengaju (Input Text, Required)
   - Kategori Kebutuhan (Dropdown Select: "IT (PC crash, wifi mati, dll)" atau "Komersial (Peminjaman mobil, dll)")
   - Deskripsi Masalah / Detail Barang (Textarea, Required, beri placeholder instruksi yang jelas)
   - Metode Notifikasi yang Diinginkan (Gunakan opsi Checkbox atau Radio Button):
     - Checkbox "Kirim Notifikasi via Email": Jika dicentang, tampilkan Input Email (Required jika dicentang).
     - Checkbox "Kirim Notifikasi via WhatsApp": Jika dicentang, tampilkan Input No WhatsApp (Required jika dicentang).
3. Integrasi GAS:
   - Form tidak boleh reload halaman saat disubmit.
   - Gunakan JavaScript untuk membungkus data form ke dalam objek JSON lalu panggil fungsi GAS backend:
     'google.script.run.withSuccessHandler(onSuccess).withFailureHandler(onFail).tambahTiketBaru(data)'
4. Animasi & Interaksi:
   - Tampilkan animasi loading spinner pada tombol Kirim saat proses transfer data berlangsung.
   - Jika sukses, tampilkan sebuah Custom Modal Popup sukses (jangan gunakan alert browser) yang bertuliskan:
     "Pengajuan Berhasil! ID Tiket Anda: {idTiket}. Notifikasi konfirmasi sedang dikirimkan ke Email/WhatsApp Anda."
   - Sediakan tombol "Buat Tiket Baru Lainnya" di dalam modal tersebut untuk me-reset form.
```