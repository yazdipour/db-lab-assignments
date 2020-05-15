# Lab Training Database Systems SS20

## Shahriar Yazdipour [matr. No:62366]

## Task 1

Dentist DB:

Description: A dentist keeps track of his employees and customers. Therefore, the following tables are necessary:

- Employee (ID, Name, Address, Telephone, Salary, OfficeNr->Room)
- Customer (Name, Address, Telephone)
- Event (ID, Name->Employee, Name->Customer, Number->Room, Time)
- Room (Number, Size, Cleaner)

## Setup

```sh
$ docker pull postgres
$ docker run --name postgres-0 -e POSTGRES_PASSWORD=toor -p 5432:5432 -d postgres
$ docker exec -it postgres-0 /bin/sh -c bash
postgres@f94abb250dd1:~$ createdb dentist
postgres@f94abb250dd1:~$ psql dentist
```

## Creating Tables

```sql
CREATE TABLE Employee(
  ID serial UNIQUE NOT NULL,
  Name varchar(64) UNIQUE NOT NULL,
  Address varchar(250),
  Telephone varchar(20),
  Salary integer,
  OfficeNr integer REFERENCES Room,
  PRIMARY KEY(ID, Name)
);
CREATE TABLE Customer(
  Name varchar(64) PRIMARY KEY NOT NULL,
  Address text,
  Telephone varchar(20)
);
CREATE TABLE Event(
  ID serial PRIMARY KEY NOT NULL,
  EName varchar(64) REFERENCES Employee(Name),
  CName varchar(64) REFERENCES Customer(Name),
  Number integer REFERENCES Room,
  Time time
);

CREATE TABLE Room(
  Number integer PRIMARY KEY NOT NULL,
  Size integer CHECK (Size > 0),
  Cleaner boolean -- what does Cleaner Means?!
);
```

## Inserting Data

```sql
INSERT INTO Room VALUES (13, 2, TRUE);

dentist=# select * from room;
 number | size | cleaner
--------+------+---------
     13 |    4 | t
     11 |    1 | t
     22 |    2 | t
     33 |    3 | t
     77 |    8 | t
     44 |    8 | f
     55 |    4 | f
     66 |    4 | f
     88 |    4 | t
     99 |    4 | t

INSERT INTO Employee (Name, Address, Telephone, Salary, OfficeNr) VALUES ('Franz', 'MaxPlk Ring',  '+4911000000', 1100, 13);

dentist=# select * from Employee;
 id |  name  |   address   |  telephone  | salary | officenr
----+--------+-------------+-------------+--------+----------
  1 | Franz  | MaxPlk Ring | +4911000000 |   1100 |       13
  3 | Franz2 | MaxPlk Ring | +4911222222 |   1200 |       22
  4 | Franz3 | MaxPlk Ring | +4911333333 |   1300 |       22
  5 | Franz4 | MaxPlk Ring | +4911444444 |   1400 |       13
  6 | Franz5 | MaxPlk Ring | +4911555555 |   1500 |       55
  7 | Franz6 | MaxPlk Ring | +4911666666 |   1600 |       13
  8 | Franz7 | MaxPlk Ring | +4911777777 |   1700 |       77
  9 | Franz8 | MaxPlk Ring | +4911888888 |   1800 |       88
 10 | Franz9 | MaxPlk Ring | +4911999999 |   1900 |       99

INSERT INTO Customer VALUES ('Heinz', 'MaxPlk Ring', '+49123123123');

dentist=# select * from Customer;
  name  |   address   |  telephone
--------+-------------+--------------
 Heinz  | MaxPlk Ring | +49123123123
 Heinz1 | MaxPlk Ring | +49122111111
 Heinz2 | MaxPlk Ring | +49122222222
 Heinz3 | MaxPlk Ring | +49122333333
 Heinz4 | MaxPlk Ring | +49122444444
 Heinz5 | MaxPlk Ring | +49122555555
 Heinz6 | MaxPlk Ring | +49122666666
 Heinz7 | MaxPlk Ring | +49122777777
 Heinz8 | MaxPlk Ring | +49122888888
 Heinz9 | MaxPlk Ring | +49122999999

INSERT INTO Event (EName, CName, Number, Time) VALUES ('Franz', 'Heinz', 13, '2020-04-15 16:00:00');

dentist=# select * from event;
 id | ename  | cname  | number |        time
----+--------+--------+--------+---------------------
  4 | Franz3 | Heinz3 |     33 | 2020-05-15 17:07:03
  5 | Franz4 | Heinz4 |     44 | 2020-05-15 17:07:03
 12 | Franz7 | Heinz7 |     77 | 2020-05-15 17:07:03
  6 | Franz5 | Heinz5 |     55 | 2020-04-15 17:07:03
  7 | Franz6 | Heinz6 |     66 | 2020-04-15 16:00:03
 11 | Franz9 | Heinz9 |     77 | 2020-05-15 16:00:03
  1 | Franz  | Heinz  |     13 | 2020-04-15 16:00:00
  3 | Franz3 | Heinz2 |     55 | 2020-04-15 17:07:03
  9 | Franz5 | Heinz8 |     55 | 2020-05-15 16:00:03
```

## Queries

a) What is the average salary of employees sitting in the room number 13?

```sql
SELECT AVG(Salary) FROM employee JOIN room ON OfficeNr=Number WHERE Number=13;
          avg
-----------------------
 1366.6666666666666667
```

b) How many rooms are used at 16:00 by the employees with the three most highest salaries?

```sql
-- Franz7, Franz8, Franz9 have the highest salaries.

SELECT COUNT(DISTINCT Room.Number)
FROM Event JOIN Room ON Event.Number=Room.Number
WHERE TO_CHAR(Event.Time, 'HH24:MI')='16:00'
AND EName IN (SELECT Name FROM Employee ORDER BY Employee.Salary DESC LIMIT 3);

 count
-------
    1
```

c) Give the numbers of rooms where the customer "Heinz" was treated by employee "Franz" in the past.

I did not understand what the question exactly wants.

If by `numbers of rooms` you mean, room numbers:

```sql
SELECT Number FROM Event
WHERE EName='Franz' AND CName='Heinz' AND time<now();
 number
--------
     13
(1 row)
```

But if by `numbers of rooms` you mean, how many rooms:

```sql
SELECT count(Number) FROM Event
WHERE EName='Franz' AND CName='Heinz' AND time<now();
 count
-------
     1
(1 row)
```

Two additional tasks:

d) Sum of Rooms Size that where allocated each day for events.

```sql
SELECT sum(size), TO_CHAR(time, 'yyyy-mm-dd') as dt
from event left join room on room.number=event.number group by dt;
 sum |     dt
-----+------------
  16 | 2020-04-15
  31 | 2020-05-15
(2 rows)
```

e) Average salaries of two employee who give service to most customers.

```sql
select avg(salary)
from (select count(*), ename from event group by ename ORDER BY count DESC LIMIT 2) as active
join employee on ename=employee.name;
          avg
-----------------------
 1400.0000000000000000
(1 row)
```
