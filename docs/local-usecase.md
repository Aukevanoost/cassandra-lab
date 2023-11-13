# How to recreate the recommendation keyspaces

This file contains all of the commands required to follow the tutorial of CH3. 

## Recommendation 1: Focussing on high collision-prone areas

As described in the tutorial, the goal of this recommendation is to find out which zip codes have 
the most collisions and which of their streets/locations require some safety revisions. 

### Step one, create the keyspace: 

```
CREATE KEYSPACE collision_prone_areas 
WITH replication = {'class': 'SimpleStrategy', 'replication_factor': '2'}  
AND durable_writes = true;

USE collision_prone_areas
```

### Step one, create the tables: 


**Copying the CSV files over to the node:**
```
 docker cp collisions_by_zipcode.csv cass1:/collisions_by_zipcode.csv
 docker cp collisions_by_street.csv cass1:/collisions_by_street.csv
 docker cp history_by_street.csv cass1:/history_by_street_prepared.csv
```

**Table one: collisions_by_zipcode**
```
CREATE COLUMNFAMILY collisions_by_zipcode (   
    country_iso_code varchar,   
    zip_code int,  
    number_of_persons_injured int,   
    number_of_persons_killed int,   
    count int,   
    PRIMARY KEY (country_iso_code, zip_code, count) 
);

COPY collision_prone_areas.collisions_by_zipcode (
    country_iso_code, 
    zip_code, 
    number_of_persons_injured, 
    number_of_persons_killed, 
    count
) 
FROM 'collisions_by_zipcode.csv' 
WITH DELIMITER = ',' 
AND HEADER = TRUE;
```

**Table two: collisions_by_zipcode**

```
CREATE COLUMNFAMILY collisions_by_street (      
    country_iso_code varchar,  
    zip_code int,  
    on_street_name varchar,  
    number_of_persons_injured int,   number_of_persons_killed int,  
    count int,  
    PRIMARY KEY ((country_iso_code, zip_code), count) 
);

COPY collision_prone_areas.collisions_by_street(
    country_iso_code, 
    zip_code, 
    on_street_name, 
    number_of_persons_injured, 
    number_of_persons_killed, 
    count
) 
FROM 'collisions_by_street.csv' 
WITH DELIMITER = ',' 
AND HEADER = TRUE;
```

**Table three: history_by_zipcode**

```
CREATE COLUMNFAMILY history_by_street (      
    country_iso_code varchar,  
    zip_code int,  
    on_street_name varchar,  
    year int,
    month int,
    day int,
    number_of_persons_injured int, 
    number_of_persons_killed int,  
    PRIMARY KEY ((country_iso_code, zip_code, on_street_name), year, month, day) 
);

COPY collision_prone_areas.history_by_street(
    year, 
    month, 
    day, 
    zip_code, 
    on_street_name, 
    number_of_persons_injured,   
    number_of_persons_killed, 
    country_iso_code 
) 
FROM 'history_by_street.csv' 
WITH DELIMITER = ',' 
AND HEADER = TRUE;
```

