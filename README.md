# Project‑CRUD‑Demo‑MySQL‑Menu‑Driven‑Application  

A **menu‑driven Java console program** that demonstrates full **CRUD** (Create, Read, Update, Delete) operations on a MySQL‑backed *project* database.  
The current focus is the **Update** and **Delete** phases, showing how to modify records safely and how `ON DELETE CASCADE` removes dependent rows automatically.

---  

## Table of Contents
1. [Project Overview](#project-overview)  
2. [Features](#features)  
3. [Technology Stack](#technology-stack)  
4. [Database Schema](#database-schema)  
5. [Prerequisites](#prerequisites)  
6. [Setup & Installation](#setup--installation)  
7. [Running the Application](#running-the-application)  
8. [Menu Options](#menu-options)  
9. [Learning Objectives](#learning-objectives)  
10. [Testing](#testing)  
11. [Contributing](#contributing)  
12. [License](#license)  

---  

## Project Overview
The application lets a user interact with a **project management** database through a simple numbered menu:

* **Create** – insert new projects (and optional employee links).  
* **Read** – list all projects or view full details of a specific project.  
* **Update** – change any mutable project field using a prepared `UPDATE` statement.  
* **Delete** – remove a project; related rows in the many‑to‑many join table are deleted automatically via `ON DELETE CASCADE`.  

All database interactions use **JDBC prepared statements** and clean‑up resources with *try‑with‑resources*.  

---  

## Features
| Feature | Description |
|---------|-------------|
| **Create** | Insert rows into `projects`, `employees`, and `project_employee`. |
| **Read** | • List all projects (`id – name`). <br>• Show full record for a selected project. |
| **Update** | Parameterised `UPDATE` for any editable column; verifies success with `executeUpdate()` return value. |
| **Delete** | Single `DELETE` on `projects` that cascades to dependent rows (`project_employee`). |
| **Result Feedback** | `executeUpdate()` > 0 ⇒ operation succeeded; 0 ⇒ no rows affected. |
| **Console UI** | Easy‑to‑use numbered menu with input validation. |

---  

## Technology Stack
| Layer | Tech |
|-------|------|
| Language | Java 11+ |
| Build tool | Maven |
| Database | MySQL 5.7+ |
| JDBC driver | MySQL Connector/J 8.0+ |
| Testing | JUnit 5 (uses H2 in‑memory for fast unit tests) |

---  

## Database Schema
```sql
-- projects (parent)
CREATE TABLE projects (
    project_id   INT AUTO_INCREMENT PRIMARY KEY,
    name         VARCHAR(100) NOT NULL,
    description  TEXT,
    start_date   DATE,
    end_date     DATE
);

-- employees (parent)
CREATE TABLE employees (
    employee_id  INT AUTO_INCREMENT PRIMARY KEY,
    first_name   VARCHAR(50),
    last_name    VARCHAR(50),
    email        VARCHAR(100) UNIQUE
);

-- many‑to‑many relation
CREATE TABLE project_employee (
    project_id   INT,
    employee_id INT,
    PRIMARY KEY (project_id, employee_id),
    FOREIGN KEY (project_id)   REFERENCES projects(project_id)   ON DELETE CASCADE,
    FOREIGN KEY (employee_id)  REFERENCES employees(employee_id)  ON DELETE CASCADE
);
```
*`ON DELETE CASCADE` guarantees that removing a project (or an employee) automatically cleans up the join table, preventing orphaned rows.*

---  

## Prerequisites
- **JDK** 11 or newer  
- **Maven** 3.6+  
- **MySQL** server (local or remote)  
- Network access to the MySQL instance  

---  

## Setup & Installation  

1. **Clone the repository**  
   ```bash
   git clone https://github.com/Karla-Sanchez-Osuna/Project-CRUD-Demo-MySQL-Menu-Driven-Application.git
   cd Project-CRUD-Demo-MySQL-Menu-Driven-Application
   ```

2. **Create the database** (replace credentials as needed)  
   ```sql
   CREATE DATABASE project_crud_demo;
   USE project_crud_demo;
   SOURCE src/main/resources/schema.sql;   -- contains the tables above
   ```

3. **Edit connection properties** – `src/main/resources/db.properties`  
   ```properties
   jdbc.url=jdbc:mysql://localhost:3306/project_crud_demo
   jdbc.username=YOUR_USERNAME
   jdbc.password=YOUR_PASSWORD
   ```

4. **Build the project**  
   ```bash
   mvn clean package
   ```

5. **Run the application**  
   ```bash
   java -jar target/Project-CRUD-Demo-MySQL-Menu-Driven-Application-1.0.jar
   ```

---  

## Running the Application  

When the program starts you’ll see:

```
=== Project CRUD Demo ===
1) Insert a new project
2) List all projects
3) View project details
4) Update a project
5) Delete a project
0) Exit
Enter your choice:
```

Enter the option number and follow the prompts. Invalid input simply returns you to the menu.

---  

## Menu Options  

| Option | Action |
|--------|--------|
| **1** | Prompt for project information and insert it. |
| **2** | Show a brief list of all projects (`project_id – name`). |
| **3** | Ask for a `project_id` and display every column for that project. |
| **4** | Choose a project, then select which field(s) to modify; updates are executed with a prepared `UPDATE`. |
| **5** | Delete a project by ID; related rows in `project_employee` disappear automatically thanks to `ON DELETE CASCADE`. |
| **0** | Close the JDBC connection and terminate the program. |

---  

## Learning Objectives  

- **Update** a record safely using a parameterised `UPDATE` statement.  
- **Delete** a parent row and automatically cascade the removal to child rows.  
- **Observe** the effect of `ON DELETE CASCADE` in a many‑to‑many relationship.  
- **Interpret** the integer returned by `PreparedStatement.executeUpdate()` to confirm success or detect “no rows affected”.  

---  

## Testing  

Unit tests are located under `src/test/java` and run against an H2 in‑memory database that mirrors the MySQL schema.

```bash
mvn test
```

For manual verification, perform an **Update** or **Delete** and then use the **Read** options to confirm the database state.

---  

## Contributing  

1. Fork the repository.  
2. Create a feature branch (`git checkout -b feature/your-feature`).  
3. Commit with clear, descriptive messages.  
4. Open a Pull Request describing the change and any required testing steps.  

Maintain the existing code style (Google Java Style) and update this README if new functionality is added.

---  

## License  

This project is licensed under the **MIT License**. See the `LICENSE` file for full terms.
