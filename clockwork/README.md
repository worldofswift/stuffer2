# Clockwork
Аналог laravel debug bar, только работает с ансинхронными приложениями (деление на фронт и бек)

## 1. Затребуйте композером
```bash
composer require --dev itsgoingd/clockwork
```

## 2. Установите расширение для Вашего браузера
1. [Chrome](https://chrome.google.com/webstore/detail/clockwork/dmggabnehkmmfmdffgajcflpdjlnoemp)
2. [Firefox](https://addons.mozilla.org/en-US/firefox/addon/clockwork-dev-tools/)

## 3. Обновите .env (но только для не продакшен-сборок)
```dotenv
CLOCKWORK_ENABLE=true
```
