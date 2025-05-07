
# [Собеседование по системному дизайну — инсайдерское руководство (Том 1)](https://bytebytego.com/courses/system-design-interview)
Эти заметки основаны на книгах System Design Interview — [Том 1, 2-е издание](https://www.goodreads.com/book/show/54109255-system-design-interview-an-insider-s-guide)


**Примечание:** Эти заметки находятся в стадии разработки.


* [Глава 1 — Масштабирование от нуля до миллионов пользователей](./01.%20Scaling/)
* [Глава 2 — Прикидка «на коленке»](./02.%20Back%20Of%20the%20Envelope%20Estimation/)
* [Глава 3 — Фреймворк для интервью по системному дизайну](./03.%20System%20Design%20Framework/)
* [Глава 4 — Проектирование ограничителя скорости](./04.%20Rate%20Limiter//)
* [Глава 5 — Проектирование консистентного хеширования](./05.%20Consistent%20Hashing/)
* [Глава 6 — Проектирование хранилища ключ-значение](./06.%20Key-Value%20Store/)
* [Глава 7 — Проектирование генератора уникальных идентификаторов в распределённых системах](./07.%20Unique-Id%20Generator/)
* [Глава 8 — Проектирование системы сокращения URL](./08.%20URL%20Shortener/)
* [Глава 9 — Проектирование веб-краулера](./09.%20Web%20Crawler/)
* [Глава 10 — Проектирование системы уведомлений](./10.%20Notification%20System/)
* [Глава 11 — Проектирование системы новостной ленты](./11.%20News%20Feed%20System/)
* [Глава 12 — Проектирование чат-системы](./12.%20Chat%20System/)
* [Глава 13 — Проектирование системы автозаполнения поиска](./13.%20Search%20Autocomplete/)
* [Глава 14 — Проектирование YouTube](./14.%20Youtube/)
* [Глава 15 — Проектирование Google Drive](./15.%20Google%20Drive/)
* [Глава 16 — Служба определения близости](./16.%20Proximity%20Service/)




# Дополнительные ресурсы

### Ограничение скорости
- [Алгоритм Circuit Breaker](https://martinfowler.com/bliki/CircuitBreaker.html)
- [Uber Rate Limiter](https://github.com/uber-go/ratelimit/blob/master/ratelimit.go)

### Консистентное хеширование
- [Консистентное хеширование](https://tom-e-white.com/2007/11/consistent-hashing.html)
- [CS168: Введение и консистентное хеширование](http://theory.stanford.edu/~tim/s16/l/l1.pdf)
- [Apache Cassandra](http://www.cs.cornell.edu/Projects/ladis2009/papers/Lakshman-ladis2009.PDF)
- [Масштабирование Discord](https://blog.discord.com/scaling-elixir-f9b8e1e7c29b)
- [Google Maglev](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/44824.pdf)


### Хранилище ключ-значение
- [Amazon Dynamo](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf)
- [Архитектура Cassandra](https://docs.datastax.com/en/archived/cassandra/3.0/cassandra/architecture/archIntro.html)
- [Архитектура Google BigTable](https://static.googleusercontent.com/media/research.google.com/en//archive/bigtable-osdi06.pdf)
- [Внутреннее устройство Amazon Dynamo DB](https://www.allthingsdistributed.com/2007/10/amazons_dynamo.html)
- [Паттерны проектирования в Amazon Dynamo DB](https://www.youtube.com/watch?v=HaEPXoXVf2k)
- [Внутренности Amazon Dynamo DB](https://www.youtube.com/watch?v=yvBR71D0nAQ)


### Генератор уникальных идентификаторов
- [Ticket Servers: Distributed Unique Primary Keys on the Cheap](https://code.flickr.net/2010/02/08/ticket-servers-distributed-unique-primary-keys-on-the-cheap)
- [Snowflake](https://blog.twitter.com/engineering/en_us/a/2010/announcing-snowflake.html)


### Веб-краулер
- [Web Crawling](http://infolab.stanford.edu/~olston/publications/crawling_survey.pdf)
- [Google Dynamic Rendering](https://developers.google.com/search/docs/guides/dynamic-rendering)



### Чат-системы
- [Как Discord хранит миллиарды сообщений](https://discord.com/blog/how-discord-stores-billions-of-messages)
- [Flannel: кэширование на уровне приложения в Slack для масштабирования](https://slack.engineering/flannel-an-application-level-edge-cache-to-make-slack-scale/)


### Автозаполнение поиска
- [Как мы создали Prefixy](https://medium.com/@prefixyteam/how-we-built-prefixy-a-scalable-prefix-search-service-for-powering-autocomplete-c20f98e2eff1)
- [Prefix Hash Tree](https://people.eecs.berkeley.edu/~sylvia/papers/pht.pdf)


### YouTube
- [Архитектура YouTube](http://highscalability.com/youtube-architecture)
- [Масштабируемость YouTube 2012](https://www.youtube.com/watch?v=w5WVu624fY8)
- [Транскодирование видео в масштабе](https://www.egnyte.com/blog/2018/12/transcoding-how-we-serve-videos-at-scale/)
- [Видео-трансляции Facebook](https://engineering.fb.com/ios/under-the-hood-broadcasting-live-video-to-millions/)
- [Кодирование видео Netflix в масштабе](https://netflixtechblog.com/high-quality-video-encoding-at-scale-d159db052746)
- [Shot-based encoding Netflix](https://netflixtechblog.com/optimized-shot-based-encodes-now-streaming-4b9464204830)


### Google Drive
- [Differential Synchronization](https://neil.fraser.name/writing/sync/)
- [Видео по Differential Synchronization](https://www.youtube.com/watch?v=S2Hp_1jqpY8)
- [Как мы масштабировали Dropbox](https://www.youtube.com/watch?v=PE4gwstWhmc&feature=youtu.be)
