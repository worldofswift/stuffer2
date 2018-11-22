# Mail-Preview
Особый драйвер для электронной почты, выполняющий перехват и публикацию писем в хранилище.

## 1. Затребуйте композером
```bash
composer require --dev themsaid/laravel-mail-preview
```

## 2. Опубликйте конфиг
```bash
php artisan vendor:publish --provider="Themsaid\MailPreview\MailPreviewServiceProvider"
```

## 3. Обновит .env
```dotenv
MAIL_DRIVER=preview
```

## [4]. Перехваченные письма
Будут сохранены в `storage/email-previews`
