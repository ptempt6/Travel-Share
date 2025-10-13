# Диаграммы последовательностей и активностей

## Диаграмма последовательности 1: Регистрация и восстановление пароля

**Описание:** Процесс регистрации нового пользователя и восстановления пароля

```mermaid
sequenceDiagram
    actor Гость as Гость
    participant UI as UI Layer
    participant Auth as AuthController
    participant Service as UserService
    participant DB as Database
    participant Email as EmailService

    Note over Гость,Email: Основной поток: Вход в систему
    Гость->>UI: Нажимает "Sign In"
    UI->>UI: Показать форму входа
    Гость->>UI: Вводит email и пароль
    UI->>Auth: POST /login
    Auth->>Service: authenticateUser()
    Service->>DB: Проверить учетные данные
    DB-->>Service: Данные пользователя
    Service->>Auth: Успешная аутентификация
    Auth->>UI: Установить сессию, перенаправить
    UI->>Гость: Показать главный экран

    Note over Гость,Email: Альтернативный поток: Регистрация
    Гость->>UI: Нажимает "Sign up here"
    UI->>UI: Показать окно регистрации
    
    Гость->>UI: Вводит данные регистрации
    UI->>Auth: POST /register
    Auth->>Service: registerUser()
    Service->>DB: Проверить уникальность email
    DB-->>Service: Email свободен
    Service->>DB: Сохранить пользователя
    Service->>Auth: Успешная регистрация
    Auth->>UI: Автоматическая аутентификация
    Auth->>UI: Перенаправить на главную
    UI->>Гость: Показать главный экран

    Note over Гость,Email: Альтернативный поток: Восстановление пароля
    Гость->>UI: Нажимает "Forgot password"
    UI->>Auth: GET /password-reset
    Auth->>UI: Показать форму email
    Гость->>UI: Вводит email
    UI->>Auth: POST /password-reset
    Auth->>Service: initiatePasswordReset()
    Service->>DB: Найти пользователя
    Service->>Email: Отправить ссылку сброса
    Email-->>Гость: Письмо со ссылкой
    
    Гость->>UI: Переходит по ссылке
    UI->>Auth: GET /reset-password?token=xxx
    Auth->>UI: Показать форму смены пароля
    Гость->>UI: Вводит новый пароль
    UI->>Auth: POST /reset-password
    Auth->>Service: resetPassword()
    Service->>DB: Обновить пароль
    Service->>Auth: Пароль обновлен
    Auth->>UI: Перенаправить на вход
    UI->>Гость: Показать окно входа
```

## Диаграмма последовательности 2: Поиск и просмотр публичных маршрутов

**Описание:** Процесс поиска маршрутов по фильтрам и просмотра деталей маршрута

```mermaid
sequenceDiagram
    actor Пользователь as Пользователь
    participant UI as UI Layer
    participant RouteCtrl as RouteController
    participant RouteService as RouteService
    participant Search as SearchService
    participant DB as Database

    Пользователь->>UI: Выбирает фильтры/время
    Пользователь->>UI: Вводит ключевые слова
    UI->>RouteCtrl: GET /routes?filters=...
    RouteCtrl->>Search: searchRoutes()
    Search->>DB: Запрос с фильтрами
    DB-->>Search: Список маршрутов
    Search-->>RouteCtrl: Отфильтрованные маршруты
    RouteCtrl-->>UI: Данные для таблицы
    UI->>Пользователь: Показать отфильтрованные маршруты

    Пользователь->>UI: Нажимает "Route page"
    UI->>RouteCtrl: GET /routes/{id}
    RouteCtrl->>RouteService: getRouteDetails()
    RouteService->>DB: Загрузить маршрут + места
    DB-->>RouteService: Данные маршрута
    RouteService->>RouteService: Загрузить информацию о владельце
    RouteService-->>RouteCtrl: Полные данные маршрута
    RouteCtrl-->>UI: Данные для страницы маршрута
    UI->>Пользователь: Показать окно просмотра маршрута
```

## Диаграмма последовательности 3: Создание маршрута с местами

**Описание:** Процесс создания нового маршрута с добавлением мест

```mermaid
sequenceDiagram
    actor Пользователь as Пользователь
    participant UI as UI Layer
    participant RouteCtrl as RouteController
    participant PlaceCtrl as PlaceController
    participant RouteService as RouteService
    participant PlaceService as PlaceService
    participant DB as Database

    Пользователь->>UI: Нажимает "Add Route"
    UI->>UI: Показать окно создания маршрута
    
    Пользователь->>UI: Нажимает "Add Place"
    UI->>UI: Показать окно создания места
    Пользователь->>UI: Заполняет данные места
    UI->>PlaceCtrl: POST /places
    PlaceCtrl->>PlaceService: createPlace()
    PlaceService->>DB: Сохранить место
    DB-->>PlaceService: ID созданного места
    PlaceService-->>PlaceCtrl: Успешное создание
    PlaceCtrl-->>UI: Данные созданного места
    UI->>UI: Добавить место в текущий маршрут
    
    Пользователь->>UI: Нажимает "Create Place"
    UI->>UI: Вернуться к форме маршрута
    
    Пользователь->>UI: Заполняет данные маршрута
    Пользователь->>UI: Нажимает "Create Route"
    UI->>RouteCtrl: POST /routes
    RouteCtrl->>RouteService: createRouteWithPlaces()
    RouteService->>DB: Сохранить маршрут
    RouteService->>DB: Привязать места к маршруту
    DB-->>RouteService: ID созданного маршрута
    RouteService-->>RouteCtrl: Успешное создание
    RouteCtrl-->>UI: Данные созданного маршрута
    UI->>Пользователь: Показать окно просмотра маршрута
```

