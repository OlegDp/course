Ми створюємо docker-compose.yml , який би дозволив передавати credentials (login/password) безпечно — не вказуючи їх текстом в файлі compose. Бо паролі не варто зберігати в plain text незашифрованих файлах.
1. За допомогою entrypoint скриптів додаємо контейнерам можливість брати значення для змінних з файлів образу. Замість MYSQL_ROOT_PASSWORD зі значенням паролю root вказуємо MYSQL_ROOT_PASSWORD_FILE та вказуємо файл, з якого брати пароль. Таким же чином вказуємо дані, з якого файлу брати дані для MYSQL_USER, MYSQL_PASSWORD, WORDPRESS_DB_USER, WORDPRESS_DB_PASSWORD тільки вже з додаванням _FILE відповідно.

В secrets можемо побачити "external: true" для кожного значення. Якщо встановлено значення true, це означає, що цю конфігурацію вже створено (пункт 3). Docker не намагається створити його.

2. Запустили сам Swarm (Initialize a swarm) на початку (так як в моєму випадку команда була виконана раніше тому отримуємо відповідь про це):

$ docker swarm init

Error response from daemon: This node is already part of a swarm. Use "docker swarm leave" to leave this swarm and join another one.

3. Створюємо секрети для db_user, db_password та db_root_password:

$ printf "ім'я користувача" | docker secret create db_user -

$ printf "пароль" | docker secret create db_password -

$ printf "пароль для root" | docker secret create db_root_password -

4. Перевіряємо чи наші паролі та ім'я користувача стали secret, командою (бачимо що все спрацювало так як у виводі ми побачимо ID, NAME, CREATED and UPDATED для наших файлів): 

$ docker secret ls

5. Compose файл деплоїмо командою та отримуємо вивід про створення самого wordpress та можемо побачити, що тепер відсутнє при виводі creating secret, так як docker secret був створений до цього.

$ docker stack deploy -c docker-compose.yml wordpress

Ignoring unsupported options: restart

Creating network wordpress_default
Creating service wordpress_db
Creating service wordpress_wordpress

6. Перевіряємо, чи створені контейнери mysql та wordpress:

$ docker ps

CONTAINER ID           IMAGE              COMMAND                  CREATED          STATUS          PORTS                 NAMES
16fff5fc08a1   wordpress:latest   "docker-entrypoint.s…"   54 seconds ago   Up 51 seconds   80/tcp                wordpress_wordpress.1.ixjm3s1ys0j31cdpivj8jkf6x
93f3da0ad050   mysql:latest       "docker-entrypoint.s…"   3 minutes ago    Up 3 minutes    3306/tcp, 33060/tcp   wordpress_db.1.2cjjnfrj8xq8qiirl42mkezom

7. Через браузер заходимо по вказаному нами порту (8080) в docker-compose.yml і бачимо що все працює.