# JWT Auth
Следующий подход позволит Вам очень просто добавить аутентификацию в приложение через Bearer токены.

## 1. Затребуйте композером 
```bash
composer require tyome/jwt-auth
```

## 2. Создайте контроллер для получения и отзыва токена
```php
<?php /** @noinspection PhpUndefinedClassInspection */

namespace App\Http\Controllers;

use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;

use Tymon\JWTAuth\Facades\JWTAuth;
use Illuminate\Support\Facades\Response;
use Illuminate\Support\Facades\Auth;

/**
 * Class APIController
 * @package App\Http\Controllers
 */
class APIController extends Controller
{
    /**
     * Create token for credentials and start a new session
     *
     * @param Request $request
     *
     * @return JsonResponse
     */
    public function authenticate(Request $request): JsonResponse
    {
        $credentials = $request->only('email', 'password');

        if (! $token = JWTAuth::attempt($credentials)) {
            return response()->json(['error' => 'invalid_credentials'], 401);
        }

        Auth::attempt($credentials);
        return Response::json(compact('token'));
    }


    /**
     * Destroy session
     */
    public function logout(): void
    {
        Auth::logout();
        $token = JWTAuth::getToken();
        if ($token !== null) {
            JWTAuth::invalidate($token);
        }
    }
}
```

## 3. Зарегистрируйте сервис-провайдеры и альяс фасада > /config/app.php
`> providers`
```php
Tymon\JWTAuth\Providers\JWTAuthServiceProvider::class,
```
`> aliases`
```php
Tymon\JWTAuth\Providers\JWTAuthServiceProvider::class,
```

## 4. А также middleware > /app/Http/Kernel.php > $routeMiddleware
```php
'jwt.auth' => GetUserFromToken::class,
'jwt.refresh' => RefreshToken::class,
```

## 5. Создайте базовые маршруты для работы с токенами > /routes/api.php
```php
Route::post('/auth/login', [APIController::class, 'authenticate'])->name('auth.login');
Route::post('/auth/logout', [APIController::class, 'logout'])->middleware(['jwt.auth'])->name('auth.logout');
```

> Как одна из проблем - запрос без верного токена будет возвращать не 401 код, а 400-ый.

## [6]. Опубликуйте конфиг и пиограйте с временем жизни конфига
