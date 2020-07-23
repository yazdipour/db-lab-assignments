# Task 3 - Lab Training Database Systems SS20

## Shahriar Yazdipour [matr. No:62366]

a) Transform the schema accordingly to **a star schema and also a snowflake** schema.

b) Run your five queries from Task 1 again on both of the schemas.

## Star schema

```sh
f94abb250dd1:~$ createdb -U postgres dentist_star;
f94abb250dd1:~$ psql -U postgres dentist_star;
```

```sql
-- create
CREATE TABLE dimEmployee(
  ID serial UNIQUE NOT NULL,
  Name varchar(64) UNIQUE NOT NULL,
  Address varchar(250),
  Telephone varchar(20),
  Salary integer,
  PRIMARY KEY(ID, Name)
);

CREATE TABLE dimCustomer(
  Name varchar(64) PRIMARY KEY NOT NULL,
  Address text,
  Telephone varchar(20)
);

CREATE TABLE dimRoom(
  Number integer PRIMARY KEY NOT NULL,
  Size integer CHECK (Size > 0),
  Cleaner boolean
);

CREATE TABLE dimEvent(
  ID serial PRIMARY KEY NOT NULL,
  Time time
);

CREATE TABLE factTable(
  EventID serial NOT NULL,
  EName varchar(64),
  CName varchar(64),
  RoomNr integer,
  FOREIGN KEY (EventID) REFERENCES dimEvent,
  FOREIGN KEY (EName) REFERENCES dimEmployee(Name),
  FOREIGN KEY (CName) REFERENCES dimCustomer(Name),
  FOREIGN KEY (RoomNr) REFERENCES dimRoom,
  fact integer,
  constraint pk primary key (EventID ,EName, CName, RoomNr)
);

-- insert
INSERT INTO dimEmployee (Name, Address, Telephone, Salary) VALUES ('Franz', 'MaxPlk Ring',  '+4911000000', 1100);
INSERT INTO dimCustomer VALUES ('Heinz', 'MaxPlk Ring', '+49123123123');
INSERT INTO dimRoom VALUES (13, 2, TRUE);
INSERT INTO dimEvent VALUES ('2020-04-15 16:00:00');
```

### Queries

a) What is the average salary of employees sitting in the room number 13?

```sql
SELECT AVG(Salary) FROM dimEmployee JOIN factTable ON Name=EName WHERE RoomNr=13;
```

b) How many rooms are used at 16:00 by the employees with the three most highest salaries?

```sql
SELECT COUNT(DISTINCT dimRoom.Number)
FROM dimRoom JOIN factTable ON RoomNr=dimRoom.Number
JOIN dimEvent ON EventID=dimEvent.ID
WHERE TO_CHAR(Time, 'HH24:MI')='16:00'
AND EName IN (SELECT Name FROM dimEmployee ORDER BY dimEmployee.Salary DESC LIMIT 3);
```

c) Give the numbers of rooms where the customer "Heinz" was treated by employee "Franz" in the past.

```sql
SELECT Number FROM dimRoom
JOIN factTable ON RoomNr=dimRoom.Number
JOIN dimEvent ON EventID=dimEvent.ID
WHERE EName='Franz' AND CName='Heinz' AND time<now();
```

d) Sum of rooms Size that where allocated each day for events.

```sql
SELECT sum(size), TO_CHAR(time, 'yyyy-mm-dd') as dt
from factTable join dimRoom on room.number=RoomNr
join dimEvent on EventID=Event.ID group by dt;
```

e) Average salaries of two employee who give service to most customers.

```sql
select avg(salary)
from (select count(*), ename from factTable group by ename ORDER BY count DESC LIMIT 2) as active
join dimEmployee on ename=dimEmployee.name;
```

## Snowflake schema

```sh
f94abb250dd1:~$ createdb -U postgres dentist_snowflake;
f94abb250dd1:~$ psql -U postgres dentist_snowflake;
```

- **Split Address and telephone information into seperate table for normalization**
  - dimTelephone(ID, Telephone)
  - dimAddress(ID, Address)

```sql
-- create
CREATE TABLE dimTelephone(
  ID serial UNIQUE NOT NULL,
  Telephone varchar(20)
);

CREATE TABLE dimAddress(
  ID serial UNIQUE NOT NULL,
  Address text
);

CREATE TABLE dimEmployee(
  ID serial UNIQUE NOT NULL,
  Name varchar(64) UNIQUE NOT NULL,
  Address text REFERENCES dimAddress(Address),
  Telephone varchar(20) REFERENCES dimTelephone(Telephone),
  Salary integer,
  PRIMARY KEY(ID, Name)
);

CREATE TABLE dimCustomer(
  Name varchar(64) PRIMARY KEY NOT NULL,
  Address text REFERENCES dimAddress(Address),
  Telephone varchar(20) REFERENCES dimTelephone(Telephone)
);

CREATE TABLE dimRoom(
  Number integer PRIMARY KEY NOT NULL,
  Size integer CHECK (Size > 0),
  Cleaner boolean
);

CREATE TABLE dimEvent(
  ID serial PRIMARY KEY NOT NULL,
  Time time
);

CREATE TABLE factTable(
  EventID serial NOT NULL,
  EName varchar(64) REFERENCES dimEmployee(Name),
  CName varchar(64) REFERENCES dimCustomer(Name),
  RoomNr integer,
  FOREIGN KEY (EventID) REFERENCES dimEvent,
  FOREIGN KEY (EName) REFERENCES dimEmployee(Name),
  FOREIGN KEY (CName) REFERENCES dimCustomer(Name),
  FOREIGN KEY (RoomNr) REFERENCES dimRoom,
  fact integer,
  constraint pk primary key (EventID ,EName, CName, RoomNr)
);

-- insert
INSERT INTO dimAddress VALUES ('MaxPlk Ring');
INSERT INTO dimTelephone VALUES ('+49123123123');
INSERT INTO dimEmployee (Name, Address, Telephone, Salary) VALUES ('Franz', 1 ,2 , 1100);
INSERT INTO dimCustomer VALUES ('Heinz', 2, 1);
INSERT INTO dimRoom VALUES (13, 2, TRUE);
INSERT INTO dimEvent VALUES ('2020-04-15 16:00:00');
```

### Queries

a) What is the average salary of employees sitting in the room number 13?

```sql
SELECT AVG(Salary) FROM dimEmployee JOIN factTable ON Name=EName WHERE RoomNr=13;
```

b) How many rooms are used at 16:00 by the employees with the three most highest salaries?

```sql
SELECT COUNT(DISTINCT dimRoom.Number)
FROM dimRoom JOIN factTable ON RoomNr=dimRoom.Number
JOIN dimEvent ON EventID=dimEvent.ID
WHERE TO_CHAR(Time, 'HH24:MI')='16:00'
AND EName IN (SELECT Name FROM dimEmployee ORDER BY dimEmployee.Salary DESC LIMIT 3);
```

c) Give the numbers of rooms where the customer "Heinz" was treated by employee "Franz" in the past.

```sql
SELECT Number FROM dimRoom
JOIN factTable ON RoomNr=dimRoom.Number
JOIN dimEvent ON EventID=dimEvent.ID
WHERE EName='Franz' AND CName='Heinz' AND time<now();
```

d) Sum of rooms Size that where allocated each day for events.

```sql
SELECT sum(size), TO_CHAR(time, 'yyyy-mm-dd') as dt
from factTable join dimRoom on room.number=RoomNr
join dimEvent on EventID=Event.ID group by dt;
```

e) Average salaries of two employee who give service to most customers.

```sql
select avg(salary)
from (select count(*), ename from factTable group by ename ORDER BY count DESC LIMIT 2) as active
join dimEmployee on ename=dimEmployee.name;
```
