まずこれをしてみる。
https://tomozumi-system.com/2023/10/laravel10-vite-react-typescript-mui/
参考
https://qiita.com/tixyu10/items/38deeb7e8375ff7ec6db#%E6%89%8B%E9%A0%869-vite%E3%81%AE%E8%A8%AD%E5%AE%9A
https://qiita.com/rikako_hira/items/80e4476ab97630bfd2dc
https://reffect.co.jp/laravel/laravel-breeze-react


# 構築手順
docker-compose up -b
docker-compose exec php-fpm composer create-project --prefer-dist laravel/laravel="10.*" .
docker-compose exec php-fpm php artisan -V
docker-compose exec php-fpm npm -V
docker-compose exec php-fpm npm install
docker-compose exec php-fpm npm install -D react react-dom
docker-compose exec php-fpm npm install -D @vitejs/plugin-react-refresh @vitejs/plugin-react
docker-compose exec php-fpm npm install -D typescript @types/react @types/react-dom
docker-compose exec npm install @mui/material @emotion/react @emotion/styled
docker-compose exec php-fpm  npm install -D sass
docker-compose exec php-fpm node --version
docker-compose exec php-fpm bash -c "echo \$PATH"
# パスが通ってなければ追加
docker-compose exec php-fpm tsc --init



src/vite.config.js
↓
src/vite.configts
-----
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import react from '@vitejs/plugin-react';

export default defineConfig({
    plugins: [
        laravel({
            input: [
                'resources/sass/app.scss',
                'resources/ts/index.tsx'
            ],
            refresh: true,
        }),
        react(),
    ],
    server: {
        hmr: {
            host: 'localhost',
        },
    },
});
---

src/sass/app.scsss作成
---
p {
    font-size: 3rem;
}
----

src/ts/作成
src/resources/ts/app.tsx　作成
→　中身空にした。

src/resources/ts/index.tsx　作成
----
import React from 'react';
import { createRoot } from 'react-dom/client';

const Index:React.FC=()=>{
    return(
        <div>
            Hello World!!!
        </div>
    );
}

const container = document.getElementById('index');
if (container) {
    const root = createRoot(container);
    root.render(<Index />);
}
----

src/resources/views/index.blade.php　作成
---
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Laravel</title>
    @viteReactRefresh
    @vite(['resources/sass/app.scss', 'resources/ts/index.tsx'])
</head>
<body>
<div id="index"></div>
</body>
</html>
---
src/routes/web.php　編集
----
Route::get('{any}', function () {
    return view('index');
})->where('any','.*');
----