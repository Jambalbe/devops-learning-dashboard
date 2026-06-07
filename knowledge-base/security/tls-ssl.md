# TLS/SSL и HTTPS — практическое руководство

## Что такое TLS/SSL

TLS (Transport Layer Security) — криптографический протокол, обеспечивающий шифрование данных при передаче по сети. SSL — устаревшая версия (говорят SSL, подразумевают TLS).

## Зачем нужен HTTPS

- Шифрование данных (защита от перехвата)
- Аутентификация сервера (удостоверение, что сайт настоящий)
- Целостность данных (защита от подмены)

## Типы шифрования

| Тип | Как работает | Скорость | Применение |
|-----|--------------|----------|------------|
| Симметричное | Один ключ для шифрования и расшифровки | Быстро | Сами данные |
| Асимметричное | Пара ключей (публичный + приватный) | Медленно | Обмен ключами (handshake) |

## Создание самоподписанного сертификата

```bash
# Одной командой (ключ + сертификат)
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes -subj "/CN=example.com"

# Пошагово:
# 1. Создать приватный ключ
openssl genrsa -out key.pem 2048

# 2. Создать запрос на сертификат (CSR)
openssl req -new -key key.pem -out request.csr -subj "/CN=example.com"

# 3. Подписать запрос (самоподпись)
openssl x509 -req -in request.csr -signkey key.pem -out cert.pem -days 365
Просмотр информации о сертификате
bash
# Основная информация (субъект, issuer, даты)
openssl x509 -in cert.pem -text -noout

# Только даты действия
openssl x509 -in cert.pem -dates -noout

# Только субъект (CN, O, OU)
openssl x509 -in cert.pem -subject -noout

# Отпечаток (fingerprint)
openssl x509 -in cert.pem -fingerprint -noout
Проверка сертификата удалённого сервера
bash
# Подключиться и показать сертификат
openssl s_client -connect google.com:443 -showcerts

# Показать только сертификат (без лишнего вывода)
echo | openssl s_client -servername google.com -connect google.com:443 2>/dev/null | openssl x509 -text

# Проверить, истекает ли сертификат в ближайшие 30 дней
echo | openssl s_client -servername google.com -connect google.com:443 2>/dev/null | openssl x509 -noout -checkend 2592000
#Настройка HTTPS в nginx
nginx
# /etc/nginx/sites-available/example
server {
    listen 443 ssl;
    server_name example.com;
    
    ssl_certificate /etc/ssl/certs/example.crt;
    ssl_certificate_key /etc/ssl/private/example.key;
    
    # Рекомендуемые настройки (безопасные шифры)
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    
    root /var/www/html;
    index index.html;
}

# HTTP → HTTPS редирект
server {
    listen 80;
    server_name example.com;
    return 301 https://$server_name$request_uri;
}
Проверка HTTPS
bash
# Через curl
curl -k https://localhost          # игнорировать проверку сертификата (для самоподписанного)
curl -vI https://example.com       # детальный вывод (покажет сертификат)

# Проверить поддержку TLS версий
curl --tlsv1.2 https://example.com -v 2>&1 | grep "SSL connection"
curl --tlsv1.3 https://example.com -v 2>&1 | grep "SSL connection"
TLS handshake (схема работы)
text
Клиент                                     Сервер
   |                                          |
   |------- Client Hello (TLS версии, шифры) --->|
   |                                          |
   |<------ Server Hello + Certificate + Server Hello Done ---|
   |                                          |
   |--- Client Key Exchange (секрет, зашифрованный публичным ключом) -->|
   |                                          |
   |--- Change Cipher Spec, Finished --------->|
   |                                          |
   |<--- Change Cipher Spec, Finished ---------|
   |                                          |
   |<============= Зашифрованные данные =============>|

#Получение бесплатного сертификата от Let's Encrypt
bash
# Установка certbot
sudo apt install certbot python3-certbot-nginx -y

# Получить сертификат для nginx (автоматическая настройка)
sudo certbot --nginx -d example.com -d www.example.com

# Получить сертификат вручную (только файлы)
sudo certbot certonly --standalone -d example.com

# Автоматическое продление
sudo certbot renew --dry-run          # тест
sudo certbot renew                     # реальное продление
Форматы сертификатов (конвертация)
bash
# PEM → DER (бинарный)
openssl x509 -in cert.pem -outform DER -out cert.der

# DER → PEM
openssl x509 -inform DER -in cert.der -out cert.pem

# PEM + ключ → PKCS#12 (для импорта в браузер/Windows)
openssl pkcs12 -export -in cert.pem -inkey key.pem -out cert.p12

#Проверка цепочки сертификатов
bash
# Проверить, что сертификат подписан CA
openssl verify -CAfile ca.crt server.crt

# Показать полную цепочку (включая промежуточные)
openssl s_client -connect example.com:443 -showcerts
Генерация сертификата с Subject Alternative Names (SAN)
bash
# Создать конфиг
cat > san.conf << EOF
[req]
distinguished_name = req_distinguished_name
req_extensions = req_ext
prompt = no

[req_distinguished_name]
CN = example.com

[req_ext]
subjectAltName = @alt_names

[alt_names]
DNS.1 = example.com
DNS.2 = www.example.com
DNS.3 = api.example.com
IP.1 = 192.168.1.147
EOF

# Создать ключ и CSR
openssl req -newkey rsa:2048 -nodes -keyout key.pem -out request.csr -config san.conf

# Самоподпись с SAN
openssl x509 -req -in request.csr -signkey key.pem -out cert.pem -days 365 -extensions req_ext -extfile san.conf
Создание собственного CA (Certificate Authority)
bash
# 1. Создать корневой ключ CA
openssl genrsa -out ca.key 2048

# 2. Создать корневой сертификат CA (10 лет)
openssl req -x509 -new -key ca.key -out ca.crt -days 3650 -subj "/CN=My Root CA"

# 3. Создать сертификат сервера, подписанный CA
openssl genrsa -out server.key 2048
openssl req -new -key server.key -out server.csr -subj "/CN=myserver.local"
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 365

# 4. На клиенте: добавить ca.crt в доверенные
# (инструкция зависит от ОС)

#Проблема: истекающий сертификат (мониторинг)
bash
# Скрипт для проверки срока действия
#!/bin/bash
DOMAIN="example.com"
EXPIRY=$(echo | openssl s_client -servername $DOMAIN -connect $DOMAIN:443 2>/dev/null | openssl x509 -noout -enddate | cut -d= -f2)
EXPIRY_EPOCH=$(date -d "$EXPIRY" +%s)
NOW_EPOCH=$(date +%s)
DAYS_LEFT=$(( ($EXPIRY_EPOCH - $NOW_EPOCH) / 86400 ))
echo "Certificate for $DOMAIN expires in $DAYS_LEFT days"
Краткая шпаргалка по командам openssl
Команда	Назначение
openssl req -x509 -newkey rsa:4096 -nodes -keyout key.pem -out cert.pem -days 365	Самоподписанный сертификат
openssl x509 -in cert.pem -text -noout	Посмотреть сертификат
openssl s_client -connect host:443	Подключиться к серверу
openssl verify -CAfile ca.crt cert.crt	Проверить подпись
openssl genrsa -out key.pem 2048	Сгенерировать ключ
openssl pkcs12 -export -in cert.pem -inkey key.pem -out bundle.p12	Экспорт в PKCS#12
