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
![alt text](Gambar/image.png)


## 💻 Panduan Awal Menjalankan Web Menggunakan Laragon

### 1. Menyalakan Laragon
- Buka aplikasi **Laragon**
- Pastikan **Apache** dan **MySQL** sudah berjalan

### 2. Menempatkan Folder Proyek
- Salin folder project CodeIgniter 4 ke direktori:
