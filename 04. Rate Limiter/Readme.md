# Глава 4: Проектирование ограничителя скорости (Rate Limiter)

## Введение
В этой главе рассматривается проектирование и реализация ограничителя скорости — системного компонента для управления частотой запросов от клиентов или сервисов. Ограничители скорости важны для предотвращения злоупотреблений, снижения затрат и обеспечения стабильности серверных ресурсов. Примеры использования включают ограничение постов, создание аккаунтов и получение наград.

## Преимущества ограничения скорости
- **Предотвращение DoS-атак:** Блокировка лишних запросов для защиты ресурсов.
- **Снижение затрат:** Ограничение ненужных запросов для уменьшения нагрузки на сервер.
- **Предотвращение перегрузок:** Отсечение избыточных запросов для стабилизации работы сервера.

## Шаг 1: Понимание задачи
### Основные функции
- Ограничение частоты запросов на стороне сервера (API).
- Поддержка нескольких правил ограничения.
- Работа в масштабируемых распределённых системах.
- Возможность реализации как отдельной службы или в виде кода на уровне приложения.
- Уведомление пользователей о превышении лимитов.

### Требования
- Точное ограничение запросов.
- Минимальная задержка.
- Низкое потребление памяти.
- Работа в распределённой среде.
- Чёткая обработка исключений.
- Высокая отказоустойчивость.

## Шаг 2: Архитектура высокого уровня
### Варианты размещения
<div style="margin-left:2rem">
    <img src="./images/rate_limiter_architecture.png"  alt="Архитектура middleware для ограничения скорости" width="550">
</div>

1. **Реализация на стороне клиента:** Ненадёжна из-за возможности обхода.
2. **Реализация на стороне сервера:** Предпочтительный подход для контроля и надёжности.
3. **Middleware (API-шлюз):** Гибкий вариант для встроенного ограничения скорости.


### Рекомендации по размещению
- Оцените текущий технологический стек и выберите эффективный вариант.
- Выбирайте алгоритмы в зависимости от бизнес-требований.
- Используйте API-шлюз при наличии микросервисов.
- Выбирайте коммерческие решения при ограниченных ресурсах.

## Шаг 3: Алгоритмы ограничения скорости
### 1. Token Bucket (ведро с токенами)
<div style="margin-left:2rem">
  <img src="./images/token-bucket.png"  alt="Алгоритм Token Bucket" width="550">
</div>

- **Описание:** Токены добавляются в ведро с фиксированной скоростью; каждый запрос использует один токен.
- **Параметры:** Размер ведра и скорость пополнения.
- **Плюсы:** Простая реализация, эффективное использование памяти, поддержка всплесков трафика.
- **Минусы:** Требует точной настройки параметров.



### 2. Leaking Bucket (протекающее ведро)
<div style="margin-left:2rem">
  <img src="./images/leaking-bucket.png"  alt="Алгоритм Leaking Bucket" width="550">
</div>

- **Описание:** Обработка запросов с постоянной скоростью с помощью очереди FIFO.
- **Плюсы:** Экономия памяти, стабильная скорость обработки.
- **Минусы:** Всплески трафика могут задерживать новые запросы.


	Пример: https://github.com/uber-go/ratelimit



### 3. Fixed Window Counter (счётчик с фиксированным окном)
<div style="margin-left:2rem">
  <img src="./images/fixed-window-counter.png"  alt="Fixed Window Counter" width="550">
</div>

- **Описание:** Время делится на фиксированные интервалы, и для каждого интервала используется счётчик.
- **Плюсы:** Простота, эффективность в ограниченных сценариях.
- **Минусы:** Всплески в границах окон могут привести к превышению лимита.


<img src="./images/fixed-window-issue.png"  alt="Проблема Fixed Window" width="550">


### 4. Sliding Window Log (скользящее окно с логами)
<div style="margin-left:2rem">
  <img src="./images/sliding-window-log.png"  alt="Sliding Window Log" width="550">
</div>

- **Описание:** Хранятся временные метки запросов в скользящем окне.
- **Плюсы:** Высокая точность ограничения.
- **Минусы:** Большое потребление памяти.



### 5. Sliding Window Counter (счётчик в скользящем окне)
<div style="margin-left:2rem">
  <img src="./images/sliding-window-counter.png"  alt="Sliding Window Counter" width="550">
</div>

- **Описание:** Комбинация фиксированных окон и скользящих логов для сглаживания пиков.
- **Плюсы:** Экономия памяти, устойчивость к всплескам трафика.
- **Минусы:** Ограничение неточное, приближённое.




## Архитектура высокого уровня
<div style="margin-left:2rem">
  <img src="./images/architecture.png" style="margin-left: 40px; margin-top: 40px; margin-bottom: 20px;" alt="Архитектура" width="550">
</div>

- **Хранение данных:** Используется in-memory кэш (например, Redis) для быстрого доступа к счётчикам.
- **Этапы:**
  1. Клиент отправляет запрос в middleware.
  2. Middleware проверяет счётчики в Redis.
  3. Запрос обрабатывается или отклоняется в зависимости от лимитов.


## Продвинутые аспекты
### Распределённые системы
- **Проблемы:** Состояния гонки, сложности синхронизации.
- **Решения:** Использование блокировок, Lua-скриптов или Sorted Set в Redis. Централизованные хранилища данных для синхронизации.

### Оптимизация производительности
- Размещение в нескольких дата-центрах для снижения задержек.
- Использование моделей eventual consistency для синхронизации.

### Мониторинг
- Регулярный анализ для оценки эффективности алгоритма и корректировки правил.

