# Lab2: Вимоги та Use Cases (EARS + user stories + Gherkin)

**Мета:** структуровані вимоги, сценарії використання і трасування.

**Артефакт (текст + рендер):** EARS-вимоги (WHEN <тригер>, система SHALL <реакція> / WHILE <стан>, …) + user stories + Gherkin-сценарії (Given/When/Then); матриця трасування «вимога-id ↔ сценарій/UC» як таблиця в Markdown. Рендер — Use Case Diagram (PlantUML / Mermaid) з ≥2 типами акторів, include/extend, узагальненням.


### Актори 

- **USER** - Зареестрований користувач 
- **Guest** - Незареєстрований відвідувач 
- **Payment: Apple Pay / Google Pay** - зовнішня система оплати 
- **Notification Service: Push / SMS**: зовнішня система повідомлень

### Посилання

- [Models](./models/)
  - [Main_model](./models/main_model.md)
- [Features](./features/)
- [Specification](./spec.md)
- [Requirements](./requirements.md)