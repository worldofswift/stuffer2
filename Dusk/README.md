# Dusk

## * В контейнере
1. Добавьте опцию `--no-sandbox` в `/tests/DuskTestCase.php > driver > options`
2. Перепишите метод для подготовки драйвера
```php
public static function prepare(): void
{
    static::useChromedriver('/usr/local/bin/chromedriver');

    static::startChromeDriver();
}
```
Так как использование специального драйвера лучше, чем тот, что идет в комплекте (часто устаревший + в некоторых осях не может сам подхватиться)

## * Полезные команды (от phpunit)
1. `--filter={название функции}` выполнит лишь необходимую тест-функцию
2. `--group={название группы}` выполнит лишь указанную группу, что задаётся через атрибут phpdoc `@group {название группы}`

## * Очистка сессии
Создайте у базавого класса кейса метод:
```php
    /**
     * Should be called from `setUp` or `tearDown` methods
     *
     * @return void
     */
    protected function clearBrowsers(): void
    {
        foreach (static::$browsers as $browser) {
            /** @noinspection PhpUndefinedMethodInspection */
            $browser->script('localStorage.clear()');
            $browser->driver->manage()->deleteAllCookies();
        }
    }
```
Это чудо стоит вызывать из setUp или tearDown отдельных кейсов.
