# API contracts

Базовый адрес API:
`http://localhost:8080/api/v1`

Все запросы и ответы передаются в формате `application/json`.

Для защищенных ручек используется заголовок:
`Authorization: Bearer <token>`

Для единообразия ниже используются пути в множественном числе:
- `/profiles/{userId}`
- `/subjects`
- `/teachers`
- `/groups`
- `/students/me/...`
- `/teachers/me/...`
- `/admin/...`

## Общее

### 1. Авторизация

`POST /auth/login`

Описание:
Авторизация пользователя в системе по email/логину и паролю.

Headers
`Content-Type: application/json`

Request body
```json
{
  "email": "string",
  "password": "string"
}
```

Response 200 OK
```json
{
  "token": "string",
  "user": {
    "id": "uuid",
    "roleId": "uuid",
    "roleName": "string",
    "firstName": "string",
    "lastName": "string",
    "fatherName": "string|null",
    "email": "string",
    "userType": "string"
  }
}
```

Response codes
- `200 OK` — авторизация успешна
- `400 Bad Request` — неверный формат запроса
- `401 Unauthorized` — неверный логин или пароль
- `500 Internal Server Error` — ошибка сервера

Пример запроса
```json
{
  "email": "timur@stud.kpfu.ru",
  "password": "123456"
}
```

Пример ответа
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI...",
  "user": {
    "id": "7c8c8d9a-1234-4567-8901-aabbccddeeff",
    "roleId": "21b2d50d-8f11-4d0a-b88a-11f111111111",
    "roleName": "student",
    "firstName": "Тимур",
    "lastName": "Закиров",
    "fatherName": "Салаватович",
    "email": "timur@stud.kpfu.ru",
    "userType": "student"
  }
}
```

### 2. Получение профиля

`GET /profiles/{userId}`

Описание:
Получение профиля пользователя по его идентификатору.
Владелец профиля может видеть свои данные полностью, другие пользователи — только данные, доступные для просмотра по роли.

Path params
```json
{
  "userId": "uuid"
}
```

Headers
- `Authorization: Bearer <token>`

Response 200 OK
```json
{
  "id": "uuid",
  "roleName": "string",
  "firstName": "string",
  "lastName": "string",
  "fatherName": "string|null",
  "email": "string",
  "phone": "string|null",
  "profile": {
    "groupId": "uuid|null",
    "groupName": "string|null"
  }
}
```

Response codes
- `200 OK` — профиль найден
- `401 Unauthorized` — пользователь не авторизован
- `403 Forbidden` — недостаточно прав
- `404 Not Found` — профиль не найден

Пример ответа для студента
```json
{
  "id": "7c8c8d9a-1234-4567-8901-aabbccddeeff",
  "roleName": "student",
  "firstName": "Тимур",
  "lastName": "Закиров",
  "fatherName": "Салаватович",
  "email": "timur@stud.kpfu.ru",
  "phone": "+79001234567",
  "profile": {
    "groupId": "11111111-2222-3333-4444-555555555555",
    "groupName": "09-411"
  }
}
```

Пример ответа для преподавателя
```json
{
  "id": "aaaa1111-2222-3333-4444-bbbbbbbbbbbb",
  "roleName": "teacher",
  "firstName": "Тимур",
  "lastName": "Закиров",
  "fatherName": "Салаватович",
  "email": "t.zakirov@kpfu.ru",
  "phone": "+78432337109",
  "profile": {
    "groupId": null,
    "groupName": null
  }
}
```

### 3. Получение расписания на текущий день

`GET /schedule/day`

Описание:
Возвращает расписание пользователя на выбранный день.

Query params
```json
{
  "date": "string (YYYY-MM-DD)",
  "groupId": "uuid|null",
  "teacherId": "uuid|null"
}
```

`date` — обязательный параметр.

Если `groupId` и `teacherId` не переданы, возвращается расписание текущего авторизованного пользователя.

Headers
- `Authorization: Bearer <token>`

Response 200 OK
```json
{
  "date": "string",
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
      "cabinet": "string|null",
      "type": "string",
      "startsAt": "string",
      "endsAt": "string"
    }
  ],
  "summary": {
    "totalLessons": "int",
    "finishedLessons": "int",
    "remainingLessons": "int"
  }
}
```

Response codes
- `200 OK` — расписание получено
- `400 Bad Request` — дата не передана или неверный формат
- `401 Unauthorized` — пользователь не авторизован
- `404 Not Found` — расписание не найдено

Пример запроса
```http
GET /schedule/day?date=2026-04-09
```

Пример ответа
```json
{
  "date": "2026-04-09",
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
      "cabinet": "1101",
      "type": "lecture",
      "startsAt": "10:20",
      "endsAt": "11:50"
    }
  ],
  "summary": {
    "totalLessons": 4,
    "finishedLessons": 1,
    "remainingLessons": 3
  }
}
```

### 4. Получение расписания на неделю

`GET /schedule/week`

Описание:
Возвращает расписание пользователя на неделю, в которую входит переданная дата.

Query params
```json
{
  "date": "string (YYYY-MM-DD)",
  "groupId": "uuid|null",
  "teacherId": "uuid|null"
}
```

`date` — обязательный параметр.

Headers
- `Authorization: Bearer <token>`

Response 200 OK
```json
{
  "weekStart": "string",
  "weekEnd": "string",
  "days": [
    {
      "date": "string",
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
          "cabinet": "string|null",
          "type": "string",
          "startsAt": "string",
          "endsAt": "string"
        }
      ]
    }
  ],
  "summary": {
    "totalLessons": "int",
    "studyDays": "int",
    "freeDays": "int"
  }
}
```

Response codes
- `200 OK` — расписание получено
- `400 Bad Request` — дата не передана или неверный формат
- `401 Unauthorized` — пользователь не авторизован
- `404 Not Found` — расписание не найдено

Пример запроса
```http
GET /schedule/week?date=2026-04-14
```

Пример ответа
```json
{
  "weekStart": "2026-04-14",
  "weekEnd": "2026-04-20",
  "days": [
    {
      "date": "2026-04-14",
      "items": [
        {
          "lessonId": "1d1d1d1d-1111-2222-3333-444444444444",
          "subjectId": "44444444-aaaa-bbbb-cccc-444444444444",
          "subjectName": "Математический анализ",
          "teacherId": "55555555-aaaa-bbbb-cccc-555555555555",
          "teacherFirstName": "Иван",
          "teacherLastName": "Иванов",
          "teacherFatherName": "Иванович",
          "groupId": "66666666-aaaa-bbbb-cccc-666666666666",
          "groupName": "09-352",
          "cabinet": "108",
          "type": "lecture",
          "startsAt": "08:30",
          "endsAt": "10:00"
        }
      ]
    },
    {
      "date": "2026-04-16",
      "items": []
    }
  ],
  "summary": {
    "totalLessons": 6,
    "studyDays": 4,
    "freeDays": 3
  }
}
```

### 5. Получение уведомлений

`GET /notifications`

Описание:
Возвращает список уведомлений текущего пользователя.

Query params
```json
{
  "isRead": "boolean|null",
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
      "title": "string",
      "body": "string",
      "type": "string",
      "isRead": "boolean",
      "createdAt": "string"
    }
  ],
  "total": "int"
}
```

Response codes
- `200 OK` — уведомления получены
- `401 Unauthorized` — пользователь не авторизован
- `500 Internal Server Error` — ошибка сервера

Пример запроса
```http
GET /notifications?isRead=false&limit=20&offset=0
```

Пример ответа
```json
{
  "items": [
    {
      "id": "77777777-aaaa-bbbb-cccc-777777777777",
      "title": "Изменение в расписании",
      "body": "Пара по базам данных перенесена в кабинет 1103",
      "type": "schedule",
      "isRead": false,
      "createdAt": "2026-04-16T09:10:00"
    }
  ],
  "total": 1
}
```

### 6. Получение списка предметов

`GET /subjects`

Описание:
Возвращает список предметов, доступных текущему пользователю.

Query params
```json
{
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
      "id": "uuid",
      "name": "string",
      "groupId": "uuid",
      "groupName": "string",
      "teacherId": "uuid",
      "teacherFirstName": "string",
      "teacherLastName": "string",
      "teacherFatherName": "string|null"
    }
  ],
  "total": "int"
}
```

Response codes
- `200 OK` — список предметов получен
- `401 Unauthorized` — пользователь не авторизован
- `500 Internal Server Error` — ошибка сервера

Пример запроса
```http
GET /subjects?groupId=33333333-aaaa-bbbb-cccc-333333333333
```

Пример ответа
```json
{
  "items": [
    {
      "id": "11111111-aaaa-bbbb-cccc-111111111111",
      "name": "Базы данных",
      "groupId": "33333333-aaaa-bbbb-cccc-333333333333",
      "groupName": "09-352",
      "teacherId": "22222222-aaaa-bbbb-cccc-222222222222",
      "teacherFirstName": "Ринат",
      "teacherLastName": "Сафрутдинов",
      "teacherFatherName": "Наилевич"
    }
  ],
  "total": 1
}
```

### 7. Получение информации по предмету

`GET /subjects/{subjectId}`

Описание:
Возвращает основную информацию по выбранному предмету.

Path params
```json
{
  "subjectId": "uuid"
}
```

Headers
- `Authorization: Bearer <token>`

Response 200 OK
```json
{
  "id": "uuid",
  "name": "string",
  "groupId": "uuid",
  "groupName": "string",
  "teacherId": "uuid",
  "teacherFirstName": "string",
  "teacherLastName": "string",
  "teacherFatherName": "string|null"
}
```

Response codes
- `200 OK` — информация по предмету получена
- `401 Unauthorized` — пользователь не авторизован
- `404 Not Found` — предмет не найден
- `500 Internal Server Error` — ошибка сервера

Пример запроса
```http
GET /subjects/11111111-aaaa-bbbb-cccc-111111111111
```

Пример ответа
```json
{
  "id": "11111111-aaaa-bbbb-cccc-111111111111",
  "name": "Базы данных",
  "groupId": "33333333-aaaa-bbbb-cccc-333333333333",
  "groupName": "09-352",
  "teacherId": "22222222-aaaa-bbbb-cccc-222222222222",
  "teacherFirstName": "Ринат",
  "teacherLastName": "Сафрутдинов",
  "teacherFatherName": "Наилевич"
}
```

### 8. Получение списка преподавателей

`GET /teachers`

Описание:
Возвращает список преподавателей.

Query params
```json
{
  "subjectId": "uuid|null"
}
```

Headers
- `Authorization: Bearer <token>`

Response 200 OK
```json
{
  "items": [
    {
      "teacherId": "uuid",
      "userId": "uuid",
      "firstName": "string",
      "lastName": "string",
      "fatherName": "string|null",
      "email": "string"
    }
  ],
  "total": "int"
}
```

Response codes
- `200 OK` — список преподавателей получен
- `401 Unauthorized` — пользователь не авторизован
- `500 Internal Server Error` — ошибка сервера

Пример запроса
```http
GET /teachers
```

Пример ответа
```json
{
  "items": [
    {
      "teacherId": "88888888-aaaa-bbbb-cccc-888888888888",
      "userId": "aaaa1111-2222-3333-4444-bbbbbbbbbbbb",
      "firstName": "Тимур",
      "lastName": "Закиров",
      "fatherName": "Салаватович",
      "email": "t.zakirov@kpfu.ru"
    }
  ],
  "total": 1
}
```

### 9. Получение списка групп

`GET /groups`

Описание:
Возвращает список учебных групп.

Query params
```json
{}
```

Headers
- `Authorization: Bearer <token>`

Response 200 OK
```json
{
  "items": [
    {
      "id": "uuid",
      "name": "string"
    }
  ],
  "total": "int"
}
```

Response codes
- `200 OK` — список групп получен
- `401 Unauthorized` — пользователь не авторизован
- `500 Internal Server Error` — ошибка сервера

Пример запроса
```http
GET /groups
```

Пример ответа
```json
{
  "items": [
    {
      "id": "33333333-aaaa-bbbb-cccc-333333333333",
      "name": "09-352"
    }
  ],
  "total": 1
}
```


## Для студента

### 11. Получение журнала оценок

`GET /students/me/grades`

Описание:
Возвращает журнал оценок текущего студента.

Query params
```json
{
  "subjectId": "uuid|null"
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
      "subjectId": "uuid",
      "subjectName": "string",
      "lessonId": "uuid|null",
      "lessonDate": "string|null",
      "grade": "int",
      "createdAt": "string"
    }
  ],
  "total": "int"
}
```

Response codes
- `200 OK` — журнал оценок получен
- `401 Unauthorized` — пользователь не авторизован
- `403 Forbidden` — доступ только для студента
- `500 Internal Server Error` — ошибка сервера

Пример запроса
```http
GET /students/me/grades?subjectId=11111111-aaaa-bbbb-cccc-111111111111
```

Пример ответа
```json
{
  "items": [
    {
      "id": "99999999-aaaa-bbbb-cccc-999999999999",
      "subjectId": "11111111-aaaa-bbbb-cccc-111111111111",
      "subjectName": "Базы данных",
      "lessonId": "b1b1b1b1-1111-2222-3333-444444444444",
      "lessonDate": "2026-04-09",
      "grade": 9,
      "createdAt": "2026-04-09T13:10:00"
    }
  ],
  "total": 1
}
```

### 13. Получение рейтинга

`GET /students/me/rating`

Описание:
Возвращает текущую позицию студента в рейтинге группы или по предмету.

Query params
```json
{
  "subjectId": "uuid|null"
}
```

Headers
- `Authorization: Bearer <token>`

Response 200 OK
```json
{
  "groupId": "uuid",
  "groupName": "string",
  "subjectId": "uuid|null",
  "subjectName": "string|null",
  "ratingPosition": "int",
  "totalGrade": "int",
  "updatedAt": "string",
  "topStudents": [
    {
      "studentId": "uuid",
      "userId": "uuid",
      "firstName": "string",
      "lastName": "string",
      "fatherName": "string|null",
      "totalGrade": "int",
      "ratingPosition": "int"
    }
  ]
}
```

Response codes
- `200 OK` — рейтинг получен
- `401 Unauthorized` — пользователь не авторизован
- `403 Forbidden` — доступ только для студента
- `500 Internal Server Error` — ошибка сервера

Пример запроса
```http
GET /students/me/rating
```

Пример ответа
```json
{
  "groupId": "33333333-aaaa-bbbb-cccc-333333333333",
  "groupName": "09-352",
  "subjectId": null,
  "subjectName": null,
  "ratingPosition": 4,
  "totalGrade": 94,
  "updatedAt": "2026-04-16T08:00:00",
  "topStudents": [
    {
      "studentId": "13131313-aaaa-bbbb-cccc-131313131313",
      "userId": "14141414-aaaa-bbbb-cccc-141414141414",
      "firstName": "Алия",
      "lastName": "Нигматуллина",
      "fatherName": "Раилевна",
      "totalGrade": 99,
      "ratingPosition": 1
    }
  ]
}
```

### 14. Получение своих результатов по предмету

`GET /students/me/subjects/{subjectId}/results`

Описание:
Возвращает сводную информацию студента по выбранному предмету.

Path params
```json
{
  "subjectId": "uuid"
}
```

Headers
- `Authorization: Bearer <token>`

Response 200 OK
```json
{
  "subjectId": "uuid",
  "subjectName": "string",
  "teacherId": "uuid",
  "teacherFirstName": "string",
  "teacherLastName": "string",
  "teacherFatherName": "string|null",
  "totalGrade": "int",
  "ratingPosition": "int|null",
  "grades": [
    {
      "id": "uuid",
      "lessonId": "uuid|null",
      "grade": "int",
      "createdAt": "string"
    }
  ]
}
```

Response codes
- `200 OK` — результаты по предмету получены
- `401 Unauthorized` — пользователь не авторизован
- `403 Forbidden` — доступ только для студента
- `404 Not Found` — предмет не найден
- `500 Internal Server Error` — ошибка сервера

Пример запроса
```http
GET /students/me/subjects/11111111-aaaa-bbbb-cccc-111111111111/results
```

Пример ответа
```json
{
  "subjectId": "11111111-aaaa-bbbb-cccc-111111111111",
  "subjectName": "Базы данных",
  "teacherId": "22222222-aaaa-bbbb-cccc-222222222222",
  "teacherFirstName": "Ринат",
  "teacherLastName": "Сафрутдинов",
  "teacherFatherName": "Наилевич",
  "totalGrade": 86,
  "ratingPosition": 5,
  "grades": [
    {
      "id": "99999999-aaaa-bbbb-cccc-999999999999",
      "lessonId": "b1b1b1b1-1111-2222-3333-444444444444",
      "grade": 9,
      "createdAt": "2026-04-09T13:10:00"
    }
  ]
}
```

### 15. Получение истории уведомлений

`GET /students/me/notifications/history`

Описание:
Возвращает полную историю уведомлений текущего студента.

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
      "title": "string",
      "body": "string",
      "type": "string",
      "isRead": "boolean",
      "createdAt": "string"
    }
  ],
  "total": "int"
}
```

Response codes
- `200 OK` — история уведомлений получена
- `401 Unauthorized` — пользователь не авторизован
- `403 Forbidden` — доступ только для студента
- `500 Internal Server Error` — ошибка сервера

Пример запроса
```http
GET /students/me/notifications/history?limit=20&offset=0
```

Пример ответа
```json
{
  "items": [
    {
      "id": "77777777-aaaa-bbbb-cccc-777777777777",
      "title": "Изменение в расписании",
      "body": "Пара по базам данных перенесена в кабинет 1103",
      "type": "schedule",
      "isRead": true,
      "createdAt": "2026-04-16T09:10:00"
    }
  ],
  "total": 1
}
```

## Для преподавателя

### 16. Получение своих занятий

`GET /teachers/me/lessons`

Описание:
Возвращает занятия текущего преподавателя на выбранный день.

Query params
```json
{
  "date": "string (YYYY-MM-DD)"
}
```

`date` — обязательный параметр.

Headers
- `Authorization: Bearer <token>`

Response 200 OK
```json
{
  "date": "string",
  "items": [
    {
      "lessonId": "uuid",
      "subjectId": "uuid",
      "subjectName": "string",
      "groupId": "uuid",
      "groupName": "string",
      "cabinet": "string|null",
      "type": "string",
      "startsAt": "string",
      "endsAt": "string"
    }
  ]
}
```

Response codes
- `200 OK` — занятия получены
- `400 Bad Request` — дата не передана или неверный формат
- `401 Unauthorized` — пользователь не авторизован
- `403 Forbidden` — доступ только для преподавателя
- `500 Internal Server Error` — ошибка сервера

Пример запроса
```http
GET /teachers/me/lessons?date=2026-04-16
```

Пример ответа
```json
{
  "date": "2026-04-16",
  "items": [
    {
      "lessonId": "b1b1b1b1-1111-2222-3333-444444444444",
      "subjectId": "11111111-aaaa-bbbb-cccc-111111111111",
      "subjectName": "Базы данных",
      "groupId": "33333333-aaaa-bbbb-cccc-333333333333",
      "groupName": "09-352",
      "cabinet": "1101",
      "type": "lecture",
      "startsAt": "10:20",
      "endsAt": "11:50"
    }
  ]
}
```

### 17. Получение списка групп преподавателя

`GET /teachers/me/groups`

Описание:
Возвращает список групп, с которыми работает текущий преподаватель.

Query params
```json
{
  "subjectId": "uuid|null"
}
```

Headers
- `Authorization: Bearer <token>`

Response 200 OK
```json
{
  "items": [
    {
      "groupId": "uuid",
      "groupName": "string"
    }
  ],
  "total": "int"
}
```

Response codes
- `200 OK` — список групп получен
- `401 Unauthorized` — пользователь не авторизован
- `403 Forbidden` — доступ только для преподавателя
- `500 Internal Server Error` — ошибка сервера

Пример запроса
```http
GET /teachers/me/groups
```

Пример ответа
```json
{
  "items": [
    {
      "groupId": "33333333-aaaa-bbbb-cccc-333333333333",
      "groupName": "09-352"
    }
  ],
  "total": 1
}
```

### 18. Получение списка студентов группы

`GET /teachers/me/groups/{groupId}/students`

Описание:
Возвращает список студентов выбранной группы. При необходимости добавляются доп агреторы типа totalGrade.

Path params
```json
{
  "groupId": "uuid"
}
```


Headers
- `Authorization: Bearer <token>`

Response 200 OK
```json
{
  "groupId": "uuid",
  "groupName": "string",
  "subjectId": "uuid|null",
  "items": [
    {
      "studentId": "uuid",
      "userId": "uuid",
      "firstName": "string",
      "lastName": "string",
      "fatherName": "string|null",
      "email": "string",
      "totalGrade": "int|null"
    }
  ]
}
```

Response codes
- `200 OK` — список студентов получен
- `401 Unauthorized` — пользователь не авторизован
- `403 Forbidden` — доступ только для преподавателя
- `404 Not Found` — группа не найдена
- `500 Internal Server Error` — ошибка сервера

Пример запроса
```http
GET /teachers/me/groups/33333333-aaaa-bbbb-cccc-333333333333/students?subjectId=11111111-aaaa-bbbb-cccc-111111111111
```

Пример ответа
```json
{
  "groupId": "33333333-aaaa-bbbb-cccc-333333333333",
  "groupName": "09-352",
  "subjectId": "11111111-aaaa-bbbb-cccc-111111111111",
  "items": [
    {
      "studentId": "15151515-aaaa-bbbb-cccc-151515151515",
      "userId": "7c8c8d9a-1234-4567-8901-aabbccddeeff",
      "firstName": "Тимур",
      "lastName": "Закиров",
      "fatherName": "Салаватович",
      "email": "timur@stud.kpfu.ru",
      "totalGrade": 86
    }
  ]
}
```

### 19. Заполнение журнала занятия

`PUT /lessons/{lessonId}/journal`

Описание:
Сохраняет значения журнала по конкретному занятию. Для каждого студента можно передать либо отметку посещаемости `attended`, либо числовую оценку `grade`.

Если передано `attended`, API сохраняет запись в `lesson_participation`.
Если передано `grade`, API сохраняет запись в `student_grades`.
В одном элементе `items` должно быть заполнено ровно одно из полей: `attended` или `grade`.

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
  "items": [
    {
      "studentId": "uuid",
      "attended": "boolean|null",
      "grade": "int|null"
    }
  ]
}
```

Response 200 OK
```json
{
  "lessonId": "uuid",
  "items": [
    {
      "studentId": "uuid",
      "attended": "boolean|null",
      "grade": "int|null"
    }
  ]
}
```

Response codes
- `200 OK` — журнал сохранен
- `400 Bad Request` — неверные данные в запросе
- `401 Unauthorized` — пользователь не авторизован
- `403 Forbidden` — доступ только для преподавателя
- `404 Not Found` — занятие или студент не найден
- `500 Internal Server Error` — ошибка сервера

Пример запроса
```json
{
  "items": [
    {
      "studentId": "15151515-aaaa-bbbb-cccc-151515151515",
      "attended": null,
      "grade": 5
    },
    {
      "studentId": "16161616-aaaa-bbbb-cccc-161616161616",
      "attended": true,
      "grade": null
    },
    {
      "studentId": "17171717-aaaa-bbbb-cccc-171717171717",
      "attended": false,
      "grade": null
    }
  ]
}
```

Пример ответа
```json
{
  "lessonId": "b1b1b1b1-1111-2222-3333-444444444444",
  "items": [
    {
      "studentId": "15151515-aaaa-bbbb-cccc-151515151515",
      "attended": null,
      "grade": 5
    },
    {
      "studentId": "16161616-aaaa-bbbb-cccc-161616161616",
      "attended": true,
      "grade": null
    },
    {
      "studentId": "17171717-aaaa-bbbb-cccc-171717171717",
      "attended": false,
      "grade": null
    }
  ]
}
```

### 20. Обновление оценки

`PATCH /grades/{gradeId}`

Описание:
Обновляет ранее выставленную оценку.

Path params
```json
{
  "gradeId": "uuid"
}
```

Headers
- `Authorization: Bearer <token>`
- `Content-Type: application/json`

Request body
```json
{
  "grade": "int"
}
```

Response 200 OK
```json
{
  "id": "uuid",
  "studentId": "uuid",
  "lessonId": "uuid",
  "grade": "int",
  "updatedAt": "string"
}
```

Response codes
- `200 OK` — оценка обновлена
- `400 Bad Request` — неверные данные в запросе
- `401 Unauthorized` — пользователь не авторизован
- `403 Forbidden` — доступ только для преподавателя
- `404 Not Found` — оценка не найдена
- `500 Internal Server Error` — ошибка сервера

Пример запроса
```json
{
  "grade": 10
}
```

Пример ответа
```json
{
  "id": "99999999-aaaa-bbbb-cccc-999999999999",
  "studentId": "15151515-aaaa-bbbb-cccc-151515151515",
  "lessonId": "b1b1b1b1-1111-2222-3333-444444444444",
  "grade": 10,
  "updatedAt": "2026-04-16T12:20:00"
}
```

### 21. Отправка сообщения группе

`POST /teacher-messages`

Описание:
Отправляет сообщение группе студентов.

Request body
```json
{
  "groupId": "uuid",
  "title": "string",
  "message": "string"
}
```

Response 201 Created
```json
{
  "id": "uuid",
  "teacherId": "uuid",
  "groupId": "uuid",
  "title": "string",
  "message": "string",
  "createdAt": "string"
}
```

Response codes
- `201 Created` — сообщение отправлено
- `400 Bad Request` — неверные данные в запросе
- `401 Unauthorized` — пользователь не авторизован
- `403 Forbidden` — доступ только для преподавателя
- `404 Not Found` — группа не найдена
- `500 Internal Server Error` — ошибка сервера

Пример запроса
```json
{
  "groupId": "33333333-aaaa-bbbb-cccc-333333333333",
  "title": "Перенос занятия",
  "message": "Практика в пятницу начнется в 12:10"
}
```

Пример ответа
```json
{
  "id": "17171717-aaaa-bbbb-cccc-171717171717",
  "teacherId": "88888888-aaaa-bbbb-cccc-888888888888",
  "groupId": "33333333-aaaa-bbbb-cccc-333333333333",
  "title": "Перенос занятия",
  "message": "Практика в пятницу начнется в 12:10",
  "createdAt": "2026-04-16T13:00:00"
}
```

### 22. Получение предметов преподавателя

`GET /teachers/me/subjects`

Описание:
Возвращает список предметов преподавателя.

Headers
- `Authorization: Bearer <token>`

Response 200 OK
```json
{
  "items": [
    {
      "subjectId": "uuid",
      "subjectName": "string",
      "groupId": "uuid",
      "groupName": "string"
    }
  ]
}
```

Response codes
- `200 OK` — предметы преподавателя получены
- `401 Unauthorized` — пользователь не авторизован
- `403 Forbidden` — доступ только для преподавателя
- `500 Internal Server Error` — ошибка сервера

Пример запроса
```http
GET /teachers/me/subjects
```

Пример ответа
```json
{
  "items": [
    {
      "subjectId": "11111111-aaaa-bbbb-cccc-111111111111",
      "subjectName": "Базы данных",
      "groupId": "33333333-aaaa-bbbb-cccc-333333333333",
      "groupName": "09-352"
    }
  ]
}
```
