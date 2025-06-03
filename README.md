# Инфраструктура math.spbu

Данный проект содержит набор сервисов для домена math.spbu:
- Nginx (веб-сервер)
- CoreDNS (DNS-сервер)
- SMTP (почтовый сервер)
- SSH (сервер для удаленного доступа)
- SNTP (сервер синхронизации времени)

## Требования

- Docker
- Docker Compose
- dig (для тестирования DNS)
- ntpdate (для тестирования SNTP)
- telnet (для тестирования SMTP)
- curl (для тестирования веб-сервера)

## Запуск

1. Клонируйте репозиторий:
```bash
git clone <repository-url>
cd <repository-directory>
```

2. Настройка SSH-ключей:
```bash
# Генерация SSH-ключа (если еще нет)
ssh-keygen -t ed25519 -C "your_email@example.com"

# Добавление публичного ключа на сервер
cat ~/.ssh/id_ed25519.pub > ssh/authorized_keys
```

3. Запустите все сервисы:
```bash
docker-compose up -d
```

## Тестирование сервисов

### 1. Веб-сервер (Nginx)

Проверка доступности веб-страницы:
```bash
# Через IP
curl http://192.168.31.128:81

# Через домен (если DNS настроен)
curl http://math.spbu:81
curl http://www.math.spbu:81
```

### 2. DNS-сервер (CoreDNS)

Проверка DNS-записей:
```bash
# Проверка A-записи
dig @192.168.31.128 -p 54 math.spbu A
dig @192.168.31.128 -p 54 www.math.spbu A
dig @192.168.31.128 -p 54 mail.math.spbu A
dig @192.168.31.128 -p 54 ssh.math.spbu A
dig @192.168.31.128 -p 54 ntp.math.spbu A

# Проверка NS-записи
dig @192.168.31.128 -p 54 math.spbu NS

# Проверка SOA-записи
dig @192.168.31.128 -p 54 math.spbu SOA
```

### 3. SMTP-сервер

Тестирование SMTP-сервера:
```bash
# Подключение к SMTP-серверу
telnet mail.math.spbu 26

# После подключения можно отправить тестовое письмо:
HELO math.spbu
MAIL FROM: <test@math.spbu>
RCPT TO: <recipient@example.com>
DATA
Subject: Test Email
This is a test email.
.
QUIT
```

Проверка отправки писем:
```bash
# 1. Отправка тестового письма на реальный адрес
telnet mail.math.spbu 25
HELO math.spbu
MAIL FROM: <test@math.spbu>
RCPT TO: <your-real-email@example.com>
DATA
Subject: Test Email from math.spbu
This is a test email from math.spbu SMTP server.
.
QUIT

# 2. Проверка логов SMTP-сервера
docker-compose logs -f smtp

# 3. Проверка очереди писем
docker-compose exec smtp mailq

# 4. Проверка статуса доставки
docker-compose exec smtp postqueue -p
```

Для более удобного тестирования можно использовать Python-скрипт:
```python
#!/usr/bin/env python3
import smtplib
from email.mime.text import MIMEText

# Настройки
smtp_server = "mail.math.spbu"
smtp_port = 25
sender = "test@math.spbu"
recipient = "your-email@example.com"

# Создание сообщения
msg = MIMEText("This is a test email from math.spbu SMTP server.")
msg['Subject'] = 'Test Email from math.spbu'
msg['From'] = sender
msg['To'] = recipient

# Отправка письма
try:
    with smtplib.SMTP(smtp_server, smtp_port) as server:
        server.send_message(msg)
    print("Email sent successfully!")
except Exception as e:
    print(f"Failed to send email: {e}")
```

Сохраните скрипт как `test_smtp.py` и запустите:
```bash
python3 test_smtp.py
```

### 4. SSH-сервер

#### Настройка SSH-ключей для разных ОС

##### macOS
```bash
# 1. Генерация ключа (если еще нет)
ssh-keygen -t ed25519 -C "your_email@example.com"
# Нажмите Enter для сохранения в стандартном месте (~/.ssh/id_ed25519)
# Можно оставить пароль пустым, нажав Enter дважды

# 2. Просмотр публичного ключа
cat ~/.ssh/id_ed25519.pub

# 3. Копирование ключа на сервер
# Вариант 1: Ручное копирование
cat ~/.ssh/id_ed25519.pub > ssh/authorized_keys

# Вариант 2: Через ssh-copy-id (если сервер уже доступен по паролю)
ssh-copy-id -i ~/.ssh/id_ed25519.pub admin@ssh.math.spbu
```

##### Windows
```powershell
# 1. Генерация ключа через PowerShell
ssh-keygen -t ed25519 -C "your_email@example.com"
# Нажмите Enter для сохранения в стандартном месте (C:\Users\YourUsername\.ssh\id_ed25519)
# Можно оставить пароль пустым, нажав Enter дважды

# 2. Просмотр публичного ключа
type $env:USERPROFILE\.ssh\id_ed25519.pub

# 3. Копирование ключа на сервер
# Вариант 1: Ручное копирование
# Скопируйте содержимое файла id_ed25519.pub и вставьте его в файл ssh/authorized_keys

# Вариант 2: Через PowerShell
Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub | Set-Content -Path ssh/authorized_keys
```

##### Linux
```bash
# 1. Генерация ключа (если еще нет)
ssh-keygen -t ed25519 -C "your_email@example.com"
# Нажмите Enter для сохранения в стандартном месте (~/.ssh/id_ed25519)
# Можно оставить пароль пустым, нажав Enter дважды

# 2. Просмотр публичного ключа
cat ~/.ssh/id_ed25519.pub

# 3. Копирование ключа на сервер
# Вариант 1: Ручное копирование
cat ~/.ssh/id_ed25519.pub > ssh/authorized_keys

# Вариант 2: Через ssh-copy-id (если сервер уже доступен по паролю)
ssh-copy-id -i ~/.ssh/id_ed25519.pub admin@ssh.math.spbu
```

#### Проверка подключения
```bash
# Подключение по IP
ssh -p 23 admin@192.168.31.128

# Подключение по домену (если DNS настроен)
ssh -p 23 admin@ssh.math.spbu
```

#### Добавление нескольких ключей
Если нужно добавить несколько ключей (например, для разных компьютеров), просто добавьте их в файл `ssh/authorized_keys`, каждый с новой строки:
```bash
# Добавление дополнительного ключа
echo "ssh-rsa AAAA..." >> ssh/authorized_keys
```

#### Устранение проблем
1. Проверка прав доступа:
```bash
# На клиенте
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub

# На сервере
chmod 700 ssh/authorized_keys
```

2. Проверка подключения с подробным выводом:
```bash
ssh -v admin@ssh.math.spbu
```

### 5. SNTP-сервер

Проверка синхронизации времени:
```bash
# Проверка через ntpdate
ntpdate -p 124 ntp.math.spbu

# Или через IP
ntpdate -p 124 192.168.31.128
```

## Настройка DNS на локальной машине

Для удобства тестирования можно добавить запись в локальный файл hosts:

```bash
# Добавьте в /etc/hosts:
192.168.31.128 math.spbu www.math.spbu mail.math.spbu ssh.math.spbu ntp.math.spbu
```

## Остановка сервисов

```bash
docker-compose down
```

## Мониторинг логов

Просмотр логов всех сервисов:
```bash
docker-compose logs -f
```

Просмотр логов конкретного сервиса:
```bash
docker-compose logs -f nginx
docker-compose logs -f coredns
docker-compose logs -f smtp
docker-compose logs -f ssh
docker-compose logs -f sntp
``` 