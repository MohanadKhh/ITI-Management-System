# 🎓 ITI Management System

A desktop application built with **Windows Forms (.NET 10)** for managing ITI school data including Courses, Instructors, Departments, Students, Course Sessions and Attendance. The project follows a clean **3-Tier Architecture** with Entity Framework Core (Code First).

---

## 📋 Table of Contents

- [Features](#-features)
- [Architecture Overview](#-architecture-overview)
- [Technologies Used](#-technologies-used)
- [Database Schema](#-database-schema)
- [Demo](#-demo)
- [Installation & Setup](#-installation--setup)

---

## ✨ Features

- **Course Management** — Add, update, soft-delete and search courses with department and instructor assignment
- **Instructor Management** — Full CRUD operations for instructors with department assignment
- **Department Management** — Manage departments with assigned manager (instructor)
- **Student Management** — Manage student records
- **Course Sessions** — Track sessions per course with assigned instructor and date
- **Attendance Tracking** — Record student attendance per session with grade and notes
- **Soft Delete** — Records are never hard-deleted from DB; `IsDeleted` flag used throughout
- **Search & Filter** — Live search on all grids by name
- **Data Validation** — BLL-level validation with user-friendly error messages
- **Referential Integrity** — Prevent deletion of instructors or departments that have related records

---

## 🏗 Architecture Overview

The project is structured as a **3-Tier Architecture** separating concerns across three independent layers:

```
ITI Management System
│
├── ITIApp.DAL          → Data Access Layer
│   ├── Entities/       → EF Core entity models
│   ├── Repositories/   → Generic + specific repositories
│   └── AppDbContext.cs → EF Core DbContext with Fluent API config & seed data
│
├── ITIApp.BLL          → Business Logic Layer
│   ├── Interfaces/     → Service contracts (IManager interfaces)
│   ├── Managers/       → Business rules, validation, service implementations
│   └── Extensions/     → DAL & BLL DI service registration extensions
│
└── ITIApp.UI           → Presentation Layer (Windows Forms)
    ├── Forms/          → MainForm, CourseForm, InstructorForm
    └── Program.cs      → DI container wiring & app entry point
```

### Dependency Flow

```
UI  →  IBLLManager  ←  BLLManager  →  IRepository  ←  Repository  →  AppDbContext
```

Each layer depends only on the **interface** of the layer below — never on the concrete class. All wiring happens in `Program.cs`.

### Key Design Decisions

| Decision | Reason |
|---|---|
| `IDbContextFactory<AppDbContext>` | Prevents concurrency crashes in async WinForms events |
| `async/await` throughout | Non-blocking UI for all DB operations |
| Generic `IRepository<T>` | DRY — shared CRUD for all entities |
| Soft delete (`IsDeleted`) | Preserve data history, safer than hard delete |
| `async void` event handlers | WinForms requirement — always wrapped in try/catch |
| Unsubscribe `SelectionChanged` during load | Prevents unwanted event cascades during DataGridView binding |

---

## 🛠 Technologies Used

| Technology | Version | Purpose |
|---|---|---|
| C# / .NET | 10.0 | Core language and runtime |
| Windows Forms | .NET 10 | Desktop UI framework |
| Entity Framework Core | 10.0.4 | ORM — database access |
| EF Core SQL Server | 10.0.4 | SQL Server provider |
| Microsoft.Extensions.DependencyInjection | 10.0 | DI container |
| Microsoft.Extensions.Configuration | 10.0 | Connection string management |
| SQL Server / SQL Express | — | Database engine |

---

## 🗄 Database Schema

### Entity Relationship Overview

```
Department ──< Instructor >── Course ──< CourseSession
                                │               │
                           Course_Student   CourseSessionAttendance
                                │               │
                            Student ────────────┘
```

### Tables

| Table | Key Columns | Notes |
|---|---|---|
| `Departments` | `DepartmentId`, `ManagerId` (FK → Instructor) | `ManagerId` nullable — circular ref with Instructor |
| `Instructors` | `Id`, `DepartmentId` (FK → Department) | Restrict delete if assigned to Course or Session |
| `Courses` | `Id`, `DepartmentId`, `InstructorId` | Soft delete via `IsDeleted` |
| `Students` | `Id`, `FirstName`, `LastName`, `Phone` | — |
| `Course_Student` | `CourseId`, `StudentId` (composite PK) | Many-to-Many junction table |
| `CourseSessions` | `Id`, `CourseId`, `InstructorId`, `Date` | Soft delete via `IsDeleted` |
| `CourseSessionAttendances` | `Id`, `CourseSessionId`, `StudentId`, `Grade`, `Notes` | Grade and Notes nullable |

### Relationship Rules

```
Department  → Instructors     : One-to-Many   (DeleteBehavior.Restrict)
Department  → Manager         : One-to-One    (DeleteBehavior.Restrict)
Instructor  → Courses         : One-to-Many   (DeleteBehavior.Restrict)
Instructor  → CourseSessions  : One-to-Many   (DeleteBehavior.Restrict)
Course      → CourseSessions  : One-to-Many   (Cascade)
Course      ↔ Students        : Many-to-Many  via Course_Student
CourseSession → Attendances   : One-to-Many   (Cascade)
Student     → Attendances     : One-to-Many   (Cascade)
```

---

## 🎬 Demo

<video 
  src="https://github.com/user-attachments/assets/aad56b23-42fe-425e-92a2-7242664177e9"
  controls
  width="320"
  style="border-radius:20px;">
</video>

---

## ⚙️ Installation & Setup

### Prerequisites

- [.NET 10 SDK](https://dotnet.microsoft.com/download)
- [Visual Studio 2022+](https://visualstudio.microsoft.com/) with **Windows Desktop** workload
- SQL Server or SQL Server Express

### Steps

**1. Clone the repository**
```bash
git clone https://github.com/your-username/ITI-Management-System.git
cd ITI-Management-System
```

**2. Update the connection string**

In `ITIApp.DAL/Extensions/DALServicesExtension.cs`:
```csharp
options.UseSqlServer(@"Server=.\SQLEXPRESS;Database=ITIApp;Trusted_Connection=True;TrustServerCertificate=True;");
```
Replace with your SQL Server instance name if different.

**3. Apply migrations**

Open **Package Manager Console** in Visual Studio, set default project to `ITIApp.DAL` and run:
```powershell
Update-Database
```
This creates the database and seeds initial data automatically.

**4. Set startup project**

Right-click `ITIApp.UI` → **Set as Startup Project**

**5. Run**
```
F5 or Ctrl+F5
```

---

## 📁 Project Structure

```
ITI-DB-Management/
│
├── ITIApp.DAL/
│   ├── Entities/
│   │   ├── Course.cs
│   │   ├── Instructor.cs
│   │   ├── Department.cs
│   │   ├── Student.cs
│   │   ├── CourseStudent.cs
│   │   ├── CourseSession.cs
│   │   └── CourseSessionAttendance.cs
│   ├── Repositories/
│   │   ├── Generic/
│   │   │   ├── IGenericRepository.cs
│   │   │   └── GenericRepository.cs
│   │   ├── Course/
│   │   │   ├── ICourseRepository.cs
│   │   │   └── CourseRepository.cs
│   │   └── Instructor/
│   │       ├── IInstructorRepository.cs
│   │       └── InstructorRepository.cs
│   ├── AppDbContext.cs
│   └── Extensions/
│       └── DALServicesExtension.cs
│
├── ITIApp.BLL/
│   ├── Managers/
│   │   ├── Course/
│   │   │   ├── ICourseManager.cs
│   │   │   └── CourseManager.cs
│   │   └── Instructor/
│   │       ├── IInstructorManager.cs
│   │       └── InstructorManager.cs
│   └── Extensions/
│       └── BLLServicesExtension.cs
│
└── ITIApp.UI/
    ├── Forms/
    │   ├── MainForm.cs / .Designer.cs / .resx
    │   ├── CourseForm.cs / .Designer.cs / .resx
    │   └── InstructorForm.cs / .Designer.cs / .resx
    └── Program.cs
```

---

## 👨‍💻 Author

**Mohanad Khaled**
> Built as a training project at ITI (Information Technology Institute)
