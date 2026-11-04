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






## Task 07 — GROUP BY + HAVING: фильтрация групп

### SQL

```sql
USE PracticeDB;

-- 1) Отделы с > 2 сотрудников
SELECT DepartmentId, COUNT(*) AS EmployeeCount
FROM Employees
GROUP BY DepartmentId
HAVING COUNT(*) > 2
ORDER BY EmployeeCount DESC;

-- 2) Отделы со средней зарплатой > 55000
SELECT DepartmentId, ROUND(AVG(Salary), 2) AS AvgSalary
FROM Employees
GROUP BY DepartmentId
HAVING AVG(Salary) > 55000
ORDER BY AvgSalary DESC;

-- 3) Отделы с суммарной зарплатой > 150000
SELECT DepartmentId, SUM(Salary) AS TotalSalary
FROM Employees
GROUP BY DepartmentId
HAVING SUM(Salary) > 150000
ORDER BY TotalSalary DESC;

-- Бонус: все три условия сразу
SELECT DepartmentId,
       COUNT(*)              AS EmployeeCount,
       ROUND(AVG(Salary), 2) AS AvgSalary,
       SUM(Salary)           AS TotalSalary
FROM Employees
GROUP BY DepartmentId
HAVING COUNT(*) > 2
   AND AVG(Salary) > 55000
   AND SUM(Salary) > 150000
ORDER BY AvgSalary DESC;
```

### Результат

1) COUNT(*) > 2 → отделы 1 (4 сотрудника) и 2 (3).

2) AVG(Salary) > 55000 → отделы 4 (90000), 1 (60000), 2 (57000).

3) SUM(Salary) > 150000 → отделы 1 (240000) и 2 (171000).

Бонус (все три условия) → отделы 1 и 2 (оба проходят по COUNT, AVG и SUM).

Все возвращённые отделы удовлетворяют условиям; отдел 3 и 4
отфильтрованы.

### Вывод

- `HAVING` фильтрует **группы после** агрегации, `WHERE` — строки
  **до** группировки. Агрегат (`COUNT`, `SUM`, `AVG`) нельзя
  использовать в WHERE — только в HAVING или в SELECT.
- AVG и SUM дают разную картину: отдел 4 лидирует по средней зарплате
  (90000), но не проходит по SUM (>150000) и COUNT (>2), т.к. в нём
  один сотрудник.
- В HAVING можно комбинировать несколько агрегатных условий через AND/OR.
- HAVING с не-агрегатными условиями технически возможен, но
  неэффективен — такие условия лучше переносить в WHERE.




    ## Task 08 — INNER JOIN: сотрудники и отделы

### SQL

```sql
USE PracticeDB;

SELECT
    e.EmployeeName,
    d.DepartmentName,
    e.Salary
FROM Employees e
INNER JOIN Departments d ON d.DepartmentId = e.DepartmentId
ORDER BY d.DepartmentName ASC, e.Salary DESC;

-- Проверка: сотрудников без отдела нет
SELECT COUNT(*) AS Orphans
FROM Employees e
LEFT JOIN Departments d ON d.DepartmentId = e.DepartmentId
WHERE d.DepartmentId IS NULL;
```

### Результат

10 строк, у каждого сотрудника есть DepartmentName:

| EmployeeName | DepartmentName | Salary |
|--------------|----------------|--------|
| Ольга Смирнова | Аналитика | 61000 |
| Дмитрий Орлов | Аналитика | 58000 |
| Елена Волкова | Аналитика | 52000 |
| Алексей Лебедев | Менеджмент | 90000 |
| Пётр Сидоров | Разработка | 72000 |
| Анна Иванова | Разработка | 65000 |
| Мария Кузнецова | Разработка | 55000 |
| Иван Петров | Разработка | 48000 |
| Сергей Морозов | Тестирование | 45000 |
| Наталья Соколова | Тестирование | 43000 |

Проверка Orphans = 0: сотрудников без отдела нет, INNER JOIN
не отбросил ни одной строки (в нашей БД INNER и LEFT дают 10 строк).

### Вывод

- INNER JOIN возвращает только совпавшие пары; строки без пары
  отбрасываются с обеих сторон. У нас FK гарантирует, что сирот нет,
  поэтому вернулись все 10 сотрудников.







    ## Task 09 — LEFT JOIN: сотрудники без проектов

### SQL

```sql
USE PracticeDB;

SELECT
    e.EmployeeId,
    e.EmployeeName
FROM Employees e
LEFT JOIN EmployeeProjects ep ON ep.EmployeeId = e.EmployeeId
WHERE ep.EmployeeId IS NULL
ORDER BY e.EmployeeId;
```

### Результат

| EmployeeId | EmployeeName |
|------------|--------------|
| 9 | Наталья Соколова |

Из 10 сотрудников 9 назначены хотя бы на один проект; единственный
без назначений — Наталья Соколова (Id 9). Проверка: 10 − 9 = 1.

### Вывод

- Условие `IS NULL` должно быть в WHERE, не в ON. Если положить его
  в ON, LEFT JOIN вырождается и возвращает всех 10 сотрудников.
- `= NULL` не работает (трёхзначная логика) — только `IS NULL`.
- Альтернативы: NOT EXISTS (часто эффективнее) и NOT IN (осторожно:
  при NULL в подзапросе вернёт пусто). Все три дают тот же результат.







   ## Task 10 — Множественный JOIN: сотрудники, отделы, проекты

### SQL

```sql
USE PracticeDB;

SELECT
    e.EmployeeName,
    d.DepartmentName,
    p.ProjectName,
    ep.Hours
FROM Employees e
INNER JOIN Departments      d  ON d.DepartmentId = e.DepartmentId
INNER JOIN EmployeeProjects ep ON ep.EmployeeId  = e.EmployeeId
INNER JOIN Projects         p  ON p.ProjectId    = ep.ProjectId
ORDER BY d.DepartmentName ASC, e.EmployeeName ASC, p.ProjectName ASC;

-- Sanity check: столько же строк, сколько назначений
SELECT COUNT(*) AS AssignedRows FROM EmployeeProjects;
```

### Результат

14 строк — по одной на каждое назначение, у каждого видны все 4
атрибута (сотрудник, отдел, проект, часы). COUNT совпадает с
EmployeeProjects (14), значит INNER JOIN не отбросил ни одной строки.

Порядок: отдел ↑ (Аналитика, Менеджмент, Разработка, Тестирование),
внутри отдела — сотрудник ↑, внутри сотрудника — проект ↑.

Фрагмент:
| EmployeeName | DepartmentName | ProjectName | Hours |
|--------------|----------------|-------------|-------|
| Дмитрий Орлов | Аналитика | AI Assistant | 50 |
| Дмитрий Орлов | Аналитика | Data Warehouse | 70 |
| ... | ... | ... | ... |
| Сергей Морозов | Тестирование | Legacy Refactor | 130 |

### Вывод

- Четыре таблицы соединяются цепочкой INNER JOIN по FK: Employees →
  Departments, Employees → EmployeeProjects, EmployeeProjects → Projects.
- Порядок JOIN в тексте запроса не влияет на результат — важен граф
  условий ON; оптимизатор сам выбирает порядок соединения.
- INNER JOIN отбрасывает строки без пары на каждом шаге. У нас 14 = 14,
  потерь нет.
- Алиасы обязательны: DepartmentId, EmployeeId, ProjectId встречаются
  в двух таблицах каждая.






   ## Task 11 — Self-join: иерархия руководителей

### SQL

```sql
USE PracticeDB;

SELECT
    e.EmployeeName,
    m.EmployeeName AS ManagerName
FROM Employees e
LEFT JOIN Employees m ON m.EmployeeId = e.ManagerId
ORDER BY e.EmployeeId;
```

### Результат

10 строк — по одной на каждого сотрудника:

| EmployeeName | ManagerName |
|--------------|-------------|
| Анна Иванова | NULL |
| Пётр Сидоров | NULL |
| Мария Кузнецова | Пётр Сидоров |
| Иван Петров | Анна Иванова |
| Ольга Смирнова | NULL |
| Дмитрий Орлов | Ольга Смирнова |
| Елена Волкова | NULL |
| Сергей Морозов | NULL |
| Наталья Соколова | Алексей Лебедев |
| Алексей Лебедев | NULL |

У 4 сотрудников с ManagerId виден руководитель, у 6 (ManagerId IS NULL) —
ManagerName = NULL. COUNT = 10 = COUNT(*) из Employees.

### Вывод

- Self-join: Employees соединяется сама с собой через алиасы e (сотрудник)
  и m (менеджер).
- LEFT JOIN обязателен: при INNER JOIN строки с ManagerId IS NULL не
  нашли бы пару и выпали — вернулось бы 4 строки вместо 10, а требование
  «без руководителя → NULL» нарушилось бы.
- Алиасы e/m обязательны, иначе SQL Server не различит два экземпляра
  одной таблицы.
- m.EmployeeName AS ManagerName — переименование, иначе две колонки с
  одним именем.
- Self-join даёт только один уровень иерархии; для многоуровневой нужен
  рекурсивный CTE (уровень 3).






    ## Task 12 — Подзапросы в WHERE

### SQL

```sql
USE PracticeDB;

-- 1) Зарплата выше средней (средняя = 58900)
SELECT EmployeeId, EmployeeName, Salary
FROM Employees
WHERE Salary > (SELECT AVG(Salary) FROM Employees)
ORDER BY Salary DESC;

-- 2) Сотрудники с максимальной зарплатой (= 90000)
SELECT EmployeeId, EmployeeName, Salary
FROM Employees
WHERE Salary = (SELECT MAX(Salary) FROM Employees);
```

### Результат

Средняя зарплата = 589000 / 10 = 58900.

1) Salary > 58900 → 4 строки:
   Лебедев 90000, Сидоров 72000, Иванова 65000, Смирнова 61000.

2) Salary = 90000 → 1 строка: Алексей Лебедев.

Оба запроса возвращают только подходящих сотрудников; ручная проверка
совпадает.

### Вывод

- Скалярный подзапрос возвращает одно значение; в WHERE с ним можно
  сравнивать через >, <, =, >=, <=, <>. Для множеств — IN/EXISTS
  (Task 13).
- Агрегат (AVG/MAX) нельзя использовать в WHERE напрямую — нужен
  подзапрос (или HAVING/окно).
- Некоррелированный подзапрос (без ссылки на внешнюю таблицу)
  вычисляется один раз, результат используется для всех строк.
- Граница > строгая: ровно 58900 не попал бы. = MAX вернёт всех
  с максимальной зарплатой, если их несколько.
- Если бы AVG вернул NULL (пустая таблица), Salary > NULL дал бы
  UNKNOWN → 0 строк.





   ## Task 13 — Скалярные подзапросы в SELECT

### SQL

```sql
USE PracticeDB;

SELECT
    e.EmployeeName,
    e.Salary,
    e.DepartmentId,
    (SELECT AVG(e2.Salary)
     FROM Employees e2
     WHERE e2.DepartmentId = e.DepartmentId) AS DeptAvgSalary,
    ROUND(e.Salary - (SELECT AVG(e2.Salary)
                      FROM Employees e2
                      WHERE e2.DepartmentId = e.DepartmentId), 2) AS Deviation
FROM Employees e
ORDER BY e.DepartmentId, e.EmployeeId;
```

### Результат 

10 строк; средние по отделам: 1 → 60000, 2 → 57000, 3 → 44000, 
4 → 90000. Отклонения (Salary − Avg): 

| EmployeeName | Salary | Dept | Avg | Deviation |
|--------------|--------|------|-----|-----------|
| Анна Иванова | 65000 | 1 | 60000 | 5000.00 |
| Пётр Сидоров | 72000 | 1 | 60000 | 12000.00 |
| Мария Кузнецова | 55000 | 1 | 60000 | -5000.00 |
| Иван Петров | 48000 | 1 | 60000 | -12000.00 |
| Ольга Смирнова | 61000 | 2 | 57000 | 4000.00 |
| Дмитрий Орлов | 58000 | 2 | 57000 | 1000.00 |
| Елена Волкова | 52000 | 2 | 57000 | -5000.00 |
| Сергей Морозов | 45000 | 3 | 44000 | 1000.00 |
| Наталья Соколова | 43000 | 3 | 44000 | -1000.00 |
| Алексей Лебедев | 90000 | 4 | 90000 | 0.00 |

Sanity check: сумма отклонений внутри каждого отдела = 0. 
(проверено отдельным запросом — по каждому DeptId SumDev = 0.00) 

### Вывод 

- Подзапрос коррелированный: ссылается на e.DepartmentId внешней строки,
  поэтому вычисляется для каждой строки. 
- Скалярность: AVG с WHERE DepartmentId = … возвращает одно значение —
  валидно в SELECT. 
- Подзапрос в моём варианте дублируется (для Avg и Deviation); чище —
  через CTE или оконную функцию AVG() OVER (PARTITION BY DepartmentId)
  (уровень 3). 
- Сумма отклонений внутри отдела = 0 — свойство среднего, хороший
  sanity check. 





   ## Task 14 — EXISTS / NOT EXISTS

### SQL

```sql
USE PracticeDB;

-- 1) Сотрудники с хотя бы одним проектом (EXISTS)
SELECT e.EmployeeId, e.EmployeeName
FROM Employees e
WHERE EXISTS (
    SELECT 1 FROM EmployeeProjects ep
    WHERE ep.EmployeeId = e.EmployeeId
)
ORDER BY e.EmployeeId;

-- 2) Сотрудники без проектов (NOT EXISTS)
SELECT e.EmployeeId, e.EmployeeName
FROM Employees e
WHERE NOT EXISTS (
    SELECT 1 FROM EmployeeProjects ep
    WHERE ep.EmployeeId = e.EmployeeId
)
ORDER BY e.EmployeeId;

-- Sanity check: 9 + 1 = 10
SELECT
  (SELECT COUNT(*) FROM Employees e
   WHERE EXISTS (SELECT 1 FROM EmployeeProjects ep
                 WHERE ep.EmployeeId = e.EmployeeId))      AS WithProjects,
  (SELECT COUNT(*) FROM Employees e
   WHERE NOT EXISTS (SELECT 1 FROM EmployeeProjects ep
                     WHERE ep.EmployeeId = e.EmployeeId))  AS WithoutProjects,
  (SELECT COUNT(*) FROM Employees)                          AS Total;
```

### Результат

1) EXISTS → 9 строк: сотрудники 1, 2, 3, 4, 5, 6, 7, 8, 10.

2) NOT EXISTS → 1 строка: Наталья Соколова (Id 9).

Sanity check: WithProjects = 9, WithoutProjects = 1, Total = 10.
9 + 1 = 10 — полное разбиение, совпадает с заданиями 8 (9 человек
с проектами) и 9 (1 без проектов).









## Task 15 — UNION / UNION ALL

### SQL

```sql
USE PracticeDB;

-- A: Salary > 60000 (4 строки)
-- B: HireDate < '2022-01-01' (6 строк)

-- 1) UNION ALL — с дублями
SELECT EmployeeId, EmployeeName, Salary, HireDate
FROM Employees WHERE Salary > 60000
UNION ALL
SELECT EmployeeId, EmployeeName, Salary, HireDate
FROM Employees WHERE HireDate < '2022-01-01'
ORDER BY EmployeeId;

-- 2) UNION — уникальные строки
SELECT EmployeeId, EmployeeName, Salary, HireDate
FROM Employees WHERE Salary > 60000
UNION
SELECT EmployeeId, EmployeeName, Salary, HireDate
FROM Employees WHERE HireDate < '2022-01-01'
ORDER BY EmployeeId;

-- 3) Сравнение количества строк
SELECT 'UNION ALL' AS Variant, COUNT(*) AS Cnt FROM (
    SELECT EmployeeId FROM Employees WHERE Salary > 60000
    UNION ALL
    SELECT EmployeeId FROM Employees WHERE HireDate < '2022-01-01'
) t
UNION ALL
SELECT 'UNION', COUNT(*) FROM (
    SELECT EmployeeId FROM Employees WHERE Salary > 60000
    UNION
    SELECT EmployeeId FROM Employees WHERE HireDate < '2022-01-01'
) t;
```

### Результат

A (Salary > 60000) = 4 строки: Иванова, Сидоров, Смирнова, Лебедев.
B (HireDate < 2022-01-01) = 6 строк: Иванова, Сидоров, Кузнецова,
Смирнова, Морозов, Лебедев.

Пересечение A ∩ B = 4 (Иванова, Сидоров, Смирнова, Лебедев) —
попадают в оба запроса.

| Variant | Cnt |
|---------|-----|
| UNION ALL | 10 |
| UNION | 6 |

Разница 10 − 6 = 4 = размер пересечения. UNION убрал дубли,
UNION ALL оставил. Требование «разница видна по количеству строк»
выполнено.

### Вывод

- UNION ALL оставляет все строки (4 + 6 = 10); UNION удаляет дубли
  (10 − 4 = 6). UNION = UNION ALL + DISTINCT.
- ORDER BY при UNION — только в конце, после последнего SELECT;
  имена колонок для сортировки берутся из первого SELECT.
- UNION требует одинакового числа колонок и совместимых типов;
  имена колонок результата — из первого SELECT.





   ## Task 16 — UPDATE: изменение данных

### SQL


```sql
USE PracticeDB;

-- === ДО ===
SELECT EmployeeId, EmployeeName, Salary, HireDate, DepartmentId
FROM Employees ORDER BY EmployeeId;

BEGIN TRANSACTION;

-- UPDATE 1: зарплата сотрудников отдела 1 +10%
UPDATE Employees SET Salary = Salary * 1.10 WHERE DepartmentId = 1;

-- UPDATE 2: зарплата нанятых до 2022 года +5000
UPDATE Employees SET Salary = Salary + 5000 WHERE HireDate < '2022-01-01';

-- UPDATE 3: HireDate одного сотрудника (Id 4)
UPDATE Employees SET HireDate = '2023-03-01' WHERE EmployeeId = 4;

-- === ПОСЛЕ ===
SELECT EmployeeId, EmployeeName, Salary, HireDate, DepartmentId
FROM Employees ORDER BY EmployeeId;

COMMIT;
```

### Результат

**До UPDATE (исходные значения):**

| Id | Имя | Salary | HireDate | Dept |
|----|-----|--------|----------|------|
| 1 | Анна Иванова | 65000.00 | 2020-03-15 | 1 |
| 2 | Пётр Сидоров | 72000.00 | 2019-07-01 | 1 |
| 3 | Мария Кузнецова | 55000.00 | 2021-11-20 | 1 |
| 4 | Иван Петров | 48000.00 | 2023-02-10 | 1 |
| 5 | Ольга Смирнова | 61000.00 | 2020-09-05 | 2 |
| 6 | Дмитрий Орлов | 58000.00 | 2022-04-18 | 2 |
| 7 | Елена Волкова | 52000.00 | 2023-06-01 | 2 |
| 8 | Сергей Морозов | 45000.00 | 2021-12-12 | 3 |
| 9 | Наталья Соколова | 43000.00 | 2024-01-15 | 3 |
| 10 | Алексей Лебедев | 90000.00 | 2018-05-25 | 4 |

**После трёх UPDATE:**

| Id | Имя | Salary было → стало | HireDate было → стало | Dept |
|----|-----|---------------------|------------------------|------|
| 1 | Анна Иванова | 65000 → **76500.00** | 2020-03-15 (без изменений) | 1 |
| 2 | Пётр Сидоров | 72000 → **84200.00** | 2019-07-01 (без изменений) | 1 |
| 3 | Мария Кузнецова | 55000 → **65500.00** | 2021-11-20 (без изменений) | 1 |
| 4 | Иван Петров | 48000 → **52800.00** | 2023-02-10 → **2023-03-01** | 1 |
| 5 | Ольга Смирнова | 61000 → **66000.00** | 2020-09-05 (без изменений) | 2 |
| 6 | Дмитрий Орлов | 58000 → **58000.00** (не тронут) | 2022-04-18 (без изменений) | 2 |
| 7 | Елена Волкова | 52000 → **52000.00** (не тронута) | 2023-06-01 (без изменений) | 2 |
| 8 | Сергей Морозов | 45000 → **50000.00** | 2021-12-12 (без изменений) | 3 |
| 9 | Наталья Соколова | 43000 → **43000.00** (не тронута) | 2024-01-15 (без изменений) | 3 |
| 10 | Алексей Лебедев | 90000 → **95000.00** | 2018-05-25 (без изменений) | 4 |

**Кто изменён:**
- По зарплате: Id **1, 2, 3, 4** (UPDATE 1, отдел 1) и Id **1, 2, 3, 5, 8, 10** (UPDATE 2, HireDate < 2022). Объединение — **7 строк** (1, 2, 3, 4, 5, 8, 10).
- Не тронуты по зарплате: **6, 7, 9** (не из отдела 1 и наняты в 2022+).
- HireDate изменён только у **Id 4**.