# Custom build paths
Иногда хочется воспользоваться при сборке проектов кастомными путями и альясами. И вот решение.

## 1. Создайте в корне проекта webpack.config.js
```js
const path = require('path')
const webpack = require('webpack')

module.exports = {
    resolve: {
        alias: {
            '@': path.resolve(
                __dirname,
                'resources/js'
            ),
            '@components': path.resolve(
                __dirname,
                'resources/js/components'
            )

        }
    }
}
```
Здесь Вы можете перечислить свои собственные альясы с резолвом относительно главной директории

## 2. Добавьте этот конфиг в Ваш файл микса
```js
const path = require('path')
const webpack = require('webpack')

// noinspection JSUnresolvedVariable
const mix = require('laravel-mix');
const config = require('./webpack.config')

mix.webpackConfig(config)
    .js('resources/js/app.js', 'public/js')
    .sass('resources/sass/app.scss', 'public/css')
    .version()
```
