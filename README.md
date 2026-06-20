# DroneRest

**Drone Dispatch Controller — RESTful API Example**

A sample drone dispatch service that shows the core technologies, patterns, and code
organization behind a typical RESTful API. The domain models a fleet of delivery drones
that carry medications, so the examples stay concrete rather than abstract.

---
1. [Technology Stack](#technology-stack)  
2. [Domain Example](#domain-example)  
   2.1. [Requirements](#requirements)  
   2.2. [Functional requirements](#functional-requirements)  
   2.3. [Non-functional requirements](#non-functional-requirements)  
3. [Development](#development)
---

## Technology Stack

- Core
  - Java (or Kotlin)
  - Spring Boot
- REST API
  - Spring Web MVC
  - Open API
  - Swagger Codegen
- Data Management
  - Spring Data JPA / Hibernate
  - Liquibase
- Logging
  - Slf4j + Logback
- Jobs Scheduling
  - Quartz Scheduler
- DevOps
  - Gradle + Kotlin extension
  - Docker
- Testing
  - JUnit 5
  - Testcontainers
- Other tools
  - Mapstruct
  - Lombok

## Domain Example

Drones are well suited to delivering small, time-sensitive items to places that are hard
to reach by road. This example is built around that idea: a central **dispatch controller**
that registers drones, loads them with medications, and tracks each drone's state and
battery level throughout a delivery.

### Requirements

The system manages a fleet of **10 drones**. Each drone carries a small payload — here,
**medications**.

A **Drone** has:
- serial number (100 characters max);
- model (Lightweight, Middleweight, Cruiserweight, Heavyweight);
- weight limit (500gr max);
- battery capacity (percentage);
- state (IDLE, LOADING, LOADED, DELIVERING, DELIVERED, RETURNING).

A **Medication** has:
- name (letters, numbers, `-` and `_` only);
- weight;
- code (upper-case letters, numbers and `_` only);
- image (picture of the medication case).

Clients interact with the fleet through a REST API — the **dispatch controller**. The
low-level communication between the controller and each drone is outside the scope of this
example.

The API supports:
- registering a drone;
- loading a drone with medication items;
- listing the medications loaded onto a given drone;
- listing the drones available for loading;
- checking the battery level of a given drone.

> Assumptions about the design approach are documented under [Development](#development).

#### Functional requirements

- No user interface is required;
- a drone cannot be loaded beyond its weight limit;
- a drone cannot enter the LOADING state while its battery is **below 25%**;
- a periodic task checks drone battery levels and writes a history/audit event log.

#### Non-functional requirements

- input and output are JSON;
- the project is buildable and runnable;
- build, run, and test instructions are documented, and the database runs locally via containers;
- reference and sample data are preloaded into the database;
- the core logic is covered by JUnit tests.

## Development

### Assumptions

- *Drone.weightLimit* depends on *Drone.model*;
- *Drone.batteryCapacity* is a percentage of the battery's ideal value and depends on *Drone.model*;
- **Drone.serialNumber** is unique across all drones;
- **Medication.code** is unique across all medications;
- available drones are those in the **IDLE** state;
- a newly registered drone starts in the **IDLE** state.

Drone model characteristics:

| Drone Model   | Weight limit (gr) | Battery capacity (%) |
|---------------|-------------------|----------------------|
| Lightweight   | from 1 to 100     | from 20 to 40        |
| Middleweight  | from 101 to 200   | from 41 to 60        |
| Cruiserweight | from 201 to 350   | from 61 to 80        |
| Heavyweight   | from 351 to 500   | from 81 to 100       |


### Launching

Environment requirements:

- JDK _20_ or above
  ```shell
  java version "20.0.1" 2023-04-18
  Java(TM) SE Runtime Environment (build 20.0.1+9-29)
  Java HotSpot(TM) 64-Bit Server VM (build 20.0.1+9-29, mixed mode, sharing)
  ```
- Docker Desktop `v4.26.1`

The application runs on Testcontainers, so there's no database to set up by hand — this
keeps preparation for code reviewers to a minimum. The service listens on port **8084**.

To build the project from the console:

```shell
./gradlew build
```

To run the application:

```shell
./gradlew bootRun
```

To run the tests:

```shell
./gradlew test
```

### Additional information

- the project uses the _gradle wrapper_, so a local Gradle installation isn't needed;
- the HTTP API and DTO entities are generated from an OpenAPI specification (OAS);
- `postman.json` is a Postman collection with ready-made HTTP requests;
- `src/main/resources/db/changelog/db.changelog-dev.xml` contains the preloaded data;
- application logs are written to the `logs/` folder in the project root.
