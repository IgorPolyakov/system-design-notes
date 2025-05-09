# Глава 6: Проектирование хранилища ключ-значение

## Введение
**Хранилище ключ-значение** — это тип нереляционной базы данных, где данные хранятся в виде пар ключ-значение. Каждый ключ уникален, и значения доступны по этим ключам. В этой главе описано, как спроектировать масштабируемое, отказоустойчивое распределённое хранилище ключ-значение, поддерживающее операции:
- `put(key, value)` — вставка данных.
- `get(key)` — получение данных.

### Особенности дизайна
- Небольшие пары ключ-значение (<10 КБ).
- Поддержка больших объёмов данных с высокой доступностью и масштабируемостью.
- Автоматическое масштабирование и настраиваемая согласованность.
- Низкая задержка.

---

## Хранилище ключ-значение на одном сервере
### Реализация
- Использование **хеш-таблицы** для хранения пар в памяти.
- Оптимизации:
  - Сжатие данных.
  - Сохранение редко используемых данных на диск.

### Ограничение
Память одного сервера ограничена, поэтому требуется **распределённый подход** для масштабирования.

---

## Распределённое хранилище ключ-значение
**Распределённое хранилище** делит данные между серверами и учитывает ограничения **теоремы CAP**.

### Теорема CAP
1. **Согласованность (Consistency):** Все клиенты видят одинаковые данные одновременно.
2. **Доступность (Availability):** Система отвечает на все запросы, даже при отказе узлов.
3. **Устойчивость к разделению (Partition Tolerance):** Работа продолжается при сбоях сети.

**Компромисс:** Можно выбрать только два свойства из трёх.

<p align="center">
  <img src="./images/cap.png" alt="CAP" width="400">
</p>

#### Типы систем:
- **CP-системы:** Согласованность и устойчивость, в ущерб доступности.
- **AP-системы:** Доступность и устойчивость, в ущерб согласованности.
- **CA-системы:** Не существуют в реальности, так как разделения сети неизбежны.

    **Поскольку сбои сети неизбежны, распределённая система должна выдерживать сетевые разделения. Таким образом, CA-системы не могут существовать в реальных приложениях.**

    В распределённой системе разделения неизбежны. Когда происходит разделение, необходимо выбирать между согласованностью и доступностью. Например, если узел n3 выходит из строя, любые данные, записанные на узлы n1 или n2, не могут быть переданы на n3. И наоборот, если данные записаны на n3, но ещё не переданы на n1 и n2, узлы n1 и n2 будут содержать устаревшие данные.

    <p align="center">
    <img src="./images/server-down.png"  alt="Server down" width="400">
    </p>
    
- Если мы выбираем CP-систему, необходимо заблокировать все операции записи на n1 и n2, чтобы избежать несогласованности данных.
- Если мы выбираем AP-систему, система продолжает принимать чтения, даже если они могут возвращать устаревшие данные. 
Для операций записи n1 и n2 продолжают принимать записи, 
а данные будут синхронизированы с n3 после устранения сетевого разделения.

---

## Компоненты системы
### 1. Разделение данных
- **Метод:** Согласованное хеширование.
- **Преимущества:**
  - Автоматическое масштабирование.
  - Использование виртуальных узлов для учёта разной мощности серверов.

### 2. Репликация данных
- Данные дублируются на `N` серверах.
- Реплики размещаются в разных дата-центрах.

    <p align="center">
    <img src="./images/data-replication.png" alt="Data replication" width="300">
    </p>

### 3. Согласованность
Для синхронизации реплик используется:
- **Кворум:**
  - `N` — общее число реплик.
  - `W` — количество реплик для подтверждения записи.
  - `R` — количество реплик для чтения.
  - **Условие:** `W + R > N` гарантирует сильную согласованность.
  - Настройка значений W, R и N представляет собой типичный компромисс между задержкой и согласованностью.

    <p align="center">
    <img src="./images/quorum-consensus.png"   alt="Quorum consensus" width="400">
    </p>

    - Если R = 1 и W = N, система оптимизируется для быстрой операции чтения.
    - Если W = 1 и R = N, система оптимизируется для быстрой операции записи.
    - Если W + R > N, гарантируется сильная согласованность (обычно N = 3, W = R = 2).
    - Если W + R <= N, сильная согласованность не гарантируется.

- **Модели:**
  - **Сильная согласованность:** Чтение возвращает актуальные данные.
  - **Слабая:** Возможен возврат устаревших значений.
  - **Итоговая:** Со временем все реплики синхронизируются.


### 4. Разрешение несогласованностей
Репликация обеспечивает высокую доступность, но вызывает несогласованности между репликами. Для решения этих проблем используются версионирование и векторные часы.
- **Версионирование:**
    - Использовать **векторные часы** для отслеживания версий данных и разрешения конфликтов.
    - Версионирование означает, что каждая модификация данных рассматривается как новая неизменяемая версия.
        <div>
        <img src="./images/consistent-server.png"   alt="Consisten hashing" width="400">
        <img src="./images/inconsistent-server.png"   alt="Inconsistent server" height="230">
        </div>
    - Сервер 1 изменяет имя, и сервер 2 также изменяет имя одновременно. Теперь возникают конфликтующие значения, называемые версиями v1 и v2.

- **Векторные часы**
    1. **Настройка:** Векторные часы — это пара [сервер, версия], связанная с данным элементом. Они используются для проверки, предшествует ли одна версия другой, следует за ней или конфликтует с ней.
        - Предположим, что векторные часы представлены как D([S1, v1], [S2, v2], …, [Sn, vn]). Если элемент данных D записывается на сервер Si, система должна выполнить одно из следующих действий.
        - Где: `D` — это элемент данных, `Si` — идентификатор сервера, `vi` — счётчик версии данных на сервере `Si`.
    2. **Обновление векторных часов:** Когда элемент данных изменяется на сервере:
        - Если сервер присутствует во векторных часах, его счётчик версии увеличивается.
        - В противном случае в векторные часы добавляется новая запись.
    3. **Обнаружение конфликтов:**
        - **Нет конфликта:** Версия X является предком версии Y, если все счётчики в X меньше или равны соответствующим счётчикам в Y.
        - **Есть конфликт:** Две версии являются «сиблингами», если хотя бы один счётчик в Y меньше соответствующего счётчика в X.
    4. **Разрешение конфликтов:** При обнаружении конфликтов (версии-сиблинги) система опирается на логику приложения или вмешательство клиента для согласования данных.
<p align="center">
  <img src="./images/vector-clock.png" alt="Server hashing" width="500">
</p>
- **Проблемы:**
  - Повышенная сложность для клиентов.
  - Размер векторных часов может расти при большом количестве обновлений, что требует стратегий обрезки для ограничения их размера.

### 5. Работа при сбоях

#### a. Обнаружение сбоев
Недостаточно полагаться на то, что сервер считается недоступным только по информации от другого сервера. Обычно требуется как минимум два независимых источника информации, чтобы отметить сервер как недоступный.
- **Протокол сплетни (gossip):**
<div style="margin-left:3rem">
  <img src="./images/gossip-protocol.png" alt="Gossip protocol" width="600">
</div>
    - Каждый узел хранит идентификаторы участников и счётчики «heartbeat».
    - Каждый узел периодически увеличивает свой счётчик heartbeat.
    - Каждый узел периодически отправляет heartbeat набору случайных узлов.
    - Если счётчик heartbeat не увеличивался более заданного времени, узел считается офлайн

#### b. Временные сбои
- **Нечёткий кворум (sloppy quorum):** Использовать доступные узлы для временного поддержания работы.
<p align="center">
  <img src="./images/sloppy-quorum.png" alt="Sloppy Quorum" width="400">
</p>
    - После обнаружения сбоев система должна задействовать определённые механизмы для обеспечения доступности.
    - Вместо строгого требования кворума система выбирает первые W работоспособных серверов для операций записи и первые R работоспособных серверов для чтения на кольце хеширования.
    - Офлайн-серверы игнорируются. Если сервер недоступен, запросы временно обрабатывает другой сервер

- **Отложенная передача (hinted handoff):** Офлайн-серверы получают пропущенные изменения при восстановлении.
    - Когда вышедший из строя сервер вновь становится доступен, изменения будут переданы ему для достижения согласованности данных

#### c. Постоянные сбои
- Использовать **деревья Меркла** для эффективной синхронизации между репликами.
Дерево Меркла (или хеш-дерево) — это структура данных для эффективного обнаружения и разрешения несогласованностей между репликами при постоянных сбоях.

- Принцип работы
    1. **Структура:**
        - **Листовые узлы** хранят хеш отдельных блоков данных.
        - **Внутренние узлы** хранят хеши своих дочерних узлов.
        - **Корневой хеш** отражает объединённое состояние всех данных дерева.

    2. **Построение дерева Меркла:**
        - **Шаг 1:** Разделить пространство ключей на группы (buckets).
            <img src="./images/key-bucket.png" alt="Key Bucket" width="500">
        - **Шаг 2:** Вычислить хеш каждого ключа в группе с помощью равномерного хеширования.
            <img src="./images/hash-key-bucket.png" alt="Hash Key Bucket" width="500">
        - **Шаг 3:** Создать единый хеш для каждой группы.
            <img src="./images/hash-bucket.png" alt="Hash Bucket" width="500">
        - **Шаг 4:** Скомбинировать хеши групп для вычисления хешей более высокого уровня, завершая расчётом корневого хеша.
            <img src="./images/merkel-tree.png" alt="Merkel Tree" width="500">

    3. **Синхронизация:**
        - Для синхронизации двух реплик:
            - Сравнить их корневые хеши.
            - Если корневые хеши совпадают, реплики согласованы.
            - Если корневые хеши отличаются, рекурсивно сравнить хеши дочерних узлов, чтобы найти конфликтующие группы.
        - Синхронизируются только конфликтующие данные.


- **Преимущества:**
    - **Эффективность:** Синхронизируются только конфликтующие данные, что снижает объём передаваемых данных.
    - **Масштабируемость:** Эффективно для больших наборов данных с минимальными накладными расходами на синхронизацию.
    - **Надёжность:** Гарантирует согласованность данных между репликами.


### 6. Сбои дата-центра
- Репликация между дата-центрами обеспечивает доступность.

---

## Пути записи и чтения
### 1. Путь записи (на основе архитектуры Cassandra)
<div style="margin-left:3rem">
  <img src="./images/write-path.png" alt="Hash Bucket" width="500">
</div>
- Зафиксировать запись в **журнале транзакций (commit log)**.
- Сохранить данные во **встроенный кэш (memory cache)**.
- Сбросить данные в **SSTable** (отсортированная таблица строк) на диск, когда кэш заполнится.

### 2. Путь чтения
<div style="margin-left:3rem">
  <img src="./images/read-path.png" alt="Hash Bucket" width="500">
  <img src="./images/read-path-without-cache.png" alt="Hash Bucket" width="500">
</div>
- Проверить **встроенный кэш** на наличие данных.
- Если данных нет, использовать **фильтр Блума** для поиска данных в SSTables.
- Извлечь и вернуть данные.

---

## Итоговая архитектура
<p align="center">
  <img src="./images/final-architecture.png" alt="Hash Bucket" width="500">
</p>

- Клиенты взаимодействуют с хранилищем ключ-значение через простые API: get(key) и put(key, value).
- Координатор — это узел, выступающий посредником между клиентом и хранилищем ключ-значение.
- Узлы распределены по кольцу с использованием консистентного хеширования.
- Система полностью децентрализована, поэтому добавление и перемещение узлов может происходить автоматически.
- Данные реплицируются на нескольких узлах.
- Нет единой точки отказа, так как каждый узел несёт одинаковые обязанности.


