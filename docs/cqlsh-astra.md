# Creation

You can create a free account, valid for 2 weeks at https://astra.datastax.com.

# Connection

To use the Astra CLI, you need to create a token with the "Organization Administrator" role: [https://astra.datastax.com/org/\<your-org-id\>/settings/tokens]()

Follow this document to install and connect to an online Astra DB https://awesome-astra.github.io/docs/pages/astra/astra-cli/, or try `curl -Ls https://dtsx.io/get-astra-cli | bash`.

Once set up and connected, you can use `$ astra db cqlsh \<name of database\> --token=AstraCS:\<token\>` to start a cqlsh.

# Working

just like [./local-usecase.md](local-usecase).

reload by zipcode

```cql
COPY collision_prone_areas.collisions_by_street(
    country_iso_code, 
    zip_code, 
    on_street_name, 
    number_of_persons_injured, 
    number_of_persons_killed, 
    count
) 
FROM '/mnt/c/<path-to-csv>.csv'
WITH DELIMITER = ',' 
AND HEADER = TRUE;
```

/mnt/c/TUD/source/databases/cassandra-lab/datasets/prepared/collisions_by_street.csv

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
FROM '/mnt/c/TUD/source/databases/cassandra-lab/datasets/prepared/affected_groups_by_contributing_factor.csv' 
WITH DELIMITER = ',' 
AND HEADER = TRUE;
```