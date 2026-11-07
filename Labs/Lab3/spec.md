# Specification

## 1. Обраний Use Case та опис

Для деталізації внутрішньої взаємодії обрано із Lab2 фокусний прецедент UC1_1 "Забронювати стіл (до 3 столів)" (US1.1, REQ1-REQ4), а також пов'язані сценарії підсистеми управління посадкою та бронюванням (UC1_2-UC1_4, UC1_8, US1.2-US1.4, REQ5-REQ8)

Мета прецеденту: забезпечити клієнту або адміністратору можливість бронювати від 1 до 3 вільних столів із гарантією відсутності подвійного бронювання, автоматичним переведенням столів у статус "Заброньовано" та відправкою підтвердження

## 2. Учасники взаємодії

- Client / Admin (Актори, відповідають сутностям USERS та ROLES з ER-моделі Lab1)
- ClientApp (Клієнтський інтерфейс: мобільний застосунок або вебсайт)
- ReservationService (Бекенд): контролер і бізнес-логіка обробки бронювань та посадки
- TABLE (Сутність БД з ER-моделі Lab1): таблиця столів залу зі статусами FREE, RESERVED, OCCUPIED
- RESERVATION (Сутність БД з ER-моделі Lab1): таблиця бронювань зі статусами CONFIRMED, SEATED, CANCELLED
- NotificationService (Зовнішній сервіс): система надсилання Push / SMS сповіщень

## 3. Сценарії та кроки взаємодії

Коли зареєстрований клієнт або адміністратор ресторану бажає забронювати столи та керувати посадкою відвідувачів, система дозволяє гарантувати наявність вільних місць та організувати комфортне розміщення гостей

### 3.1. Сценарій 1: Автоматичний вибір вільного столу

Опис: Клієнт вказує дату, час і кількість гостей без вибору конкретного столу, а система підбирає та бронює вільний стіл (або декілька до 3 столів)

1. Client вводить дату, час, кількість гостей у ClientApp і викликає submitAutoReservation(date, time, guests_count)
2. ClientApp передає запит createAutoReservation(client_id, start_time, end_time, guests_count) до ReservationService
3. ReservationService перевіряє, чи є достатня кількість відповідних вільних місць на цей час і дату, та викликає findAvailableTables(start_time, end_time, guests_count) до TABLE
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
5. ReservationService створює запис бронювання saveReservation(user_id, start_time, end_time, status="CONFIRMED") у RESERVATION
6. ReservationService оновлює статус столу 5 через updateTableStatus(table_id=5, status="RESERVED") у TABLE
7. ReservationService викликає sendConfirmation(user_id, reserv_details) у NotificationService
8. ClientApp отримує відповідь reservationConfirmed і відображає користувачу підтвердження бронювання столу 5

### 3.3. Сценарій 3: Відсутність вільних столів у залі

Опис: На обраний інтервал часу всі відповідні столи в залі вже зайняті або заброньовані, тому система пропонує альтернативні доступні столи або інший час візиту

1. Актор надсилає запит на бронювання checkAndBook(date, time, guests_count) через ClientApp
2. ReservationService звертається до TABLE із запитом findAvailableTables(start_time, end_time, guests_count)
3. TABLE повертає порожній список, оскільки всі столи на цей час зайняті
4. ReservationService запускає пошук findAlternativeSlotsAndTables(date, guests_count) через TABLE та RESERVATION
5. TABLE та RESERVATION повертають список доступних варіантів alternativeOptions
6. ReservationService повертає статус noTablesAvailable разом зі списком alternativeOptions до ClientApp
7. ClientApp сповіщає користувача про відсутність вільних місць на обраний час і відображає запропоновані альтернативні столи або інші часові проміжки

### 3.4. Сценарій 4: Відхилення бронювання при перевищенні ліміту в 3 столи

Опис: Користувач намагається обрати більше 3 столів у межах одного бронювання, і система блокує операцію до запису в базу даних

1. Актор обирає 4 або більше столів на карті залу і викликає submitManualReservation(date, time, guests, table_id=[1, 2, 3, 4]) у ClientApp
2. ClientApp передає запит createReservation(user_id, start_time, end_time, table_id) до ReservationService
3. ReservationService викликає перевірку validateTableLimit(table_id), яка виявляє порушення умови count(table_id) > 3
4. ReservationService блокує подальше виконання операції (записи в БД не створюються)
5. ReservationService повертає помилку LimitExceededError("Максимальна кількість столів для одного бронювання: 3") до ClientApp
6. ClientApp відображає користувачу повідомлення про помилку та залишає форму відкритою для зміни вибору столів

### 3.5. Сценарій 5: Скасування бронювання клієнтом

Опис: Клієнт або адміністратор скасовує активне бронювання до початку візиту, і система звільняє закріплені столи

1. Актор відкриває список активних бронювань у ClientApp і викликає cancelReservation(reserv_id)
2. ClientApp передає запит cancelBooking(user_id, reserv_id) до ReservationService
3. ReservationService перевіряє статус броні у RESERVATION та оновлює його через updateReservationStatus(reserv_id, status="CANCELLED")
4. ReservationService звільняє закріплений стіл 5 та оновлює його статус через updateTableStatus(table_id=5, status="FREE") у TABLE
5. ReservationService викликає sendCancellationNotice(user_id, reserv_id) у NotificationService
6. ClientApp отримує підтвердження скасування reservationCancelled і відображає оновлений статус користувачу

### 3.6. Сценарій 6: Посадка гостей за живою чергою

Опис: До закладу приходять гості без попередньої броні, і адміністратор оперативно закріплює за ними вільний стіл на інтерактивній карті залу

1. Admin відкриває інтерактивну карту залу в ClientApp і викликає getHallMapStatus()
2. ReservationService запитує поточні статуси всіх столів getAllTables() у TABLE та повертає актуальну схему залу (де стіл 8 має статус FREE)
3. Admin обирає вільний стіл 8 для компанії гостей і викликає seatWalkInGuests(admin_id, table_id=8)
4. ReservationService у транзакції оновлює статус столу 8 через updateTableStatus(table_id=8, status="OCCUPIED") у TABLE
5. ReservationService повертає новий стан карти залу tableOccupied(table_id=8) у ClientApp для синхронізації між пристроями

### 3.7. Сценарій 7: Завчасне нагадування про бронювання

Опис: За налаштований інтервал часу до початку заброньованого візиту система автоматично надсилає клієнту нагадування з деталями броні

1. За розкладом таймера ReservationService викликає checkUpcomingReservations(current_time, reminder_interval) до RESERVATION
2. RESERVATION повертає список підтверджених бронювань (status="CONFIRMED"), час початку яких настає через заданий інтервал
3. Для кожного знайденого бронювання ReservationService формує деталі візиту (час, номер столу з TABLE) та викликає sendReminder(client_id, reserv_details) у NotificationService
4. NotificationService доставляє клієнту повідомлення з нагадуванням у ClientApp та повертає статус доставки reminderSent до ReservationService

## 4. Посилання на використані матеріали

- Матеріали Lab1:
  - [Spec](../Lab1/spec.md)
  - [ER-модель](../Lab1/models/base_model.mmd)
- Матеріали Lab2:
  - [Spec](../Lab2/spec.md)
  - [Системні вимоги та матриця трасування](../Lab2/requirements.md)
  - [Use Case діаграма підсистеми бронювання (UC1_1)](../Lab2/models/boarding_and_reservation.mmd)
  - [Gherkin-сценарії до UC1_1](../Lab2/features/boarding_and_reservation.feature)
