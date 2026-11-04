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




## Task 02 — INSERT: заполнение данными

### SQL

```sql
-- Вставки из init.sql (один INSERT на таблицу, несколько VALUES).

INSERT INTO Departments (DepartmentName) VALUES
    (N'Разработка'),
    (N'Аналитика'),
    (N'Тестирование'),
    (N'Менеджмент');

INSERT INTO Employees (EmployeeName, Salary, HireDate, DepartmentId, ManagerId) VALUES
    (N'Анна Иванова',    65000, '2020-03-15', 1, NULL),
    (N'Пётр Сидоров',    72000, '2019-07-01', 1, NULL),
    (N'Мария Кузнецова', 55000, '2021-11-20', 1, NULL),
    (N'Иван Петров',     48000, '2023-02-10', 1, NULL),
    (N'Ольга Смирнова',  61000, '2020-09-05', 2, NULL),
    (N'Дмитрий Орлов',   58000, '2022-04-18', 2, NULL),
    (N'Елена Волкова',   52000, '2023-06-01', 2, NULL),
    (N'Сергей Морозов',  45000, '2021-12-12', 3, NULL),
    (N'Наталья Соколова',43000, '2024-01-15', 3, NULL),
    (N'Алексей Лебедев', 90000, '2018-05-25', 4, NULL);

-- ManagerId проставляется после вставки (self-FK):
UPDATE Employees SET ManagerId = 1  WHERE EmployeeId = 4;
UPDATE Employees SET ManagerId = 2  WHERE EmployeeId = 3;
UPDATE Employees SET ManagerId = 5  WHERE EmployeeId = 6;
UPDATE Employees SET ManagerId = 10 WHERE EmployeeId = 9;

INSERT INTO Projects (ProjectName, StartDate) VALUES
    (N'CRM System',      '2023-01-10'),
    (N'Mobile App',      '2023-05-01'),
    (N'Data Warehouse',  '2022-11-20'),
    (N'AI Assistant',    '2024-02-01'),
    (N'Legacy Refactor', '2021-06-15');

INSERT INTO EmployeeProjects (EmployeeId, ProjectId, Hours) VALUES
    (1, 1, 120), (1, 3, 80),
    (2, 1, 150), (2, 4, 60),
    (3, 2, 100), (4, 2, 90),
    (4, 5, 40),  (5, 3, 110),
    (6, 3, 70),  (6, 4, 50),
    (7, 2, 95),  (8, 5, 130),
    (10, 4, 200),(10, 1, 30);

-- Проверка количества строк:
SELECT 'Departments'      AS Tbl, COUNT(*) AS Cnt FROM Departments
UNION ALL SELECT 'Employees',        COUNT(*) FROM Employees
UNION ALL SELECT 'Projects',         COUNT(*) FROM Projects
UNION ALL SELECT 'EmployeeProjects', COUNT(*) FROM EmployeeProjects;
```

### Результат

| Tbl              | Cnt |
|------------------|-----|
| Departments      | 4   |
| Employees        | 10  |
| Projects         | 5   |
| EmployeeProjects | 14  |

Все таблицы заполнены в рамках требований (3–4 / 8–10 / 4–5 / ≥10).
Вставки выполнены одним `INSERT … VALUES (…), (…), …;` на каждую таблицу.

### Вывод

Требование «один INSERT с несколькими VALUES» выполнено во всех четырёх
таблицах. `ManagerId` проставлен отдельными UPDATE после вставки — иначе
self-FK на `Employees` не дал бы вставить строку, ссылающуюся на ещё не
существующего сотрудника. Идентификаторы (`EmployeeId`, `ProjectId`)
не задаются явно — используются `IDENTITY(1,1)`, поэтому назначения в
`EmployeeProjects` ссылаются на Id 1..10 / 1..5 в порядке вставки.




## Task 03 — SELECT + WHERE: базовые выборки

### SQL

```sql
USE PracticeDB;

-- 1) Зарплата > 50000
SELECT EmployeeId, EmployeeName, Salary
FROM Employees
WHERE Salary > 50000
ORDER BY Salary DESC;

-- 2) Нанятые в 2023
SELECT EmployeeId, EmployeeName, HireDate
FROM Employees
WHERE YEAR(HireDate) = 2023
ORDER BY HireDate;

-- 3) Имя начинается на «А»
SELECT EmployeeId, EmployeeName
FROM Employees
WHERE EmployeeName LIKE N'А%'
ORDER BY EmployeeName;

-- 4) Отдел 2
SELECT EmployeeId, EmployeeName, DepartmentId
FROM Employees
WHERE DepartmentId = 2
ORDER BY EmployeeName;

-- 5) Зарплата 40000–60000
SELECT EmployeeId, EmployeeName, Salary
FROM Employees
WHERE Salary BETWEEN 40000 AND 60000
ORDER BY Salary;
```

### Результат

1) > 50000 — 7 строк (Лебедев 90000, Сидоров 72000, Иванова 65000,
   Смирнова 61000, Орлов 58000, Кузнецова 55000, Волкова 52000).

2) 2023 год — 2 строки: Иван Петров (2023-02-10), Елена Волкова (2023-06-01).

3) LIKE N'А%' — 2 строки: Анна Иванова, Алексей Лебедев.

4) DepartmentId = 2 — 3 строки: Ольга Смирнова, Дмитрий Орлов, Елена Волкова.

5) BETWEEN 40000 AND 60000 — 6 строк: Соколова 43000, Морозов 45000,
   Петров 48000, Волкова 52000, Кузнецова 55000, Орлов 58000.

Все пять запросов вернули непустой результат, условия корректны.

### Вывод

- `Salary > 50000` — строгое неравенство (граница не включается).
- `YEAR(HireDate) = 2023` работает, но не sargable (не использует индекс
  на HireDate); для больших таблиц лучше диапазон
  `HireDate >= '2023-01-01' AND HireDate < '2024-01-01'`.
- `LIKE N'А%'` — префикс `N` обязателен для NVARCHAR, иначе кириллица
  сравнивается некорректно при не-Unicode collation.
- `BETWEEN 40000 AND 60000` включает обе границы (>= и <=).
- `DepartmentId = 2` — простой equality по FK-колонке.






## Task 04 — ORDER BY + TOP: сортировка и лимит

### SQL

```sql
USE PracticeDB;

-- 1) Топ-3 по зарплате (убывание)
SELECT TOP (3) EmployeeId, EmployeeName, Salary
FROM Employees
ORDER BY Salary DESC;

-- 2) Топ-5 самых давних по HireDate (возрастание)
SELECT TOP (5) EmployeeId, EmployeeName, HireDate
FROM Employees
ORDER BY HireDate ASC;

-- 3) По отделу (возр.), затем по зарплате (убыв.)
SELECT EmployeeId, EmployeeName, DepartmentId, Salary
FROM Employees
ORDER BY DepartmentId ASC, Salary DESC;
```

### Результат

1) TOP 3 по Salary DESC:
   Лебедев 90000, Сидоров 72000, Иванова 65000.

2) TOP 5 по HireDate ASC (самые давние):
   Лебедев 2018-05-25, Сидоров 2019-07-01, Иванова 2020-03-15,
   Смирнова 2020-09-05, Кузнецова 2021-11-20.

3) ORDER BY DepartmentId ASC, Salary DESC — 10 строк:
   отдел 1: Сидоров 72000, Иванова 65000, Кузнецова 55000, Петров 48000;
   отдел 2: Смирнова 61000, Орлов 58000, Волкова 52000;
   отдел 3: Морозов 45000, Соколова 43000;
   отдел 4: Лебедев 90000.

Порядок строк соответствует заданным правилам.

### Вывод

- TOP без ORDER BY недетерминирован — SQL Server вернёт «любые N»
  строк. Для топа по зарплате обязателен `ORDER BY Salary DESC`.
- «Самые давние» = минимальная HireDate → `ORDER BY HireDate ASC`.
- В многоуровневой сортировке первый ключ доминирует: Лебедев (90000)
  стоит последним, т.к. его отдел (4) последний по DepartmentId.
- При равных значениях ключей сортировка недетерминирована; для
  устойчивости добавляют уникальный ключ (EmployeeId).
- `TOP (N)` — синтаксис с скобками; альтернатива — `OFFSET 0 ROWS
  FETCH NEXT N ROWS ONLY` (SQL Server 2012+).





    ## Task 05 — Агрегация: COUNT, SUM, AVG, MIN, MAX

### SQL

```sql
USE PracticeDB;

-- Все 5 показателей одним запросом
SELECT
    COUNT(*)                                    AS TotalEmployees,
    SUM(Salary)                                 AS TotalSalary,
    ROUND(AVG(Salary), 2)                       AS AvgSalary,
    MIN(Salary)                                 AS MinSalary,
    MAX(Salary)                                 AS MaxSalary,
    COUNT(CASE WHEN Salary > 50000 THEN 1 END)  AS Above50000
FROM Employees;
```

### Результат

| TotalEmployees | TotalSalary | AvgSalary | MinSalary | MaxSalary | Above50000 |
|----------------|-------------|-----------|-----------|-----------|------------|
| 10 | 589000.00 | 58900.00 | 43000.00 | 90000.00 | 7 |

Ручная проверка:
- COUNT = 10 (все сотрудники);
- SUM = 65000+72000+55000+48000+61000+58000+52000+45000+43000+90000 = 589000;
- AVG = 589000 / 10 = 58900.00;
- MIN = 43000 (Соколова), MAX = 90000 (Лебедев);
- > 50000: Иванова, Сидоров, Смирнова, Орлов, Кузнецова, Волкова, Лебедев = 7.

Все 5 значений совпадают с ручной проверкой.

### Вывод

- `COUNT(*)` считает строки, `COUNT(column)` — не-NULL значения колонки.
- `AVG` над DECIMAL(10,2) сохраняет дробную часть; `ROUND(..., 2)`
  приводит к 2 знакам. Ловушка: над INT `AVG` вернул бы целое.
- Количество с зарплатой > 50000 нельзя получить через `WHERE` в том же
  SELECT с общими агрегатами — `WHERE` отфильтрует строки до агрегации.
  Правильно — условный агрегат `COUNT(CASE WHEN Salary > 50000 THEN 1 END)`.
- На пустой таблице COUNT вернёт 0, а SUM/AVG/MIN/MAX — NULL.






## Task 06 — GROUP BY: статистика по отделам

### SQL

```sql
USE PracticeDB;

SELECT
    DepartmentId,
    COUNT(*)              AS EmployeeCount,
    ROUND(AVG(Salary), 2) AS AvgSalary,
    SUM(Salary)           AS TotalSalary
FROM Employees
GROUP BY DepartmentId
ORDER BY AvgSalary DESC;
```

### Результат

| DepartmentId | EmployeeCount | AvgSalary | TotalSalary |
|--------------|---------------|-----------|-------------|
| 4 | 1 | 90000.00 | 90000.00 |
| 1 | 4 | 60000.00 | 240000.00 |
| 2 | 3 | 57000.00 | 171000.00 |
| 3 | 2 | 44000.00 | 88000.00 |

Проверка: сумма EmployeeCount = 10, сумма TotalSalary = 589000 —
совпадает с общими агрегатами из Task 05. На каждый отдел ровно одна
строка, сортировка по средней зарплате по убыванию корректна.

### Вывод

- `GROUP BY DepartmentId` даёт одну строку на отдел; в SELECT можно
  выводить только сгруппированные колонки и агрегаты.
- `ORDER BY AvgSalary` — сортировка по алиасу работает, т.к. ORDER BY
  вычисляется после SELECT (в GROUP BY/WHERE алиасы недоступны).
- Чтобы вывести DepartmentName, нужно либо добавить его в GROUP BY,
  либо джойнить Departments (см. доп. запрос).
- Сумма агрегатов по группам = общий агрегат — хороший sanity check.
