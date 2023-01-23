Ми створюємо docker-compose.yml , який би дозволив передавати credentials (login/password) безпечно — не вказуючи їх текстом в файлі compose. Бо паролі не варто зберігати в plain text незашифрованих файлах.
1. Pа допомогою entrypoint скриптів додаємо контейнерам можливість брати значення для змінних з файлів.Замість MYSQL_ROOT_PASSWORD зі значенням паролю root вказуємо MYSQL_ROOT_PASSWORD_FILE та вказуємо файл (попередньо створений), з якого брати пароль. Таким же чином вказуємо дані, з якого файлу брати дані для MYSQL_USER, MYSQL_PASSWORD, WORDPRESS_DB_USER, WORDPRESS_DB_PASSWORD тільки вже з додаванням _FILE відповідно.

В кінці файлу docker-compose.yml прописуємо примінення secrets для db_user.txt, db_password.txt та db_root_password.txt (або ж як ви їх назвете у своєму docker-compose.yml)

2. Запустили сам Swarm (Initialize a swarm) на початку (так як в моєму випадку команда була виконана раніше тому отримуємо відповідь про це):

$ docker swarm init

Error response from daemon: This node is already part of a swarm. Use "docker swarm leave" to leave this swarm and join another one.

3. Compose файл деплоїмо командою та отримуємо вивід про створення secret для наших логіну User та паролів, шлях до яких ми вказали в нашому файлі .yml, а також бачимо вивід про створення самого wordpress:

$ docker stack deploy -c docker-compose.yml wordpress

Ignoring unsupported options: restart
Creating secret wordpress_db_user
Creating secret wordpress_db_password
Creating secret wordpress_db_root_password
Creating service wordpress_db
Creating service wordpress_wordpress

4. Перевіряємо чи наші паролі та ім'я користувача стали secret, командою (бачимо що все спрацювало): 

$ docker secret ls

ID                          NAME                         DRIVER    CREATED          UPDATED
idn010s0q1ho2futbeuba9xx1   wordpress_db_password                  21 seconds ago   21 seconds ago
x3pj3669r8sexntxzw7owg6o6   wordpress_db_root_password             21 seconds ago   21 seconds ago
lufwg5sm4hjnvxr1v2bjuxwr0   wordpress_db_user                      21 seconds ago   21 seconds ago

5. Перевіряємо, чи створені контейнери mysql та wordpress:

$ docker ps

CONTAINER ID           IMAGE              COMMAND                  CREATED          STATUS          PORTS                 NAMES
16fff5fc08a1   wordpress:latest   "docker-entrypoint.s…"   54 seconds ago   Up 51 seconds   80/tcp                wordpress_wordpress.1.ixjm3s1ys0j31cdpivj8jkf6x
93f3da0ad050   mysql:latest       "docker-entrypoint.s…"   3 minutes ago    Up 3 minutes    3306/tcp, 33060/tcp   wordpress_db.1.2cjjnfrj8xq8qiirl42mkezom
