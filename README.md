# **Penggunaan CodeIgniter dengan RestServer**
REST (Representational State Transfer), adalah suatu gaya arsitektur perangkat lunak untuk untuk pendistibusian sistem hipermedia seperti www. REST server menyediakan resources (sumber daya/data), REST client dapat mengakses dan menampilkan resource tersebut untuk penggunaan selanjutnya.
<br><br> Dalam penerapan REST pada CodeIgniter diperlukan beberapa library tambahan yang tidak disediakan CodeIgniter, salah satu library yang dapat digunakan adalah library dari Ardisaurus. Penerapan REST pada CodeIgniter dibawah ini akan dijelaskan langkah-langkah pembuatan REST API server tentang CRUD kontak nomor telepon
## Persiapan
* PHP 7.2 or greater
* CodeIgniter 3.1.11+
* Webserver seperti Xampp, Wampp, atau lainnya
* Codeigniter dan library REST server dapat diunduh di https://github.com/chriskacerguis/codeigniter-restserver untuk versi terbaru, versi yang saya buat menggunakan https://github.com/ardisaurus/ci-restserver.
##  
Extract CodeIgniter dan library REST server. Pindah ke htdocs pada direktori xampp lalu rename folder CodeIgniter dan library REST server menjadi rest_ci. Masukan pada browser 
<br>`http://127.0.0.1/rest_ci/index.php/rest_server`
## Konfigurasi database
Buat database baru dengan nama "kontak".
<br> <code> CREATE DATABASE kontak; </code>
<br>Buat tabel baru dengan nama "telepon" dengan field id (int 3), nama (varchar 50), nomor (varchar 13).
<pre> <code> CREATE TABLE telepon (id int(3) NOT NULL, 
<br>      nama varchar(50) NOT NULL, 
<br>      nomor varchar(13) NOT NULL, 
<br>      PRIMARY KEY (id); </code> </pre>
Masukan beberapa data
<pre> <code> INSERT INTO telepon (id, nama, nomor) VALUES 
<br>      (1, 'Mohammad', '08576666762'), 
<br>      (2, 'Ainun', '08576666770'),
<br>      (3, 'Ardiansyah', '08576666765'); </code> </pre>
Atur databse pada rest_ci/application/config/database.php 
<pre> <code>
defined('BASEPATH') OR exit('No direct script access allowed');
$active_group = 'default';
$query_builder = TRUE;

// arahkan ke database kontak
$db['default'] = array(
	'dsn' => '',
	'hostname' => 'localhost',
	'username' => 'root',
	'password' => '',
	'database' => 'kontak',
	'dbdriver' => 'mysqli',
	'dbprefix' => '',
	'pconnect' => FALSE,
	'db_debug' => (ENVIRONMENT !== 'production'),
	'cache_on' => FALSE,
	'cachedir' => '',
	'char_set' => 'utf8',
	'dbcollat' => 'utf8_general_ci',
	'swap_pre' => '',
	'encrypt' => FALSE,
	'compress' => FALSE,
	'stricton' => FALSE,
	'failover' => array(),
	'save_queries' => TRUE
); </code> </pre>
## GET
Metode GET menyediakan akses baca pada sumber daya yang disediakan oleh REST API. Untuk membaca data dari database dapat dilakukan dengan active record yang telah disediakan CodeIgniter. Fungsi GET memeriksa terlebih dahulu apakah terdapat property id pada address bar sehingga data yang ditampilkan dapat di seleksi berdasarkan id atau ditampilkan semua.
<br> Buat file php baru di di rest_ci/application/controller dengan nama kontak.php.
<pre> <code>
defined('BASEPATH') OR exit('No direct script access allowed');

require APPPATH . '/libraries/REST_Controller.php';
use Restserver\Libraries\REST_Controller;

class Kontak extends REST_Controller {

    // constructor
    function __construct($config = 'rest') {
        parent::__construct($config);
        // atur untuk load database
        $this->load->database();
    }

    //Menampilkan data kontak
    function index_get() {
        $id = $this->get('id');
        if ($id == '') {
            $kontak = $this->db->get('telepon')->result();
        } else {
            $this->db->where('id', $id);
            $kontak = $this->db->get('telepon')->result();
        }
        $this->response($kontak, 200);
    }
} </code> </pre>
Buka Postman, Pilih metode GET, masukan http://127.0.0.1/rest_ci/index.php/kontak lalu klik "Send".
## POST
Metode POST digunakan untuk mengirimkan data baru dari client ke server REST API. Sebagai contohnya digunakan untuk menambahkan kontak baru yang terdiri dari id, nama, dan nomor.
<pre> <code>
//Mengirim atau menambah data kontak baru
    function index_post() {
        $data = array(
                    'id'           => $this->post('id'),
                    'nama'          => $this->post('nama'),
                    'nomor'    => $this->post('nomor'));
        $insert = $this->db->insert('telepon', $data);
        if ($insert) {
            $this->response($data, 200);
        } else {
            $this->response(array('status' => 'fail', 502));
        }
    }
</code> </pre>
buka Postman, pilih metode POST, masukan http://127.0.0.1/rest_ci/index.php/kontak , klik "Body", pilih x-www-form-urlencoded, masukan key dan value (id, nama, nomor), lalu klik "Send". Lakukan metode GET untuk melihat data terbaru.

## PUT
Metode PUT digunakan untuk memperbarui data yang telah ada di server REST API.
<pre> <code>
// Mengubah data
    function index_put() {
        $id = $this->put('id');
        // jadikan array
        $data = array(
                    'id'       => $this->put('id'),
                    'nama'          => $this->put('nama'),
                    'nomor'    => $this->put('nomor'));
        // Ubah berdasar id 
        $this->db->where('id', $id);
        $update = $this->db->update('telepon', $data);
        if ($update) {
            $this->response($data, 200);
        } else {
            $this->response(array('status' => 'fail', 502));
        }
    }
</code> </pre>
buka Postman, pilih metode PUT, masukan http://127.0.0.1/rest_ci/index.php/kontak , klik "Body", pilih x-www-form-urlencoded, masukan key id dan value id yang akan diubah, lalu klik "Send".

## DELETE
Metode DELETE digunakan untuk menghapus data yang telah ada di server REST API.
<pre> <code>
// Menghapus data
    function index_delete() {
        $id = $this->delete('id');
        // hapus berdasar id
        $this->db->where('id', $id);
        $delete = $this->db->delete('telepon');
        if ($delete) {
            $this->response(array('status' => 'success'), 201);
        } else {
            $this->response(array('status' => 'fail', 502));
        }
    }
</code> </pre>
buka Postman, pilih metode DELETE, masukan http://127.0.0.1/rest_ci/index.php/kontak , klik "Body", pilih x-www-form-urlencoded, masukan key id dan value id yang akan dihapus, lalu klik "Send".
