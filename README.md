
# Hosting Websites 

Docker base hosting uses 5 images: Apache web server, MariaDB database, Adminer a web GUI to manage database, postfix mail server, and Roundcube a web gui to manage emails.

```sh
version: '3'
services:
  apache:
    image: php:7.4-apache
    restart: always
    ports:
      - 80:80
      - 443:443
    volumes:
      - /root/hosting/websites/apache-configs:/etc/apache2 [1]
      - /root/hosting/websites/files:/var/www [2]
      - /root/hosting/websites/php-configs:/usr/local/etc/php [3]
  database:
    image: mariadb:latest
    restart: always
    environment:
      MARIADB_ROOT_PASSWORD: DATABASE_PASS [4]
    volumes:
      - /root/hosting/database/files:/var/lib/mysql [5]
      - /root/hosting/database/configs:/etc/mysql/conf.d [6]
  adminer:
    image: adminer
    restart: always
    environment:
      ADMINER_DEFAULT_SERVER: database
    ports:
      - 8080:8080
  mailserver:
    image: h0perium/postfix-dovecot-dkim
    restart: always
    ports:
      - 25:25
      - 993:993
      - 995:995
      - 587:587
      - 143:143
      - 110:110
    volumes:
      - mailserver-storage:/var/mail
    environment:
      MTP_HOST: webiox.net [7]
      INIT_EMAIL: info@webiox.net
      INIT_EMAIL_PASS: DATABASE_PASS [8]
      MTP_DESTINATION: '6a6a6586e71d, localhost.localdomain, localhost'
  roundcube:
    depends_on:
      - database
    image: h0perium/roundcube
    environment:
      ROUNDCUBEMAIL_DEFAULT_HOST: "mailserver"
      ROUNDCUBEMAIL_SMTP_SERVER:  "mailserver"
      ROUNDCUBEMAIL_SMTP_PORT: 25
      ROUNDCUBEMAIL_DB_TYPE: mysql
      ROUNDCUBEMAIL_DB_HOST: database
      ROUNDCUBEMAIL_DB_USER: root
      ROUNDCUBEMAIL_DB_PASSWORD: DATABASE_PASS [9]
      ROUNDCUBEMAIL_DB_NAME: admin

    ports:
      - 8880:80
    restart: always
volumes:
   mailserver-storage:

```


## Configs

[1] = Path for apache config

[2] = Path for virtual host files for each website

[3] = Path for php config

[4] , [8] , [9] = Password for database

[5] = Path for database file that you can backup

[6] = Path for mariaDB config

[7] = Your Domain


## More info about mailserver configs

[postfix-dovecot-dkim](https://github.com/h0perium/postfix-dovecot-dkim) - DKIM And Other Configs

## Run Command
Change directory to docker folder and then run :

```sh
docker compose up -d
```


