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
*   **Пакет установки:** fat-jar на jvm 21 или docker-контейнер. 
*   **БД:** Mariadb. Протестировано на `10.11.14-MariaDB`
*   **Система оповещения мобильных клиентов:** Google Firebase.

### Настройка и запуск сервера

#### 1. Установить и настроить Mariadb  

Установить Mariadb, создать пустую БД и пользователя с правами доступа к ней.  
Если работа с БД будет осуществляться из docker контейнера с фиксированным IP (в примере `172.18.0.100`) иметь это в виду при настройке прав.

#### 2. Подготовить проект Google Firebase

Подготовить параметры Google Firebase для Android и IOS. См. документацию в интернете.

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

Пример:
```json

```

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

