
# 📸 Photo Admin CRUD — Full MVC Example

This extends the previous image upload functionality by adding an admin interface for:

✔ Viewing uploaded photos in a grid  
✔ Editing titles inline  
✔ Updating/replacing image  
✔ Deleting images  
✔ Full CRUD operations  
✔ Same `Photo` model  
✔ Clean separation: admin vs user UI  

---

## 1️⃣ Create admin controller

```bash
./vendor/bin/sail artisan make:controller PhotoAdminController
```

---

## 2️⃣ Admin routes

In `routes/web.php`:

```php
use App\Http\Controllers\PhotoAdminController;

Route::middleware(['auth'])->group(function () {
    
    Route::get('/admin/photos', [PhotoAdminController::class, 'index'])->name('admin.photos.index');
    Route::post('/admin/photos', [PhotoAdminController::class, 'store'])->name('admin.photos.store');
    Route::get('/admin/photos/{photo}/edit', [PhotoAdminController::class, 'edit'])->name('admin.photos.edit');
    Route::patch('/admin/photos/{photo}', [PhotoAdminController::class, 'update'])->name('admin.photos.update');
    Route::delete('/admin/photos/{photo}', [PhotoAdminController::class, 'destroy'])->name('admin.photos.destroy');
});
```

---

## 3️⃣ Controller implementation

`app/Http/Controllers/PhotoAdminController.php`

```php
<?php

namespace App\Http\Controllers;

use App\Models\Photo;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Storage;

class PhotoAdminController extends Controller
{
    public function index()
    {
        $photos = Photo::latest()->get();
        return view('admin.photos.index', compact('photos'));
    }

    public function store(Request $request)
    {
        $request->validate([
            'photo' => 'required|image|max:2048',
            'title' => 'nullable|string|max:255'
        ]);

        $path = $request->file('photo')->store('photos', 'public');

        Photo::create([
            'path' => $path,
            'title' => $request->title
        ]);

        return back()->with('success', 'Photo uploaded successfully');
    }

    public function edit(Photo $photo)
    {
        return view('admin.photos.edit', compact('photo'));
    }

    public function update(Request $request, Photo $photo)
    {
        $request->validate([
            'title' => 'nullable|string|max:255',
            'photo' => 'nullable|image|max:2048'
        ]);

        $photo->title = $request->title;

        if ($request->hasFile('photo')) {
            Storage::disk('public')->delete($photo->path);
            $photo->path = $request->file('photo')->store('photos', 'public');
        }

        $photo->save();

        return redirect()->route('admin.photos.index')->with('success', 'Photo updated');
    }

    public function destroy(Photo $photo)
    {
        Storage::disk('public')->delete($photo->path);
        $photo->delete();

        return back()->with('success', 'Photo deleted');
    }
}
```
---

## 4️⃣ Admin list view — Grid gallery

`resources/views/admin/photos/index.blade.php`:

```blade
<x-app-layout>
    <x-slot name="header">
        <h2 class="font-semibold text-xl text-gray-800">
            📸 Admin – Photo Manager
        </h2>
    </x-slot>

    <div class="max-w-5xl mx-auto p-6">
        @if(session('success'))
            <div class="bg-green-200 text-green-800 p-2 rounded mb-4">
                {{ session('success') }}
            </div>
        @endif

        <form method="POST" enctype="multipart/form-data" action="{{ route('admin.photos.store') }}" class="mb-6">
            @csrf
            <div class="flex gap-2">
                <input type="text" name="title" placeholder="Title (optional)" class="border p-2 flex-1">
                <input type="file" name="photo" class="border p-2">
                <button class="bg-blue-500 text-white px-4 py-2 rounded">Upload</button>
            </div>
        </form>

        <div class="grid grid-cols-2 md:grid-cols-4 gap-4">
            @foreach($photos as $photo)
                <div class="relative border rounded">
                    <img src="{{ asset('storage/' . $photo->path) }}" class="rounded w-full h-40 object-cover">

                    <div class="p-2">
                        <div class="font-semibold">
                            {{ $photo->title ?? 'No title' }}
                        </div>
                    </div>

                    <div class="flex justify-between p-2">
                        <a href="{{ route('admin.photos.edit', $photo) }}" class="text-blue-600">Edit</a>

                        <form method="POST" action="{{ route('admin.photos.destroy', $photo) }}">
                            @csrf
                            @method('DELETE')
                            <button class="text-red-600">Delete</button>
                        </form>
                    </div>
                </div>
            @endforeach
        </div>
    </div>
</x-app-layout>
```

---

## 5️⃣ Admin edit view

`resources/views/admin/photos/edit.blade.php`:

```blade
<x-app-layout>
    <x-slot name="header">
        <h2 class="font-semibold text-xl text-gray-800">
            ✏️ Edit Photo
        </h2>
    </x-slot>

    <div class="max-w-xl mx-auto p-6">

        <img src="{{ asset('storage/' . $photo->path) }}" class="w-full rounded mb-4">

        <form method="POST" enctype="multipart/form-data" action="{{ route('admin.photos.update', $photo) }}">
            @csrf
            @method('PATCH')

            <label class="block mb-2 font-semibold">Title</label>
            <input type="text" name="title" value="{{ $photo->title }}" class="border p-2 w-full mb-4">

            <label class="block mb-2 font-semibold">Replace Image (optional)</label>
            <input type="file" name="photo" class="border p-2 w-full mb-4">

            <button class="bg-green-600 text-white px-4 py-2 rounded">Save</button>
        </form>

    </div>
</x-app-layout>
```

---

## 6️⃣ Add Admin link to navigation

In `resources/views/livewire/layout/navigation.blade.php`:

```blade
<x-nav-link :href="route('admin.photos.index')" :active="request()->routeIs('admin.photos.*')">
    🛠 Admin Photos
</x-nav-link>
```

---

## 7️⃣ MVC mapping

| Layer | File |
|--------|------|
| Model | `app/Models/Photo.php` |
| Controller | `app/Http/Controllers/PhotoAdminController.php` |
| View list | `resources/views/admin/photos/index.blade.php` |
| View edit | `resources/views/admin/photos/edit.blade.php` |
| Routes | `routes/web.php` |
| Storage | `storage/app/public/photos/` |

---

## 8️⃣ Result

You now have:
✔ User-facing photo gallery at `/photos`  
✔ Admin CRUD interface at `/admin/photos`  
✔ Both use the same model  
✔ Clean MVC separation  
✔ Excellent for demonstration  
✔ Looks like a real admin panel  
