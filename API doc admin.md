## Для администратора

### 24. Получение dashboard

`GET /admin/dashboard`

Описание:
Возвращает сводную статистику по системе.

Headers
- `Authorization: Bearer <token>`

Response 200 OK
```json
{
  "usersCount": "int",
  "studentsCount": "int",
  "teachersCount": "int",
  "groupsCount": "int",
  "subjectsCount": "int",
  "lessonsCount": "int",
  "notificationsCount": "int",
  "deviceTokensCount": "int"
}
```

Response codes
- `200 OK` — статистика получена
- `401 Unauthorized` — пользователь не авторизован
- `403 Forbidden` — доступ только для администратора
- `500 Internal Server Error` — ошибка сервера

Пример запроса
```http
GET /admin/dashboard
```

Пример ответа
```json
{
  "usersCount": 412,
  "studentsCount": 360,
  "teachersCount": 38,
  "groupsCount": 16,
  "subjectsCount": 42,
  "lessonsCount": 728,
  "notificationsCount": 1540,
  "deviceTokensCount": 297
}
```

### 25. Получение списка пользователей

`GET /admin/users`

Описание:
Возвращает список пользователей системы.

Query params
```json
{
  "roleName": "string|null",
  "search": "string|null",
  "limit": "int|null",
  "offset": "int|null"
}
```

Headers
- `Authorization: Bearer <token>`

Response 200 OK
```json
{
  "items": [
    {
      "id": "uuid",
      "roleId": "uuid",
      "roleName": "string",
      "firstName": "string",
      "lastName": "string",
      "fatherName": "string|null",
      "email": "string",
      "phone": "string|null"
    }
  ],
  "total": "int"
}
```

Response codes
- `200 OK` — список пользователей получен
- `401 Unauthorized` — пользователь не авторизован
- `403 Forbidden` — доступ только для администратора
- `500 Internal Server Error` — ошибка сервера

Пример запроса
```http
GET /admin/users?roleName=student&limit=20&offset=0
```

Пример ответа
```json
{
  "items": [
    {
      "id": "7c8c8d9a-1234-4567-8901-aabbccddeeff",
      "roleId": "21b2d50d-8f11-4d0a-b88a-11f111111111",
      "roleName": "student",
      "firstName": "Тимур",
      "lastName": "Закиров",
      "fatherName": "Салаватович",
      "email": "timur@stud.kpfu.ru",
      "phone": "+79001234567"
    }
  ],
  "total": 1
}
```

### 26. Создание пользователя

`POST /admin/users`

Описание:
Создает нового пользователя. Дополнительные поля профиля зависят от роли.

Headers
- `Authorization: Bearer <token>`
- `Content-Type: application/json`

Request body
```json
{
  "password": "string",
  "roleId": "uuid",
  "firstName": "string",
  "lastName": "string",
  "fatherName": "string|null",
  "email": "string",
  "phone": "string|null",
  "groupId": "uuid|null"
}
```

Response 201 Created
```json
{
  "id": "uuid",
  "roleId": "uuid",
  "roleName": "string",
  "firstName": "string",
  "lastName": "string",
  "fatherName": "string|null",
  "email": "string",
  "phone": "string|null"
}
```

Response codes
- `201 Created` — пользователь создан
- `400 Bad Request` — неверные данные в запросе
- `401 Unauthorized` — пользователь не авторизован
- `403 Forbidden` — доступ только для администратора
- `500 Internal Server Error` — ошибка сервера

Пример запроса
```json
{
  "password": "123456",
  "roleId": "21b2d50d-8f11-4d0a-b88a-11f111111111",
  "firstName": "Алина",
  "lastName": "Петрова",
  "fatherName": "Романовна",
  "email": "petrova@stud.kpfu.ru",
  "phone": "+79001112233",
  "groupId": "33333333-aaaa-bbbb-cccc-333333333333"
}
```

Пример ответа
```json
{
  "id": "18181818-aaaa-bbbb-cccc-181818181818",
  "roleId": "21b2d50d-8f11-4d0a-b88a-11f111111111",
  "roleName": "student",
  "firstName": "Алина",
  "lastName": "Петрова",
  "fatherName": "Романовна",
  "email": "petrova@stud.kpfu.ru",
  "phone": "+79001112233"
}
```

### 27. Изменение пользователя

`PATCH /admin/users/{userId}`

Описание:
Обновляет данные пользователя и его профиля.

Path params
```json
{
  "userId": "uuid"
}
```

Headers
- `Authorization: Bearer <token>`
- `Content-Type: application/json`

Request body
```json
{
  "firstName": "string",
  "lastName": "string",
  "fatherName": "string|null",
  "email": "string",
  "phone": "string|null",
  "groupId": "uuid|null"
}
```

Response 200 OK
```json
{
  "id": "uuid",
  "roleId": "uuid",
  "roleName": "string",
  "firstName": "string",
  "lastName": "string",
  "fatherName": "string|null",
  "email": "string",
  "phone": "string|null"
}
```

Response codes
- `200 OK` — пользователь обновлен
- `400 Bad Request` — неверные данные в запросе
- `401 Unauthorized` — пользователь не авторизован
- `403 Forbidden` — доступ только для администратора
- `404 Not Found` — пользователь не найден
- `500 Internal Server Error` — ошибка сервера

Пример запроса
```json
{
  "phone": "+79005556677"
}
```

Пример ответа
```json
{
  "id": "aaaa1111-2222-3333-4444-bbbbbbbbbbbb",
  "roleId": "31b2d50d-8f11-4d0a-b88a-22f222222222",
  "roleName": "teacher",
  "firstName": "Тимур",
  "lastName": "Закиров",
  "fatherName": "Салаватович",
  "email": "t.zakirov@kpfu.ru",
  "phone": "+79005556677"
}
```

### 28. Создание группы

`POST /admin/groups`

Описание:
Создает новую учебную группу.

Headers
- `Authorization: Bearer <token>`
- `Content-Type: application/json`

Request body
```json
{
  "name": "string"
}
```

Response 201 Created
```json
{
  "id": "uuid",
  "name": "string"
}
```

Response codes
- `201 Created` — группа создана
- `400 Bad Request` — неверные данные в запросе
- `401 Unauthorized` — пользователь не авторизован
- `403 Forbidden` — доступ только для администратора
- `500 Internal Server Error` — ошибка сервера

Пример запроса
```json
{
  "name": "09-453"
}
```

Пример ответа
```json
{
  "id": "19191919-aaaa-bbbb-cccc-191919191919",
  "name": "09-453"
}
```

### 29. Изменение группы

`PATCH /admin/groups/{groupId}`

Описание:
Обновляет данные учебной группы.

Path params
```json
{
  "groupId": "uuid"
}
```

Headers
- `Authorization: Bearer <token>`
- `Content-Type: application/json`

Request body
```json
{
  "name": "string"
}
```

Response 200 OK
```json
{
  "id": "uuid",
  "name": "string"
}
```

Response codes
- `200 OK` — группа обновлена
- `400 Bad Request` — неверные данные в запросе
- `401 Unauthorized` — пользователь не авторизован
- `403 Forbidden` — доступ только для администратора
- `404 Not Found` — группа не найдена
- `500 Internal Server Error` — ошибка сервера

Пример запроса
```json
{
  "name": "09-452"
}
```

Пример ответа
```json
{
  "id": "33333333-aaaa-bbbb-cccc-333333333333",
  "name": "09-452"
}
```

### 30. Создание предмета

`POST /admin/subjects`

Описание:
Создает новый предмет.

Headers
- `Authorization: Bearer <token>`
- `Content-Type: application/json`

Request body
```json
{
  "name": "string"
}
```

Response 201 Created
```json
{
  "id": "uuid",
  "name": "string"
}
```

Response codes
- `201 Created` — предмет создан
- `400 Bad Request` — неверные данные в запросе
- `401 Unauthorized` — пользователь не авторизован
- `403 Forbidden` — доступ только для администратора
- `500 Internal Server Error` — ошибка сервера

Пример запроса
```json
{
  "name": "Архитектура информационных систем"
}
```

Пример ответа
```json
{
  "id": "20202020-aaaa-bbbb-cccc-202020202020",
  "name": "Архитектура информационных систем"
}
```

### 31. Изменение предмета

`PATCH /admin/subjects/{subjectId}`

Описание:
Обновляет данные предмета.

Path params
```json
{
  "subjectId": "uuid"
}
```

Headers
- `Authorization: Bearer <token>`
- `Content-Type: application/json`

Request body
```json
{
  "name": "string"
}
```

Response 200 OK
```json
{
  "id": "uuid",
  "name": "string"
}
```

Response codes
- `200 OK` — предмет обновлен
- `400 Bad Request` — неверные данные в запросе
- `401 Unauthorized` — пользователь не авторизован
- `403 Forbidden` — доступ только для администратора
- `404 Not Found` — предмет не найден
- `500 Internal Server Error` — ошибка сервера

Пример запроса
```json
{
  "name": "Базы данных"
}
```

Пример ответа
```json
{
  "id": "11111111-aaaa-bbbb-cccc-111111111111",
  "name": "Базы данных"
}
```

### 32. Получение расписания для управления

`GET /admin/lessons`

Описание:
Возвращает список занятий для управления расписанием.

Query params
```json
{
  "dateFrom": "string|null (YYYY-MM-DD)",
  "dateTo": "string|null (YYYY-MM-DD)",
  "groupId": "uuid|null",
  "teacherId": "uuid|null"
}
```

Headers
- `Authorization: Bearer <token>`

Response 200 OK
```json
{
  "items": [
    {
      "lessonId": "uuid",
      "subjectId": "uuid",
      "subjectName": "string",
      "teacherId": "uuid",
      "teacherFirstName": "string",
      "teacherLastName": "string",
      "teacherFatherName": "string|null",
      "groupId": "uuid",
      "groupName": "string",
      "date": "string",
      "cabinet": "string|null",
      "type": "string",
      "startsAt": "string",
      "endsAt": "string"
    }
  ],
  "total": "int"
}
```

Response codes
- `200 OK` — расписание получено
- `401 Unauthorized` — пользователь не авторизован
- `403 Forbidden` — доступ только для администратора
- `500 Internal Server Error` — ошибка сервера

Пример запроса
```http
GET /admin/lessons?dateFrom=2026-04-14&dateTo=2026-04-20&groupId=33333333-aaaa-bbbb-cccc-333333333333
```

Пример ответа
```json
{
  "items": [
    {
      "lessonId": "b1b1b1b1-1111-2222-3333-444444444444",
      "subjectId": "11111111-aaaa-bbbb-cccc-111111111111",
      "subjectName": "Базы данных",
      "teacherId": "22222222-aaaa-bbbb-cccc-222222222222",
      "teacherFirstName": "Ринат",
      "teacherLastName": "Сафрутдинов",
      "teacherFatherName": "Наилевич",
      "groupId": "33333333-aaaa-bbbb-cccc-333333333333",
      "groupName": "09-352",
      "date": "2026-04-16",
      "cabinet": "1101",
      "type": "lecture",
      "startsAt": "10:20",
      "endsAt": "11:50"
    }
  ],
  "total": 1
}
```

### 33. Создание занятия в расписании

`POST /admin/lessons`

Описание:
Создает новое занятие в расписании.

Headers
- `Authorization: Bearer <token>`
- `Content-Type: application/json`

Request body
```json
{
  "subjectId": "uuid",
  "teacherId": "uuid",
  "groupId": "uuid",
  "date": "string",
  "cabinet": "string|null",
  "type": "string",
  "startsAt": "string",
  "endsAt": "string"
}
```

Response 201 Created
```json
{
  "lessonId": "uuid",
  "subjectId": "uuid",
  "teacherId": "uuid",
  "groupId": "uuid",
  "date": "string",
  "cabinet": "string|null",
  "type": "string",
  "startsAt": "string",
  "endsAt": "string"
}
```

Response codes
- `201 Created` — занятие создано
- `400 Bad Request` — неверные данные в запросе
- `401 Unauthorized` — пользователь не авторизован
- `403 Forbidden` — доступ только для администратора
- `404 Not Found` — группа, преподаватель или предмет не найдены
- `500 Internal Server Error` — ошибка сервера

Пример запроса
```json
{
  "subjectId": "11111111-aaaa-bbbb-cccc-111111111111",
  "teacherId": "22222222-aaaa-bbbb-cccc-222222222222",
  "groupId": "33333333-aaaa-bbbb-cccc-333333333333",
  "date": "2026-04-22",
  "cabinet": "1101",
  "type": "lecture",
  "startsAt": "10:20",
  "endsAt": "11:50"
}
```

Пример ответа
```json
{
  "lessonId": "21212121-aaaa-bbbb-cccc-212121212121",
  "subjectId": "11111111-aaaa-bbbb-cccc-111111111111",
  "teacherId": "22222222-aaaa-bbbb-cccc-222222222222",
  "groupId": "33333333-aaaa-bbbb-cccc-333333333333",
  "date": "2026-04-22",
  "cabinet": "1101",
  "type": "lecture",
  "startsAt": "10:20",
  "endsAt": "11:50"
}
```

### 34. Изменение занятия в расписании

`PATCH /admin/lessons/{lessonId}`

Описание:
Обновляет данные занятия в расписании.

Path params
```json
{
  "lessonId": "uuid"
}
```

Headers
- `Authorization: Bearer <token>`
- `Content-Type: application/json`

Request body
```json
{
  "date": "string",
  "cabinet": "string|null",
  "type": "string",
  "startsAt": "string",
  "endsAt": "string"
}
```

Response 200 OK
```json
{
  "lessonId": "uuid",
  "subjectId": "uuid",
  "teacherId": "uuid",
  "groupId": "uuid",
  "date": "string",
  "cabinet": "string|null",
  "type": "string",
  "startsAt": "string",
  "endsAt": "string"
}
```

Response codes
- `200 OK` — занятие обновлено
- `400 Bad Request` — неверные данные в запросе
- `401 Unauthorized` — пользователь не авторизован
- `403 Forbidden` — доступ только для администратора
- `404 Not Found` — занятие не найдено
- `500 Internal Server Error` — ошибка сервера

Пример запроса
```json
{
  "cabinet": "1103",
  "startsAt": "12:10",
  "endsAt": "13:40"
}
```

Пример ответа
```json
{
  "lessonId": "b1b1b1b1-1111-2222-3333-444444444444",
  "subjectId": "11111111-aaaa-bbbb-cccc-111111111111",
  "teacherId": "22222222-aaaa-bbbb-cccc-222222222222",
  "groupId": "33333333-aaaa-bbbb-cccc-333333333333",
  "date": "2026-04-16",
  "cabinet": "1103",
  "type": "lecture",
  "startsAt": "12:10",
  "endsAt": "13:40"
}
```

### 35. Получение push token-ов (опционально)

`GET /admin/device-tokens`

Описание:
Возвращает список зарегистрированных push token-ов устройств.

Query params
```json
{
  "limit": "int|null",
  "offset": "int|null"
}
```

Headers
- `Authorization: Bearer <token>`

Response 200 OK
```json
{
  "items": [
    {
      "id": "uuid",
      "userId": "uuid",
      "token": "string",
      "platform": "string",
      "createdAt": "string",
      "revokedAt": "string|null"
    }
  ],
  "total": "int"
}
```

Response codes
- `200 OK` — список token-ов получен
- `401 Unauthorized` — пользователь не авторизован
- `403 Forbidden` — доступ только для администратора
- `500 Internal Server Error` — ошибка сервера

Пример запроса
```http
GET /admin/device-tokens?limit=20&offset=0
```

Пример ответа
```json
{
  "items": [
    {
      "id": "22222222-bbbb-cccc-dddd-222222222222",
      "userId": "7c8c8d9a-1234-4567-8901-aabbccddeeff",
      "token": "fcm-token-123456",
      "platform": "android",
      "createdAt": "2026-04-16T09:10:00",
      "revokedAt": null
    }
  ],
  "total": 1
}
```
