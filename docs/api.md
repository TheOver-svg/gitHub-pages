Сервер надає REST API. Інтерактивна документація (Swagger) доступна за адресою `http://localhost:5045/swagger`, де кожен запит можна виконати прямо в браузері.

## Ендпоінти

| Метод | Адреса | Призначення |
|---|---|---|
| GET | `/api/symptoms` | список усіх ознак |
| GET | `/api/symptoms/{code}` | ознака за кодом |
| POST | `/api/symptoms` | додати ознаку |
| GET | `/api/rules` | список усіх правил |
| POST | `/api/rules` | додати правило |
| DELETE | `/api/rules/{id}` | видалити правило |
| POST | `/api/diagnosis` | діагноз за списком ознак (без діалогу) |
| POST | `/api/consultation/next` | наступний крок діалогу |

## Додати ознаку

`POST /api/symptoms`

```json
{ "code": "s_battery_old", "label": "Акумулятор старий", "group": "Пуск двигуна" }
```

Поле `id` не вказуйте: його створює база.

## Додати правило

`POST /api/rules`

```json
{
  "ruleCode": "R17",
  "conditions": ["s_starter_yes", "s_spark_no", "s_fuel_smell"],
  "conclusionCode": "c17",
  "conclusionText": "Замінити свічку і перевірити паливо."
}
```

## Діагноз одним запитом

`POST /api/diagnosis`

```json
{ "selectedSymptomCodes": ["s_starter_yes", "s_spark_no"] }
```

Відповідь містить список правил, що спрацювали:

```json
{ "firedRules": [ { "ruleCode": "R2", "conclusionCode": "c2", "conclusionText": "Несправна система запалювання..." } ] }
```

## Крок діалогу

`POST /api/consultation/next`

Запит. У `answers` вказують **усі** попередні відповіді, `value` може бути `true`, `false` або `null` («Не знаю»):

```json
{
  "group": "Пуск двигуна",
  "answers": [ { "code": "s_starter_yes", "value": true } ]
}
```

Відповідь:

```json
{
  "finished": false,
  "question": { "code": "s_spark_no", "label": "Немає іскри на свічці", "group": "Пуск двигуна" },
  "hypotheses": [ { "ruleCode": "R2", "text": "Несправна система запалювання...", "matched": 1, "total": 2 } ],
  "firedRules": [],
  "questionNumber": 2
}
```

| Поле | Значення |
|---|---|
| `finished` | `false` — показати питання; `true` — показати підсумок |
| `question` | наступне питання (`null`, коли діалог завершено) |
| `hypotheses` | версії, підтверджені частково: `matched` з `total` умов |
| `firedRules` | правила, що вже спрацювали; `because` містить пояснення |
| `questionNumber` | номер поточного питання |
