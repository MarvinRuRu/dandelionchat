## Dandelion Messenger

### 📝 Project Description
Client-server messenger application. Developed for **educational** purposes and as a **portfolio**.

Closed source. Binaries are available for free download and deployment.

### 🏗️ Features
* **User Chats:** Chats for 2 or more users.
* **Message Reactions and Comments**
* **Comment Tracking:** Tracks the last 500 messages on which a user has left a comment.
* **No open registration:** User accounts are created exclusively by the system administrator.
* **Soft Delete:** Deleting messages, chats, and reactions does not physically delete them from the system. The information is marked with a 'deleted' flag in the database and is stored to ensure compliance with applicable laws.

### ⚠️ Legal Disclaimer
> **IMPORTANT:** Before using the messenger, we strongly recommend that you review and strictly comply with your country's laws and local regulations governing messengers and digital communications.** Depending on your jurisdiction, launching or using independent messaging platforms may require compliance with data retention, licensing, or mandatory user identification laws. The developers are not responsible for any unauthorized use of the software.

* ### 📄 **Client**

* ### 📄 **Server**

#### Key Features
* **Installation Package:** 📦 **[Fat JAR](https://github.com/MarvinRuRu/dandelionchat/releases/)** on jvm 21 or docker-image `zzhekaa/dandelionserver:latest`
* **DB:** Mariadb. Tested on `10.11.14-MariaDB`
* **Mobile Client Notification System:** Google Firebase.

### Setting Up and Running the Server

#### 1. Install and Configure Mariadb

Install Mariadb, create an empty database and a user with access rights to it.
If you'll be accessing the database from a Docker container with a fixed IP (in this example, 172.18.0.100), keep this in mind when setting permissions.

#### 2. Prepare the Google Firebase project

2.1 Prepare the Google Firebase parameters for Android and iOS, which will be used later in step 4. See the online documentation.

IOS App Store ID: `6788429394`

IOS Bundle ID: `zh.dandelion.client`

IOS Team ID: `DC7N64S349`

Android package name: `zh.dandelion.client.full`

2.2 Download the `Firebase Server Private Key` to a directory accessible to the server (for example, to the directory with the `settings.json` configuration file from step 4).

#### 3. Set up an Apache Reverse Proxy site for the server

Register a domain for your messenger, for example, `dandelion-test.com`.

Example Apache Reverse Proxy configuration file `/etc/apache2/sites-available/dandelion-test.com.conf` (for server port `8085`, domain `dandelion-test.com`, and a `letsencrypt` certificate):
```
<IfModule mod_ssl.c>
<VirtualHost *:443>
ServerName dandelion-test.com

SSLEngine on

RewriteEngine on
RewriteCond %{HTTP:Upgrade} =websocket [NC]
RewriteRule ^/(.*) ws://127.0.0.1:8085/$1 [P,L]

ProxyPreserveHost on
ProxyPass / http://127.0.0.1:8085/
ProxyPassReverse / http://127.0.0.1:8085/

RequestHeader Set X-Forwarded-Proto "https"

ErrorLog ${APACHE_LOG_DIR}/error.log
CustomLog ${APACHE_LOG_DIR}/access.log combined
Include /etc/letsencrypt/options-ssl-apache.conf

SSLCertificateFile /etc/letsencrypt/live/dandelion-test.com/fullchain.pem
SSLCertificateKeyFile /etc/letsencrypt/live/dandelion-test.com/privkey.pem
</VirtualHost>
</IfModule>

```

#### 4. Fill in the server configuration file settings.json

Example (for running the server from a Docker container, so mariadb host is `172.17.0.1`):
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
    "first_run_settings": {
    	"create_users": [
    	    { "login": "admin", "password": "12345", "is_admin": true },
    	    { "login": "user1", "password": "12345", "is_admin": false },
    	    { "login": "user2", "password": "12345", "is_admin": false },
    	    { "login": "user3", "password": "12345", "is_admin": false },
    	    { "login": "user4", "password": "12345", "is_admin": false },
    	    { "login": "user5", "password": "12345", "is_admin": false },
    	    { "login": "user6", "password": "12345", "is_admin": false },
    	    { "login": "user7", "password": "12345", "is_admin": false },
    	    { "login": "user8", "password": "12345", "is_admin": false },
    	    { "login": "user9", "password": "12345", "is_admin": false }
    	]
    },
    "test_bots": {
        "bots_on": 1,
        "bots_reactions_on": 1,
        "create_bots_if_not_exist": 1,
        "bots_logins_to_start": [ "user1", "user2", "user3", "user4", "user5", "user6", "user7", "user8", "user9", "user10" ]
    }
}
```
* **db:** Database connection parameters.
* **jwt:** JSON Web Token generation parameters for mobile clients.
* **server_instance_id:** Server ID for when you have multiple servers configured, you switch between them, and the client needs to understand which data belongs to which server.
* **notifications:** Parameters for receiving Google Firebase notifications for Android and iOS mobile apps from step 2.
* **first_run_settings:** First-run settings - user creation.
* **test_bots:** Enables bots that can be used to test server operation. When added to the chat, bots respond with random messages and respond during prolonged silence in the chat.

#### 5. Start the server

#### Environment variables for starting the server

* **PORT:** Port where the server will run
* **HOST:** Host where the server will run
* **CONFIG:** Path to the server configuration file

#### 5.1 Starting the server via fat-jar

Example run:
```sh
CONFIG="config/settings.json" PORT=8085 HOST="127.0.0.1" /usr/lib/jvm/java-21-openjdk-amd64/bin/java -jar dandelionserver.jar
```

#### 5.2 Starting the server from a Docker container

**docker-compose.yml** to start the server on port 8085, on a fixed IP address of 172.18.0.100 in the virtual network. It is assumed that the server configuration file is located at `config/settings.json` relative to `docker-compose.yml`:

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

Running a container from a directory with docker-compose.yml:

```sh
docker-compose up -d
```
