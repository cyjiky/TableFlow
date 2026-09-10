```mermaid
---
config:
    layout: elk
---
flowchart LR
    Client["Client"]
    Waiter["Waiter"]
    Admin["Admin"]
    Guest["Guest"]
    Payment["Payment: Apple Pay / Google Pay"]
    Client["Клієнт (CLIENT)"]
    Waiter["Офіціант (WAITER)"]
    Guest["Гість (GUEST)"]
    Payment["Платіжний шлюз (Apple Pay / Google Pay)"]

    subgraph System ["Підсистема оплати"]
      UC1(["Оплата онлайн"])
      UC2(["Приймати оплату від гостей"])
      UC3(["Унікальний цифровий код для оплати"])
      UC4(["Оплата подарунковим сертифікатом"])
      UC5(["Зберігати електронні чеки"])
      UC_Pay(["Оплатити замовлення"])
      UC1(["Онлайн-оплата в застосунку"])
      UC2(["Розрахунок у залі (готівка / POS-термінал)"])
      UC3(["Оплата Takeaway готівкою за кодом"])
      UC4(["Застосувати подарунковий сертифікат"])
      UC5(["Зберегти електронний фіскальний чек"])
      UC6(["Верифікувати цифровий код видачі"])
    end

    Client --> UC1
    Client --> UC4
    Client --> UC5
    Client --> UC2

    Waiter --> UC2

    Admin --> UC2

    Guest --> UC1
    Guest --> UC4
    Guest --> UC3

    UC1 -.->|"<<extend>> якщо платник — Guest"| UC3
    UC1 --|> UC_Pay
    UC2 --|> UC_Pay
    UC3 --|> UC_Pay

    UC1 --> Payment

    UC1 -.->|"<<include>>"| UC5
    UC3 -.->|"<<include>>"| UC6

    UC4 -.->|"<<extend>>"| UC_Pay
```