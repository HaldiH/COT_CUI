---
theme: default
_class: lead
paginate: true
backgroundColor: #fff
backgroundImage: url('https://marp.app/assets/hero-background.jpg')
marp: true
style: |
  p, li, pre, code, .hljs {
    font-size: 25px;
  }
---

# Docker deployment architecture

---

## Docker compose

Dockers compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application's services. Then, with a single command, you create and start all the services from your configuration.

In compose, we define the architecture of our application in a YAML file. This means that we establish what services we need, how they are connected, and how they are configured.

---

## Docker compose (2)

To start the application, we use the `docker-compose up` command in the directory containing the compose file (`{docker-}?compose.{yml,yaml}`). This command will create the resources defined in the compose file, and start the application.

When a service is modified, you just have to run `docker-compose up -d` to update the application. Docker compose will detect the changes and update the resources accordingly, without recreating the unchanged resources.

---

## Components

Compose contains these main components:

- Services: the containers that compose your app (e.g. web server, database, etc.)
- Networks: the networks that connect your services
- Volumes: the volumes that you can use to persist data
- Secrets: the secrets that you can use to store sensitive data

---

## Compose step-by-step

First, we need to define what is the main service of our application. In our example, it is Wordpress.

```yaml
version: '3.1'

services:
  wordpress:
    image: wordpress
    restart: always
    ports:
      - 8080:80
```

This is equivalent to the following command:

```bash
docker run -d -p 8080:80 --restart always wordpress
```

---

## Compose step-by-step (2)

If you try to access the application at http://localhost:8080, you will see that wordpress successfully starts, but it asks for a database.

So, we need to add a database to our application. We will use MySQL.

```yaml
services:
  wordpress: ...
  db:
    image: mysql:8.0
    restart: always
```

This will create an empty MySQL database. We need to configure Wordpress to use this database. This configuration won't works out of the box, because we haven't defined the database name, the user, and the password.

---

## Compose step-by-step (3)

To configure default database name, user, and password for MySQL, we can use environment variables.

```yaml
services:
  wordpress: ...
  db:
    ...
    environment:
      MYSQL_DATABASE: exampledb
      MYSQL_USER: exampleuser
      MYSQL_PASSWORD: examplepass
      MYSQL_RANDOM_ROOT_PASSWORD: '1'
```

Now, if we start the application, Wordpress will ask for the database name, user, and password.

Note that the two container are connected to the same network because Docker compose create a `default` network for the application. Services can communicate with each other using their name as hostname.

---

## Compose step-by-step (4)

As we don't want to enter the database name, user, and password each time we start the application, we can define them in the Wordpress configuration.

```yaml
services:
  wordpress:
    ...
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: exampleuser
      WORDPRESS_DB_PASSWORD: examplepass
      WORDPRESS_DB_NAME: exampledb
  db: ...
```

---

## Compose step-by-step (5)

Finally, we need to persist the data of the database and Wordpress. To do so, we can use volumes.

```yaml
services:
  wordpress:
    ...
    volumes:
      - wordpress:/var/www/html
  db:
    ...
    volumes:
      - db:/var/lib/mysql

volumes:
  wordpress:
  db:
```

---

### Final example

```yaml
version: '3.1'

services:

  wordpress:
    image: wordpress
    restart: always
    ports:
      - 8080:80
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: exampleuser
      WORDPRESS_DB_PASSWORD: examplepass
      WORDPRESS_DB_NAME: exampledb
    volumes:
      - wordpress:/var/www/html

  db:
    image: mysql:8.0
    restart: always
    environment:
      MYSQL_DATABASE: exampledb
      MYSQL_USER: exampleuser
      MYSQL_PASSWORD: examplepass
      MYSQL_RANDOM_ROOT_PASSWORD: '1'
    volumes:
      - db:/var/lib/mysql

volumes:
  wordpress:
  db:
```

---

By default, the resources created by Docker compose are prefixed with the name of the directory containing the compose file. To change this behavior, you can use the `-p` option.

```bash
docker-compose -p myproject up -d
```

For example, if you define a volume named `wordpress`, it will be named `myproject_wordpress` inside Docker.

---

### Services

Services defines how containers should be created and run. In the example above, we have two services: `wordpress` and `db`.

A service can be built from a Dockerfile, or from an image. In the example above, we use the `wordpress` image, and the `mysql:8.0` image.

Inside the service definition, we can define the following properties:

- `image`: the image to use
- `build`: the Dockerfile to use to build the image (instead of `image`)
- `ports`: the ports to expose
- `environment`: the environment variables to set
- `volumes`: the volumes to mount
- `networks`: the networks to connect to

See the [Services Compose file reference](https://docs.docker.com/compose/compose-file/compose-file-v3/#service-configuration-reference) for more details.

---

### 



