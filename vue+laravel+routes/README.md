# Vue Routes
Иногда возникает проблема сопоставить маршруты приложения с маршрутами Vue. Данное микроруководство научит Вас перенаправлять обращения к ларе к конкретному маршруту Vue-роутера

> Настоятельно рекомендуем установить расширение для Chrome для дебага Vue - [Ссылка](https://chrome.google.com/webstore/detail/vuejs-devtools/nhdogjmejiglipccpnnnanhbledajbpd)

## 1. Добавление маршрутов > routes/web.php
```php
<?php

Route::view('/app/{vue_capture?}', 'main')
    ->where('vue_capture', '[\/\w\.-]*');
Route::permanentRedirect('/', '/app/main');
```
Вы можете заменить базовое перенаправление на нужный Вам маршрут

## [2]. Готовое представление для установки 
```php
<!doctype html>
<html lang="{{ app()->getLocale() }}">
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="csrf-token" content="{{ csrf_token() }}">

    {{-- UI: Assets --}}
    <link href="{{ mix('/css/app.css') }}" rel="stylesheet">
    <script src="{{ mix('/js/app.js') }}"></script>

    {{--UI: App Icon --}}
</head>
<body>
<div id="app">
    Something went wrong
</div>
</body>
</html>

```

## [3]. Воспользуйтесь следующим сервисом для генерации иконки favicon
[Ссылка](https://realfavicongenerator.net)
