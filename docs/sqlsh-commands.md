Lost? go [back](./../readme.md)

# 2. SQLSH Keyspace setup

**CQL basic commands cheatsheet!** Based on the cassandra tutorial CH6 (ref in readme.md).

## 2.1 Creating the keyspace

```
CREATE KEYSPACE catalog 
WITH replication = {'class': 'SimpleStrategy', 'replication_factor': '3'}  
AND durable_writes = true;
```

See available keyspaces with `DESCRIBE keyspaces;`


## 2.2 Creating a table

Enter the keyspace:
```
USE catalog;
```
Create a new product column family:
```
CREATE COLUMNFAMILY product (
    productId varchar,
    title TEXT,
    brand varchar,
    publisher varchar,
    length int,
    breadth int,
    height int,
    PRIMARY KEY  (productId) 
);
```

## 2.3 Altering your table

Add a column: 
```
ALTER COLUMNFAMILY product add modelid text;
```

Altering specs
```
ALTER COLUMNFAMILY product WITH gc_grace_seconds=86400;
```

## 2.4 Adding data to your table

Adding rows:
```
INSERT INTO product(productid, brand, modelid) 
VALUES ('MOB1', 'Samsung', 'GalaxyS6');

INSERT INTO product(productid, title, publisher) 
VALUES ('BOK1', 'The Kite Runner', 'Riverbed Books');

INSERT INTO product(productid, title, breadth, length) 
VALUES  ('POST1', 'hard work', 22, 36);

INSERT INTO product(productid, title, breadth, length) 
VALUES  ('POST2', 'led zeppelin', 22, 36) 
USING TIMESTAMP 1468580580000;
```

Overriding rows:
```
INSERT INTO product(productid, title, breadth, length) 
VALUES ('POST2', 'adele', 22, 36) 
USING TTL 86400;
```

## 2.5 Querying data

Showing all data in your column:
```
SELECT * FROM product;
```

To show row for a productid with EQ :
```
SELECT * FROM product WHERE productid = 'POST1';
```

To show row for a productid with EQ :
```
SELECT productid, title, modelid 
FROM product 
WHERE productid IN ('POST1', 'MOB1');
```

## 2.6 Advanced data types (List, Set and Map)

Adding a `SET` data-type 
```
ALTER COLUMNFAMILY product 
ADD keyfeatures set<TEXT>;

INSERT INTO product(productid, title, brand, keyfeatures) 
VALUES ('COM1', 'Acer ONE', 'ACER', {'detachable keyboard', 'multitouch display'}); 
```

Adding a `LIST` data-type 
```
ALTER COLUMNFAMILY product 
ADD service_type list<TEXT>;

INSERT INTO product(productid, title, brand, service_type) 
VALUES ('SOFA1', 'Urban Living Derby Sofa', 'Urban Living', ['Needs to call seller', 'Service Engineer will Come to the Site ']);
```

Adding a `MAP` data-type 
```
## To add column with data type MAP
ALTER COLUMNFAMILY product ADD camera map<TEXT, TEXT>;

INSERT INTO product(productid, title, brand, camera) 
VALUES ('COM1', 'Acer ONE', 'ACER', {'front':'VGA', 'rear':'2MP'});
```

Adding a `COUNTER` to your product
```
CREATE COLUMNFAMILY productviewcount(productid text primary key, viewcount COUNTER);

UPDATE productviewcount 
SET viewcount = viewcount + 1 
WHERE productid = 'COM1';

SELECT * 
FROM productviewcount 
WHERE productid = 'COM1';
```

## 2.7 Updating simple columns

To update a column of a product:
```
UPDATE  product 
SET modelid = 'S6' 
WHERE productid = 'MOB1';
```

To update multiple columns of a product:
```
UPDATE product 
SET breadth = 18, length = 25, height = 2 
WHERE productid in ('COM1', 'COM2');
```

## 2.8 Updating advanced columns

### 2.8.1 Managing `LIST` columns

Add:
```
UPDATE product 
SET service_type = service_type + ['Service engineer will inspect the damages'] 
WHERE productid = 'SOFA1';
```

Remove:
```
UPDATE product 
SET service_type = service_type - ['Needs to Call Seller'] 
WHERE productid = 'SOFA1';
```

Replace:
```
UPDATE product 
SET service_type[1] = 'service engineer will call for appointment' 
WHERE productid = 'SOFA1';
```

### 2.8.2 Managing `SET` columns

Update (entry is valid for one day, will then be deleted)
```
UPDATE product 
USING TTL 86400 
SET keyfeatures = keyfeatures + {'FLAT 10% off for 1 day'} 
WHERE productid IN ('COM1', 'COM2');
```

Remove:
```
UPDATE product 
SET keyfeatures = keyfeatures - {'detachable keyboard'} 
WHERE productid IN ('COM1', 'COM2');
```

### 2.8.3 Managing `MAP` columns

Add:
```
UPDATE product 
SET camera = camera + {'frontVideo':'1MP'} 
WHERE productid = 'COM1';
```

Remove:
```
DELETE camera['frontVideo'] 
FROM product 
WHERE productid = 'COM1';

```

Replace:
```
UPDATE product 
SET camera['front'] = '1MP' 
WHERE productid = 'COM1';
```
