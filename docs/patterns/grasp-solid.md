# GRASP

- **Information Expert** (Информационные эксперт) — Класс который владеет всей необходимой информацией. Если объект
  владеет всей нужной информацией для какой-то операции или функционала, значит этот объект будет выполнять либо
  делегировать выполнение этой операции.
- **Creator (Создатель)** — создает другие объекты. Но он содержит создаваемые объекты\использует их\знает как
  создать\записывает создаваемые объекты.
- **Controller (Контроллер)** — Обработка действий клиента.
- **Low Coupling (Слабая связанность)** — код должен быть слабо связан и зависеть от абстракций.
- **High Cohesion (Высокая сцепленность)** — Один класс должен решать только какую-то одну задачу.
- **Pure Fabrication (Чистая выдумка или чистое синтезирование)** — Когда вводим DAO или REPO.
- **Indirection (Посредник)** — чтобы небыло сильного связывания, вводим объект-посредник.
- **Protected Variations (Сокрытие реализации или защищенные изменения)** — определить “точки изменений” и зафиксировать
  их в абстракции (интерфейсе). “Точки изменений” – не что иное как наши объекты, которые могут меняться.
- **Polymorphism (Полиморфизм)** — Способность обрабатывать данные разных типов.

---

# SOLID

- **Single Responsibility Principle — принцип единственной ответственности.**
    - Each class should have only one area of responsibility.
    - Каждый класс должен иметь только одну зону ответственности.

- **Open closed Principle — принцип открытости-закрытости.**
    - Classes should be open for expansion but closed for modification.
    - Классы должны быть открыты для расширения, но закрыты для изменения.

- **Liskov substitution Principle — принцип подстановки Барбары Лисков.**
    - Barbara Liskov substitution principle. It should be possible to substitute any of its successors instead of the
      parent type, and the program operation should not change.
    - Должна быть возможность вместо родительского типа подставить любой его наследник, при этом работа программы не
      должна измениться.

- **Interface Segregation Principle — принцип разделения интерфейсов.**
    - You should not force the client to implement an interface that has nothing to do with it.
    - Не нужно заставлять клиента реализовывать интерфейс, который не имеет к нему отношения.

- **Dependency Inversion Principle — принцип инверсии зависимостей.**
    - You should not force the client to implement an interface that has nothing to do with it.
      Dependency Inversion Principle. Top-level modules should not depend on lower-level modules. All should depend on
      an abstraction. Abstractions should not depend on parts. Parts should depend on abstractions.
    - Модули верхнего уровня не должны зависеть от модулей нижнего. Все должны зависеть от абстракции. Абстракции не
      должны зависеть от деталей. Детали должны зависеть от абстракций.