# Laravel Deployment to cPanel — Step-by-Step Guide

> **MANDATORY**: Follow this guide exactly when deploying Laravel to cPanel/shared hosting.
> Never upload the entire Laravel project into `public_html/`.

---

## Prerequisites

- cPanel access with File Manager or FTP/SSH
- PHP version on cPanel matches your Laravel requirement (check `composer.json`)
- MySQL/MariaDB database created on cPanel
- SSH access (recommended) or terminal via cPanel

---

## Step 1: Local Build Preparation

Run these commands **locally** before uploading:

```bash
# 1. Install production dependencies only
composer install --optimize-autoloader --no-dev

# 2. Build frontend assets (if using Vite)
npm ci
npm run build

# 3. Cache config, routes, views (optional but recommended)
php artisan config:cache
php artisan route:cache
php artisan view:cache

# 4. Clear any development-only caches
php artisan clear-compiled
```

> **Note**: Skip `config:cache` if your config uses `env()` calls outside of `.env` files — it will break on cached config.

---

## Step 2: Prepare Directory Structure on cPanel

On the cPanel server, create the following structure:

```
/home/USERNAME/
├── public_html/          ← Web root (cPanel serves this)
│   ├── index.php         ← Will be modified
│   ├── .htaccess
│   ├── assets/           ← Built frontend assets
│   └── storage -> ../nama-project/storage/app/public  ← Symlink
│
└── nama-project/          ← Laravel app root (OUTSIDE public_html)
    ├── app/
    ├── bootstrap/
    ├── config/
    ├── database/
    ├── resources/
    ├── routes/
    ├── storage/
    ├── vendor/
    ├── .env
    ├── artisan
    └── composer.json
```

Create the `nama-project/` directory if it doesn't exist:

```bash
mkdir -p ~/nama-project
```

---

## Step 3: Upload Files

### Option A: Via ZIP (Recommended)

```bash
# Local: Create deployment archive (exclude unnecessary files)
zip -r laravel-deploy.zip . \
  -x ".git/*" \
  -x "node_modules/*" \
  -x ".sisyphus/*" \
  -x "tests/*" \
  -x ".env.example" \
  -x "phpunit.xml"

# Upload laravel-deploy.zip to cPanel via File Manager or SCP
# Extract on server:
cd ~/nama-project
unzip ~/laravel-deploy.zip
```

### Option B: Via SCP/FTP

```bash
# Upload everything EXCEPT public/ contents to nama-project/
scp -r app bootstrap config database resources routes storage vendor \
    composer.json composer.lock artisan .env \
    USERNAME@yourdomain.com:~/nama-project/

# Upload ONLY public/ contents to public_html/
scp -r public/* USERNAME@yourdomain.com:~/public_html/
```

---

## Step 4: Modify `public_html/index.php`

Edit `~/public_html/index.php` to point to the correct paths:

```php
<?php

use Illuminate\Contracts\Http\Kernel;
use Illuminate\Http\Request;

define('LARAVEL_START', microtime(true));

// Determine if the application is in maintenance mode...
if (file_exists($maintenance = __DIR__.'/../nama-project/storage/framework/maintenance.php')) {
    require $maintenance;
}

// Register the Composer autoloader...
require __DIR__.'/../nama-project/vendor/autoload.php';

// Bootstrap Laravel and handle the request...
$app = require_once __DIR__.'/../nama-project/bootstrap/app.php';

$kernel = $app->make(Kernel::class);

$response = $kernel->handle(
    $request = Request::capture()
)->send();

$kernel->terminate($request, $response);
```

> **Key changes**: All `__DIR__.'/../'` paths now include `nama-project/` subfolder.

---

## Step 5: Configure `.env`

Edit `~/nama-project/.env` with production values:

```env
APP_NAME="Your App Name"
APP_ENV=production
APP_KEY=base64:YOUR_APP_KEY_HERE
APP_DEBUG=false
APP_URL=https://yourdomain.com

LOG_CHANNEL=stack
LOG_LEVEL=error

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=your_cpanel_db_name
DB_USERNAME=your_cpanel_db_user
DB_PASSWORD=your_strong_password

CACHE_DRIVER=file
QUEUE_CONNECTION=sync
SESSION_DRIVER=file
SESSION_LIFETIME=120
```

Generate app key if not already set:

```bash
cd ~/nama-project
php artisan key:generate
```

---

## Step 6: Database Migration

```bash
cd ~/nama-project

# Run migrations
php artisan migrate --force

# Seed if needed (first deploy only)
php artisan db:seed --force
```

> **Warning**: Always backup the database before running migrations on production.

---

## Step 7: Create Storage Symlink

This is **critical** for file uploads and asset serving.

> **Why manual symlink?** `php artisan storage:link` creates a relative symlink pointing to `public/storage`, but since `public/` no longer exists (it's `public_html/`), you must create it manually.

### Option A: Via SSH (if available)

```bash
# Remove any existing storage symlink in public_html
rm -f ~/public_html/storage

# Create symlink: public_html/storage → nama-project/storage/app/public
ln -s ~/nama-project/storage/app/public ~/public_html/storage
```

Verify the symlink:

```bash
ls -la ~/public_html/storage
# Should show: storage -> /home/USERNAME/nama-project/storage/app/public
```

### Option B: Via `symlink.php` (no SSH access)

If your cPanel hosting **does not provide SSH access**, create a helper file:

**Create `public_html/symlink.php`:**

```php
<?php
// symlink.php — One-time use only. DELETE after running.

$target = __DIR__ . '/../nama-project/storage/app/public';
$link   = __DIR__ . '/storage';

// Resolve to absolute paths
$target = realpath($target);

if (!$target) {
    die('ERROR: Target path does not exist. Check your folder structure.');
}

if (file_exists($link) || is_link($link)) {
    die('ERROR: Symlink or file already exists at: ' . $link);
}

if (symlink($target, $link)) {
    echo 'SUCCESS: Symlink created!<br>';
    echo 'Target: ' . $target . '<br>';
    echo 'Link: ' . $link . '<br><br>';
    echo '<strong style="color:red;">⚠ DELETE this file (symlink.php) now for security!</strong>';
} else {
    die('ERROR: Failed to create symlink. Check permissions.');
}
```

**Steps:**
1. Upload `symlink.php` to `~/public_html/`
2. Open in browser: `https://yourdomain.com/symlink.php`
3. Verify success message
4. **DELETE `symlink.php` immediately** — leaving it on the server is a security risk

### Verify Symlink Works

Test by accessing: `https://yourdomain.com/storage/test.txt`

---

## Step 8: Set Permissions

```bash
# Storage and bootstrap/cache must be writable
chmod -R 775 ~/nama-project/storage
chmod -R 775 ~/nama-project/bootstrap/cache

# If using shared hosting without proper user group, use:
chmod -R 755 ~/nama-project/storage
chmod -R 755 ~/nama-project/bootstrap/cache
```

---

## Step 9: Configure `.htaccess` (if needed)

Ensure `~/public_html/.htaccess` exists with proper rewrite rules:

```apache
<IfModule mod_rewrite.c>
    <IfModule mod_negotiation.c>
        Options -MultiViews -Indexes
    </IfModule>

    RewriteEngine On

    # Handle Authorization Header
    RewriteCond %{HTTP:Authorization} .
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

    # Redirect Trailing Slashes...
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteRule ^(.*)/$ /$1 [L,R=301]

    # Handle Front Controller...
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^ index.php [L]
</IfModule>
```

---

## Step 10: Final Verification

```bash
# 1. Check Laravel logs for errors
tail -f ~/nama-project/storage/logs/laravel.log

# 2. Test artisan commands work
cd ~/nama-project
php artisan --version
php artisan route:list

# 3. Clear caches after deployment
php artisan config:clear
php artisan cache:clear
php artisan view:clear
```

Then visit:
- `https://yourdomain.com` — Main app
- `https://yourdomain.com/storage/test.txt` — Symlink working
- `https://yourdomain.com/api/health` — API endpoint (if applicable)

---

## Quick Deployment Checklist

- [ ] `composer install --optimize-autoloader --no-dev`
- [ ] `npm run build` (if applicable)
- [ ] Upload Laravel app to `~/nama-project/` (NOT `public_html/`)
- [ ] Upload `public/` contents to `~/public_html/`
- [ ] Update `public_html/index.php` paths to `../nama-project/`
- [ ] Configure `.env` with production values
- [ ] Run `php artisan migrate --force`
- [ ] Create storage symlink: SSH `ln -s` or upload `symlink.php` + run via browser + delete
- [ ] Set permissions on `storage/` and `bootstrap/cache/`
- [ ] Verify `.htaccess` exists in `public_html/`
- [ ] Clear caches and test

---

## Troubleshooting

### 500 Internal Server Error
```bash
# Check error logs
tail -100 ~/nama-project/storage/logs/laravel.log
# Check cPanel error log
tail -100 ~/public_html/error.log
```

### Storage symlink not working
```bash
# Verify symlink target exists
ls -la ~/nama-project/storage/app/public/

# Recreate symlink with absolute path
rm ~/public_html/storage
ln -s /home/USERNAME/nama-project/storage/app/public /home/USERNAME/public_html/storage
```

### Composer not found on cPanel
```bash
# Use full path or install locally
which composer
# Or download composer.phar
curl -sS https://getcomposer.org/installer | php
php composer.phar install --optimize-autoloader --no-dev
```

### Permission denied on storage
```bash
# Check current owner
ls -la ~/nama-project/storage/

# Fix ownership (if you have sudo)
chown -R USERNAME:USERNAME ~/nama-project/storage/
chmod -R 775 ~/nama-project/storage/
```
