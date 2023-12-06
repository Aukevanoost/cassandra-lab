Lost? go [back](./../readme.md)

# How to recreate the recommendation keyspaces

This file contains all of the commands required to follow the tutorial of CH3 (Research document). 

## Background information

Cassandra works a little different than traditional databases if it comes to modelling your data. Whereas often you would normalize your data to save precious storage, Cassandra would focus on optimizing your queries for fast data lookups. In other words, one table per question (query) and focussing on the properties that we will be using for filtering and ordering. The process of building these massive tables from normalized datasets is conveniently called ‘denormalisation’. 
We start with a raw freshly downloaded CSV file containing all the vehicle collisions and their connected data, the file contains 29 columns of metadata like how many vehicles were involved, if people got injured, the location, date, time etc. [Check their point of view](https://cassandra.apache.org/doc/latest/cassandra/developing/data-modeling/intro.html).

The first recommendation would be focussing on high collision-prone areas, according to the analysis of Mrs. Ayodele there are areas that are more prone to errors than other areas. That would mean that we would focus on locations as a starting point and per table, then dive deeper into the characteristics of the collisions based on their (specific) location.  Every table provides the input for the next table to carve out a path towards the collision-heavy streets and their weak points (where most incidents happen).

To thicken the datasets, we've copied the dataset CSV's and added a country_iso_code to simulate a bigger dataset. Note that in Cassandra a Partitioning key (first argument of an PRIMARY KEY) is required in the where clause, this is for the simple reason that the partitioning key tells cassandra where the data (on which node) is located. 

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


**Copying the CSV files over to the node:** <br />
*Only if not already added to the mount folder*
```
 docker cp collisions_by_zipcode_XYZ.csv cass1:/collisions_by_zipcode_XYZ.csv
 docker cp collisions_by_street.csv cass1:/collisions_by_street.csv
 docker cp history_by_street_XYZ.csv cass1:/history_by_street_XYZ.csv
```



**Table one: collisions_by_zipcode**
```
CREATE COLUMNFAMILY collisions_by_zipcode (   
    country_iso_code varchar,   
    zip_code int,  
    number_of_persons_injured int,   
    number_of_persons_killed int,   
    count int,   
    PRIMARY KEY (country_iso_code, count, zip_code) 
);

COPY collision_prone_areas.collisions_by_zipcode (
    country_iso_code, 
    zip_code, 
    number_of_persons_injured, 
    number_of_persons_killed, 
    count
) 
FROM 'collisions_by_zipcode_USA.csv' 
WITH DELIMITER = ',' 
AND HEADER = TRUE;
```
or use the absolute path from the mount folder: `/var/lib/cassandra/recommendation_one/collisions_by_zipcode_USA.csv`

**Table two: collisions_by_street**

```
CREATE COLUMNFAMILY collisions_by_street (      
    country_iso_code varchar,  
    zip_code int,  
    on_street_name varchar,  
    number_of_persons_injured int,   number_of_persons_killed int,  
    count int,  
    PRIMARY KEY ((country_iso_code, zip_code), count, on_street_name) 
);

COPY collision_prone_areas.collisions_by_street(
    country_iso_code, 
    zip_code, 
    on_street_name, 
    number_of_persons_injured, 
    number_of_persons_killed, 
    count
) 
FROM 'collisions_by_street_USA.csv' 
WITH DELIMITER = ',' 
AND HEADER = TRUE;
```

**Table three: history_by_street**

```
CREATE COLUMNFAMILY history_by_street (
    id varchar,
    country_iso_code varchar,  
    zip_code int,  
    on_street_name varchar,  
    year int,
    month int,
    day int,
    number_of_persons_injured int, 
    number_of_persons_killed int,  
    PRIMARY KEY ((country_iso_code, zip_code, on_street_name), year, month, day, id) 
);

COPY collision_prone_areas.history_by_street(
    year, 
    month, 
    day, 
    zip_code, 
    on_street_name, 
    number_of_persons_injured,   
    number_of_persons_killed, 
    id,
    country_iso_code
) 
FROM 'history_by_street_AUS.csv' 
WITH DELIMITER = ',' 
AND HEADER = TRUE;
```

#### Querying data from the tables: 

```
SELECT * FROM collisions_by_zipcode WHERE country_iso_code = 'USA' ORDER BY count DESC LIMIT 10;

SELECT * FROM collisions_by_street WHERE country_iso_code = 'USA' AND zip_code = 11207 ORDER BY count DESC LIMIT 10;

SELECT  * FROM history_by_street WHERE country_iso_code = 'USA' AND zip_code = 11207 AND on_street_name = 'PENNSYLVANIA AVENUE' LIMIT 10;
```

#### Get street with most collisions

```
SELECT 
    year, 
    COUNT(*) as incidents, 
    SUM(number_of_persons_killed) as killed, 
    SUM(number_of_persons_injured) as injured 
FROM history_by_street 
WHERE country_iso_code = 'SWE' 
AND zip_code = 11207 
AND on_street_name = 'PENNSYLVANIA AVENUE'
GROUP BY year;

SELECT * FROM history_by_street 
WHERE country_iso_code = 'SWE' 
AND zip_code = 11207 
AND on_street_name = 'PENNSYLVANIA AVENUE';
```

## Recommendation 3

### First, creating a keyspace

```
CREATE KEYSPACE collision_prone_vehicles 
WITH replication = {'class': 'SimpleStrategy', 'replication_factor': '2'}  
AND durable_writes = true;

USE collision_prone_vehicles
```

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

COPY collision_prone_vehicles.affected_groups_by_contributing_factor(
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
FROM '/var/lib/cassandra/recommendation_three/affected_groups_by_contributing_factor_USA.csv' 
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
    NUMBER_OF_PEDESTRIANS_INJURED int,
    NUMBER_OF_PEDESTRIANS_KILLED int,
    NUMBER_OF_CYCLIST_INJURED int,
    NUMBER_OF_CYCLIST_KILLED int,
    NUMBER_OF_MOTORIST_INJURED int,
    NUMBER_OF_MOTORIST_KILLED int,
    PRIMARY KEY ((country_iso_code, CONTRIBUTING_FACTOR_VEHICLE), count) 
);

COPY collision_prone_vehicles.vehicle_type_by_contributing_factor(
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
FROM '/var/lib/cassandra/recommendation_three/vehicle_type_by_contributing_factor_USA.csv' 
WITH DELIMITER = ',' 
AND HEADER = TRUE;
```

**Table six: affected_groups_by_vehicle_type**

```cql
CREATE COLUMNFAMILY affected_groups_by_vehicle_type (
    id varchar,
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
    PRIMARY KEY ((country_iso_code, CONTRIBUTING_FACTOR_VEHICLE), ZIP_CODE, vehicle_type, on_street_name, id) 
);

COPY collision_prone_vehicles.affected_groups_by_vehicle_type(
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
    ID,
    VEHICLE_TYPE,
    COUNTRY_ISO_CODE
) 
FROM '/var/lib/cassandra/recommendation_three/affected_group_by_vehicle_type_USA.csv' 
WITH DELIMITER = ',' 
AND HEADER = TRUE;
```

**Materialised view on table six: affected_zip_codes_by_vehicle_type**

hint: not available in Astra Serverless DB as of 15th November 2023.

```cql
CREATE MATERIALIZED VIEW affected_zip_codes_by_vehicle_type AS
    SELECT
        id,
        country_iso_code,  
        VEHICLE_TYPE,
        CONTRIBUTING_FACTOR_VEHICLE,
        ZIP_CODE,
        ON_STREET_NAME,
        NUMBER_OF_PERSONS_INJURED,
        NUMBER_OF_PERSONS_KILLED,
        NUMBER_OF_PEDESTRIANS_INJURED,NUMBER_OF_PEDESTRIANS_KILLED,
        NUMBER_OF_CYCLIST_INJURED,
        NUMBER_OF_CYCLIST_KILLED,
        NUMBER_OF_MOTORIST_INJURED,
        NUMBER_OF_MOTORIST_KILLED
    FROM affected_groups_by_vehicle_type
PRIMARY KEY ((country_iso_code, CONTRIBUTING_FACTOR_VEHICLE), vehicle_type, ZIP_CODE, on_street_name, id);
```

# How to answer our questions

## Question 1

Query 1: Selecting the zip codes with the most collisions:
```CQL
SELECT *
FROM collisions_by_zipcode
WHERE country_iso_code='USA'
ORDER BY count desc
LIMIT 10;
```

Query 2: Selecting the streets with the most collisions, filtering by the zip code with the most collisions:
```CQL
SELECT *
FROM collisions_by_street
WHERE
    country_iso_code='USA' and
    zip_code=11207
ORDER BY count DESC
LIMIT 10;
```

Query 3: Selecting the history of that particular street:
```CQL
SELECT
    month,
    sum(number_of_persons_injured) as persons_injured,
    sum(number_of_persons_killed) as persons_killed
FROM history_by_street
WHERE
    country_iso_code='USA' and
    zip_code=11207 and
    on_street_name='PENNSYLVANIA AVENUE' and
    year=2022
GROUP BY month
LIMIT 10;
```
## Question 3

Query 4: Selecting the contributing factor leading to the most incidents:
```CQL
SELECT
    count as incidents,
    contributing_factor_vehicle,
    number_of_persons_injured as injured,
    number_of_persons_killed as killed,
    number_of_cyclist_injured as cyclist_i,
    number_of_cyclist_killed as cyclist_k,
    number_of_motorist_injured as motorist_i,
    number_of_motorist_killed as motorist_k,
    number_of_pedestrians_injured as pedestrians_i,
    number_of_pedestrians_killed as pedestrians_k
FROM affected_groups_by_contributing_factor
WHERE country_iso_code='USA'
ORDER BY count desc
LIMIT 10;
```
Query 5: Selecting the vehicle types involved in incidents with that particular contributing factor:
```CQL
SELECT
    VEHICLE_TYPE,
    count as incidents,
    number_of_persons_injured as injured,
    number_of_persons_killed as killed,
    number_of_cyclist_injured as cyclist_i,
    number_of_cyclist_killed as cyclist_k,
    number_of_motorist_injured as motorist_i,
    number_of_motorist_killed as motorist_k,
    number_of_pedestrians_injured as pedestrians_i,
    number_of_pedestrians_killed as pedestrians_k
FROM vehicle_type_by_contributing_factor
WHERE
    country_iso_code='USA' and
    contributing_factor_vehicle='DRIVER INATTENTION/DISTRACTION'
ORDER BY count desc
LIMIT 10;
```

Query 6: Selecting the zip codes and street names where the combination of vehicle type and contributing factor cause the most incidents:
```CQL
SELECT
    zip_code,
    count(*) as incidents,
    sum(number_of_persons_injured) as injured,
    sum(number_of_persons_killed) as killed,
    sum(number_of_cyclist_injured) as cyclist_i,
    sum(number_of_cyclist_killed) as cyclist_k,
    sum(number_of_motorist_injured) as motorist_i,
    sum(number_of_motorist_killed) as motorist_k,
    sum(number_of_pedestrians_injured) as pedestrians_i,
    sum(number_of_pedestrians_killed) as pedestrians_k
FROM affected_groups_by_vehicle_type
WHERE
    country_iso_code='USA' and
    contributing_factor_vehicle='DRIVER INATTENTION/DISTRACTION' and
    vehicle_type='SEDAN' and
    zip_code='11207'
LIMIT 10;
```
