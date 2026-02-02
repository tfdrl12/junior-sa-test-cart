Задание 2. Проектирование REST API — экран «Выберите магазин»

Цель:
При открытии экрана мобильного приложения показать список магазинов-партнёров с:
- названием,
- логотипом,
- информацией о доставке (окно времени или “от X до Y минут”),
- ссылкой на внешний ресурс (по клику на карточку магазина).

--------------------------------------------
1) Пример REST API запроса (при входе на экран)

Метод: GET
URL: /api/v1/partner-stores

Пример запроса:
GET /api/v1/partner-stores?city_id=77&limit=20&offset=0 HTTP/1.1
Host: api.petrushka-green.ru
Accept: application/json
Accept-Language: ru-RU
X-Client: mobile
X-Client-Version: 1.12.0

Пояснение:
- city_id — чтобы отдать магазины, которые реально работают в городе пользователя.
- limit/offset — на случай, если партнёров станет много (пагинация).
- Accept-Language — чтобы тексты можно было локализовать.
- X-Client / X-Client-Version — полезно для аналитики и обратной совместимости.

--------------------------------------------
2) Пример JSON ответа (в соответствии с макетом)

HTTP/1.1 200 OK
Content-Type: application/json

{
  "city_id": 77,
  "timezone": "Europe/Moscow",
  "limit": 20,
  "offset": 0,
  "total": 4,
  "items": [
    {
      "id": "metro",
      "name": "METRO",
      "logo_url": "https://cdn.petrushka-green.ru/partners/metro.png",

      "delivery_info": {
        "type": "next_window",
        "title": "Ближайшая доставка",
        "subtitle": "сегодня 21:00–23:00"
      },

      "action": {
        "type": "external_link",
        "url": "https://partner.example/metro"
      }
    },
    {
      "id": "auchan",
      "name": "Ашан",
      "logo_url": "https://cdn.petrushka-green.ru/partners/auchan.png",

      "delivery_info": {
        "type": "next_window",
        "title": "Ближайшая доставка",
        "subtitle": "сегодня 18:00–20:00"
      },

      "action": {
        "type": "external_link",
        "url": "https://partner.example/auchan"
      }
    },
    {
      "id": "vkusvill",
      "name": "ВкусВилл",
      "logo_url": "https://cdn.petrushka-green.ru/partners/vkusvill.png",

      "delivery_info": {
        "type": "fast_delivery",
        "title": "Быстрая доставка",
        "subtitle": "от 20 до 60 минут",
        "eta_min_minutes": 20,
        "eta_max_minutes": 60
      },

      "action": {
        "type": "external_link",
        "url": "https://partner.example/vkusvill"
      }
    },
    {
      "id": "victoria",
      "name": "ВИКТОРИЯ",
      "logo_url": "https://cdn.petrushka-green.ru/partners/victoria.png",

      "delivery_info": {
        "type": "next_window",
        "title": "Ближайшая доставка",
        "subtitle": "сегодня 17:00–19:00"
      },

      "action": {
        "type": "external_link",
        "url": "https://partner.example/victoria"
      }
    }
  ]
}

Пояснение по полям:
- items[] — список партнёров для отображения карточками.
- id — стабильный идентификатор партнёра (удобно для логов и аналитики).
- name/logo_url — то, что видно на карточке.
- delivery_info — блок текста “как на макете”.
  - type=next_window — “Ближайшая доставка сегодня …”
  - type=fast_delivery — “Быстрая доставка от X до Y минут”
- action.url — внешний URL. При клике на карточку приложение открывает этот url во внешнем ресурсе.

--------------------------------------------
3) Ошибки (минимально, чтобы было понятно)

Если city_id не передан:
HTTP 400
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "city_id is required"
  }
}

Если в городе нет партнёров:
HTTP 200
{
  "city_id": 77,
  "timezone": "Europe/Moscow",
  "limit": 20,
  "offset": 0,
  "total": 0,
  "items": []
}

Почему так:
- 200 + пустой список — нормальный кейс, приложение просто покажет “нет доступных магазинов”.

--------------------------------------------
4) Допущения (чтобы решение было завершённым)

- Доставка и доступность партнёров зависят от города (city_id).
- Временные окна в ответе отдаются уже в корректной таймзоне города (timezone).
- Внешняя ссылка (action.url) обязательна для каждой карточки, иначе не выполнить требование “переход на внешний ресурс”.
