# price-data — цены Сыктывкара в JSON

Снапшоты цен и каталога **Магнита** и **Ленты** по Сыктывкару. Данные собираются
локально парсером (`export-json.ts`, проект `price-export`) и публикуются здесь
для мобильного приложения и любого другего клиента.

Репозиторий открытый, файлы отдаются через GitHub Pages с CORS
(`Access-Control-Allow-Origin: *`) — читать можно обычным `fetch`.

## Что внутри

| Сеть | Точки | Товаров | Полок | Собрано (UTC) |
|---|---|---|---|---|
| **Магнит** | `113813` (пр-кт Бумажников, 45) | 4 984 | 26 | 2026-11-11 00:13 |
| **Лента** | `3587`, `3615`, `5357` (ТК22, ТК106, ТК2671) | 30 730 | 300 | 2026-11-10 23:34 |

Всего: **35 714 товаров**, ~22 МБ несжатого JSON. По сети файлы разложены по
категориям («полкам»), чтобы клиент качал только нужную, а не весь каталог.

## Как читать

Базовый URL:

```
https://myasorybov.github.io/price-data/
```

| Что | Путь |
|---|---|
| Сводка по сетям | [`data/index.json`](data/index.json) |
| Полки Магнита | [`data/magnit/syktyvkar/index.json`](data/magnit/syktyvkar/index.json) |
| Полка «молоко» (Магнит) | [`data/magnit/syktyvkar/moloko.json`](data/magnit/syktyvkar/moloko.json) |
| Полки Ленты | [`data/lenta/syktyvkar/index.json`](data/lenta/syktyvkar/index.json) |
| Полка «молоко» (Лента) | [`data/lenta/syktyvkar/moloko.json`](data/lenta/syktyvkar/moloko.json) |

Пример:

```bash
curl -s https://myasorybov.github.io/price-data/data/magnit/syktyvkar/moloko.json \
  | python3 -c 'import json,sys; d=json.load(sys.stdin); print(d["count"], d["collected_at"]); print(d["products"][0]["name"], d["products"][0]["price"])'
```

### Структура файлов

```
data/
├── index.json                    # сводка: сети, файлы, число товаров, дата
├── magnit/syktyvkar/
│   ├── index.json                # список полок: имя, размер, sha256, число товаров
│   ├── moloko.json               # одна полка
│   └── …
└── lenta/syktyvkar/
    ├── index.json
    └── …
```

### Схема полки

```json
{
  "schema": "price-aggregator/shard@1",
  "store": "lenta",
  "store_name": "Лента",
  "city": "syktyvkar",
  "store_code": "3587,3615",
  "category": "молоко",
  "collected_at": "2026-11-10T23:34:21.519Z",
  "count": 285,
  "products": [
    {
      "id": "lenta-359472",
      "sku": "359472",
      "name": "Молоко пастеризованное ПРАВИЛЬНОЕ МОЛОКО 3,2–4% 2000мл",
      "brand": null,
      "unit": "2000мл",
      "price": 219.99,
      "old_price": 273.69,
      "promo_price": 219.99,
      "in_stock": true,
      "category": "молоко",
      "image": "https://cdn.api.lenta.com/…",
      "url": "https://lenta.com/product/…-359472/",
      "store": "lenta",
      "store_code": "3587",
      "city": "syktyvkar",
      "collected_at": "2026-11-10T23:34:21.519Z"
    }
  ]
}
```

Ключ товара — пара `(store_code, id)`: у разных магазинов одной сети `id`
совпадает, а цены разные.

## Как обновляется

Парсер запускается **локально** (GitHub-раннеры для этого не подходят: сети режут
чужие адреса). Сбор → раскладка по полкам → публикация:

```bash
npm run export:json -- \
  --city syktyvkar --stores magnit --max-stores 1 \
  --mode catalog --shard --out data

./deploy/publish.sh ../price-data ../price-export/data
```

`publish.sh` синхронизирует `data/` и пересоздаёт ветку `main` одним коммитом
(orphan + `--force`), поэтому история git не раздувается от ежедневных снапшотов.

## Оговорки

- Данные получены из неофициальных API сетей. Условия сайтов это запрещают, сеть
  может сменить формат ответа или заблокировать доступ. Выгрузка — раз в сутки.
- Каталог Ленты собирается объединением выдачи по названиям категорий (живой
  каталог сети закрыт), поэтому он неполон.
- Снапшот отражает цены на момент `collected_at`; в магазине они могут отличаться.
- Актуальность проверяй по `collected_at` в `data/index.json`.

## Источники

- [Магнит](https://magnit.ru) — `https://magnit.ru`
- [Лента](https://lenta.com) — `https://lenta.com`
- Скрипт выгрузки: `export-json.ts` и `deploy/publish.sh` из проекта `price-export`
