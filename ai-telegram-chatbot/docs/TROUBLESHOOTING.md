# Устранение неполадок

## Обзор

Это руководство поможет решить наиболее распространенные проблемы при работе с телеграм-ботом «ПетербургГаз».

## Диагностика проблем

### 1. Проверка статуса бота

#### Проверка запущенных процессов
```bash
# Windows
tasklist | findstr python

# Linux/Mac
ps aux | grep python
```

#### Проверка логов
```bash
# Если используется systemd
sudo journalctl -u petersburg-gaz-bot -f

# Если запущен вручную
# Логи отображаются в консоли
```

### 2. Проверка конфигурации

#### Валидация .env файла
```bash
# Проверьте наличие всех обязательных переменных
cat .env | grep -E "TELEGRAM_BOT_TOKEN|GIGACHAT_API_KEY|GIGACHAT_BASE_URL|GIGACHAT_MODEL|ADMIN_IDS"
```

#### Тестирование конфигурации
```python
# Создайте test_config.py
from config import validate_config, get_config_summary

try:
    validate_config()
    print("✅ Конфигурация корректна")
    print(get_config_summary())
except Exception as e:
    print(f"❌ Ошибка конфигурации: {e}")
```

## Частые проблемы и решения

### Проблема 1: Бот не запускается

#### Симптомы
- Ошибка при запуске `python run_bot.py`
- Сообщение "Не все требования выполнены"
- Исключения при инициализации

#### Возможные причины и решения

**1.1. Отсутствует файл .env**
```bash
# Решение
cp env_example.txt .env
# Отредактируйте .env файл
```

**1.2. Неверный формат токенов**
```bash
# Проверьте формат токена Telegram
# Должен быть: 123456789:ABCdefGHIjklMNOpqrsTUVwxyz

# Проверьте формат GigaChat API ключа
# Должен быть Base64 encoded Client ID:Client Secret
```

**1.3. Отсутствует файл FAQ**
```bash
# Создайте файл faq.json
cp faq_example.json faq.json
# Или создайте свой файл FAQ
```

**1.4. Не установлены зависимости**
```bash
# Установите зависимости
pip install -r requirements.txt

# Или переустановите
pip install --upgrade -r requirements.txt
```

#### Диагностика
```python
# Создайте test_startup.py
import sys
import os

print("=== Диагностика запуска ===")

# Проверка Python версии
print(f"Python версия: {sys.version}")

# Проверка .env файла
if os.path.exists('.env'):
    print("✅ Файл .env найден")
else:
    print("❌ Файл .env не найден")

# Проверка FAQ файла
if os.path.exists('faq.json'):
    print("✅ Файл faq.json найден")
else:
    print("❌ Файл faq.json не найден")

# Проверка зависимостей
try:
    import telegram
    print("✅ python-telegram-bot установлен")
except ImportError:
    print("❌ python-telegram-bot не установлен")

try:
    import requests
    print("✅ requests установлен")
except ImportError:
    print("❌ requests не установлен")

try:
    import sqlite3
    print("✅ sqlite3 доступен")
except ImportError:
    print("❌ sqlite3 недоступен")
```

### Проблема 2: GigaChat API недоступен

#### Симптомы
- Сообщение "Сервис временно не доступен"
- Ошибки 401, 403, 500 при обращении к GigaChat
- Бот отвечает только из FAQ

#### Возможные причины и решения

**2.1. Неверный API ключ**
```python
# Создайте test_gigachat.py
from gigachat_client import GigaChatClient
from config import GIGACHAT_API_KEY, GIGACHAT_BASE_URL, GIGACHAT_MODEL

client = GigaChatClient()
try:
    result = client.test_connection()
    print(f"✅ GigaChat доступен: {result}")
except Exception as e:
    print(f"❌ Ошибка GigaChat: {e}")
```

**2.2. Неверный URL API**
```bash
# Проверьте URL в .env
# Должен быть: https://gigachat.devices.sberbank.ru
# БЕЗ /api/v1 на конце
```

**2.3. Проблемы с SSL сертификатами**
```python
# В gigachat_client.py уже настроено verify=False
# Если проблема остается, проверьте корпоративный прокси
```

**2.4. Неверная модель**
```bash
# Проверьте доступные модели
python -c "
from gigachat_client import GigaChatClient
client = GigaChatClient()
try:
    models = client.get_available_models()
    print('Доступные модели:', models)
except Exception as e:
    print('Ошибка:', e)
"
```

#### Диагностика
```python
# Создайте test_gigachat_detailed.py
import requests
import base64
from config import GIGACHAT_API_KEY, GIGACHAT_BASE_URL

print("=== Диагностика GigaChat ===")

# Проверка API ключа
try:
    decoded = base64.b64decode(GIGACHAT_API_KEY).decode('utf-8')
    print(f"✅ API ключ декодирован: {decoded[:20]}...")
except Exception as e:
    print(f"❌ Ошибка декодирования API ключа: {e}")

# Проверка URL
print(f"URL API: {GIGACHAT_BASE_URL}")

# Тест получения токена
try:
    url = "https://ngw.devices.sberbank.ru:9443/api/v2/oauth"
    headers = {
        "Authorization": f"Basic {GIGACHAT_API_KEY}",
        "Content-Type": "application/x-www-form-urlencoded"
    }
    data = "scope=GIGACHAT_API_PERS"
    
    response = requests.post(url, headers=headers, data=data, verify=False, timeout=30)
    print(f"Статус получения токена: {response.status_code}")
    
    if response.status_code == 200:
        token_data = response.json()
        print(f"✅ Токен получен: {token_data.get('access_token', '')[:20]}...")
    else:
        print(f"❌ Ошибка получения токена: {response.text}")
        
except Exception as e:
    print(f"❌ Ошибка запроса: {e}")
```

### Проблема 3: Ошибки команды /stats

#### Симптомы
- "У вас нет прав для выполнения этой команды"
- Ошибки парсинга Markdown
- Некорректное отображение статистики

#### Возможные причины и решения

**3.1. Пользователь не является администратором**
```python
# Проверьте ID пользователя
# В Telegram напишите @userinfobot для получения своего ID
# Добавьте ID в ADMIN_IDS в .env файле
```

**3.2. Неверный формат ADMIN_IDS**
```bash
# Правильный формат: 123456789,987654321
# Только цифры, разделенные запятыми
# Без пробелов
```

**3.3. Ошибки Markdown парсинга**
```python
# Эта проблема уже исправлена в коде
# Статистика теперь отправляется как plain text
```

#### Диагностика
```python
# Создайте test_admin.py
from config import ADMIN_IDS

print("=== Диагностика администраторов ===")
print(f"ADMIN_IDS: {ADMIN_IDS}")

# Проверьте формат
try:
    admin_list = [int(x.strip()) for x in ADMIN_IDS.split(',')]
    print(f"✅ Список администраторов: {admin_list}")
except Exception as e:
    print(f"❌ Ошибка формата ADMIN_IDS: {e}")
```

### Проблема 4: Оффтопные вопросы обрабатываются неправильно

#### Симптомы
- Бот отвечает на оффтопные вопросы
- Показывает кнопки оценки для оффтопа
- Отправляет "Информации недостаточно" для оффтопа

#### Возможные причины и решения

**4.1. Неправильная настройка ключевых слов**
```python
# Проверьте OFFTOP_KEYWORDS в constants.py
# Добавьте недостающие ключевые слова
```

**4.2. Проблемы с логикой определения оффтопа**
```python
# Создайте test_offtop.py
from bot import PetersburgGazBot
from constants import OFFTOP_KEYWORDS

bot = PetersburgGazBot()

test_messages = [
    "Как приготовить пирог?",
    "Расскажи про медицину",
    "Как подключить газ?",
    "Сколько стоит газ?"
]

for message in test_messages:
    is_offtop = bot._check_offtop(message)
    print(f"'{message}' -> {'Оффтоп' if is_offtop else 'Тематический'}")
```

### Проблема 5: Проблемы с базой данных

#### Симптомы
- Ошибки SQLite
- Потеря данных
- Блокировка базы данных

#### Возможные причины и решения

**5.1. Блокировка базы данных**
```bash
# Остановите все процессы бота
# Windows
taskkill /f /im python.exe

# Linux
pkill -f "python run_bot.py"
```

**5.2. Повреждение базы данных**
```bash
# Создайте резервную копию
cp bot.db bot.db.backup

# Попробуйте восстановить
sqlite3 bot.db ".recover" | sqlite3 bot_recovered.db
```

**5.3. Недостаток места на диске**
```bash
# Проверьте свободное место
df -h  # Linux
dir     # Windows
```

#### Диагностика
```python
# Создайте test_database.py
import sqlite3
import os

print("=== Диагностика базы данных ===")

# Проверка файла базы данных
if os.path.exists('bot.db'):
    print("✅ Файл bot.db найден")
    size = os.path.getsize('bot.db')
    print(f"Размер базы данных: {size} байт")
else:
    print("❌ Файл bot.db не найден")

# Проверка подключения к базе данных
try:
    conn = sqlite3.connect('bot.db')
    cursor = conn.cursor()
    
    # Проверка таблиц
    cursor.execute("SELECT name FROM sqlite_master WHERE type='table';")
    tables = cursor.fetchall()
    print(f"✅ Таблицы в базе данных: {[table[0] for table in tables]}")
    
    # Проверка данных
    cursor.execute("SELECT COUNT(*) FROM users")
    user_count = cursor.fetchone()[0]
    print(f"Количество пользователей: {user_count}")
    
    cursor.execute("SELECT COUNT(*) FROM messages")
    message_count = cursor.fetchone()[0]
    print(f"Количество сообщений: {message_count}")
    
    conn.close()
    print("✅ Подключение к базе данных работает")
    
except Exception as e:
    print(f"❌ Ошибка базы данных: {e}")
```

### Проблема 6: Проблемы с парсингом сайта

#### Симптомы
- Ошибки при парсинге peterburggaz.ru
- Медленная работа бота
- Отсутствие контекста с сайта

#### Возможные причины и решения

**6.1. Сайт недоступен**
```python
# Создайте test_site.py
import requests
from retrievers import SiteRetriever

print("=== Диагностика парсинга сайта ===")

# Проверка доступности сайта
try:
    response = requests.get("https://peterburggaz.ru", timeout=10)
    print(f"✅ Сайт доступен: {response.status_code}")
except Exception as e:
    print(f"❌ Сайт недоступен: {e}")

# Тест парсинга
try:
    retriever = SiteRetriever()
    results = retriever.search_site("газ", top_k=1)
    print(f"✅ Парсинг работает: найдено {len(results)} результатов")
except Exception as e:
    print(f"❌ Ошибка парсинга: {e}")
```

**6.2. Изменения в структуре сайта**
```python
# Проверьте, изменилась ли структура сайта
# Возможно, нужно обновить селекторы в SiteRetriever
```

## Мониторинг и профилактика

### 1. Регулярные проверки

#### Ежедневные проверки
```bash
# Проверка статуса бота
sudo systemctl status petersburg-gaz-bot

# Проверка логов на ошибки
sudo journalctl -u petersburg-gaz-bot --since "1 day ago" | grep ERROR

# Проверка использования ресурсов
ps aux | grep python | grep run_bot
```

#### Еженедельные проверки
```bash
# Проверка размера базы данных
ls -lh bot.db

# Проверка логов на предупреждения
sudo journalctl -u petersburg-gaz-bot --since "1 week ago" | grep WARNING

# Проверка доступности внешних сервисов
curl -s https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/getMe
```

### 2. Автоматический мониторинг

#### Скрипт мониторинга
```bash
#!/bin/bash
# monitor.sh

# Проверка статуса бота
if ! systemctl is-active --quiet petersburg-gaz-bot; then
    echo "ALERT: Bot is down!" | mail -s "Bot Alert" admin@company.com
    systemctl restart petersburg-gaz-bot
fi

# Проверка размера базы данных
DB_SIZE=$(du -m bot.db | cut -f1)
if [ $DB_SIZE -gt 100 ]; then
    echo "WARNING: Database size is ${DB_SIZE}MB" | mail -s "Bot Warning" admin@company.com
fi

# Проверка использования памяти
MEMORY_USAGE=$(ps -o rss= -p $(pgrep -f "python run_bot.py"))
if [ $MEMORY_USAGE -gt 500000 ]; then
    echo "WARNING: High memory usage: ${MEMORY_USAGE}KB" | mail -s "Bot Warning" admin@company.com
fi
```

### 3. Резервное копирование

#### Автоматическое резервное копирование
```bash
#!/bin/bash
# backup.sh

BACKUP_DIR="/var/backups/petersburg-gaz-bot"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p $BACKUP_DIR

# Резервное копирование базы данных
cp bot.db $BACKUP_DIR/bot_$DATE.db

# Резервное копирование конфигурации
cp .env $BACKUP_DIR/env_$DATE
cp faq.json $BACKUP_DIR/faq_$DATE.json

# Удаление старых резервных копий
find $BACKUP_DIR -name "*.db" -mtime +30 -delete
find $BACKUP_DIR -name "env_*" -mtime +30 -delete
find $BACKUP_DIR -name "faq_*" -mtime +30 -delete
```

## Контакты и поддержка

### Внутренняя поддержка
- **Техническая поддержка:** admin@company.com
- **Документация:** docs/README.md
- **API документация:** docs/API.md

### Внешние ресурсы
- **Telegram Bot API:** https://core.telegram.org/bots/api
- **GigaChat API:** https://developers.sber.ru/portal/products/gigachat
- **Python Telegram Bot:** https://docs.python-telegram-bot.org/

### Логи и отладка
- **Логи приложения:** Консоль или /var/log/petersburg-gaz-bot/
- **Системные логи:** `sudo journalctl -u petersburg-gaz-bot`
- **Отладочная информация:** Установите `LOG_LEVEL=DEBUG` в .env

## Заключение

Большинство проблем можно решить, следуя этому руководству. Если проблема не решается:

1. Проверьте логи на наличие ошибок
2. Убедитесь в корректности конфигурации
3. Проверьте доступность внешних сервисов
4. Обратитесь к технической поддержке с подробным описанием проблемы

Помните: регулярное резервное копирование и мониторинг помогут предотвратить многие проблемы!
