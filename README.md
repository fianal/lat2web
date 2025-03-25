# lab7_php_ci

| Data diri| 
|-----------------|
| Nama : Alfian Nur Rizki  | 
| Kelas : TI 23.A6 | 
| NIM : 312310665 | 

<h1>Praktikum 2: Framework Lanjutan (CRUD)</h1>

## Membuat Database: Studi Kasus Data Artikel

| Field  | Tipe Data | Ukuran | Keterangan  |
|--------|----------|--------|-------------|
| id     | INT      | 11     |             |
| judul  | VARCHAR  | 200    |             |
| isi    | TEXT     |        |             |
| gambar | VARCHAR  | 200    |             |
| status | TINYINT  | 1      | DEFAULT 0   |
| slug   | VARCHAR  | 200    |             |

<p>Membuat Database</p>

```
CREATE DATABASE lab_ci4;
```
<p>Membuat Tabel</p>

```
CREATE TABLE artikel (
  id INT(11) auto_increment,
  judul VARCHAR(200) NOT NULL,
  isi TEXT,
  gambar VARCHAR(200),
  status TINYINT(1) DEFAULT 0,
  slug VARCHAR(200),
  PRIMARY KEY(id)
);
```

## Konfigurasi koneksi database

<P>Selanjutnya membuat konfigurasi untuk menghubungkan dengan database server. Konfigurasi dapat dilakukan dengan du acara, yaitu pada file app/config/database.php atau menggunakan file .env. Pada praktikum ini kita gunakan konfigurasi pada file .env.</P>

## Membuat Model

<p>Selanjutnya adalah membuat Model untuk memproses data Artikel. Buat file baru pada direktori app/Models dengan nama ArtikelModel.php</p>

```
<?php

namespace App\Models;

use CodeIgniter\Model;

class ArtikelModel extends Model
{
        protected $table = 'artikel';
        protected $primaryKey = 'id';
        protected $useAutoIncrement = true;
        protected $allowedFields = ['judul', 'isi', 'status', 'slug',
    'gambar'];
}
```

## Membuat Controller

<p>Buat Controller baru dengan nama Artikel.php pada direktori app/Controllers.</p>

```
<?php
namespace App\Controllers;
use App\Models\ArtikelModel;
class Artikel extends BaseController
{
  public function index()
  {
    $title = 'Daftar Artikel';
    $model = new ArtikelModel();
    $artikel = $model->findAll();
    return view('artikel/index', compact('artikel', 'title'));
  }
}
```

## Membuat View

<p>Buat direktori baru dengan nama artikel pada direktori app/views, kemudian buat file baru dengan nama index.php.</p>

```
<?= $this->include('template/header'); ?>

<?php if($artikel): foreach($artikel as $row): ?>
<article class="entry">
  <h2<a href="<?= base_url('/artikel/' . $row['slug']);?>"><?=
$row['judul']; ?></a>
</h2>
  <img src="<?= base_url('/gambar/' . $row['gambar']);?>" alt="<?=
$row['judul']; ?>">
  <p><?= substr($row['isi'], 0, 200); ?></p>
</article>
<hr class="divider" />
<?php endforeach; else: ?>
<article class="entry">
  <h2>Belum ada data.</h2>
</article>
<?php endif; ?>

<?= $this->include('template/footer'); ?>
```
<p> Selanjutnya buka browser kembali, dengan mengakses url http://localhost:8080/artikel </p>

![img](https://github.com/fianal/lat2web/blob/main/img_lat2web/no_data.png)

<p>Belum ada data yang diampilkan. Kemudian coba tambahkan beberapa data pada database agar dapat ditampilkan datanya.</p>

```
INSERT INTO artikel (judul, isi, slug) VALUE
('Artikel pertama', 'Lorem Ipsum adalah contoh teks atau dummy dalam industri percetakan dan penataan huruf atau typesetting. Lorem Ipsum telah
menjadi standar contoh teks sejak tahun 1500an, saat seorang tukang cetak
yang tidak dikenal mengambil sebuah kumpulan teks dan mengacaknya untuk
menjadi sebuah buku contoh huruf.', 'artikel-pertama'),
('Artikel kedua', 'Tidak seperti anggapan banyak orang, Lorem Ipsum
bukanlah teks-teks yang diacak. Ia berakar dari sebuah naskah sastra latin
klasik dari era 45 sebelum masehi, hingga bisa dipastikan usianya telah
mencapai lebih dari 2000 tahun.', 'artikel-kedua');
```

Refresh kembali browser, sehingga akan ditampilkan hasilnya.

![img](https://github.com/fianal/lat2web/blob/main/img_lat2web/tampilan_art.png)

## Membuat Tampilan Detail Artikel

<p>Tampilan pada saat judul berita di klik maka akan diarahkan ke halaman yang berbeda. Tambahkan fungsi baru pada Controller Artikel dengan nama view().</p>

```
public function view($slug)
{
  $model = new ArtikelModel();
  $artikel = $model->where([
  'slug' => $slug
  ])->first();
  // Menampilkan error apabila data tidak ada.
  if (!$artikel)
  {
  throw PageNotFoundException::forPageNotFound();
  }
  $title = $artikel['judul'];
  return view('artikel/detail', compact('artikel', 'title'));
}
```

## Membuat View Detail

<p>Buat view baru untuk halaman detail dengan nama app/views/artikel/detail.php.</p>

```
<?= $this->include('template/header'); ?>

<article class="entry">
    <h2><?= $artikel['judul']; ?></h2>
    <img src="<?= base_url('/gambar/' . $artikel['gambar']);?>" alt="<?=
$artikel['judul']; ?>">
    <p><?= $artikel['isi']; ?></p>
</article>

<?= $this->include('template/footer'); ?>
```

## Membuat Routing Untuk Artikel Detail

<p>Buka Kembali file app/config/Routes.php, kemudian tambahkan routing untuk artikel detail.</p>

```
$routes->get('/artikel/(:any)', 'Artikel::view/$1');
```

![img](https://github.com/fianal/lat2web/blob/main/img_lat2web/isi_art.png)

## Membuat Menu Admin

<p>Menu admin adalah untuk proses CRUD data artikel. Buat method baru pada Controller Artikel dengan nama admin_index().</p>

```
public function admin_index()
{
$title = 'Daftar Artikel';
$model = new ArtikelModel();
$artikel = $model->findAll();
return view('artikel/admin_index', compact('artikel', 'title'));
}
```

<p>Selanjutnya buat view untuk tampilan admin dengan nama admin_index.php</p>

```
<?= $this->include('template/admin_header'); ?>

<table class="table">
    <thead>
        <tr>
            <th>ID</th>
            <th>Judul</th>
            <th>Status</th>
            <th>Aksi</th>
        </tr>
    </thead>
    <tbody>
        <?php if ($artikel): ?>
            <?php foreach ($artikel as $row): ?>
                <tr>
                    <td><?= esc($row['id']); ?></td>
                    <td>
                        <b><?= esc($row['judul']); ?></b>
                        <p><small><?= esc(substr($row['isi'], 0, 50)); ?>...</small></p>
                    </td>
                    <td><?= esc($row['status']); ?></td>
                    <td>
                        <a class="btn" href="<?= base_url('/admin/artikel/edit/' . $row['id']); ?>">Ubah</a>
                        <a class="btn btn-danger" onclick="return confirm('Yakin menghapus data?');" 
                           href="<?= base_url('/admin/artikel/delete/' . $row['id']); ?>">Hapus</a>
                    </td>
                </tr>
            <?php endforeach; ?>
        <?php else: ?>
            <tr>
                <td colspan="4">Belum ada data.</td>
            </tr>
        <?php endif; ?>
    </tbody>
    <tfoot>
        <tr>
            <th>ID</th>
            <th>Judul</th>
            <th>Status</th>
            <th>Aksi</th>
        </tr>
    </tfoot>
</table>

<?= $this->include('template/admin_footer'); ?>
```

<p>Tambahkan Routing untuk menu Admin</p>

```
$routes->group('admin', function($routes) {
$routes->get('artikel', 'Artikel::admin_index');
$routes->add('artikel/add', 'Artikel::add');
$routes->add('artikel/edit/(:any)', 'Artikel::edit/$1');
$routes->get('artikel/delete/(:any)', 'Artikel::delete/$1');
});
```

<p>Akses menu admin dengan url http://localhost:8080/admin/artikel</p>

![img](https://github.com/fianal/lat2web/blob/main/img_lat2web/CRUD_art.png)

## Menambah Data Artikel

<p>Tambahkan fungsi/method baru pada Controller Artikel dengan nama add().</p>

```
public function add()
{
// validasi data.
$validation = \Config\Services::validation();
$validation->setRules(['judul' => 'required']);
$isDataValid = $validation->withRequest($this->request)->run();
if ($isDataValid)
{
$artikel = new ArtikelModel();
$artikel->insert([
'judul' => $this->request->getPost('judul'),
'isi' => $this->request->getPost('isi'),
'slug' => url_title($this->request->getPost('judul')),
]);
return redirect('admin/artikel');
}
$title = "Tambah Artikel";
return view('artikel/form_add', compact('title'));
}
```

<p>Kemudian buat view untuk form tambah dengan nama form_add.php</p>

```
<?= $this->include('template/admin_header'); ?>
<h2><?= $title; ?></h2>
<form action="" method="post">
  <p>
  <input type="text" name="judul">
  </p>
  <p>
  <textarea name="isi" cols="50" rows="10"></textarea>
  </p>
  <p><input type="submit" value="Kirim" class="btn btn-large"></p>
</form>
<?= $this->include('template/admin_footer'); ?>
```

## Mengubah Data 

<p>Tambahkan fungsi/method baru pada Controller Artikel dengan nama edit().</p>

```
public function edit($id)
{
  $artikel = new ArtikelModel();
  // validasi data.
  $validation = \Config\Services::validation();
  $validation->setRules(['judul' => 'required']);
  $isDataValid = $validation->withRequest($this->request)->run();
  if ($isDataValid)
  {
  $artikel->update($id, [
  'judul' => $this->request->getPost('judul'),
  'isi' => $this->request->getPost('isi'),
  ]);
  return redirect('admin/artikel');
  }
  // ambil data lama
  $data = $artikel->where('id', $id)->first();
  $title = "Edit Artikel";
  return view('artikel/form_edit', compact('title', 'data'));
}
```

<p>Kemudian buat view untuk form tambah dengan nama form_edit.php</p>

```
<?= $this->include('template/admin_header'); ?>

<h2><?= $title; ?></h2>
<form action="" method="post">
  <p>
  <input type="text" name="judul" value="<?= $data['judul'];?>" >
  </p>
  <p>
<textarea name="isi" cols="50" rows="10"><?=
$data['isi'];?></textarea>
  </p>
  <p><input type="submit" value="Kirim" class="btn btn-large"></p>
</form>

<?= $this->include('template/admin_footer'); ?>
```

![img](https://github.com/fianal/lat2web/blob/main/img_lat2web/Edit_art.png)

## Menghapus Data
<p>Tambahkan fungsi/method baru pada Controller Artikel dengan nama delete(). </p>

```
public function delete($id)
{
$artikel = new ArtikelModel();
$artikel->delete($id);
return redirect('admin/artikel');
}
```


