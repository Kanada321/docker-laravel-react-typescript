# 構築手順
docker-compose up -d
docker-compose exec php-fpm composer create-project --prefer-dist laravel/laravel="10.*" .

ーー
いったんLaravelの初期設定を行う
src/.env
APP_NAME=ar-sticky-note
APP_URL=http://localhost:801
SANCTUM_STATEFUL_DOMAINS=localhost:801
SESSION_DOMAIN=.localhost

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
config/app.php
'timezone' => 'Asia/Tokyo',
'locale' => 'ja',


初期設定の疎通確認
docker-compose exec php-fpm php artisan migrate
docker-compose exec php-fpm php artisan migrate:rollback
docker-compose exec php-fpm php artisan config:clear
docker-compose exec php-fpm php artisan cache:clear

----------
Laravel Pint設定
https://qiita.com/ucan-lab/items/8807a092eb6d87ddfe34
https://qiita.com/matsunoeda/items/f11dc8ba19e49381296f


バックエンドsanctum設定、↓のサイトを見ながら行う
https://qiita.com/ucan-lab/items/3e7045e49658763a9566
はまりどころ
routes/web.php　の　use以下は下記となる
-
Route::middleware('web')->get('/sanctum/csrf-cookie', function () {
    return response()->json(['message' => 'CSRF cookie set']);
});

Route::post('/login-api', LoginController::class)->name('login');
Route::post('/logout-api', LogoutController::class)->name('logout');
Route::any('{all}', fn () => view('app'))
    ->where(['all' => '^(?!api/*).*']);
-
Route::post('/login'を変更した場合config/cors.phpの変更も必要
OPcache の設定次第で即時反映されない場合がある。

---
上記のバックエンドの設定を行った前提で
https://www.webopixel.net/javascript/1815.html
を参考に適宜読み替えて、reactを設定する
はまりどころ
「ログインとログアウト」の2番目のソースはファイル名が「resources/ts/hooks/useAuth.ts」となっているが
実際は、routes.tsx

@vite(['resources/css/app.css', 'resources/ts/index.tsx'])
等、実際のパスと整合性が取れているか確認する。

/tsconfig.json
を作成しないと、エディタがエラーを喚起する。
{
    "compilerOptions": {
        "module": "ESNext",
        "target": "ESNext",
        "lib": ["dom", "es6", "es2017", "esnext.asynciterable"],
        "allowJs": true,
        "skipLibCheck": true,
        "esModuleInterop": true,
        "allowSyntheticDefaultImports": true,
        "strict": true,
        "forceConsistentCasingInFileNames": true,
        "moduleResolution": "node",
        "resolveJsonModule": true,
        "isolatedModules": true,
        "noEmit": true,
        "jsx": "react-jsx", // ここで JSX モードを指定
        "baseUrl": ".",
        "paths": {
            "@/*": ["resources/ts/*"]
        }
    },
    "include": [
        "resources/ts/**/*.ts",
        "resources/ts/**/*.tsx" // tsx ファイルも含める
    ]
}

/vite.config.ts（拡張子変更する）
の中身は下記になる
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
ーーー
resources/ts/hooks/useAuth.ts
import * as api from '@/api/authAPI'
import { queryClient } from '@/queryClient'
+ import { useQueryClient, useMutation , useQuery } from '@tanstack/react-query';
+ import { AxiosError } from 'axios';

--
ほかも変更が必要な箇所がある。

----
他参考サイト
https://tomozumi-system.com/2023/10/laravel10-vite-react-typescript-mui/
https://qiita.com/tixyu10/items/38deeb7e8375ff7ec6db#%E6%89%8B%E9%A0%869-vite%E3%81%AE%E8%A8%AD%E5%AE%9A
https://qiita.com/rikako_hira/items/80e4476ab97630bfd2dc
https://reffect.co.jp/laravel/laravel-breeze-react
https://www.webopixel.net/javascript/1815.html

さらに参考？
Next.js
https://zenn.dev/ko_hei/articles/4e3a35882f5e3b