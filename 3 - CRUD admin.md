# 🐾 Laravel Admin CRUD — Animals (with one photo per animal)

This example demonstrates a complete, working admin CRUD panel for animals.  
Each animal has a single main photo, chosen from uploaded photos.

It teaches:

✔ Laravel MVC  
✔ Migrations  
✔ Models & relations  
✔ Validation  
✔ Searching  
✔ Sorting  
✔ Pagination  
✔ Blade views  
✔ Routes  
✔ Controller logic  
✔ Admin layout integration  

---

# Requirements

You already have:

✔ Laravel installed  
✔ Sail / Docker working  
✔ Users authentication (Laravel Breeze etc)  
✔ Photo model + photos in database  
✔ Storage linked (`./vendor/bin/sail artisan storage:link`)  

---

# 1️⃣ Create Model + Migration + Controller

```bash
./vendor/bin/sail artisan make:model Animal -mcr
```

Creates:

```
app/Models/Animal.php
app/Http/Controllers/AnimalController.php
database/migrations/xxxx_create_animals_table.php
```

---

# 2️⃣ Migration

`database/migrations/...create_animals_table.php`:

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('animals', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->enum('species', ['dog', 'cat', 'bird', 'horse', 'fish', 'other']);
            $table->integer('age')->nullable();
            $table->text('description')->nullable();
            $table->foreignId('photo_id')->nullable()->constrained()->nullOnDelete();
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('animals');
    }
};
```

Run:

```bash
./vendor/bin/sail artisan migrate
```

---

# 3️⃣ Animal Model

`app/Models/Animal.php`

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Animal extends Model
{
    use HasFactory;

    protected $fillable = [
        'name',
        'species',
        'age',
        'description',
        'photo_id',
    ];

    public function photo()
    {
        return $this->belongsTo(Photo::class);
    }
}
```

---

# 4️⃣ Routes

`routes/web.php`

```php
use App\Http\Controllers\AnimalController;

Route::middleware(['auth'])->group(function () {
    Route::resource('/admin/animals', AnimalController::class)
        ->names('admin.animals');
});
```

---

# 5️⃣ Controller (with search + sort + pagination)

`app/Http/Controllers/AnimalController.php`

```php
<?php

namespace App\Http\Controllers;

use App\Models\Animal;
use App\Models\Photo;
use Illuminate\Http\Request;

class AnimalController extends Controller
{
    public function index(Request $request)
    {
        $query = Animal::query();

        if ($request->filled('search')) {
            $query->where(function($q) use ($request) {
                $q->where('name', 'like', '%' . $request->search . '%')
                  ->orWhere('species', 'like', '%' . $request->search . '%');
            });
        }

        $allowedSorts = ['name', 'species', 'age'];
        if ($request->filled('sort') && in_array($request->sort, $allowedSorts)) {
            $direction = $request->direction === 'desc' ? 'desc' : 'asc';
            $query->orderBy($request->sort, $direction);
        }

        $animals = $query->paginate(10)->withQueryString();
        return view('admin.animals.index', compact('animals'));
    }

    public function create()
    {
        $photos = Photo::all();
        return view('admin.animals.create', compact('photos'));
    }

    public function store(Request $request)
    {
        $request->validate([
            'name' => 'required',
            'species' => 'required',
            'age' => 'nullable|integer',
            'description' => 'nullable',
            'photo_id' => 'nullable|exists:photos,id'
        ]);

        Animal::create($request->all());

        return redirect()->route('admin.animals.index')
            ->with('success', 'Animal created');
    }

    public function edit(Animal $animal)
    {
        $photos = Photo::all();
        return view('admin.animals.edit', compact('animal', 'photos'));
    }

    public function update(Request $request, Animal $animal)
    {
        $request->validate([
            'name' => 'required',
            'species' => 'required',
            'age' => 'nullable|integer',
            'description' => 'nullable',
            'photo_id' => 'nullable|exists:photos,id'
        ]);

        $animal->update($request->all());

        return redirect()->route('admin.animals.index')
            ->with('success', 'Animal updated');
    }

    public function destroy(Animal $animal)
    {
        $animal->delete();

        return redirect()->route('admin.animals.index')
            ->with('success', 'Animal deleted');
    }
}
```

---

# 6️⃣ Index View — Listing + Searching + Sorting + Pagination

`resources/views/admin/animals/index.blade.php`

```blade
<x-app-layout>
    <x-slot name="header">
        🐾 Animals
    </x-slot>

    <div class="max-w-6xl mx-auto p-6">

        <a href="{{ route('admin.animals.create') }}"
           class="bg-blue-500 text-white px-4 py-2 rounded mb-4 inline-block">
           ➕ Add Animal
        </a>

        <form method="GET" class="mb-4">
            <input type="text" name="search" placeholder="Search..."
                   value="{{ request('search') }}"
                   class="border p-2 rounded">
            <button type="submit" class="bg-blue-500 text-white px-4 py-2 rounded">
                Search
            </button>
        </form>

        @php
            $direction = request('direction') === 'asc' ? 'desc' : 'asc';
        @endphp

        <table class="w-full border mb-4">
            <tr class="bg-gray-200">
                <th class="p-2 border">Photo</th>
                <th class="p-2 border">
                    <a href="?sort=name&direction={{ $direction }}">
                        Name
                        @if(request('sort')==='name')
                            {{ request('direction')==='asc'?'▲':'▼' }}
                        @endif
                    </a>
                </th>
                <th class="p-2 border">
                    <a href="?sort=species&direction={{ $direction }}">
                        Species
                        @if(request('sort')==='species')
                            {{ request('direction')==='asc'?'▲':'▼' }}
                        @endif
                    </a>
                </th>
                <th class="p-2 border">
                    <a href="?sort=age&direction={{ $direction }}">
                        Age
                        @if(request('sort')==='age')
                            {{ request('direction')==='asc'?'▲':'▼' }}
                        @endif
                    </a>
                </th>
                <th class="p-2 border">Actions</th>
            </tr>

            @forelse($animals as $animal)
                <tr>
                    <td class="p-2 border">
                        @if($animal->photo)
                            <img src="{{ asset('storage/' . $animal->photo->path) }}" class="w-12 h-12 rounded object-cover">
                        @else
                            —
                        @endif
                    </td>
                    <td class="p-2 border">{{ $animal->name }}</td>
                    <td class="p-2 border">{{ $animal->species }}</td>
                    <td class="p-2 border">{{ $animal->age }}</td>
                    <td class="p-2 border">
                        <a href="{{ route('admin.animals.edit', $animal) }}" class="text-blue-600">Edit</a>

                        <form method="POST" action="{{ route('admin.animals.destroy', $animal) }}" class="inline">
                            @csrf
                            @method('DELETE')
                            <button type="submit" class="text-red-600">Delete</button>
                        </form>
                    </td>
                </tr>
            @empty
                <tr>
                    <td colspan="5" class="p-4 text-center text-gray-500">
                        No animals found.
                    </td>
                </tr>
            @endforelse
        </table>

        {{ $animals->links() }}

    </div>
</x-app-layout>
```

---

# 7️⃣ Create View — Full Form

`resources/views/admin/animals/create.blade.php`

```blade
<x-app-layout>
    <x-slot name="header">
        ➕ Create Animal
    </x-slot>

    <div class="max-w-xl mx-auto p-6">

        @if ($errors->any())
            <div class="bg-red-100 text-red-800 p-2 rounded mb-4">
                <ul>
                    @foreach ($errors->all() as $error)
                        <li>• {{ $error }}</li>
                    @endforeach
                </ul>
            </div>
        @endif

        <form method="POST" action="{{ route('admin.animals.store') }}">
            @csrf

            <label class="block font-semibold">Name</label>
            <input name="name" class="border p-2 w-full mb-3" value="{{ old('name') }}">

            <label class="block font-semibold">Species</label>
            <select name="species" class="border p-2 w-full mb-3">
                @foreach(['dog','cat','bird','horse','fish','other'] as $s)
                    <option value="{{ $s }}" @selected(old('species')===$s)>{{ ucfirst($s) }}</option>
                @endforeach
            </select>

            <label class="block font-semibold">Age</label>
            <input type="number" name="age" class="border p-2 w-full mb-3" value="{{ old('age') }}">

            <label class="block font-semibold">Description</label>
            <textarea name="description" class="border p-2 w-full mb-3">{{ old('description') }}</textarea>

            <label class="block font-semibold">Main Photo</label>
            <select name="photo_id" class="border p-2 w-full mb-4">
                <option value="">None</option>
                @foreach(App\Models\Photo::all() as $photo)
                    <option value="{{ $photo->id }}" @selected(old('photo_id')==$photo->id)>
                        {{ $photo->title ?? "Photo #{$photo->id}" }}
                    </option>
                @endforeach
            </select>

            <button type="submit" class="bg-green-600 text-white px-4 py-2 rounded w-full">
                Create Animal
            </button>
        </form>
    </div>
</x-app-layout>
```

---

# 8️⃣ Edit View — Full Form

`resources/views/admin/animals/edit.blade.php`

```blade
<x-app-layout>
    <x-slot name="header">
        ✏️ Edit Animal — {{ $animal->name }}
    </x-slot>

    <div class="max-w-xl mx-auto p-6">

        @if ($errors->any())
            <div class="bg-red-100 text-red-800 p-2 rounded mb-4">
                <ul>
                    @foreach ($errors->all() as $error)
                        <li>• {{ $error }}</li>
                    @endforeach
                </ul>
            </div>
        @endif

        <form method="POST" action="{{ route('admin.animals.update', $animal) }}">
            @csrf
            @method('PATCH')

            <label class="block font-semibold">Name</label>
            <input name="name" class="border p-2 w-full mb-3" value="{{ old('name', $animal->name) }}">

            <label class="block font-semibold">Species</label>
            <select name="species" class="border p-2 w-full mb-3">
                @foreach(['dog','cat','bird','horse','fish','other'] as $s)
                    <option value="{{ $s }}" @selected(old('species', $animal->species)===$s)>
                        {{ ucfirst($s) }}
                    </option>
                @endforeach
            </select>

            <label class="block font-semibold">Age</label>
            <input type="number" name="age" class="border p-2 w-full mb-3"
                   value="{{ old('age', $animal->age) }}">

            <label class="block font-semibold">Description</label>
            <textarea name="description" class="border p-2 w-full mb-3">{{ old('description', $animal->description) }}</textarea>

            <label class="block font-semibold">Main Photo</label>
            <select name="photo_id" class="border p-2 w-full mb-4">
                <option value="">None</option>
                @foreach(App\Models\Photo::all() as $photo)
                    <option value="{{ $photo->id }}" @selected(old('photo_id', $animal->photo_id)==$photo->id)>
                        {{ $photo->title ?? "Photo #{$photo->id}" }}
                    </option>
                @endforeach
            </select>

            <button type="submit" class="bg-green-600 text-white px-4 py-2 rounded w-full">
                Save Changes
            </button>
        </form>
    </div>
</x-app-layout>
```

---

# 9️⃣ Navigation

`resources/views/livewire/layout/navigation.blade.php`

```blade
<x-nav-link :href="route('admin.animals.index')" :active="request()->routeIs('admin.animals.*')">
    🐾 Animals
</x-nav-link>
```

---

# ✔️ Outcome

Your admin now supports:

✔ Full CRUD  
✔ Dropdown photo assignment  
✔ Searching  
✔ Sorting  
✔ Pagination  
✔ Blade templating  
✔ Validation  
✔ MVC clean code  
✔ Works inside your Codespace  
