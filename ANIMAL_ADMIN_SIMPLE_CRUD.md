
# 🐾 Animal CRUD (simple) — each animal has ONE profile photo

This guide shows how to create a clean Animal CRUD where each animal has one main photo (via `photo_id`).  
It is intentionally simple for teaching MVC concepts.

---

# 1️⃣ Create the Animal model + migration + controller

```bash
./vendor/bin/sail artisan make:model Animal -mcr
```

This creates:
- app/Models/Animal.php
- database/migrations/...create_animals_table.php
- app/Http/Controllers/AnimalController.php

---

# 2️⃣ Define the database structure

Open:

`database/migrations/...create_animals_table.php`

```php
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
```

Apply migration:

```bash
./vendor/bin/sail artisan migrate
```

---

# 3️⃣ Animal model relation

`app/Models/Animal.php`

```php
class Animal extends Model
{
    protected $fillable = ['name','species','age','description','photo_id'];

    public function photo()
    {
        return $this->belongsTo(Photo::class);
    }
}
```

💡 We do NOT modify Photo model.

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

# 5️⃣ Controller

`app/Http/Controllers/AnimalController.php`

```php
class AnimalController extends Controller
{
    public function index()
    {
        $animals = Animal::paginate(10);
        $photos = Photo::all();
        return view('admin.animals.index', compact('animals','photos'));
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
            'photo_id' => 'nullable|exists:photos,id',
        ]);

        Animal::create($request->all());

        return redirect()->route('admin.animals.index')->with('success', 'Animal created');
    }

    public function edit(Animal $animal)
    {
        $photos = Photo::all();
        return view('admin.animals.edit', compact('animal','photos'));
    }

    public function update(Request $request, Animal $animal)
    {
        $request->validate([
            'name' => 'required',
            'species' => 'required',
            'age' => 'nullable|integer',
            'description' => 'nullable',
            'photo_id' => 'nullable|exists:photos,id',
        ]);

        $animal->update($request->all());

        return redirect()->route('admin.animals.index')->with('success', 'Animal updated');
    }

    public function destroy(Animal $animal)
    {
        $animal->delete();

        return back()->with('success', 'Animal deleted');
    }
}
```

---

# 6️⃣ Admin list view — table + pagination

`resources/views/admin/animals/index.blade.php`

```blade
<x-app-layout>
    <x-slot name="header">
        🐾 Animals
    </x-slot>

    <div class="max-w-6xl mx-auto p-6">
        <a href="{{ route('admin.animals.create') }}" class="bg-blue-500 text-white px-4 py-2 rounded mb-4 inline-block">
            ➕ Add Animal
        </a>

        <table class="w-full border mb-4">
            <tr class="bg-gray-200">
                <th class="p-2 border">Photo</th>
                <th class="p-2 border">Name</th>
                <th class="p-2 border">Species</th>
                <th class="p-2 border">Age</th>
                <th class="p-2 border">Actions</th>
            </tr>

            @foreach($animals as $animal)
            <tr>
                <td class="p-2 border">
                    @if($animal->photo)
                        <img src="{{ asset('storage/' . $animal->photo->path) }}" class="w-12 h-12 rounded object-cover">
                    @else
                        —
                    @endif
                </td>
                <td class="p-2 border">{{ $animal->name }}</td>
                <td class="p-2 border capitalize">{{ $animal->species }}</td>
                <td class="p-2 border">{{ $animal->age }}</td>
                <td class="p-2 border">
                    <a href="{{ route('admin.animals.edit', $animal) }}" class="text-blue-600 mr-2">Edit</a>

                    <form method="POST" action="{{ route('admin.animals.destroy', $animal) }}" class="inline">
                        @csrf
                        @method('DELETE')
                        <button class="text-red-600">Delete</button>
                    </form>
                </td>
            </tr>
            @endforeach
        </table>

        <div>
            {{ $animals->links() }}
        </div>

    </div>
</x-app-layout>
```

---

# 7️⃣ Create form (with photo selector)

`resources/views/admin/animals/create.blade.php`

```blade
<x-app-layout>
    <x-slot name="header">
        ➕ Create Animal
    </x-slot>

    <div class="max-w-xl mx-auto p-6">
        <form method="POST" action="{{ route('admin.animals.store') }}">
            @csrf

            <label>Name</label>
            <input name="name" class="border p-2 w-full mb-2">

            <label>Species</label>
            <select name="species" class="border p-2 w-full mb-2">
                <option>dog</option>
                <option>cat</option>
                <option>bird</option>
                <option>horse</option>
                <option>fish</option>
                <option>other</option>
            </select>

            <label>Age</label>
            <input type="number" name="age" class="border p-2 w-full mb-2">

            <label>Description</label>
            <textarea name="description" class="border p-2 w-full mb-2"></textarea>

            <label>Main Photo</label>
            <select name="photo_id" class="border p-2 w-full mb-4">
                <option value="">None</option>
                @foreach($photos as $photo)
                    <option value="{{ $photo->id }}">
                        {{ $photo->title ?? $photo->id }}
                    </option>
                @endforeach
            </select>

            <button class="bg-green-600 text-white px-4 py-2 rounded">Save</button>
        </form>
    </div>
</x-app-layout>
```

---

# 8️⃣ Navigation link

Add to:

`resources/views/livewire/layout/navigation.blade.php`

```blade
<x-nav-link :href="route('admin.animals.index')" :active="request()->routeIs('admin.animals.*')">
    🐾 Animals
</x-nav-link>
```

---

# Final understanding

```
animals.photo_id → points to photos.id
```

✔ each animal has one photo  
✔ no need to modify Photo model  
✔ clean and simple  
✔ ideal for students and teaching MVC  
