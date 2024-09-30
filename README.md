# Dockerized Web, Database, and Mail Server Setup
The Docker topology for your personal hosting setup involves using Nginx as a reverse proxy and Apache to serve customer requests. You'll have a Docker host running both an Nginx container, configured to listen on ports 80 and 443, directing traffic to an Apache container operating on an internal port 80

This repository contains a Docker Compose file that sets up a web stack consisting of:

- Nginx reverse proxy
- Apache servers for two websites
- MariaDB database
- Adminer for database management
- Postfix/Dovecot mail server with DKIM support
- Roundcube for webmail access

## Prerequisites

- [Docker](https://www.docker.com/get-started)
- [Docker Compose](https://docs.docker.com/compose/install/)
- [h0perium/postfix-dovecot-dkim Details](https://github.com/h0perium/postfix-dovecot-dkim)

## Getting Started

1. **Clone the repository:**
   ```bash
   git clone https://github.com/mehransalehi/hosting.git
   cd hosting
1. **Update environment variables:** 
The services for MariaDB, Mailserver, and Roundcube use environment variables for configuration. Create a .env file at the root of the project and include the following variables:
   ```bash
    MYSQL_ROOT_PASSWORD=your_root_password
    MYSQL_PASSWORD=your_db_password
    INIT_EMAIL_PASSWORD=your_email_password
2. **Run the services:**
After setting up the environment variables, run:
    ```bash
    docker-compose up -d
This will start all the services in detached mode.

# Services Overview

This project sets up a multi-service application using Docker, including an Nginx reverse proxy, multiple Apache servers for hosting websites, a MariaDB database, a mail server, and a webmail client.

## Table of Contents
- [Nginx Reverse Proxy](#nginx-reverse-proxy)
- [Apache Servers](#apache-servers)
  - [Apache for Domain A (apache1)](#apache-for-domain-a-apache1)
  - [Apache for Domain B (apache2)](#apache-for-domain-b-apache2)
- [MariaDB Database](#mariadb-database)
- [Adminer (Database Management)](#adminer-database-management)
- [Mail Server (Postfix/Dovecot)](#mail-server-postfixdovecot)
- [Roundcube (Webmail)](#roundcube-webmail)
- [Customization Options](#customization-options)
- [Networking](#networking)
- [Volumes](#volumes)
- [Troubleshooting](#troubleshooting)

## Nginx Reverse Proxy

- **Image:** `nginx:latest`
- **Ports:** 
  - HTTP: `80:80`
  - HTTPS: `443:443`
- **Volumes:**
  - `./nginx/nginx.conf:/etc/nginx/nginx.conf:ro`: Customize the Nginx configuration here.
  - `./nginx/certs:/etc/nginx/certs:ro`: SSL certificates for HTTPS.
  - `./nginx/html:/usr/share/nginx/html`: Default directory for serving static files.

### Customization
- Edit `nginx/nginx.conf` to modify the reverse proxy rules.
- Place SSL certificates in `nginx/certs` for HTTPS support.

## Apache Servers

### Apache for Domain A (apache1)
- **Image:** `php:7.4-apache`
- **Volumes:**
  - `./websites/domain-a/html:/var/www/html`: Website files for Domain A.
  - `./websites/domain-a/apache-configs:/etc/apache2`: Apache configuration for Domain A.
  - `./websites/domain-a/php-configs:/usr/local/etc/php`: PHP configuration.

### Customization
- Edit files in `websites/domain-a/html` for your website's content.
- Modify Apache and PHP configurations as needed.

### Apache for Domain B (apache2)
- **Image:** `php:7.4-apache`
- **Volumes:**
  - `./websites/domain-b/html:/var/www/html`: Website files for Domain B.
  - `./websites/domain-b/apache-configs:/etc/apache2`: Apache configuration for Domain B.
  - `./websites/domain-b/php-configs:/usr/local/etc/php`: PHP configuration.

### Customization
- Edit files in `websites/domain-b/html` for your website's content.
- Modify Apache and PHP configurations as needed.

## MariaDB Database

- **Image:** `mariadb:latest`
- **Ports:** Not exposed outside the Docker network.
- **Volumes:**
  - `./database/files:/var/lib/mysql`: Persistent storage for database files.
  - `./database/configs:/etc/mysql/conf.d`: Custom database configuration.
- **Environment Variables:**
  - `MYSQL_ROOT_PASSWORD`: Root password (from `.env` file).
  - `MYSQL_DATABASE`: The default database to create (e.g., `mydatabase`).
  - `MYSQL_USER`: The default user (e.g., `myuser`).
  - `MYSQL_PASSWORD`: Password for `myuser` (from `.env` file).

### Customization
- Modify `MYSQL_DATABASE`, `MYSQL_USER`, and `MYSQL_PASSWORD` in the Compose file to customize the database and users.
- Place custom MySQL configuration in `database/configs`.

## Adminer (Database Management)

- **Image:** `adminer:latest`
- **Ports:** `8080:8080`
- **Access:** Adminer is accessible at [http://localhost:8080](http://localhost:8080).

### Customization
- No major customization needed; adjust the port if necessary.

## Mail Server (Postfix/Dovecot)

- **Image:** `h0perium/postfix-dovecot-dkim`
- **Ports:**
  - SMTP: `25:25`
  - SMTP Submission: `587:587`
  - IMAP: `143:143`
  - IMAP over SSL: `993:993`
  - POP3: `110:110`
  - POP3 over SSL: `995:995`
- **Volumes:**
  - `mailserver-storage:/var/mail`: Persistent storage for mail data.
- **Environment Variables:**
  - `MTP_HOST`: The mail server hostname (e.g., `domain-a.net`).
  - `INIT_EMAIL`: Initial email address (e.g., `info@domain-a.net`).
  - `INIT_EMAIL_PASS`: Password for the initial email (from `.env` file).
  - `MTP_DESTINATION`: Domains the mail server should accept mail for.

### Customization
- Modify `MTP_HOST`, `INIT_EMAIL`, and `MTP_DESTINATION` fields based on your domain setup.
- Set `INIT_EMAIL_PASS` securely via the `.env` file.

## Roundcube (Webmail)

- **Image:** `h0perium/roundcube`
- **Ports:** `8880:80`
- **Depends on:** `database`, `mailserver`
- **Access:** Roundcube is accessible at [http://localhost:8880](http://localhost:8880).
- **Environment Variables:**
  - `ROUNDCUBEMAIL_DEFAULT_HOST`: The mail server hostname (e.g., `mailserver`).
  - `ROUNDCUBEMAIL_SMTP_SERVER`: SMTP server (e.g., `mailserver`).
  - `ROUNDCUBEMAIL_DB_HOST`: Database host (e.g., `database`).
  - `ROUNDCUBEMAIL_DB_USER`: Database user (e.g., `root`).
  - `ROUNDCUBEMAIL_DB_PASSWORD`: Database root password (from `.env`).
  - `ROUNDCUBEMAIL_DB_NAME`: Roundcube's database name (e.g., `admin`).

### Customization
- Update the environment variables if you're using different database credentials.

## Customization Options

### Environment Variables
- Define `MYSQL_ROOT_PASSWORD`, `MYSQL_PASSWORD`, and `INIT_EMAIL_PASSWORD` in the `.env` file.

### Volume Mapping
- Map different paths in the volumes section to customize website files, Nginx configuration, SSL certificates, mail server storage, and database files.

### Network Ports
- Customize the exposed ports for each service if they conflict with other services on your host machine.

## Networking

All services are connected via a Docker bridge network (`proxy-network`), which allows them to communicate internally without exposing unnecessary ports externally.

## Volumes

- `mailserver-storage`: Persistent storage for the mail server.

## Troubleshooting

- Use `docker-compose logs [service-name]` to view logs for any service.
- Use `docker-compose down` to stop and remove the containers, or `docker-compose restart` to restart them.

