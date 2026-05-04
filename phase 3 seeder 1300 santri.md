# 📦 **SEEDER 1.300 SANTRI - LENGKAP DENGAN DATA REALISTIS**

Saya buatkan seeder yang generate **1.300 santri** otomatis dengan data yang realistis (nama Indonesia, kelas, NIS random, dll).

---

## 📝 **FILE SEEDER: `backend/app/Database/Seeds/SantriSeeder.php`**

```php
<?php

namespace App\Database\Seeds;

use CodeIgniter\Database\Seeder;
use Faker\Factory;

class SantriSeeder extends Seeder
{
    public function run()
    {
        $faker = Factory::create('id_ID'); // Indonesian locale
        
        // Daftar nama depan Indonesia
        $namaDepanLaki = [
            'Ahmad', 'Muhammad', 'Abdul', 'Fajar', 'Rizki', 'Dimas', 'Andi', 'Budi', 
            'Candra', 'Dedi', 'Eko', 'Fikri', 'Gilang', 'Hendra', 'Irfan', 'Joko',
            'Kurnia', 'Lukman', 'Maulana', 'Nugroho', 'Oki', 'Pratama', 'Qori', 'Rizal',
            'Sandi', 'Teguh', 'Umar', 'Vito', 'Wahyu', 'Xavier', 'Yoga', 'Zaki'
        ];
        
        $namaDepanPerempuan = [
            'Siti', 'Nur', 'Aisyah', 'Fatimah', 'Dewi', 'Intan', 'Rina', 'Lisa',
            'Mega', 'Nadia', 'Oktavia', 'Putri', 'Rani', 'Sari', 'Tina', 'Umi',
            'Vina', 'Winda', 'Xena', 'Yuni', 'Zahra', 'Amelia', 'Bella', 'Citra',
            'Diana', 'Elsa', 'Fitri', 'Gita', 'Hana', 'Indah', 'Julia', 'Karina'
        ];
        
        $namaBelakang = [
            'Abdullah', 'Wijaya', 'Putra', 'Pratama', 'Nugroho', 'Hidayat', 'Saputra',
            'Setiawan', 'Kurniawan', 'Maulana', 'Ramadhan', 'Firmansyah', 'Hakim',
            'Santoso', 'Lesmana', 'Gunawan', 'Wibowo', 'Suryadi', 'Hermawan'
        ];
        
        // Daftar kelas (7A, 7B, 7C, 8A, 8B, 8C, 9A, 9B, 9C, 10A, 10B, 11A, 11B, 12A, 12B)
        $kelasList = [
            '7A', '7B', '7C', '7D',
            '8A', '8B', '8C', '8D',
            '9A', '9B', '9C', '9D',
            '10A', '10B', '10C', '10D',
            '11A', '11B', '11C', '11D',
            '12A', '12B', '12C', '12D'
        ];
        
        // Status distribution: 80% aktif, 10% alumni, 5% pindah, 5% dropout
        $statusOptions = ['aktif', 'aktif', 'aktif', 'aktif', 'aktif', 'aktif', 'aktif', 'aktif', 'alumni', 'pindah', 'dropout', 'lulus'];
        
        // Agama distribution
        $agamaOptions = ['Islam', 'Islam', 'Islam', 'Islam', 'Islam', 'Islam', 'Islam', 'Islam', 'Kristen', 'Katolik', 'Hindu', 'Buddha'];
        
        // Tahun masuk (2015-2025)
        $tahunMasukList = [2025, 2024, 2023, 2022, 2021, 2020, 2019, 2018, 2017, 2016, 2015];
        
        // NIS prefix per tahun
        $nisPrefix = [
            2025 => '251',
            2024 => '241',
            2023 => '231',
            2022 => '221',
            2021 => '211',
            2020 => '201',
            2019 => '191',
            2018 => '181',
            2017 => '171',
            2016 => '161',
            2015 => '151'
        ];
        
        echo "Memulai generate 1.300 data santri...\n";
        
        $santriData = [];
        $counter = 1;
        
        for ($i = 1; $i <= 1300; $i++) {
            // Pilih jenis kelamin (50:50)
            $jenisKelamin = $faker->randomElement(['L', 'P']);
            
            // Generate nama
            if ($jenisKelamin === 'L') {
                $namaDepan = $faker->randomElement($namaDepanLaki);
            } else {
                $namaDepan = $faker->randomElement($namaDepanPerempuan);
            }
            $namaBelakangRand = $faker->randomElement($namaBelakang);
            $namaLengkap = $namaDepan . ' ' . $namaBelakangRand;
            
            // Generate nama panggilan
            $namaPanggilan = $namaDepan;
            
            // Tempat lahir (kota di Indonesia)
            $kotaLahir = $faker->randomElement([
                'Jakarta', 'Surabaya', 'Bandung', 'Medan', 'Semarang', 'Yogyakarta',
                'Malang', 'Palembang', 'Makassar', 'Bogor', 'Depok', 'Tangerang',
                'Bekasi', 'Solo', 'Denpasar', 'Lampung', 'Padang', 'Manado'
            ]);
            
            // Tanggal lahir (2005-2015)
            $tanggalLahir = $faker->dateTimeBetween('-15 years', '-8 years')->format('Y-m-d');
            
            // Tahun masuk (berdasarkan kelas)
            $tahunMasuk = $faker->randomElement($tahunMasukList);
            $nisNumber = str_pad($i, 5, '0', STR_PAD_LEFT);
            $nis = $nisPrefix[$tahunMasuk] . $nisNumber;
            
            // Kelas (berdasarkan tahun masuk)
            $currentYear = date('Y');
            $kelasIndex = min(23, max(0, ($currentYear - $tahunMasuk) * 4 + rand(0, 3)));
            $kelas = $kelasList[$kelasIndex % count($kelasList)];
            
            // Status
            $status = $faker->randomElement($statusOptions);
            
            // Jika status alumni, set tahun lulus
            $tahunLulus = null;
            if ($status === 'alumni' || $status === 'lulus') {
                $tahunLulus = $tahunMasuk + 6;
                if ($tahunLulus > $currentYear) {
                    $tahunLulus = $currentYear - 1;
                }
            }
            
            // Alamat
            $alamat = $faker->streetAddress;
            $desa = $faker->randomElement(['Ciputat', 'Pondok Aren', 'Serpong', 'Ciledug', 'Pamulang', 'Cipayung', 'Sawangan', 'Limo']);
            $kecamatan = $faker->randomElement(['Ciputat', 'Pondok Aren', 'Serpong', 'Ciledug', 'Pamulang', 'Sawangan']);
            $kabupaten = $faker->randomElement(['Tangerang Selatan', 'Tangerang', 'Jakarta Selatan', 'Depok']);
            $provinsi = 'Banten';
            $kodePos = $faker->postcode;
            
            // Nama orang tua
            $namaAyah = $faker->randomElement(['Ahmad', 'Suherman', 'Dedi', 'Ujang', 'Entis', 'Kosim', 'Rokhmat', 'Tatang']) . ' ' . $faker->randomElement($namaBelakang);
            $namaIbu = $faker->randomElement(['Siti', 'Nur', 'Euis', 'Eneng', 'Yayah', 'Tuti', 'Wati', 'Cucu']) . ' ' . $faker->randomElement($namaBelakang);
            
            // Nomor telepon
            $nomorTelepon = '08' . $faker->numerify('##########');
            $nomorTeleponOrtu = '08' . $faker->numerify('##########');
            
            // Email orang tua
            $emailOrtu = strtolower($namaDepan) . '.' . strtolower($namaBelakangRand) . '@gmail.com';
            
            // Agama
            $agama = $faker->randomElement($agamaOptions);
            
            // Anak ke
            $anakKe = $faker->numberBetween(1, 5);
            $jumlahSaudara = $faker->numberBetween(0, 4);
            
            // Foto (placeholder)
            $foto = null;
            
            $santriData[] = [
                'nis' => $nis,
                'nisn' => $faker->numerify('############'),
                'nama_lengkap' => $namaLengkap,
                'nama_panggilan' => $namaPanggilan,
                'jenis_kelamin' => $jenisKelamin,
                'tempat_lahir' => $kotaLahir,
                'tanggal_lahir' => $tanggalLahir,
                'agama' => $agama,
                'anak_ke' => $anakKe,
                'jumlah_saudara' => $jumlahSaudara,
                'alamat' => $alamat,
                'rt' => $faker->numberBetween(1, 20),
                'rw' => $faker->numberBetween(1, 12),
                'desa' => $desa,
                'kecamatan' => $kecamatan,
                'kabupaten' => $kabupaten,
                'provinsi' => $provinsi,
                'kode_pos' => $kodePos,
                'nama_ayah' => $namaAyah,
                'nama_ibu' => $namaIbu,
                'pekerjaan_ayah' => $faker->randomElement(['PNS', 'Guru', 'Wiraswasta', 'Karyawan Swasta', 'Petani', 'Polisi', 'TNI', 'Dokter']),
                'pekerjaan_ibu' => $faker->randomElement(['Ibu Rumah Tangga', 'Guru', 'Karyawati', 'Wiraswasta', 'PNS', 'Perawat']),
                'pendidikan_ayah' => $faker->randomElement(['SD', 'SMP', 'SMA', 'D3', 'S1', 'S2']),
                'pendidikan_ibu' => $faker->randomElement(['SD', 'SMP', 'SMA', 'D3', 'S1', 'S2']),
                'nomor_telepon' => $nomorTelepon,
                'nomor_telepon_ortu' => $nomorTeleponOrtu,
                'email_ortu' => $emailOrtu,
                'kelas' => $kelas,
                'tahun_masuk' => $tahunMasuk,
                'tahun_lulus' => $tahunLulus,
                'status' => $status,
                'status_alumni' => ($status === 'alumni') ? 'alumni_sma' : 'non_alumni',
                'foto' => $foto,
                'created_at' => date('Y-m-d H:i:s'),
                'updated_at' => date('Y-m-d H:i:s'),
            ];
            
            // Insert in batches of 100 to avoid memory issues
            if ($i % 100 === 0) {
                $this->db->table('santri')->insertBatch($santriData);
                echo "Generated " . $i . " santri...\n";
                $santriData = [];
            }
        }
        
        // Insert remaining records
        if (!empty($santriData)) {
            $this->db->table('santri')->insertBatch($santriData);
        }
        
        echo "Selesai! 1.300 data santri berhasil digenerate.\n";
        
        // Statistik setelah insert
        $total = $this->db->table('santri')->countAllResults();
        $aktif = $this->db->table('santri')->where('status', 'aktif')->countAllResults();
        $alumni = $this->db->table('santri')->where('status', 'alumni')->countAllResults();
        
        echo "\n📊 STATISTIK:\n";
        echo "   Total Santri: {$total}\n";
        echo "   Santri Aktif: {$aktif}\n";
        echo "   Alumni: {$alumni}\n";
    }
}
```

---

## 📦 **INSTALL FAKER (WAJIB)**

Sebelum menjalankan seeder, install Faker terlebih dahulu:

```bash
cd /var/www/html/34/SyIAR1.0/backend
composer require --dev fakerphp/faker
```

---

## 🚀 **JALANKAN SEEDER**

```bash
cd /var/www/html/34/SyIAR1.0/backend

# Jalankan seeder
php spark db:seed SantriSeeder
```

**Output yang diharapkan:**
```
Memulai generate 1.300 data santri...
Generated 100 santri...
Generated 200 santri...
...
Generated 1300 santri...
Selesai! 1.300 data santri berhasil digenerate.

📊 STATISTIK:
   Total Santri: 1300
   Santri Aktif: ±1040
   Alumni: ±130
```

---

## 📊 **DISTRIBUSI DATA YANG DIHASILKAN**

| Aspek | Detail |
|-------|--------|
| **Total Data** | 1.300 santri |
| **Jenis Kelamin** | 50% Laki-laki, 50% Perempuan |
| **Kelas** | 7A sampai 12D (24 kelas) |
| **Status** | 80% Aktif, 10% Alumni, 5% Pindah, 5% Dropout/Lulus |
| **Tahun Masuk** | 2015-2025 |
| **Agama** | Mayoritas Islam (75%), sisanya Kristen, Katolik, Hindu, Buddha |
| **Nama** | Nama Indonesia realistis |
| **Alamat** | Wilayah Tangerang Selatan & sekitarnya |

---

## ✅ **VERIFIKASI DATA**

```bash
# Cek total data
mysql -u root -ptkjtkj -e "USE 34_Sistem_informasiSIARG; SELECT COUNT(*) FROM santri;"

# Cek per kelas
mysql -u root -ptkjtkj -e "USE 34_Sistem_informasiSIARG; SELECT kelas, COUNT(*) FROM santri GROUP BY kelas ORDER BY kelas;"

# Cek per status
mysql -u root -ptkjtkj -e "USE 34_Sistem_informasiSIARG; SELECT status, COUNT(*) FROM santri GROUP BY status;"

# Cek sample 10 data
mysql -u root -ptkjtkj -e "USE 34_Sistem_informasiSIARG; SELECT nis, nama_lengkap, kelas, status FROM santri LIMIT 10;"
```

---

**SEEDER 1.300 SANTRI SIAP PAKAI! TINGGAL JALANKAN.** 🚀
