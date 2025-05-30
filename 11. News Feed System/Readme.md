# Глава 11: Проектирование системы новостной ленты

## Введение
**Система новостной ленты** отображает постоянно обновляемый список публикаций (обновления статуса, фото, видео и ссылки) от связей пользователя. Примеры включают новостную ленту Facebook, ленту Instagram и таймлайн Twitter. В этой главе рассматривается проектирование масштабируемой системы новостной ленты.

---

## Шаг 1: Понимание задачи

### Требования
1. **Платформа:** Система поддерживает веб- и мобильные приложения.
2. **Функции:**
   - Пользователи могут публиковать записи.
   - Пользователи могут просматривать записи друзей в своей ленте.
3. **Сортировка:** Лента сортируется в **обратном хронологическом порядке** для простоты.
4. **Масштаб:**
   - У пользователя может быть до 5000 друзей.
   - 10 миллионов активных пользователей в день (DAU).
   - Лента может содержать текст, изображения и видео.

---

## Шаг 2: Высокоуровневый дизайн

### Обзор
Дизайн включает два основных потока:
1. **Публикация в ленту:** пользователь публикует запись, которая записывается в базу данных и распространяется по лентам друзей.
2. **Формирование ленты новостей:** пользователь получает свою ленту, собирая записи друзей в обратном хронологическом порядке.

---

### API новостной ленты
1. **API публикации:**
   - **Endpoint:** `POST /v1/me/feed`
   - **Параметры:** `content` (текст записи) и `auth_token` (токен аутентификации).

2. **API получения ленты:**
   - **Endpoint:** `GET /v1/me/feed`
   - **Параметры:** `auth_token` (токен аутентификации).

---

### Публикация в ленту

<div style="margin-left:3rem">
   <img src="./images/feed-publishing.png" alt="Публикация в ленту" width="400">
</div>

1. **Взаимодействие пользователя:** пользователь публикует запись через API публикации.
2. **Балансировщик нагрузки:** распределяет трафик на веб-серверы.
3. **Веб-серверы:** аутентификация запросов и перенаправление к сервисам.
4. **Сервис постов:** сохраняет запись в базе данных и кэше.
5. **Сервис фанаута:** распространяет запись в кэше лент друзей.
6. **Сервис уведомлений:** отправляет уведомления друзьям.

---

### Формирование новостной ленты

<div style="margin-left:3rem">
   <img src="./images/news-feed-building.png" alt="Формирование ленты новостей" width="400">
</div>

1. **Взаимодействие пользователя:** пользователь запрашивает свою ленту через API получения.
2. **Балансировщик нагрузки:** распределяет трафик на веб-серверы.
3. **Веб-серверы:** перенаправляют запросы к сервису новостной ленты.
4. **Сервис новостной ленты:** извлекает идентификаторы записей из кэша ленты и получает детали записей из базы данных или кэша.

   
---

## Шаг 3: Подробное проектирование

### Глубокий обзор публикации
1. **Веб-серверы:**
   - Аутентификация пользователей по `auth_token`.
   - Ограничение частоты запросов для предотвращения спама.

2. **Сервис фанаута:**
   - **Fanout при записи:** push-модель, отправляет записи в ленты друзей при публикации.
     - **Плюсы:** обновления в реальном времени, быстрая выдача ленты.
     - **Минусы:** ресурсоёмко для пользователей с большим числом друзей.
   - **Fanout при чтении:** pull-модель, собирает записи при получении ленты.
     - **Плюсы:** эффективно для неактивных пользователей.
     - **Минусы:** медленнее выдача ленты.
   - **Гибридный подход:** push для большинства пользователей и pull для пользователей с большим числом связей (например, знаменитостей).

    <img src="./images/feed-publishing-deep-dive.png" alt="Глубокий обзор публикации" width="500">

    **Сервис фанаута** работает следующим образом:

    1. **Получение списка друзей:** извлечение списка друзей из графовой базы данных.
    2. **Фильтрация друзей из кэша:** доступ к настройкам пользователя в кэше для исключения некоторых друзей (например, отключённых или избранных).
    3. **Отправка в очередь сообщений:** отправка списка друзей и ID новой записи в очередь сообщений для обработки.
    4. **Рабочие фанаута:** получают данные из очереди и обновляют кэш ленты. В кэше хранятся сопоставления `<post_id, user_id>`, а не полные объекты.
    5. **Сохранение в кэше ленты:** добавление новых ID записей в кэш ленты друзей. Ограничение по количеству сохраняемых записей делает потребление памяти управляемым.

    <img src="./images/fanout-service.png" alt="Сервис фанаута" width="500">

## Глубокий обзор получения ленты

### Архитектура кэша
Кэш разделён на пять слоёв:
1. **Кэш ленты:** хранит ID записей для быстрой выдачи.
2. **Кэш контента:** хранит детали записей (популярные записи в hot cache).
3. **Кэш социальной графа:** хранит данные о связях пользователей.
4. **Кэш действий:** отслеживает действия пользователей (лайки, ответы, репосты).
5. **Кэш счётчиков:** хранит счётчики лайков, ответов, подписчиков и т.д.

    <img src="./images/cache-architecture.png" alt="Архитектура кэша" width="500">
---

## Ключевые оптимизации

### Масштабирование
1. **Масштабирование базы данных:**
   - Горизонтальное масштабирование и шардирование.
   - Использование реплик для высоконагруженных запросов.
2. **Статeless веб-слой:** веб-серверы без хранения состояния для горизонтального масштабирования.

### Кэширование
1. Часто запрашиваемые данные хранятся в памяти.
2. Использование слоёв кэша для снижения задержек и нагрузки на базу.

### Надёжность
1. **Консистентное хеширование:** равномерное распределение запросов между серверами.
2. **Очереди сообщений:** декуплирование компонентов системы и буферизация трафика.

### Мониторинг
1. Отслеживание ключевых метрик: QPS (запросы в секунду) и задержка.
2. Мониторинг коэффициента попаданий в кэш и корректировка конфигураций.
