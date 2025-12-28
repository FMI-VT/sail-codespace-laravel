# 📸 Adding Image Uploads in Laravel MVC

This guide extends the Laravel Todos demo with an **image upload** feature to show how controllers handle file inputs, validation, and storage — all within the MVC architecture.

---

## 8️⃣ Create Model, Migration, and Controller

```bash
./vendor/bin/sail artisan make:model Photo -mcr
```

This creates:
- `app/Models/Photo.php`
- `database/migrations/...create_photos_table.php`
- `app/Http/Controllers/PhotoController.php`

---

## 8.1) Define the Database Schema

Edit the migration file in `database/migrations/...create_photos_table.php`:

```php
public function up(): void
{
    Schema::create('photos', function (Blueprint $table) {
        $table->id();
        $table->string('path');      // file path in storage
        $table->string('title')->nullable();
        $table->timestamps();
    });
}
```

Then run:
```bash
./vendor/bin/sail artisan migrate
```

---

## 8.2) Define the Model

`app/Models/Photo.php`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Photo extends Model
{
    use HasFactory;

    protected $fillable = ['path', 'title'];
}
```

---

## 8.3) Controller Logic

Replace `app/Http/Controllers/PhotoController.php` with:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Photo;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Storage;

class PhotoController extends Controller
{
    public function index()
    {
        $photos = Photo::latest()->get();
        return view('photos.index', compact('photos'));
    }

    public function store(Request $request)
    {
        $request->validate([
            'photo' => 'required|image|max:2048', // 2MB max
            'title' => 'nullable|string|max:255',
        ]);

        $path = $request->file('photo')->store('photos', 'public');

        Photo::create([
            'path' => $path,
            'title' => $request->title,
        ]);

        return back()->with('success', 'Photo uploaded successfully!');
    }

    public function destroy(Photo $photo)
    {
        Storage::disk('public')->delete($photo->path);
        $photo->delete();

        return back();
    }
}
```

---

## 8.4) Routes

`routes/web.php`:

```php
use App\Http\Controllers\PhotoController;

Route::middleware(['auth'])->group(function () {
    Route::get('/photos', [PhotoController::class, 'index'])->name('photos.index');
    Route::post('/photos', [PhotoController::class, 'store'])->name('photos.store');
    Route::delete('/photos/{photo}', [PhotoController::class, 'destroy'])->name('photos.destroy');
});
```

---

## 8.5) View Template

`resources/views/photos/index.blade.php`:

```blade
<x-app-layout>
    <x-slot name="header">
        <h2 class="font-semibold text-xl text-gray-800 leading-tight">
            📸 Photo Uploads
        </h2>
    </x-slot>

    <div class="py-6">
        <div class="max-w-2xl mx-auto sm:px-6 lg:px-8">
            <div class="bg-white shadow-sm sm:rounded-lg p-6">
                @if (session('success'))
                    <div class="bg-green-100 text-green-800 px-3 py-2 rounded mb-4">
                        {{ session('success') }}
                    </div>
                @endif

                <form method="POST" action="{{ route('photos.store') }}" enctype="multipart/form-data" class="mb-6">
                    @csrf
                    <input type="text" name="title" placeholder="Title (optional)" class="w-full border rounded p-2 mb-2">
                    <input type="file" name="photo" accept="image/*" class="w-full border rounded p-2 mb-2">
                    <button class="bg-blue-500 text-white px-4 py-2 rounded">Upload</button>
                </form>

                <div class="grid grid-cols-2 gap-4">
                    @foreach ($photos as $photo)
                        <div class="relative group">
                            <img src="{{ asset('storage/' . $photo->path) }}" class="rounded shadow" alt="Photo">

                            <form method="POST" action="{{ route('photos.destroy', $photo) }}" class="absolute top-2 right-2">
                                @csrf
                                @method('DELETE')
                                <button class="bg-red-500 text-white px-2 py-1 rounded opacity-75 hover:opacity-100">
                                    ✖
                                </button>
                            </form>

                            @if($photo->title)
                                <p class="text-center text-sm mt-1">{{ $photo->title }}</p>
                            @endif
                        </div>
                    @endforeach
                </div>
            </div>
        </div>
    </div>
</x-app-layout>
```

---

## 8.6) Add Link to Navigation

Open `resources/views/livewire/layout/navigation.blade.php` and add:

```blade
<x-nav-link :href="route('photos.index')" :active="request()->routeIs('photos.*')">
    📸 {{ __('Photos') }}
</x-nav-link>
```

Now “Photos” appears next to Dashboard and Todos.

---

## 8.7) Enable Public Storage

Laravel needs a symlink for serving uploaded files:

```bash
./vendor/bin/sail artisan storage:link
```

This links `storage/app/public` → `public/storage`.

---

## ✅ Test the Upload

1. Run the server:  
   ```bash
   ./vendor/bin/sail up -d
   ```

2. Visit `/photos`  
   Upload a few images.

3. Files appear in `storage/app/public/photos`  
   and accessible via `public/storage/photos/...`

4. Deletion removes both DB record and physical file.

---

## 🧩 MVC Mapping Summary

| Layer | File | Role |
|--------|------|------|
| **Model** | `app/Models/Photo.php` | Defines structure for stored files |
| **Controller** | `app/Http/Controllers/PhotoController.php` | Handles validation, saving, and deleting |
| **View** | `resources/views/photos/index.blade.php` | Upload form and gallery |
| **Route** | `routes/web.php` | Defines URLs `/photos`, `/photos/{photo}` |
| **Storage** | `storage/app/public/photos` | Actual file location |

---

🎯 You now have a working image upload module — fully MVC-compliant — ideal for demonstrating file handling in Laravel.
