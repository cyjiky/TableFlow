# Lab2: Вимоги та Use Cases (EARS + user stories + Gherkin)

**Мета:** структуровані вимоги, сценарії використання і трасування.

**Артефакт (текст + рендер):** EARS-вимоги (WHEN <тригер>, система SHALL <реакція> / WHILE <стан>, …) + user stories + Gherkin-сценарії (Given/When/Then); матриця трасування «вимога-id ↔ сценарій/UC» як таблиця в Markdown. Рендер — Use Case Diagram (PlantUML / Mermaid) з ≥2 типами акторів, include/extend, узагальненням.


### Актори

- USER - Зареєстрований користувач (ролі: CLIENT, ADMIN, WAITER, CHEF)
- Guest - Незареєстрований відвідувач

### Зовнішні системи (деталі реалізації)

- Payment: Apple Pay / Google Pay - зовнішня система оплати
- Notification Service: Push / SMS - зовнішня система сповіщень та повідомлень


### Структура

```text                  
📁 Lab2
├── 📁 AI
│   ├── 📝 comments.md                   # Аналітичний звіт щодо узгодженості вимог та моделей
│   └── 📝 07_11_26.md                   # Журнал взаємодії з AI, промпти та обґрунтування рішень
├── 📁 features                          # Gherkin-сценарії
├── 📁 models                            # Use Case Diagrams
├── 📝 requirements.md                   # EARS-вимоги, User Stories, матриця трасування та NFR
├── 📝 spec.md                           # Специфікація цілей, акторів та NFR
├── 📝 DEFENSE.md                        # Захист та перевірка узгодженості артефактів
└── 📍 README.md                         # Опис лабораторної роботи
```


### Посилання

- [Specification](./spec.md)
- [Requirements & Traceability Matrix](./requirements.md)
- [AI Audit & Comments](./AI/comments.md)
- [AI Session Log (07.11.2026)](./AI/07_11_26.md)
- [Models](./models/models.md)
  - [Головна Use Case модель](./models/main_model.mmd)
  - [Управління посадкою та бронюванням](./models/boarding_and_reservation.mmd)
  - [Оформлення та редагування замовлень](./models/orders_operations.mmd)
  - [Підсистема оплати](./models/pay_system.mmd)
  - [Комунікація між залом та кухнею](./models/communication.mmd)
- [Gherkin-сценарії](./features/)
  - [Управління посадкою та бронюванням](./features/boarding_and_reservation.feature)
  - [Оформлення та редагування замовлень](./features/orders_operations.feature)
  - [Підсистема оплати](./features/pay_system.feature)
  - [Комунікація між залом та кухнею](./features/communication.feature)
