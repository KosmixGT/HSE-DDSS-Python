# Отчет по практической работе №2 - Разделение на сервисы

## Выполненные задачи

### 1. Деление серверной части на сервисы
#### Архитектура системы
Монолитное приложение было разделено на следующие микросервисы:

1.	**Gateway Service** (server/)
-	Единая точка входа для клиентов
-	Маршрутизация запросов к нужным сервисам
2.  **Auth Service** (services/auth/)
-   Аутентификация и авторизация пользователей
-   Управление пользователями и ролями
-   JWT токены для безопасности
3.	**Mailing Service** (services/mailing/)
-	Управление рассылками
-	Работа с получателями и адресами
-	Публикация сообщений в RabbitMQ
4.	**Delivery Service** (services/delivery/)
-	Обработка очереди сообщений
-	Отправка через SMTP и Telegram
-	Consumer для RabbitMQ
5.	**History Service** (services/history/)
-	История рассылок
-   Статистика отправок

#### Clean Architecture в микросервисах
Каждый сервис сохраняет принципы Clean Architecture:
```bash
services/mailing/app/
├── api/v1/            # API слой
├── application/       # Application слой
│   ├── dto/          
│   └── services/     
├── domain/           # Domain слой
│   ├── interfaces/   
│   └── models/       
└── infrastructure/   # Infrastructure слой
    ├── database/     
    └── repositories/ 
```
#### Общие компоненты
Создан shared модуль (services/shared/):
``` bash
mailforge_shared/
├── core/
│   ├── config/       # Общие настройки
│   ├── infrastructure/  # RabbitMQ, DB
│   └── interfaces/   # Базовые интерфейсы
```
#### Docker интеграция
Сервисы запускаются через docker-compose:
```bash
services:
  server:
    build: ./server
  mailing:
    build: ./services/mailing
  delivery:
    build: ./services/delivery
  history:
    build: ./services/history
```
#### Преимущества микросервисной архитектуры
1.	Независимое масштабирование сервисов
2.	Изолированная кодовая база
3.	Возможность использования разных технологий
4.	Упрощенное тестирование и деплой  
Каждый сервис сохраняет принципы Clean Architecture, но теперь работает независимо и взаимодействует через API и очереди сообщений.



### 2. Реализация асинхронного взаимодействия через RabbitMQ
#### Архитектура решения  
В системе реализовано асинхронное взаимодействие между сервисами через брокер сообщений RabbitMQ.

#### Сервисы:

Mailing Service - создает рассылки и публикует их в очередь  
Delivery Service - получает рассылки из очереди и выполняет отправку  
RabbitMQ - брокер сообщений для асинхронного взаимодействия  

#### Реализация Publisher (Mailing Service)  
В сервисе Mailing реализована публикация сообщений в очередь:

```python
message_queue = RabbitMQQueue("amqp://guest:guest@rabbitmq:5672/")
```

При создании рассылки, данные публикуются в очередь mailings.created:

```python
await self.queue.publish("mailings.created", message_data)
```

#### Реализация Consumer (Delivery Service)  
В Delivery сервисе реализован подписчик на очередь:

```python
async def start_consumer():
    connection = await connect_robust("amqp://delivery:delivery@rabbitmq:5672/")
    channel = await connection.channel()
    queue = await channel.declare_queue("mailings.created")
    await queue.consume(process_mailing)
```

Обработка сообщений:

```python
async def process_mailing(message):
    data = json.loads(message.body.decode())
    await delivery_service.DeliveryService().send_mailing(data)
    await message.ack()
```

#### Настройка в Docker
Сервисы связаны через docker-compose:

```yaml
services:
  mailing:
    depends_on:
      - rabbitmq
  delivery:
    depends_on:
      - rabbitmq
  rabbitmq:
    image: rabbitmq:management
```


#### Преимущества решения
1. Асинхронная обработка рассылок
2. Надежная доставка сообщений
3. Масштабируемость сервисов
4. Отказоустойчивость при ошибках

#### Результат
Система успешно отправляет рассылки через очередь сообщений. Delivery сервис получает и обрабатывает их асинхронно, что позволяет масштабировать обработку при необходимости.

3. 

4. 

### 


### Примеры кода

1. 

2. 

### 

### Тестирование
- 
- 

### Заключение
Проект успешно переведен на новую архитектуру с четким разделением слоев. Внедрены асинхронные операции для работы с базой данных. Основные компоненты системы теперь независимы и легко тестируемы.

Весь программный код системы доступен в репозитории GitHub - [Ссылка!](https://github.com/KosmixGT/mailforge)