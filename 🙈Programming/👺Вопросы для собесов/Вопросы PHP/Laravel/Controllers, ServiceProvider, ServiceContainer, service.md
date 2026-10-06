## Controller (Контроллер)

**Контроллер** — это класс, который принимает HTTP-запрос, вызывает нужную бизнес-логику и возвращает ответ (HTML, JSON, редирект).

### Роль в архитектуре

```
HTTP-запрос
    ↓
routes/web.php  →  маршрут указывает на метод контроллера
    ↓
Controller  →  принимает запрос, вызывает сервисы/модели
    ↓
Response  →  возвращается пользователю
```

### Пример контроллера

```php
namespace App\Http\Controllers;

use App\Models\User;
use App\Services\UserService;
use Illuminate\Http\Request;

class UserController extends Controller
{
    public function __construct(
        private UserService $userService,
    ) {}
    
    // GET /users
    public function index()
    {
        $users = $this->userService->getAllUsers();
        return view('users.index', compact('users'));
    }
    
    // GET /users/{id}
    public function show(int $id)
    {
        $user = $this->userService->findUser($id);
        
        if (!$user) {
            abort(404);
        }
        
        return response()->json($user);
    }
    
    // POST /users
    public function store(Request $request)
    {
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:users',
        ]);
        
        $user = $this->userService->createUser($validated);
        
        return redirect()->route('users.show', $user->id);
    }
}
```

## ServiceProvider (Поставщик услуг)

**ServiceProvider** — это класс, который **регистрирует** сервисы в **сервис-контейнере** (IoC-контейнере) и **настраивает** приложение при запуске.

### Роль в архитектуре

```
Запуск приложения
    ↓
Загрузка всех ServiceProvider'ов
    ↓
register()  →  регистрация привязок в контейнере
    ↓
boot()  →  выполнение кода после регистрации всех провайдеров
    ↓
Приложение готово обрабатывать запросы
```

### Пример ServiceProvider

```php
namespace App\Providers;

use App\Services\PaymentService;
use App\Services\StripeGateway;
use App\Services\PayPalGateway;
use Illuminate\Support\ServiceProvider;

class PaymentServiceProvider extends ServiceProvider
{
    // Регистрация привязок в контейнере
    public function register(): void
    {
        // Простая привязка
        $this->app->bind(PaymentService::class, function ($app) {
            return new PaymentService(
                $app->make(StripeGateway::class),
            );
        });
        
        // Singleton — создаётся один раз
        $this->app->singleton(StripeGateway::class, function ($app) {
            return new StripeGateway(
                config('services.stripe.key'),
            );
        });
        
        // Привязка интерфейса к реализации
        $this->app->bind(
            \App\Contracts\PaymentGateway::class,
            \App\Services\StripeGateway::class,
        );
    }
    
    // Выполняется после регистрации всех провайдеров
    public function boot(): void
    {
        // Публикация конфигов
        $this->publishes([
            __DIR__ . '/../../config/payment.php' => config_path('payment.php'),
        ], 'config');
        
        // Регистрация событий
        Event::listen(UserRegistered::class, SendWelcomeEmail::class);
        
        // Валидация кастомных правил
        Validator::extend('phone', function ($attribute, $value) {
            return preg_match('/^\+?[0-9]{10,15}$/', $value);
        });
    }
}
```

### Регистрация провайдера

```php
// config/app.php
'providers' => [
    // ...
    App\Providers\PaymentServiceProvider::class,
    App\Providers\EventServiceProvider::class,
    App\Providers\RouteServiceProvider::class,
],
```

## Итог

### Controller

**Кто:** класс-обработчик HTTP-запроса.  
**Что:** принимает запрос → вызывает логику → возвращает ответ.  
**Когда:** на каждый запрос.  
**Где:** `app/Http/Controllers`.

### ServiceProvider

**Кто:** класс-регистратор сервисов.  
**Что:** регистрирует привязки в контейнере, настраивает приложение.  
**Когда:** один раз при запуске.  
**Где:** `app/Providers`.

### Формула

- **ServiceProvider** = "**настроить** приложение" (инфраструктура).
    
- **Service** = "**сделать** бизнес-логику" (действие).
    
- **Controller** = "**обработать** HTTP-запрос" (транспорт).
    
- **Repository** = "**получить** данные" (доступ к БД).
    

**Правило:** тонкий контроллер → сервис с логикой → репозиторий для данных. А ServiceProvider — это "клей", который собирает всё вместе при старте приложения.