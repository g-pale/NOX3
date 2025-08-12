# Руководство по использованию NOX Platform

## Обзор

NOX Platform - это модульная сетевая операционная система, написанная на языке NAX. Платформа предоставляет гибкую архитектуру для создания сетевых приложений и сервисов.

## Быстрый старт

### Установка

1. Скачайте последнюю версию NOX Platform
2. Распакуйте архив
3. Запустите установку:

```bash
nax run build.nax install
```

### Первый запуск

```bash
# Запуск платформы
nox

# Запуск с конфигурацией
nox --config config/system.conf

# Запуск в режиме отладки
nox --debug
```

## Архитектура

### Основные компоненты

1. **Ядро системы** (`core/system.nax`)
   - Управление процессами
   - Обработка событий
   - Управление памятью
   - Конфигурация

2. **Сетевой стек** (`core/network.nax`)
   - TCP/UDP протоколы
   - IP маршрутизация
   - Ethernet интерфейсы
   - Управление соединениями

3. **Модульная система** (`core/modules.nax`)
   - Загрузка модулей
   - Управление зависимостями
   - Жизненный цикл модулей

4. **API сервер** (`core/api.nax`)
   - REST API
   - Аутентификация
   - Управление запросами

### Модули

Модули - это расширения платформы, которые добавляют новую функциональность.

#### Структура модуля

```nax
class MyModule extends Module {
    constructor() {
        super("MyModule", "1.0.0")
        this.setDescription("Описание модуля")
        this.setAuthor("Автор")
        this.addDependency("CoreModule")
    }
    
    public function initialize(): void {
        // Инициализация модуля
    }
    
    public function update(): void {
        // Обновление модуля
    }
    
    public function shutdown(): void {
        // Завершение работы модуля
    }
}
```

#### Создание модуля

1. Создайте файл модуля в директории `modules/`
2. Реализуйте абстрактные методы
3. Добавьте модуль в конфигурацию
4. Перезапустите платформу

#### Пример модуля

```nax
// modules/ExampleModule.nax
import core.modules

class ExampleModule extends Module {
    private var data: Map<String, Object>
    
    constructor() {
        super("ExampleModule", "1.0.0")
        this.setDescription("Пример модуля")
        this.data = new Map<String, Object>()
    }
    
    public function initialize(): void {
        println("Инициализация ExampleModule")
        this.data.put("startTime", getCurrentTime())
    }
    
    public function update(): void {
        // Обновление каждые 10 секунд
        var currentTime = getCurrentTime()
        var startTime = this.data.get("startTime") as long
        
        if (currentTime - startTime > 10000) {
            println("ExampleModule: обновление данных")
            this.data.put("lastUpdate", currentTime)
        }
    }
    
    public function shutdown(): void {
        println("Завершение ExampleModule")
        this.data.clear()
    }
    
    // Публичные методы для API
    public function getData(): Map<String, Object> {
        return this.data
    }
}
```

## API

### REST API

NOX Platform предоставляет REST API для управления системой.

#### Аутентификация

```bash
# Вход в систему
curl -X POST http://localhost:8080/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username": "admin", "password": "admin123"}'

# Использование токена
curl -H "Authorization: Bearer YOUR_TOKEN" \
  http://localhost:8080/api/v1/system/status
```

#### Основные эндпоинты

**Система:**
- `GET /api/v1/system/status` - статус системы
- `GET /api/v1/system/info` - информация о системе
- `POST /api/v1/system/shutdown` - завершение работы

**Модули:**
- `GET /api/v1/modules` - список модулей
- `POST /api/v1/modules/{name}/enable` - включение модуля
- `POST /api/v1/modules/{name}/disable` - отключение модуля
- `GET /api/v1/modules/{name}/status` - статус модуля

**Сеть:**
- `GET /api/v1/network/interfaces` - сетевые интерфейсы
- `GET /api/v1/network/connections` - активные соединения
- `POST /api/v1/network/connect` - создание соединения

#### Примеры запросов

```bash
# Получение статуса системы
curl http://localhost:8080/api/v1/system/status

# Включение модуля
curl -X POST http://localhost:8080/api/v1/modules/ExampleModule/enable \
  -H "Authorization: Bearer YOUR_TOKEN"

# Создание сетевого соединения
curl -X POST http://localhost:8080/api/v1/network/connect \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"host": "192.168.1.100", "port": 80, "protocol": "TCP"}'
```

## Конфигурация

### Основные настройки

Файл конфигурации: `config/system.conf`

```ini
[System]
name = NOX Platform
version = 3.0.0
debug = false
log_level = INFO

[Network]
default_port = 8080
max_connections = 1000

[API]
enabled = true
port = 8080
auth_enabled = true

[Modules]
auto_load = true
module_directory = modules/
```

### Переменные окружения

- `NOX_CONFIG` - путь к файлу конфигурации
- `NOX_DEBUG` - включение режима отладки
- `NOX_LOG_LEVEL` - уровень логирования
- `NOX_PORT` - порт API сервера

## Разработка

### Создание приложения

1. Создайте новый модуль
2. Реализуйте необходимую функциональность
3. Добавьте API эндпоинты
4. Протестируйте модуль

### Отладка

```bash
# Запуск в режиме отладки
nox --debug --log-level DEBUG

# Просмотр логов
tail -f logs/nox.log
```

### Тестирование

```bash
# Запуск тестов
nax run build.nax test

# Запуск конкретного теста
nax run tests/MyModuleTest.nax
```

## Мониторинг

### Метрики

NOX Platform собирает следующие метрики:

- Использование CPU и памяти
- Количество активных соединений
- Статистика модулей
- Время отклика API

### Алерты

Система может отправлять алерты при:

- Высоком использовании ресурсов
- Ошибках в модулях
- Проблемах с сетью
- Сбоях API

## Безопасность

### Аутентификация

- JWT токены
- Время жизни токенов
- Автоматическое обновление

### Авторизация

- Роли пользователей
- Права доступа к модулям
- Ограничения по IP

### Шифрование

- SSL/TLS для API
- Шифрование конфигурации
- Безопасное хранение паролей

## Устранение неполадок

### Частые проблемы

1. **Модуль не загружается**
   - Проверьте зависимости
   - Убедитесь в правильности синтаксиса
   - Проверьте логи

2. **API недоступен**
   - Проверьте порт
   - Убедитесь, что сервер запущен
   - Проверьте файрвол

3. **Высокое использование памяти**
   - Проверьте модули на утечки
   - Увеличьте лимиты памяти
   - Перезапустите систему

### Логи

Логи находятся в директории `logs/`:

- `nox.log` - основные логи
- `error.log` - ошибки
- `access.log` - доступ к API

### Диагностика

```bash
# Проверка статуса системы
curl http://localhost:8080/api/v1/system/status

# Проверка модулей
curl http://localhost:8080/api/v1/modules

# Проверка сети
curl http://localhost:8080/api/v1/network/interfaces
```

## Производительность

### Оптимизация

1. **Настройка памяти**
   - Увеличьте лимиты для больших нагрузок
   - Настройте сборщик мусора

2. **Сетевые настройки**
   - Оптимизируйте размер буферов
   - Настройте keep-alive

3. **Модули**
   - Отключите ненужные модули
   - Оптимизируйте циклы обновления

### Бенчмарки

```bash
# Тест производительности API
ab -n 1000 -c 10 http://localhost:8080/api/v1/system/status

# Тест сетевых соединений
netcat -z localhost 8080
```

## Расширение

### Создание собственных модулей

1. Изучите API модулей
2. Создайте базовую структуру
3. Реализуйте функциональность
4. Добавьте тесты
5. Документируйте модуль

### Интеграция с внешними системами

- REST API для интеграции
- WebSocket для real-time данных
- Плагины для расширения функциональности

## Поддержка

### Документация

- [Руководство разработчика](DEVELOPER.md)
- [API Reference](API.md)
- [Примеры](EXAMPLES.md)

### Сообщество

- GitHub Issues для багов
- GitHub Discussions для вопросов
- Wiki для документации

### Лицензия

NOX Platform распространяется под лицензией MIT. 