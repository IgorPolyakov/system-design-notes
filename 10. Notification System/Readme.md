# Глава 10: Проектирование системы уведомлений

## Введение
**Система уведомлений** неотъемлемая часть современных приложений, обеспечивая своевременное информирование о новых продуктах, событиях, предложениях и предупреждениях. Уведомления могут отправляться через:
1. **Push-уведомления** (мобильные или десктопные),
2. **SMS-сообщения**, и
3. **Email**.

Глава фокусируется на проектировании масштабируемой системы, способной отправлять миллионы уведомлений ежедневно.

---

## Шаг 1: Понимание задачи
### Требования
- **Типы уведомлений:** Push, SMS, Email.
- **Доставка:**  система с мягкими ограничениями по времени (soft real-time) с минимальными задержками.
- **Платформы:** iOS, Android и десктоп.
- **Триггеры:** уведомления инициируются клиентскими приложениями или запланированы на серверах.
- **Масштаб:**
  - **Push-уведомления:** 10 млн/день,
  - **SMS:** 1 млн/день,
  - **Email:** 5 млн/день.
- **Отказ от подписки:** пользователи могут отключать отдельные типы уведомлений.

---

## Шаг 2: Высокоуровневый дизайн

### Компоненты

1. **Типы уведомлений:**
   - **iOS Push:** Apple Push Notification Service (APNS).
   - **Android Push:** Firebase Cloud Messaging (FCM).
   - **SMS:** Twilio, Nexmo.
   - **Email:** SendGrid, Mailchimp.

2. **Сбор и хранение контактных данных:**
   <div style="margin-left:3rem">
      <img src="./images/contact-info-gathering.png" alt="Contact Info Gathering" width="500">
   </div>

   - Собирайте токены устройств, номера телефонов и Email при установке приложения или регистрации.
   - Сохраняйте данные в базе данных:
     - **Таблица Device Tokens:** для Push.
     - **Таблица Users:** для Email и телефонов.


3. **Поток отправки уведомлений:**

   <div style="margin-left:3rem">
      <img src="./images/high-level-design.png" alt="High Level Design" width="500">
   </div>

   - **Trigger Services:**
      - генерируют события (напр., напоминания о платеже, обновления доставки).
   - **Notification Server:**
     - API для отправки уведомлений.
     - Валидация Email и телефона.
     - Запрос метаданных из кэша/БД.
   - **Third-Party Services:** доставка уведомлений.



### Проблемы первоначального дизайна
- **SPOF:** единая точка отказа.
- **Масштабируемость:** сложно масштабировать БД и кэш.
- **Узкие места:** высокая нагрузка при массовой отправке.

### Улучшенный дизайн
   <div style="margin-left:3rem">
      <img src="./images/improved-design.png" alt="Improved Design" width="500">
   </div>
- Вынести БД и кэш за пределы Notification Server.
- Горизонтальное масштабирование: несколько серверов уведомлений.
- Очереди сообщений для декуплинга.
- Воркеры читают события из очередей и отправляют уведомления.

---

## Шаг 3: Детальный дизайн

### Надёжность
1. **Потеря данных:**
   <div style="margin-left:3rem">
      <img src="./images/data-loss.png" alt="Data Loss" width="400">
   </div>
   - Сохранять уведомления в БД.
   - Реализовать retry-механизм.
   - Хранить лог уведомлений.


2. **Дедупликация:**
   - Проверять ID события.
   - Если событие уже обработано — игнорировать.

### Дополнительные компоненты
   <div style="margin-left:3rem">
      <img src="./images/events-tracking.png" alt="Events Tracking" width="400">
   </div>
1. **Шаблоны:** заранее подготовленные.
2. **Настройки уведомлений:**
   - opt-in/opt-out
   - хранение в отдельной таблице.
3. **Rate limiting:** ограничение частоты отправки.
4. **Retry-механизм:** повтор при сбоях.
5. **Мониторинг очередей:** динамическое масштабирование числа воркеров.
6. **Трекер событий:** метрики открытий, кликов, вовлечённости.

### Безопасность
- AppKey и AppSecret для защиты API Push.

### Поток уведомлений

   <div style="margin-left:3rem">
      <img src="./images/updated-design.png" alt="Updated Design" width="500">
   </div>

1. Trigger Services вызывают API.
2. Notification Servers валидируют и получают метаданные.
3. События отправляются в очередь сообщений.
4. Воркеры обрабатывают события.
5. Сторонние сервисы доставляют уведомления.


---

## Ключевые оптимизации
1. Горизонтальное масштабирование.
2. Очереди сообщений.
3. Кэширование.
4. Географически распределённая доставка.

