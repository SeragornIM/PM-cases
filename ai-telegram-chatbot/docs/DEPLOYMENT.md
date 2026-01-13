# Руководство по развертыванию

## Обзор

Это руководство описывает процесс развертывания телеграм-бота «ПетербургГаз» в различных средах.

## Требования к системе

### Минимальные требования
- **ОС:** Windows 10/11, Ubuntu 18.04+, CentOS 7+
- **Python:** 3.8 или выше
- **RAM:** 512 MB (минимум), 1 GB (рекомендуется)
- **Диск:** 100 MB свободного места
- **Сеть:** Стабильное интернет-соединение

### Рекомендуемые требования
- **ОС:** Ubuntu 20.04+ или Windows Server 2019+
- **Python:** 3.9 или выше
- **RAM:** 2 GB или больше
- **Диск:** 1 GB свободного места
- **Сеть:** Выделенное интернет-соединение

## Подготовка к развертыванию

### 1. Получение необходимых ключей

#### Telegram Bot Token
1. Напишите [@BotFather](https://t.me/BotFather) в Telegram
2. Выполните команду `/newbot`
3. Следуйте инструкциям для создания бота
4. Сохраните полученный токен

#### GigaChat API ключ
1. Зарегистрируйтесь на [developers.sber.ru](https://developers.sber.ru)
2. Создайте приложение в разделе GigaChat
3. Получите Client ID и Client Secret
4. Закодируйте их в Base64: `base64(Client ID:Client Secret)`

### 2. Подготовка файлов

#### FAQ файл
Создайте файл `faq.json` в корне проекта:
```json
[
  {
    "question": "Как подать заявку на подключение газа?",
    "answer": "Для подачи заявки на подключение газа необходимо обратиться в офис компании или подать заявку онлайн через официальный сайт."
  },
  {
    "question": "Сколько стоит подключение газа?",
    "answer": "Стоимость подключения газа зависит от технических условий и расстояния до газопровода. Точную стоимость можно узнать при подаче заявки."
  }
]
```

## Локальное развертывание

### Windows

1. **Скачайте и установите Python:**
   - Скачайте Python 3.8+ с [python.org](https://python.org)
   - Установите с опцией "Add Python to PATH"

2. **Клонируйте репозиторий:**
   ```cmd
   git clone <repository-url>
   cd Telegabot
   ```

3. **Создайте виртуальное окружение:**
   ```cmd
   python -m venv .venv
   .venv\Scripts\activate
   ```

4. **Установите зависимости:**
   ```cmd
   pip install -r requirements.txt
   ```

5. **Настройте конфигурацию:**
   - Скопируйте `env_example.txt` в `.env`
   - Заполните все необходимые параметры

6. **Запустите бота:**
   ```cmd
   python run_bot.py
   ```

### Linux/Ubuntu

1. **Установите Python и pip:**
   ```bash
   sudo apt update
   sudo apt install python3 python3-pip python3-venv
   ```

2. **Клонируйте репозиторий:**
   ```bash
   git clone <repository-url>
   cd Telegabot
   ```

3. **Создайте виртуальное окружение:**
   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   ```

4. **Установите зависимости:**
   ```bash
   pip install -r requirements.txt
   ```

5. **Настройте конфигурацию:**
   ```bash
   cp env_example.txt .env
   nano .env  # Отредактируйте файл
   ```

6. **Запустите бота:**
   ```bash
   python run_bot.py
   ```

## Развертывание на сервере

### Создание systemd сервиса (Linux)

1. **Создайте файл сервиса:**
   ```bash
   sudo nano /etc/systemd/system/petersburg-gaz-bot.service
   ```

2. **Добавьте содержимое:**
   ```ini
   [Unit]
   Description=PetersburgGaz Telegram Bot
   After=network.target

   [Service]
   Type=simple
   User=bot
   WorkingDirectory=/opt/petersburg-gaz-bot
   Environment=PATH=/opt/petersburg-gaz-bot/.venv/bin
   ExecStart=/opt/petersburg-gaz-bot/.venv/bin/python run_bot.py
   Restart=always
   RestartSec=10

   [Install]
   WantedBy=multi-user.target
   ```

3. **Создайте пользователя для бота:**
   ```bash
   sudo useradd -r -s /bin/false bot
   sudo mkdir -p /opt/petersburg-gaz-bot
   sudo chown -R bot:bot /opt/petersburg-gaz-bot
   ```

4. **Скопируйте файлы проекта:**
   ```bash
   sudo cp -r * /opt/petersburg-gaz-bot/
   sudo chown -R bot:bot /opt/petersburg-gaz-bot
   ```

5. **Активируйте и запустите сервис:**
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable petersburg-gaz-bot
   sudo systemctl start petersburg-gaz-bot
   ```

6. **Проверьте статус:**
   ```bash
   sudo systemctl status petersburg-gaz-bot
   ```

### Docker развертывание

1. **Создайте Dockerfile:**
   ```dockerfile
   FROM python:3.9-slim

   WORKDIR /app

   COPY requirements.txt .
   RUN pip install --no-cache-dir -r requirements.txt

   COPY . .

   CMD ["python", "run_bot.py"]
   ```

2. **Создайте docker-compose.yml:**
   ```yaml
   version: '3.8'

   services:
     bot:
       build: .
       container_name: petersburg-gaz-bot
       restart: unless-stopped
       env_file:
         - .env
       volumes:
         - ./faq.json:/app/faq.json
         - ./bot.db:/app/bot.db
       networks:
         - bot-network

   networks:
     bot-network:
       driver: bridge
   ```

3. **Запустите контейнер:**
   ```bash
   docker-compose up -d
   ```

## Конфигурация для продакшена

### Переменные окружения

Создайте файл `.env` с продакшен настройками:

```bash
# Telegram Bot
TELEGRAM_BOT_TOKEN=your_production_token

# GigaChat API
GIGACHAT_API_KEY=your_production_key
GIGACHAT_BASE_URL=https://gigachat.devices.sberbank.ru
GIGACHAT_MODEL=GigaChat-2

# Администраторы
ADMIN_IDS=123456789,987654321

# Пути к файлам
FAQ_PATH=faq.json
DB_PATH=/var/lib/petersburg-gaz-bot/bot.db

# Настройки производительности
REQUEST_TIMEOUT=30
MAX_CONTEXT_LENGTH=4000

# Логирование
LOG_LEVEL=INFO
LOG_FILE=/var/log/petersburg-gaz-bot/bot.log
```

### Настройка логирования

1. **Создайте директорию для логов:**
   ```bash
   sudo mkdir -p /var/log/petersburg-gaz-bot
   sudo chown bot:bot /var/log/petersburg-gaz-bot
   ```

2. **Настройте logrotate:**
   ```bash
   sudo nano /etc/logrotate.d/petersburg-gaz-bot
   ```

3. **Добавьте конфигурацию:**
   ```
   /var/log/petersburg-gaz-bot/*.log {
       daily
       missingok
       rotate 30
       compress
       delaycompress
       notifempty
       create 644 bot bot
   }
   ```

### Настройка мониторинга

#### Prometheus метрики

Добавьте в `bot.py`:

```python
from prometheus_client import Counter, Histogram, start_http_server

# Метрики
messages_total = Counter('bot_messages_total', 'Total messages processed')
response_time = Histogram('bot_response_time_seconds', 'Response time')
gigachat_requests = Counter('bot_gigachat_requests_total', 'GigaChat API requests')

# В начале main()
start_http_server(8000)  # Prometheus endpoint на порту 8000
```

#### Health check endpoint

```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/health')
def health_check():
    return jsonify({
        'status': 'healthy',
        'timestamp': datetime.now().isoformat(),
        'version': '1.0.0'
    })

# Запуск в отдельном потоке
import threading
threading.Thread(target=lambda: app.run(host='0.0.0.0', port=8080)).start()
```

## Резервное копирование

### Автоматическое резервное копирование

1. **Создайте скрипт резервного копирования:**
   ```bash
   #!/bin/bash
   # backup.sh
   
   BACKUP_DIR="/var/backups/petersburg-gaz-bot"
   DB_PATH="/var/lib/petersburg-gaz-bot/bot.db"
   DATE=$(date +%Y%m%d_%H%M%S)
   
   mkdir -p $BACKUP_DIR
   
   # Резервное копирование базы данных
   cp $DB_PATH $BACKUP_DIR/bot_$DATE.db
   
   # Резервное копирование конфигурации
   cp .env $BACKUP_DIR/env_$DATE
   cp faq.json $BACKUP_DIR/faq_$DATE.json
   
   # Удаление старых резервных копий (старше 30 дней)
   find $BACKUP_DIR -name "*.db" -mtime +30 -delete
   find $BACKUP_DIR -name "env_*" -mtime +30 -delete
   find $BACKUP_DIR -name "faq_*" -mtime +30 -delete
   ```

2. **Добавьте в crontab:**
   ```bash
   # Ежедневное резервное копирование в 2:00
   0 2 * * * /opt/petersburg-gaz-bot/backup.sh
   ```

## Масштабирование

### Горизонтальное масштабирование

Для обработки большого количества пользователей:

1. **Запустите несколько экземпляров бота**
2. **Используйте общую базу данных** (PostgreSQL вместо SQLite)
3. **Настройте load balancer** для распределения нагрузки
4. **Используйте Redis** для кэширования

### Вертикальное масштабирование

1. **Увеличьте RAM** для кэширования
2. **Используйте SSD** для быстрого доступа к базе данных
3. **Настройте мониторинг** производительности

## Безопасность

### Рекомендации по безопасности

1. **Используйте отдельного пользователя** для запуска бота
2. **Ограничьте права доступа** к файлам конфигурации
3. **Регулярно обновляйте** зависимости
4. **Мониторьте логи** на предмет подозрительной активности
5. **Используйте HTTPS** для всех внешних соединений

### Настройка файрвола

```bash
# Разрешить только необходимые порты
sudo ufw allow 22    # SSH
sudo ufw allow 80    # HTTP (если нужен)
sudo ufw allow 443   # HTTPS (если нужен)
sudo ufw enable
```

## Мониторинг и алерты

### Настройка алертов

1. **Мониторинг доступности бота:**
   ```bash
   # Скрипт проверки
   #!/bin/bash
   if ! systemctl is-active --quiet petersburg-gaz-bot; then
       echo "Bot is down!" | mail -s "Bot Alert" admin@company.com
   fi
   ```

2. **Мониторинг использования ресурсов:**
   ```bash
   # Проверка использования памяти
   if [ $(ps -o rss= -p $(pgrep -f "python run_bot.py")) -gt 500000 ]; then
       echo "High memory usage!" | mail -s "Bot Alert" admin@company.com
   fi
   ```

### Логирование

1. **Настройте централизованное логирование** (ELK Stack)
2. **Используйте structured logging** (JSON формат)
3. **Настройте алерты** на критические ошибки

## Обновление

### Процесс обновления

1. **Остановите бота:**
   ```bash
   sudo systemctl stop petersburg-gaz-bot
   ```

2. **Создайте резервную копию:**
   ```bash
   ./backup.sh
   ```

3. **Обновите код:**
   ```bash
   git pull origin main
   pip install -r requirements.txt
   ```

4. **Запустите бота:**
   ```bash
   sudo systemctl start petersburg-gaz-bot
   ```

5. **Проверьте статус:**
   ```bash
   sudo systemctl status petersburg-gaz-bot
   ```

## Устранение неполадок

### Частые проблемы

1. **Бот не запускается:**
   - Проверьте логи: `sudo journalctl -u petersburg-gaz-bot -f`
   - Проверьте права доступа к файлам
   - Убедитесь, что все зависимости установлены

2. **GigaChat API недоступен:**
   - Проверьте сетевое соединение
   - Убедитесь в правильности API ключа
   - Проверьте статус сервиса GigaChat

3. **Высокое использование памяти:**
   - Перезапустите бота
   - Проверьте на утечки памяти
   - Увеличьте лимиты системы

### Диагностические команды

```bash
# Проверка статуса сервиса
sudo systemctl status petersburg-gaz-bot

# Просмотр логов
sudo journalctl -u petersburg-gaz-bot -f

# Проверка использования ресурсов
ps aux | grep python
top -p $(pgrep -f "python run_bot.py")

# Проверка сетевых соединений
netstat -tulpn | grep python
```

## Заключение

Следуя этому руководству, вы сможете успешно развернуть и поддерживать телеграм-бота «ПетербургГаз» в продакшене. Регулярно обновляйте систему и мониторьте её работу для обеспечения стабильной работы.
