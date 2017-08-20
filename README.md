```
version: '2'
services:
  db:
    image: mysql
    environment:
      - MYSQL_ROOT_PASSWORD=123
  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    environment:
      - MYSQL_ROOT_PASSWORD=123
      - MYSQL_USER=root
      - MYSQL_PASSWORD=123
      - VIRTUAL_HOST=phpmyadmin.dev
  nginx-proxy:
    image: jwilder/nginx-proxy
    ports:
      - 80:80
    volumes:
      - /var/run/docker.sock:/tmp/docker.sock:ro
  project:
    image: shcoder/php-fpm-nginx
    environment:
      - VIRTUAL_HOST=project.dev
    volumes:
      - /path/to/project:/var/www
      - ./conf/default.conf:/etc/nginx/conf.d/default.conf
```

**db, nginx-proxy, phpmyadmin, project** - это имена контейнеров(сервисов) которые создаст докер, они могут быть любые. Спомощью этих имён можно обращаться с одного контейнера в другой, т.е. имя это сетевой адрес контейнера. Например: в своём приложении чтобы подключится к бд нужно написать имя базы данных т.е. db вместо localhost

**image** - это название образа который выкачивается с hub.docker.com

**environment** - Это массив окружений которые мы передаём в контейнер и они будут доступны изнутри. Так же они могут использоваться образами которые мы используем. Например выставить дефолтные пароли и тд. Обычно в репозиториях образов в редми это описано, ну или в dockerfile

**ports** - так мы пробрасываем порты наружу из контейнера порт до двоеточия это порт локально(хост) машины, после двоеточия это порт на который мы пробрасываем внутри контейнера. Если посмотреть то мы пробросили порты только у одного нашего сервиса *nginx-proxy* потому что нам нужен только доступ к nginx-у а всё остальное работает между собой в одной сети и не нуждается в пробросе. Например сервис *project* взаимодействует с *db* без проброса портов по 3306 порту, так как при запуске docker-compose создаётся общий сетевой интерфейс для всех сервисов описанных в docker-compose.yml

**volumes** - таким образом мы пробрасываем директории и файлы во внутрь нашего контейнера, тут так же как и спортами до двоеточия это файлы с локальной машины, после путь до этих файлов в контейнере, ну а то что после третьего двоеточия это флаги ro - readonly, rw - readwrite и тд... Используя volumes мы можем затирать файли и папки которые были изначально в образе который мы скачали тем самым переконфигурирую что-то или дополняя. Например у нас в сервисе *project* есть такая строчка `./conf/default.conf:/etc/nginx/conf.d/default.conf` тут мы говорим возьми наш конфиг и пробрось его во внутрь контейнера, тем самым мы затрём конфиг автора образа и получим результат который нам требуется, в данном случае дефолтный конфиг nginx-a мы затёрли конфигом заточенным под yii2 приложение.

### Чтобы начать:

1. нужeн docker
2. Нужен docker-compose
3. изменить путь до своего проекта `- /path/to/project`, `/var/www` - нужно оставить так как есть, так как он так же прописан в conf/default.conf для nginx-a
4. `docker-compose up` - чтобы запустить всё это чудо, если дописать флаг -d то всё это дело будет запущено в фоне
5. Пойти на локальной машине в /etc/hosts и добавить строчки
  ```
    127.0.0.1 phpmyadmin.dev
    127.0.0.1 project.dev
  ```
6. В браузере набрать phpmyadmin.dev зайти и создать бд ну и залить её. Так же можно воспользоваться консолькой `docker-compose exec db mysql -u root < dump.sql`
7. Пойти в файлики конфигурации бд приложения и заменить там localhost на db а так же логин и пароль
8. Перейти в браузере на project.dev и наслаждаться или если не получилось начать дебажить

**docker-compose** - это тулза которая позволяет запускать докер контейнеры пачками, она создаёт общий сетевой интерфейс и кое что ещё.

### Полезные команды docker-compose
`docker-compose up` - запускает сервисы, нужно находиться в папке с файлом docker-compose.yml

`docker-compose down` - останавливает сервисы, нужно находиться в папке с файлом docker-compose.yml

`docker-compose logs` - позволяет просмотреть все логи запущенных сервисов, если после дописать название сервиса то можно посмотреть логи только его.

`docker-compose exec имя сервиса команда` - позволяет выполнить любую команду внутри контейнера. Если написать sh то тем самым мы запустим шел и попадём внутрь контейнера

`docker-compose ps` - показывает сервисы и статусы

### Если нужно запустить больше одного проекта:

В таком случае можно скопировать блок *project* назвать его как-нибудь изменить пути и доменное имя и добавить в /etc/hosts ещё одну запись и перезапустить docker-compose.

```
version: '2'
services:
  db:
    image: mysql
    environment:
      - MYSQL_ROOT_PASSWORD=123
  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    environment:
      - MYSQL_ROOT_PASSWORD=123
      - MYSQL_USER=root
      - MYSQL_PASSWORD=123
      - VIRTUAL_HOST=phpmyadmin.dev
  nginx-proxy:
    image: jwilder/nginx-proxy
    ports:
      - 80:80
    volumes:
      - /var/run/docker.sock:/tmp/docker.sock:ro
  project:
    image: shcoder/php-fpm-nginx
    environment:
      - VIRTUAL_HOST=project.dev
    volumes:
      - /path/to/project:/var/www
      - ./conf/default.conf:/etc/nginx/conf.d/default.conf
  project1:
    image: shcoder/php-fpm-nginx
    environment:
      - VIRTUAL_HOST=project1.dev
    volumes:
      - /path/to/project1:/var/www
      - ./conf/default.conf:/etc/nginx/conf.d/default.conf
```
