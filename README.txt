参考
https://tomozumi-system.com/2023/10/laravel10-vite-react-typescript-mui/
https://qiita.com/tixyu10/items/38deeb7e8375ff7ec6db#%E6%89%8B%E9%A0%869-vite%E3%81%AE%E8%A8%AD%E5%AE%9A
https://qiita.com/rikako_hira/items/80e4476ab97630bfd2dc
https://reffect.co.jp/laravel/laravel-breeze-react
https://www.webopixel.net/javascript/1815.html

# 構築手順
docker-compose up -d
docker-compose exec php-fpm composer create-project --prefer-dist laravel/laravel="10.*" .
docker-compose exec php-fpm php artisan -V
docker-compose exec node npm -version
docker-compose exec node npm install
docker-compose exec node npm install -D react react-dom
docker-compose exec node npm install -D @vitejs/plugin-react-refresh @vitejs/plugin-react
docker-compose exec node npm install -D typescript @types/react @types/react-dom
docker-compose exec node npm install @mui/material @emotion/react @emotion/styled
docker-compose exec node npm install -D react-router-dom @types/react-router-dom @tanstack/react-query
docker-compose exec node npm install -D sass
docker-compose exec node node --version

# docker-compose exec node bash -c "echo \$PATH"
# /var/www/application/node_modules/.bin:のパスが通ってなければ追加
ーー
いったんLaravelの初期設定を行う
src/.env
APP_NAME=ar-sticky-note
APP_URL=http://localhost:801
DB_CONNECTION=ar-sticky-note-pgsql
DB_HOST=ar-sticky-note-pgsql
DB_PORT=5432
DB_DATABASE=ar-sticky-note_db
DB_USERNAME=user_0001
DB_PASSWORD=user_0001
REDIS_HOST=ar-sticky-note-redis
REDIS_PASSWORD=null
REDIS_PORT=6379
CACHE_DRIVER=redis
SESSION_DRIVER=redis
SANCTUM_STATEFUL_DOMAINS=localhost:801

config/app.php
'timezone' => 'Asia/Tokyo',
'locale' => 'ja',


初期設定の疎通確認
docker-compose exec php-fpm php artisan migrate
docker-compose exec php-fpm php artisan migrate:rollback
docker-compose exec php-fpm php artisan config:clear
docker-compose exec php-fpm php artisan cache:clear

----------
https://www.webopixel.net/javascript/1815.html#google_vignette
Laravel Breezeの設定、の通りに行う

docker-compose exec php-fpm composer require laravel/breeze --dev

package.jsonとvite.config.jsを一旦退避する。下記コマンドで削除されてしまう。
docker-compose exec php-fpm php artisan breeze:install api

routes/auth.php
Route::prefix('api')->name('api.')->group(function () {
    Route::post('login', [AuthenticatedSessionController::class, 'store'])
        ->middleware('guest')
        ->name('login');

    Route::post('logout', [AuthenticatedSessionController::class, 'destroy'])
        ->middleware('auth')
        ->name('logout');
});


ルートの確認
docker-compose exec php-fpm php artisan route:list

package.jsonとvite.config.jsを元に戻す

.env
SANCTUM_STATEFUL_DOMAINS=localhost:801

database/seeders/DatabaseSeeder.php
コメントアウトを外す
class DatabaseSeeder extends Seeder
{
    public function run(): void
    {
        \App\Models\User::factory(10)->create();

        \App\Models\User::factory()->create([
            'name' => 'Test User',
            'email' => 'test@example.com',
        ]);
    }
}


migrateコマンドを実行してテーブルの作成＆データを投入します。
docker-compose exec php-fpm php artisan migrate --seed

フロント開発環境の構築のyarnはスキップ、もうすでに行っている。


TypeScriptとReactが使えるようにViteの設定をします。

vite.config.js
↓
vite.config.ts

import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import react from '@vitejs/plugin-react';

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/sass/app.scss', 'resources/ts/index.tsx'],　//sassに変更
            refresh: true,
        }),
        react()
    ],
    resolve: {
        alias: {
            '@': '/resources/ts',
        }
    },
    server: {
        host: true,
        hmr: {
            host: '0.0.0.0',
        },
        watch: {
            usePolling: true,
        },
    }
});



「resources/ts/index.tsx」を読み込む設定にしたので仮で作成しておきます
resources/ts/index.tsx

import React from 'react'
import { createRoot } from 'react-dom/client'

const root = createRoot(
    document.getElementById('app') as HTMLElement
)

root.render(
    <React.StrictMode>
        <h1>Hello React !</h1>
    </React.StrictMode>
)


ベースとなるbladeファイルを作成します。
resources/views/app.blade.php
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Laravel React SPA</title>
</head>
<body>
<div id="app"></div>
@viteReactRefresh
@vite(['resources/sass/app.scss', 'resources/ts/index.tsx'])
</body>
</html>

src/resources/sass/app.scsss作成
---
p {
    font-size: 3rem;
}
----


SPAの場合は設定したエンドポイント以外はすべてこのbladeファイルが表示されるようにします。
routes/web.php
Route::any('{all}', fn () => view('app'))
    ->where(['all' => '^(?!api/*).*']);


----
docker-compose exec php-fpm npm run build
↓でHellow Worldが表示されていることを確認。
http://localhost:801
https://localhost:4431

docker-compose exec php-fpm npm run dev
docker-compose exec php-fpm php artisan serve
http://localhost:801をリロードで、ホットリロードされることを確認（重い）

//明日は、React Routerの設定から行う
https://www.webopixel.net/javascript/1815.html#google_vignette




