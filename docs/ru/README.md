## Мессенджер Одуванчик

### 📝 Описание проекта
Клиент-серверное приложение-мессенджер. Разработано в **учебных целях** и в качестве **портфолио**.  
Исходники закрыты. Бинарники доступны для свободного скачивания и развертывания.

### 🏗️ Функционал
*   **Чаты пользователей:** Чаты на 2 и более пользователей.
*   **Реакции и комментарии сообщений**
*   **Отслеживание комментариев:** Отслеживание 500 последних сообщений, на которые пользователь оставил комментарий.
*   **Без открытой регистрации:** Учетные записи пользователей создаются исключительно администратором системы.
*   **Мягкое удаление (Soft Delete):** При удалении сообщений, чатов, реакций, они не стираются физически из системы. Информация помечается флагом `удалено` в базе данных и сохраняется для обеспечения соответствия применимому законодательству.

### ⚠️ Юридическое предупреждение
> **ВАЖНО:** Перед введением в эксплуатацию мессенджера настоятельно рекомендуется изучить и строго соблюдать **законодательство своей страны и местные нормативно-правовые акты, регулирующие работу мессенджеров и цифровые коммуникации**. В зависимости от вашей юрисдикции, запуск или использование независимых платформ обмена сообщениями может требовать соблюдения законов о хранении данных, лицензировании или обязательной идентификации пользователей. Разработчики не несут ответственности за неправомерное использование ПО.

*   ### 📄 **Клиент**

*   ### 📄 **Сервер**

#### Ключевые Особенности
*   **Пакет установки:** fat-jar на jvm 21 или docker-image `zzhekaa/dandelionserver:latest`. 
*   **БД:** Mariadb. Протестировано на `10.11.14-MariaDB`
*   **Система оповещения мобильных клиентов:** Google Firebase.

### Настройка и запуск сервера

#### 1. Установить и настроить Mariadb  

Установить Mariadb, создать пустую БД и пользователя с правами доступа к ней.  
Если работа с БД будет осуществляться из docker контейнера с фиксированным IP (в примере `172.18.0.100`) иметь это в виду при настройке прав.

#### 2. Подготовить проект Google Firebase

2.1 Подготовить параметры Google Firebase для Android и IOS, которые позже будут использованы в п.4. См. документацию в интернете.  

IOS App Store ID: `6788429394`  
IOS Bundle ID: `zh.dandelion.client`  
IOS Team ID: `DC7N64S349`  
  
Android package name: `zh.dandelion.client.full`


2.2 Скачать `Firebase Server Private Key` в доступную для сервера директорию (например в директорию с конфигурационным файлом `settings.json` из п.4).

#### 3. Настроить Apache Reverse Proxy сайт для сервера

Зарегистрировать домен для вашего мессенджера, например `dandelion-test.com`.  

Пример Apache Reverse Proxy конфигурации `/etc/apache2/sites-available/dandelion-test.com.conf` (для порта `8085` сервера, домена `dandelion-test.com` с `letsencrypt` сертификатом):
```
<IfModule mod_ssl.c>
    <VirtualHost *:443>
        ServerName dandelion-test.com

        SSLEngine on

        RewriteEngine On
        RewriteCond %{HTTP:Upgrade} =websocket [NC] 
        RewriteRule ^/(.*) ws://127.0.0.1:8085/$1 [P,L]

        ProxyPreserveHost On
        ProxyPass / http://127.0.0.1:8085/
        ProxyPassReverse / http://127.0.0.1:8085/

        RequestHeader set X-Forwarded-Proto "https"
        
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
        Include /etc/letsencrypt/options-ssl-apache.conf

        SSLCertificateFile /etc/letsencrypt/live/dandelion-test.com/fullchain.pem
        SSLCertificateKeyFile /etc/letsencrypt/live/dandelion-test.com/privkey.pem
    </VirtualHost>
</IfModule>

```

#### 4. Заполнить конфигурационный файл сервера settings.json

Пример (для запуска сервера из docker контейнера, потому mariadb host `172.17.0.1`):
```json
{
    "db": {
        "user": "mariadb_dandelion_user",
        "password": "*** mariadb password ***",
        "host": "172.17.0.1",
        "port": 3306,
        "database": "dandelion_db",
        "ssl": 0
    },
    "jwt": {
        "secret": "*** your JSON Web Token secret ***",
        "issuer": "https://dandelion-test.com",
        "audience": "https://dandelion-test.com",
        "realm": "https://dandelion-test.com"
    },
    "server_instance_id": "*** you server instance id ***",
    "notifications": {
        "firebase_project_android": {
            "sender_id": "567891234567",
            "api_key": "AIsdSyBCYBUFb1X2Fl3jDairbKVVaRpa4-_Ts_H",
            "app_id": "1:567891234567:android:1beab2c3f4f567b89cc12b",
            "project_id": "dandelion-test-12fc4fb6"
        },
        "firebase_project_ios": {
            "sender_id": "567891234567",
            "api_key": "AISsdBCYBUFb1X7Fl2jDairbKVVaRpa3-_Ts_H",
            "app_id": "1:567891234567:ios:f123456f7891d2345cc67b",
            "project_id": "dandelion-test-12fc4fb6"
        },
        "firebase_server_private_key_path": "/app/config/dandelion-test-12fc4fb6-firebase-adminsdk-fbsvc-e12cf3fd45.json"
    },
    "test_bots": {
        "bots_on": 1,
        "bots_reactions_on": 1,
        "create_bots_if_not_exist": 1,
        "bots_logins_to_start": [ "user1", "user2", "user3", "user4", "user5", "user6", "user7", "user8", "user9", "user10" ]
    }
}
```
*   **db:** параметры подключения к БД.
*   **jwt:** параметры генерации JSON Web Token для мобильных клиентов.
*   **server_instance_id:** идентификатор сервера для случая когда у вас настроено несколько серверов, вы переключаетесь между ними и клиент должен понимать какие данные к какому серверу относятся.
*   **notifications:** параметры для получения Google Firebase оповещений Android и IOS мобильных приложений из п.2.
*   **test_bots:** включает ботов, при помощи которых можно протестировать работу сервера. При добавлении в чат боты отвечают случайными сообщениями и пишут при длительной тишине в чате.


#### 5. Запустить сервер

#### Переменные окружения для запуска сервера

*   **PORT:** порт, на котором будет работать сервер
*   **HOST:** хост, на котором будет работать сервер
*   **CONFIG:** путь к файлу конфигурации сервера

#### 5.1 Запуск сервера через fat-jar

Пример запуска:
```sh
CONFIG="config/settings.json" PORT=8085 HOST="127.0.0.1" /usr/lib/jvm/java-21-openjdk-amd64/bin/java -jar dandelionserver.jar
```

#### 5.2 Запуск сервера из docker контейнера

**docker-compose.yml** для запуска сервера на порту `8085`, на фиксированном ip `172.18.0.100` виртуальной сети. Предполагается, что конфигурационный файл сервера лежит по пути `config/settings.json` относительно `docker-compose.yml`:

```yaml
version: '3.8'

services:
  kotlin-server:
    image: zzhekaa/dandelionserver:latest
    container_name: dandelion-server
    ports:
      - "127.0.0.1:8085:8085"
    environment:
      - PORT=8085
      - HOST=0.0.0.0
      - CONFIG=/app/config/settings.json
    networks:
      dandelion-net:
        ipv4_address: 172.18.0.100
    volumes:
      - ./config:/app/config
    restart: unless-stopped

networks:
  dandelion-net:
    driver: bridge
    ipam:
      config:
        - subnet: 172.18.0.0/16

```

Запуск контейнера из директории с docker-compose.yml:

```sh
docker-compose up -d
```

