# Mock-собес по архитектуре (Week 14)

## Цель
Подготовка к архитектурной секции собеседования (System Design Interview). В этой секции проверяется умение проектировать масштабируемые, надежные и поддерживаемые системы.

---

## Структура ответа на System Design (FRAMEWORK)

### 1. Уточнение требований (Requirements Clarification) - 5-10 мин
Не бросаться сразу рисовать квадраты! Задавать вопросы.
*   **Функциональные требования:** Что система должна делать? (MVP)
    *   Пример: "Пользователь может загружать фото", "Пользователь видит ленту новостей".
*   **Нефункциональные требования:**
    *   **Масштабируемость (Scalability):** DAU/MAU (Daily/Monthly Active Users), RPS (Requests Per Second).
    *   **Производительность (Performance):** Latency (задержка), Throughput (пропускная способность).
    *   **Надежность (Reliability/Availability):** 99.9% vs 99.99% (SLA/SLO).
    *   **Консистентность (Consistency):** Strong vs Eventual (CAP-теорема).

### 2. Оценка нагрузки (Back-of-the-envelope estimation) - опционально
*   Примерный объем данных (хранение фоток, логов).
*   Bandwidth (ширина канала).
*   Помогает выбрать между SQL vs NoSQL, Sharding и т.д.

### 3. Высокоуровневая архитектура (High-Level Design) - 10-15 мин
Рисуем основные блоки системы.
*   **Client** (Mobile/Web)
*   **Load Balancer** (Nginx, AWS ELB) - распределение нагрузки.
*   **API Gateway** - аутентификация, rate limiting, маршрутизация.
*   **Application Servers** (Stateless сервисы).
*   **Database** (SQL vs NoSQL). Выбор базы данных критичен!
    *   *SQL (PostgreSQL, MySQL):* Строгая структура, транзакции (ACID), сложные join-ы.
    *   *NoSQL (MongoDB, Cassandra, Redis):* Гибкая схема, горизонтальное масштабирование, eventual consistency.

### 4. Детальное проектирование (Deep Dive) - 15-20 мин
Углубляемся в специфические компоненты в зависимости от задачи.
*   **Data Model:** Схема базы данных (таблицы, ключи).
*   **API Design:** REST vs GraphQL vs gRPC.
*   **Кэширование (Caching):** Redis/Memcached. Где кэшировать? (CDN, Database, In-memory). Стратегии инвалидации (LRU, TTL).
*   **Очереди сообщений (Message Queues):** Kafka/RabbitMQ для асинхронной обработки (например, генерация превью видео).

### 5. Масштабирование и отказоустойчивость (Scaling & Bottlenecks)
*   **Database Scaling:** Replication (Master-Slave), Sharding (Horizontal Partitioning).
*   **Single Point of Failure (SPOF):** Как избежать? (Redundancy, Failover).
*   **Monitoring & Logging:** ELK Stack, Prometheus, Grafana.

---

## Типовые задачи на System Design

1.  **Design Instagram / Twitter News Feed**
    *   *Focus:* Fan-out service (Push vs Pull model), хранение медиа (S3 + CDN), кэширование ленты.
2.  **Design Messenger (WhatsApp / Telegram)**
    *   *Focus:* WebSockets (real-time), One-on-one vs Group chat, статус "прочитано", хранение истории.
3.  **Design URL Shortener (TinyURL)**
    *   *Focus:* Генерация уникальных ID (Base62, Key Generation Service), высокая скорость чтения (Read-heavy).
4.  **Design Uber / Grab**
    *   *Focus:* Гео-локация (QuadTree / GeoHash), matching алгоритмы, real-time обновления.
5.  **Design YouTube / Netflix**
    *   *Focus:* Хранение больших файлов (Chunking), CDN, адаптивный стриминг (HLS/DASH).

---

## Чек-лист для самопроверки
- [ ] Я задал вопросы по требованиям?
- [ ] Я обосновал выбор базы данных (SQL vs NoSQL)?
- [ ] Я упомянул Load Balancer и CDN?
- [ ] Я рассмотрел сценарии отказа (что если база упадет)?
- [ ] Я обсудил trade-offs (компромиссы) моего решения?

## Полезные ресурсы
*   *System Design Primer* (GitHub)
*   *Grokking the System Design Interview*
*   *High Scalability* (blog)
