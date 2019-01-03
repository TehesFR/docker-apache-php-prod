# About this repo

Prod **PHP** optimized images with **APACHE**.

Available tags are:
- 7.3-light, latest ([7.3-light/Dockerfile](https://github.com/TehesFR/docker-apache-php-prod/blob/master/7.3-light/Dockerfile))
- 7.1-light ([7.1-light/Dockerfile](https://github.com/TehesFR/docker-apache-php-prod/blob/master/7.1-light/Dockerfile))
- 7.1 ([7.1/Dockerfile](https://github.com/TehesFR/docker-apache-php-prod/blob/master/7.1/Dockerfile))
- 5.6-light ([5.6-light/Dockerfile](https://github.com/TehesFR/docker-apache-php-prod/blob/master/5.6-light/Dockerfile))

**docker-compose example:**

```yaml
version: '3.3'
services:
  web:
    image: tehes/docker-apache-php-prod/7.3-light
    ports:
      - "8080:80"
    environment:
      # define servername in apache vhost
      - SERVERNAME=www.mydomain.xyz
      # define the domain name to use for the smtp container
      - EMAIL_DOMAIN=mydomain.xyz
      # define the outgoing default email address, if not changed in the app code
      - OUTGOING_ADDRESS=no-reply@mydomain.xyz
      # define the folder in /var/www/html where the files will be located
      - DOCUMENTROOT=htdocs
      # define the name of the smtp container (do not change)
      - SMTP=smtp
    volumes:
      - type: bind
        # this directory has to have a folder called "htdocs" with the website content
        source: /data/mydomain.xyz
        # do not change the target folder
        target: /var/www/html
    networks:
      - net
    tty: true

  # smtp container to send emails. please see readme
  smtp:
    build: tehes/docker-smtp
    # environment:
    #   - MAILNAME=outgoing_server_fqdn_name
    #   - SMARTHOST_ADDRESS=smtp_relay_ip_address
    #   - SMARTHOST_PORT=25
    networks:
      - net

networks:
  net:
    driver: overlay
```
