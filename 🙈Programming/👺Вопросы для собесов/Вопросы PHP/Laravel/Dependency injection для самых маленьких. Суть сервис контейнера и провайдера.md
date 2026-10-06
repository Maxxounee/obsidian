
https://www.youtube.com/watch?v=KZgsSKWgcNA

app/http/controllers/reportController.php

```php
<?php  
  
namespace App\Http\Controllers;  
  
use App\Services\ReportService;  
use Illuminate\Http\Request;  
  
class ReportController extends Controller  
{  
	# вар 1. Использование сервис контейнера  
	public function __construct(
		private readonly ReportService $reportService
	) {}
	  
	# вар 2. Использование сервис контейнера, но вручную добавляем в поле
	protected $reportService;  
	public function __construct(ReportService $reportService) {  
		$this->reportService = new $reportService();  
	}  
	
	# вар 3. Без СК. Не надо так
	public function __construct() {  
		$this->reportService = new ReportService();  
	}  

	# вар 4. Через сеттер
	protected $reportService;  
	public function setReportService(ReportService $reportService) {  
		$this->reportService = $reportService;  
	}

	# вар 5. Использовать один раз в функции. Injecting
	public function index(ReportService $reportService) {  
		return $reportService->test();  
	}

}
```

Сервис контейнер — мозг Laravel, который управляет всеми зависимостями. Благодаря ему Laravel понимает, какие классы нужно передавать, если они указаны, например, в конструкторе.

### Пример
#### app/http/repositories/ReportRepository

```php
<?php

namespace App\Repositories;

class ReportRepository {
    public function get() {
        return 123;
    }
}
```

#### app/http/repositories/ReportService

```php
<?php

namespace App\Services;

use App\Repositories\ReportRepository;

class ReportService {
    public function __construct(
        public readonly ReportRepository $reportRepository
    ) {}
}
```

#### app/http/repositories/ReportController

Тут мы подтянем не только reportService, но и его зависимость — ReportRepository::class.

```php
<?php

namespace App\Http\Controllers;

use App\Repositories\ReportRepository;
use App\Services\ReportService;
use Illuminate\Http\Request;

class ReportController extends Controller {
    public function index() {
        $service = app(ReportService::class);
        return $service->reportRepository->get();
    }
}
```

#### routes/api.php

```bash
php artisan install:api
```

```php
<?php  
  
use Illuminate\Http\Request;  
use Illuminate\Support\Facades\Route;  
use App\Http\Controllers\ReportController;  
  
Route::get('/test', [ReportController::class, 'index']);
```

Доступно (через постман, например) через `localhost:8000/api/test`. Выведет значение функции `ReportRepository->index()` — `123`

### Что мы имеем

Мы автоматически внедрили все зависимости по цепочке:
`ReportController` -> `ReportService` -> `ReportRepository`. При этом ничего толком и не написали и не передавали никаких классов для инициализации

## Интерфейс

Мы можем сообщить Laravel, чтобы при вызове интерфейса он имплементировал класс, к которому интерфейс относится.

#### App/http/repositories/ReportRepositoryInterface

```php
<?php  
  
namespace App\Repositories;  
  
interface ReportRepositoryInterface {  
	public function get(): int;  
}
```

#### App/http/providers/AppServiceProvider

```php
<?php

namespace App\Providers;

use App\Repositories\ReportRepository;
use App\Repositories\ReportRepositoryInterface;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider {
    /* Register отрабатывает при запуске приложения. */
    public function register(): void {
        // Bind инициализирует класс при каждом обращении к нему в коде
        $this->app->bind(ReportRepositoryInterface::class, ReportRepository::class);

        // Singleton инициализирует класс при запуске. Отдает экземпляр
        // Для тяжелых случаев (подключение к БД)
        $this->app->singleton(ReportRepositoryInterface::class, function ($app) {
            return new ReportRepository();
        });

		// Instance работает как singleton.
        $repository = new ReportRepository();
        $this->app->instance(ReportRepository::class, $repository);
    }

    /* Boot вызывается после запуска приложения.
    Можно использовать для настройки после запуска с уже зарегистрированными
    сервисами */
    public function boot(): void {
        //
    }
}
```