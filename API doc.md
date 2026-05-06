# API contracts

Базовый адрес API:
`http://localhost:8080/api`

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

### 1. Авторизация

`POST /login выполнено` 

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
}
```
### 2. Получение профиля

`GET /student/me выполнено`
`GET /teacher/me выполнено`

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

Для студента:
```json
{
  "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "roleName": "string",
  "firstName": "string",
  "lastName": "string",
  "fatherName": "string",
  "email": "string",
  "studentId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "groupId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "groupName": "string"
}
```

#Для учителя:
```
{
  "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "firstName": "string",
  "lastName": "string",
  "fatherName": "string",
  "email": "string",
  "teacherId": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
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
  "studentId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "groupId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
}
```

Пример ответа для преподавателя
```json
{
  "id": "7c8c8d9a-1234-4567-8901-aabbccddeeff",
  "roleName": "student",
  "firstName": "Тимур",
  "lastName": "Закиров",
  "fatherName": "Салаватович",
  "email": "timur@stud.kpfu.ru",
  "studentId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "groupId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
}
```

### 3. Получение расписания на текущий день учителя и студента

`GET /schedule/student/today выполнено`
`GET /schedule/teacher/today выполнено`

Описание:
Возвращает расписание студента на выбранный день.

Query params
```json
{
  "date": "string (YYYY-MM-DD)||null",
}
```

`date` — обязательный параметр.

Headers
- `Authorization: Bearer <token>`
  в токене передаём userId

Response 200 OK
Для студента
```json
{
  "date": "string",
  "dayName": "string",
  "weekNumber": 0,
  "lessonsCount": 0,
  "items": [
    {
      "lessonsId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
      "subjectId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
      "subjectName": "string",
      "teacherId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
      "teacherFirstName": "string",
      "teacherLastName": "string",
      "teacherFatherName": "string",
      "cabinet": "string",
      "type": "string",
      "startsAt": "string",
      "endsAt": "string"
    }
  ]
}
```

Для учителя
```
{
  "date": "string",
  "dayName": "string",
  "weekNumber": 0,
  "lessonsCount": 0,
  "items": [
    {
      "lessonsId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
      "subjectId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
      "subjectName": "string",
      "cabinet": "string",
      "type": "string",
      "startsAt": "string",
      "endsAt": "string"
    }
  ]
}
```

Response codes
- `200 OK` — расписание получено
- `400 Bad Request` — дата не передана или неверный формат
- `401 Unauthorized` — пользователь не авторизован
- `404 Not Found` — расписание не найдено

Пример запроса
```http
GET /schedule/student/today?date=2026-04-09
```

### 4. Получение расписания на неделю

`GET /schedule/student/week выполнено`
`GET /schedule/teacher/week выполнено`

Описание:
Возвращает расписание пользователя на неделю, в которую входит переданная дата.

Query params
```json
{
  "date": "string (YYYY-MM-DD)",
}
```

`date` — обязательный параметр.

Headers
- `Authorization: Bearer <token>`
  userId передаётся в токене

Response 200 OK
Для студента
```
{
  "dateStart": "string",
  "dateEnd": "string",
  "items": [
    {
      "date": "string",
      "dayName": "string",
      "weekNumber": 0,
      "lessonsCount": 0,
      "items": [
        {
          "lessonsId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
          "subjectId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
          "subjectName": "string",
          "teacherId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
          "teacherFirstName": "string",
          "teacherLastName": "string",
          "teacherFatherName": "string",
          "cabinet": "string",
          "type": "string",
          "startsAt": "string",
          "endsAt": "string"
        }
      ]
    }
  ]
}
```

Для учителя
```json
{
  "dateStart": "string",
  "dateEnd": "string",
  "items": [
    {
      "date": "string",
      "dayName": "string",
      "weekNumber": 0,
      "lessonsCount": 0,
      "items": [
        {
          "lessonsId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
          "subjectId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
          "subjectName": "string",
          "cabinet": "string",
          "type": "string",
          "startsAt": "string",
          "endsAt": "string"
        }
      ]
    }
  ]
}
```

Response codes
- `200 OK` — расписание получено
- `400 Bad Request` — дата не передана или неверный формат
- `401 Unauthorized` — пользователь не авторизован
- `404 Not Found` — расписание не найдено

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
      "SenderName": "string",
      "SenderLastName": "string",
      "title": "string",
      "body": "string",
      "type": "string",
      "isRead": "boolean",
      "createdAt": "string"
    }
  ]
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

## 6. Получение информации по предмету

`GET /subjects/{subjectId}`

**Описание**

Возвращает основную информацию по выбранному предмету.

**Path parameters**

```json
{
  "subjectId": "uuid"
}
```

**Headers**

```
Authorization: Bearer <token>
```

**Response (200 OK)**

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

**Коды ответа**

- `200 OK` — информация по предмету получена
- `401 Unauthorized` — пользователь не авторизован
- `404 Not Found` — предмет не найден
- `500 Internal Server Error` — ошибка сервера

**Пример запроса**

```http
GET /subjects/11111111-aaaa-bbbb-cccc-111111111111
```

**Пример ответа**

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

---

## 7. Получение рейтинга

`GET /students/me/rating выполнено`

**Описание**

Возвращает текущую позицию студента в рейтинге группы или по предмету.

**Query parameters**

```json
{
  "subjectId": "uuid|null"
}
```

В случае пустого параметра будут выбираться все предметы.
В случае передачи конкретного предмета будет возвращаться рейтинг по конкретному предмету.

**Headers**

```
Authorization: Bearer <token>
```

**Response (200 OK)**

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

**Коды ответа**

- `200 OK` — рейтинг получен
- `401 Unauthorized` — пользователь не авторизован
- `403 Forbidden` — доступ только для студента
- `500 Internal Server Error` — ошибка сервера

**Пример запроса**

```http
GET /students/me/rating
```

**Пример ответа**

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

---


## 8. Обновление оценки

`PATCH /grades/{gradeId} выполнено`

**Описание**

Обновляет ранее выставленную оценку.

**Path parameters**

```json
{
  "gradeId": "uuid"
}
```

**Headers**

```
Authorization: Bearer <token>
Content-Type: application/json
```

**Request body**

```json
{
  "grade": "int"
}
```

**Response (200 OK)**

```json
{
  "id": "uuid",
  "studentId": "uuid",
  "lessonId": "uuid",
  "grade": "int",
  "updatedAt": "string"
}
```

**Коды ответа**

- `200 OK` — оценка обновлена
- `400 Bad Request` — неверные данные в запросе
- `401 Unauthorized` — пользователь не авторизован
- `403 Forbidden` — доступ только для преподавателя
- `404 Not Found` — оценка не найдена
- `500 Internal Server Error` — ошибка сервера

**Пример запроса**

```json
{
  "grade": 10
}
```

**Пример ответа**

```json
{
  "id": "99999999-aaaa-bbbb-cccc-999999999999",
  "studentId": "15151515-aaaa-bbbb-cccc-151515151515",
  "lessonId": "b1b1b1b1-1111-2222-3333-444444444444",
  "grade": 10,
  "updatedAt": "2026-04-16T12:20:00"
}
```

---

## 9. Отправка сообщения группе

`POST /teacher-messages`

**Описание**

Отправляет сообщение группе студентов.

**Headers**

```
Authorization: Bearer <token>
Content-Type: application/json
```

**Request body**

```json
{
  "groupId": "uuid",
  "title": "string",
  "message": "string"
}
```

**Response (201 Created)**

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

**Коды ответа**

- `201 Created` — сообщение отправлено
- `400 Bad Request` — неверные данные в запросе
- `401 Unauthorized` — пользователь не авторизован
- `403 Forbidden` — доступ только для преподавателя
- `404 Not Found` — группа не найдена
- `500 Internal Server Error` — ошибка сервера

**Пример запроса**

```json
{
  "groupId": "33333333-aaaa-bbbb-cccc-333333333333",
  "title": "Перенос занятия",
  "message": "Практика в пятницу начнется в 12:10"
}
```

**Пример ответа**

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

---

## 10. Получение предметов преподавателя

`GET /teachers/me/subjects`

**Описание**

Возвращает список предметов преподавателя. Для каждого предмета указаны группы, у которых он ведёт.

**Headers**

```
Authorization: Bearer <token>
```

**Response (200 OK)**

```json
{
  "items": [
    {
      "subjectId": "uuid",
      "subjectName": "string",
      "groups": [
        {
          "groupId": "uuid",
          "groupName": "string"
        }
      ]
    }
  ]
}
```

**Коды ответа**

- `200 OK` — предметы преподавателя получены
- `401 Unauthorized` — пользователь не авторизован
- `403 Forbidden` — доступ только для преподавателя
- `500 Internal Server Error` — ошибка сервера

**Пример запроса**

```http
GET /teachers/me/subjects
```

**Пример ответа**

```json
{
  "items": [
    {
      "subjectId": "11111111-aaaa-bbbb-cccc-111111111111",
      "subjectName": "Базы данных",
      "groups": [
        {
          "groupId": "33333333-aaaa-bbbb-cccc-333333333333",
          "groupName": "09-352"
        },
        {
          "groupId": "44444444-aaaa-bbbb-cccc-444444444444",
          "groupName": "09-353"
        }
      ]
    }
  ]
}
```

---

## 11. Получение журнала группы по предмету

`GET /journal/{subjectId}/{groupId}`

**Описание**

Возвращает журнал группы по конкретному предмету: список студентов, список оценок за каждый урок и список дат занятий. Структура ответа уточняется.

**Path parameters**

```json
{
  "subjectId": "uuid",
  "groupId": "uuid"
}
```

**Headers**

```
Authorization: Bearer <token>
```

**Response (200 OK)**

> Структура ответа требует уточнения.

**Коды ответа**

- `200 OK` — журнал получен
- `401 Unauthorized` — пользователь не авторизован
- `403 Forbidden` — недостаточно прав
- `404 Not Found` — предмет или группа не найдены
- `500 Internal Server Error` — ошибка сервера

**Пример запроса**

```http
GET /journal/11111111-aaaa-bbbb-cccc-111111111111/33333333-aaaa-bbbb-cccc-333333333333
```

### 12. Заполнение журнала занятия

`PUT /lessons/{lessonId}/journal выполнено`

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
