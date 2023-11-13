Copying the files over to the node:
```
 docker cp collisions_by_zipcode.csv cass1:/collisions_by_zipcode.csv
 docker cp collisions_by_street.csv cass1:/collisions_by_street.csv
 docker cp history_by_street.csv cass1:/history_by_street_prepared.csv
```

# Question 1

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

**Table two: collisions_by_street**

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

**Table three: history_by_street**

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

# Question 3

**Table four: collisions_by_contributing_factor**

```cql
CREATE COLUMNFAMILY collisions_by_contributing_factor (
    country_iso_code varchar,  
    count int,  
    CONTRIBUTING_FACTOR_VEHICLE varchar,
    number_of_persons_injured int, 
    number_of_persons_killed int,  
    PRIMARY KEY ((country_iso_code), count) 
);

COPY collision_prone_areas.collisions_by_contributing_factor(
    COUNTRY_ISO_CODE,
    CONTRIBUTING_FACTOR_VEHICLE,
    NUMBER_OF_PERSONS_INJURED,
    NUMBER_OF_PERSONS_KILLED,
    COUNT
) 
FROM 'affected_group_by_contributing_factor.csv' 
WITH DELIMITER = ',' 
AND HEADER = TRUE;
```