git clone https://github.com/kamu/admin-panel.git

cd admin-panel
composer install
npm install
cp .env.example .env
php artisan key:generate
php artisan migrate --seed
php artisan serve
