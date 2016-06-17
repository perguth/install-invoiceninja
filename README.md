# Install InvoiceNinja on Uberspace

## Checkout the code

Say you want to host it under: `invoice.domain.com`:

```sh
cd $HOME/$USER
git clone https://github.com/invoiceninja/invoiceninja.git invoice.domain.com
```

Add domain to uberspace:

```sh
uberspace-add-domain -d invoice.domain.com -w
```

Setup a HTTPS certificate. See here: [Let's-Encrypt-Zertifikate](https://wiki.uberspace.de/webserver:https?s[]=encrypt#let_s-encrypt-zertifikate).

## Fix .htaccess

Within `<IfModule mod_rewrite.c>` in `.htaccess` add:

```sh
  RewriteCond %{ENV:HTTPS} !=on
  RewriteRule ^(.*) https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]

  RewriteBase /
  RewriteRule ^(.*)$ public/$1 [L]
```

Within `public/.htaccess` add:
```sh
RewriteBase /
```

## Update PHP
```sh
echo "PHPVERSION=7.0" > ~/etc/phpversion
killall php-cgi
```

## Install composer
```sh
# https://getcomposer.org/download/
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('SHA384', 'composer-setup.php') === '070854512ef404f16bac87071a6db9fd9721da1684cd4589b1196c3faf71b9a2682e2311b36a5079825e155ac7ce150d') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
mkdir ~/.composer
php composer-setup.php --install-dir=~/.composer
php -r "unlink('composer-setup.php');"
```

Create a shellscript for composer in `~/bin/composer`:
```sh
#/bin/bash
php ~/.composer/composer.phar "$@"
```

`chmod +x ~/bin/composer`

## Install dependencies
```sh
composer install
```

## Crontab
```sh
0 8 * * * /usr/local/bin/php /path/to/ninja/artisan ninja:send-invoices
0 8 * * * /usr/local/bin/php /path/to/ninja/artisan ninja:send-reminders
```

# Configure InvoiceNinja

In `config/app.php` set `APP_URL` to `https://invoice.domain.com` and APP_KEY to a [32 character long random string](https://www.random.org/strings/?num=2&len=16&digits=on&upperalpha=on&loweralpha=on&unique=on&format=html&rnd=new).

# Configure MySQL

See here: [](https://wiki.uberspace.de/database:mysql?s[]=mysql).

# Finish installation

Perfom the final steps via the frontend of InvoiceNinja: https://invoice.domain.com.

--
:rocket:
