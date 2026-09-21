# Обработка callback платёжного провайдера

## Описание

Метод вызывается внешним платёжным провайдером после завершения оплаты. Он по
итоговому статусу платежа меняет состояние платежа, заказа и связанных билетов.

## URL

`POST /v1/payment-callbacks`

## Логика обработки запроса

1. Найти запись в таблице [payments](../../db/data-model.dbml), для которой
   `payments.provider_payment_id` равен идентификатору платежа из callback.
   * Если платёж не найден, прекратить обработку и вернуть `404 Not Found`.
2. Получить запись в [orders](../../db/data-model.dbml), для которой
   `orders.id` = `payments.order_id`, и все связанные с заказом записи в
   [tickets](../../db/data-model.dbml).
3. Проверить текущий статус платежа:
   * если `payments.status` уже имеет итоговое значение `succeeded` или `failed`,
     не изменять данные и вернуть `204 No Content`;
   * иначе продолжить обработку в одной транзакции.
4. Проверить статус, полученный в callback:
   * если он равен `succeeded`, изменить `payments.status` на `succeeded`,
     заполнить `payments.paid_at`, изменить `orders.status` на `paid` и изменить
     `tickets.status` всех билетов заказа на `paid`;
   * если он равен `failed`, изменить `payments.status` на `failed`,
     `orders.status` на `cancelled` и `tickets.status` всех билетов заказа на
     `cancelled`.
5. Зафиксировать транзакцию и вернуть `204 No Content`.

### Маппинг callback в локальные данные

| Целевое поле | Источник данных | Правило заполнения |
|:-------------|:----------------|:-------------------|
| `payments.provider_payment_id` | `callback.providerPaymentId` | Использовать для поиска платежа; значение не изменять. |
| `payments.status` | `callback.status` | При `succeeded` сохранить `succeeded`; при `failed` сохранить `failed`. |
| `payments.paid_at` | Системное время обработки callback | Заполнить только при статусе `succeeded`. |
| `orders.status` | `callback.status` | При `succeeded` сохранить `paid`; при `failed` сохранить `cancelled`. |
| `tickets.status` | `callback.status` | Для всех билетов заказа при `succeeded` сохранить `paid`; при `failed` — `cancelled`. |

## Связанные документы

- [OpenAPI](../openapi.yaml) — HTTP-контракт обработки callback.
- [Payment provider API](../../external-systems/payment-provider/openapi.yaml) —
  источник формата callback.
- [Data model](../../db/data-model.dbml) — таблицы `payments`, `orders` и
  `tickets`.
