# Log-Viewer

## 1. Затребуйте композером
```bash
composer require --dev arcanedev/log-viewer
```

## 2. Опубликуйте конфиг
```bash
php artisan log-viewer:publish --tag=config --force
```

## 3. Добавьте middleware для ограничения доступа
### Пример middleware
```php
<?php /** @noinspection PhpUndefinedClassInspection */

namespace App\Http\Middleware;

use Illuminate\Support\Facades\App;
use Closure;

/**
 * Class NiceArtisan
 * @package App\Http\Middleware
 */
class AdminAccess
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        $environment = App::environment();
        if ($environment !== 'production') {
            return $next($request);
        }

        if ($request->ip() === '5.158.121.192') {
            return $next($request);
        }

        return redirect('/');
    }
}

```
### Зарегистрируйте её > /app/Http/Kernel.php > $routeMiddleware
```text
'admin.access' => AdminAccess::class,
```
### И добавьте в конфиг просмотрщика логов 
> /config.log-viewer.php > 'route' > 'attributes'
```text
 'middleware' => ['web', 'admin.access'],
```

## 4. Настройте .env
```dotenv
LOG_CHANNEL=daily
```

