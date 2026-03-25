# Project‑CRUD‑Demo‑MySQL‑Menu‑Driven‑Application  

A **menu‑driven Java console** that performs full **CRUD** (Create, Read, Update, Delete) operations on a **project‑management** database.  
The current lesson focuses on **updating** project data and **deleting** a project together with all of its dependent rows by using `ON DELETE CASCADE`.

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



---  

## Project Overview
The program lets a user manage **projects**, their **materials**, **steps**, and **categories** through a simple numbered console menu:

* **Create** – add new projects, materials, steps, categories, and links.  
* **Read** – list projects, view a project’s full data (including related rows).  
* **Update** – modify any mutable column of a project (or its child tables).  
* **Delete** – remove a project; all related rows in `material`, `step`, and `project_category` are automatically deleted thanks to `ON DELETE CASCADE`.  

All database interactions are performed with **JDBC prepared statements**, and the return value of `PreparedStatement.executeUpdate()` is used to confirm success.

---  

## Features
| Feature | Description |
|---------|-------------|
| **Create** | Insert rows into `project`, `material`, `step`, `category`, and `project_category`. |
| **Read** | List all projects; view details of a single project (including its materials, steps, and categories). |
| **Update** | Parameterised `UPDATE` statements for any editable column; success is verified via the `executeUpdate()` result. |
| **Delete** | Delete a project; cascade removes dependent rows in `material`, `step`, and `project_category`. |
| **Feedback** | `executeUpdate()` > 0 ⇒ operation succeeded; = 0 ⇒ no rows affected. |
| **Console UI** | Simple numbered menu with input validation; runs in any terminal that supports Java. |

---  

## Technology Stack
| Layer | Technology |
|-------|------------|
| Language | Java 11+ |
| Build | Maven |
| Database | MySQL 5.7+ (InnoDB) |
| JDBC driver | MySQL Connector/J 8.0+ |


---  

## Database Schema  

> **Save the statements below as `src/main/resources/schema.sql`** (or execute them directly in MySQL after creating the database).

```sql
-- ----------------------------------------------------
-- 1️⃣  DROP existing objects (useful for re‑runs)
-- ----------------------------------------------------
DROP TABLE IF EXISTS project_category;
DROP TABLE IF EXISTS category;
DROP TABLE IF EXISTS step;
DROP TABLE IF EXISTS material;
DROP TABLE IF EXISTS project;

-- ----------------------------------------------------
-- 2️⃣  TABLE: project
-- ----------------------------------------------------
CREATE TABLE project (
    project_id      INT AUTO_INCREMENT NOT NULL,
    project_name    VARCHAR(128) NOT NULL,
    estimated_hours DECIMAL(7,2),
    actual_hours    DECIMAL(7,2),
    difficulty      INT,
    notes           TEXT,
    PRIMARY KEY (project_id)
) ENGINE=InnoDB;

-- ----------------------------------------------------
-- 3️⃣  TABLE: material (one‑to‑many → project)
-- ----------------------------------------------------
CREATE TABLE material (
    material_id   INT AUTO_INCREMENT NOT NULL,
    project_id    INT NOT NULL,
    material_name VARCHAR(128) NOT NULL,
    num_required  INT,
    cost          DECIMAL(7,2),
    PRIMARY KEY (material_id),
    FOREIGN KEY (project_id)
        REFERENCES project(project_id)
        ON DELETE CASCADE
) ENGINE=InnoDB;

-- ----------------------------------------------------
-- 4️⃣  TABLE: step (one‑to‑many → project)
-- ----------------------------------------------------
CREATE TABLE step (
    step_id    INT AUTO_INCREMENT NOT NULL,
    project_id INT NOT NULL,
    step_text  TEXT NOT NULL,
    step_order INT NOT NULL,
    PRIMARY KEY (step_id),
    FOREIGN KEY (project_id)
        REFERENCES project(project_id)
        ON DELETE CASCADE
) ENGINE=InnoDB;

-- ----------------------------------------------------
-- 5️⃣  TABLE: category (stand‑alone)
-- ----------------------------------------------------
CREATE TABLE category (
    category_id   INT AUTO_INCREMENT NOT NULL,
    category_name VARCHAR(128),
    PRIMARY KEY (category_id)
) ENGINE=InnoDB;

-- ----------------------------------------------------
-- 6️⃣  TABLE: project_category (many‑to‑many)
-- ----------------------------------------------------
CREATE TABLE project_category (
    project_id  INT NOT NULL,
    category_id INT NOT NULL,
    FOREIGN KEY (project_id)
        REFERENCES project(project_id)
        ON DELETE CASCADE,
    FOREIGN KEY (category_id)
        REFERENCES category(category_id)
        ON DELETE CASCADE,
    UNIQUE KEY (project_id, category_id)
) ENGINE=InnoDB;
```

### Why this schema works
* All tables use **InnoDB**, so foreign‑key constraints are enforced.  
* `material`, `step`, and `project_category` reference `project` (or `category`) with **`ON DELETE CASCADE`**, guaranteeing automatic cleanup of child rows when a project or category is removed.  
* A **composite unique key** on `(project_id, category_id)` prevents duplicate links between the same project and category.  

---  

## Prerequisites
- JDK 11 or newer  
- Maven 3.6+  
- MySQL server (5.7+), with the InnoDB engine enabled  
- Network access to the MySQL instance  

---  

## Setup & Installation  

1. **Clone the repository**  
   ```bash
   git clone https://github.com/Karla-Sanchez-Osuna/Project-CRUD-Demo-MySQL-Menu-Driven-Application.git
   cd Project-CRUD-Demo-MySQL-Menu-Driven-Application
   ```

2. **Create the database** (replace *your_user*/*your_password* as appropriate)  
   ```sql
   CREATE DATABASE projects;
   USE projects;
   SOURCE src/main/resources/schema.sql;   -- runs the DDL above
   ```

3. **Configure the JDBC connection** – edit `src/main/resources/db.properties`  

   ```properties
   jdbc.url=jdbc:mysql://localhost:3306/projects
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

When started you’ll see:

```
=== Project CRUD Demo ===
1) Insert a new project
2) List all projects
3) View project details
4) Update a project
5) Delete a project
0) Exit
Enter your selection:
```

Enter the number of the desired operation and follow the prompts. All input is validated; erroneous entries simply return you to the menu.

---  

## Menu Options  

| Option | Action |
|--------|--------|
| **1** | Prompt for project data (name, estimated/actual hours, difficulty, notes) and insert it. |
| **2** | Show a compact list of all projects (`project_id – project_name`). |
| **3** | Ask for a `project_id` and display the full project record **plus** its associated materials, steps, and categories. |
| **4** | Choose a project, then select which field(s) to modify; the program issues a prepared `UPDATE` and reports the number of rows affected. |
| **5** | Delete a project by ID; MySQL automatically removes related rows in `material`, `step`, and `project_category` because of `ON DELETE CASCADE`. |
| **0** | Gracefully close the JDBC connection and exit. |

---  

## Learning Objectives  

- **Modify** project attributes using a prepared `UPDATE` statement.  
- **Delete** a project and rely on `ON DELETE CASCADE` to clean up child rows in `material`, `step`, and the many‑to‑many link table.  
- **Observe** the cascade effect by checking that after a deletion the related rows are gone.  
- **Use** the integer returned by `PreparedStatement.executeUpdate()` to determine whether the operation actually changed any rows.  


```

For manual verification, after performing an **Update** or **Delete**, run the corresponding **Read** option to see the updated state of the database.

---  

