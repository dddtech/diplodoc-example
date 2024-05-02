# REST delivery/create rest 1
## Алгоритм обработки

### 1. Получение запроса
1.1 Получаем запрос, переходим к следующему шагу

### 2. Формирование и отправка сообщения о создании доставки
2.1 Вставка выборки таблицы + Фильтр таблиц

![](/diplodoc-example/docs/en/Analysis/firstDiagram.png)


<details>

```
@startuml firstDiagram

skinparam maxMessageSize 300
skinparam sequenceArrowThickness 2
skinparam roundcorner 10
skinparam backgroundColor EFEFEF
skinparam style strictuml
skinparam ParticipantBorderColor EFEFEF
skinparam DatabaseBorderColor EFEFEF

skinparam sequence {
ArrowColor 49494A
LifeLineBorderColor 49494A
LifeLineBackgroundColor 49494A
ArrowFontSize 12
GroupBorderColor LightBlue
GroupFontSize 10 
GroupHeaderFontColor 49494A
} 

participant Клиент
participant Какой_то_топик
participant LSP
participant Orders
participant Магазин


Клиент -> Клиент: Оформляет онлайн-заказ
Клиент -> Клиент: Подходит к магазину

alt Клиент нажал кнопку "Я около магазина"
     Клиент -> Клиент: Нажимает кнопку \n"Я около магазина" \n (MobileApp)
     LSP --> Какой_то_топик: Получает сообщение о \n том, что клиент около магазина
     LSP -> Orders: Запрашивает информацию по заказу
     LSP -> LSP: Проверяет в БД связь \nмагазин - главный сборщик
     LSP -> Магазин: Отправляет сообщение главному сборщику магазина \nо необходимости\nпереместить заказ в зону выдачи
     note right: Уведомление получено сборщиком. 
     Магазин -> Клиент: Выдаёт заказ
     deactivate Магазин

else Клиент не нажал кнопку "Я около магазина"
    Клиент -> Клиент: Заходит в магазин
    Клиент -> Клиент: Подходит к киоску\n и набирает свой номер телефона\n или заказа 
    LSP --> Какой_то_топик: Получает сообщение о \n том, что клиент в магазине
    LSP -> Orders: Запрашивает информацию по заказу
    LSP -> LSP: Проверяет в БД связь \nмагазин - главный сборщик
    LSP -> Магазин: Отправляет сообщение главному сборщику магазина \nо необходимости\nпереместить заказ в зону выдачи
    note right: Уведомление получено сборщиком. 
    Магазин -> Клиент: Выдаёт заказ
    deactivate Магазин
    end

@enduml
```

</details>


{% cut "Пример сообщения" %}

```json
{
  "orderNumber": "1239874563",
  "orderType": "DIRECT",
  "orderTimestamp": "2017-12-12T14:12:23.502Z",
  "saleChannel": "CALL_CENTER",
  "transferDocumentNumber": "4618732341",
  "checkoutObject": "S002",
  "brand": "MVIDEO",
  "sourceSystem": "CRM",
  "deliveryParams": {
    "deliveryType": "PICKUP",
    "deliveryDate": "2022-02-22",
    "deliveryTimeFrom": "17:00",
    "deliveryTimeTo": "18:00",
    "comment": " ",
    "express": true,
    "hiTechnic": false
  },
  "deliveryItem": [
    {
      "positionNumber": 10,
      "extItemId": "4ac60e6e-ed67-4bc0-8c74-aeb07dfec106",
      "itemType": "PRODUCT",
      "materialNumber": "20065055",
      "status": "CREATED",
      "reserveObject": "S790",
      "handoverObject": "S790",
      "deliveryDate": "2022-02-22",
      "quantity": 1,
      "quantityUnit": ".",
      "priceDiscount": 1900,
      "priceTotal": 2000,
      "showcaseSample": true,
      "transferDocumentParams": {
        "shipmentGuid": "2136afba-9d8d-40a4-b2bd-f1192a50385a",
        "itemDocNumber": "0000000001"
      },
      "reserveInfo": {
        "reserveNumber": "123573565",
        "reserveNumberPosition": "000001",
        "reserveRequestId": "CRM000003215",
        "externalPositionNumber": "001"
      }
    }
  ],
  "address": {
    "postCode": "390048",
    "city": "",
    "city2": ". ",
    "region": " ",
    "street": "",
    "house": "31",
    "building": "2",
    "intercom": "322B12",
    "apartment": "214",
    "entrance": "6",
    "floor": "4",
    "kladrCode": "77000000000151900"
  },
  "payment": {
    "paymentType": "ONLINE",
    "paymentStatus": "PAID"
  },
  "recipient": {
    "recipientName": " ",
    "phoneNumber": "+79129999999",
    "addPhoneNumber": "+79129999999"
  }
}

``` 
{% endcut %}

| Поле                    | Тип данных | Обязательность | Пример             | Описание                            | Маппинг из REST запроса в CRM в топик `delivery-command` |
|-------------------------|------------|----------------|--------------------|-------------------------------------|---------------------------------------------------------|
| `root`                  | object     | 1              |                    | Корневой объект                     |                                                         |
| `orderNumber`           | string     | 1              | 1239874563         | Номер заказа                        |                                                         |
| `orderType`             | enum       | 1              | DIRECT             | Тип заказа в CRM                   |                                                         |
|                         |            |                |                    | DIRECT - Заказ (продажа)           |                                                         |
|                         |            |                |                    | EXPORT - Возврат                   |                                                         |
| `orderTimestamp`        | timestamp  | 1              | 2017-12-12T14:12:23.502Z | Дата и время оформления заказа |                                                         |
| `transferDocumentNumber`| string     | 0              | 4618732341         | Номер документа передачи           |                                                         |
| `saleChannel`           | enum       | 1              | CALL_CENTER        | Канал продаж                       |                                                         |
|                         |            |                |                    | IN_STORE - Продажи через торговую систему в рознице (FOBO) |                                                         |
|                         |            |                |                    | WEBSITE - Заказы с сайта (старого и нового) |                                                         |
|                         |            |                |                    | CALL_CENTER - Заказы из call-центра |                                                         |
|                         |            |                |                    | CLIENT_MOBILE_APP - Мобильное приложение клиента МПК |                                                         |
|                         |            |                |                    | SBER_MEGA_MARKET - Заказы с маркетплейса SberMegaMarket (ex. Goods) |                                                         |
|                         |            |                |                    | SELLER_MOBILE_APP - Мобильное приложение продавца МПП/RTD |                                                         |
| `checkoutObject`        | string     | 1              | S002               | Объект оформления заказа           |                                                         |
| `brand`                 | enum       | 1              | MVIDEO             | Бренд, которому принадлежит заказ |                                                         |
|                         |            |                |                    | MVIDEO                             |                                                         |
|                         |            |                |                    | ELDORADO                           |                                                         |
| `sourceSystem`          | enum       | 1              | CRM                | Система-источник заказа            |                                                         |
|                         |            |                |                    | CRM                               |                                                         |
|                         |            |                |                    | OEX                               |                                                         |
| `deliveryParams`        | object     | 1              |                    | Параметры доставки                 |                                                         |
| `deliveryType`          | enum       | 1              | PICKUP             | Тип доставки                       |                                                         |
|                         |            |                |                    | PICKUP - Самовывоз из магазина Mvideo |                                                         |
|                         |            |                |                    | PARTNER_PICKUP - Самовывоз из пункта выдачи |                                                         |
|                         |            |                |                    | INTERVAL_DELIVERY - Доставка к определенному интервалу времени |                                                         |
|                         |            |                |                    | SHORT_INTERVAL_DELIVERY - Доставка к узкому интервалу времени |                                                         |
|                         |            |                |                    | ETA_DELIVERY - Доставка за определенное время на такси |                                                         |
|                         |            |                |                    | ELECTRONIC_DELIVERY - Электронная доставка ЦК |                                                         |
| `deliveryDate`          | date       | 1              | 2022-02-22         | Предполагаемая дата доставки       |                                                         |
| `deliveryTimeFrom`      | time       | 0              | 17:00              | Начало интервала доставки          |                                                         |
| `deliveryTimeTo`        | time       | 0              | 18:00              | Конец интервала доставки           |                                                         |
| `comment`               | string     | 0              | раз два три        | Комментарий к доставке              |                                                         |
| `express`               | boolean    | 0              | true               | Признак экспресс доставки           |                                                         |
| `hiTechnic`             | boolean    | 0              | false              | Признак доставки комфорт           |                                                         |
| `deliveryItem`          | array of object | 1        |                    | Список позиций заказа               |                                                         |
| `positionNumber`        | number     | 1              | 10                 | Номер позиции заказа                |                                                         |
| `extItemId`             | string     | 1              | 4ac60e6e-ed67-4bc0-8c74-aeb07dfec106 | Идентификатор позиции внешней системы |                                                         |
| `itemType`              | enum       | 1              | PRODUCT            | Тип позиции                        |                                                         |
|                         |            |                |                    | PRODUCT_KIT – Товар (заголовок комплекта) |                                                         |
|                         |            |                |                    | PRODUCT – Товар                    |                                                         |
|                         |            |                |                    | DELIVERY – Доставка                |                                                         |
|                         |            |                |                    | GIFT – Подарок                     |                                                         |
|                         |            |                |                    | SERVICE - Услуга                   |                                                         |
| `materialNumber`        | string     | 1              | 20065055           | SKU позиции                        |                                                         |
| `status`                | enum       | 0              | CREATED            | Статус исполнения позиции          |                                                         |
|                         |            |                |                    | Статусная модель                   |                                                         |
|                         |            |                |                    | CREATED                            |                                                         |
|                         |            |                |                    | IN_PROGRESS                        |                                                         |
|                         |            |                |                    | DELIVERED                          |                                                         |
|                         |            |                |                    | NOT_DELIVERED                      |                                                         |
|                         |            |                |                    | REJECTED                           |                                                         |
|                         |            |                |                    | ERROR                              |                                                         |
|                         |            |                |                    | CANCELED                           |                                                         |
| `reserveObject`         | string     | 0              | S790               | Объект резерва                     |                                                         |
| `handoverObject`        | string     | 0              | S790               | Объект выдачи                      |                                                         |
| `deliveryDate`          | date       | 1              | 2022-02-22         | Дата доставки/выдачи               |                                                         |
| `quantity`              | number     | 1              | 1                  | Количество товара в позиции        |                                                         |
| `quantityUnit`          | string     | 1              | шт.                | Единица измерения                  |                                                         |
|                         |            |                |                    | Коды единиц измерения (все, что с *) |                                                         |
| `priceDiscount`         | number     | 1              | 1900.00            | Цена за единицу товара в рублях с учетом скидки |                                                         |
| `priceTotal`            | number     | 1              | 2000.00            | Стоимость позиции в рублях без учета скидки |                                                         |
| `showcaseSample`        | boolean    | 0              | false              | Признак витринного образца          |                                                         |
| `transferDocumentParams`| object     | 0              |                    | Информация о документах транспортировки |                                                         |
| `shipmentGuid`          | string     | 1              | 2136afba-9d8d-40a4-b2bd-f1192a50385a | GUID партии поставки          |                                                         |
| `itemDocNumber`         | string     | 1              | 0000000001         | Номер позиции документа передачи |                                                         |
| `reserveInfo`           | object     | 0              |                    | Информация о резерве                |                                                         |
| `reserveNumber`         | string     | 1              | 123573565          | Номер документа резервирования     |                                                         |
| `reserveNumberPosition` | string     | 0              | 000001             | Позиция документа резервирования SAP |                                                         |
| `reserveRequestId`      | string     | 0              | CRM000003215       | Уникальный идентификатор запроса на резерв |                                                         |
| `externalPositionNumber`| string     | 0              | 001                | Номер позиции внешнего документа резерва |                                                         |
| `address`               | object     | 0              |                    | Информация об адресе доставки клиента |                                                         |
| `postCode`              | string     | 0              | 390048             | Почтовый индекс                    |                                                         |
| `city`                  | string     | 0              | Рязань             | Город                               |                                                         |
| `city2`                 | string     | 0              | п. Воробушки      | Населенный пункт (settlement)      |                                                         |
| `region`                | string     | 0              | Пермский край      | Регион                              |                                                         |
| `street`                | string     | 0              | Зубковой           | Улица                               |                                                         |
| `house`                 | string     | 0              | 31                 | Дом                                 |                                                         |
| `building`              | string     | 0              | 2                  | Корпус                              |                                                         |
| `intercom`              | string     | 0              | 322B12             | Код домофона                        |                                                         |
| `apartment`             | string     | 0              | 214                | Квартира                            |                                                         |
| `entrance`              | string     | 0              | 6                  | Подъезд                             |                                                         |
| `floor`                 | string     | 0              | 4                  | Этаж                                |                                                         |
| `kladrCode`             | string     | 0              | 77000000000151900 | Код КЛАДР                          |                                                         |
| `payment`               | object     | 1              |                    | Информация об оплате                |                                                         |
| `paymentType`           | enum       | 1              | ONLINE             | Виды оплат                         |                                                         |
|                         |            |                |                    | ON_RECEIPT - при получении          |                                                         |
|                         |            |                |                    | ONLINE - онлайн                    |                                                         |
|                         |            |                |                    | BANK_TRANSFER - перевод            |                                                         |
|                         |            |                |                    | CREDIT - кредит                    |                                                         |
|                         |            |                |                    | IN_STORE - оплата в магазине       |                                                         |
|                         |            |                |                    | EMONEY - Электронные деньги        |                                                         |
| `paymentStatus`         | enum       | 1              | PAID               | Статусы оплаты                     |                                                         |
|                         |            |                |                    | NOT_PAID - Не оплачен              |                                                         |
|                         |            |                |                    | PAID - Оплачен                     |                                                         |
|                         |            |                |                    | BANK_APPROVED - Одобрено банком     |                                                         |
| `recipient`             | object     | 1              |                    | Информация о получателе            |                                                         |
| `recipientName`         | string     | 1              | Тестовый Тест Тестович | ФИО получателя              |                                                         |
| `phoneNumber`           | string     | 1              | +79129999999      | Контактный телефон получателя     |                                                         |
| `addPhoneNumber`        | string     | 0              | +79129999999      | Дополнительный телефон получателя  |                                                         |

2.2  Include A Shared Block + Фильтр таблиц
