=========================================
PROBLEM STATEMENT 1: BANK SCHEMA
=========================================
CREATE DATABASE IF NOT EXISTS bank_practical;
USE bank_practical;

-- Table Creation
CREATE TABLE Customer (cust_id INT PRIMARY KEY, cust_name VARCHAR(50), city VARCHAR(50), phone VARCHAR(15));
CREATE TABLE Branch (branch_id INT PRIMARY KEY, branch_name VARCHAR(50), branch_city VARCHAR(50));
CREATE TABLE Account (acc_no INT PRIMARY KEY, cust_id INT, branch_id INT, balance DECIMAL(10,2), FOREIGN KEY (cust_id) REFERENCES Customer(cust_id) ON DELETE CASCADE, FOREIGN KEY (branch_id) REFERENCES Branch(branch_id) ON DELETE CASCADE);
CREATE TABLE Loan (loan_no INT PRIMARY KEY, cust_id INT, branch_id INT, amount DECIMAL(10,2), FOREIGN KEY (cust_id) REFERENCES Customer(cust_id) ON DELETE CASCADE, FOREIGN KEY (branch_id) REFERENCES Branch(branch_id) ON DELETE CASCADE);

-- 1. Display all customers in alphabetical order by name
SELECT * FROM Customer ORDER BY cust_name ASC;

-- 2. Display names and cities of customers starting with 'P'
SELECT cust_name, city FROM Customer WHERE cust_name LIKE 'P%';

-- 3. Display customers in Pune/Mumbai who have a phone number
SELECT * FROM Customer WHERE city IN ('Pune', 'Mumbai') AND phone IS NOT NULL;

-- 4. Display customers missing a phone number
SELECT * FROM Customer WHERE phone IS NULL;

-- 5. Display average account balance per branch
SELECT branch_id, AVG(balance) AS avg_balance FROM Account GROUP BY branch_id;

-- 6. Display branch average balance only if it is greater than 40000
SELECT branch_id, AVG(balance) AS avg_balance FROM Account GROUP BY branch_id HAVING AVG(balance) > 40000;

-- 7. Display balance with a 10% interest addition (without modifying real data)
SELECT acc_no, balance, (balance + (balance * 0.10)) AS balance_with_interest FROM Account;

-- 8. Update a specific customer's phone number
UPDATE Customer SET phone = '9999999999' WHERE cust_name = 'Neha';

-- 9. Delete all customers from Pune (Cascades to accounts/loans)
DELETE FROM Customer WHERE city = 'Pune';

-- 10. Display customers with EITHER an account OR a loan using UNION
SELECT Customer.cust_name FROM Customer INNER JOIN Account ON Customer.cust_id = Account.cust_id 
UNION 
SELECT Customer.cust_name FROM Customer INNER JOIN Loan ON Customer.cust_id = Loan.cust_id;


=========================================
PROBLEM STATEMENT 2: STUDENT SCHEMA
=========================================
CREATE DATABASE IF NOT EXISTS student_practical;
USE student_practical;

-- Table Creation
CREATE TABLE Students (student_id INT PRIMARY KEY, name VARCHAR(50), city VARCHAR(50));
CREATE TABLE Enrollments (enroll_id INT PRIMARY KEY, student_id INT, course VARCHAR(50), marks INT, FOREIGN KEY (student_id) REFERENCES Students(student_id) ON DELETE CASCADE);

-- 1. Display students, courses, and marks (Only enrolled students)
SELECT s.name, e.course, e.marks FROM Students s INNER JOIN Enrollments e ON s.student_id = e.student_id;

-- 2. Display all students (INCLUDING those with no courses using LEFT JOIN)
SELECT s.name, e.course, e.marks FROM Students s LEFT JOIN Enrollments e ON s.student_id = e.student_id;

-- 3. Display enrollments with the matching student name
SELECT e.enroll_id, e.course, e.marks, s.name FROM Enrollments e INNER JOIN Students s ON e.student_id = s.student_id;

-- 4. Display all possible combinations of students and courses (CROSS JOIN)
SELECT s.name, distinct_courses.course FROM Students s CROSS JOIN (SELECT DISTINCT course FROM Enrollments) AS distinct_courses;

-- 5. Display students with marks higher than the overall average
SELECT s.name, e.marks FROM Students s INNER JOIN Enrollments e ON s.student_id = e.student_id WHERE e.marks > (SELECT AVG(marks) FROM Enrollments);

-- 6. Display students who scored above 80
SELECT s.name, e.marks FROM Students s INNER JOIN Enrollments e ON s.student_id = e.student_id WHERE e.marks > 80;

-- 7. Display students enrolled specifically in DBMS
SELECT s.name FROM Students s INNER JOIN Enrollments e ON s.student_id = e.student_id WHERE e.course = 'DBMS';

-- 8. Display student ID of the highest scorer
SELECT student_id FROM Enrollments WHERE marks = (SELECT MAX(marks) FROM Enrollments);

-- 9a. Create a View for student names, courses, and marks
CREATE VIEW student_course_view AS SELECT s.student_id, s.name, e.course, e.marks FROM Students s INNER JOIN Enrollments e ON s.student_id = e.student_id;
-- 9b. Display the View
SELECT * FROM student_course_view;
-- 9c. Update data through the View
UPDATE student_course_view SET marks = 95 WHERE name = 'Aditya' AND course = 'DBMS';

-- 10. Create index on name and a unique index on enroll_id
CREATE INDEX idx_student_name ON Students(name);
CREATE UNIQUE INDEX idx_unique_enroll ON Enrollments(enroll_id);


=========================================
PROBLEM STATEMENT 3: STUDENT QUERIES (REUSES PS 2 TABLES)
=========================================
USE student_practical;

-- 1. Display student names and their enrolled courses
SELECT s.name, e.course FROM Students s INNER JOIN Enrollments e ON s.student_id = e.student_id;

-- 2. Display all students and courses (INCLUDING not enrolled)
SELECT s.name, e.course, e.marks FROM Students s LEFT JOIN Enrollments e ON s.student_id = e.student_id;

-- 3. Display students in DBMS with their marks
SELECT s.name, e.marks FROM Students s INNER JOIN Enrollments e ON s.student_id = e.student_id WHERE e.course = 'DBMS';

-- 4. Display orphan enrollments (Returns empty because FK prevents them)
SELECT e.* FROM Enrollments e LEFT JOIN Students s ON e.student_id = s.student_id WHERE s.student_id IS NULL;

-- 5. Display students with marks BELOW the overall average
SELECT s.name, e.marks FROM Students s INNER JOIN Enrollments e ON s.student_id = e.student_id WHERE e.marks < (SELECT AVG(marks) FROM Enrollments);

-- 6. Display students with the exact highest marks
SELECT s.name, e.marks FROM Students s INNER JOIN Enrollments e ON s.student_id = e.student_id WHERE e.marks = (SELECT MAX(marks) FROM Enrollments);

-- 7. Display students from Pune
SELECT * FROM Students WHERE city = 'Pune';

-- 8. Display students enrolled in Python
SELECT s.name FROM Students s INNER JOIN Enrollments e ON s.student_id = e.student_id WHERE e.course = 'Python';

-- 9a. Create View showing average marks per course
CREATE VIEW course_average_view AS SELECT course, AVG(marks) AS avg_marks FROM Enrollments GROUP BY course;
-- 9b. Display the View
SELECT * FROM course_average_view;

-- 10. Create index on city and unique index on student_id
CREATE INDEX idx_student_city ON Students(city);
CREATE UNIQUE INDEX idx_unique_student_id ON Students(student_id);

=========================================
DATABASE SETUP (FOR PS 10 & PS 11)
=========================================
CREATE DATABASE IF NOT EXISTS advanced_bank;
USE advanced_bank;

-- 1. Create Tables with Constraints
CREATE TABLE Branch (
    branch_name VARCHAR(50) PRIMARY KEY,
    branch_city VARCHAR(50),
    assets DECIMAL(15,2)
);

CREATE TABLE Customer (
    cust_name VARCHAR(50) PRIMARY KEY,
    cust_street VARCHAR(50),
    cust_city VARCHAR(50) NOT NULL
);

CREATE TABLE Account (
    acc_no INT PRIMARY KEY,
    branch_name VARCHAR(50),
    balance DECIMAL(10,2),
    FOREIGN KEY (branch_name) REFERENCES Branch(branch_name) ON DELETE CASCADE
);

CREATE TABLE Loan (
    loan_no INT PRIMARY KEY,
    branch_name VARCHAR(50),
    amount DECIMAL(15,2),
    FOREIGN KEY (branch_name) REFERENCES Branch(branch_name) ON DELETE CASCADE
);

CREATE TABLE Depositor (
    cust_name VARCHAR(50),
    acc_no INT,
    PRIMARY KEY (cust_name, acc_no),
    FOREIGN KEY (cust_name) REFERENCES Customer(cust_name) ON DELETE CASCADE,
    FOREIGN KEY (acc_no) REFERENCES Account(acc_no) ON DELETE CASCADE
);

CREATE TABLE Borrower (
    cust_name VARCHAR(50),
    loan_no INT,
    PRIMARY KEY (cust_name, loan_no),
    FOREIGN KEY (cust_name) REFERENCES Customer(cust_name) ON DELETE CASCADE,
    FOREIGN KEY (loan_no) REFERENCES Loan(loan_no) ON DELETE CASCADE
);

-- Insert Sample Data (Tailored for Akurdi, Chinchwad, and specific query limits)
INSERT INTO Branch VALUES 
('Akurdi', 'Pune', 10000000), ('Chinchwad', 'Pune', 8500000), ('Shivajinagar', 'Pune', 9500000);

INSERT INTO Customer VALUES 
('Aditya', 'Main Road', 'Pune'), ('Amit', NULL, 'Mumbai'), ('Bhavna', 'FC Road', 'Pune'), 
('Chetan', NULL, 'Nashik'), ('Deepa', 'MG Road', 'Pune');

INSERT INTO Account VALUES 
(101, 'Akurdi', 45000), (102, 'Chinchwad', 15000), (103, 'Akurdi', 11000), (104, 'Shivajinagar', 60000);

INSERT INTO Loan VALUES 
(201, 'Akurdi', 15000), (202, 'Chinchwad', 600000), (203, 'Shivajinagar', 1400), (204, 'Akurdi', 550000);

INSERT INTO Depositor VALUES 
('Aditya', 101), ('Amit', 102), ('Bhavna', 103), ('Chetan', 104);

INSERT INTO Borrower VALUES 
('Aditya', 201), ('Amit', 202), ('Deepa', 203), ('Aditya', 204);


=========================================
PROBLEM STATEMENT 10: QUERIES
=========================================
-- 2. Add customer phone number in Customer table
ALTER TABLE Customer ADD phone VARCHAR(15);

-- 3. Delete phone number attribute from Customer table
ALTER TABLE Customer DROP COLUMN phone;

-- 4. Find the names of all branches in loan relation
SELECT DISTINCT branch_name FROM Loan;

-- 5. Find customers with a loan, showing names, loan_no, and amount
SELECT b.cust_name, l.loan_no, l.amount 
FROM Borrower b 
INNER JOIN Loan l ON b.loan_no = l.loan_no;

-- 6. List all customers alphabetically who have a loan from Akurdi branch
SELECT b.cust_name 
FROM Borrower b 
INNER JOIN Loan l ON b.loan_no = l.loan_no 
WHERE l.branch_name = 'Akurdi' 
ORDER BY b.cust_name ASC;

-- 7. Find average account balance at Chinchwad branch
SELECT AVG(balance) AS avg_chinchwad_balance FROM Account WHERE branch_name = 'Chinchwad';

-- 8. Find no. of depositors at each branch
SELECT a.branch_name, COUNT(d.cust_name) AS no_of_depositors 
FROM Depositor d 
INNER JOIN Account a ON d.acc_no = a.acc_no 
GROUP BY a.branch_name;

-- 9. Find the total loan amount for each branch
SELECT branch_name, SUM(amount) AS total_loan FROM Loan GROUP BY branch_name;

-- 10. Find the branches where the average loan amount is greater than 5,00,000
SELECT branch_name 
FROM Loan 
GROUP BY branch_name 
HAVING AVG(amount) > 500000;


=========================================
PROBLEM STATEMENT 11: QUERIES
=========================================
-- 1. Modify the assets attribute of the Branch table to Property
ALTER TABLE Branch CHANGE assets Property DECIMAL(15,2);

-- 2. Find loan numbers at Akurdi branch with amount > 12,000
SELECT loan_no FROM Loan WHERE branch_name = 'Akurdi' AND amount > 12000;

-- 3. Find the average account balance at each branch
SELECT branch_name, AVG(balance) AS avg_balance FROM Account GROUP BY branch_name;

-- 4. Find branches where average account balance is > 12,000
SELECT branch_name FROM Account GROUP BY branch_name HAVING AVG(balance) > 12000;

-- 5. Find the number of tuples (rows) in the Customer relation
SELECT COUNT(*) AS total_customers FROM Customer;

-- 6. Calculate the total loan amount given by the bank
SELECT SUM(amount) AS total_bank_loans FROM Loan;

-- 7. Delete all loans with an amount between 1,300 and 1,500
DELETE FROM Loan WHERE amount BETWEEN 1300 AND 1500;

-- 8. Find the customers whose street address is not available
SELECT cust_name FROM Customer WHERE cust_street IS NULL;

-- 9. Find the branches where the branch city is available
SELECT branch_name FROM Branch WHERE branch_city IS NOT NULL;

-- 10. Find the names of customers whose names start with the letter 'A'
SELECT cust_name FROM Customer WHERE cust_name LIKE 'A%';

=========================================
PROBLEM STATEMENT 12: MONGODB (ICEM / EMPLOYEE)
=========================================
// 1. Create DB, Collection, and Insert Data
use ICEM;
db.employee.insertMany([
    {empid: 1, ename: "Aditya", dept: "Computer", salary: 45000},
    {empid: 2, ename: "Bhavna", dept: "IT", salary: 25000},
    {empid: 3, ename: "Chetan", dept: "Mechanical", salary: 35000},
    {empid: 4, ename: "Deepa", dept: "Computer", salary: 50000},
    {empid: 5, ename: "Eshan", dept: "Civil", salary: 28000}
]);

// 1. Display all employees
db.employee.find().pretty();

// 2. Display employees in Computer department
db.employee.find({dept: "Computer"}).pretty();

// 3. Display employees in Computer OR IT using logical operator
db.employee.find({$or: [{dept: "Computer"}, {dept: "IT"}]}).pretty();

// 4. Display employees whose salary is > 30000
db.employee.find({salary: {$gt: 30000}}).pretty();

// 5. Display employees with salary >= 40000 AND work in Computer
db.employee.find({$and: [{salary: {$gte: 40000}}, {dept: "Computer"}]}).pretty();

// 6. Create index on ename, then drop it
db.employee.createIndex({ename: 1});
db.employee.dropIndex("ename_1");

// 7. Display employees whose salary is between 20000 and 40000
db.employee.find({salary: {$gte: 20000, $lte: 40000}}).pretty();

// 8. Display employees whose department is NOT IT
db.employee.find({dept: {$ne: "IT"}}).pretty();

// 9. Display the total salary paid to employees in each department
db.employee.aggregate([ {$group: {_id: "$dept", total_salary: {$sum: "$salary"}}} ]);

// 10. Display the average salary of employees in each department
db.employee.aggregate([ {$group: {_id: "$dept", avg_salary: {$avg: "$salary"}}} ]);


=========================================
PROBLEM STATEMENT 13: MONGODB (STUDENT)
=========================================
// 1. Create DB, Collection, and Insert Data
use Student;
db.student.insertMany([
    {roll_no: 1, sname: "Aditya", dept: "Computer", marks: 85},
    {roll_no: 2, sname: "Bhavna", dept: "IT", marks: 35},
    {roll_no: 3, sname: "Chetan", dept: "Mechanical", marks: 60},
    {roll_no: 4, sname: "Deepa", dept: "Computer", marks: 92},
    {roll_no: 5, sname: "Eshan", dept: "IT", marks: 75}
]);

// 1. Display all students
db.student.find().pretty();

// 2. Display students in Computer department
db.student.find({dept: "Computer"}).pretty();

// 3. Display students in Computer OR IT
db.student.find({$or: [{dept: "Computer"}, {dept: "IT"}]}).pretty();

// 4. Display students with marks > 70
db.student.find({marks: {$gt: 70}}).pretty();

// 5. Display students with marks >= 80 AND belong to Computer
db.student.find({$and: [{marks: {$gte: 80}}, {dept: "Computer"}]}).pretty();

// 6. Display students whose marks < 40 OR belong to IT
db.student.find({$or: [{marks: {$lt: 40}}, {dept: "IT"}]}).pretty();

// 7. Display students with marks between 40 and 80
db.student.find({marks: {$gte: 40, $lte: 80}}).pretty();

// 8. Display students whose department is NOT IT
db.student.find({dept: {$ne: "IT"}}).pretty();

// 9. Display total marks of students in each department
db.student.aggregate([ {$group: {_id: "$dept", total_marks: {$sum: "$marks"}}} ]);

// 10. Create index on sname, then drop it
db.student.createIndex({sname: 1});
db.student.dropIndex("sname_1");

=========================================
PROBLEM STATEMENT 14: ORACLE SET OPERATIONS
=========================================
-- 1. Create Tables and Insert Data (Oracle Syntax)
CREATE TABLE Customer (
    cust_id NUMBER PRIMARY KEY,
    cust_name VARCHAR2(50),
    city VARCHAR2(50),
    phone VARCHAR2(15)
);

CREATE TABLE Branch (
    branch_id NUMBER PRIMARY KEY,
    branch_name VARCHAR2(50),
    branch_city VARCHAR2(50)
);

CREATE TABLE Account (
    acc_no NUMBER PRIMARY KEY,
    cust_id NUMBER REFERENCES Customer(cust_id),
    branch_id NUMBER REFERENCES Branch(branch_id),
    balance NUMBER
);

CREATE TABLE Loan (
    loan_no NUMBER PRIMARY KEY,
    cust_id NUMBER REFERENCES Customer(cust_id),
    branch_id NUMBER REFERENCES Branch(branch_id),
    amount NUMBER
);

INSERT INTO Customer VALUES (1, 'Aditya', 'Pune', '9000000001');
INSERT INTO Customer VALUES (2, 'Priya', 'Mumbai', NULL);
INSERT INTO Customer VALUES (3, 'Rahul', 'Delhi', '9000000003');
INSERT INTO Customer VALUES (4, 'Neha', 'Pune', '9000000004');

INSERT INTO Branch VALUES (101, 'Deccan', 'Pune');
INSERT INTO Branch VALUES (102, 'Bandra', 'Mumbai');

-- Aditya has an account AND a loan
INSERT INTO Account VALUES (2001, 1, 101, 45000);
INSERT INTO Loan VALUES (5001, 1, 101, 150000);

-- Priya has ONLY an account (No loan)
INSERT INTO Account VALUES (2005, 2, 102, 35000);

COMMIT;

-- 1. Display customers in Pune or Mumbai, alphabetically by name
SELECT * FROM Customer WHERE city IN ('Pune', 'Mumbai') ORDER BY cust_name ASC;

-- 2. Display names of customers whose names contain the letter 'a' (case-insensitive in Oracle: %a% or %A%)
SELECT cust_name FROM Customer WHERE LOWER(cust_name) LIKE '%a%';

-- 3. Display all customers whose phone number is NOT NULL
SELECT * FROM Customer WHERE phone IS NOT NULL;

-- 4. Display customers in Pune starting with 'P', OR customers in Mumbai
SELECT * FROM Customer WHERE (city = 'Pune' AND cust_name LIKE 'P%') OR city = 'Mumbai';

-- 5. Display maximum and average loan amount per branch
SELECT branch_id, MAX(amount) AS max_loan, AVG(amount) AS avg_loan FROM Loan GROUP BY branch_id;

-- 6. Display branches whose total loan amount is > 1,00,000
SELECT branch_id FROM Loan GROUP BY branch_id HAVING SUM(amount) > 100000;

-- 7. Display each account number with balance after adding 500
SELECT acc_no, balance, (balance + 500) AS new_balance FROM Account;

-- 8. Change the branch of account 2005 to a different branch (101)
UPDATE Account SET branch_id = 101 WHERE acc_no = 2005;
COMMIT;

-- 9. Delete customers whose phone number is NULL (Fails if they have accounts without CASCADE, but this is the exact query)
DELETE FROM Customer WHERE phone IS NULL;

-- 10a. Customers who have BOTH an account and a loan (INTERSECT)
SELECT cust_id FROM Account
INTERSECT
SELECT cust_id FROM Loan;

-- 10b. Customers who have an account BUT NO loan (MINUS)
SELECT cust_id FROM Account
MINUS
SELECT cust_id FROM Loan;
=========================================
PROBLEM STATEMENT 1: BANK SCHEMA (MYSQL)
=========================================
CREATE DATABASE IF NOT EXISTS bank_practical;
USE bank_practical;

CREATE TABLE Customer (cust_id INT PRIMARY KEY, cust_name VARCHAR(50), city VARCHAR(50), phone VARCHAR(15));
CREATE TABLE Branch (branch_id INT PRIMARY KEY, branch_name VARCHAR(50), branch_city VARCHAR(50));
CREATE TABLE Account (acc_no INT PRIMARY KEY, cust_id INT, branch_id INT, balance DECIMAL(10,2), FOREIGN KEY (cust_id) REFERENCES Customer(cust_id) ON DELETE CASCADE, FOREIGN KEY (branch_id) REFERENCES Branch(branch_id) ON DELETE CASCADE);
CREATE TABLE Loan (loan_no INT PRIMARY KEY, cust_id INT, branch_id INT, amount DECIMAL(10,2), FOREIGN KEY (cust_id) REFERENCES Customer(cust_id) ON DELETE CASCADE, FOREIGN KEY (branch_id) REFERENCES Branch(branch_id) ON DELETE CASCADE);

SELECT * FROM Customer ORDER BY cust_name ASC;
SELECT cust_name, city FROM Customer WHERE cust_name LIKE 'P%';
SELECT * FROM Customer WHERE city IN ('Pune', 'Mumbai') AND phone IS NOT NULL;
SELECT * FROM Customer WHERE phone IS NULL;
SELECT branch_id, AVG(balance) AS avg_balance FROM Account GROUP BY branch_id;
SELECT branch_id, AVG(balance) AS avg_balance FROM Account GROUP BY branch_id HAVING AVG(balance) > 40000;
SELECT acc_no, balance, (balance + (balance * 0.10)) AS balance_with_interest FROM Account;
UPDATE Customer SET phone = '9999999999' WHERE cust_name = 'Neha';
DELETE FROM Customer WHERE city = 'Pune';
SELECT Customer.cust_name FROM Customer INNER JOIN Account ON Customer.cust_id = Account.cust_id UNION SELECT Customer.cust_name FROM Customer INNER JOIN Loan ON Customer.cust_id = Loan.cust_id;


=========================================
PROBLEM STATEMENT 2 & 3: STUDENT SCHEMA (MYSQL)
=========================================
CREATE DATABASE IF NOT EXISTS student_practical;
USE student_practical;

CREATE TABLE Students (student_id INT PRIMARY KEY, name VARCHAR(50), city VARCHAR(50));
CREATE TABLE Enrollments (enroll_id INT PRIMARY KEY, student_id INT, course VARCHAR(50), marks INT, FOREIGN KEY (student_id) REFERENCES Students(student_id) ON DELETE CASCADE);

-- PS2 Queries
SELECT s.name, e.course, e.marks FROM Students s INNER JOIN Enrollments e ON s.student_id = e.student_id;
SELECT s.name, e.course, e.marks FROM Students s LEFT JOIN Enrollments e ON s.student_id = e.student_id;
SELECT e.enroll_id, e.course, e.marks, s.name FROM Enrollments e INNER JOIN Students s ON e.student_id = s.student_id;
SELECT s.name, distinct_courses.course FROM Students s CROSS JOIN (SELECT DISTINCT course FROM Enrollments) AS distinct_courses;
SELECT s.name, e.marks FROM Students s INNER JOIN Enrollments e ON s.student_id = e.student_id WHERE e.marks > (SELECT AVG(marks) FROM Enrollments);
SELECT s.name, e.marks FROM Students s INNER JOIN Enrollments e ON s.student_id = e.student_id WHERE e.marks > 80;
SELECT s.name FROM Students s INNER JOIN Enrollments e ON s.student_id = e.student_id WHERE e.course = 'DBMS';
SELECT student_id FROM Enrollments WHERE marks = (SELECT MAX(marks) FROM Enrollments);
CREATE VIEW student_course_view AS SELECT s.student_id, s.name, e.course, e.marks FROM Students s INNER JOIN Enrollments e ON s.student_id = e.student_id;
SELECT * FROM student_course_view;
UPDATE student_course_view SET marks = 95 WHERE name = 'Aditya' AND course = 'DBMS';
CREATE INDEX idx_student_name ON Students(name);
CREATE UNIQUE INDEX idx_unique_enroll ON Enrollments(enroll_id);

-- PS3 Queries
SELECT s.name, e.course FROM Students s INNER JOIN Enrollments e ON s.student_id = e.student_id;
SELECT s.name, e.course, e.marks FROM Students s LEFT JOIN Enrollments e ON s.student_id = e.student_id;
SELECT s.name, e.marks FROM Students s INNER JOIN Enrollments e ON s.student_id = e.student_id WHERE e.course = 'DBMS';
SELECT e.* FROM Enrollments e LEFT JOIN Students s ON e.student_id = s.student_id WHERE s.student_id IS NULL;
SELECT s.name, e.marks FROM Students s INNER JOIN Enrollments e ON s.student_id = e.student_id WHERE e.marks < (SELECT AVG(marks) FROM Enrollments);
SELECT s.name, e.marks FROM Students s INNER JOIN Enrollments e ON s.student_id = e.student_id WHERE e.marks = (SELECT MAX(marks) FROM Enrollments);
SELECT * FROM Students WHERE city = 'Pune';
SELECT s.name FROM Students s INNER JOIN Enrollments e ON s.student_id = e.student_id WHERE e.course = 'Python';
CREATE VIEW course_average_view AS SELECT course, AVG(marks) AS avg_marks FROM Enrollments GROUP BY course;
SELECT * FROM course_average_view;
CREATE INDEX idx_student_city ON Students(city);
CREATE UNIQUE INDEX idx_unique_student_id ON Students(student_id);


=========================================
PROBLEM STATEMENT 4: STORED PROCEDURE (MYSQL)
=========================================
CREATE TABLE Stud_Marks (rollno INT, name VARCHAR(30), total_marks INT);
CREATE TABLE Result (Roll INT, Name VARCHAR(30), Class VARCHAR(30));

DELIMITER //
CREATE PROCEDURE proc_Grade(IN p_roll INT, IN p_name VARCHAR(30))
BEGIN
    DECLARE v_marks INT;
    DECLARE v_class VARCHAR(30);
    SELECT total_marks INTO v_marks FROM Stud_Marks WHERE name = p_name;
    
    IF v_marks >= 990 AND v_marks <= 1500 THEN SET v_class = 'Distinction';
    ELSEIF v_marks >= 900 AND v_marks <= 989 THEN SET v_class = 'First Class';
    ELSEIF v_marks >= 825 AND v_marks <= 899 THEN SET v_class = 'Higher Second Class';
    ELSE SET v_class = 'Not Categorised';
    END IF;
    
    INSERT INTO Result (Roll, Name, Class) VALUES (p_roll, p_name, v_class);
END //
DELIMITER ;
-- RUN: CALL proc_Grade(1, 'Aditya');


=========================================
PROBLEM STATEMENT 5: PL/SQL LOOP (ORACLE)
=========================================
-- Run in Oracle
CREATE TABLE areas (radius NUMBER, area NUMBER(10,2));

DECLARE
    v_area NUMBER(10,2);
    v_radius NUMBER;
BEGIN
    FOR v_radius IN 5..9 LOOP
        v_area := 3.14159 * v_radius * v_radius;
        INSERT INTO areas VALUES (v_radius, v_area);
    END LOOP;
END;
/


=========================================
PROBLEM STATEMENT 6: PL/SQL EXPLICIT CURSOR (ORACLE)
=========================================
-- Run in Oracle
CREATE TABLE Employee (Emp_ID NUMBER(5) PRIMARY KEY, Emp_Name VARCHAR2(30), Department VARCHAR2(20), Salary NUMBER(10,2));

SET SERVEROUTPUT ON;
DECLARE
    CURSOR emp_cur IS SELECT Emp_ID, Emp_Name, Department, Salary FROM Employee WHERE Salary > 30000;
    v_id Employee.Emp_ID%TYPE;
    v_name Employee.Emp_Name%TYPE;
    v_dept Employee.Department%TYPE;
    v_sal Employee.Salary%TYPE;
BEGIN
    OPEN emp_cur;
    LOOP
        FETCH emp_cur INTO v_id, v_name, v_dept, v_sal;
        EXIT WHEN emp_cur%NOTFOUND;
        DBMS_OUTPUT.PUT_LINE(v_id || ' | ' || v_name || ' | ' || v_sal);
    END LOOP;
    CLOSE emp_cur;
END;
/


=========================================
PROBLEM STATEMENT 7: DATABASE TRIGGER (MYSQL)
=========================================
CREATE TABLE Library (book_id INT PRIMARY KEY, book_name VARCHAR(50), author VARCHAR(50), category VARCHAR(30), date DATE, operation VARCHAR(20));
CREATE TABLE Library_Audit (book_id INT, old_book_name VARCHAR(50), old_author VARCHAR(50), category VARCHAR(30), date DATE, operation VARCHAR(20));

DELIMITER //
CREATE TRIGGER before_book_update BEFORE UPDATE ON Library FOR EACH ROW
BEGIN
    INSERT INTO Library_Audit (book_id, old_book_name, old_author, category, date, operation)
    VALUES (OLD.book_id, OLD.book_name, OLD.author, OLD.category, CURDATE(), 'UPDATED');
END //

CREATE TRIGGER before_book_delete BEFORE DELETE ON Library FOR EACH ROW
BEGIN
    INSERT INTO Library_Audit (book_id, old_book_name, old_author, category, date, operation)
    VALUES (OLD.book_id, OLD.book_name, OLD.author, OLD.category, CURDATE(), 'DELETED');
END //
DELIMITER ;


=========================================
PROBLEM STATEMENT 8: PYTHON TO MYSQL
=========================================
-- Create a file named mysql_connect.py
import mysql.connector
db = mysql.connector.connect(host="localhost", user="root", password="", database="student_practical")
cursor = db.cursor()

# Add Record
cursor.execute("INSERT INTO Students (student_id, name, city) VALUES (%s, %s, %s)", (101, "New Student", "Pune"))
db.commit()

# View Records
cursor.execute("SELECT * FROM Students")
for row in cursor.fetchall(): print(row)


=========================================
PROBLEM STATEMENT 9: PYTHON TO MONGODB
=========================================
-- Create a file named mongo_connect.py
import pymongo
client = pymongo.MongoClient("mongodb://localhost:27017/")
db = client["Student"]
col = db["student"]

# Add Record
col.insert_one({"roll_no": 101, "sname": "New Student", "dept": "Computer", "marks": 95})

# View Records
for x in col.find(): print(x)


=========================================
PROBLEM STATEMENT 10 & 11: ADVANCED BANK (MYSQL)
=========================================
CREATE DATABASE IF NOT EXISTS advanced_bank;
USE advanced_bank;

CREATE TABLE Branch (branch_name VARCHAR(50) PRIMARY KEY, branch_city VARCHAR(50), assets DECIMAL(15,2));
CREATE TABLE Customer (cust_name VARCHAR(50) PRIMARY KEY, cust_street VARCHAR(50), cust_city VARCHAR(50) NOT NULL);
CREATE TABLE Account (acc_no INT PRIMARY KEY, branch_name VARCHAR(50), balance DECIMAL(10,2), FOREIGN KEY (branch_name) REFERENCES Branch(branch_name) ON DELETE CASCADE);
CREATE TABLE Loan (loan_no INT PRIMARY KEY, branch_name VARCHAR(50), amount DECIMAL(15,2), FOREIGN KEY (branch_name) REFERENCES Branch(branch_name) ON DELETE CASCADE);
CREATE TABLE Depositor (cust_name VARCHAR(50), acc_no INT, PRIMARY KEY (cust_name, acc_no), FOREIGN KEY (cust_name) REFERENCES Customer(cust_name) ON DELETE CASCADE, FOREIGN KEY (acc_no) REFERENCES Account(acc_no) ON DELETE CASCADE);
CREATE TABLE Borrower (cust_name VARCHAR(50), loan_no INT, PRIMARY KEY (cust_name, loan_no), FOREIGN KEY (cust_name) REFERENCES Customer(cust_name) ON DELETE CASCADE, FOREIGN KEY (loan_no) REFERENCES Loan(loan_no) ON DELETE CASCADE);

-- PS 10
ALTER TABLE Customer ADD phone VARCHAR(15);
ALTER TABLE Customer DROP COLUMN phone;
SELECT DISTINCT branch_name FROM Loan;
SELECT b.cust_name, l.loan_no, l.amount FROM Borrower b INNER JOIN Loan l ON b.loan_no = l.loan_no;
SELECT b.cust_name FROM Borrower b INNER JOIN Loan l ON b.loan_no = l.loan_no WHERE l.branch_name = 'Akurdi' ORDER BY b.cust_name ASC;
SELECT AVG(balance) AS avg_chinchwad_balance FROM Account WHERE branch_name = 'Chinchwad';
SELECT a.branch_name, COUNT(d.cust_name) AS no_of_depositors FROM Depositor d INNER JOIN Account a ON d.acc_no = a.acc_no GROUP BY a.branch_name;
SELECT branch_name, SUM(amount) AS total_loan FROM Loan GROUP BY branch_name;
SELECT branch_name FROM Loan GROUP BY branch_name HAVING AVG(amount) > 500000;

-- PS 11
ALTER TABLE Branch CHANGE assets Property DECIMAL(15,2);
SELECT loan_no FROM Loan WHERE branch_name = 'Akurdi' AND amount > 12000;
SELECT branch_name, AVG(balance) AS avg_balance FROM Account GROUP BY branch_name;
SELECT branch_name FROM Account GROUP BY branch_name HAVING AVG(balance) > 12000;
SELECT COUNT(*) AS total_customers FROM Customer;
SELECT SUM(amount) AS total_bank_loans FROM Loan;
DELETE FROM Loan WHERE amount BETWEEN 1300 AND 1500;
SELECT cust_name FROM Customer WHERE cust_street IS NULL;
SELECT branch_name FROM Branch WHERE branch_city IS NOT NULL;
SELECT cust_name FROM Customer WHERE cust_name LIKE 'A%';


=========================================
PROBLEM STATEMENT 12: MONGODB (ICEM / EMPLOYEE)
=========================================
use ICEM;
db.employee.find().pretty();
db.employee.find({dept: "Computer"}).pretty();
db.employee.find({$or: [{dept: "Computer"}, {dept: "IT"}]}).pretty();
db.employee.find({salary: {$gt: 30000}}).pretty();
db.employee.find({$and: [{salary: {$gte: 40000}}, {dept: "Computer"}]}).pretty();
db.employee.createIndex({ename: 1});
db.employee.dropIndex("ename_1");
db.employee.find({salary: {$gte: 20000, $lte: 40000}}).pretty();
db.employee.find({dept: {$ne: "IT"}}).pretty();
db.employee.aggregate([ {$group: {_id: "$dept", total_salary: {$sum: "$salary"}}} ]);
db.employee.aggregate([ {$group: {_id: "$dept", avg_salary: {$avg: "$salary"}}} ]);


=========================================
PROBLEM STATEMENT 13: MONGODB (STUDENT)
=========================================
use Student;
db.student.find().pretty();
db.student.find({dept: "Computer"}).pretty();
db.student.find({$or: [{dept: "Computer"}, {dept: "IT"}]}).pretty();
db.student.find({marks: {$gt: 70}}).pretty();
db.student.find({$and: [{marks: {$gte: 80}}, {dept: "Computer"}]}).pretty();
db.student.find({$or: [{marks: {$lt: 40}}, {dept: "IT"}]}).pretty();
db.student.find({marks: {$gte: 40, $lte: 80}}).pretty();
db.student.find({dept: {$ne: "IT"}}).pretty();
db.student.aggregate([ {$group: {_id: "$dept", total_marks: {$sum: "$marks"}}} ]);
db.student.createIndex({sname: 1});
db.student.dropIndex("sname_1");


=========================================
PROBLEM STATEMENT 14: ORACLE SET OPERATIONS
=========================================
-- Setup (Assuming tables Customer, Branch, Account, Loan are created)
SELECT * FROM Customer WHERE city IN ('Pune', 'Mumbai') ORDER BY cust_name ASC;
SELECT cust_name FROM Customer WHERE LOWER(cust_name) LIKE '%a%';
SELECT * FROM Customer WHERE phone IS NOT NULL;
SELECT * FROM Customer WHERE (city = 'Pune' AND cust_name LIKE 'P%') OR city = 'Mumbai';
SELECT branch_id, MAX(amount) AS max_loan, AVG(amount) AS avg_loan FROM Loan GROUP BY branch_id;
SELECT branch_id FROM Loan GROUP BY branch_id HAVING SUM(amount) > 100000;
SELECT acc_no, balance, (balance + 500) AS new_balance FROM Account;
UPDATE Account SET branch_id = 101 WHERE acc_no = 2005;
COMMIT;
DELETE FROM Customer WHERE phone IS NULL;

-- 10a. INTERSECT: Customers with BOTH an account and a loan
SELECT cust_id FROM Account INTERSECT SELECT cust_id FROM Loan;

-- 10b. MINUS: Customers with an account BUT NO loan
SELECT cust_id FROM Account MINUS SELECT cust_id FROM Loan;

Here is a condensed, quick-reference version of your Ubuntu setup and execution steps.

**1. Create & Run Python Scripts (PS 8-9)**

* **Create/Edit:** `nano mysql_connect.py` (or `mongo_connect.py`)
* **Save & Exit:** Paste code > `Ctrl + O` > `Enter` > `Ctrl + X`
* **Run:** `python3 mysql_connect.py`

**2. MySQL Setup (PS 1-4, 7, 10-11)**

* **Install & Start:**
```bash
sudo apt update && sudo apt install mysql-server
sudo systemctl start mysql

```


* **Enter Terminal:** `sudo mysql -u root -p`

**3. MongoDB Setup (PS 12-13)**

* **Install & Start:**
```bash
sudo apt install mongodb
sudo systemctl start mongodb

```


* **Enter Terminal:** `mongosh` (or `mongo`)

**4. Oracle Setup via Docker (PS 5, 6, 14)**

* **Install Docker:** `sudo apt install docker.io`
* **Download & Run Container:**
```bash
sudo docker run -d -p 1521:1521 -e ORACLE_PASSWORD=oracle --name oracle_db gvenzl/oracle-xe

```


* **Enter Terminal:** `sudo docker exec -it oracle_db sqlplus system/oracle`

**5. Python & Drivers Setup (PS 8-9)**

* **Install Python & PIP:** `sudo apt install python3 python3-pip`
* **Install DB Connectors:** `python3 -m pip install mysql-connector-python pymongo`
