# 🚕 Дневник смен водителя (Driver Shift Diary)

Комплексное кроссплатформенное решение для водителей такси и курьеров: быстрый учет смен, расчет комиссии сервиса/парка, общей выручки и дохода **«на руки»**, с разделением по способам оплаты (наличные и банковская карта).

Проект включает в себя:
1. **REST API & Backend** (FastAPI, Python, Pydantic v2, потокобезопасное хранилище).
2. **Мобильное веб-приложение** (адаптивный интерфейс в виде кассового термочека, быстрый ввод, жесты свайпа).
3. **Android-приложение** в папке `android/` (проект Android Studio на Kotlin, готовый к сборке APK).
4. **Набор автоматических тестов** (pytest).

---

## 🏗 Архитектура решения

```text
test_driver/
├── app/                  # Серверная часть и веб-клиент
│   ├── main.py           # FastAPI роутер, эндпоинты, CORS, статика
│   ├── models.py         # Pydantic v2 схемы и строгая валидация
│   ├── storage.py        # Потокобезопасное хранилище с защитой от дублей
│   ├── summary.py        # Чистая финансовая логика (точность Decimal)
│   └── static/           # Мобильный веб-интерфейс
│       ├── index.html    # Разметка с термочеком и Bottom Sheet шторкой
│       ├── style.css     # Адаптивные стили (Mobile First, эргономика)
│       └── app.js        # Клиентская логика, свайпы, Web Share API
├── android/              # Проект Android-приложения (Android Studio)
│   ├── build.gradle.kts  # Конфигурация сборки Gradle
│   ├── settings.gradle.kts
│   ├── gradlew / gradlew.bat
│   └── app/
│       ├── build.gradle.kts
│       └── src/main/
│           ├── AndroidManifest.xml
│           ├── java/com/driver/diary/MainActivity.kt  # Kotlin WebView + JS Bridge
│           └── assets/  # Офлайн-бандл (работает без подключения к сети)
├── data/
│   ├── trips.json        # Файл базы данных поездок
│   └── trips_sample.json # Исходные демонстрационные данные
├── tests/                # Набор модульных и интеграционных тестов
│   ├── test_api.py       # Тесты API эндпоинтов и кодов ответов
│   ├── test_duplicates.py# Тесты защиты от дублей
│   └── test_summary.py   # Тесты расчетов выручки и комиссий
├── requirements.txt      # Python-зависимости
├── run.py                # Скрипт быстрого запуска сервера
└── README.md             # Документация проекта
```

---

## ✨ Ключевые возможности

### 1. Финансовый учет смен
- **Чистый доход «на руки»** (`net_income = total_amount - total_commission`).
- **Общая выручка** («грязными»).
- **Комиссия сервиса/парка** (сумма и процентная доля).
- **Разделение по способам оплаты**:
  - Наличные: сумма и число заказов (`cash_amount`, `cash_trips`).
  - Карта (безнал): сумма и число заказов (`card_amount`, `card_trips`).
- **Финансовая точность**: расчет ведется через модуль `decimal.Decimal` с банковским округлением `ROUND_HALF_UP` во избежание погрешностей чисел с плавающей точкой (`float`).

### 2. Валидация и защита от дублей (Идемпотентность)
- Сумма поездки строго `> 0`.
- Время окончания строго позже времени начала (`end > start`).
- Проверка совместимости часовых поясов (ISO 8601).
- Комиссия неотрицательна и не может превышать сумму заказа.
- **Защита от повторных отправок**: при повторной отправке поездки (по уникальному `id` или по совпадению всех параметров `start`, `end`, `amount`, `payment`, `commission`) система **не создает дубликат**, а возвращает существующую запись со статусом `is_duplicate: true`.

### 3. Представление результатов в виде термочека
Результаты смены формируются в виде реалистичного кассового чека (как в веб-интерфейсе, так и в Android-приложении):

```text
       Дневник смен       
        01.10.2026        

08:10–08:32 карта          2 400 ₸
09:05–09:20 нал.           1 500 ₸
----------------------------------
Поездок                          2
Выручка                    3 900 ₸
Комиссия                    -585 ₸
Наличные / карта     1 500 / 2 400
----------------------------------
На руки                    3 315 ₸
```

---

## 🚀 Быстрый запуск веб-версии и сервера

### 1. Установка зависимостей
```bash
pip install -r requirements.txt
```

### 2. Запуск приложения
```bash
python run.py
```
Скрипт автоматически запустит сервер и выведет адреса:
- **На компьютере (ПК)**: [http://127.0.0.1:8000](http://127.0.0.1:8000)
- **С телефона по домашнему/офисному Wi-Fi**: `http://<ваш-локальный-IP>:8000`
- **Интерактивная документация API (Swagger UI)**: [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)
- **Текстовый чек**: [http://127.0.0.1:8000/api/receipt?date=2026-10-01](http://127.0.0.1:8000/api/receipt?date=2026-10-01)

---

## 📱 Запуск и сборка Android-приложения

Проект Android находится в папке `android/`.

### Способ 1: Открытие в Android Studio (Рекомендуется)
1. Запустите **Android Studio**.
2. Выберите **File → Open...** и укажите папку:
   `test_driver/android`
3. Дождитесь автоматической синхронизации Gradle.
4. Нажмите зелёную кнопку **Run ▶** для запуска на подключенном смартфоне или эмуляторе.

### Способ 2: Сборка APK через командную строку
В папке `android/`:
```bash
# Windows:
gradlew.bat assembleDebug

# Linux / macOS:
./gradlew assembleDebug
```
Собранный установочный файл APK появится по пути:
`android/app/build/outputs/apk/debug/app-debug.apk`

### Особенности Android-приложения:
- **Офлайн-режим**: приложение содержит встроенный бандл ассетов (`file:///android_asset/`), поэтому работает полноценно даже без подключения к интернету.
- **Шторка отправки отчета**: кнопка «📤 Чек» вызывает нативный системный диалог Android `Intent.ACTION_SEND` (можно сразу отправить чек смены в WhatsApp, Telegram или скопировать).
- **Жесты свайпа**: переключение между сменами смахиванием влево/вправо по экрану.
- **Pull-to-Refresh**: потягивание экрана вниз (`SwipeRefreshLayout`) для обновления данных.

---

## 🧪 Запуск автоматических тестов

Все тесты покрывают расчеты сводки, граничные случаи и защиту от дублей:

```bash
python -m pytest -v
```

Вывод тестов:
```text
collected 19 items

tests/test_api.py::test_get_trips_for_day PASSED                         [  5%]
tests/test_api.py::test_get_trips_default_date PASSED                    [ 10%]
tests/test_api.py::test_get_day_summary_endpoint PASSED                  [ 15%]
tests/test_api.py::test_get_day_receipt_text_endpoint PASSED             [ 21%]
tests/test_api.py::test_add_trip_success PASSED                          [ 26%]
tests/test_api.py::test_add_trip_duplicate_idempotent PASSED             [ 31%]
tests/test_api.py::test_add_trip_duplicate_strict_conflict PASSED        [ 36%]
tests/test_api.py::test_validation_amount_must_be_positive PASSED        [ 42%]
tests/test_api.py::test_validation_end_must_be_after_start PASSED        [ 47%]
tests/test_api.py::test_validation_commission_cannot_exceed_amount PASSED [ 52%]
tests/test_api.py::test_validation_invalid_payment PASSED                [ 57%]
tests/test_api.py::test_delete_trip PASSED                               [ 63%]
tests/test_duplicates.py::test_duplicate_protection_by_id PASSED         [ 68%]
tests/test_duplicates.py::test_duplicate_protection_by_content_without_id PASSED [ 73%]
tests/test_duplicates.py::test_different_trips_are_added PASSED          [ 78%]
tests/test_summary.py::test_calculate_day_summary_sample_data PASSED     [ 84%]
tests/test_summary.py::test_calculate_day_summary_empty PASSED           [ 89%]
tests/test_summary.py::test_calculate_day_summary_decimal_precision PASSED [ 94%]
tests/test_summary.py::test_filter_trips_by_date PASSED                  [100%]

======================== 19 passed in 0.27s ========================
```

---

## 📡 Основные API Эндпоинты

| Метод | Путь | Описание |
| :--- | :--- | :--- |
| `GET` | `/api/trips?date=YYYY-MM-DD` | Список поездок и финансовая сводка за день |
| `GET` | `/api/summary?date=YYYY-MM-DD` | Сводка за день (выручка, комиссия, на руки, нал/безнал) |
| `GET` | `/api/receipt?date=YYYY-MM-DD` | Текстовый кассовый чек смены |
| `GET` | `/api/dates` | Список всех дат со сменами |
| `POST` | `/api/trips` | Добавление поездки с валидацией и защитой от дублей |
| `DELETE` | `/api/trips/{id}` | Удаление поездки по ID |
| `POST` | `/api/reset` | Сброс данных к начальному состоянию |

---

## 📄 Лицензия
MIT. Свободное использование.
