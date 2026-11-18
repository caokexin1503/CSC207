# My answers to the written exercises

The coded exercises (ex01–ex18) live in `exercises/`. This file holds the
exercises that have no automated test.

- [Chapter 3, Question 1: from UML to Java](#chapter-3-question-1-from-uml-to-java)
- [Chapter 3, Question 2: from Java to UML](#chapter-3-question-2-from-java-to-uml)
- [Chapter 4.3.1: Cancel clears the text fields](#chapter-431-cancel-clears-the-text-fields)
- [Chapter 8, Question 1: noun–verb analysis and class diagram](#chapter-8-question-1-nounverb-analysis-and-class-diagram)
- [Chapter 8, Question 2: scenario walk-through](#chapter-8-question-2-scenario-walk-through)
- [Chapter 8, Question 3: from a user story to a use case](#chapter-8-question-3-from-a-user-story-to-a-use-case)
- [Chapter 9.3: SOLID in Java's language features](#chapter-93-solid-in-javas-language-features)
- [Chapter 9.4: which principle is violated?](#chapter-94-which-principle-is-violated)
- [Chapter 12: deriving the patterns from the principles](#chapter-12-deriving-the-patterns-from-the-principles)

---

## Chapter 3, Question 1: from UML to Java

Reading the arrows: a solid line with a hollow triangle means `extends`, and a
dashed line with a hollow triangle means `implements`. `Account` is in italics,
so it is abstract, and so is its `withdraw` method.

```java
abstract class Account {
    protected double balance;          // "#" = protected

    public Account(double balance) {
        this.balance = balance;
    }

    public void deposit(double amount) {
        // ...
    }

    public abstract boolean withdraw(double amount);
}

interface InterestBearing {
    void addInterest();
}

class ChequingAccount extends Account {
    private double overdraftLimit;     // "-" = private

    public ChequingAccount(double balance) {
        super(balance);                // Account has no no-arg constructor
    }

    @Override
    public boolean withdraw(double amount) {
        // ...
    }
}

class SavingsAccount extends Account implements InterestBearing {
    private double rate;

    public SavingsAccount(double balance) {
        super(balance);
    }

    @Override
    public boolean withdraw(double amount) {
        // ...
    }

    @Override
    public void addInterest() {
        // ...
    }
}
```

The diagram does not show the subclass constructors. They are still required:
`Account` only has an `Account(double)` constructor, so every subclass must call
`super(balance)`.

## Chapter 3, Question 2: from Java to UML

The diagram is in
[`plantuml/exercises/vehicles-uml-starter.puml`](../plantuml/exercises/vehicles-uml-starter.puml).

- `Car` extends `Vehicle` and `ElectricCar` extends `Car`: two solid-line,
  hollow-triangle (generalization) arrows.
- `ElectricCar` implements `Chargeable`: a dashed-line, hollow-triangle
  (realization) arrow.
- `Vehicle` and its `maxSpeed()` are abstract, so they are drawn in italics.
- `make` is `#` (protected), `batteryLevel` is `-` (private), and the methods
  are `+` (public).

## Chapter 4.3.1: Cancel clears the text fields

This is done in
[`code/src/main/java/gui/ButtonClickExample.java`](../code/src/main/java/gui/ButtonClickExample.java).
First I pulled the Cancel button out into a `cancel` variable. Then I gave it
an `ActionListener` that calls `setText("")` on both text fields.

## Chapter 8, Question 1: noun–verb analysis and class diagram

The diagram is in
[`plantuml/exercises/library-uml-starter.puml`](../plantuml/exercises/library-uml-starter.puml).

### 1. Nouns

| Noun | Decision | Reason |
|---|---|---|
| library | **class** `Library` | the system as a whole; it holds the catalogue and the members, and handles login |
| member | **class** `Member` | an actor with its own state (loans, late fee) |
| item | **class** `Item` (abstract) | the common parent of everything in the catalogue |
| title | attribute of `Item` | just a value |
| catalogue number | attribute of `Item` | just a value (the unique key) |
| book | **class** `Book` | a kind of borrowable item |
| DVD | **class** `DVD` | a kind of borrowable item |
| three weeks / one week | attribute value (`loanPeriodDays`) | a number, not a thing with behaviour |
| reference-only item | **class** `ReferenceItem` | an item that is *not* borrowable |
| member number | attribute (`id`) | a value used to log in |
| hold | **class** `Hold` | must be remembered over time and links a member to an item |
| loan ("on loan") | **class** `Loan` | must record who has the item and when it is due |
| librarian | **class** `Librarian` | an actor with different responsibilities from a member |
| catalogue | **class** `Catalogue` | the collection that items are added to |
| late fee | attribute of `Member` | a number |

### 2. Verbs: responsibilities

| Responsibility | Class |
|---|---|
| borrow an item | `Borrowable` / `BorrowableItem.checkOut(member)` |
| know the loan period | `BorrowableItem` (the `loanPeriodDays` field) |
| refuse to lend reference-only items | `ReferenceItem` does not implement `Borrowable` |
| log in with a member number | `Library.login(memberNumber)` |
| place a hold on an item on loan | `Member.placeHold(item)`, which delegates to `BorrowableItem.placeHold(member)` |
| add new items to the catalogue | `Librarian.addItem(catalogue, item)`, which calls `Catalogue.addItem(item)` |
| waive a member's late fee | `Librarian.waiveLateFee(member)`, which calls `Member.setLateFee(0)` |

### 3. Inheritance and interfaces

- **Reference-only versus borrowable items.** This is a difference in what an
  item can *do*. That makes it an interface: `Borrowable`. Only
  `BorrowableItem` implements it; `ReferenceItem` does not. If `borrow()` were
  on `Item`, `ReferenceItem` would have to throw an exception from it, which
  breaks the Liskov Substitution Principle. With the interface, trying to
  borrow a reference item fails at compile time.
- **Book versus DVD.** They differ only in a number (21 days or 7 days), so the
  loan period is a field of `BorrowableItem`, not overridden behaviour. I
  still kept two small subclasses whose constructors set that field. That way
  the catalogue can tell books and DVDs apart, and new differences can be
  added later. Another defensible design is a single `BorrowableItem` with a
  `type` or `loanPeriod` field and no subclasses at all.
- **Member and Librarian.** Both are people the library knows about, and both
  have an id and a name. They share an abstract `LibraryUser`. The
  specification does not say how librarians log in, so the shared `id` is an
  assumption I would check with the client.

## Chapter 8, Question 2: scenario walk-through

> A member places a hold on an item that is currently on loan.

| Step | Responsible class | Collaborators | Gaps found, then fixed |
|---|---|---|---|
| 1. The member logs in with their member number. | `Library` (`login`) | `Member` | `Library` needs a lookup from member number to member, so I added `members : Map<String, Member>`. |
| 2. The member looks the item up by catalogue number. | `Catalogue` (`findItem`) | `Item` | The library needs access to the catalogue, so I added `Library.getCatalogue()`. |
| 3. The system checks that the item can be held at all. | the type system | `Borrowable` | A `ReferenceItem` is not `Borrowable`, so a hold on it cannot even be requested. |
| 4. The system checks that the item is currently on loan. | `BorrowableItem` (`isOnLoan`) | `Loan` | Something must know about the current loan, so I added `currentLoan : Loan` to `BorrowableItem`. |
| 5. A `Hold` is created and remembered. | `BorrowableItem` (`placeHold`) | `Hold`, `Member` | Something must store the holds, so I added `holds : List<Hold>` to `BorrowableItem`. |
| 6. The hold is confirmed to the member. | `Member` (`placeHold` returns the `Hold`) | `Hold` | none |
| 7. Later the item is returned, and the next member in line must be told. | `BorrowableItem` (`checkIn`) | `Hold`, `Member` | `Hold` needs `getMember()`. With that, the design stabilizes. |

**Which class should know whether an item is currently on loan?**

1. **The item itself (`BorrowableItem.currentLoan`).** This is my choice.
   - Advantages: answering "is it on loan?" or "who is waiting for it?"
     starts from the object you already have, and returns and holds naturally
     go through the item.
   - Drawbacks: the item is coupled to `Loan`, and finding all of a member's
     loans means keeping a second link (`Member.loans`) in sync.
2. **The member (`Member.loans`).**
   - Advantages: "what does this member have out?" and late-fee calculations
     are easy.
   - Drawbacks: "is this item on loan?" means searching every member.
3. **A separate `LoanRegistry` or `Circulation` class** that owns every `Loan`
   and `Hold`.
   - Advantages: one source of truth and no data duplicated between objects.
     Items stay simple and the class has a single responsibility (SRP).
   - Drawbacks: one more class to pass around, and every question about an
     item has to go through it.

## Chapter 8, Question 3: from a user story to a use case

**Name:** Reserve a study room
**Primary actor:** Student

**Preconditions**
- The student is signed in.
- The student has selected a room, a date, and a time slot.

**Trigger:** The student submits the reservation request.

**Success outcome:** A reservation exists for that student, room, and time
slot. No other reservation exists for the same room at that time, and the
student sees a confirmation.

**Main success scenario**
1. The student submits a reservation for the selected room, date, and time.
2. The system checks that the room has no reservation that overlaps this time.
3. The system records the reservation for this student.
4. The system displays a confirmation showing the room, date, and time.

**Extensions**
- **2a. The room is no longer available.** Someone else reserved it after the
  student selected it.
  1. The system does not create a reservation; nothing changes.
  2. The system tells the student that the room is no longer available at that
     time and lets them choose another room or time.
  3. The use case ends without success, or resumes at step 1 if the student
     picks again.
- **1a. The selected time is in the past or outside the building's opening
  hours.**
  1. The system rejects the request without recording anything; nothing
     changes.
  2. The student sees a message explaining why the time is invalid and is
     asked to choose a valid time.

**Review checklist**
- *One user goal?* Yes: reserving one room for one time.
- *Preconditions are facts?* Yes: "is signed in" and "has selected", not setup
  steps.
- *Each step is an actor action or an observable system responsibility?* Yes.
- *Alternative paths are kept separate?* Yes: they are extensions 1a and 2a.
- *Testable?* Yes: the success outcome is a checkable state plus a visible
  confirmation.
- *No premature UI or database detail?* Yes: there are no widgets, tables, or
  method names.

## Chapter 9.3: SOLID in Java's language features

- **The child's constructor must call the parent's constructor (`super(...)`).**
  The parent's fields are always set up properly, so a subclass object really
  is a valid instance of the parent. That supports the **Liskov Substitution
  Principle**.
- **Generics.** Code such as `List<T>` or `Box<T>` is written once and reused
  for any type without being modified. That is **Open/Closed**. Generics also
  let you depend on an abstraction such as `Comparable<T>` instead of a
  concrete type, which is **Dependency Inversion**.
- **The `private` keyword.** It hides how a class works inside. Callers depend
  only on its public behaviour, so the internals can change without changing
  any callers. That is **Open/Closed**, and it keeps each class focused on its
  own job (**Single Responsibility**).
- **An interface as a reference type** (`IWorker w = new SuperWorker();`).
  High-level code depends on the abstraction rather than on concrete classes.
  That is **Dependency Inversion**, as in exercise 15. Because a class can
  implement several small interfaces, interfaces also support **Interface
  Segregation**.
- **`@Override` and abstract methods.** You add behaviour by writing a new
  subclass instead of editing existing code (**Open/Closed**). The compiler
  checks that overrides keep the parent's method signature, which helps with
  **Liskov Substitution**.

## Chapter 9.4: which principle is violated?

`Report` breaks the **Single Responsibility Principle**. It has three reasons
to change: what the report says, how files are written, and how email is sent.

Fix: split it into `Report` (which only produces the text), `ReportFileWriter`,
and `ReportEmailer`. Ideally both writers implement a `ReportOutput` interface,
so callers depend on the abstraction. That applies the DIP as well.

## Chapter 12: deriving the patterns from the principles

- **Strategy.** Pull the part that varies (the algorithm) out behind an
  interface. The context class depends only on that interface (DIP). New
  algorithms are added as new classes without editing the context (OCP), and
  each algorithm sits in its own class (SRP).
- **Observer.** The subject depends only on an `Observer` interface rather
  than on concrete listeners (DIP). New kinds of observers can be added
  without changing the subject (OCP).
- **Adapter.** Wraps an existing class so it matches the interface the client
  expects. The client keeps depending on its own abstraction (DIP), and
  neither the client nor the adapted class is modified (OCP).
- **Facade.** Gives one simple entry point to a group of classes. Callers
  depend on a small interface instead of many details (ISP), and each class
  behind the facade keeps its single job (SRP).
- **Factory and Abstract Factory.** Creating objects becomes a responsibility
  of its own (SRP). Client code works with the abstract product type instead
  of calling `new` on concrete classes (DIP), so new products only need new
  factories (OCP).
- **Builder.** Separates building a complex object from the object itself
  (SRP), and avoids a long list of overloaded constructors.
