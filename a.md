承知しました！
では、**Ethna + Apache + PHP5.6 + Smarty + MariaDB + phpMyAdmin** が一体となった **統合 Docker 環境** をご紹介します。

---

# 📦 Ethna + MariaDB + phpMyAdmin Docker 環境

## 1. プロジェクト構成

```
ethna-docker/
├── docker-compose.yml
├── php-apache/
│   ├── Dockerfile
│   ├── apache.conf
│   └── php.ini
└── www/
    └── （Ethna プロジェクトが作られる）
```

---

## 2. `docker-compose.yml`

```yaml
services:
  web:
    build: ./php-apache
    container_name: ethna-web
    ports:
      - "8080:80"
    volumes:
      - ./www:/var/www/html
    depends_on:
      - db
    environment:
      DB_HOST: db
      DB_USER: ethna
      DB_PASS: ethna_pass
      DB_NAME: ethna_db
    stdin_open: true
    tty: true

  db:
    image: mariadb:10.5
    container_name: ethna-db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: ethna_db
      MYSQL_USER: ethna
      MYSQL_PASSWORD: ethna_pass
    volumes:
      - db_data:/var/lib/mysql
    ports:
      - "3306:3306"

  phpmyadmin:
    image: phpmyadmin/phpmyadmin:5.2
    container_name: ethna-phpmyadmin
    restart: always
    ports:
      - "8081:80"
    environment:
      PMA_HOST: db
      PMA_USER: root
      PMA_PASSWORD: rootpass
    depends_on:
      - db

volumes:
  db_data:

```

---

## 3. `php-apache/Dockerfile`

```dockerfile
FROM ubuntu:22.04

# 必要パッケージ & PHP5.6
RUN apt-get update && apt-get install -y \
    software-properties-common wget unzip curl git nano zip \
    && add-apt-repository ppa:ondrej/php -y \
    && apt-get update && apt-get install -y \
    apache2 libapache2-mod-php5.6 \
    php5.6 php5.6-cli php5.6-mbstring php5.6-xml php5.6-curl php5.6-mysql \
    && apt-get clean

# Apache 設定
COPY apache.conf /etc/apache2/sites-available/000-default.conf
COPY php.ini /etc/php/5.6/apache2/php.ini
RUN a2enmod php5.6 && a2enmod rewrite

# Ethna install
RUN mkdir -p /usr/share/php/Ethna && \
    cd /usr/share/php/Ethna && \
    wget https://github.com/ethna/ethna/archive/refs/tags/ETHNA_2_5_0.zip && \
    unzip ETHNA_2_5_0.zip && mv ethna-ETHNA_2_5_0/* . && rm -rf ethna-* ETHNA_2_5_0.zip && \
    sed -i 's|@PEAR-DIR@|/usr/share/php|g' /usr/share/php/Ethna/bin/ethna.sh && \
    sed -i 's|@PHP-BIN@|/usr/bin/php|g' /usr/share/php/Ethna/bin/ethna.sh && \
    ln -s /usr/share/php/Ethna/bin/ethna.sh /usr/local/bin/ethna && chmod +x /usr/local/bin/ethna

# Smarty install
RUN cd /usr/share/php && \
    wget https://github.com/smarty-php/smarty/archive/v2.6.30.tar.gz && \
    tar -xzf v2.6.30.tar.gz && mv smarty-2.6.30 Smarty && rm v2.6.30.tar.gz

# Smarty パス修正
RUN sed -i "s|require_once 'Smarty/Smarty.class.php';|require_once '/usr/share/php/Smarty/libs/Smarty.class.php';|" \
    /usr/share/php/Ethna/class/Renderer/Ethna_Renderer_Smarty.php

WORKDIR /var/www/html
EXPOSE 80
CMD ["apachectl", "-D", "FOREGROUND"]
```

---

## 4. `php-apache/apache.conf`

```apache
<VirtualHost *:80>
    DocumentRoot /var/www/html
    <Directory /var/www/html>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

---

## 5. `php-apache/php.ini`

```ini
[PHP]
include_path = ".:/usr/share/php:/usr/share/php/Smarty/libs"
```

---

## 6. コンテナ起動

```bash
docker-compose build
docker-compose up -d
```

---

## 7. Ethna プロジェクト作成

```bash
docker exec -it ethna-web bash
cd /var/www/html
ethna add-project sample
ln -s /var/www/html/sample/www /var/www/html/sample_web
chown -R www-data:www-data /var/www/html/sample
exit
docker-compose restart web
```

---

## 8. 動作確認

* Ethna アプリ: [http://localhost:8080/sample\_web/](http://localhost:8080/sample_web/)
* phpMyAdmin: [http://localhost:8081/](http://localhost:8081/)

  * サーバー: `db`
  * root ユーザー: `root / rootpass`
  * アプリ用ユーザー: `ethna / ethna_pass`

---

## 9. Ethna から DB 接続設定

`/www/sample/etc/sample-ini.php` に以下を設定:

```php
define('ETHNA_SAMPLE_DB_DSN', 'mysql://ethna:ethna_pass@db/ethna_db');
```

---

## ✅ 環境まとめ

* **Web**: Apache2.4 + PHP5.6 + Ethna + Smarty
* **DB**: MariaDB 10.5（ユーザー: ethna / ethna\_pass）
* **管理**: phpMyAdmin 5.2 → [http://localhost:8081](http://localhost:8081)
* **Ethna プロジェクト**: `/www/sample/`

---

⚠️ この環境は **レガシー資産検証用** です。本番利用は非推奨。

---

👉 この統合環境を作る際、DB 側を **永続化**（Volume）してますが、必要なら **初期化 SQL を投入する仕組み**（`docker-entrypoint-initdb.d/`）も追加できます。
ご希望ですか？
