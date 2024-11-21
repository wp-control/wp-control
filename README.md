![](https://komarev.com/ghpvc/?username=wp-control)

## WordPress
##### phpcs:
https://github.com/WordPress/WordPress-Coding-Standards

##### Permissions for uploads folder:
```bash
chmod -R 755 /var/www/html/wp-content/uploads
chown -R www-data:www-data /var/www/html/wp-content/uploads
```

##### CLI:
```bash
wp user generate --count=120 --role=staff
```

##### Docker Cron:
events list
```bash
sudo docker exec -it wp-cli wp cron event list --path=/var/www/html
```

execute all pending cron jobs at once
```bash
sudo docker exec -it wp-cli wp cron event run --due-now --all --path=/var/www/html
```

run specific event
```bash
sudo docker exec -it wp-cli wp cron event run name_of_the_event --path=/var/www/html
```

##### Docker configuration file:
docker-compose.yml
```yaml
version: '3.8'  
  
services:  
  wordpress:  
    image: wordpress:latest  
    container_name: wordpress  
    environment:  
      WORDPRESS_DB_HOST: mariadb:3306  
      WORDPRESS_DB_USER: root  
      WORDPRESS_DB_PASSWORD: example  
      WORDPRESS_DB_NAME: wordpress  
    volumes:  
      - ./wordpress_data/themes:/var/www/html/wp-content/themes  
      - ./wordpress_data/plugins:/var/www/html/wp-content/plugins  
    depends_on:  
      - mariadb  
      - redis  
      - rabbitmq  
    ports:  
      - "8080:80"  
    networks:  
      - app_network  
    restart: always  
  
  mariadb:  
    image: mariadb:latest  
    container_name: mariadb  
    environment:  
      MYSQL_ROOT_PASSWORD: example  
      MYSQL_DATABASE: wordpress  
      MYSQL_USER: root  
      MYSQL_PASSWORD: example  
    volumes:  
      - ./db_data:/var/lib/mysql  
    networks:  
      - app_network  
    restart: always  
  
  redis:  
    image: redis:alpine  
    container_name: redis  
    networks:  
      - app_network  
    restart: always  
  
  rabbitmq:  
    image: rabbitmq:management  
    container_name: rabbitmq  
    ports:  
      - "15672:15672" # Management UI  
      - "5672:5672"   # Default AMQP port  
    networks:  
      - app_network  
    restart: always  
  
  adminer:  
    image: adminer:latest  
    container_name: adminer  
    depends_on:  
      - mariadb  
    ports:  
      - "8081:8080"  
    networks:  
      - app_network  
    restart: always  
  
  wp-cli:  
    image: wordpress:cli  
    container_name: wp-cli  
    depends_on:  
      - wordpress  
    volumes:  
      - ./wordpress_data:/var/www/html  
    networks:  
      - app_network  
    entrypoint: /bin/sh -c 'while sleep 1000; do :; done'  
    restart: always  
  
networks:  
  app_network:  
    driver: bridge
```

Commands:
1. docker-compose up --build
2. docker-compose down
---
## Docker
```bash
sudo docker ps -a --format "table {{.ID}}\t{{.Names}}"`
```

---

## Query monitor debug
```php
do_action( 'qm/debug', 'bar' );
```

You can use any of the following actions which correspond to PSR-3 and syslog log levels:

- `qm/debug`
- `qm/info`
- `qm/notice`
- `qm/warning`
- `qm/error`
- `qm/critical`
- `qm/alert`
- `qm/emergency`

A log level of `warning` or higher will trigger a notification in Query Monitor's admin toolbar.

## Profiling[​](https://querymonitor.com/wordpress-debugging/profiling-and-logging/#profiling)

Basic profiling can be performed and displayed in the Timings panel in Query Monitor using actions in your code:

```php
// Start the 'foo' timer:
do_action( 'qm/start', 'foo' );

// Run some code
my_potentially_slow_function();

// Stop the 'foo' timer:
do_action( 'qm/stop', 'foo' );
```

The time taken and approximate memory usage used between the `qm/start` and `qm/stop` actions for the given function name will be recorded and shown in the Timings panel. Timers can be nested, although be aware that this reduces the accuracy of the memory usage calculations.

Timers can also make use of laps with the `qm/lap` action:

```php
// Start the 'bar' timer:
do_action( 'qm/start', 'bar' );

// Iterate over some data:
foreach ( range( 1, 10 ) as $i ) {
    my_potentially_slow_function( $i );
    do_action( 'qm/lap', 'bar' );
}

// Stop the 'bar' timer:
do_action( 'qm/stop', 'bar' );
```

Finally, the static logging methods on the `QM` class can be used instead of calling `do_action()`:

```php
QM::error( 'Everything is broken' );
```

