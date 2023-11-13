# How to recreate the recommendation keyspaces

This file contains all of the commands required to follow the tutorial of CH3. 

## Question 1: Focussing on high collision-prone areas

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

## Question 3

**Table four: affected_groups_by_contributing_factor**

```cql
CREATE COLUMNFAMILY affected_groups_by_contributing_factor (
    country_iso_code varchar,  
    COUNT int,
    CONTRIBUTING_FACTOR_VEHICLE varchar,
    NUMBER_OF_PERSONS_INJURED int,
    NUMBER_OF_PERSONS_KILLED int,
    NUMBER_OF_PEDESTRIANS_INJURED int,NUMBER_OF_PEDESTRIANS_KILLED int,
    NUMBER_OF_CYCLIST_INJURED int,
    NUMBER_OF_CYCLIST_KILLED int,
    NUMBER_OF_MOTORIST_INJURED int,
    NUMBER_OF_MOTORIST_KILLED int,
    PRIMARY KEY ((country_iso_code), count) 
);

COPY collision_prone_areas.affected_groups_by_contributing_factor(
    COUNTRY_ISO_CODE,
    CONTRIBUTING_FACTOR_VEHICLE,
    NUMBER_OF_PERSONS_INJURED,
    NUMBER_OF_PERSONS_KILLED,
    NUMBER_OF_PEDESTRIANS_INJURED,
    NUMBER_OF_PEDESTRIANS_KILLED,
    NUMBER_OF_CYCLIST_INJURED,
    NUMBER_OF_CYCLIST_KILLED,
    NUMBER_OF_MOTORIST_INJURED,
    NUMBER_OF_MOTORIST_KILLED,
    COUNT
) 
FROM 'affected_groups_by_contributing_factor.csv' 
WITH DELIMITER = ',' 
AND HEADER = TRUE;
```

**Table five: vehicle_type_by_contributing_factor**

```cql
CREATE COLUMNFAMILY vehicle_type_by_contributing_factor (
    country_iso_code varchar,  
    COUNT int,
    VEHICLE_TYPE varchar,
    CONTRIBUTING_FACTOR_VEHICLE varchar,
    NUMBER_OF_PERSONS_INJURED int,
    NUMBER_OF_PERSONS_KILLED int,
    NUMBER_OF_PEDESTRIANS_INJURED int,NUMBER_OF_PEDESTRIANS_KILLED int,
    NUMBER_OF_CYCLIST_INJURED int,
    NUMBER_OF_CYCLIST_KILLED int,
    NUMBER_OF_MOTORIST_INJURED int,
    NUMBER_OF_MOTORIST_KILLED int,
    PRIMARY KEY ((country_iso_code, CONTRIBUTING_FACTOR_VEHICLE), count) 
);

COPY collision_prone_areas.vehicle_type_by_contributing_factor(
    COUNTRY_ISO_CODE,
    CONTRIBUTING_FACTOR_VEHICLE,
    VEHICLE_TYPE,
    NUMBER_OF_PERSONS_INJURED,
    NUMBER_OF_PERSONS_KILLED,
    NUMBER_OF_PEDESTRIANS_INJURED,
    NUMBER_OF_PEDESTRIANS_KILLED,
    NUMBER_OF_CYCLIST_INJURED,
    NUMBER_OF_CYCLIST_KILLED,
    NUMBER_OF_MOTORIST_INJURED,
    NUMBER_OF_MOTORIST_KILLED,
    count
) 
FROM 'vehicle_type_by_contributing_factor.csv' 
WITH DELIMITER = ',' 
AND HEADER = TRUE;
```

**Table six: affected_groups_by_vehicle_type**

```cql
CREATE COLUMNFAMILY affected_groups_by_vehicle_type (
    country_iso_code varchar,  
    VEHICLE_TYPE varchar,
    CONTRIBUTING_FACTOR_VEHICLE varchar,
    ZIP_CODE varchar,
    ON_STREET_NAME varchar,
    NUMBER_OF_PERSONS_INJURED int,
    NUMBER_OF_PERSONS_KILLED int,
    NUMBER_OF_PEDESTRIANS_INJURED int,NUMBER_OF_PEDESTRIANS_KILLED int,
    NUMBER_OF_CYCLIST_INJURED int,
    NUMBER_OF_CYCLIST_KILLED int,
    NUMBER_OF_MOTORIST_INJURED int,
    NUMBER_OF_MOTORIST_KILLED int,
    PRIMARY KEY ((country_iso_code, CONTRIBUTING_FACTOR_VEHICLE), ZIP_CODE) 
);

COPY collision_prone_areas.affected_groups_by_vehicle_type(
    ZIP_CODE,
    ON_STREET_NAME,
    NUMBER_OF_PERSONS_INJURED,
    NUMBER_OF_PERSONS_KILLED,
    NUMBER_OF_PEDESTRIANS_INJURED,
    NUMBER_OF_PEDESTRIANS_KILLED,
    NUMBER_OF_CYCLIST_INJURED,
    NUMBER_OF_CYCLIST_KILLED,
    NUMBER_OF_MOTORIST_INJURED,
    NUMBER_OF_MOTORIST_KILLED,
    CONTRIBUTING_FACTOR_VEHICLE,
    VEHICLE_TYPE,
    COUNTRY_ISO_CODE
) 
FROM 'affected_group_by_vehicle_type_prepared_and_folded.csv' 
WITH DELIMITER = ',' 
AND HEADER = TRUE;
```