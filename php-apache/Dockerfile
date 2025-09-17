FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive TZ=Asia/Tokyo

# 必要パッケージ & PHP5.6
RUN apt-get update && apt-get install -y \
    software-properties-common wget unzip curl git nano zip vim \
    && add-apt-repository ppa:ondrej/php -y \
    && apt-get update && apt-get install -y \
    apache2 libapache2-mod-php5.6 \
    php5.6 php5.6-cli php5.6-mbstring php5.6-xml php5.6-curl php5.6-mysql \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

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
    sed -i 's|@PHP-BINARY@|/usr/bin/php|g' /usr/share/php/Ethna/bin/ethna.sh && \
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
