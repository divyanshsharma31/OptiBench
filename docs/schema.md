# Database Schema — AWIP Platform

## Overview

PostgreSQL schema for managing employees, projects, skills, and allocations.

---

## Tables

### 1. Organizations
Tenant/multi-org support — initially 1 org for MVP.

```sql
CREATE TABLE Organizations (
    Id SERIAL PRIMARY KEY,
    Name VARCHAR(255) NOT NULL,
    CreatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

| Column | Type | Constraint |
|--------|------|-----------|
| Id | int | PK |
| Name | varchar(255) | NOT NULL |
| CreatedAt | timestamp | DEFAULT CURRENT_TIMESTAMP |

---

### 2. Employees

Core employee data with status tracking.

```sql
CREATE TABLE Employees (
    Id SERIAL PRIMARY KEY,
    FirstName VARCHAR(100) NOT NULL,
    LastName VARCHAR(100) NOT NULL,
    Email VARCHAR(200) NOT NULL UNIQUE,
    Phone VARCHAR(50),
    Title VARCHAR(100),
    ExperienceYears INT,
    Status VARCHAR(50) NOT NULL DEFAULT 'Bench', -- 'Bench', 'Allocated', 'Training'
    OrganizationId INT NOT NULL REFERENCES Organizations(Id),
    CreatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UpdatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_employees_org ON Employees(OrganizationId);
CREATE INDEX idx_employees_status ON Employees(Status);
```

| Column | Type | Constraint |
|--------|------|-----------|
| Id | int | PK |
| FirstName | varchar(100) | NOT NULL |
| LastName | varchar(100) | NOT NULL |
| Email | varchar(200) | NOT NULL, UNIQUE |
| Phone | varchar(50) | NULL |
| Title | varchar(100) | NULL |
| ExperienceYears | int | NULL |
| Status | varchar(50) | NOT NULL, DEFAULT 'Bench' |
| OrganizationId | int | FK → Organizations(Id) |
| CreatedAt | timestamp | DEFAULT CURRENT_TIMESTAMP |
| UpdatedAt | timestamp | DEFAULT CURRENT_TIMESTAMP |

---

### 3. Skills

Catalog of skills — e.g., "C#", "React", "SQL", "AWS".

```sql
CREATE TABLE Skills (
    Id SERIAL PRIMARY KEY,
    Name VARCHAR(100) NOT NULL UNIQUE,
    Category VARCHAR(100), -- 'Backend', 'Frontend', 'DevOps', etc.
    OrganizationId INT NOT NULL REFERENCES Organizations(Id),
    CreatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_skills_org ON Skills(OrganizationId);
```

| Column | Type | Constraint |
|--------|------|-----------|
| Id | int | PK |
| Name | varchar(100) | NOT NULL, UNIQUE |
| Category | varchar(100) | NULL |
| OrganizationId | int | FK → Organizations(Id) |
| CreatedAt | timestamp | DEFAULT CURRENT_TIMESTAMP |

---

### 4. EmployeeSkills

Junction table: employee + skill + proficiency level.

```sql
CREATE TABLE EmployeeSkills (
    Id SERIAL PRIMARY KEY,
    EmployeeId INT NOT NULL REFERENCES Employees(Id) ON DELETE CASCADE,
    SkillId INT NOT NULL REFERENCES Skills(Id) ON DELETE CASCADE,
    ProficiencyLevel INT NOT NULL CHECK (ProficiencyLevel >= 1 AND ProficiencyLevel <= 10), -- 1-10
    YearsOfExperience INT,
    CreatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UpdatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(EmployeeId, SkillId)
);

CREATE INDEX idx_employeeskills_employee ON EmployeeSkills(EmployeeId);
CREATE INDEX idx_employeeskills_skill ON EmployeeSkills(SkillId);
```

| Column | Type | Constraint |
|--------|------|-----------|
| Id | int | PK |
| EmployeeId | int | FK → Employees(Id), ON DELETE CASCADE |
| SkillId | int | FK → Skills(Id), ON DELETE CASCADE |
| ProficiencyLevel | int | CHECK 1–10 |
| YearsOfExperience | int | NULL |
| CreatedAt | timestamp | DEFAULT CURRENT_TIMESTAMP |
| UpdatedAt | timestamp | DEFAULT CURRENT_TIMESTAMP |
| (EmployeeId, SkillId) | - | UNIQUE |

---

### 5. Projects

Project metadata with status and dates.

```sql
CREATE TABLE Projects (
    Id SERIAL PRIMARY KEY,
    Name VARCHAR(255) NOT NULL,
    Description TEXT,
    Status VARCHAR(50) NOT NULL DEFAULT 'Planning', -- 'Planning', 'In Progress', 'On Hold', 'Completed'
    StartDate DATE,
    EndDate DATE,
    Budget DECIMAL(18,2),
    OrganizationId INT NOT NULL REFERENCES Organizations(Id),
    CreatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UpdatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_projects_org ON Projects(OrganizationId);
CREATE INDEX idx_projects_status ON Projects(Status);
```

| Column | Type | Constraint |
|--------|------|-----------|
| Id | int | PK |
| Name | varchar(255) | NOT NULL |
| Description | text | NULL |
| Status | varchar(50) | NOT NULL, DEFAULT 'Planning' |
| StartDate | date | NULL |
| EndDate | date | NULL |
| Budget | decimal(18,2) | NULL |
| OrganizationId | int | FK → Organizations(Id) |
| CreatedAt | timestamp | DEFAULT CURRENT_TIMESTAMP |
| UpdatedAt | timestamp | DEFAULT CURRENT_TIMESTAMP |

---

### 6. ProjectRequirements

Skill requirements per project — e.g., "need 2 C# devs at level 7+".

```sql
CREATE TABLE ProjectRequirements (
    Id SERIAL PRIMARY KEY,
    ProjectId INT NOT NULL REFERENCES Projects(Id) ON DELETE CASCADE,
    SkillId INT NOT NULL REFERENCES Skills(Id),
    RequiredLevel INT NOT NULL CHECK (RequiredLevel >= 1 AND RequiredLevel <= 10), -- min proficiency
    PositionsNeeded INT NOT NULL DEFAULT 1,
    PositionsFilled INT NOT NULL DEFAULT 0,
    CreatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UpdatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_projectreqs_project ON ProjectRequirements(ProjectId);
CREATE INDEX idx_projectreqs_skill ON ProjectRequirements(SkillId);
```

| Column | Type | Constraint |
|--------|------|-----------|
| Id | int | PK |
| ProjectId | int | FK → Projects(Id), ON DELETE CASCADE |
| SkillId | int | FK → Skills(Id) |
| RequiredLevel | int | CHECK 1–10 |
| PositionsNeeded | int | NOT NULL, DEFAULT 1 |
| PositionsFilled | int | NOT NULL, DEFAULT 0 |
| CreatedAt | timestamp | DEFAULT CURRENT_TIMESTAMP |
| UpdatedAt | timestamp | DEFAULT CURRENT_TIMESTAMP |

---

### 7. Assignments

Track which employees are assigned to which projects.

```sql
CREATE TABLE Assignments (
    Id SERIAL PRIMARY KEY,
    EmployeeId INT NOT NULL REFERENCES Employees(Id),
    ProjectId INT NOT NULL REFERENCES Projects(Id) ON DELETE CASCADE,
    StartDate DATE NOT NULL,
    EndDate DATE,
    Role VARCHAR(100), -- e.g., "Backend Lead", "Frontend Developer"
    Status VARCHAR(50) NOT NULL DEFAULT 'Active', -- 'Active', 'Paused', 'Completed'
    CreatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UpdatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(EmployeeId, ProjectId)
);

CREATE INDEX idx_assignments_employee ON Assignments(EmployeeId);
CREATE INDEX idx_assignments_project ON Assignments(ProjectId);
```

| Column | Type | Constraint |
|--------|------|-----------|
| Id | int | PK |
| EmployeeId | int | FK → Employees(Id) |
| ProjectId | int | FK → Projects(Id), ON DELETE CASCADE |
| StartDate | date | NOT NULL |
| EndDate | date | NULL |
| Role | varchar(100) | NULL |
| Status | varchar(50) | NOT NULL, DEFAULT 'Active' |
| CreatedAt | timestamp | DEFAULT CURRENT_TIMESTAMP |
| UpdatedAt | timestamp | DEFAULT CURRENT_TIMESTAMP |
| (EmployeeId, ProjectId) | - | UNIQUE |

---

## Relationships Diagram

```
Organizations (1)
  ├─→ (∞) Employees
  │     ├─→ (∞) EmployeeSkills ←─ (1) Skills
  │     │
  │     └─→ (∞) Assignments ←─ (1) Projects
  │
  ├─→ (∞) Skills
  │
  ├─→ (∞) Projects
  │     ├─→ (∞) ProjectRequirements ←─ (1) Skills
  │     └─→ (∞) Assignments ←─ (1) Employees
```

---

## Seed Data Notes

**Day 4 (AWIP-8)**: HK will seed **200 employees** from `Employees.xlsx` with random skills (100+ skill combinations) and proficiency levels 1–10.

**Day 4 (AWIP-10)**: HK will seed **30 projects** with random skill requirements.

All inserts will use `OrganizationId = 1` (hardcoded for MVP).

---

## Migration Strategy (Entity Framework Core)

```bash
# In server/ folder
dotnet ef migrations add InitialCreate
dotnet ef database update
```

Models will be in `server/Models/`:
- `Organization.cs`
- `Employee.cs`
- `Skill.cs`
- `EmployeeSkill.cs`
- `Project.cs`
- `ProjectRequirement.cs`
- `Assignment.cs`

`DbContext` at `server/Data/AwipDbContext.cs`.
