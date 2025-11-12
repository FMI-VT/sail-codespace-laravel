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

## 📝 7️⃣ Add a mini Todo module

### 7.1) Create model, migration and controller

```bash
./vendor/bin/sail artisan make:model Todo -mcr
```

This creates:
- `app/Models/Todo.php`
- `database/migrations/...create_todos_table.php`
- `app/Http/Controllers/TodoController.php`

---

### 7.2) Define the table structure

Edit `database/migrations/...create_todos_table.php`:

```php
public function up(): void
{
    Schema::create('todos', function (Blueprint $table) {
        $table->id();
        $table->string('title');
        $table->boolean('completed')->default(false);
        $table->timestamps();
    });
}
```

Run migration:

```bash
./vendor/bin/sail artisan migrate
```

---

### 7.3) Controller logic

Replace `app/Http/Controllers/TodoController.php` with:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Todo;
use Illuminate\Http\Request;

class TodoController extends Controller
{
    public function index()
    {
        $todos = Todo::latest()->get();
        return view('todos.index', compact('todos'));
    }

    public function store(Request $request)
    {
        $request->validate(['title' => 'required|string|max:255']);
        Todo::create(['title' => $request->title]);
        return back();
    }

    public function update(Todo $todo)
    {
        $todo->update(['completed' => !$todo->completed]);
        return back();
    }

    public function destroy(Todo $todo)
    {
        $todo->delete();
        return back();
    }
}
```

---

### 7.4) Model fillable fields

In `app/Models/Todo.php`:

```php
protected $fillable = ['title', 'completed'];
```

---

### 7.5) Routes

Add to `routes/web.php`:

```php
use App\Http\Controllers\TodoController;

Route::get('/todos', [TodoController::class, 'index'])->name('todos.index');
Route::post('/todos', [TodoController::class, 'store'])->name('todos.store');
Route::patch('/todos/{todo}', [TodoController::class, 'update'])->name('todos.update');
Route::delete('/todos/{todo}', [TodoController::class, 'destroy'])->name('todos.destroy');
```

---

### 7.6) Blade view

Create `resources/views/todos/index.blade.php`:

```blade
@extends('layouts.app')

@section('content')
<div class="max-w-md mx-auto mt-10 p-6 bg-white rounded-xl shadow-md">
    <h1 class="text-2xl font-bold mb-4">📝 My Todo List</h1>

    <form method="POST" action="{{ route('todos.store') }}" class="mb-4 flex">
        @csrf
        <input name="title" placeholder="New task..."
               class="flex-grow border rounded-l px-3 py-2 focus:outline-none" />
        <button class="bg-blue-500 text-white px-4 rounded-r">Add</button>
    </form>

    <ul>
        @foreach($todos as $todo)
            <li class="flex justify-between items-center mb-2 {{ $todo->completed ? 'line-through text-gray-500' : '' }}">
                <form method="POST" action="{{ route('todos.update', $todo) }}">
                    @csrf
                    @method('PATCH')
                    <button class="text-left" title="Toggle complete">{{ $todo->title }}</button>
                </form>

                <form method="POST" action="{{ route('todos.destroy', $todo) }}">
                    @csrf
                    @method('DELETE')
                    <button class="text-red-500">✖</button>
                </form>
            </li>
        @endforeach
    </ul>
</div>
@endsection
```

---

### 7.7) Test

Open `/todos` (Codespaces → Port 80 → **Open in Browser**).

✅ You can now:
- Add tasks  
- Mark them as done (click title)  
- Delete tasks  

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
| Routes List | `./vendor/bin/sail artisan route:list` |

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
