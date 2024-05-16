DROP SCHEMA IF EXISTS pewlett_hackard CASCADE;

CREATE SCHEMA pewlett_hackard;
SET search_path TO pewlett_hackard;

CREATE TABLE Departments (
    dept_no CHARACTER(4),
    dept_name VARCHAR
);

CREATE TABLE Department_Employees (
    emp_no INT PRIMARY KEY,
    dept_no CHARACTER(4)
);

CREATE TABLE editing_department_employees (
    emp_no INT,
    dept_no CHARACTER(4)
);

CREATE TABLE Department_Managers (
    dept_no CHARACTER(4),
    emp_no INT
);

CREATE TABLE Employee_Info (
    emp_no INT PRIMARY KEY,
    emp_title_id CHARACTER(5),
	birth_date VARCHAR,
	first_name VARCHAR,
	last_name VARCHAR,
	sex CHARACTER,
	hire_date VARCHAR
);

CREATE TABLE Salaries (
    emp_no INT PRIMARY KEY,
    salary INT
);

CREATE TABLE Titles (
    title_id CHARACTER(5),
    title VARCHAR
);

DELETE FROM Departments
WHERE dept_no IS NULL OR dept_name IS NULL;

DELETE FROM Department_Employees
WHERE emp_no IS NULL OR dept_no IS NULL;

DELETE FROM Department_Managers
WHERE dept_no IS NULL OR emp_no IS NULL;

DELETE FROM Employee_Info
WHERE emp_no IS NULL OR emp_title_id IS NULL OR birth_date IS NULL OR first_name IS NULL OR last_name IS NULL OR sex IS NULL OR hire_date IS NULL;

DELETE FROM Salaries
WHERE emp_no IS NULL OR salary IS NULL;

DELETE FROM Titles
WHERE title_id IS NULL OR title IS NULL;

COPY Departments(dept_no, dept_name)
FROM '/Users/Kyle_McDaniel_Python/Desktop/Columbia_Analytics_Bootcamp/sql_challenge/data/departments.csv' 
DELIMITER ',' 
CSV HEADER;

COPY editing_department_employees(emp_no, dept_no)
FROM '/Users/Kyle_McDaniel_Python/Desktop/Columbia_Analytics_Bootcamp/sql_challenge/data/dept_emp.csv' 
DELIMITER ',' 
CSV HEADER;

INSERT INTO Department_Employees(emp_no, dept_no)
SELECT DISTINCT emp_no, dept_no FROM editing_department_employees
ON CONFLICT (emp_no) DO NOTHING;

DROP TABLE editing_department_employees;

COPY Department_Managers(dept_no, emp_no)
FROM '/Users/Kyle_McDaniel_Python/Desktop/Columbia_Analytics_Bootcamp/sql_challenge/data/dept_manager.csv' 
DELIMITER ',' 
CSV HEADER;

COPY Employee_Info(emp_no, emp_title_id, birth_date, first_name, last_name, sex, hire_date)
FROM '/Users/Kyle_McDaniel_Python/Desktop/Columbia_Analytics_Bootcamp/sql_challenge/data/employees.csv' 
DELIMITER ',' 
CSV HEADER;

COPY Salaries(emp_no, salary)
FROM '/Users/Kyle_McDaniel_Python/Desktop/Columbia_Analytics_Bootcamp/sql_challenge/data/salaries.csv' 
DELIMITER ',' 
CSV HEADER;

COPY Titles(title_id, title)
FROM '/Users/Kyle_McDaniel_Python/Desktop/Columbia_Analytics_Bootcamp/sql_challenge/data/titles.csv' 
DELIMITER ',' 
CSV HEADER;

CREATE TABLE employee_salaries (
    emp_no INT,
    last_name VARCHAR,
    first_name VARCHAR,
    sex CHAR,
    salary INT,
    CONSTRAINT fk_employee_info
        FOREIGN KEY (emp_no) 
        REFERENCES Employee_Info(emp_no),
    CONSTRAINT fk_salaries
        FOREIGN KEY (emp_no) 
        REFERENCES Salaries(emp_no)
);

INSERT INTO employee_salaries (emp_no, last_name, first_name, sex, salary)
SELECT ei.emp_no, ei.last_name, ei.first_name, ei.sex, s.salary
FROM Employee_Info AS ei
JOIN Salaries AS s ON ei.emp_no = s.emp_no;

CREATE TABLE hire_1986 AS
SELECT first_name, last_name, hire_date
FROM Employee_Info
WHERE hire_date LIKE '%1986';

ALTER TABLE Departments
ADD CONSTRAINT unique_dept_no UNIQUE (dept_no);

CREATE TABLE manager_info (
    dept_no CHARACTER(4),
    dept_name VARCHAR,
	emp_no INT,
	last_name VARCHAR,
	first_name VARCHAR,
    -- Add foreign key constraints
    CONSTRAINT fk_dept_no
        FOREIGN KEY (dept_no) 
        REFERENCES Departments(dept_no),
    CONSTRAINT fk_emp_no
        FOREIGN KEY (emp_no) 
        REFERENCES Employee_Info(emp_no)
	--CONSTRAINT fk_dept_man
		--FOREIGN KEY (dept_no)
		--REFERENCES Department_Managers(dept_no)
		--FIX VARIBLE CHARACTER TYPES. I THINK IT HELPS ENSURE DATA INTEGRITY.
);

INSERT INTO manager_info (dept_no, dept_name, emp_no, last_name, first_name)
SELECT dm.dept_no, d.dept_name, ei.emp_no, ei.last_name, ei.first_name
FROM Department_Managers AS dm
JOIN Departments AS d ON d.dept_no = dm.dept_no
JOIN Employee_Info AS ei ON ei.emp_no = dm.emp_no;

CREATE TABLE employee_department_details (
    dept_no CHARACTER (4),
	emp_no INT,
    last_name VARCHAR,
    first_name VARCHAR,
    dept_name VARCHAR,
    CONSTRAINT fk_department_employees
        FOREIGN KEY (emp_no) 
        REFERENCES Employee_Info(emp_no),
    CONSTRAINT fk_dept_no
        FOREIGN KEY (dept_no) 
        REFERENCES departments(dept_no)
);

INSERT INTO employee_department_details (dept_no, emp_no, last_name, first_name, dept_name)
SELECT de.dept_no, ei.emp_no, ei.last_name, ei.first_name, d.dept_name
FROM Employee_Info AS ei
JOIN department_employees AS de ON de.emp_no = ei.emp_no
JOIN Departments AS d ON d.dept_no = de.dept_no;

CREATE TABLE greek_gods (
	first_name VARCHAR,
	last_name VARCHAR,
	sex CHARACTER
);

INSERT INTO greek_gods (first_name, last_name, sex)
SELECT first_name, last_name, sex
FROM Employee_Info
WHERE first_name = 'Hercules' AND last_name LIKE 'B%';

CREATE TABLE sales_department (
	emp_no INT,
	last_name VARCHAR,
	first_name VARCHAR
);

INSERT INTO sales_department (emp_no, last_name, first_name)
SELECT emp_no, last_name, first_name
FROM Employee_Department_Details
WHERE dept_name = 'Sales';

CREATE TABLE sales_dev_departments (
	emp_no INT,
	last_name VARCHAR,
	first_name VARCHAR,
	dept_name VARCHAR
);

INSERT INTO sales_dev_departments (emp_no, last_name, first_name, dept_name)
SELECT emp_no, last_name, first_name, dept_name
FROM Employee_Department_Details
WHERE dept_name = 'Sales' OR dept_name = 'Development';

CREATE TABLE last_name_counts (
	last_name VARCHAR,
	counts INT
);

INSERT INTO last_name_counts (last_name, counts)
SELECT last_name, COUNT(*) AS counts
FROM Employee_Info
GROUP BY last_name
ORDER BY counts DESC;