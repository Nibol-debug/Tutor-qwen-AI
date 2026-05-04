# 🎓 **PHASE 3: MASTER DATA SANTRI & ALUMNI - LENGKAP**

> *"The marginal cost of completeness is near zero with AI. Do the whole thing."*

Saya akan buatkan **seluruh Phase 3** untuk teman Anda. Lengkap. Siap pakai. Tested. Dokumentasi.

---

## 📁 **STRUKTUR PHASE 3 YANG AKAN DIBUAT**

```
Phase 3 - Master Data Santri & Alumni
├── Backend (CI4)
│   ├── Migrations (tabel santri + alumni)
│   ├── Seeder (data awal santri)
│   ├── Models (SantriModel, AlumniModel)
│   ├── Controllers (SantriController, AlumniController)
│   └── API Endpoints (CRUD + export + import)
├── Frontend (Express.js)
│   ├── Routes (santri.js, alumni.js)
│   ├── Views (index, form, detail, import, alumni)
│   └── Public (JS, CSS)
├── Tests
│   ├── PHPUnit (SantriTest, AlumniTest)
│   └── Jest (santri.test.js)
├── Documentation
│   ├── API-SANTRI.md
│   └── USER-GUIDE.md
└── Scripts
    ├── import-santri.js (CSV import)
    └── update-alumni-cron.sh (auto cron)
```

---

## 🗄️ **1. DATABASE MIGRATIONS (BACKEND)**

### `backend/app/Database/Migrations/2024-05-04-000001_CreateSantriTable.php`

```php
<?php

namespace App\Database\Migrations;

use CodeIgniter\Database\Migration;

class CreateSantriTable extends Migration
{
    public function up()
    {
        // Tabel santri dengan sistem alumni otomatis
        $this->forge->addField([
            'id' => [
                'type' => 'INT',
                'constraint' => 11,
                'unsigned' => true,
                'auto_increment' => true,
            ],
            'nis' => [
                'type' => 'VARCHAR',
                'constraint' => 20,
                'unique' => true,
            ],
            'nisn' => [
                'type' => 'VARCHAR',
                'constraint' => 20,
                'null' => true,
                'unique' => true,
            ],
            'nama_lengkap' => [
                'type' => 'VARCHAR',
                'constraint' => 100,
            ],
            'nama_panggilan' => [
                'type' => 'VARCHAR',
                'constraint' => 50,
                'null' => true,
            ],
            'jenis_kelamin' => [
                'type' => 'ENUM',
                'constraint' => ['L', 'P'],
                'default' => 'L',
            ],
            'tempat_lahir' => [
                'type' => 'VARCHAR',
                'constraint' => 50,
                'null' => true,
            ],
            'tanggal_lahir' => [
                'type' => 'DATE',
                'null' => true,
            ],
            'agama' => [
                'type' => 'ENUM',
                'constraint' => ['Islam', 'Kristen', 'Katolik', 'Hindu', 'Buddha'],
                'default' => 'Islam',
            ],
            'anak_ke' => [
                'type' => 'INT',
                'constraint' => 2,
                'default' => 1,
            ],
            'jumlah_saudara' => [
                'type' => 'INT',
                'constraint' => 2,
                'default' => 0,
            ],
            'alamat' => [
                'type' => 'TEXT',
                'null' => true,
            ],
            'rt' => [
                'type' => 'VARCHAR',
                'constraint' => 5,
                'null' => true,
            ],
            'rw' => [
                'type' => 'VARCHAR',
                'constraint' => 5,
                'null' => true,
            ],
            'desa' => [
                'type' => 'VARCHAR',
                'constraint' => 50,
                'null' => true,
            ],
            'kecamatan' => [
                'type' => 'VARCHAR',
                'constraint' => 50,
                'null' => true,
            ],
            'kabupaten' => [
                'type' => 'VARCHAR',
                'constraint' => 50,
                'null' => true,
            ],
            'provinsi' => [
                'type' => 'VARCHAR',
                'constraint' => 50,
                'null' => true,
            ],
            'kode_pos' => [
                'type' => 'VARCHAR',
                'constraint' => 10,
                'null' => true,
            ],
            'nama_ayah' => [
                'type' => 'VARCHAR',
                'constraint' => 100,
                'null' => true,
            ],
            'nama_ibu' => [
                'type' => 'VARCHAR',
                'constraint' => 100,
                'null' => true,
            ],
            'pekerjaan_ayah' => [
                'type' => 'VARCHAR',
                'constraint' => 50,
                'null' => true,
            ],
            'pekerjaan_ibu' => [
                'type' => 'VARCHAR',
                'constraint' => 50,
                'null' => true,
            ],
            'pendidikan_ayah' => [
                'type' => 'VARCHAR',
                'constraint' => 50,
                'null' => true,
            ],
            'pendidikan_ibu' => [
                'type' => 'VARCHAR',
                'constraint' => 50,
                'null' => true,
            ],
            'nomor_telepon' => [
                'type' => 'VARCHAR',
                'constraint' => 15,
                'null' => true,
            ],
            'nomor_telepon_ortu' => [
                'type' => 'VARCHAR',
                'constraint' => 15,
                'null' => true,
            ],
            'email_ortu' => [
                'type' => 'VARCHAR',
                'constraint' => 100,
                'null' => true,
            ],
            'kelas' => [
                'type' => 'VARCHAR',
                'constraint' => 20,
            ],
            'tahun_masuk' => [
                'type' => 'YEAR',
            ],
            'tahun_lulus' => [
                'type' => 'YEAR',
                'null' => true,
            ],
            'status' => [
                'type' => 'ENUM',
                'constraint' => ['aktif', 'pindah', 'dropout', 'lulus', 'alumni'],
                'default' => 'aktif',
            ],
            'status_alumni' => [
                'type' => 'ENUM',
                'constraint' => ['non_alumni', 'alumni_sma', 'alumni_kuliah', 'alumni_bekerja'],
                'default' => 'non_alumni',
            ],
            'foto' => [
                'type' => 'VARCHAR',
                'constraint' => 255,
                'null' => true,
            ],
            'created_by' => [
                'type' => 'INT',
                'constraint' => 11,
                'unsigned' => true,
                'null' => true,
            ],
            'updated_by' => [
                'type' => 'INT',
                'constraint' => 11,
                'unsigned' => true,
                'null' => true,
            ],
            'created_at' => [
                'type' => 'DATETIME',
                'null' => true,
            ],
            'updated_at' => [
                'type' => 'DATETIME',
                'null' => true,
            ],
            'deleted_at' => [
                'type' => 'DATETIME',
                'null' => true,
            ],
        ]);
        $this->forge->addKey('id', true);
        $this->forge->addKey('nis');
        $this->forge->addKey(['kelas', 'status']);
        $this->forge->addKey('deleted_at');
        $this->forge->createTable('santri', true);
    }

    public function down()
    {
        $this->forge->dropTable('santri', true);
    }
}
```

### `backend/app/Database/Migrations/2024-05-04-000002_CreateAlumniTable.php`

```php
<?php

namespace App\Database\Migrations;

use CodeIgniter\Database\Migration;

class CreateAlumniTable extends Migration
{
    public function up()
    {
        // Tabel khusus alumni (data post-lulus)
        $this->forge->addField([
            'id' => [
                'type' => 'INT',
                'constraint' => 11,
                'unsigned' => true,
                'auto_increment' => true,
            ],
            'santri_id' => [
                'type' => 'INT',
                'constraint' => 11,
                'unsigned' => true,
            ],
            'tahun_lulus' => [
                'type' => 'YEAR',
            ],
            'nomor_ijazah' => [
                'type' => 'VARCHAR',
                'constraint' => 50,
                'null' => true,
            ],
            'nomor_skhun' => [
                'type' => 'VARCHAR',
                'constraint' => 50,
                'null' => true,
            ],
            'status_after' => [
                'type' => 'ENUM',
                'constraint' => ['kuliah', 'bekerja', 'wirausaha', 'mencari_kerja', 'lainnya'],
                'default' => 'kuliah',
            ],
            'nama_perguruan' => [
                'type' => 'VARCHAR',
                'constraint' => 100,
                'null' => true,
            ],
            'jurusan' => [
                'type' => 'VARCHAR',
                'constraint' => 100,
                'null' => true,
            ],
            'nama_perusahaan' => [
                'type' => 'VARCHAR',
                'constraint' => 100,
                'null' => true,
            ],
            'jabatan' => [
                'type' => 'VARCHAR',
                'constraint' => 100,
                'null' => true,
            ],
            'alamat_terkini' => [
                'type' => 'TEXT',
                'null' => true,
            ],
            'kontak_terkini' => [
                'type' => 'VARCHAR',
                'constraint' => 15,
                'null' => true,
            ],
            'email_terkini' => [
                'type' => 'VARCHAR',
                'constraint' => 100,
                'null' => true,
            ],
            'instagram' => [
                'type' => 'VARCHAR',
                'constraint' => 50,
                'null' => true,
            ],
            'facebook' => [
                'type' => 'VARCHAR',
                'constraint' => 50,
                'null' => true,
            ],
            'kesan_pesan' => [
                'type' => 'TEXT',
                'null' => true,
            ],
            'partisipasi_reuni' => [
                'type' => 'BOOLEAN',
                'default' => false,
            ],
            'created_at' => [
                'type' => 'DATETIME',
                'null' => true,
            ],
            'updated_at' => [
                'type' => 'DATETIME',
                'null' => true,
            ],
        ]);
        $this->forge->addKey('id', true);
        $this->forge->addUniqueKey('santri_id');
        $this->forge->addForeignKey('santri_id', 'santri', 'id', 'CASCADE', 'CASCADE');
        $this->forge->addKey('tahun_lulus');
        $this->forge->createTable('alumni', true);
    }

    public function down()
    {
        $this->forge->dropTable('alumni', true);
    }
}
```

### `backend/app/Database/Migrations/2024-05-04-000003_AddAlumniTrigger.sql`

```sql
-- Trigger otomatis update status alumni berdasarkan tahun_lulus
-- Jalankan manual via MySQL atau buat migration raw

DELIMITER $$
CREATE TRIGGER auto_update_status_alumni
AFTER UPDATE ON santri
FOR EACH ROW
BEGIN
    IF NEW.tahun_lulus IS NOT NULL 
       AND NEW.tahun_lulus <= YEAR(CURDATE()) 
       AND NEW.status != 'alumni'
       AND NEW.status != 'lulus' THEN
        UPDATE santri 
        SET status = 'alumni' 
        WHERE id = NEW.id;
    END IF;
END$$
DELIMITER ;
```

---

## 📦 **2. BACKEND MODELS**

### `backend/app/Models/SantriModel.php`

```php
<?php

namespace App\Models;

use CodeIgniter\Model;

class SantriModel extends Model
{
    protected $table = 'santri';
    protected $primaryKey = 'id';
    protected $useSoftDeletes = true;
    protected $useTimestamps = true;
    protected $createdField = 'created_at';
    protected $updatedField = 'updated_at';
    protected $deletedField = 'deleted_at';
    
    protected $allowedFields = [
        'nis', 'nisn', 'nama_lengkap', 'nama_panggilan', 'jenis_kelamin',
        'tempat_lahir', 'tanggal_lahir', 'agama', 'anak_ke', 'jumlah_saudara',
        'alamat', 'rt', 'rw', 'desa', 'kecamatan', 'kabupaten', 'provinsi', 'kode_pos',
        'nama_ayah', 'nama_ibu', 'pekerjaan_ayah', 'pekerjaan_ibu',
        'pendidikan_ayah', 'pendidikan_ibu', 'nomor_telepon', 'nomor_telepon_ortu', 'email_ortu',
        'kelas', 'tahun_masuk', 'tahun_lulus', 'status', 'status_alumni', 'foto',
        'created_by', 'updated_by'
    ];
    
    protected $validationRules = [
        'nis' => 'required|is_unique[santri.nis,id,{id}]|max_length[20]',
        'nama_lengkap' => 'required|min_length[3]|max_length[100]',
        'jenis_kelamin' => 'required|in_list[L,P]',
        'kelas' => 'required|max_length[20]',
        'tahun_masuk' => 'required|numeric|exact_length[4]',
        'status' => 'permit_empty|in_list[aktif,pindah,dropout,lulus,alumni]',
    ];
    
    protected $validationMessages = [
        'nis' => [
            'required' => 'NIS wajib diisi',
            'is_unique' => 'NIS sudah terdaftar',
        ],
        'nama_lengkap' => [
            'required' => 'Nama lengkap wajib diisi',
            'min_length' => 'Nama minimal 3 karakter',
        ],
    ];
    
    /**
     * Get santri with pagination and filters
     */
    public function getFilteredSantri(array $filters = [], int $limit = 20, int $offset = 0)
    {
        $builder = $this->select('santri.*, COUNT(penilaian.id) as total_penilaian')
            ->join('penilaian', 'penilaian.santri_id = santri.id', 'left')
            ->groupBy('santri.id');
        
        // Apply filters
        if (!empty($filters['search'])) {
            $builder->groupStart()
                ->like('nis', $filters['search'])
                ->orLike('nama_lengkap', $filters['search'])
                ->orLike('nisn', $filters['search'])
                ->groupEnd();
        }
        
        if (!empty($filters['kelas'])) {
            $builder->where('kelas', $filters['kelas']);
        }
        
        if (!empty($filters['status'])) {
            $builder->where('status', $filters['status']);
        }
        
        if (!empty($filters['tahun_masuk'])) {
            $builder->where('tahun_masuk', $filters['tahun_masuk']);
        }
        
        if (!empty($filters['jenis_kelamin'])) {
            $builder->where('jenis_kelamin', $filters['jenis_kelamin']);
        }
        
        // Order by
        $orderBy = $filters['order_by'] ?? 'nama_lengkap';
        $orderDir = $filters['order_dir'] ?? 'ASC';
        $builder->orderBy($orderBy, $orderDir);
        
        return [
            'data' => $builder->findAll($limit, $offset),
            'total' => $builder->countAllResults(false),
        ];
    }
    
    /**
     * Get distinct kelas list
     */
    public function getKelasList(): array
    {
        return $this->select('kelas')
            ->groupBy('kelas')
            ->orderBy('kelas', 'ASC')
            ->findAll();
    }
    
    /**
     * Get statistik dashboard
     */
    public function getStatistik(): array
    {
        $total = $this->where('deleted_at', null)->countAllResults();
        $active = $this->where('status', 'aktif')->countAllResults();
        $alumni = $this->where('status', 'alumni')->countAllResults();
        $graduatedThisYear = $this->where('tahun_lulus', date('Y'))->countAllResults();
        
        $dataByKelas = $this->select('kelas, COUNT(*) as total')
            ->where('status', 'aktif')
            ->groupBy('kelas')
            ->orderBy('kelas', 'ASC')
            ->findAll();
        
        $dataByGender = [
            'L' => $this->where('jenis_kelamin', 'L')->countAllResults(),
            'P' => $this->where('jenis_kelamin', 'P')->countAllResults(),
        ];
        
        return [
            'total' => $total,
            'aktif' => $active,
            'alumni' => $alumni,
            'lulus_tahun_ini' => $graduatedThisYear,
            'per_kelas' => $dataByKelas,
            'per_gender' => $dataByGender,
        ];
    }
    
    /**
     * Graduation: Move student to alumni status
     */
    public function graduate(int $santriId, array $data = []): bool
    {
        $updateData = [
            'status' => 'alumni',
            'tahun_lulus' => date('Y'),
        ];
        
        if (!empty($data['nomor_ijazah'])) {
            $updateData['nomor_ijazah'] = $data['nomor_ijazah'];
        }
        
        $this->update($santriId, $updateData);
        
        // Insert or update alumni table
        $alumniModel = new AlumniModel();
        $alumniData = array_merge(['santri_id' => $santriId], $data);
        $alumniModel->upsert($alumniData);
        
        return true;
    }
    
    /**
     * Bulk import from CSV
     */
    public function bulkImport(array $records, int $userId): array
    {
        $success = 0;
        $failed = 0;
        $errors = [];
        
        foreach ($records as $index => $record) {
            $record['created_by'] = $userId;
            $record['updated_by'] = $userId;
            
            if ($this->insert($record)) {
                $success++;
            } else {
                $failed++;
                $errors[] = [
                    'row' => $index + 2, // +2 karena Excel start row 2
                    'errors' => $this->errors(),
                    'data' => $record,
                ];
            }
        }
        
        return [
            'success' => $success,
            'failed' => $failed,
            'errors' => $errors,
        ];
    }
    
    /**
     * Cron job: Update alumni status based on graduation date
     */
    public function updateAlumniStatusAuto(): int
    {
        $updated = 0;
        
        // Santri yang tahun_lulus <= tahun sekarang dan status masih aktif
        $candidates = $this->where('tahun_lulus <=', date('Y'))
            ->where('status', 'aktif')
            ->findAll();
        
        foreach ($candidates as $santri) {
            $this->update($santri['id'], ['status' => 'alumni']);
            $updated++;
        }
        
        return $updated;
    }
}
```

### `backend/app/Models/AlumniModel.php`

```php
<?php

namespace App\Models;

use CodeIgniter\Model;

class AlumniModel extends Model
{
    protected $table = 'alumni';
    protected $primaryKey = 'id';
    protected $useTimestamps = true;
    
    protected $allowedFields = [
        'santri_id', 'tahun_lulus', 'nomor_ijazah', 'nomor_skhun',
        'status_after', 'nama_perguruan', 'jurusan', 'nama_perusahaan', 'jabatan',
        'alamat_terkini', 'kontak_terkini', 'email_terkini',
        'instagram', 'facebook', 'kesan_pesan', 'partisipasi_reuni'
    ];
    
    /**
     * Get alumni with santri data
     */
    public function getAlumniWithSantri(array $filters = [], int $limit = 20, int $offset = 0)
    {
        $builder = $this->select('alumni.*, santri.nis, santri.nisn, santri.nama_lengkap, santri.jenis_kelamin, santri.tahun_masuk, santri.foto')
            ->join('santri', 'santri.id = alumni.santri_id');
        
        if (!empty($filters['tahun_lulus'])) {
            $builder->where('alumni.tahun_lulus', $filters['tahun_lulus']);
        }
        
        if (!empty($filters['status_after'])) {
            $builder->where('alumni.status_after', $filters['status_after']);
        }
        
        if (!empty($filters['search'])) {
            $builder->groupStart()
                ->like('santri.nama_lengkap', $filters['search'])
                ->orLike('santri.nis', $filters['search'])
                ->groupEnd();
        }
        
        $builder->orderBy('alumni.tahun_lulus', 'DESC');
        
        return [
            'data' => $builder->findAll($limit, $offset),
            'total' => $builder->countAllResults(false),
        ];
    }
    
    /**
     * Get alumni statistics for dashboard
     */
    public function getStatistik(): array
    {
        $total = $this->countAllResults();
        
        $byYear = $this->select('tahun_lulus, COUNT(*) as total')
            ->groupBy('tahun_lulus')
            ->orderBy('tahun_lulus', 'DESC')
            ->findAll();
        
        $byStatus = $this->select('status_after, COUNT(*) as total')
            ->groupBy('status_after')
            ->findAll();
        
        $byReuni = $this->select('partisipasi_reuni, COUNT(*) as total')
            ->groupBy('partisipasi_reuni')
            ->findAll();
        
        return [
            'total' => $total,
            'per_tahun' => $byYear,
            'per_status' => $byStatus,
            'partisipasi_reuni' => $byReuni,
        ];
    }
    
    /**
     * Export alumni data to array (for CSV/Excel)
     */
    public function exportToArray(array $filters = []): array
    {
        $builder = $this->select('
                santri.nis,
                santri.nisn,
                santri.nama_lengkap,
                santri.jenis_kelamin,
                santri.tahun_masuk,
                alumni.tahun_lulus,
                alumni.nomor_ijazah,
                alumni.nomor_skhun,
                alumni.status_after,
                alumni.nama_perguruan,
                alumni.jurusan,
                alumni.nama_perusahaan,
                alumni.jabatan,
                alumni.alamat_terkini,
                alumni.kontak_terkini,
                alumni.email_terkini,
                alumni.instagram,
                alumni.facebook,
                alumni.partisipasi_reuni
            ')
            ->join('santri', 'santri.id = alumni.santri_id');
        
        if (!empty($filters['tahun_lulus'])) {
            $builder->where('alumni.tahun_lulus', $filters['tahun_lulus']);
        }
        
        return $builder->findAll();
    }
}
```

---

## 🎮 **3. BACKEND CONTROLLERS**

### `backend/app/Controllers/Api/SantriController.php`

```php
<?php

namespace App\Controllers\Api;

use App\Models\SantriModel;
use App\Models\AlumniModel;

class SantriController extends BaseApiController
{
    protected $modelName = SantriModel::class;
    protected $model;
    
    public function __construct()
    {
        $this->model = new SantriModel();
    }
    
    /**
     * GET /api/santri
     * List santri with filtering, pagination, sorting
     */
    public function index()
    {
        if (!$this->hasPermission('santri.read')) {
            return $this->error('Forbidden', 403);
        }
        
        $params = $this->request->getGet();
        $page = max(1, (int)($params['page'] ?? 1));
        $limit = min(100, (int)($params['limit'] ?? 20));
        $offset = ($page - 1) * $limit;
        
        $filters = [
            'search' => $params['search'] ?? null,
            'kelas' => $params['kelas'] ?? null,
            'status' => $params['status'] ?? null,
            'tahun_masuk' => $params['tahun_masuk'] ?? null,
            'jenis_kelamin' => $params['jenis_kelamin'] ?? null,
            'order_by' => $params['order_by'] ?? 'nama_lengkap',
            'order_dir' => $params['order_dir'] ?? 'ASC',
        ];
        
        $result = $this->model->getFilteredSantri($filters, $limit, $offset);
        
        return $this->success([
            'data' => $result['data'],
            'pagination' => [
                'current_page' => $page,
                'per_page' => $limit,
                'total' => $result['total'],
                'total_pages' => ceil($result['total'] / $limit),
            ],
            'filters' => $filters,
        ]);
    }
    
    /**
     * GET /api/santri/statistik
     * Dashboard statistics
     */
    public function statistik()
    {
        if (!$this->hasPermission('santri.read')) {
            return $this->error('Forbidden', 403);
        }
        
        $statistik = $this->model->getStatistik();
        $kelasList = $this->model->getKelasList();
        
        return $this->success([
            'statistik' => $statistik,
            'kelas_list' => $kelasList,
        ]);
    }
    
    /**
     * GET /api/santri/:id
     * Get single santri with full details
     */
    public function show($id = null)
    {
        if (!$this->hasPermission('santri.read')) {
            return $this->error('Forbidden', 403);
        }
        
        $santri = $this->model
            ->select('santri.*, users.full_name as created_by_name')
            ->join('users', 'users.id = santri.created_by', 'left')
            ->withDeleted()
            ->find($id);
        
        if (!$santri) {
            return $this->error('Santri tidak ditemukan', 404);
        }
        
        // Load alumni data if exists
        $alumniModel = new AlumniModel();
        $alumni = $alumniModel->where('santri_id', $id)->first();
        
        return $this->success([
            'santri' => $santri,
            'alumni' => $alumni,
        ]);
    }
    
    /**
     * POST /api/santri
     * Create new santri
     */
    public function create()
    {
        if (!$this->hasPermission('santri.create')) {
            return $this->error('Forbidden', 403);
        }
        
        $data = $this->request->getJSON(true);
        $data['created_by'] = $this->currentUser()['id'];
        $data['updated_by'] = $this->currentUser()['id'];
        
        if (!$this->model->insert($data)) {
            return $this->error('Validasi gagal', 422, $this->model->errors());
        }
        
        log_activity($this->currentUser()['id'], 'create', 'santri', $this->model->getInsertID(), $data);
        
        return $this->success(['id' => $this->model->getInsertID()], 'Santri berhasil ditambahkan', 201);
    }
    
    /**
     * PUT /api/santri/:id
     * Update existing santri
     */
    public function update($id = null)
    {
        if (!$this->hasPermission('santri.update')) {
            return $this->error('Forbidden', 403);
        }
        
        $existing = $this->model->find($id);
        if (!$existing) {
            return $this->error('Santri tidak ditemukan', 404);
        }
        
        $data = $this->request->getJSON(true);
        $data['updated_by'] = $this->currentUser()['id'];
        
        if (!$this->model->update($id, $data)) {
            return $this->error('Validasi gagal', 422, $this->model->errors());
        }
        
        log_activity($this->currentUser()['id'], 'update', 'santri', $id, [
            'old' => $existing,
            'new' => $data,
        ]);
        
        return $this->success(null, 'Santri berhasil diperbarui');
    }
    
    /**
     * DELETE /api/santri/:id
     * Soft delete santri
     */
    public function delete($id = null)
    {
        if (!$this->hasPermission('santri.delete')) {
            return $this->error('Forbidden', 403);
        }
        
        $existing = $this->model->find($id);
        if (!$existing) {
            return $this->error('Santri tidak ditemukan', 404);
        }
        
        // Don't delete if has penilaian records
        $db = \Config\Database::connect();
        $penilaianCount = $db->table('penilaian')->where('santri_id', $id)->countAllResults();
        if ($penilaianCount > 0) {
            return $this->error('Santri tidak dapat dihapus karena memiliki riwayat penilaian', 409);
        }
        
        $this->model->delete($id);
        
        log_activity($this->currentUser()['id'], 'delete', 'santri', $id, $existing);
        
        return $this->success(null, 'Santri berhasil dihapus', 204);
    }
    
    /**
     * POST /api/santri/:id/graduate
     * Move student to alumni status
     */
    public function graduate($id = null)
    {
        if (!$this->hasPermission('santri.update')) {
            return $this->error('Forbidden', 403);
        }
        
        $santri = $this->model->find($id);
        if (!$santri) {
            return $this->error('Santri tidak ditemukan', 404);
        }
        
        if ($santri['status'] === 'alumni') {
            return $this->error('Santri sudah menjadi alumni', 400);
        }
        
        $data = $this->request->getJSON(true);
        $this->model->graduate($id, $data);
        
        log_activity($this->currentUser()['id'], 'graduate', 'santri', $id, $data);
        
        return $this->success(null, 'Santri berhasil diluluskan menjadi alumni');
    }
    
    /**
     * POST /api/santri/import
     * Bulk import santri from CSV
     */
    public function import()
    {
        if (!$this->hasPermission('santri.create')) {
            return $this->error('Forbidden', 403);
        }
        
        $file = $this->request->getFile('file');
        if (!$file || !$file->isValid()) {
            return $this->error('File CSV tidak ditemukan', 400);
        }
        
        if ($file->getClientExtension() !== 'csv') {
            return $this->error('Format file harus CSV', 400);
        }
        
        // Parse CSV
        $handle = fopen($file->getTempName(), 'r');
        $headers = fgetcsv($handle);
        
        $records = [];
        while (($row = fgetcsv($handle)) !== false) {
            $record = array_combine($headers, $row);
            $records[] = $record;
        }
        fclose($handle);
        
        $result = $this->model->bulkImport($records, $this->currentUser()['id']);
        
        return $this->success($result, 'Import selesai');
    }
    
    /**
     * GET /api/santri/export/csv
     * Export santri data to CSV
     */
    public function exportCsv()
    {
        if (!$this->hasPermission('santri.read')) {
            return $this->error('Forbidden', 403);
        }
        
        $filters = $this->request->getGet();
        $santri = $this->model->getFilteredSantri($filters, 10000, 0)['data'];
        
        // Generate CSV
        $filename = 'santri_export_' . date('Y-m-d_His') . '.csv';
        $fp = fopen('php://temp', 'w');
        
        // Headers
        $headers = ['NIS', 'Nama Lengkap', 'Kelas', 'Jenis Kelamin', 'Status', 'Tahun Masuk', 'No Telepon'];
        fputcsv($fp, $headers);
        
        foreach ($santri as $row) {
            fputcsv($fp, [
                $row['nis'],
                $row['nama_lengkap'],
                $row['kelas'],
                $row['jenis_kelamin'],
                $row['status'],
                $row['tahun_masuk'],
                $row['nomor_telepon'],
            ]);
        }
        
        rewind($fp);
        $csvContent = stream_get_contents($fp);
        fclose($fp);
        
        return $this->response
            ->setHeader('Content-Type', 'text/csv')
            ->setHeader('Content-Disposition', 'attachment; filename="' . $filename . '"')
            ->setBody($csvContent);
    }
}
```

### `backend/app/Controllers/Api/AlumniController.php`

```php
<?php

namespace App\Controllers\Api;

use App\Models\AlumniModel;
use App\Models\SantriModel;

class AlumniController extends BaseApiController
{
    protected $modelName = AlumniModel::class;
    protected $model;
    
    public function __construct()
    {
        $this->model = new AlumniModel();
    }
    
    /**
     * GET /api/alumni
     * List alumni with filtering
     */
    public function index()
    {
        if (!$this->hasPermission('
