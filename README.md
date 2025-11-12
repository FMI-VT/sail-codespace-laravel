# 🚀 Laravel 12 + Sail + Vite in GitHub Codespaces
*A clean, minimal guide for students (2025 edition)*

---

## 1️⃣ Create and open your Codespace
1. On GitHub → **New Repository** → name it `laravel-todos`.
2. Click **Code → Create codespace on main**.
3. Wait for the VS Code window in your browser to start.

---

## 2️⃣ Create the Laravel project (inside a subfolder)

```bash
composer create-project laravel/laravel laravel-todos
cd laravel-todos
```

---

## 3️⃣ Install Sail with MySQL and Redis

```bash
php artisan sail:install --with=mysql,redis
./vendor/bin/sail up -d
```

Sail will pull and start the Docker containers automatically.

---

## 4️⃣ Configure the environment

Open **`laravel-todos/.env`** and set:

```env
CODESPACE_NAME=mycodespace-name-xxxxxxxxxxxxxxx
APP_URL=https://${CODESPACE_NAME}-80.app.github.dev
VITE_APP_URL="${APP_URL}"
ASSET_URL=/
```

---

## 5️⃣ Trust the Codespaces/Docker proxy

Open **`bootstrap/app.php`** and modify the return block to include:

```php
return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware): void {
        // Trust the Codespaces / Docker proxy headers
        $middleware->trustProxies(at: '*');
    })
    ->create();
```

---

## 6️⃣ Add authentication with Breeze (Livewire)

```bash
./vendor/bin/sail composer require laravel/breeze --dev
./vendor/bin/sail artisan breeze:install livewire
./vendor/bin/sail npm install
./vendor/bin/sail npm run build
./vendor/bin/sail artisan migrate
```

This scaffolds `/login`, `/register`, and `/profile`.

---

## 🧹 Handy commands

| Action | Command |
| --- | --- |
| Start Sail | `./vendor/bin/sail up -d` |
| Stop Sail | `./vendor/bin/sail down` |
| Rebuild assets | `./vendor/bin/sail npm run build` |
| Run migrations | `./vendor/bin/sail artisan migrate` |
| Clear caches | `./vendor/bin/sail artisan optimize:clear` |
| Open Tinker | `./vendor/bin/sail artisan tinker` |

---

## 🧠 Why this works

| Problem | Cause | Fix |
| --- | --- | --- |
| Redirects to `localhost` | App didn’t trust proxy headers | `trustProxies('*')` in `bootstrap/app.php` |
| Assets had full URLs | Absolute mode by default | `ASSET_URL=/` + `VITE_APP_URL="${APP_URL}"` |
| HTTPS mismatch | Proxy hides real scheme | Trusted proxies forward correct scheme |

---

## 🎯 Result
A fully functional **Laravel 12 + Sail + Vite** setup in GitHub Codespaces with:
- ✅ Correct domain & HTTPS handling  
- ✅ Relative asset paths  
- ✅ MySQL + Redis via Sail  
- ✅ Zero local installation
