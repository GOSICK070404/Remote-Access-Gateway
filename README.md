# Remote-Access-Gateway

Защищенный удаленный доступ на базе Apache Guacamole и Authentik: исходный код стенда и скрипты развертывания.

## "Команды.txt" — Полный пошаговый скрипт развертывания стенда (TVM + GVM) + небольшая описательная часть работы с GUI.
*   Все действия выполняются строго в порядке, описанном в файле.

### 1. Подготовка Целевой ВМ (TVM - 192.168.56.20)
*   Настройте сеть (Host-Only) и установите OpenSSH Server.
*   Настройте UFW: разрешите вход только с IP шлюза (192.168.56.10) на порты 22 и 3389.
*   Установите и настройте xrdp + XFCE.

### 2. Настройка Шлюза (GVM - 192.168.56.10)
*   Настройте сеть и установите Docker и OpenSSH Server.
*   Создайте директорию ~/guacamole-stack и сгенерируйте docker-compose.yml, выполнив соответствующий блок из "Команды.txt".
*   Инициализируйте БД Guacamole (не забудьте про sleep 10 перед initdb!).
*   Запустите Authentik и создайте OIDC Provider (Client ID/Secret).
*   Обновите docker-compose.yml реальными данными OIDC и запустите Guacamole.

### 3. Финализация безопасности
*   Зарегистрируйте TOTP устройство для администратора в Authentik.
*   Включите OIDC в Guacamole (OPENID_ENABLED: "true").
*   Создайте пользователя akadmin и отключите дефолтного guacadmin.
*   Сгенерируйте реальный AUTHENTIK_SECRET_KEY.
*   Отключите SSH на самом шлюзе для харденинга.

##  Управление пользователями

### Добавление нового пользователя
1.  Зайдите в панель управления Authentik (http://192.168.56.10:9000).
2.  Перейдите в "Directory" → "Users" → "Create".
3.  Заполните данные (Username, Email, Password).
4.  При первом входе пользователь будет обязан настроить MFA (TOTP), если это включено в Flow аутентификации.

### Настройка MFA (TOTP)
Пользователь может настроить второй фактор самостоятельно после первого входа:
1.  Войти в Guacamole через OIDC.
2.  Нажать на имя пользователя (справа сверху) → Settings.
3.  Выбрать "MFA Devices" → "Register new device".
4.  Отсканировать QR-код в Google Authenticator.

##  Создание подключений к ресурсам

Настройка осуществляется в веб-интерфейсе Guacamole (http://192.168.56.10:8080/guacamole/api/ext/openid/login) под учетной записью администратора (akadmin).

1.  Перейдите в "Settings" → "Connections" → "New Connection".
2.  Для SSH:
    *   Name: "Name"
    *   Protocol: "SSH"
    *   Hostname: "IP", Port: "22"
    *   Authentication: "Username/Password".
3.  Для RDP:
    *   Name: "Name"
    *   Protocol: "RDP"
    *   Hostname: "IP", Port: "3389"
    *   Authentication: "Username/Password".
4.  Нажмите "Save". Подключение появится в списке "All Connections".

##  Как подключаться

1.  Откройте браузер на любой машине в сети.
2.  Перейдите по ссылке входа: http://192.168.56.10:8080/guacamole/api/ext/openid/login
3.  Введите логин/пароль от Authentik.
4.  Введите 6-значный код из Google Authenticator.
5.  После успешной аутентификации вы попадете в дашборд Guacamole.
6.  Кликните на нужное подключение (SSH или RDP) в левой панели.
7.  Готово
