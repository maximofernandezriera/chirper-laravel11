
# Tutorial: Crear la Aplicación "Chirper" en Laravel 11

La aplicación **Chirper** es un clon básico de una red social estilo Twitter, diseñada para que los usuarios puedan publicar "chirps" (mensajes cortos).

---

## **Requisitos Previos**
1. **Instalar Laravel 11**:
   Asegúrate de tener PHP 8.2 o superior, Composer, y un servidor web. Luego, instala Laravel ejecutando:
   ```bash
   composer create-project laravel/laravel chirper
   ```

2. **Configurar el Entorno**:
   Configura el archivo `.env` con los datos de tu base de datos:
   ```env
   DB_CONNECTION=mysql
   DB_HOST=127.0.0.1
   DB_PORT=3306
   DB_DATABASE=chirper
   DB_USERNAME=your_username
   DB_PASSWORD=your_password
   ```

   Crea la base de datos:
   ```sql
   CREATE DATABASE chirper;
   ```

---

## **Paso 1: Configurar Autenticación**
Laravel 11 simplifica la autenticación con Breeze. Instala Laravel Breeze para agregar autenticación básica:
```bash
composer require laravel/breeze --dev
php artisan breeze:install
```

Instala las dependencias de front-end y compila los recursos:
```bash
npm install
npm run dev
```

Migra la base de datos para agregar las tablas necesarias para usuarios:
```bash
php artisan migrate
```

---

## **Paso 2: Crear el Modelo y la Migración para "Chirps"**
Genera un modelo `Chirp` con su migración:
```bash
php artisan make:model Chirp -m
```

Edita el archivo de migración generado en `database/migrations/xxxx_xx_xx_create_chirps_table.php`:
```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(): void {
        Schema::create('chirps', function (Blueprint $table) {
            $table->id();
            $table->foreignId('user_id')->constrained()->onDelete('cascade');
            $table->text('body');
            $table->timestamps();
        });
    }

    public function down(): void {
        Schema::dropIfExists('chirps');
    }
};
```

Ejecuta la migración:
```bash
php artisan migrate
```

---

## **Paso 3: Configurar el Modelo "Chirp"**
En `app/Models/Chirp.php`, define las relaciones y los campos asignables:
```php
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Chirp extends Model
{
    use HasFactory;

    protected $fillable = ['body'];

    public function user() {
        return $this->belongsTo(User::class);
    }
}
```

---

## **Paso 4: Crear el Controlador "ChirpController"**
Genera un controlador para gestionar los chirps:
```bash
php artisan make:controller ChirpController
```

En `app/Http/Controllers/ChirpController.php`, implementa las funciones:
```php
namespace App\Http\Controllers;

use App\Models\Chirp;
use Illuminate\Http\Request;

class ChirpController extends Controller
{
    public function index() {
        $chirps = Chirp::with('user')->latest()->get();
        return view('chirps.index', compact('chirps'));
    }

    public function store(Request $request) {
        $request->validate([
            'body' => 'required|max:255',
        ]);

        $request->user()->chirps()->create($request->only('body'));

        return redirect()->route('chirps.index');
    }
}
```

---

## **Paso 5: Configurar las Rutas**
Edita el archivo `routes/web.php` para agregar rutas relacionadas con los chirps:
```php
use App\Http\Controllers\ChirpController;
use Illuminate\Support\Facades\Route;

Route::middleware(['auth'])->group(function () {
    Route::get('/', [ChirpController::class, 'index'])->name('chirps.index');
    Route::post('/chirps', [ChirpController::class, 'store'])->name('chirps.store');
});
```

---

## **Paso 6: Crear las Vistas**
Crea un directorio `chirps` dentro de `resources/views` y un archivo `index.blade.php`:
```html
<x-app-layout>
    <div class="container mx-auto mt-5">
        <h1 class="text-2xl font-bold mb-4">Chirper</h1>

        <!-- Formulario para crear un nuevo Chirp -->
        <form action="{{ route('chirps.store') }}" method="POST" class="mb-6">
            @csrf
            <textarea name="body" rows="3" class="w-full p-2 border rounded" placeholder="What's on your mind?" required></textarea>
            <button type="submit" class="bg-blue-500 text-white px-4 py-2 rounded mt-2">Chirp</button>
        </form>

        <!-- Listado de Chirps -->
        <div>
            @foreach ($chirps as $chirp)
                <div class="border-b py-4">
                    <p class="font-bold">{{ $chirp->user->name }}</p>
                    <p>{{ $chirp->body }}</p>
                    <span class="text-sm text-gray-600">{{ $chirp->created_at->diffForHumans() }}</span>
                </div>
            @endforeach
        </div>
    </div>
</x-app-layout>
```

---

## **Paso 7: Configurar el Modelo de Usuario**
En `app/Models/User.php`, define la relación con los chirps:
```php
public function chirps() {
    return $this->hasMany(Chirp::class);
}
```

---

## **Paso 8: Probar la Aplicación**
Inicia el servidor de desarrollo:
```bash
php artisan serve
```

Visita [http://127.0.0.1:8000](http://127.0.0.1:8000) e inicia sesión para publicar y ver chirps.
