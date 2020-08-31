
# ACID Behaviour

## Postgres


This is a brief _how-to_ taken from [[1]], demonstrating
how transactions work in postgres.


### Container


Launch a postgres container (we are using the official [dockerhub](https://hub.docker.com/_/postgres) container)
```
docker run --rm   --name pg-docker -e POSTGRES_PASSWORD=docker -d -p 5432:5432  postgres
```

Connect to your running container:
```
PGPASSWORD='docker' psql -h localhost -U postgres
```

### Create the database and populate

```
CREATE DATABASE acid;
\c acid;
```

Create the table:

```
DROP TABLE accounts IF EXISTS;

CREATE TABLE accounts (
    id INT GENERATED BY DEFAULT AS IDENTITY,
    name VARCHAR(100) NOT NULL,
    balance DEC(15,2) NOT NULL,
    PRIMARY KEY(id)
);
```

Populate it with some data:

```
INSERT INTO accounts(name,balance)
VALUES('Bob',10000),('Alice',10000);
```

### Observe Transactions

Let's create a transaction.
Select from the table in your current session (say SessionA)

```
---SessionA

BEGIN;

UPDATE ACCOUNTS SET balance=balance-1000 where id=1;


SELECT * FROM accounts;
 id | name  | balance  
----+-------+----------
  1 | Bob   | 9000.00
  2 | Alice | 10000.00
```


Log into a new session (SessionB)

```
---SessionB
SELECT * FROM accounts;
id | name | balance  
----+------+----------
 1 | Bob  | 10000.00
```

Ah ha! It is not there. let's commit that work

```
COMMIT;
```



How would we structure a transaction in general then?

```
-- start a transaction
BEGIN;

-- deduct 1000 from account 1
UPDATE accounts
SET balance = balance - 1000
WHERE id = 1;

-- add 1000 to account 2
UPDATE accounts
SET balance = balance + 1000
WHERE id = 2;

-- commit the transaction
COMMIT;
```


[1]: https://www.postgresqltutorial.com/postgresql-transaction/