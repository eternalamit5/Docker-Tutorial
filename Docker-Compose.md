# Docker Compose

Docker compose is a CLI tool that uses YAML-formatted file describing

- Containers
- networks
- volumes

allowing to configure relationships between containers, and saving docker container run settings in easy-to-read file. Thus providing one-liner developer environment startups.

Let's talk about docker compose yaml file

- compose yaml format has it's own versions: 1,2,2.1,3,,3.1

- YAML file can be used with docker-compose command for local docker automation.

- By default docker-compose CLI command will look for filename "docker-compose.yml", but if we are using different filename then we need to add option -f  .

  ```bash
  docker-compose -f <docker-file-name=docker-compose.yml>
  ```

##### Template for docker-compose file

```yaml
version: '3.1'  # if no version is specified then v1 is assumed. Recommend v2 minimum

services:  # containers. same as docker run
  servicename: # a friendly name. this is also DNS name inside network
    image: # Optional if you use build:
    command: # Optional, replace the default CMD specified by the image
    environment: # Optional, same as -e in docker run
    volumes: # Optional, same as -v in docker run
  servicename2:

volumes: # Optional, same as docker volume create

networks: # Optional, same as docker network create

```

### docker-compose CLI 

- docker-compose in Linux need to be downloaded separately
- docker-compose is not production-grade tool but ideal for local development and test
- docker-compose automatically create network even without manually specifying

Two step on-boarding process for the project

- git clone <git-repo-url>
- docker-compose -f <docker-filename> up

*******

:bell: 

 **port** in **docker-compose**  and **EXPOSE**  in **Dockerfile** are not same. **EXPOSE** does not publish the port, but **port** publishes the port

****

### Example for docker-compose files

##### Example-1

- jekyll docker-compose file

```yaml
version: '3'

# same as 
# docker run -p 80:4000 -v $(pwd):/site bretfisher/jekyll-serve

services:
  jekyll:
    image: bretfisher/jekyll-serve
    volumes:
      - .:/site
    ports:
      - '80:4000'

```

##### Example-2

- Wordpress setup

```yaml
version: '3'

services:

  wordpress:
    image: wordpress
    ports:
      - 8080:80
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_NAME: wordpress
      WORDPRESS_DB_USER: example
      WORDPRESS_DB_PASSWORD: examplePW
    volumes:
      - ./wordpress-data:/var/www/html

  mysql:
    image: mariadb
    environment:
      MYSQL_ROOT_PASSWORD: examplerootPW
      MYSQL_DATABASE: wordpress
      MYSQL_USER: example
      MYSQL_PASSWORD: examplePW
    volumes:
      - mysql-data:/var/lib/mysql

volumes:
  mysql-data:
```

##### 	Example-3

- 3 database Cluster behind ghost server

```yaml
version: '3'

services:
  ghost:
    image: ghost
    ports:
      - "80:2368"
    environment:
      - URL=http://localhost
      - NODE_ENV=production
      - MYSQL_HOST=mysql-primary
      - MYSQL_PASSWORD=mypass
      - MYSQL_DATABASE=ghost
    volumes:
      - ./config.js:/var/lib/ghost/config.js
    depends_on:
      - mysql-primary
      - mysql-secondary
  proxysql:
    image: percona/proxysql
    environment: 
      - CLUSTER_NAME=mycluster
      - CLUSTER_JOIN=mysql-primary,mysql-secondary
      - MYSQL_ROOT_PASSWORD=mypass
   
      - MYSQL_PROXY_USER=proxyuser
      - MYSQL_PROXY_PASSWORD=s3cret
  mysql-primary:
    image: percona/percona-xtradb-cluster:5.7
    environment: 
      - CLUSTER_NAME=mycluster
      - MYSQL_ROOT_PASSWORD=mypass
      - MYSQL_DATABASE=ghost
      - MYSQL_PROXY_USER=proxyuser
      - MYSQL_PROXY_PASSWORD=s3cret
  mysql-secondary:
    image: percona/percona-xtradb-cluster:5.7
    environment: 
      - CLUSTER_NAME=mycluster
      - MYSQL_ROOT_PASSWORD=mypass
   
      - CLUSTER_JOIN=mysql-primary
      - MYSQL_PROXY_USER=proxyuser
      - MYSQL_PROXY_PASSWORD=s3cret
    depends_on:
      - mysql-primary
```



##### Example-4  

- Influxdb, mosquitto, telegraph, grafana, mongo stack

```yaml
version: "3"

services:
    influxdb:
        image: influxdb
        container_name: influxdb
        env_file: prototype.env
        ports: 
            - "8086:8086"
        volumes: 
            - ./influxdb:/var/lib/influxdb
        networks: 
            - "iotstack"

    mosquitto:
        image: eclipse-mosquitto
        container_name: mosquitto
        volumes: 
            - ./mosquitto/config/:/mosquitto/config/
            - ./mosquitto/log:/mosquitto/log
            - ./mosquitto/data:/mosquitto/data
        user: "1000:1000"
        ports:
            - "1887:1883"
        networks: 
            - "iotstack"
    
    telegraf:
        image: telegraf
        container_name: telegraf
        env_file: prototype.env
        volumes: 
            - ./telegraf/telegraf.conf:/etc/telegraf/telegraf.conf:ro
        networks: 
            - "iotstack"


    grafana:
        image: grafana/grafana
        container_name: grafana
        ports: 
            - "3005:3000"
        env_file: prototype.env
        volumes: 
            - ./grafana/data:/var/lib/grafana
        networks: 
            - "iotstack"
                                           

    mongo:
        image: mongo
        container_name: mongo
        env_file: prototype.env
        ports: 
            - "27017:27017"
        volumes: 
            - ./mongo/data/db:/data/db
        networks: 
            - "iotstack"
                  
                  
volumes: 
    influxdb:
    grafana:

networks: 
    iotstack:
```

##### Example-5

- Adding image to compose file

```yaml
version: '3'

services:
  proxy:
    build:
      context: .
      dockerfile: nginx.Dockerfile
    ports:
      - '80:80'
  web:
    image: httpd
    volumes:
      - ./html:/usr/local/apache2/htdocs/
```

:octopus: Build and up the containers in single command

```
docker-compose up --build 
```

:octopus: Build and up the containers separately

```bash
# build the image
docker-compose build

#up the containers
docker-compose up
```



##### Example-8

- Adding image to compose file

```yaml
version: '2'
# NOTE: move this answer file up a directory so it'll work

services:

  drupal:
    image: custom-drupal
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:80"
    volumes:
      - drupal-modules:/var/www/html/modules
      - drupal-profiles:/var/www/html/profiles       
      - drupal-sites:/var/www/html/sites      
      - drupal-themes:/var/www/html/themes
 
  postgres:
    image: postgres:12.1
    environment:
      - POSTGRES_PASSWORD=mypasswd
    volumes:
      - drupal-data:/var/lib/postgresql/data

volumes:
  drupal-data:
  drupal-modules:
  drupal-profiles:
  drupal-sites:
  drupal-themes:
```

```bash
docker-compose up --build
```

