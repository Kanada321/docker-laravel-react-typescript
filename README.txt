# ビルド
docker-compose up -d --build

# php laravel 準備
docker-compose exec php-fpm composer install
docker-compose exec php-fpm php artisan key:generate
docker-compose exec php-fpm php artisan migrate

# docker compose run echo-server laravel-echo-server init
# npmコマンド実行
docker-compose exec nodejs npm install
docker-compose exec nodejs npm add moment
docker-compose exec nodejs npm run dev

#
docker-compose exec php-fpm php artisan command:setUsageResults --collection_month=202302

# 他コマンドメモ
# コンテナ起動
docker-compose up -d

# コンテナ停止
docker-compose stop

# コンテナビルド
docker-compose build

# 
# シーダ実行
docker-compose exec php-fpm php artisan db:seed --class=MailTemplatesSeeder

# 〜〜
docker-compose exec php-fpm php artisan route:list
