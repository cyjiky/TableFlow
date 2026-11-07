# Specification

## 1. Обраний Use Case та опис

Для деталізації внутрішньої взаємодії обрано із Lab2 фокусний прецедент UC1_1 "Забронювати стіл (до 3 столів)" (US1.1, REQ1-REQ4), а також пов'язані сценарії підсистеми управління посадкою та бронюванням (UC1_2-UC1_4, UC1_8, US1.2-US1.4, REQ5-REQ8)

**Мета прецеденту:** забезпечити клієнту або адміністратору можливість бронювати від 1 до 3 вільних столів із гарантією відсутності подвійного бронювання, автоматичним переведенням столів у статус "Заброньовано", керуванням посадкою в залі та відправкою сповіщень

## 2. Учасники взаємодії

- Client / Admin (Актори, відповідають сутностям USERS та ROLES з ER-моделі Lab1 з ідентифікаторами client_id та admin_id)
- ClientApp (Інтерфейс користувача: мобільний застосунок або вебсайт для клієнта, вебпанель на ноутбуці або комп'ютері для адміністратора)
- ReservationService (Бекенд): контролер і бізнес-логіка обробки бронювань та посадки
- TABLE (Сутність БД з ER-моделі Lab1): таблиця столів залу з полями table_number, capacity та поточними статусами FREE, RESERVED, OCCUPIED
- RESERVATION (Сутність БД з ER-моделі Lab1): таблиця бронювань з полями client_id, admin_id, часовими інтервалами start_time, end_time та статусами CONFIRMED, RESCHEDULED, CANCELLED
- NotificationService (Зовнішній сервіс): система надсилання Push / SMS сповіщень

## 3. Сценарії, комбіновані фрагменти та кроки взаємодії

Коли зареєстрований клієнт або адміністратор ресторану бажає забронювати столи та керувати посадкою відвідувачів, система дозволяє гарантувати наявність вільних місць та організувати комфортне розміщення гостей

У кожному сценарії для Sequence-діаграм застосовується правило нерозривності ліній активації: учасники ClientApp та ReservationService, які координують процес, розпочинають свою активацію до входу в комбіновані фрагменти (alt, opt, loop) і завершують її лише після повного виходу з фрагмента та повернення фінальної відповіді

### 3.1. Сценарій 1: Автоматичний вибір вільного столу

Опис: Авторизований клієнт з роллю CLIENT вказує дату, час і кількість гостей без вибору конкретного столу, а система перевіряє вільні часові слоти, автоматично підбирає і закріплює вільний стіл (від 1 до 3 столів), переводить його у статус "Заброньовано" та надсилає підтвердження через Notification Service

Комбіновані фрагменти та Activation Bars:

- Лінія активації ClientApp стартує на кроці 1 і завершується після кроку 10
- Лінія активації ReservationService стартує на кроці 2 до початку фрагментів loop та opt і завершується на кроці 10 після повернення відповіді у ClientApp
- Використовується фрагмент loop для оновлення статусу кожного автоматично підібраного столу (від 1 до 3 столів) та фрагмент opt для відправки зовнішнього сповіщення

Кроки взаємодії:

1. Client (авторизований з роллю CLIENT) вводить дату, час, кількість гостей у ClientApp і викликає submitAutoReservation(date, time, guests_count) (початок активації ClientApp)
2. ClientApp передає запит createAutoReservation(client_id, start_time, end_time, guests_count) до ReservationService (початок активації ReservationService)
3. ReservationService перевіряє зайняті часові інтервали та викликає getReservedTableIds(start_time, end_time) до RESERVATION (активація та повернення списку зайнятих столів reserved_table_ids з RESERVATION)
4. ReservationService викликає findAvailableTables(guests_count, reserved_table_ids) до TABLE для підбору вільних столів потрібної місткості
5. TABLE повертає список підібраних доступних столів availableTables зі статусом FREE (від 1 до 3 столів у межах однієї броні) та завершує активацію
6. ReservationService відкриває транзакцію і створює запис бронювання saveReservation(client_id, admin_id=null, start_time, end_time, table_ids, status="CONFIRMED") у RESERVATION, отримуючи створений reserv_id
7. Фрагмент loop [Для кожного підібраного столу з availableTables (від 1 до 3 столів)]:
   - ReservationService закріплює стіл та оновлює його статус через updateTableStatus(table_id, status="RESERVED") у TABLE
   - TABLE фіксує статус RESERVED і повертає підтвердження statusUpdated
8. Фрагмент opt [Надсилання Push / SMS підтвердження клієнту]:
   - ReservationService викликає sendConfirmation(client_id, reserv_details) у NotificationService
   - NotificationService надсилає клієнту підтвердження бронювання і повертає статус confirmationSent
9. ReservationService повертає успішний результат reservationConfirmed(reserv_details) у ClientApp (завершення активації ReservationService)
10. ClientApp відображає клієнту підтвердження та деталі створеного бронювання (завершення активації ClientApp)

### 3.2. Сценарій 2: Ручний вибір конкретного столу

Опис: Авторизований користувач з роллю CLIENT або ADMIN обирає дату, час, кількість гостей та конкретний вільний стіл №5 (від 1 до 3 столів) зі списку доступних і підтверджує бронювання, після чого система перевіряє доступність столу №5 на обраний інтервал, переводить його у статус "Заброньовано" і надсилає клієнту підтвердження з деталями візиту

Комбіновані фрагменти та Activation Bars:

- На етапі підтвердження бронювання активація ClientApp (крок 4) та ReservationService (крок 5) починається до умовного фрагмента alt і завершується після виходу з фрагмента alt на кроці 11
- Використовується фрагмент alt для транзакційної перевірки доступності обраного столу №5 на заданий часовий інтервал та вкладений фрагмент loop для закріплення обраних столів (від 1 до 3)

Кроки взаємодії:

1. Актор (Client або Admin) обирає дату, час і кількість гостей у ClientApp для перегляду доступних столів getAvailableTables(date, time, guests_count) (початок першої активації ClientApp і ReservationService)
2. ReservationService запитує зайняті на цей інтервал столи через getReservedTableIds(start_time, end_time) у RESERVATION і отримує список reserved_table_ids
3. ReservationService викликає getFreeTables(reserved_table_ids) у TABLE та повертає у ClientApp перелік доступних столів, де стіл №5 має статус FREE (завершення першої активації)
4. Актор обирає стіл №5 зі списку доступних і підтверджує бронювання викликом submitManualReservation(date, time, guests_count, table_ids=[5]) у ClientApp (початок основної активації ClientApp)
5. ClientApp передає запит createManualReservation(client_id, admin_id, start_time, end_time, guests_count, table_ids=[5]) до ReservationService (початок основної активації ReservationService)
6. ReservationService виконує перевірку ліміту validateTableLimit(table_ids=[5]) (кількість столів <= 3) та у транзакції SERIALIZABLE перевіряє відсутність перетину часу checkTableTimeSlot(table_id=5, start_time, end_time) у RESERVATION і поточний статус checkTableStatus(table_id=5) у TABLE
7. Фрагмент alt [Стіл №5 вільний на обраний часовий інтервал і має статус FREE]:
   - 7.1. ReservationService створює запис бронювання saveReservation(client_id, admin_id, start_time, end_time, table_ids=[5], status="CONFIRMED") у RESERVATION та отримує reserv_id
   - 7.2. Фрагмент loop [Для кожного обраного столу з table_ids (від 1 до 3 столів)]: ReservationService оновлює статус столу №5 через updateTableStatus(table_id=5, status="RESERVED") у TABLE та отримує statusUpdated
   - 7.3. ReservationService викликає sendConfirmation(client_id, reserv_details) у NotificationService, який доставляє клієнту підтвердження з деталями візиту і повертає confirmationSent
   - 7.4. ReservationService повертає успішну відповідь reservationConfirmed(reserv_details) у ClientApp
8. Гілка else [Стіл №5 був паралельно зайнятий або заброньований іншим користувачем]:
   - 8.1. ReservationService скасовує транзакцію та повертає помилку tableConflictError(table_id=5) у ClientApp
9. Поза фрагментом alt: ReservationService завершує свою активацію, а ClientApp відображає користувачу результат операції (підтвердження бронювання столу №5 або повідомлення про конфлікт) і завершує активацію

### 3.3. Сценарій 3: Відсутність вільних столів у залі

Опис: Користувач з роллю CLIENT або ADMIN обирає дату, час і кількість гостей, але на обраний інтервал часу всі відповідні столи мають статус "Зайнятий" або "Заброньовано", тому система фіксує відсутність вільних місць і пропонує альтернативні доступні столи або інший час візиту

Комбіновані фрагменти та Activation Bars:

- Активація ClientApp (крок 1) та ReservationService (крок 2) розпочинається до фрагмента opt / alt і триває безперервно до фінального повернення результату на кроках 8-9
- Використовується фрагмент opt (при отриманні порожнього списку вільних столів) із вкладеним фрагментом alt для обробки наявності або відсутності альтернативних варіантів

Кроки взаємодії:

1. Актор (Client або Admin) обирає дату, час та кількість гостей і надсилає запит checkAndBook(date, time, guests_count) через ClientApp (початок активації ClientApp)
2. ClientApp викликає checkAvailabilityForBooking(client_id, admin_id, start_time, end_time, guests_count) у ReservationService (початок активації ReservationService)
3. ReservationService перевіряє активні бронювання на цей інтервал через getReservedTableIds(start_time, end_time) у RESERVATION та отримує список заброньованих столів
4. ReservationService звертається до TABLE із запитом findAvailableTables(guests_count, reserved_table_ids)
5. TABLE повертає порожній список emptyList, оскільки всі відповідні столи на обраний інтервал мають статус OCCUPIED або RESERVED
6. Фрагмент opt [Вільні столи на обраний інтервал відсутні (availableTables == emptyList)]:
   - 6.1. ReservationService запитує найближчі вільні часові вікна findAlternativeTimeSlots(date, guests_count) у RESERVATION та альтернативні комбінації столів findAlternativeTables(guests_count) у TABLE
   - 6.2. RESERVATION і TABLE повертають знайдені варіанти alternativeOptions (альтернативні столи та інший час візиту)
   - 6.3. Фрагмент alt [Знайдено альтернативні доступні столи або інший час візиту]:
     - ReservationService повертає статус noTablesAvailable разом зі списком alternativeOptions до ClientApp
   - 6.4. Гілка else [На обрану дату вільних місць і слотів немає]:
     - ReservationService повертає статус noSlotsOnDate до ClientApp із пропозицією обрати іншу дату
7. Поза фрагментом opt: ReservationService завершує активацію
8. ClientApp сповіщає користувача про відсутність вільних місць на обраний час і відображає запропоновані альтернативні доступні столи або інший час візиту (завершення активації ClientApp)

### 3.4. Сценарій 4: Відхилення бронювання при перевищенні ліміту в 3 столи

Опис: Авторизований користувач з роллю CLIENT або ADMIN намагається обрати більше 3 столів у межах одного бронювання, і система блокує створення бронювання до запису в базу даних та відображає повідомлення про помилку

Комбіновані фрагменти та Activation Bars:

- Активація ClientApp (крок 1) та ReservationService (крок 2) починається перед перевіркою ліміту до входу у фрагмент alt і закінчується після завершення фрагмента alt
- Використовується фрагмент alt за умовою перевірки кількості столів count(table_ids) > 3

Кроки взаємодії:

1. Актор (Client або Admin) обирає більше 3 столів (наприклад, 4 столи) і викликає submitManualReservation(date, time, guests_count, table_ids=[1, 2, 3, 4]) у ClientApp (початок активації ClientApp)
2. ClientApp передає запит createManualReservation(client_id, admin_id, start_time, end_time, guests_count, table_ids=[1, 2, 3, 4]) до ReservationService (початок активації ReservationService)
3. ReservationService викликає внутрішній метод валідації validateTableLimit(table_ids)
4. Фрагмент alt [Кількість обраних столів перевищує ліміт (count(table_ids) > 3)]:
   - 4.1. ReservationService блокує створення бронювання без звернення до таблиць RESERVATION і TABLE
   - 4.2. ReservationService повертає помилку LimitExceededError("Максимальна кількість столів для одного бронювання: 3") до ClientApp
5. Гілка else [Кількість обраних столів у межах ліміту (1 <= count(table_ids) <= 3)]:
   - 5.1. ReservationService продовжує стандартну перевірку часового інтервалу в RESERVATION і доступності столів у TABLE
6. Поза фрагментом alt: ReservationService завершує активацію, а ClientApp відображає користувачу повідомлення про помилку "Максимальна кількість столів для одного бронювання: 3" і залишає форму відкритою для зміни вибору (завершення активації ClientApp)

### 3.5. Сценарій 5: Скасування або перенесення бронювання клієнтом або адміністратором

Опис: Клієнт має підтверджене бронювання зі закріпленим столом №5 у статусі "Заброньовано", і коли клієнт або адміністратор скасовує або переносить бронювання до початку візиту, система знімає резервування зі столу №5, оновлює його статус на "Вільний" та надсилає клієнту повідомлення про скасування або зміну бронювання

Комбіновані фрагменти та Activation Bars:

- Активація ClientApp (крок 1) та ReservationService (крок 2) починається до фрагмента alt і завершується після виходу з фрагмента alt на кроках 6-7
- Використовується фрагмент alt для розмежування скасування броні (статус CANCELLED) та перенесення броні (статус RESCHEDULED), а також вкладений фрагмент loop для звільнення всіх закріплених за бронею столів

Кроки взаємодії:

1. Актор (Client або Admin) відкриває активне бронювання зі столом №5 у ClientApp і викликає cancelOrRescheduleReservation(reserv_id, action, new_start_time, new_end_time) (початок активації ClientApp)
2. ClientApp передає запит processBookingChange(client_id, admin_id, reserv_id, action, new_start_time, new_end_time) до ReservationService (початок активації ReservationService)
3. ReservationService звертається до RESERVATION із запитом getReservationById(reserv_id) і отримує дані підтвердженої броні (client_id, table_ids=[5], status="CONFIRMED")
4. Фрагмент alt [Користувач скасовує бронювання до початку візиту (action == "CANCEL")]:
   - 4.1. ReservationService оновлює статус броні через updateReservationStatus(reserv_id, status="CANCELLED") у RESERVATION
   - 4.2. Фрагмент loop [Для кожного закріпленого столу з table_ids]: ReservationService знімає резервування зі столу №5 через updateTableStatus(table_id=5, status="FREE") у TABLE та отримує statusUpdated
   - 4.3. ReservationService викликає sendCancellationNotice(client_id, reserv_id) у NotificationService, який надсилає клієнту повідомлення про скасування бронювання
   - 4.4. ReservationService повертає результат reservationCancelled(reserv_id, table_id=5) до ClientApp
5. Гілка else [Користувач переносить бронювання на інший час (action == "RESCHEDULE")]:
   - 5.1. ReservationService оновлює часовий інтервал і переводить бронювання у статус RESCHEDULED через updateReservationSchedule(reserv_id, new_start_time, new_end_time, status="RESCHEDULED") у RESERVATION
   - 5.2. Фрагмент loop [Для кожного закріпленого столу з попереднього слоту table_ids]: ReservationService звільняє стіл №5 через updateTableStatus(table_id=5, status="FREE") у TABLE та отримує statusUpdated
   - 5.3. ReservationService викликає sendRescheduleNotice(client_id, reserv_details) у NotificationService, який надсилає клієнту повідомлення про зміну бронювання
   - 5.4. ReservationService повертає результат reservationRescheduled(reserv_details) до ClientApp
6. Поза фрагментом alt: ReservationService завершує активацію
7. ClientApp відображає оновлений статус бронювання та столу №5 ("Вільний") користувачу (завершення активації ClientApp)

### 3.6. Сценарій 6: Посадка гостей за живою чергою

Опис: Адміністратор з роллю ADMIN відкриває інтерактивну карту залу на ноутбуці або комп'ютері, бачить актуальні статуси всіх столів (де стіл №8 має статус "Вільний"), і коли прибуває компанія гостей у порядку живої черги без попередньої броні, закріплює за ними стіл №8, який переходить у статус "Зайнятий" із синхронізацією карти залу для всього персоналу

Комбіновані фрагменти та Activation Bars:

- На етапі посадки гостей активація ClientApp (крок 4) та ReservationService (крок 5) починається до фрагмента alt і завершується після виходу з фрагмента alt на кроках 8-9
- Використовується фрагмент alt для перевірки, що стіл №8 вільний у поточний момент і не має активного бронювання на цей часовий інтервал

Кроки взаємодії:

1. Admin (авторизований з роллю ADMIN на ноутбуці або комп'ютері) відкриває інтерактивну карту залу в ClientApp і викликає getHallMapStatus() (перша активація ClientApp і ReservationService)
2. ReservationService запитує поточні статуси всіх столів getAllTablesStatus() у TABLE та активні бронювання на поточний час getCurrentReservations(current_time) у RESERVATION
3. ReservationService повертає актуальну схему залу hallMapData (де стіл №8 має статус FREE) у ClientApp (завершення першої активації)
4. Коли прибуває компанія гостей за живою чергою, Admin обирає вільний стіл №8 на карті залу і викликає seatWalkInGuests(admin_id, table_id=8) у ClientApp (початок основної активації ClientApp)
5. ClientApp передає запит occupyTableForWalkIn(admin_id, table_id=8, current_time) до ReservationService (початок основної активації ReservationService)
6. ReservationService у транзакції перевіряє відсутність перетину з найближчими бронюваннями checkImmediateOverlap(table_id=8, current_time) у RESERVATION та актуальний статус checkTableStatus(table_id=8) у TABLE
7. Фрагмент alt [Стіл №8 має статус FREE і не заброньований на поточний час]:
   - 7.1. ReservationService закріплює стіл №8 та встановлює для нього статус OCCUPIED через updateTableStatus(table_id=8, status="OCCUPIED") у TABLE
   - 7.2. TABLE підтверджує зміну статусу statusUpdated(table_id=8, status="OCCUPIED")
   - 7.3. ReservationService повертає подію оновлення карти залу tableOccupied(table_id=8, status="OCCUPIED") у ClientApp
8. Гілка else [Стіл №8 вже зайнятий або заброньований]:
   - 8.1. ReservationService повертає повідомлення про недоступність столу tableNotAvailableError(table_id=8) у ClientApp
9. Поза фрагментом alt: ReservationService завершує активацію, а ClientApp оновлює статус столу №8 ("Зайнятий") на інтерактивній карті залу для всього персоналу (завершення активації ClientApp)

### 3.7. Сценарій 7: Завчасне нагадування про бронювання

Опис: У системі є активне підтверджене бронювання клієнта зі статусом столу "Заброньовано", і коли настає налаштований інтервал часу до початку візиту, система автоматично надсилає клієнту нагадування з деталями броні через сервіс сповіщень Notification Service

Комбіновані фрагменти та Activation Bars:

- Активація ReservationService починається за тригером системного таймера на кроці 1 до входу у фрагменти opt та loop і завершується на кроці 5 після обробки всіх знайдених бронювань
- Використовується фрагмент opt (якщо знайдено активні бронювання на найближчий час) та вкладений фрагмент loop для розсилки нагадувань по кожному бронюванню

Кроки взаємодії:

1. Коли настає налаштований інтервал часу до візиту, за тригером таймера ReservationService активується і викликає checkUpcomingReservations(current_time, reminder_interval) до RESERVATION (початок активації ReservationService)
2. RESERVATION перевіряє записи зі статусами CONFIRMED або RESCHEDULED і повертає список актуальних бронювань upcomingReservations (разом із client_id та table_ids)
3. Фрагмент opt [Список upcomingReservations не порожній]:
   - 3.1. Фрагмент loop [Для кожного активного бронювання з upcomingReservations]:
     - ReservationService запитує деталі закріплених столів зі статусом RESERVED через getReservedTablesInfo(table_ids) у TABLE та отримує номери столів
     - ReservationService викликає sendReminder(client_id, reserv_details) у NotificationService
     - NotificationService доставляє клієнту (Client) у ClientApp нагадування з деталями броні і повертає статус доставки reminderSent до ReservationService
4. Поза фрагментом opt та loop: ReservationService фіксує завершення циклу перевірки нагадувань і завершує свою активацію

## 4. Посилання на використані матеріали

- Матеріали Lab1:
  - [Spec](../Lab1/spec.md)
  - [ER-модель](../Lab1/models/base_model.mmd)
- Матеріали Lab2:
  - [Spec](../Lab2/spec.md)
  - [Системні вимоги та матриця трасування](../Lab2/requirements.md)
  - [Use Case діаграма підсистеми бронювання (UC1_1)](../Lab2/models/boarding_and_reservation.mmd)
  - [Gherkin-сценарії до UC1_1](../Lab2/features/boarding_and_reservation.feature)
