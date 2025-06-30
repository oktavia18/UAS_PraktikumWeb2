## 👤 Profil Mahasiswa

| Atribut         | Keterangan            |
| --------------- | --------------------- |
| **Nama**        | Oktavia Rizkha Kurniawati         |
| **NIM**         | 312310509             |
| **Kelas**       | TI.23.A.5             |
| **Mata Kuliah** | Pemrograman Website 2 |

# 📌 Tugas Praktikum 7-11

# Praktikum 7: Relasi Tabel dan Query Builder

## Deskripsi
Praktikum ini merupakan pengembangan dari praktikum sebelumnya. Kali ini, kita akan belajar bagaimana menghubungkan dua tabel di database menggunakan relasi One-to-Many, serta menampilkan data yang saling terkait menggunakan Query Builder di CodeIgniter 4. Praktikum ini dijalankan menggunakan Laragon sebagai local server.

## Tujuan Praktikum
1. Memahami dasar relasi antar tabel (One-to-Many)
2. Menghubungkan tabel artikel dan kategori
3. Menggunakan Query Builder untuk join data antar tabel
4. Menampilkan data artikel beserta nama kategorinya

## Langkah-langkah Praktikum
### 1. Persiapan Database

Memastikan MySQL Server berjalan dan membuka database `lab_ci4`.

### 2. Membuat Tabel Kategori

Membuat tabel baru bernama `kategori` dengan struktur:

- `id_kategori` (INT, PRIMARY KEY, AUTO_INCREMENT)
- `nama_kategori` (VARCHAR 100)
- `slug_kategori` (VARCHAR 100)

**Query SQL:**

```sql
CREATE TABLE kategori (
    id_kategori INT(11) AUTO_INCREMENT,
    nama_kategori VARCHAR(100) NOT NULL,
    slug_kategori VARCHAR(100),
    PRIMARY KEY (id_kategori)
);
```

**Screenshot:**
![table_kategori](https://github.com/user-attachments/assets/48ed31e0-2837-4113-9205-9d28fe4ec0cd)

### 3. Modifikasi Tabel Artikel

Menambahkan foreign key `id_kategori` pada tabel `artikel` untuk membuat relasi dengan tabel `kategori`.

**Query SQL:**

```sql
ALTER TABLE artikel
ADD COLUMN id_kategori INT(11),
ADD CONSTRAINT fk_kategori_artikel
FOREIGN KEY (id_kategori) REFERENCES kategori(id_kategori);
```

### 4. Membuat Model `KategoriModel`

Membuat file `KategoriModel.php` di folder `app/Models/` untuk mengelola data kategori.

**Kode KategoriModel.php:**

```php
<?php
namespace App\Models;
use CodeIgniter\Model;

class KategoriModel extends Model
{
    protected $table = 'kategori';
    protected $primaryKey = 'id_kategori';
    protected $useAutoIncrement = true;
    protected $allowedFields = ['nama_kategori', 'slug_kategori'];
}
```


### 5. Modifikasi Model Artikel

Memodifikasi `ArtikelModel.php` dengan menambahkan method `getArtikelDenganKategori()` untuk melakukan join dengan tabel kategori.

**Kode ArtikelModel.php:**

```php
<?php
namespace App\Models;
use CodeIgniter\Model;

class ArtikelModel extends Model
{
    protected $table = 'artikel';
    protected $primaryKey = 'id';
    protected $useAutoIncrement = true;
    protected $allowedFields = ['judul', 'isi', 'status', 'slug', 'gambar', 'id_kategori'];

    public function getArtikelDenganKategori()
    {
        return $this->db->table('artikel')
            ->select('artikel.*, kategori.nama_kategori')
            ->join('kategori', 'kategori.id_kategori = artikel.id_kategori')
            ->get()
            ->getResultArray();
    }
}
```


### 6. Modifikasi Controller Artikel.php

Memperbarui `Artikel.php` controller untuk:

- Menggunakan method join dari model
- Menambahkan filter berdasarkan kategori
- Menangani kategori pada form tambah dan edit artikel

**Kode Controller Artikel.php:**

```php
<?php
namespace App\Controllers;
use App\Models\ArtikelModel;
use App\Models\KategoriModel;

class Artikel extends BaseController
{
    public function index()
    {
        $title = 'Daftar Artikel';
        $model = new ArtikelModel();
        $artikel = $model->getArtikelDenganKategori(); // Use the new method
        return view('artikel/index', compact('artikel', 'title'));
    }

    public function admin_index()
    {
        $title = 'Daftar Artikel (Admin)';
        $model = new ArtikelModel();

        // Get search keyword
        $q = $this->request->getVar('q') ?? '';

        // Get category filter
        $kategori_id = $this->request->getVar('kategori_id') ?? '';

        $data = [
            'title' => $title,
            'q' => $q,
            'kategori_id' => $kategori_id,
        ];

        // Building the query
        $builder = $model->table('artikel')
            ->select('artikel.*, kategori.nama_kategori')
            ->join('kategori', 'kategori.id_kategori = artikel.id_kategori');

        // Apply search filter if keyword is provided
        if ($q != '') {
            $builder->like('artikel.judul', $q);
        }

        // Apply category filter if category_id is provided
        if ($kategori_id != '') {
            $builder->where('artikel.id_kategori', $kategori_id);
        }

        // Apply pagination
        $data['artikel'] = $builder->paginate(10);
        $data['pager'] = $model->pager;

        // Fetch all categories for the filter dropdown
        $kategoriModel = new KategoriModel();
        $data['kategori'] = $kategoriModel->findAll();

        return view('artikel/admin_index', $data);
    }

    public function add()
    {
        // Validation...
        if ($this->request->getMethod() == 'post' && $this->validate([
            'judul' => 'required',
            'id_kategori' => 'required|integer' // Ensure id_kategori is required and an integer
        ])) {
            $model = new ArtikelModel();
            $model->insert([
                'judul' => $this->request->getPost('judul'),
                'isi' => $this->request->getPost('isi'),
                'slug' => url_title($this->request->getPost('judul')),
                'id_kategori' => $this->request->getPost('id_kategori')
            ]);
            return redirect()->to('/admin/artikel');
        } else {
            $kategoriModel = new KategoriModel();
            $data['kategori'] = $kategoriModel->findAll(); // Fetch categories for the form
            $data['title'] = "Tambah Artikel";
            return view('artikel/form_add', $data);
        }
    }

    public function edit($id)
    {
        $model = new ArtikelModel();
        if ($this->request->getMethod() == 'post' && $this->validate([
            'judul' => 'required',
            'id_kategori' => 'required|integer'
        ])) {
            $model->update($id, [
                'judul' => $this->request->getPost('judul'),
                'isi' => $this->request->getPost('isi'),
                'id_kategori' => $this->request->getPost('id_kategori')
            ]);
            return redirect()->to('/admin/artikel');
        } else {
            $data['artikel'] = $model->find($id);
            $kategoriModel = new KategoriModel();
            $data['kategori'] = $kategoriModel->findAll(); // Fetch categories for the form
            $data['title'] = "Edit Artikel";
            return view('artikel/form_edit', $data);
        }
    }

    public function delete($id)
    {
        $model = new ArtikelModel();
        $model->delete($id);
        return redirect()->to('/admin/artikel');
    }

    public function view($slug)
    {
        $model = new ArtikelModel();
        $data['artikel'] = $model->where('slug', $slug)->first();
        if (empty($data['artikel'])) {
            throw new \CodeIgniter\Exceptions\PageNotFoundException('Cannot find the article.');
        }
        $data['title'] = $data['artikel']['judul'];
        return view('artikel/detail', $data);
    }
}
```


### 7. Modifikasi Tampilan (View)

Pada bagian ini, kita akan menyesuaikan tampilan halaman website agar menampilkan data kategori yang telah dikaitkan dengan artikel. Beberapa file view perlu dimodifikasi untuk mendukung fitur relasi One-to-Many ini.

---

#### a. `index.php` (Halaman Depan)

Tampilan ini digunakan oleh pengguna untuk melihat daftar artikel secara umum. Modifikasi dilakukan agar menampilkan nama kategori dari setiap artikel.

**Kode `index.php`:**

```php
<?= $this->include('template/header'); ?>
<?php if ($artikel): foreach ($artikel as $row): ?>
<article class="entry">
    <h2><a href="<?= base_url('/artikel/' . $row['slug']); ?>"><?= $row['judul']; ?></a></h2>
    <p>Kategori: <?= $row['nama_kategori'] ?></p>
    <img src="<?= base_url('/gambar/' . $row['gambar']); ?>" alt="<?= $row['judul']; ?>">
    <p><?= substr($row['isi'], 0, 200); ?></p>
</article>
<hr class="divider" />
<?php endforeach; else: ?>
<article class="entry">
    <h2>Belum ada data.</h2>
</article>
<?php endif; ?>
<?= $this->include('template/footer'); ?>

#### b. admin_index.php (Halaman Admin)
- File ini menampilkan daftar artikel di halaman admin. Modifikasi dilakukan untuk:
- Menampilkan kolom kategori pada tabel artikel
- Menyediakan dropdown filter berdasarkan kategori
- Menambahkan fitur pencarian berdasarkan judul artikel

***Kode admin_index.php:***

php
Salin
Edit
<?= $this->include('template/admin_header'); ?>
<h2><?= $title; ?></h2>
<div class="row mb-3">
    <div class="col-md-6">
        <form method="get" class="form-inline">
            <input type="text" name="q" value="<?= $q; ?>" placeholder="Cari judul artikel" class="form-control mr-2">
            <select name="kategori_id" class="form-control mr-2">
                <option value="">Semua Kategori</option>
                <?php foreach ($kategori as $k): ?>
                <option value="<?= $k['id_kategori']; ?>" <?= ($kategori_id == $k['id_kategori']) ? 'selected' : ''; ?>><?= $k['nama_kategori']; ?></option>
                <?php endforeach; ?>
            </select>
            <input type="submit" value="Cari" class="btn btn-primary">
        </form>
    </div>
</div>
<table class="table">
    <thead>
        <tr>
            <th>ID</th>
            <th>Judul</th>
            <th>Kategori</th>
            <th>Status</th>
            <th>Aksi</th>
        </tr>
    </thead>
    <tbody>
        <?php if (count($artikel) > 0): ?>
        <?php foreach ($artikel as $row): ?>
        <tr>
            <td><?= $row->id; ?></td>
            <td>
                <b><?= $row->judul; ?></b>
                <p><small><?= substr($row->isi, 0, 50); ?></small></p>
            </td>
            <td><?= $row->nama_kategori; ?></td>
            <td><?= $row->status; ?></td>
            <td>
                <a class="btn btn-sm btn-info" href="<?= base_url('/admin/artikel/edit/' . $row->id); ?>">Ubah</a>
                <a class="btn btn-sm btn-danger" onclick="return confirm('Yakin menghapus data?');" href="<?= base_url('/admin/artikel/delete/' . $row->id); ?>">Hapus</a>
            </td>
        </tr>
        <?php endforeach; ?>
        <?php else: ?>
        <tr>
            <td colspan="5">Tidak ada data.</td>
        </tr>
        <?php endif; ?>
    </tbody>
</table>
<?= $pager->only(['q', 'kategori_id'])->links(); ?>
<?= $this->include('template/admin_footer'); ?>
```


#### c. form_add.php (Form Tambah Artikel)

Menambahkan dropdown untuk memilih kategori saat menambah artikel baru.

**Kode form_add.php:**

```php
<?= $this->include('template/admin_header'); ?>
<h2><?= $title; ?></h2>
<form action="" method="post">
    <p>
        <label for="judul">Judul</label>
        <input type="text" name="judul" id="judul" required>
    </p>
    <p>
        <label for="isi">Isi</label>
        <textarea name="isi" id="isi" cols="50" rows="10"></textarea>
    </p>
    <p>
        <label for="id_kategori">Kategori</label>
        <select name="id_kategori" id="id_kategori" required>
            <?php foreach($kategori as $k): ?>
            <option value="<?= $k['id_kategori']; ?>"><?= $k['nama_kategori']; ?></option>
            <?php endforeach; ?>
        </select>
    </p>
    <p><input type="submit" value="Kirim" class="btn btn-large"></p>
</form>
<?= $this->include('template/admin_footer'); ?>
```


#### d. form_edit.php (Form Edit Artikel)

Menambahkan dropdown kategori dengan nilai yang sudah terpilih sesuai data artikel.

**Kode form_edit.php:**

```php
<?= $this->include('template/admin_header'); ?>
<h2><?= $title; ?></h2>
<form action="" method="post">
    <p>
        <label for="judul">Judul</label>
        <input type="text" name="judul" value="<?= $artikel['judul']; ?>" id="judul" required>
    </p>
    <p>
        <label for="isi">Isi</label>
        <textarea name="isi" id="isi" cols="50" rows="10"><?= $artikel['isi']; ?></textarea>
    </p>
    <p>
        <label for="id_kategori">Kategori</label>
        <select name="id_kategori" id="id_kategori" required>
            <?php foreach($kategori as $k): ?>
            <option value="<?= $k['id_kategori']; ?>" <?= ($artikel['id_kategori'] == $k['id_kategori']) ? 'selected' : ''; ?>><?= $k['nama_kategori']; ?></option>
            <?php endforeach; ?>
        </select>
    </p>
    <p><input type="submit" value="Kirim" class="btn btn-large"></p>
</form>
<?= $this->include('template/admin_footer'); ?>
```



## 🧪 7. Testing Fitur

Berikut adalah hasil pengujian dari fitur-fitur utama dalam aplikasi:

### ✅ 1. Menampilkan Daftar Artikel Beserta Kategori
Menampilkan semua artikel yang sudah dibuat, lengkap dengan informasi kategori masing-masing.

🖼️ **Tampilan Daftar Artikel :**

![Screenshot 2025-06-30 200244](https://github.com/user-attachments/assets/370fa639-c402-4216-bc87-89cf3331a576)

---

### ✅ 2. Menambah Artikel Baru dengan Kategori
Fitur untuk menambahkan artikel baru dan memilih kategori yang sesuai.

🖼️ **Tampilan Form Tambah Artikel:**

![Screenshot 2025-06-30 200806](https://github.com/user-attachments/assets/9427c459-a7ab-40cd-b36d-f5835fa981a4)

---

### ✅ 3. Mengedit Artikel dan Ubah Kategori
Fitur untuk memperbarui isi artikel sekaligus mengganti kategori jika diperlukan.

🖼️ **Tampilan Form Edit Artikel:**

![Screenshot 2025-06-30 201032](https://github.com/user-attachments/assets/f566376a-8b62-43d5-b61e-4058d3c38f17)


🖼️ **Tampilan Setelah Edit Artikel:**

![Screenshot 2025-06-30 201728](https://github.com/user-attachments/assets/b21bf1c6-3f4e-4eb8-994d-2bfea47acd8f)

---

### ✅ 4. Menghapus Artikel
Fitur untuk menghapus artikel yang sudah tidak diperlukan.

🖼️ **Tampilan Konfirmasi/Hapus Artikel:**

![Screenshot 2025-06-30 201918](https://github.com/user-attachments/assets/f3cc9c99-409c-489a-9e96-fd21f1e623bd)


🖼️ **Tampilan Setelah Artikel Dihapus:**

![Screenshot 2025-06-30 202023](https://github.com/user-attachments/assets/d3f4a478-3a5e-492c-897c-4bec91c2d0f8)

---
## ✅ Kesimpulan

Praktikum 7 ini telah berhasil mengimplementasikan relasi **One-to-Many** antara tabel `artikel` dan `kategori` dalam aplikasi berbasis **CodeIgniter 4**. Seluruh proses dilakukan secara terstruktur, mulai dari pembuatan tabel, pemrograman model, controller, hingga view.

---

## 🔧 Fitur-Fitur yang Berhasil Diterapkan

- **Relasi Tabel One-to-Many**  
  Menghubungkan artikel ke kategori menggunakan foreign key `id_kategori`.

- **Join Tabel dengan Query Builder**  
  Data artikel ditampilkan bersama nama kategori melalui `JOIN` antar tabel.

- **Filter Berdasarkan Kategori**  
  Dropdown filter di halaman admin untuk menampilkan artikel per kategori.

- **Pencarian Artikel (Search)**  
  Form pencarian berdasarkan judul artikel menggunakan metode `like()`.

- **Tambah Artikel**  
  Form tambah artikel lengkap dengan dropdown kategori.

- **Edit Artikel**  
  Form edit artikel yang otomatis menampilkan kategori yang sudah dipilih.

- **Hapus Artikel**  
  Konfirmasi hapus dan penghapusan data artikel dari database.

- **Pagination & Tabel Interaktif**  
  Data ditampilkan dalam tabel yang dapat dipaginasi untuk navigasi yang lebih mudah.

---

## 🧰 Teknologi & Tools yang Digunakan

| Komponen        | Teknologi / Tools                      |
|-----------------|-----------------------------------------|
| **Framework**   | CodeIgniter 4 (PHP MVC Framework)       |
| **Local Server**| Laragon (Apache + MySQL)                |
| **Database**    | MySQL                                   |
| **UI Framework**| Bootstrap 5                             |
| **View Engine** | CodeIgniter View (`.php`)               |
| **Editor**      | Visual Studio Code                      |
| **Debugging**   | CI Debug Toolbar                        |
| **Front-End**   | DataTables (untuk tabel admin)          |

---

## 🎓 Manfaat yang Diperoleh

- Memahami konsep **relasi antar tabel** dan cara mengimplementasikannya langsung.
- Belajar menggunakan **Query Builder CodeIgniter** untuk `JOIN`, `FILTER`, dan `PAGINATE`.
- Menyusun kode berdasarkan konsep **MVC (Model–View–Controller)**.
- Meningkatkan keterampilan pengolahan **data dinamis** di halaman web.
- Mempersiapkan fondasi untuk membangun aplikasi **CRUD skala menengah hingga besar**.

---

## 🙌 Penutup

Praktikum ini memberikan pengalaman langsung dalam membangun fitur yang umum digunakan dalam aplikasi web, terutama dalam pengelolaan data relasional. Dengan menggabungkan konsep **database**, **backend (PHP/CI4)**, dan antarmuka pengguna yang rapi, aplikasi yang dibangun menjadi lebih fungsional dan mudah dikembangkan.

---

> Terima kasih telah membaca dokumentasi ini.  
> Semoga bermanfaat dan menginspirasi pengembangan proyek selanjutnya. 🚀
