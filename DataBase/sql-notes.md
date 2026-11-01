## Task 01 — Создание БД PracticeDB и таблиц

### SQL

```sql
-- Полный DDL — в database/init.sql (CREATE DATABASE + 4 × CREATE TABLE).
-- Здесь — проверочные запросы, подтверждающие выполнение задания.

-- 1) Таблицы существуют
SELECT TABLE_NAME
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_TYPE = 'BASE TABLE'
ORDER BY TABLE_NAME;

-- 2) Первичные ключи
SELECT tc.TABLE_NAME, tc.CONSTRAINT_NAME, kcu.COLUMN_NAME, kcu.ORDINAL_POSITION
FROM INFORMATION_SCHEMA.TABLE_CONSTRAINTS tc
JOIN INFORMATION_SCHEMA.KEY_COLUMN_USAGE kcu
  ON kcu.CONSTRAINT_NAME = tc.CONSTRAINT_NAME
 AND kcu.TABLE_NAME      = tc.TABLE_NAME
WHERE tc.CONSTRAINT_TYPE = 'PRIMARY KEY'
ORDER BY tc.TABLE_NAME, kcu.ORDINAL_POSITION;

-- 3) Внешние ключи
SELECT fk.name AS FK_Name,
       OBJECT_NAME(fk.parent_object_id)     AS ChildTable,
       COL_NAME(fkc.parent_object_id, fkc.parent_column_id)         AS ChildColumn,
       OBJECT_NAME(fk.referenced_object_id) AS ParentTable,
       COL_NAME(fkc.referenced_object_id, fkc.referenced_column_id) AS ParentColumn
FROM sys.foreign_keys fk
JOIN sys.foreign_key_columns fkc ON fkc.constraint_object_id = fk.object_id
ORDER BY ChildTable, FK_Name;

-- 4) Типы данных
SELECT TABLE_NAME, COLUMN_NAME, DATA_TYPE,
       CHARACTER_MAXIMUM_LENGTH AS MaxLen,
       NUMERIC_PRECISION AS NumPrec, NUMERIC_SCALE AS NumScale,
       IS_NULLABLE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME IN ('Departments','Employees','Projects','EmployeeProjects')
ORDER BY TABLE_NAME, ORDINAL_POSITION;
```

### Результат

**1) Таблицы (4):**

| TABLE_NAME       |
|------------------|
| Departments      |
| EmployeeProjects |
| Employees        |
| Projects         |

**2) PRIMARY KEY (5 записей):**

| TABLE_NAME       | CONSTRAINT_NAME      | COLUMN_NAME  | ORDINAL_POSITION |
|------------------|----------------------|--------------|------------------|
| Departments      | PK__Departments__…   | DepartmentId | 1                |
| EmployeeProjects | PK_EmployeeProjects  | EmployeeId   | 1                |
| EmployeeProjects | PK_EmployeeProjects  | ProjectId    | 2                |
| Employees        | PK__Employees__…     | EmployeeId   | 1                |
| Projects         | PK__Projects__…      | ProjectId    | 1                |

**3) FOREIGN KEY (4):**

| FK_Name                  | ChildTable       | ChildColumn  | ParentTable  | ParentColumn |
|--------------------------|------------------|--------------|--------------|--------------|
| FK_Employees_Departments | Employees        | DepartmentId | Departments  | DepartmentId |
| FK_Employees_Manager     | Employees        | ManagerId    | Employees    | EmployeeId   |
| FK_EP_Employees          | EmployeeProjects | EmployeeId   | Employees    | EmployeeId   |
| FK_EP_Projects           | EmployeeProjects | ProjectId    | Projects     | ProjectId    |

**4) Типы данных:** соответствуют требованию —
`INT` (Id-колонки, `Hours`), `NVARCHAR(100)` (имена), `DECIMAL(10,2)` (`Salary`), `DATE` (`HireDate`, `StartDate`); `ManagerId` — `INT NULL`.

### Вывод

DDL из `init.sql` создаёт 4 таблицы с PK и FK, включая self-FK
`Employees.ManagerId → Employees.EmployeeId` и составной PK в
`EmployeeProjects` (связь многие-ко-многим). Проверки через
INFORMATION_SCHEMA и `sys.foreign_keys` подтверждают: таблицы существуют,
связи корректны, типы соответствуют заданию.