# Troubleshooting - Planfix MCP Server

## 🔧 Устранение неполадок

Руководство по решению типичных проблем при работе с Planfix MCP Server.

---

## 🚨 Проблемы подключения

### Ошибка: "Failed to load configuration"

**Симптомы:**
```
RuntimeError: Failed to load configuration: PLANFIX_ACCOUNT environment variable is required
```

**Причины:**
- Отсутствуют обязательные переменные окружения
- Неправильно настроен `.env` файл
- Переменные имеют пустые значения

**Решение:**

1. **Проверьте `.env` файл:**
```bash
# Убедитесь, что файл .env существует и содержит:
PLANFIX_ACCOUNT=your-account-name
PLANFIX_API_KEY=your-api-key
PLANFIX_USER_KEY=your-user-key
```

2. **Проверьте переменные окружения:**
```bash
echo $PLANFIX_ACCOUNT
echo $PLANFIX_API_KEY  
echo $PLANFIX_USER_KEY
```

3. **Перезагрузите переменные:**
```bash
source .env
# или
export PLANFIX_ACCOUNT=your-account
```

---

### Ошибка: "Не удалось подключиться к Planfix API"

**Симптомы:**
```
❌ Ошибка авторизации. Проверьте API ключи Planfix.
❌ Не удалось подключиться к Planfix API
```

**Причины:**
- Неверные API ключи
- Неправильное название аккаунта
- Сетевые проблемы
- API ключи заблокированы или истекли

**Решение:**

1. **Проверьте данные аккаунта:**
   - Войдите в Planfix через браузер
   - Проверьте точное название аккаунта в URL: `https://YOUR-ACCOUNT.planfix.ru`

2. **Проверьте API ключи:**
   - Перейдите в Planfix → Настройки → API
   - Убедитесь, что API ключ активен
   - При необходимости создайте новый ключ

3. **Проверьте права доступа:**
   - API ключ должен иметь права на чтение и изменение задач/проектов
   - User Key должен соответствовать пользователю с необходимыми правами

4. **Тест подключения:**
```python
python -c "
import asyncio
from src.planfix_api import PlanfixAPI

async def test():
    api = PlanfixAPI()
    result = await api.test_connection()
    print('✅ Подключение успешно' if result else '❌ Ошибка подключения')

asyncio.run(test())
"
```

---

### Ошибка: "Connection timeout"

**Симптомы:**
```
❌ Превышено время ожидания запроса
httpx.TimeoutException
```

**Причины:**
- Медленное интернет-соединение
- Перегрузка серверов Planfix
- Блокировка файрволом

**Решение:**

1. **Увеличьте timeout в конфигурации:**
```env
REQUEST_TIMEOUT=60
```

2. **Проверьте сетевое соединение:**
```bash
ping planfix.ru
curl -I https://your-account.planfix.ru
```

3. **Проверьте proxy/firewall настройки**

---

## 🐛 Проблемы с MCP сервером

### Ошибка: "Server failed to start"

**Симптомы:**
```
❌ Критическая ошибка: Server initialization failed
ModuleNotFoundError: No module named 'mcp'
```

**Причины:**
- Не установлены зависимости
- Конфликт версий Python
- Неправильная структура проекта

**Решение:**

1. **Установите зависимости:**
```bash
uv sync
# или
pip install -r requirements.txt
```

2. **Проверьте версию Python:**
```bash
python --version  # Должно быть >= 3.8
```

3. **Проверьте установку MCP:**
```python
python -c "import mcp; print('MCP установлен успешно')"
```

---

### Ошибка: "Tools not found in Claude"

**Симптомы:**
- Сервер запускается без ошибок
- Инструменты не появляются в Claude Desktop
- Claude не видит MCP сервер

**Причины:**
- Неправильная конфигурация `claude_desktop_config.json`
- Сервер не зарегистрирован в Claude
- Ошибки в пути к скрипту

**Решение:**

1. **Проверьте путь к конфиг файлу:**

**macOS:**
```bash
~/Library/Application Support/Claude/claude_desktop_config.json
```

**Windows:**
```bash
%APPDATA%\Claude\claude_desktop_config.json
```

2. **Проверьте конфигурацию:**
```json
{
  "mcpServers": {
    "planfix": {
      "command": "python",
      "args": ["/full/path/to/planfix-mcp-server/src/planfix_server.py"],
      "env": {
        "PLANFIX_ACCOUNT": "your-account",
        "PLANFIX_API_KEY": "your-api-key",
        "PLANFIX_USER_KEY": "your-user-key"
      }
    }
  }
}
```

3. **Проверьте права доступа:**
```bash
ls -la /path/to/planfix_server.py
python /path/to/planfix_server.py  # Должен запускаться без ошибок
```

4. **Перезапустите Claude Desktop**

---

### Ошибка: "Import error в тестах"

**Симптомы:**
```
ModuleNotFoundError: No module named 'src.planfix_api'
ImportError: attempted relative import with no known parent package
```

**Причины:**
- Неправильная структура импортов
- Отсутствует `__init__.py`
- Неправильный PYTHONPATH

**Решение:**

1. **Запускайте тесты из корня проекта:**
```bash
cd planfix-mcp-server
python -m pytest tests/
```

2. **Добавьте в PYTHONPATH:**
```bash
export PYTHONPATH="${PYTHONPATH}:$(pwd)"
python -m pytest tests/
```

3. **Используйте uv для тестов:**
```bash
uv run pytest tests/
```

---

## 📊 Проблемы с данными

### Ошибка: "Task not found"

**Симптомы:**
```
❌ Объект не найден. Проверьте ID и попробуйте снова.
PlanfixNotFoundError: Resource not found
```

**Причины:**
- Неверный ID задачи/проекта
- Объект был удалён
- Нет прав доступа к объекту

**Решение:**

1. **Проверьте существование объекта в веб-интерфейсе Planfix**

2. **Проверьте права доступа пользователя**

3. **Используйте поиск для получения актуальных ID:**
```python
# Вместо прямого обращения к task/123
tasks = await api.search_tasks(query="название задачи")
```

---

### Ошибка: "Invalid date format"

**Симптомы:**
```
ValueError: time data '31/12/2024' does not match format '%Y-%m-%d'
```

**Причины:**
- Неправильный формат даты
- Использование локального формата вместо ISO

**Решение:**

1. **Используйте ISO формат дат:**
```python
# Правильно
deadline = "2024-12-31"

# Неправильно  
deadline = "31/12/2024"
deadline = "31.12.2024"
```

2. **Конвертация форматов:**
```python
from datetime import datetime
date_str = datetime.strptime("31/12/2024", "%d/%m/%Y").strftime("%Y-%m-%d")
```

---

## 🔍 Отладка

### Включение режима отладки

**В .env файле:**
```env
DEBUG=true
```

**Через переменную окружения:**
```bash
export DEBUG=true
python src/planfix_server.py
```

### Проверка логов

**Запуск с подробными логами:**
```bash
python -c "
import logging
logging.basicConfig(level=logging.DEBUG)

# Ваш код для тестирования
"
```

### Тестирование API напрямую

**Создайте тестовый скрипт:**
```python
# test_api.py
import asyncio
from src.planfix_api import PlanfixAPI

async def test_api():
    api = PlanfixAPI()
    
    # Тест подключения
    print("Тестирование подключения...")
    connection_ok = await api.test_connection()
    print(f"Подключение: {'✅' if connection_ok else '❌'}")
    
    if not connection_ok:
        return
    
    # Тест поиска задач
    print("Тестирование поиска задач...")
    try:
        tasks = await api.search_tasks(status="active")
        print(f"Найдено активных задач: {len(tasks)}")
    except Exception as e:
        print(f"Ошибка поиска: {e}")

if __name__ == "__main__":
    asyncio.run(test_api())
```

---

## 📱 Проблемы производительности

### Медленные запросы

**Симптомы:**
- Долгое время выполнения запросов
- Таймауты при больших объёмах данных

**Решение:**

1. **Используйте фильтры для уменьшения объёма данных:**
```python
# Вместо получения всех задач
tasks = await api.search_tasks()

# Используйте фильтры
tasks = await api.search_tasks(status="active", limit=50)
```

2. **Увеличьте timeout для тяжёлых операций:**
```env
REQUEST_TIMEOUT=120
```

3. **Используйте батчевые операции для множественных изменений**

---

## 🆘 Получение помощи

### Сбор информации для отчёта об ошибке

**1. Версия и окружение:**
```bash
python --version
pip list | grep mcp
uname -a  # Linux/macOS
```

**2. Логи ошибок:**
```bash
# Запустите с отладкой и сохраните логи
DEBUG=true python src/planfix_server.py 2>&1 | tee debug.log
```

**3. Конфигурация (без секретных данных):**
```bash
env | grep PLANFIX | sed 's/=.*/=***/'
```

### Где искать помощь

1. **GitHub Issues** - для сообщения об ошибках и запросов функций
2. **Документация Planfix API** - https://planfix.com/docs/api/
3. **MCP Documentation** - https://modelcontextprotocol.io/
4. **Community Discord/Slack** - для быстрых вопросов

### Полезные ресурсы

- [Официальная документация Planfix](https://planfix.com/help/)
- [MCP Specification](https://spec.modelcontextprotocol.io/)
- [Claude Desktop Configuration](https://docs.anthropic.com/claude/docs)
- [Python AsyncIO Guide](https://docs.python.org/3/library/asyncio.html)

---

## ✅ Чек-лист решения проблем

Перед обращением за помощью проверьте:

- [ ] Все переменные окружения настроены правильно
- [ ] API ключи актуальны и имеют необходимые права
- [ ] Зависимости установлены и версии совместимы
- [ ] Сеть позволяет доступ к planfix.ru
- [ ] Конфигурация Claude Desktop корректна
- [ ] Логи содержат подробную информацию об ошибке
- [ ] Тестовый скрипт воспроизводит проблему
- [ ] Версия Python >= 3.8

Следование этому чек-листу поможет быстро выявить и решить большинство проблем.