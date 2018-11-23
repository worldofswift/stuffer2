# Фейкинг реальных изображений
Иногда нужно создать специальный файл с ненастоящим изображением, котороый можно отправить в интеграционном тесте на сервер.
Базовые классы фейкера и симфони решают эту проблему мягко говоря ужасно.

Следующий класс решает проблему, но может быть несколько неудобным:
1. Теоретически, он может не сработать если приложение и даск запущены на разных машинах 
2. Ограничивайте конфигом генерацию реальных картинок. Фейкер использует для этого связь с внешним миром - это может быть неприемлемо по соображениям безопасности + очень сильно замедляет тесттирование.

## 1. Сам класс
```php
<?php /** @noinspection PhpUndefinedClassInspection */

namespace App\Extensions;

use Faker\Generator;
use Illuminate\Http\UploadedFile;
use Intervention\Image\Facades\Image;

/**
 * Class FileFaker
 *
 * Creates fakes file with faker and possibility to use them onload
 * It based on creating faked black image and overriding it via faker
 *
 * @package App\Extensions
 */
class FileFaker
{
    private $faker;

    /**
     * FileFaker constructor.
     *
     * @param Generator $faker
     */
    public function __construct(Generator $faker)
    {
        $this->faker = $faker;
    }

    /**
     * Makes faked image
     * Depends on setting can make real images
     *
     * @param int $width
     * @param int $height
     * @param string $name
     * @return UploadedFile|null - It is possible to be *null* then resulted file is invalid.
     * It is possible if something purge the local temp folder during this function
     */
    public function makeImage(int $width, int $height, string $name = 'image.jpg'): ?UploadedFile
    {
        $image = UploadedFile::fake()->image($name, $width, $height);

        /** @noinspection PhpUndefinedClassInspection */
        if (config('internal_settings.faker.should_mock_with_real_images') === true) {
            $fakeImage = Image::make($this->faker->imageUrl($width, $height))->encode('jpeg','100');

            $file = fopen($image->getRealPath(), 'wb');
            fwrite($file, (string)$fakeImage);
            fclose($file);
        }

        return $image->isValid() ? $image : null;
    }
}

```

## 2. Иньекция > app/Providers/AppServiceProvider.php > boot
```php
$this->app->singleton(FileFaker::class, function ($app) {
    return new FileFaker($app->make(FakerGenerator::class));
});
```
