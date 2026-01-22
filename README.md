# نصب وردپرس روی Amazon Linux 2023 (Kernel 6.1)

این راهنما مراحل نصب **آخرین نسخه پایدار WordPress** روی **Amazon Linux 2023** را با استفاده از **Apache، PHP و MariaDB** به‌صورت گام‌به‌گام و از طریق اتصال SSH توضیح می‌دهد.

---

## پیش‌نیازها

- یک EC2 Instance با Amazon Linux 2023
- دسترسی SSH به سرور
- باز بودن پورت `80` در Security Group
- دسترسی کاربر `ec2-user` با sudo

## 1. اتصال به سرور از طریق SSH

```bash
ssh ec2-user@YOUR_EC2_PUBLIC_IP
```
## 2. به‌روزرسانی سرور

```bash
sudo dnf update -y
```

## 3. نصب نرم‌افزارهای مورد نیاز در سرور

```bash
sudo dnf install -y httpd php php-cli php-common php-mysqlnd php-gd php-curl php-mbstring php-xml php-json php-zip mariadb105-server unzip
```

## 4. فعال‌سازی و اجرای آپاچی (Apache httpd)

```bash
sudo systemctl start httpd
sudo systemctl enable httpd
sudo systemctl is-enabled httpd
```

### 4.1 تست عملکرد آپاچی

```bash
systemctl status httpd
```

### 4.2 تست نسخه نصب شده PHP

```bash
php -v
```

## 5. فعال‌سازی و اجرای سرور بانک‌اطلاعاتی (MatiaDB)

```bash
sudo systemctl start mariadb
sudo systemctl enable mariadb
```

### 5.1 ایمن‌سازی MariaDB
```bash
sudo mysql_secure_installation
```

پاسخ‌های پیشنهادی برای ایمن‌سازی MariaDB:
- Set root password → Y
- Remove anonymous users → Y
- Disallow root login remotely → Y
- Remove test database → Y
- Reload privilege tables → Y

## 6. ایجاد بانک‌اطلاعاتی و کاربر برای WordPress
### 6.1 ورود به MariaDB
```bash
sudo mysql -u root -p
```
### 6.2 ایجاد یک بانک‌اطلاعاتی به نام WordPress
⚠️ واژه StrongPasswordHere را با یک رمز دلخواه جایگزین کنید ⚠️ 
```sql
CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'StrongPasswordHere';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

## 7. دانلود آخرین نسخه WordPress
```bash
cd /tmp
curl -O https://wordpress.org/latest.tar.gz
```

## 8. تنظیم WordPress به عنوان وب‌سایت اصلی
```bash
tar -xzf latest.tar.gz
sudo rm -rf /var/www/html/*
sudo cp -r wordpress/* /var/www/html/
```

## 9. تنظیم سطح دسترسی به فایل‌های وب‌سایت
```bash
sudo chown -R apache:apache /var/www/html
sudo chmod -R 755 /var/www/html
```

## 10. پیکربندی WordPress
### 10.1 ایجاد فایل wp-config.php
```bash
cd /var/www/html
cp wp-config-sample.php wp-config.php
```
### 10.2 ویرایش فایل wp-config.php
```bash
sudo nano wp-config.php
```
خطوط زیر را در فایل `wp-config.php` ویرایش کنید
```php
define('DB_NAME', 'wordpress');
define('DB_USER', 'wpuser');
define('DB_PASSWORD', 'رمز دلخواه که در مرحله 6.2 تعریف کردید');
define('DB_HOST', 'localhost');
```

## 11. تنظیمات اختیاری
```bash
curl -s https://api.wordpress.org/secret-key/1.1/salt/
```

## 12. ریستارت Apache
```bash
sudo systemctl restart httpd
```

## 13. ادامه نصب WordPress از طریق مرورگر وب
به آدرس ‍`http://YOUR_EC2_PUBLIC_IP/` مراجعه کنید و مراحل نصب WordPress را ادامه دهید.