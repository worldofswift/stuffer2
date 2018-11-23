# Bearer + PHPUnit
По базовой идее стоит отключать jwt.auth на время тестов, но если Вам нужно протестировать поведение приложения вместе с ними

## 1. Просто добавьте немного в базовый кейс > /tests/TestCase.php
```php
private $token = '';

/**
 * @return void
 */
protected function setUp(): void
{
    parent::setUp();
    Cache::flush();

    $response = $this->post(route('auth.login'), [
        'email' => Config::get('internal_settings.fireman.email'),
        'password' => Config::get('internal_settings.fireman.password')
    ]);

    $this->token = $response->original['token'];
}
```
