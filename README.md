# BambangShop Publisher App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases and methods to access the databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a basic functionality that makes BambangShop work: ability to create, read, and delete `Product`s.
This repository already contains a functioning `Product` model, repository, service, and controllers that you can try right away.

As this is an Observer Design Pattern tutorial repository, you need to implement another feature: `Notification`.
This feature will notify creation, promotion, and deletion of a product, to external subscribers that are interested of a certain product type.
The subscribers are another Rocket instances, so the notification will be sent using HTTP POST request to each subscriber's `receive notification` address.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Publisher" folder.
This Postman collection also contains endpoints that you need to implement later on (the `Notification` feature).

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    APP_INSTANCE_ROOT_URL="http://localhost:8000"
    ```
    Here are the details of each environment variable:
    | variable              | type   | description                                                |
    |-----------------------|--------|------------------------------------------------------------|
    | APP_INSTANCE_ROOT_URL | string | URL address where this publisher instance can be accessed. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)

## Mandatory Checklists (Publisher)
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [ ] Commit: `Create Subscriber model struct.`
    -   [ ] Commit: `Create Notification model struct.`
    -   [ ] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [ ] Commit: `Implement add function in Subscriber repository.`
    -   [ ] Commit: `Implement list_all function in Subscriber repository.`
    -   [ ] Commit: `Implement delete function in Subscriber repository.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [ ] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [ ] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [ ] Commit: `Implement publish function in Program service and Program controller.`
    -   [ ] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1

**In the Observer pattern diagram explained by the Head First Design Pattern book, Subscriber is defined as an interface. Explain based on your understanding of Observer design patterns, do we still need an interface (or trait in Rust) in this BambangShop case, or a single Model struct is enough?**

In the original Head First Design Patterns diagram, Subscriber is an interface so that various types of observers can be used interchangeably by the Publisher. In our BambangShop case, if all subscribers share the same behavior and data, you can use a single concrete model struct. However, if you foresee different kinds of subscribers or need polymorphic behavior (for example, different ways to process notifications), then defining a trait (Rust’s equivalent of an interface) would allow multiple implementations. For our simple use case, a single model struct is enough, but a trait would be beneficial for extensibility.

**id in Program and url in Subscriber is intended to be unique. Explain based on your understanding, is using Vec (list) sufficient or using DashMap (map/dictionary) like we currently use is necessary for this case?**

Since the design expects that the id (in Program) and url (in Subscriber) be unique, lookups or ensuring uniqueness can be more efficiently enforced with a map. A Vec would require linear searches to check for duplicates or to fetch an element by key, making it less efficient. The DashMap (or any map/dictionary) allows O(1) access and inherently associates a unique key (the id or url) with each subscriber. Hence, using a map/dictionary structure is more appropriate for this case.

**When programming using Rust, we are enforced by rigorous compiler constraints to make a thread-safe program. In the case of the List of Subscribers (SUBSCRIBERS) static variable, we used the DashMap external library for thread safe HashMap. Explain based on your understanding of design patterns, do we still need DashMap or we can implement Singleton pattern instead?**

Rust’s safety guarantees require proper synchronization for mutable global state. While a Singleton pattern can provide a single instance, you still need to ensure thread safety. DashMap is an external library that offers a concurrent map with built-in locking, which removes much of the manual work in implementing thread safety. Even if you implement a Singleton, you’d likely wrap its internals in synchronization primitives (like Mutex or RwLock). Therefore, using DashMap simplifies maintaining thread safety and is recommended compared to a plain Singleton without concurrency control.

#### Reflection Publisher-2

**In the Model-View Controller (MVC) compound pattern, there is no “Service” and “Repository”. Model in MVC covers both data storage and business logic. Explain based on your understanding of design principles, why we need to separate “Service” and “Repository” from a Model?**

The Model in MVC represents the domain data and business rules, but it shouldn’t also handle data persistence or orchestration of complex business processes. By separating the Repository and Service layers, we adhere to the Single Responsibility Principle. The Repository is responsible solely for data access concerns (CRUD operations, persistence), while the Service layer encapsulates business logic (orchestrating workflows, validation, calling repositories, etc.). This separation makes the codebase more modular, easier to test, and maintain because each layer has a distinct responsibility.

**What happens if we only use the Model? Explain your imagination on how the interactions between each model (Program, Subscriber, Notification) affect the code complexity for each model?**

Using only the Model would result in a monolithic structure that handles data representation, business logic, and persistence—all squeezed into a single component. For our BambangShop case, this means that interactions among Program, Subscriber, and Notification would become tangled. For example, if a product update needs to notify subscribers, the Model would have to handle both the domain logic (the product’s state changes) and the communication logic (triggering notifications), increasing complexity. Such coupling makes it harder to maintain, extend, and test individual concerns without breaking others, potentially leading to a less flexible and more error-prone system.

**Have you explored more about Postman? Tell us how this tool helps you to test your current work. You might want to also list which features in Postman you are interested in or feel like it is helpful to help your Group Project or any of your future software engineering projects.**

I’ve found Postman to be extremely useful for testing REST APIs because it allows quick creation and execution of HTTP requests. With features such as environment variables, collections, and automated test scripts, I can easily simulate scenarios (like product creation or subscriber notifications) and verify responses without writing additional client code. 

#### Reflection Publisher-3

**Observer Pattern has two variations: Push model (publisher pushes data to subscribers) and Pull model (subscribers pull data from publisher). In this tutorial case, which variation of Observer Pattern that we use?**

In this tutorial, we use the Push model. The Publisher actively sends (pushes) notifications with all relevant data (e.g., event type and product details) to the Subscriber via an HTTP POST request.

**What are the advantages and disadvantages of using the other variation of Observer Pattern for this tutorial case? (example: if you answer Q1 with Push, then imagine if we used Pull)**

Advantages:
- The Publisher is less coupled to the Subscriber.
- Subscribers can pull data on their own schedule, possibly reducing network load if subscribers only request updates when needed.
- It may be easier to handle large or continuously updating data sets because subscribers decide when to retrieve data.

Disadvantages:
- It introduces latency; subscribers may not be aware of an event until they poll for updates.
- It requires additional mechanism for subscribers to know when to pull (e.g., using timestamps or versioning).
- Increased network overhead may occur if subscribers poll too frequently.

**Explain what will happen to the program if we decide to not use multi-threading in the notification process.**

Without multi-threading, the notification process would be synchronous. That means each HTTP POST request to a Subscriber would block the main thread until it completes. As a result:
- The response time for product updates may increase.
- If a Subscriber is slow or unresponsive, it could delay notifications to others and reduce overall system throughput.
- Overall, the program would be less scalable and responsive under high load.