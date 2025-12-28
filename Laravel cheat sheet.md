# 🧠 Understanding Laravel’s MVC Architecture
*A beginner-friendly guide to Laravel project structure and how it follows the MVC pattern*

---

## 🎯 What is MVC?

**MVC** stands for **Model – View – Controller**.

It’s a design pattern that separates your application logic into **three main layers**:

| Layer | Purpose | Example in our Todo app |
|--------|----------|--------------------------|
| **Model** | Manages data and rules (database tables, business logic). | `app/Models/Todo.php` |
| **View** | What the user sees — HTML, CSS, JS templates. | `resources/views/todos/index.blade.php` |
| **Controller** | Coordinates between Model and View — receives requests, calls Models, returns Views. | `app/Http/Controllers/TodoController.php` |

---

## 🏗️ Laravel Project Structure Overview

When you create a new Laravel project, it looks like this:

```
laravel-todos/
├── app/
│   ├── Console/           # Artisan commands
│   ├── Exceptions/        # Custom error handling
│   ├── Http/
│   │   ├── Controllers/   # Controllers (C in MVC)
│   │   ├── Middleware/    # Request filters (auth, logging, etc.)
│   │   └── Kernel.php     # Defines global and route middleware
│   ├── Models/            # Models (M in MVC)
│   └── Providers/         # Service providers (bootstrapping services)
│
├── bootstrap/             # Initializes the framework
│   └── app.php            # App entry point before routes
│
├── config/                # Configuration files (database, mail, etc.)
│
├── database/
│   ├── factories/         # Model factories (for test data)
│   ├── migrations/        # Database schema definitions
│   └── seeders/           # Database initial data
│
├── public/                # Web root (index.php, assets)
│
├── resources/
│   ├── css/, js/          # Frontend assets for Vite
│   ├── views/             # Blade templates (V in MVC)
│   └── lang/              # Localization strings
│
├── routes/
│   ├── web.php            # Web routes (browser)
│   ├── api.php            # API routes (JSON)
│   ├── console.php        # CLI routes
│   └── channels.php       # Event broadcasting
│
├── storage/               # Logs, cache, uploaded files
│
├── tests/                 # PHPUnit or Pest tests
│
├── vendor/                # Composer dependencies
│
└── .env                   # Environment configuration (database, URLs, etc.)
```

---

## 🧩 How MVC works in Laravel

### 1️⃣ User makes a request
Example: User visits `/todos` in the browser.

👉 The request goes through `public/index.php` (the entry point).  
Then Laravel loads environment, middleware, and the route definitions in `routes/web.php`.

---

### 2️⃣ Router finds the Controller
In `routes/web.php`:

```php
Route::get('/todos', [TodoController::class, 'index'])->name('todos.index');
```

Laravel sees that `/todos` should go to the `index()` method in `TodoController`.

---

### 3️⃣ Controller interacts with the Model
In `app/Http/Controllers/TodoController.php`:

```php
public function index()
{
    $todos = Todo::latest()->get();
    return view('todos.index', compact('todos'));
}
```

Here:
- The **controller** uses the **Model** (`Todo`) to fetch all todos.
- Then passes data (`$todos`) to the **View** (`todos/index.blade.php`).

---

### 4️⃣ Model handles the database
In `app/Models/Todo.php`:

```php
class Todo extends Model
{
    protected $fillable = ['title', 'completed'];
}
```

This represents the `todos` table in the database.  
Laravel automatically maps it using Eloquent ORM.

- `Todo::create([...])` inserts new rows  
- `Todo::all()` retrieves all rows  
- `Todo::find($id)` retrieves one  
- `Todo::update()` changes data  

---

### 5️⃣ View displays the data
In `resources/views/todos/index.blade.php`:

```blade
@foreach($todos as $todo)
    <li>{{ $todo->title }}</li>
@endforeach
```

The view receives the `$todos` array from the controller and renders it as HTML.

---

## 🔁 The MVC Flow Summary

```text
Request (Browser) 
     ↓
Route (web.php)
     ↓
Controller (TodoController)
     ↓
Model (Todo)
     ↓
Database
     ↑
View (Blade template)
     ↑
Response (HTML page to user)
```

---

## ⚙️ Supporting Components

Laravel includes additional helpers around MVC:

| Component | Role |
|------------|------|
| **Middleware** | Filters requests before reaching controllers (e.g., authentication, CORS). |
| **Service Providers** | Bootstraps and registers services during startup. |
| **Facades** | Static-like shortcuts to core services (e.g., `Auth::user()`, `DB::table()` etc.). |
| **Blade** | Laravel’s templating engine for dynamic HTML views. |
| **Eloquent ORM** | Maps database tables to Models. |
| **Artisan CLI** | Developer command-line interface (`php artisan make:model`, etc.). |

---

## 🧱 Example: How the Todo Feature fits in MVC

| Role | File | Responsibility |
|------|------|----------------|
| **Model** | `app/Models/Todo.php` | Defines database table, fillable fields |
| **Controller** | `app/Http/Controllers/TodoController.php` | Handles routes, validation, CRUD logic |
| **View** | `resources/views/todos/index.blade.php` | Displays todos list and forms |
| **Route** | `routes/web.php` | Connects URL `/todos` to controller action |
| **Migration** | `database/migrations/..._create_todos_table.php` | Defines database schema |
| **Layout** | `resources/views/components/layouts/app.blade.php` | Shared page layout for all views |

---

## 💡 Key takeaway

Laravel’s structure cleanly enforces **separation of concerns**:

- The **Model** knows the data.
- The **Controller** decides what to do.
- The **View** shows it to the user.

This separation makes your code easier to:
✅ maintain  
✅ test  
✅ extend  

---

## 🧰 Helpful Artisan commands for MVC

| Command | Description |
|----------|--------------|
| `php artisan make:model Todo -mcr` | Creates Model, Migration, Controller, and Resource routes |
| `php artisan make:controller ExampleController` | Creates a new controller |
| `php artisan make:migration create_table_name` | Creates new migration |
| `php artisan make:view example` | Creates a new Blade view file |
| `php artisan route:list` | Lists all registered routes |
| `php artisan tinker` | Interactive REPL for testing Models and logic |

---

## 🧩 Summary Diagram

```
     ┌────────────┐
     │   Browser   │
     └──────┬──────┘
            │ Request (/todos)
            ▼
      routes/web.php
            │
            ▼
   app/Http/Controllers/TodoController
            │
            ▼
        app/Models/Todo
            │
            ▼
     Database (MySQL)
            │
            ▼
  resources/views/todos/index.blade.php
            │
            ▼
         HTML Response
```

---

✨ That’s how Laravel organizes your code in the MVC architecture.
Once you understand this flow, every part of your Laravel app will “click” into place.
