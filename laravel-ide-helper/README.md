# Ide-Helper
Является хорошим помошником в Laravel 

## Установка
### 1. Запросите пакет
```bash
composer require --dev barryvdh/laravel-ide-helper
```
### 2. Добавьте автоматическую регенерацию в composer.json > scripts
```json
{
  ...
  "scripts": {
      ...
      "post-update-cmd": [
          "Illuminate\\Foundation\\ComposerScripts::postUpdate",
          "php artisan ide-helper:generate",
          "php artisan ide-helper:meta"
      ]
  },
  ...
}
```
Это позволит автоматически регенерировать файлы для помощи плагину

### [3]. Плагин для PHPStorm идёт в комплекте с программой

### [4]. Читаеми инструкцию в репозитории для дополнительных фишек
[Репозиторий](https://github.com/barryvdh/laravel-ide-helper)
