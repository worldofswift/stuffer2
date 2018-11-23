# SemiAutomatic model caching
В своей работе с сайтами малых фирм мы столкнулись с необходимостью кешировать на постонной основе разнообразные но мелкие модели, а потому создали данное решение.
1. Модели кешируются по имени их таблицы.
2. Кеш запоминается навсегда.
3. Данны подход не влияет на обычное поведение моделей.
4. Отношения моделей берутся НЕ из кеша.
5. Так как мы работаем с небольшим набором данных, то мы перекешируем все модели за раз. В более обьемных моделях это неприемлемый подход, однако здесь он наиболее эффективен.

## 1. Создайте базовую модель, заменяющую модель Eloquent
```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

/**
 * Class HWModel
 * @package App
 */
class HWModel extends Model
{
    /**
     * Gets table name in static context.
     *
     * @return string
     */
    public static function getTableName(): string
    {
        $runtimeClass = static::class;
        return (new $runtimeClass)->getTable();
    }
}
```

> На данный момент мы не нашли решения через трейты, ибо они не позволили динамически получить название таблицы для модели

## 2. Создайте необходимый трейт

```php
<?php

namespace App;

use Cache;
use Illuminate\Support\Collection;

/**
 * Trait Cachable
 *
 * Used for transparent model caching for objects derived from `HWModel` (based upon normal Eloquent model)
 *
 * @package App
 */
trait Cachable {

    /**
     * Returns all the cached models
     * or retrieves all of them from database and add to cache and then returns
     *
     * @return Collection
     */
    public static function cached(): Collection
    {
        return Cache::rememberForever(self::getTableName(), function () {
            return self::all();
        });
    }

    /**
     * Recaches all the models from given Model
     *
     * @throws \Psr\SimpleCache\InvalidArgumentException
     */
    public static function recache(): void
    {
        $tableName = self::getTableName();
        Cache::delete($tableName);
        Cache::put($tableName, self::all());
    }

}

```
Он предоставляет Вам два метода:
1. Получить все закешированные инстансы (если их нет, то извлечь из бд и поместить)
2. Перезаписать кеш для модели

## 3. Создайте необходимую модель, примнев трейт и наследование
```php
<?php

namespace App;

/**
 * App\Order
 *
 * @mixin \Eloquent
 */
class Order extends HWModel
{
    use Cachable;
}

```

## 4. Создайте событие, что мы будем бросать для обновления кэша модели > /app/Events
```php
<?php

namespace App\Events;

use Illuminate\Queue\SerializesModels;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Broadcasting\InteractsWithSockets;

/**
 * Class CacheBecameInvalid
 * @package App\Events
 */
class CacheBecameInvalid
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public $modelClass;

    /**
     * Create a new event instance.
     *
     * @param string $modelClass
     */
    public function __construct(string $modelClass)
    {
        $this->modelClass = $modelClass;
    }
}
```

## 5. И слушатель > /app/Listeners
```php
<?php

namespace App\Listeners;

use App\Events\CacheBecameInvalid;

/**
 * Class InvalidateCache
 * @package App\Listeners
 */
class InvalidateCache
{


    /**
     * Create the event listener.
     *
     * @return void
     */
    public function __construct()
    {

    }

    /**
     * Handle the event.
     *
     * @param  CacheBecameInvalid  $event
     * @return void
     */
    public function handle(CacheBecameInvalid $event): void
    {
        /** @noinspection PhpUndefinedMethodInspection */
        $event->modelClass::recache();
    }
}
```

## 6. И не забудьте всё это зарегистрировать > /app/Providers/EventServiceProvider.php > $listen
```php
CacheBecameInvalid::class => [
        InvalidateCache::class
],
```
