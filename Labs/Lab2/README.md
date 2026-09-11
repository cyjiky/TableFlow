# Lab2: Вимоги та Use Cases (EARS + user stories + Gherkin)

**Мета:** структуровані вимоги, сценарії використання і трасування.

**Артефакт (текст + рендер):** EARS-вимоги (WHEN <тригер>, система SHALL <реакція> / WHILE <стан>, …) + user stories + Gherkin-сценарії (Given/When/Then); матриця трасування «вимога-id ↔ сценарій/UC» як таблиця в Markdown. Рендер — Use Case Diagram (PlantUML / Mermaid) з ≥2 типами акторів, include/extend, узагальненням.


### Актори 

- **USER** - Зареестрований користувач 
- **Guest** - Незареєстрований відвідувач 
- **Payment: Apple Pay / Google Pay** - зовнішня система оплати 
- **Notification Service: Push / SMS**: зовнішня система сповіщень та повідомлень


## 📂 Структура

```text                  
📁 Lab2/
├── 📁 AI/
│   └── 📝 comments.md                   # Аналітичний звіт щодо узгодженості вимог та моделей
├── 📁 features/                         # Gherkin-сценарії
├── 📁 models/                           # Use Case Diagrams
├── 📝 requirements.md                   # EARS-вимоги, User Stories та матриця трасування
├── 📝 spec.md                           # Специфікація цілей та акторів
└── 📍 README.md                         # Опис лабораторної роботи
```


### Посилання

- [Specification](./spec.md)
- [Requirements & Traceability Matrix](./requirements.md)
- [AI Audit & Comments](./AI/comments.md)
- [Models](./models/)
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