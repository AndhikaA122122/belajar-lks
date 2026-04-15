# 🏨 Grand Andhika Hotel — Cloud Guest Book

Aplikasi buku tamu hotel berbasis PHP yang di-deploy di AWS menggunakan Docker, EC2, RDS (Primary + Read Replica), dan Application Load Balancer.

---

## 📁 Struktur Proyek

```
lomba-cloud-hotel/
├── app/
│   ├── Dockerfile
│   └── src/
│       ├── index.php
│       ├── login.php
│       ├── admin.php
│       ├── add_guest.php
│       ├── logout.php
│       ├── health.php
│       ├── config.php
│       └── db_init.sql
```

---

## 🗂️ Isi File Kode

### `app/Dockerfile`

```dockerfile
FROM php:8.2-apache

# Install dependencies
RUN apt-get update && apt-get install -y \
    libpng-dev \
    libjpeg-dev \
    libfreetype6-dev \
    zip \
    unzip \
    git \
    curl \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install gd mysqli pdo pdo_mysql \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Enable Apache mod_rewrite
RUN a2enmod rewrite

# Set working directory
WORKDIR /var/www/html

# Copy application files
COPY ./src/ /var/www/html/

# Set permissions
RUN chown -R www-data:www-data /var/www/html \
    && chmod -R 755 /var/www/html

EXPOSE 80

CMD ["apache2-foreground"]
```

---

### `app/src/config.php`

```php
<?php
// Konfigurasi Database
define('DB_HOST_READ', 'rds-replica-endpoint');   // GANTI dengan endpoint RDS Read Replica
define('DB_HOST_WRITE', 'rds-primary-endpoint');  // GANTI dengan endpoint RDS Primary
define('DB_USER', 'admin');
define('DB_PASS', 'HotelAdmin2026!');
define('DB_NAME', 'grand_andhika_hotel');
?>
```

---

### `app/src/index.php`

```php
<?php
session_start();
require_once 'config.php';

$hotel_name = "Grand Andhika Hotel";
$hotel_address = "Jl. Gatot Subroto No. 88, Medan, Sumatera Utara";
?>
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title><?= $hotel_name ?> - Buku Tamu Digital</title>
    <!-- Bootstrap 5 + Icons + Google Font -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.3/font/bootstrap-icons.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background: linear-gradient(135deg, #f5f7fa 0%, #e9ecf2 100%);
            min-height: 100vh;
        }
        .navbar-brand {
            font-weight: 700;
            letter-spacing: -0.5px;
        }
        .hero-card {
            border: none;
            border-radius: 24px;
            background: rgba(255,255,255,0.85);
            backdrop-filter: blur(10px);
            box-shadow: 0 20px 40px rgba(0,0,0,0.08), 0 6px 12px rgba(0,0,0,0.05);
            transition: transform 0.2s ease;
        }
        .hero-card:hover {
            transform: translateY(-4px);
            box-shadow: 0 30px 50px rgba(0,0,0,0.12);
        }
        .section-title {
            font-weight: 700;
            color: #1e293b;
            margin-bottom: 1.5rem;
            position: relative;
            display: inline-block;
        }
        .section-title:after {
            content: '';
            position: absolute;
            bottom: -8px;
            left: 0;
            width: 60px;
            height: 4px;
            background: linear-gradient(90deg, #3b82f6, #8b5cf6);
            border-radius: 4px;
        }
        .btn-gradient {
            background: linear-gradient(145deg, #2563eb, #4f46e5);
            border: none;
            color: white;
            font-weight: 600;
            padding: 10px 24px;
            border-radius: 40px;
            box-shadow: 0 8px 16px rgba(37,99,235,0.2);
            transition: all 0.2s;
        }
        .btn-gradient:hover {
            background: linear-gradient(145deg, #1d4ed8, #4338ca);
            box-shadow: 0 12px 20px rgba(37,99,235,0.3);
            color: white;
        }
        .table-custom {
            background: white;
            border-radius: 18px;
            overflow: hidden;
            box-shadow: 0 4px 12px rgba(0,0,0,0.03);
        }
        .table-custom thead {
            background: #1e293b;
            color: white;
        }
        .table-custom th {
            font-weight: 600;
            text-transform: uppercase;
            font-size: 0.8rem;
            letter-spacing: 0.5px;
            padding: 16px;
        }
        .table-custom td {
            padding: 14px 16px;
            vertical-align: middle;
        }
        .badge-hotel {
            background: linear-gradient(145deg, #f59e0b, #f97316);
            color: white;
            font-weight: 500;
            padding: 6px 14px;
            border-radius: 40px;
        }
        .footer-note {
            font-size: 0.85rem;
            color: #64748b;
        }
        .input-group-text {
            background: white;
            border-right: none;
        }
        .form-control, .form-select {
            border-left: none;
            padding: 12px 16px;
            border-radius: 16px;
            border: 1px solid #e2e8f0;
            background: white;
        }
        .form-control:focus {
            border-color: #3b82f6;
            box-shadow: 0 0 0 3px rgba(59,130,246,0.1);
        }
    </style>
</head>
<body>

<!-- Navbar Elegan -->
<nav class="navbar navbar-expand-lg sticky-top" style="background: rgba(255,255,255,0.8); backdrop-filter: blur(12px); box-shadow: 0 4px 12px rgba(0,0,0,0.02);">
    <div class="container py-2">
        <a class="navbar-brand" href="#">
            <i class="bi bi-building fs-3 me-2" style="color: #2563eb;"></i>
            <span style="color: #0f172a;">Grand Andhika</span>
            <span style="color: #2563eb; font-weight: 300;">Hotel</span>
        </a>
        <div class="navbar-nav ms-auto">
            <?php if (isset($_SESSION['user_id'])): ?>
                <a class="nav-link me-3" href="admin.php"><i class="bi bi-speedometer2 me-1"></i> Dashboard</a>
                <a class="nav-link" href="logout.php"><i class="bi bi-box-arrow-right me-1"></i> Keluar</a>
            <?php else: ?>
                <a class="nav-link btn btn-outline-primary rounded-pill px-4" href="login.php">
                    <i class="bi bi-shield-lock me-1"></i> Login Admin
                </a>
            <?php endif; ?>
        </div>
    </div>
</nav>

<!-- Main Content -->
<div class="container py-5">
    <!-- Header dengan Alamat -->
    <div class="row mb-5">
        <div class="col-lg-8 mx-auto text-center">
            <span class="badge-hotel mb-3"><i class="bi bi-star-fill me-1"></i> Bintang 5 · Hospitality Excellence</span>
            <h1 class="display-5 fw-bold" style="color: #0f172a;"><?= $hotel_name ?></h1>
            <p class="lead text-secondary"><i class="bi bi-geo-alt-fill me-1" style="color: #dc2626;"></i> <?= $hotel_address ?></p>
        </div>
    </div>

    <div class="row g-5">
        <!-- Kolom Kiri: Form Tambah Tamu -->
        <div class="col-lg-5">
            <div class="hero-card p-4 p-xl-5">
                <h4 class="section-title"><i class="bi bi-pencil-square me-2"></i>Tulis Kesan Anda</h4>
                <p class="text-muted mb-4">Isi data diri dan pengalaman menginap Anda.</p>
                
                <form method="POST" action="add_guest.php">
                    <div class="mb-4">
                        <label class="form-label fw-semibold">Nama Lengkap</label>
                        <div class="input-group">
                            <span class="input-group-text bg-white border-end-0 rounded-start-4"><i class="bi bi-person"></i></span>
                            <input type="text" name="nama_tamu" class="form-control border-start-0 rounded-end-4" placeholder="Contoh: Budi Santoso" required>
                        </div>
                    </div>
                    <div class="mb-4">
                        <label class="form-label fw-semibold">Nomor Telepon</label>
                        <div class="input-group">
                            <span class="input-group-text bg-white border-end-0 rounded-start-4"><i class="bi bi-telephone"></i></span>
                            <input type="text" name="no_telp" class="form-control border-start-0 rounded-end-4" placeholder="0812-3456-7890" required>
                        </div>
                    </div>
                    <div class="row">
                        <div class="col-md-6 mb-4">
                            <label class="form-label fw-semibold">Nomor Kamar</label>
                            <input type="text" name="no_kamar" class="form-control rounded-4" placeholder="101" required>
                        </div>
                        <div class="col-md-6 mb-4">
                            <label class="form-label fw-semibold">Lama Inap (malam)</label>
                            <input type="number" name="lama_inap" class="form-control rounded-4" min="1" value="1" required>
                        </div>
                    </div>
                    <div class="mb-4">
                        <label class="form-label fw-semibold">Komentar / Kesan</label>
                        <textarea name="komentar" class="form-control rounded-4" rows="4" placeholder="Ceritakan pengalaman menginap Anda..."></textarea>
                    </div>
                    <button type="submit" class="btn btn-gradient w-100 py-3">
                        <i class="bi bi-send-fill me-2"></i>Kirim Kesan
                    </button>
                </form>
            </div>
        </div>

        <!-- Kolom Kanan: Daftar Tamu Terkini -->
        <div class="col-lg-7">
            <div class="hero-card p-4 p-xl-5">
                <div class="d-flex align-items-center mb-4">
                    <h4 class="section-title mb-0"><i class="bi bi-people-fill me-2"></i>Tamu Terkini</h4>
                    <span class="ms-auto badge bg-light text-dark rounded-pill px-3 py-2">
                        <i class="bi bi-database me-1"></i> Read Replica
                    </span>
                </div>
                
                <div class="table-responsive">
                    <table class="table table-custom align-middle">
                        <thead>
                            <tr>
                                <th>Nama Tamu</th>
                                <th>Kamar</th>
                                <th>Inap</th>
                                <th>Check-in</th>
                            </tr>
                        </thead>
                        <tbody>
                            <?php
                            try {
                                $conn_read = new mysqli(DB_HOST_READ, DB_USER, DB_PASS, DB_NAME);
                                if ($conn_read->connect_error) {
                                    echo '<tr><td colspan="4" class="text-center text-danger py-4"><i class="bi bi-exclamation-triangle me-2"></i>Gagal terhubung ke database</td></tr>';
                                } else {
                                    $result = $conn_read->query("SELECT nama_tamu, no_kamar, lama_inap, created_at FROM guests ORDER BY created_at DESC LIMIT 8");
                                    if ($result && $result->num_rows > 0) {
                                        while($row = $result->fetch_assoc()) {
                                            echo '<tr>';
                                            echo '<td><i class="bi bi-person-circle me-2 text-primary"></i>' . htmlspecialchars($row['nama_tamu']) . '</td>';
                                            echo '<td><span class="badge bg-light text-dark rounded-pill px-3 py-2">' . htmlspecialchars($row['no_kamar']) . '</span></td>';
                                            echo '<td>' . htmlspecialchars($row['lama_inap']) . ' malam</td>';
                                            echo '<td><i class="bi bi-calendar3 me-1 text-secondary"></i>' . date('d M Y', strtotime($row['created_at'])) . '</td>';
                                            echo '</tr>';
                                        }
                                    } else {
                                        echo '<tr><td colspan="4" class="text-center py-4 text-muted"><i class="bi bi-journal-x me-2"></i>Belum ada tamu yang mengisi buku tamu</td></tr>';
                                    }
                                    $conn_read->close();
                                }
                            } catch (Exception $e) {
                                echo '<tr><td colspan="4" class="text-center text-danger py-4">Error: ' . htmlspecialchars($e->getMessage()) . '</td></tr>';
                            }
                            ?>
                        </tbody>
                    </table>
                </div>
                <div class="footer-note mt-3 d-flex align-items-center">
                    <i class="bi bi-info-circle-fill me-2" style="color: #3b82f6;"></i>
                    Data ditampilkan dari database replica (read-only) untuk performa optimal.
                </div>
            </div>
        </div>
    </div>
</div>

<!-- Footer -->
<footer class="container text-center py-4">
    <p class="footer-note mb-0">© 2026 Grand Andhika Hotel — Dibangun dengan <i class="bi bi-heart-fill text-danger"></i> untuk keramahan terbaik</p>
</footer>

<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

---

### `app/src/login.php`

```php
<?php
session_start();
if (isset($_SESSION['user_id'])) {
    header('Location: admin.php');
    exit;
}

$error = '';
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    require_once 'config.php';
    
    $username = $_POST['username'] ?? '';
    $password = $_POST['password'] ?? '';
    
    $conn = new mysqli(DB_HOST_WRITE, DB_USER, DB_PASS, DB_NAME);
    
    if (!$conn->connect_error) {
        $stmt = $conn->prepare("SELECT id, username, password FROM admin_users WHERE username = ?");
        $stmt->bind_param("s", $username);
        $stmt->execute();
        $result = $stmt->get_result();
        
        if ($row = $result->fetch_assoc()) {
            if (password_verify($password, $row['password'])) {
                $_SESSION['user_id'] = $row['id'];
                $_SESSION['username'] = $row['username'];
                header('Location: admin.php');
                exit;
            } else {
                $error = 'Password tidak cocok!';
            }
        } else {
            $error = 'Username tidak ditemukan!';
        }
        $conn->close();
    } else {
        $error = 'Gagal terhubung ke database.';
    }
}
?>
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Login Admin · Grand Andhika Hotel</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.3/font/bootstrap-icons.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background: radial-gradient(circle at 10% 30%, #e0e7ff, #f3f4f6);
            min-height: 100vh;
            display: flex;
            align-items: center;
            padding: 20px;
        }
        .login-card {
            border: none;
            border-radius: 32px;
            background: rgba(255,255,255,0.9);
            backdrop-filter: blur(12px);
            box-shadow: 0 30px 60px rgba(0,0,0,0.1);
            overflow: hidden;
            max-width: 420px;
            width: 100%;
            margin: auto;
        }
        .login-header {
            background: linear-gradient(145deg, #1e293b, #0f172a);
            padding: 2rem 2rem 1.5rem;
            text-align: center;
        }
        .login-body {
            padding: 2.5rem 2rem;
        }
        .form-control {
            border-radius: 16px;
            padding: 14px 18px;
            border: 1px solid #e2e8f0;
            background: white;
        }
        .form-control:focus {
            border-color: #2563eb;
            box-shadow: 0 0 0 3px rgba(37,99,235,0.15);
        }
        .btn-login {
            background: linear-gradient(145deg, #2563eb, #4f46e5);
            border: none;
            border-radius: 40px;
            padding: 14px;
            font-weight: 700;
            color: white;
            width: 100%;
            transition: all 0.2s;
            box-shadow: 0 8px 16px rgba(37,99,235,0.2);
        }
        .btn-login:hover {
            transform: translateY(-2px);
            box-shadow: 0 16px 24px rgba(37,99,235,0.3);
        }
        .error-toast {
            background: #fee2e2;
            color: #b91c1c;
            border-radius: 40px;
            padding: 12px 20px;
            font-size: 0.9rem;
        }
    </style>
</head>
<body>

<div class="login-card">
    <div class="login-header">
        <i class="bi bi-building fs-1 text-white mb-2"></i>
        <h2 class="text-white fw-bold">Grand Andhika</h2>
        <p class="text-white-50 mb-0">Panel Administrasi</p>
    </div>
    <div class="login-body">
        <?php if ($error): ?>
            <div class="error-toast d-flex align-items-center mb-4">
                <i class="bi bi-exclamation-triangle-fill me-2"></i> <?= htmlspecialchars($error) ?>
            </div>
        <?php endif; ?>
        <form method="POST">
            <div class="mb-4">
                <label class="form-label fw-semibold">Username</label>
                <div class="input-group">
                    <span class="input-group-text bg-transparent border-end-0"><i class="bi bi-person"></i></span>
                    <input type="text" name="username" class="form-control border-start-0" placeholder="admin" required autofocus>
                </div>
            </div>
            <div class="mb-4">
                <label class="form-label fw-semibold">Password</label>
                <div class="input-group">
                    <span class="input-group-text bg-transparent border-end-0"><i class="bi bi-lock"></i></span>
                    <input type="password" name="password" class="form-control border-start-0" placeholder="········" required>
                </div>
            </div>
            <button type="submit" class="btn btn-login">
                <i class="bi bi-box-arrow-in-right me-2"></i>Masuk Dashboard
            </button>
        </form>
        <p class="text-muted text-center small mt-4 mb-0">
            <i class="bi bi-shield-shaded"></i> Akses terbatas hanya untuk staf hotel
        </p>
    </div>
</div>

<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

---

### `app/src/admin.php`

```php
<?php
session_start();
if (!isset($_SESSION['user_id'])) {
    header('Location: login.php');
    exit;
}
require_once 'config.php';

$message = '';

// Proses Tambah / Edit
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $conn_write = new mysqli(DB_HOST_WRITE, DB_USER, DB_PASS, DB_NAME);
    if (!$conn_write->connect_error) {
        if (isset($_POST['add'])) {
            $stmt = $conn_write->prepare("INSERT INTO guests (nama_tamu, no_telp, no_kamar, lama_inap, komentar) VALUES (?, ?, ?, ?, ?)");
            $stmt->bind_param("sssis", $_POST['nama_tamu'], $_POST['no_telp'], $_POST['no_kamar'], $_POST['lama_inap'], $_POST['komentar']);
            $stmt->execute();
            $message = 'Data tamu berhasil ditambahkan.';
        } elseif (isset($_POST['edit'])) {
            $stmt = $conn_write->prepare("UPDATE guests SET nama_tamu=?, no_telp=?, no_kamar=?, lama_inap=?, komentar=? WHERE id=?");
            $stmt->bind_param("sssisi", $_POST['nama_tamu'], $_POST['no_telp'], $_POST['no_kamar'], $_POST['lama_inap'], $_POST['komentar'], $_POST['id']);
            $stmt->execute();
            $message = 'Data berhasil diperbarui.';
        }
        $conn_write->close();
    }
}

// Proses Hapus
if (isset($_GET['delete'])) {
    $conn_write = new mysqli(DB_HOST_WRITE, DB_USER, DB_PASS, DB_NAME);
    $stmt = $conn_write->prepare("DELETE FROM guests WHERE id = ?");
    $stmt->bind_param("i", $_GET['delete']);
    $stmt->execute();
    $conn_write->close();
    header('Location: admin.php?msg=deleted');
    exit;
}

// Ambil semua data untuk ditampilkan
$conn_write = new mysqli(DB_HOST_WRITE, DB_USER, DB_PASS, DB_NAME);
$result = $conn_write->query("SELECT * FROM guests ORDER BY created_at DESC");
?>
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Admin · Grand Andhika Hotel</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.3/font/bootstrap-icons.min.css">
    <style>
        body { font-family: 'Inter', sans-serif; background: #f8fafc; }
        .navbar-admin { background: white; box-shadow: 0 4px 12px rgba(0,0,0,0.02); padding: 1rem 0; }
        .card-modern { border: none; border-radius: 24px; background: white; box-shadow: 0 10px 30px rgba(0,0,0,0.04); }
        .table-admin thead { background: #1e293b; color: white; }
        .btn-gradient { background: linear-gradient(145deg, #2563eb, #4f46e5); border: none; color: white; border-radius: 40px; padding: 8px 20px; }
    </style>
</head>
<body>

<!-- Navbar -->
<nav class="navbar-admin">
    <div class="container d-flex align-items-center">
        <a class="navbar-brand d-flex align-items-center" href="index.php">
            <i class="bi bi-building fs-3 me-2" style="color: #2563eb;"></i>
            <span class="fw-bold">Grand Andhika</span> <span class="text-muted ms-2">Admin</span>
        </a>
        <div class="ms-auto d-flex align-items-center">
            <span class="me-3 text-secondary"><i class="bi bi-person-check"></i> <?= htmlspecialchars($_SESSION['username']) ?></span>
            <a href="logout.php" class="btn btn-outline-secondary rounded-pill"><i class="bi bi-box-arrow-right"></i></a>
        </div>
    </div>
</nav>

<div class="container py-4">
    <?php if ($message): ?>
        <div class="alert alert-success alert-dismissible fade show rounded-4 border-0 shadow-sm" role="alert">
            <i class="bi bi-check-circle-fill me-2"></i><?= $message ?>
            <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
        </div>
    <?php elseif (isset($_GET['msg'])): ?>
        <div class="alert alert-info alert-dismissible fade show rounded-4 border-0 shadow-sm" role="alert">
            <i class="bi bi-info-circle-fill me-2"></i>Data berhasil dihapus.
            <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
        </div>
    <?php endif; ?>

    <div class="card-modern p-4">
        <div class="d-flex flex-wrap align-items-center justify-content-between mb-4">
            <h3 class="fw-bold mb-0"><i class="bi bi-journal-bookmark-fill me-2 text-primary"></i>Manajemen Buku Tamu</h3>
            <button class="btn btn-gradient mt-2 mt-sm-0" data-bs-toggle="modal" data-bs-target="#addModal">
                <i class="bi bi-plus-lg me-1"></i> Tambah Tamu Baru
            </button>
        </div>

        <div class="table-responsive">
            <table class="table table-hover align-middle table-admin">
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>Nama Tamu</th>
                        <th>Kontak</th>
                        <th>Kamar</th>
                        <th>Inap</th>
                        <th>Check-in</th>
                        <th>Aksi</th>
                    </tr>
                </thead>
                <tbody>
                    <?php while($row = $result->fetch_assoc()): ?>
                    <tr>
                        <td>#<?= $row['id'] ?></td>
                        <td><i class="bi bi-person-badge me-2"></i><?= htmlspecialchars($row['nama_tamu']) ?></td>
                        <td><?= htmlspecialchars($row['no_telp']) ?></td>
                        <td><span class="badge bg-light text-dark"><?= htmlspecialchars($row['no_kamar']) ?></span></td>
                        <td><?= $row['lama_inap'] ?> mlm</td>
                        <td><?= date('d/m/Y', strtotime($row['created_at'])) ?></td>
                        <td>
                            <button class="btn btn-sm btn-outline-primary me-1" data-bs-toggle="modal" data-bs-target="#editModal<?= $row['id'] ?>">
                                <i class="bi bi-pencil"></i>
                            </button>
                            <a href="?delete=<?= $row['id'] ?>" class="btn btn-sm btn-outline-danger rounded-3" onclick="return confirm('Hapus data tamu ini?')">
                                <i class="bi bi-trash3"></i>
                            </a>
                        </td>
                    </tr>

                    <!-- Modal Edit -->
                    <div class="modal fade" id="editModal<?= $row['id'] ?>" tabindex="-1">
                        <div class="modal-dialog modal-dialog-centered">
                            <div class="modal-content p-3">
                                <form method="POST">
                                    <input type="hidden" name="id" value="<?= $row['id'] ?>">
                                    <div class="modal-header border-0">
                                        <h5 class="modal-title fw-bold">Edit Data Tamu</h5>
                                        <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
                                    </div>
                                    <div class="modal-body">
                                        <div class="mb-3">
                                            <label>Nama</label>
                                            <input type="text" name="nama_tamu" class="form-control" value="<?= htmlspecialchars($row['nama_tamu']) ?>" required>
                                        </div>
                                        <div class="mb-3">
                                            <label>No. Telepon</label>
                                            <input type="text" name="no_telp" class="form-control" value="<?= htmlspecialchars($row['no_telp']) ?>" required>
                                        </div>
                                        <div class="row">
                                            <div class="col-6 mb-3">
                                                <label>Kamar</label>
                                                <input type="text" name="no_kamar" class="form-control" value="<?= htmlspecialchars($row['no_kamar']) ?>">
                                            </div>
                                            <div class="col-6 mb-3">
                                                <label>Inap</label>
                                                <input type="number" name="lama_inap" class="form-control" value="<?= $row['lama_inap'] ?>">
                                            </div>
                                        </div>
                                        <div class="mb-3">
                                            <label>Komentar</label>
                                            <textarea name="komentar" class="form-control" rows="2"><?= htmlspecialchars($row['komentar']) ?></textarea>
                                        </div>
                                    </div>
                                    <div class="modal-footer border-0">
                                        <button type="submit" name="edit" class="btn btn-primary rounded-pill px-4">Simpan</button>
                                    </div>
                                </form>
                            </div>
                        </div>
                    </div>
                    <?php endwhile; ?>
                </tbody>
            </table>
        </div>
        <p class="text-muted small mt-3"><i class="bi bi-database-fill-check"></i> Data ditulis ke database PRIMARY.</p>
    </div>
</div>

<!-- Modal Tambah -->
<div class="modal fade" id="addModal" tabindex="-1">
    <div class="modal-dialog modal-dialog-centered">
        <div class="modal-content p-3">
            <form method="POST">
                <div class="modal-header border-0">
                    <h5 class="modal-title fw-bold">Tambah Tamu Baru</h5>
                    <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
                </div>
                <div class="modal-body">
                    <div class="mb-3">
                        <label>Nama Lengkap</label>
                        <input type="text" name="nama_tamu" class="form-control" required>
                    </div>
                    <div class="mb-3">
                        <label>No. Telepon</label>
                        <input type="text" name="no_telp" class="form-control" required>
                    </div>
                    <div class="row">
                        <div class="col-6 mb-3">
                            <label>Nomor Kamar</label>
                            <input type="text" name="no_kamar" class="form-control">
                        </div>
                        <div class="col-6 mb-3">
                            <label>Lama Inap</label>
                            <input type="number" name="lama_inap" class="form-control" value="1">
                        </div>
                    </div>
                    <div class="mb-3">
                        <label>Komentar</label>
                        <textarea name="komentar" class="form-control" rows="2"></textarea>
                    </div>
                </div>
                <div class="modal-footer border-0">
                    <button type="submit" name="add" class="btn btn-gradient rounded-pill px-5">Simpan</button>
                </div>
            </form>
        </div>
    </div>
</div>

<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

---

### `app/src/add_guest.php`

```php
<?php
require_once 'config.php';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $conn_write = new mysqli(DB_HOST_WRITE, DB_USER, DB_PASS, DB_NAME);
    
    if ($conn_write->connect_error) {
        die("Koneksi gagal: " . $conn_write->connect_error);
    }
    
    $nama_tamu = $_POST['nama_tamu'] ?? '';
    $no_telp = $_POST['no_telp'] ?? '';
    $no_kamar = $_POST['no_kamar'] ?? '';
    $lama_inap = $_POST['lama_inap'] ?? 1;
    $komentar = $_POST['komentar'] ?? '';
    
    $stmt = $conn_write->prepare("INSERT INTO guests (nama_tamu, no_telp, no_kamar, lama_inap, komentar) VALUES (?, ?, ?, ?, ?)");
    $stmt->bind_param("sssis", $nama_tamu, $no_telp, $no_kamar, $lama_inap, $komentar);
    
    if ($stmt->execute()) {
        header('Location: index.php?success=1');
    } else {
        echo "Error: " . $stmt->error;
    }
    
    $stmt->close();
    $conn_write->close();
} else {
    header('Location: index.php');
}
exit;
?>
```

---

### `app/src/logout.php`

```php
<?php
session_start();
session_destroy();
header('Location: index.php');
exit;
?>
```

---

### `app/src/health.php`

```php
<?php
header('Content-Type: application/json');
http_response_code(200);
echo json_encode(['status' => 'healthy', 'timestamp' => date('Y-m-d H:i:s')]);
?>
```

---

### `app/src/db_init.sql`

```sql
-- Run this on RDS Primary
CREATE DATABASE IF NOT EXISTS grand_andhika_hotel;
USE grand_andhika_hotel;

CREATE TABLE IF NOT EXISTS guests (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nama_tamu VARCHAR(100) NOT NULL,
    no_telp VARCHAR(20) NOT NULL,
    no_kamar VARCHAR(10) NOT NULL,
    lama_inap INT NOT NULL DEFAULT 1,
    komentar TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE IF NOT EXISTS admin_users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- Insert default admin (password: admin123)
INSERT INTO admin_users (username, password) VALUES 
('admin', '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi');

-- Insert sample data
INSERT INTO guests (nama_tamu, no_telp, no_kamar, lama_inap, komentar) VALUES
('Budi Santoso', '081234567890', '101', 3, 'Kamar bersih, pelayanan ramah'),
('Siti Aminah', '085678901234', '205', 2, 'Sarapan enak, lokasi strategis'),
('Ahmad Hidayat', '081122334455', '310', 4, 'Kolam renang bersih, staf helpful');
```

---

## 🚀 Panduan Deploy AWS

### Arsitektur

```
Internet
   │
   ▼
[ALB - Application Load Balancer]
   │
   ├── EC2 Web-1 (Docker)
   ├── EC2 Web-2 (Docker)
   ├── EC2 Web-3 (Docker)
   └── EC2 Web-4 (Docker)
              │
       ┌──────┴──────┐
       ▼             ▼
  RDS Primary   RDS Replica
   (Write)        (Read)
```

---

## Langkah 1: Setup VPC

> ⏱️ ~5 menit

1. **AWS Console** → cari dan masuk ke **VPC**
2. Klik **Create VPC** → pilih **VPC and more**
3. Isi konfigurasi:

| Field | Nilai |
|---|---|
| Name | `hotel-vpc` |
| IPv4 CIDR | `10.0.0.0/16` |
| Number of AZs | `2` |
| Number of public subnets | `2` |
| Number of private subnets | `2` |
| NAT gateways | `In 1 AZ` |
| VPC endpoints | `None` |

4. Klik **Create VPC**

---

## Langkah 2: Buat Security Groups

> ⏱️ ~3 menit

**SG untuk EC2 (Web Server):**

- **EC2 Console** → **Security Groups** → **Create security group**
- Name: `hotel-web-sg` | VPC: `hotel-vpc`
- Inbound rules:

| Type | Source |
|---|---|
| SSH | Your IP/32 |
| HTTP | `0.0.0.0/0` nanti di ganti pakai SG ALB |

**SG untuk RDS:**

- Name: `hotel-rds-sg` | VPC: `hotel-vpc`
- Inbound rules:

| Type | Source |
|---|---|
| MySQL/Aurora (3306) | `hotel-web-sg` |

Langkah 1: Buat Security Group untuk ALB
Di konsol EC2, pada panel kiri pilih Security Groups di bawah menu Network & Security.
Klik Create security group.

Isi detail:
Security group name: Contoh hotel-alb-sg
Description: Contoh Security group for ALB
VPC: Pilih VPC yang sama, yaitu hotel-vpc
Di bagian Inbound rules, klik Add rule dan isi:
Type: HTTP
Source: 0.0.0.0/0 (Ini untuk mengizinkan akses HTTP dari internet ke ALB)
Klik Create security group.

🔗 Langkah 2: Pasangkan Security Group ke ALB
Di konsol EC2, pada panel kiri pilih Load Balancers.
Pilih ALB kamu (hotel-alb), lalu buka tab Security.
Klik Edit.
Hapus centang pada security group lama (jika ada), lalu centang security group baru yang sudah dibuat (hotel-alb-sg).
Klik Save changes.

✏️ Langkah 3: Ubah Inbound Rule di Security Group EC2
Kembali ke halaman Security Groups.
Pilih security group untuk EC2 (hotel-web-sg).
Buka tab Inbound rules, lalu klik Edit inbound rules.
Cari aturan dengan tipe HTTP, lalu klik Delete untuk menghapus aturan dengan sumber 0.0.0.0/0.
Klik Add rule untuk membuat aturan baru:
Type: HTTP
Source: Ketik hotel-alb-sg dan pilih security group ALB yang muncul.
Klik Save rules.

---

## Langkah 3: Setup RDS MySQL (Primary + Read Replica)

> ⏱️ ~15 menit

**Buat DB Subnet Group terlebih dahulu:**

1. **RDS Console** → navigasi kiri → **Subnet groups** → **Create DB Subnet Group**
2. Isi detail:
   - Name: `hotel-db-subnet-group`
   - VPC: `hotel-vpc`
   - Add subnets: pilih 2 AZ berbeda (misal `us-east-1a` dan `us-east-1b`), pilih **private subnet** di masing-masing AZ
3. Klik **Create**

**A. RDS Primary:**

1. **RDS Console** → **Create database**
2. Standard create → MySQL → Engine: MySQL 8.0 → Production → Single AZ
3. Konfigurasi:

| Field | Nilai |
|---|---|
| DB identifier | `hotel-primary` |
| Master username | `admin` |
| Master password | `HotelAdmin2026!` |
| Instance class | `db.t3.micro` |
| VPC | `hotel-vpc` |
| Public access | **No** |
| Security group | `hotel-rds-sg` |
| Initial database name | `grand_andhika_hotel` |

4. Klik **Create database**

**B. RDS Read Replica:**

1. Pilih database **hotel-primary** → **Actions** → **Create read replica**
2. DB identifier: `hotel-replica` | Instance: `db.t3.micro`
3. Klik **Create read replica**

> ⏳ Tunggu status **Available** (bisa lanjut ke langkah berikutnya sambil menunggu)

---

## Langkah 4: Launch 4 EC2 Debian 12/13

> ⏱️ ~10 menit

1. **EC2 Console** → **Launch instance**
2. Konfigurasi:

| Field | Nilai |
|---|---|
| Name | `hotel-web-1` (ulangi untuk `-2`, `-3`, `-4`) |
| AMI | Debian 13 |
| Instance type | `t2.micro` |
| Key pair | Create new → `hotel-key` → simpan `.pem` di `C:\Users\Andhika\Downloads\lomba-cloud-hotel` |
| VPC | `hotel-vpc` |
| Subnet | public subnet (gunakan AZ berbeda tiap instance) |
| Auto-assign public IP | **Enable** |
| Security group | `hotel-web-sg` |
| Storage | `20 GB gp3` |

3. **Advanced** → **User data**, paste script berikut:

```bash
#!/bin/bash
apt-get update -y
apt-get install -y ca-certificates curl
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
https://download.docker.com/linux/debian \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
tee /etc/apt/sources.list.d/docker.list > /dev/null
apt-get update -y
apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
systemctl start docker
systemctl enable docker
usermod -aG docker admin
```

4. Klik **Launch instance**
5. Ulangi 3x untuk `hotel-web-2`, `hotel-web-3`, `hotel-web-4`

---

## Langkah 5: Setup Application Load Balancer (ALB)

> ⏱️ ~5 menit

1. **Target group** → Create new:

| Field | Nilai |
|---|---|
| Name | `hotel-tg` |
| Health check path | `/health.php` |
| Port | `80` |
| Healthy threshold | `2` |
| Unhealthy threshold | `2` |
| Timeout | `5` |
| Interval | `10` |

Langkah-langkah Aktifkan Sticky Session:
Buka AWS Console → EC2 → Target Groups.
Pilih target group hotel-tg.
Tab Attributes → klik Edit.
Pada bagian Stickiness:
Aktifkan Stickiness.
Stickiness type: pilih Load balancer generated cookie.
Duration: biarkan default (1 day) atau sesuaikan.
Klik Save changes.

2. **EC2 Console** → **Load Balancers** → **Create** → pilih **Application Load Balancer** 

3. Konfigurasi:

| Field | Nilai |
|---|---|
| Name | `hotel-alb` |
| Scheme | `Internet-facing` |
| VPC | `hotel-vpc` |
| Mappings | Pilih kedua public subnet |
| Security group | `hotel-web-sg` |
| Listener | `HTTP:80` |

4. Register targets: pilih keempat EC2 (setelah siap) → **Create**

---

## FASE 3: Deploy Kode dan Database

## Langkah 6: Init Database via RDS Primary

1. Dari Windows, SSH ke salah satu EC2 sebagai jump host:

```powershell
ssh -i C:\lomba-cloud-hotel\hotel-key.pem admin@<EC2-PUBLIC-IP>
```

2. Di dalam EC2, install MySQL client dan connect ke RDS Primary:

```bash
sudo apt-get update -y && sudo apt-get install -y default-mysql-client
mysql -h <RDS-PRIMARY-ENDPOINT> -u admin -p
# Masukkan password: HotelAdmin2026!
```

3. Copy-paste isi file `db_init.sql` ke MySQL prompt, lalu verifikasi:

```sql
USE grand_andhika_hotel;
SELECT * FROM guests;
SELECT * FROM admin_users;
EXIT;
```
---

## Langkah 7: Upload Kode ke EC2 via SCP

Dari **PowerShell Windows**, lakukan untuk semua 4 EC2:

```powershell
scp -i C:\lomba-cloud-hotel\hotel-key.pem -r C:\lomba-cloud-hotel\app\src admin@<EC2-IP>:~/
scp -i C:\lomba-cloud-hotel\hotel-key.pem C:\lomba-cloud-hotel\app\Dockerfile admin@<EC2-IP>:~/
```

---

## Langkah 8: Update Konfigurasi Database di Kode

SSH ke masing-masing EC2 dan edit file konfigurasi:

```bash
ssh -i C:\lomba-cloud-hotel\hotel-key.pem admin@<EC2-IP>
cd ~/src
nano config.php
nano index.php
```

Update kedua endpoint dengan nilai dari AWS Console RDS:

```php
define('DB_HOST_READ',  'hotel-replica.xxxxxx.us-east-1.rds.amazonaws.com');
define('DB_HOST_WRITE', 'hotel-primary.xxxxxx.us-east-1.rds.amazonaws.com');
```

Simpan: `Ctrl+O` → `Enter` → `Ctrl+X`

---

## Langkah 9: Build dan Run Docker Container

Di **setiap EC2**, jalankan:

```bash
cd ~
sudo docker build -t hotel-guestbook .
sudo docker run -d \
  --name guestbook-app \
  --restart unless-stopped \
  -p 80:80 \
  hotel-guestbook
```

Verifikasi:

```bash
sudo docker ps
curl http://localhost/health.php
# Diharapkan: {"status":"healthy","timestamp":"..."}
```

```bash
docker exec -it guestbook-app php -r "echo password_hash('admin123', PASSWORD_BCRYPT) . PHP_EOL;"
```

```bash
mysql -h hotel-primary.xxxxxx.ap-southeast-1.rds.amazonaws.com -u admin -p
```

```sql
USE grand_andhika_hotel;

DELETE FROM admin_users WHERE username = 'admin';

INSERT INTO admin_users (username, password) VALUES 
('admin', 'HASH_DARI_LANGKAH_1');

SELECT * FROM admin_users;
EXIT;
```

```sql
CREATE USER 'appuser'@'%' IDENTIFIED BY 'AppHotel2026!';
GRANT ALL PRIVILEGES ON grand_andhika_hotel.* TO 'appuser'@'%';
FLUSH PRIVILEGES;
```

---

## FASE 4: Testing dan Finalisasi

## Langkah 10: Daftarkan EC2 ke Target Group ALB

1. **EC2 Console** → **Target Groups** → `hotel-tg`
2. Tab **Targets** → **Register targets**
3. Pilih keempat EC2 → **Include as pending below** → **Register**
4. Tunggu status semua instance menjadi **Healthy** ✅

---

## Langkah 11: Testing Sistem

**✅ Test Load Balancer**
```
Buka browser → akses http://<ALB-DNS-NAME>
Refresh beberapa kali → harus tetap tampil
```

**✅ Test Write ke Primary**
```
Di halaman utama → isi form tambah tamu → Submit
Data harus tersimpan ke database
```

**✅ Test Read dari Replica**
```
Lihat tabel di halaman utama → data baru harus tampil
(Ada delay replication lag ~1-2 detik, normal)
```

**✅ Test Login Admin**
```
Akses /login.php
Username: admin | Password: admin123
Masuk ke admin panel → bisa CRUD data
```

**✅ Test High Availability**
```bash
# Stop container di salah satu EC2
sudo docker stop guestbook-app

# Akses ALB → masih jalan (traffic ke EC2 lain)

# Hidupkan kembali
sudo docker start guestbook-app
```

**testing apakah database tersimpan atau tidak di rds**
```sql
USE grand_andhika_hotel;
SELECT * FROM guests ORDER BY created_at DESC LIMIT 5;
```

---

## 🔧 Troubleshoot

### ❌ Tidak bisa masuk database

```bash
mysql -h hotel-primary.ctuiwuks6p5s.ap-southeast-1.rds.amazonaws.com \
  -u admin -p --ssl=0
```

### ❌ Password admin bermasalah

Connect ke MySQL, jalankan:
```bash
mysql -h hotel-primary.xxxxxx.ap-southeast-1.rds.amazonaws.com -u admin -p
```

```sql
USE grand_andhika_hotel;

-- Hapus user admin lama
DELETE FROM admin_users WHERE username = 'admin';

-- Generate hash baru dulu di terminal EC2:
-- php -r "echo password_hash('admin123', PASSWORD_DEFAULT);"

-- Masukkan user baru dengan hash valid
INSERT INTO admin_users (username, password) VALUES
('admin', '<HASH_DARI_LANGKAH_DI_ATAS>');

SELECT * FROM admin_users;
EXIT;
```

### ❌ Container tidak mau start

```bash
# Cek log
sudo docker logs guestbook-app

# Rebuild dari awal
sudo docker stop guestbook-app && sudo docker rm guestbook-app
sudo docker rmi hotel-guestbook
sudo docker build -t hotel-guestbook .
sudo docker run -d --name guestbook-app --restart unless-stopped -p 80:80 hotel-guestbook
```

### ❌ Health check ALB gagal

```bash
curl http://localhost/health.php
# Harus return: {"status":"healthy",...}
```

---

## 🔐 Login Admin Default

| Field | Nilai |
|---|---|
| Username | `admin` |
| Password | `admin123` |

> ⚠️ Ganti password setelah deploy pertama!
