# BambangShop Receiver App
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
4.  `repository`: this module contains structs that serve as databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a Rocket web framework skeleton that you can work with.

As this is an Observer Design Pattern tutorial repository, you need to implement a feature: `Notification`.
This feature will receive notifications of creation, promotion, and deletion of a product, when this receiver instance is subscribed to a certain product type.
The notification will be sent using HTTP POST request, so you need to make the receiver endpoint in this project.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Receiver" folder.

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    ROCKET_PORT=8001
    APP_INSTANCE_ROOT_URL=http://localhost:${ROCKET_PORT}
    APP_PUBLISHER_ROOT_URL=http://localhost:8000
    APP_INSTANCE_NAME=Safira Sudrajat
    ```
    Here are the details of each environment variable:
    | variable                | type   | description                                                     |
    |-------------------------|--------|-----------------------------------------------------------------|
    | ROCKET_PORT             | string | Port number that will be listened by this receiver instance.    |
    | APP_INSTANCE_ROOT_URL   | string | URL address where this receiver instance can be accessed.       |
    | APP_PUUBLISHER_ROOT_URL | string | URL address where the publisher instance can be accessed.       |
    | APP_INSTANCE_NAME       | string | Name of this receiver instance, will be shown on notifications. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)
3.  To simulate multiple instances of BambangShop Receiver (as the tutorial mandates you to do so),
    you can open new terminal, then edit `ROCKET_PORT` in `.env` file, then execute another `cargo run`.

    For example, if you want to run 3 (three) instances of BambangShop Receiver at port `8001`, `8002`, and `8003`, you can do these steps:
    -   Edit `ROCKET_PORT` in `.env` to `8001`, then execute `cargo run`.
    -   Open new terminal, edit `ROCKET_PORT` in `.env` to `8002`, then execute `cargo run`.
    -   Open another new terminal, edit `ROCKET_PORT` in `.env` to `8003`, then execute `cargo run`.

## Mandatory Checklists (Subscriber)
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop-receiver to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [ ] Commit: `Create Notification model struct.`
    -   [ ] Commit: `Create SubscriberRequest model struct.`
    -   [ ] Commit: `Create Notification database and Notification repository struct skeleton.`
    -   [ ] Commit: `Implement add function in Notification repository.`
    -   [ ] Commit: `Implement list_all_as_string function in Notification repository.`
    -   [ ] Write answers of your learning module's "Reflection Subscriber-1" questions in this README.
-   **STAGE 3: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Commit: `Implement receive_notification function in Notification service.`
    -   [ ] Commit: `Implement receive function in Notification controller.`
    -   [ ] Commit: `Implement list_messages function in Notification service.`
    -   [ ] Commit: `Implement list function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Subscriber-2" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Subscriber) Reflections



#### Reflection Subscriber-1

**1\. In this tutorial, we used RwLock<> to synchronise the use of Vec of Notifications. Explain why it is necessary for this case, and explain why we do not use Mutex<> instead?**

RwLock<> (Read-Write Lock) is necessary to ensure thread safety when multiple requests hit the server concurrently, preventing data races when accessing the global Vec. We use RwLock<> instead of Mutex<> because of our access pattern. Mutex<> locks the data completely, allowing only one thread to access it at a time (whether reading or writing). RwLock<>, on the other hand, allows **multiple readers** to access the data simultaneously as long as no one is writing. Since viewing notifications (list\_all\_as\_string) happens frequently and doesn't modify data, RwLock<> is much more performant. It only exclusively locks the thread when a new notification is being added (write()).

**2\. In this tutorial, we used lazy\_static external library to define Vec and DashMap as a “static” variable. Compared to Java where we can mutate the content of a static variable via a static function, why did not Rust allow us to do so?**

Rust prioritizes memory safety and guarantees the prevention of data races at compile time. Global mutable variables (like standard static mut in Rust or typical static variables in Java) are inherently unsafe in a multi-threaded environment because multiple threads could attempt to modify the variable at the exact same time, leading to corrupted data or crashes. To enforce safety, Rust requires global static variables to be immutable by default. To mutate them, we must use "interior mutability" via thread-safe synchronization primitives like RwLock or Mutex. lazy\_static is used because Rust traditionally does not allow complex, heap-allocated initializations (like Vec::new() or DashMap::new()) in standard static declarations. lazy\_static safely delays this initialization until the exact moment the variable is first accessed at runtime.

#### Reflection Subscriber-2

**1\. Have you explored things outside of the steps in the tutorial, for example: src/lib.rs? If not, explain why you did not do so. If yes, explain things that you have learned from those other parts of code.**

Yes, exploring src/lib.rs and src/main.rs revealed how Rocket initializes the server, loads configurations, and establishes global state. Specifically, I learned how the REQWEST\_CLIENT is defined as a static, globally available HTTP client using lazy\_static. This prevents the application from continuously tearing down and creating new TCP connections for every single HTTP request, which drastically improves performance when sending multiple notifications.

**2\. Since you have completed the tutorial by now and have tried to test your notification system by spawning multiple instances of Receiver, explain how Observer pattern eases you to plug in more subscribers. How about spawning more than one instance of Main app, will it still be easy enough to add to the system?**

The Observer pattern makes adding more subscribers incredibly easy because the Publisher (Main App) is completely decoupled from the logic of the Subscribers. The Publisher doesn't care if there are 0, 3, or 100 receivers, it simply iterates over a dynamic list of URLs and fires requests. Spawning more instances of the Main App, however, would be more complex. Our current DashMap storing the subscriber list is kept in local memory. If we spawn a second Main App, it won't share that memory. To support multiple Main Apps, we would need to migrate the subscriber list from local memory to an external, shared database like PostgreSQL or Redis.

**3\. Have you tried to make your own Tests, or enhance documentation on your Postman collection? If you have tried those features, tell us whether it is useful for your work (it can be your tutorial work or your Group Project).**

Yes, enhancing the Postman collection by writing custom test scripts (like automatically checking if the response status is 200 OK or extracting IDs from responses) is incredibly useful. For our BidMart group project, specifically regarding Wallet Management, I can use Postman tests to chain requests together, for example, automatically grabbing a user's ID from a login request and feeding it directly into a "Top Up Balance" request to simulate a real user flow without manual copy-pasting.