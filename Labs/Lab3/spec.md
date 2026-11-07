# Specification

## 1. Обраний Use Case та опис

Для деталізації внутрішньої взаємодії обрано із Lab2 - UC1_1 "Забронювати стіл (до 3 столів)", який реалізує користувацьку історію US1.1 та функціональні вимоги REQ1-REQ4

Мета прецеденту: забезпечити клієнту або адміністратору можливість бронювати від 1 до 3 вільних столів із гарантією відсутності подвійного бронювання, автоматичним переведенням столів у статус "Заброньовано" та відправкою підтвердження

## 2. Учасники взаємодії

- Client / Admin (Актори)
- ClientApp (Інтерфейс)
- ReservationService (Бекенд): контролер і бізнес-логіка обробки бронювань
- TABLE
- RESERVATION
- NotificationService (Зовнішній сервіс)

## 3. Сценарії та кроки взаємодії

Як зареєстрований клієнт або адміністратор ресторану бажає забронювати столи та керувати посадкою відвідувачів, система дозволяє гарантувати наявність вільних місць та організувати комфортне розміщення гостей

### 3.1. Сценарій 1: Автоматичний вибір вільного столу

Опис: Клієнт вказує дату, час і кількість гостей без вибору конкретного столу, а система підбирає та бронює вільний стіл (або декілька до 3 столів)

1. Client вводить дату, час, кількість гостей у ClientApp і викликає submitAutoReservation(date, time, guests_count)
2. ClientApp передає запит createAutoReservation(client_id, start_time, end_time, guests_count) до ReservationService
3. ReservationService перевіряє чи є достатня кількість підходящих вільних місць на цей час і дату та викликає findAvailableTables(start_time, end_time, guests_count) до TABLE
4. TABLE повертає список доступних столів зі статусом FREE, які мають змогу вмістити вказану кількість гостей (у межах від 1 до 3 столів)
5. ReservationService відкриває транзакцію та створює запис бронювання saveReservation(client_id, start_time, end_time, status="CONFIRMED") у RESERVATION
6. ReservationService закріплює обрані столи та оновлює їхній статус updateTableStatus(table_id, status="RESERVED") у TABLE
7. ReservationService викликає sendConfirmation(client_id, reserv_details) у NotificationService для надсилання повідомлення клієнту
8. ReservationService повертає успішний результат у ClientApp, який відображає клієнту деталі підтвердженого бронювання

### 3.2. Сценарій 2: Ручний вибір конкретного столу

Опис: Користувач самостійно обирає конкретні вільні столи (від 1 до 3 столів) зі списку доступних на обраний час

1. Актор обирає дату, час і кількість гостей у ClientApp для перегляду схеми вільних столів getAvailableTables(date, time, guests)
2. ReservationService запитує актуальні статуси столів checkTablesAvailability(start_time, end_time) у TABLE та повертає перелік столів зі статусом FREE
3. Актор обирає конкретний стіл 5 (або декілька від 1 до 3 столів) і натискає підтвердження submitManualReservation(date, time, guests, table_id=[5])
4. ReservationService виконує перевірку validateTableLimit(table_id) (кількість столів <= 3) та повторно перевіряє у транзакції SERIALIZABLE, що стіл 5 досі має статус FREE у TABLE
5. ReservationService створює запис бронювання insertReservation(user_id, start_time, end_time, status="CONFIRMED") у RESERVATION
6. ReservationService оновлює статус столу 5 через setTableStatus(table_id=5, status="RESERVED") у TABLE
7. ReservationService викликає sendBookingNotification(user_id, reservationData) у NotificationService
8. ClientApp отримує відповідь reservationConfirmed і відображає користувачу підтвердження бронювання столу 5

### 3.3. Сценарій 3: Відсутність вільних столів у залі

Опис: На обраний інтервал часу всі відповідні столи в залі вже зайняті або заброньовані, тому система пропонує альтернативний час візиту

1. Актор надсилає запит на бронювання checkAndBook(date, time, guests_count) через ClientApp
2. ReservationService звертається до TABLE із запитом findFreeTables(start_time, end_time, guests_count)
3. TABLE повертає порожній список, оскільки всі столи на цей час зайняті
4. ReservationService запускає пошук findAlternativeSlotsAndTables(date, guests_count) через TABLE та RESERVATION
5. TABLE та RESERVATION повертають список доступних варіантів alternativeOptions
6. ReservationService повертає статус noTablesAvailable разом зі списком alternativeOptions до ClientApp
7. ClientApp сповіщає користувача про відсутність вільних місць на обраний час і відображає запропоновані інші часові проміжки

### 3.4. Сценарій 4: Відхилення бронювання при перевищенні ліміту в 3 столи

Опис: Користувач намагається обрати більше 3 столів у межах одного бронювання, і система блокує операцію до звернення на запис у базу даних

1. Актор обирає 4 або більше столів на карті залу і викликає submitManualReservation(date, time, guests, table_id=[1, 2, 3, 4]) у ClientApp
2. ClientApp передає запит createReservation(user_id, start_time, end_time, table_id) до ReservationService
3. ReservationService викликає перевірку validateTableLimit(table_id), яка виявляє порушення умови count(table_id) > 3
4. ReservationService блокує подальше виконання операції (записи в БД не створюються)
5. ReservationService повертає помилку LimitExceededError("Максимальна кількість столів для одного бронювання: 3") до ClientApp
6. ClientApp відображає користувачу повідомлення про помилку та залишає форму відкритою для зміни вибору столів

## 4. Посилання на використані матеріали

- Матеріали Lab2:
  - [Spec](../Lab1/spec.md)
  - [ER-модель](../Lab1/models/base_model.mmd)
- Матеріали Lab2:
  - [Spec](../Lab2/spec.md)
  - [Системні вимоги та матриця трасування](../Lab2/requirements.md)
  - [Use Case діаграма підсистеми бронювання (UC1_1)](../Lab2/models/boarding_and_reservation.mmd)
  - [Gherkin-сценарії до C1_1](../Lab2/features/boarding_and_reservation.feature)
