# TLS/SSL и HTTPS — практическое руководство

## Что такое TLS и SSL

**TLS (Transport Layer Security)** — криптографический протокол, обеспечивающий:

* шифрование данных;
* проверку подлинности сервера;
* контроль целостности передаваемой информации.

**SSL (Secure Sockets Layer)** — устаревший предшественник TLS.

На практике термин «SSL-сертификат» до сих пор используется, хотя современные системы используют TLS.

---

# Зачем нужен HTTPS

**HTTPS (HTTP Secure)** = HTTP + TLS.

Преимущества:

* защита от перехвата данных;
* подтверждение подлинности сайта;
* защита от подмены содержимого;
* повышение доверия браузеров и пользователей;
* улучшение SEO.

---

# Типы шифрования

| Тип           | Как работает                           | Скорость | Применение      |
| ------------- | -------------------------------------- | -------- | --------------- |
| Симметричное  | Один ключ для шифрования и расшифровки | Быстро   | Передача данных |
| Асимметричное | Пара ключей (Public + Private)         | Медленно | TLS Handshake   |

---

# Основные понятия TLS

## Приватный ключ (Private Key)

Хранится только на сервере.

Пример:

```text
server.key
```

Никогда не должен передаваться третьим лицам.

---

## Публичный ключ (Public Key)

Содержится внутри сертификата.

Используется клиентами для проверки сервера.

---

## CSR (Certificate Signing Request)

Запрос на выпуск сертификата.

Содержит:

* публичный ключ;
* доменное имя;
* сведения о владельце.

Пример:

```text
request.csr
```

---

## Сертификат

Файл, подтверждающий принадлежность публичного ключа домену.

Пример:

```text
server.crt
server.pem
```

---

## CA (Certificate Authority)

Центр сертификации.

Популярные CA:

* Let's Encrypt;
* DigiCert;
* Sectigo;
* GlobalSign.

---

# Создание самоподписанного сертификата

## Одной командой

```bash
openssl req \
-x509 \
-newkey rsa:4096 \
-keyout key.pem \
-out cert.pem \
-days 365 \
-nodes \
-subj "/CN=example.com"
```

Создаются:

```text
key.pem    # приватный ключ
cert.pem   # сертификат
```

---

# Создание сертификата пошагово

## 1. Создать приватный ключ

```bash
openssl genrsa -out key.pem 2048
```

---

## 2. Создать CSR

```bash
openssl req \
-new \
-key key.pem \
-out request.csr \
-subj "/CN=example.com"
```

---

## 3. Подписать сертификат

```bash
openssl x509 \
-req \
-in request.csr \
-signkey key.pem \
-out cert.pem \
-days 365
```

---

# Просмотр сертификата

## Полная информация

```bash
openssl x509 -in cert.pem -text -noout
```

Показывает:

* Subject;
* Issuer;
* SAN;
* срок действия;
* алгоритмы подписи.

---

## Даты действия

```bash
openssl x509 -in cert.pem -dates -noout
```

Пример:

```text
notBefore=Jun 1 00:00:00 2026 GMT
notAfter=Jun 1 00:00:00 2027 GMT
```

---

## Владелец сертификата

```bash
openssl x509 -in cert.pem -subject -noout
```

---

## Отпечаток сертификата

```bash
openssl x509 -in cert.pem -fingerprint -noout
```

Пример:

```text
SHA256 Fingerprint=AA:BB:CC:...
```

---

# Проверка сертификата удалённого сервера

## Показать сертификаты

```bash
openssl s_client \
-connect google.com:443 \
-showcerts
```

---

## Извлечь сертификат

```bash
echo | openssl s_client \
-servername google.com \
-connect google.com:443 \
2>/dev/null | \
openssl x509 -text
```

---

## Проверить срок действия

30 дней:

```bash
echo | openssl s_client \
-servername google.com \
-connect google.com:443 \
2>/dev/null | \
openssl x509 -noout -checkend 2592000
```

Где:

```text
2592000 секунд = 30 дней
```

---

# TLS Handshake

Упрощённая схема:

```text
Клиент                               Сервер
   |                                   |
   |---- Client Hello ---------------->|
   |                                   |
   |<--- Server Hello -----------------|
   |<--- Certificate ------------------|
   |                                   |
   |---- Key Exchange ---------------->|
   |                                   |
   |---- Finished -------------------->|
   |<--- Finished ---------------------|
   |                                   |
   |<==== Зашифрованный трафик ======>|
```

---

# Проверка HTTPS через curl

## Игнорировать сертификат

Для тестов:

```bash
curl -k https://localhost
```

---

## Посмотреть детали TLS

```bash
curl -vI https://example.com
```

---

## Проверить TLS 1.2

```bash
curl --tlsv1.2 https://example.com -v
```

---

## Проверить TLS 1.3

```bash
curl --tlsv1.3 https://example.com -v
```

---

# Настройка HTTPS в Nginx

## Базовый HTTPS-сервер

```nginx
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate     /etc/ssl/certs/example.crt;
    ssl_certificate_key /etc/ssl/private/example.key;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    root /var/www/html;
    index index.html;
}
```

---

## Редирект HTTP → HTTPS

```nginx
server {
    listen 80;
    server_name example.com;

    return 301 https://$server_name$request_uri;
}
```

---

## Проверка конфигурации

```bash
sudo nginx -t
```

---

## Перезапуск Nginx

```bash
sudo systemctl restart nginx
```

---

# Получение сертификата Let's Encrypt

## Установка Certbot

```bash
sudo apt install certbot python3-certbot-nginx -y
```

---

## Автоматическая настройка Nginx

```bash
sudo certbot --nginx \
-d example.com \
-d www.example.com
```

---

## Только получение сертификата

```bash
sudo certbot certonly \
--standalone \
-d example.com
```

---

## Проверка продления

```bash
sudo certbot renew --dry-run
```

---

## Продление сертификатов

```bash
sudo certbot renew
```

---

# Пути Let's Encrypt

Сертификаты обычно находятся здесь:

```text
/etc/letsencrypt/live/example.com/
```

Содержимое:

```text
fullchain.pem
privkey.pem
cert.pem
chain.pem
```

---

# Форматы сертификатов

## PEM → DER

```bash
openssl x509 \
-in cert.pem \
-outform DER \
-out cert.der
```

---

## DER → PEM

```bash
openssl x509 \
-inform DER \
-in cert.der \
-out cert.pem
```

---

## PEM → PKCS#12

```bash
openssl pkcs12 \
-export \
-in cert.pem \
-inkey key.pem \
-out cert.p12
```

Используется для:

* Windows;
* IIS;
* браузеров;
* Java Keystore.

---

# Проверка цепочки сертификатов

## Проверка подписи

```bash
openssl verify \
-CAfile ca.crt \
server.crt
```

---

## Посмотреть всю цепочку

```bash
openssl s_client \
-connect example.com:443 \
-showcerts
```

---

# Subject Alternative Names (SAN)

Современные браузеры используют SAN вместо CN.

---

## Создать конфигурацию

```bash
cat > san.conf << EOF
[req]
distinguished_name=req_distinguished_name
req_extensions=req_ext
prompt=no

[req_distinguished_name]
CN=example.com

[req_ext]
subjectAltName=@alt_names

[alt_names]
DNS.1=example.com
DNS.2=www.example.com
DNS.3=api.example.com
IP.1=192.168.1.147
EOF
```

---

## Создать CSR

```bash
openssl req \
-newkey rsa:2048 \
-nodes \
-keyout key.pem \
-out request.csr \
-config san.conf
```

---

## Самоподписать сертификат

```bash
openssl x509 \
-req \
-in request.csr \
-signkey key.pem \
-out cert.pem \
-days 365 \
-extensions req_ext \
-extfile san.conf
```

---

# Создание собственного CA

## Создать корневой ключ

```bash
openssl genrsa -out ca.key 2048
```

---

## Создать корневой сертификат

```bash
openssl req \
-x509 \
-new \
-key ca.key \
-out ca.crt \
-days 3650 \
-subj "/CN=My Root CA"
```

---

## Создать ключ сервера

```bash
openssl genrsa -out server.key 2048
```

---

## Создать CSR сервера

```bash
openssl req \
-new \
-key server.key \
-out server.csr \
-subj "/CN=myserver.local"
```

---

## Подписать сертификат серверу

```bash
openssl x509 \
-req \
-in server.csr \
-CA ca.crt \
-CAkey ca.key \
-CAcreateserial \
-out server.crt \
-days 365
```

---

## Добавить CA в доверенные

Зависит от ОС:

* Linux — системное хранилище сертификатов;
* Windows — Trusted Root Certification Authorities;
* macOS — Keychain Access.

---

# Мониторинг срока действия сертификатов

## Скрипт проверки

```bash
#!/bin/bash

DOMAIN="example.com"

EXPIRY=$(echo | openssl s_client \
-servername $DOMAIN \
-connect $DOMAIN:443 2>/dev/null | \
openssl x509 -noout -enddate | cut -d= -f2)

EXPIRY_EPOCH=$(date -d "$EXPIRY" +%s)
NOW_EPOCH=$(date +%s)

DAYS_LEFT=$(( ($EXPIRY_EPOCH - $NOW_EPOCH) / 86400 ))

echo "Certificate for $DOMAIN expires in $DAYS_LEFT days"
```

---

# Частые ошибки TLS

## Certificate has expired

Сертификат истёк.

Проверить:

```bash
openssl x509 -in cert.pem -dates -noout
```

---

## Self-signed certificate

Используется самоподписанный сертификат.

Решение:

* добавить CA в доверенные;
* использовать Let's Encrypt.

---

## Hostname mismatch

Имя в сертификате не совпадает с доменом.

Проверить SAN:

```bash
openssl x509 -in cert.pem -text -noout
```

---

## Certificate chain incomplete

Не передана цепочка промежуточных сертификатов.

Для Nginx использовать:

```nginx
ssl_certificate fullchain.pem;
```

а не:

```nginx
ssl_certificate cert.pem;
```

---

# Шпаргалка OpenSSL

| Команда                                      | Назначение                 |
| -------------------------------------------- | -------------------------- |
| `openssl genrsa -out key.pem 2048`           | Создать приватный ключ     |
| `openssl req -new -key key.pem -out req.csr` | Создать CSR                |
| `openssl req -x509 -newkey rsa:4096 ...`     | Самоподписанный сертификат |
| `openssl x509 -in cert.pem -text -noout`     | Просмотр сертификата       |
| `openssl x509 -in cert.pem -dates -noout`    | Срок действия              |
| `openssl s_client -connect host:443`         | Проверка TLS               |
| `openssl verify -CAfile ca.crt cert.crt`     | Проверка цепочки           |
| `openssl pkcs12 -export ...`                 | Экспорт в PKCS#12          |

---

# Запомнить

### Создать сертификат

```bash
openssl req -x509 -newkey rsa:4096 -nodes \
-keyout key.pem \
-out cert.pem \
-days 365
```

### Проверить сертификат

```bash
openssl x509 -in cert.pem -text -noout
```

### Проверить сервер

```bash
openssl s_client -connect example.com:443
```

### Получить Let's Encrypt

```bash
sudo certbot --nginx -d example.com
```

### Проверить конфиг Nginx

```bash
sudo nginx -t
```

### Продлить сертификаты

```bash
sudo certbot renew
```
