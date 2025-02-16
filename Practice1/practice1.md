# Отчет по практической работе №1
## Распределенные системы

### Выполненные задачи

1. **Реструктуризация архитектуры проекта**
   - Произведено разделение на четкие архитектурные слои:
     ```
     app/
     ├── api/           # API endpoints
     ├── application/   # Сервисы и DTO
     ├── domain/        # Бизнес-логика и интерфейсы
     └── infrastructure/# Реализация репозиториев и утилиты
     ```

2. **Внедрение асинхронных операций**
   - FastAPI уже поддерживает асинхронные операции
   - Добавлены асинхронные репозитории для работы с БД
   - Пример асинхронного репозитория:

     ```python
     class PostgresMailingRepository(MailingRepository):
         async def get_by_id(self, id: int) -> Mailing:
             query = select(MailingModel).where(MailingModel.id == id)
             result = await self.session.execute(query)
             return result.scalar_one_or_none()
     ```

3. **Работа с базой данных**
   - Использована PostgreSQL как основная СУБД
   - Реализован паттерн Repository для абстракции работы с данными
   - Добавлены интерфейсы репозиториев в domain слое

4. **Контейнеризация**
   - Dockerfile и docker-compose уже присутствовали в проекте
   - Настроена работа с PostgreSQL в контейнере

### Архитектурные решения

1. **Отказ от CQRS**
   - Текущая система не требует разделения операций чтения и записи
   - Нагрузка на чтение и запись распределена равномерно
   - Используется подход с ValueObject и DTO для передачи данных

2. **Слои приложения**
   - API: Обработка HTTP запросов
   - Application: Сервисы и преобразование данных
   - Domain: Бизнес-логика и модели
   - Infrastructure: Реализация репозиториев и внешние сервисы

### Примеры кода

1. **API Layer**
```python
@router.get("/{mailing_id}")
async def get_mailing(mailing_id: int, 
                     mailing_service: MailingService = Depends()):
    return await mailing_service.get_mailing(mailing_id)
```

2. **Service Layer**
```python
class MailingService:
    async def get_mailing(self, mailing_id: int) -> MailingDTO:
        mailing = await self.repository.get_by_id(mailing_id)
        return MailingDTO.from_domain(mailing)
```

### Добавление проверки кода

В проект добавлены инструменты для автоматической проверки кода:

1. **Что установили**
   - Flake8 - ищет ошибки в коде
   - Black - красиво форматирует код
   - Mypy - проверяет правильность типов данных

2. **Как запустить**
   ```bash
   # Проверить типы
   mypy app
   
   # Проверить стиль кода
   flake8 app
   
   # Отформатировать код
   black app
    ```

### Тестирование
- Добавлены базовые unit-тесты
- Настроен pytest для тестирования

### Заключение
Проект успешно переведен на новую архитектуру с четким разделением слоев. Внедрены асинхронные операции для работы с базой данных. Основные компоненты системы теперь независимы и легко тестируемы.

Весь программный код системы доступен в репозитории GitHub - [Ссылка!](https://github.com/KosmixGT/mailforge)