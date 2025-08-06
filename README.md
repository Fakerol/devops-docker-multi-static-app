# Deploy Multiple Web Applications Using Docker Compose

This guide outlines the steps to deploy two web applications (`mosos` and `festava`) in a single Apache container using Docker Compose. The applications are accessible via distinct URL paths on a single server.

## Folder Structure
```
project-root/
│
├── docker-compose.yml
├── apache/
│   ├── Dockerfile
│   ├── 000-default.conf
│   ├── mosos.tar.gz
│   └── festava.tar.gz
```

## Prerequisites
- A Linux system with `dnf` package manager (e.g., CentOS, RHEL, or Fedora).
- Docker and Docker Compose installed.
- Root or sudo privileges.
- The `mosos.tar.gz` and `festava.tar.gz` archive files containing the application code.
- Port `8080` available on the host machine.

## Installation Steps

### 1. Install Docker Compose
Install Docker Compose on the host machine.

```bash
sudo dnf install docker-compose -y
```

Ensure Docker is installed and running:

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

### 2. Create Apache Dockerfile
Create the file `apache/Dockerfile` and add the following configuration:

```dockerfile
FROM ubuntu:latest

LABEL Author="user1"
LABEL Project="multi-site"

RUN apt update && apt install -y apache2 tar

# Create folders for both apps
WORKDIR /var/www/html

ADD mosos.tar.gz /var/www/html/mosos
ADD festava.tar.gz /var/www/html/festava

# Replace default Apache config with multi-site config
COPY 000-default.conf /etc/apache2/sites-available/000-default.conf

EXPOSE 80

CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

*Note*: Replace `user1` with your actual name or a generic identifier.

### 3. Configure Apache Virtual Host
Create the file `apache/000-default.conf` and add the following configuration:

```apache
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html

    Alias /mosos /var/www/html/mosos
    <Directory /var/www/html/mosos>
        Require all granted
    </Directory>

    Alias /festava /var/www/html/festava
    <Directory /var/www/html/festava>
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

### 4. Create Docker Compose Configuration
Create the file `docker-compose.yml` in the `project-root/` directory and add the following:

```yaml
version: "3.8"

services:
  apache:
    build: ./apache
    container_name: apache-server
    ports:
      - "8080:80"
    volumes:
      - apache_logs:/var/log/apache2

volumes:
  apache_logs:
```

### 5. Build and Start Containers
Navigate to the `project-root/` directory and run:

```bash
docker-compose up --build -d
```

This command builds the Apache image and starts the container in detached mode.

### 6. Access the Applications
Once the container is running, access the applications at:

- `http://192.0.2.100:8080/mosos/`
- `http://192.0.2.100:8080/festava/`

*Note*: Replace `192.0.2.100` with your server's actual IP address.

## Notes
- Ensure the `mosos.tar.gz` and `festava.tar.gz` files are valid archives containing the application code.
- Verify that port `8080` is open in your firewall if applicable (e.g., `sudo firewall-cmd --permanent --add-port=8080/tcp && sudo firewall-cmd --reload`).
- Check container logs for debugging: `docker logs apache-server`.
- To stop the containers, run: `docker-compose down`.
- For production, consider adding SSL/TLS for secure access and adjust file permissions as needed.
- Replace `192.0.2.100` with your server's actual IP address.
