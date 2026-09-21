# Получение мест на сеанс

## Описание

Метод вызывается покупателем после выбора сеанса. Он показывает схему мест его
зала и вычисляет доступность каждого места по активным билетам, не изменяя
данные.

## URL

`GET /v1/screenings/{screeningId}/seats`

## Логика обработки запроса

1. Найти запись в таблице [screenings](../../db/data-model.dbml), для которой
   `screenings.id` =
   `screeningId`.
   * Если сеанс не найден, прекратить обработку и вернуть `404 Not Found`.
2. Получить записи из таблицы [seats](../../db/data-model.dbml), для которых
   `seats.hall_id` = `screenings.hall_id` найденного сеанса.
3. Определить доступность каждого места:
   * место недоступно, если существует запись в [tickets](../../db/data-model.dbml),
     для которой `tickets.screening_id` = `screeningId`, `tickets.seat_id` =
     `seats.id` и `tickets.status` имеет значение `held` или `paid`;
   * иначе место доступно.
4. Вернуть схему мест с вычисленной доступностью в соответствии с
   [контрактом](../openapi.yaml).

## Связанные документы

- [OpenAPI](../openapi.yaml) — HTTP-контракт получения схемы мест.
- [Data model](../../db/data-model.dbml) — таблицы `screenings`, `seats` и
  `tickets`.
