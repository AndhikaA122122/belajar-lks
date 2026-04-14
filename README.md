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
// Ganti dengan endpoint RDS Anda dari AWS Console

define('DB_HOST_READ',  'rds-replica-endpoint');   // GANTI dengan endpoint RDS Read Replica
define('DB_HOST_WRITE', 'rds-primary-endpoint');   // GANTI dengan endpoint RDS Primary
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

// Database configuration - READ REPLICA (public view)
define('DB_HOST_READ',  'rds-replica-endpoint');  // GANTI dengan endpoint RDS Read Replica
define('DB_HOST_WRITE', 'rds-primary-endpoint'); // GANTI dengan endpoint RDS Primary
define('DB_USER', 'admin');
define('DB_PASS', 'HotelAdmin2026!');
define('DB_NAME', 'grand_andhika_hotel');

$hotel_name    = "Grand Andhika Hotel";
$hotel_address = "Jl. Gatot Subroto No. 88, Medan, Sumatera Utara";
?>
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Grand Andhika Hotel - Guest Book</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.0/font/bootstrap-icons.css">
</head>
<body>
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
        <div class="container">
            <a class="navbar-brand" href="#"><i class="bi bi-building"></i> <?= $hotel_name ?></a>
            <div class="navbar-nav ms-auto">
                <?php if (isset($_SESSION['user_id'])): ?>
                    <a class="nav-link" href="admin.php"><i class="bi bi-speedometer2"></i> Admin Panel</a>
                    <a class="nav-link" href="logout.php"><i class="bi bi-box-arrow-right"></i> Logout</a>
                <?php else: ?>
                    <a class="nav-link" href="login.php"><i class="bi bi-box-arrow-in-right"></i> Login</a>
                <?php endif; ?>
            </div>
        </div>
    </nav>

    <div class="container mt-4">
        <div class="row">
            <div class="col-md-8 mx-auto">
                <div class="card shadow">
                    <div class="card-header bg-primary text-white">
                        <h3 class="mb-0"><i class="bi bi-book"></i> Buku Tamu - <?= $hotel_name ?></h3>
                    </div>
                    <div class="card-body">
                        <p class="text-muted"><i class="bi bi-geo-alt"></i> <?= $hotel_address ?></p>

                        <!-- Form Tambah Tamu (Publik) -->
                        <h5>Tambah Data Tamu</h5>
                        <form method="POST" action="add_guest.php" class="mb-4">
                            <div class="row">
                                <div class="col-md-6 mb-3">
                                    <label>Nama Tamu *</label>
                                    <input type="text" name="nama_tamu" class="form-control" required>
                                </div>
                                <div class="col-md-6 mb-3">
                                    <label>No. Telepon *</label>
                                    <input type="text" name="no_telp" class="form-control" required>
                                </div>
                            </div>
                            <div class="row">
                                <div class="col-md-6 mb-3">
                                    <label>Nomor Kamar *</label>
                                    <input type="text" name="no_kamar" class="form-control" required>
                                </div>
                                <div class="col-md-6 mb-3">
                                    <label>Lama Menginap (malam) *</label>
                                    <input type="number" name="lama_inap" class="form-control" min="1" required>
                                </div>
                            </div>
                            <div class="mb-3">
                                <label>Komentar / Kesan</label>
                                <textarea name="komentar" class="form-control" rows="3"></textarea>
                            </div>
                            <button type="submit" class="btn btn-success">
                                <i class="bi bi-check-circle"></i> Simpan Data Tamu
                            </button>
                        </form>

                        <!-- Daftar Tamu (Read from Replica) -->
                        <h5>Daftar Tamu Terkini</h5>
                        <div class="table-responsive">
                            <table class="table table-striped table-hover">
                                <thead>
                                    <tr>
                                        <th>Nama</th>
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
                                            echo "<tr><td colspan='4' class='text-danger'>Error koneksi database</td></tr>";
                                        } else {
                                            $result = $conn_read->query(
                                                "SELECT nama_tamu, no_kamar, lama_inap, created_at FROM guests ORDER BY created_at DESC LIMIT 10"
                                            );
                                            if ($result && $result->num_rows > 0) {
                                                while ($row = $result->fetch_assoc()) {
                                                    echo "<tr>";
                                                    echo "<td>" . htmlspecialchars($row['nama_tamu'])  . "</td>";
                                                    echo "<td>" . htmlspecialchars($row['no_kamar'])   . "</td>";
                                                    echo "<td>" . htmlspecialchars($row['lama_inap'])  . " malam</td>";
                                                    echo "<td>" . htmlspecialchars($row['created_at']) . "</td>";
                                                    echo "</tr>";
                                                }
                                            } else {
                                                echo "<tr><td colspan='4'>Belum ada data tamu</td></tr>";
                                            }
                                        }
                                    } catch (Exception $e) {
                                        echo "<tr><td colspan='4' class='text-danger'>Error: Database belum dikonfigurasi</td></tr>";
                                    }
                                    ?>
                                </tbody>
                            </table>
                        </div>
                        <small class="text-muted">
                            <i class="bi bi-info-circle"></i> Data dibaca dari database replica (read-only)
                        </small>
                    </div>
                </div>
            </div>
        </div>
    </div>

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

    $stmt = $conn->prepare("SELECT id, username, password FROM admin_users WHERE username = ?");
    $stmt->bind_param("s", $username);
    $stmt->execute();
    $result = $stmt->get_result();

    if ($row = $result->fetch_assoc()) {
        if (password_verify($password, $row['password'])) {
            $_SESSION['user_id']  = $row['id'];
            $_SESSION['username'] = $row['username'];
            header('Location: admin.php');
            exit;
        } else {
            $error = 'Password salah!';
        }
    } else {
        $error = 'Username tidak ditemukan!';
    }
}
?>
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Login Admin - Grand Andhika Hotel</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body class="bg-light">
    <div class="container mt-5">
        <div class="row">
            <div class="col-md-4 mx-auto">
                <div class="card shadow">
                    <div class="card-header bg-primary text-white">
                        <h4 class="mb-0">Login Admin</h4>
                    </div>
                    <div class="card-body">
                        <?php if ($error): ?>
                            <div class="alert alert-danger"><?= $error ?></div>
                        <?php endif; ?>
                        <form method="POST">
                            <div class="mb-3">
                                <label>Username</label>
                                <input type="text" name="username" class="form-control" required>
                            </div>
                            <div class="mb-3">
                                <label>Password</label>
                                <input type="password" name="password" class="form-control" required>
                            </div>
                            <button type="submit" class="btn btn-primary w-100">Login</button>
                        </form>
                        <hr>
                        <small class="text-muted">Default: admin / admin123</small>
                    </div>
                </div>
            </div>
        </div>
    </div>
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

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $conn_write = new mysqli(DB_HOST_WRITE, DB_USER, DB_PASS, DB_NAME);

    if (isset($_POST['add'])) {
        $stmt = $conn_write->prepare(
            "INSERT INTO guests (nama_tamu, no_telp, no_kamar, lama_inap, komentar) VALUES (?, ?, ?, ?, ?)"
        );
        $stmt->bind_param("sssis", $_POST['nama_tamu'], $_POST['no_telp'], $_POST['no_kamar'], $_POST['lama_inap'], $_POST['komentar']);
        $stmt->execute();
        $message = 'Data tamu berhasil ditambahkan!';
    }

    if (isset($_POST['edit'])) {
        $stmt = $conn_write->prepare(
            "UPDATE guests SET nama_tamu=?, no_telp=?, no_kamar=?, lama_inap=?, komentar=? WHERE id=?"
        );
        $stmt->bind_param("sssisi", $_POST['nama_tamu'], $_POST['no_telp'], $_POST['no_kamar'], $_POST['lama_inap'], $_POST['komentar'], $_POST['id']);
        $stmt->execute();
        $message = 'Data tamu berhasil diupdate!';
    }

    $conn_write->close();
}

if (isset($_GET['delete'])) {
    $conn_write = new mysqli(DB_HOST_WRITE, DB_USER, DB_PASS, DB_NAME);
    $stmt = $conn_write->prepare("DELETE FROM guests WHERE id = ?");
    $stmt->bind_param("i", $_GET['delete']);
    $stmt->execute();
    $conn_write->close();
    header('Location: admin.php?msg=deleted');
    exit;
}
?>
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Admin Panel - Grand Andhika Hotel</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.0/font/bootstrap-icons.css">
</head>
<body>
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
        <div class="container">
            <a class="navbar-brand" href="index.php"><i class="bi bi-building"></i> Grand Andhika Hotel</a>
            <div class="navbar-nav ms-auto">
                <span class="navbar-text me-3">Welcome, <?= htmlspecialchars($_SESSION['username']) ?></span>
                <a class="nav-link" href="logout.php"><i class="bi bi-box-arrow-right"></i> Logout</a>
            </div>
        </div>
    </nav>

    <div class="container mt-4">
        <?php if ($message): ?>
            <div class="alert alert-success alert-dismissible fade show">
                <?= $message ?>
                <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
            </div>
        <?php endif; ?>

        <?php if (isset($_GET['msg'])): ?>
            <div class="alert alert-info alert-dismissible fade show">
                Data berhasil dihapus!
                <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
            </div>
        <?php endif; ?>

        <div class="card shadow">
            <div class="card-header bg-primary text-white">
                <h4 class="mb-0"><i class="bi bi-database"></i> Manajemen Data Tamu (Write to Primary DB)</h4>
            </div>
            <div class="card-body">
                <button class="btn btn-success mb-3" data-bs-toggle="modal" data-bs-target="#addModal">
                    <i class="bi bi-plus-circle"></i> Tambah Tamu
                </button>

                <div class="table-responsive">
                    <table class="table table-striped table-hover">
                        <thead>
                            <tr>
                                <th>ID</th><th>Nama</th><th>No. Telp</th>
                                <th>Kamar</th><th>Inap</th><th>Check-in</th><th>Aksi</th>
                            </tr>
                        </thead>
                        <tbody>
                            <?php
                            $conn_write = new mysqli(DB_HOST_WRITE, DB_USER, DB_PASS, DB_NAME);
                            $result = $conn_write->query("SELECT * FROM guests ORDER BY created_at DESC");
                            while ($row = $result->fetch_assoc()):
                            ?>
                            <tr>
                                <td><?= $row['id'] ?></td>
                                <td><?= htmlspecialchars($row['nama_tamu']) ?></td>
                                <td><?= htmlspecialchars($row['no_telp']) ?></td>
                                <td><?= htmlspecialchars($row['no_kamar']) ?></td>
                                <td><?= $row['lama_inap'] ?> malam</td>
                                <td><?= $row['created_at'] ?></td>
                                <td>
                                    <button class="btn btn-sm btn-warning"
                                        data-bs-toggle="modal"
                                        data-bs-target="#editModal<?= $row['id'] ?>">
                                        <i class="bi bi-pencil"></i>
                                    </button>
                                    <a href="?delete=<?= $row['id'] ?>"
                                        class="btn btn-sm btn-danger"
                                        onclick="return confirm('Yakin hapus data ini?')">
                                        <i class="bi bi-trash"></i>
                                    </a>
                                </td>
                            </tr>
                            <?php endwhile; ?>
                        </tbody>
                    </table>
                </div>
                <small class="text-muted">
                    <i class="bi bi-info-circle"></i> Data ditulis ke database PRIMARY (read-write)
                </small>
            </div>
        </div>
    </div>

    <!-- Add Modal -->
    <div class="modal fade" id="addModal" tabindex="-1">
        <div class="modal-dialog">
            <div class="modal-content">
                <form method="POST">
                    <div class="modal-header">
                        <h5 class="modal-title">Tambah Tamu Baru</h5>
                        <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
                    </div>
                    <div class="modal-body">
                        <div class="mb-3"><label>Nama Tamu</label>
                            <input type="text" name="nama_tamu" class="form-control" required></div>
                        <div class="mb-3"><label>No. Telepon</label>
                            <input type="text" name="no_telp" class="form-control" required></div>
                        <div class="mb-3"><label>Nomor Kamar</label>
                            <input type="text" name="no_kamar" class="form-control" required></div>
                        <div class="mb-3"><label>Lama Menginap (malam)</label>
                            <input type="number" name="lama_inap" class="form-control" min="1" required></div>
                        <div class="mb-3"><label>Komentar</label>
                            <textarea name="komentar" class="form-control" rows="3"></textarea></div>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Batal</button>
                        <button type="submit" name="add" class="btn btn-primary">Simpan</button>
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
    $no_telp   = $_POST['no_telp']   ?? '';
    $no_kamar  = $_POST['no_kamar']  ?? '';
    $lama_inap = $_POST['lama_inap'] ?? 1;
    $komentar  = $_POST['komentar']  ?? '';

    $stmt = $conn_write->prepare(
        "INSERT INTO guests (nama_tamu, no_telp, no_kamar, lama_inap, komentar) VALUES (?, ?, ?, ?, ?)"
    );
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
-- ============================================
-- Grand Andhika Hotel - Database Initialization
-- Jalankan script ini di RDS Primary instance
-- ============================================

CREATE DATABASE IF NOT EXISTS grand_andhika_hotel;
USE grand_andhika_hotel;

CREATE TABLE IF NOT EXISTS guests (
    id         INT AUTO_INCREMENT PRIMARY KEY,
    nama_tamu  VARCHAR(100) NOT NULL,
    no_telp    VARCHAR(20)  NOT NULL,
    no_kamar   VARCHAR(10)  NOT NULL,
    lama_inap  INT          NOT NULL DEFAULT 1,
    komentar   TEXT,
    created_at TIMESTAMP    DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE IF NOT EXISTS admin_users (
    id         INT AUTO_INCREMENT PRIMARY KEY,
    username   VARCHAR(50)  UNIQUE NOT NULL,
    password   VARCHAR(255) NOT NULL,
    created_at TIMESTAMP    DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- Default admin (password: admin123)
INSERT INTO admin_users (username, password) VALUES
('admin', '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi');

-- Sample data
INSERT INTO guests (nama_tamu, no_telp, no_kamar, lama_inap, komentar) VALUES
('Budi Santoso',  '081234567890', '101', 3, 'Kamar bersih, pelayanan ramah'),
('Siti Aminah',   '085678901234', '205', 2, 'Sarapan enak, lokasi strategis'),
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
