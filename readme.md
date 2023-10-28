# Cassandra lab

Created to get a better understanding of how Cassandra works and how to manage your Database/Cluster.


## Table of Contents

1. [Setting up your cluster](./docs/setup-docker.md)
2. [[CH6] Basic SQLSH commands](./docs/sqlsh-commands.md)


## Sources used for this repository

*Create a docker cluster* - 
https://blog.digitalis.io/containerized-cassandra-cluster-for-local-testing-60d24d70dcc4<br />
*How to cassandra 101* - 
https://www.udemy.com/course/from-0-to-1-the-cassandra-distributed-database


<br />
<br />
<br />
<br />

## EXTRA COMMANDS
```
    ## To create keycaps with case sensitive name
    CREATE KEYSPACE "MyCatalog" 
    WITH replication = {'class': 'SimpleStrategy', 'replication_factor': '3'}  
    AND durable_writes = true;
    
    ## To list all columnfamilies in catalog keyspace:
    describe columnfamilies;

    ## To create columnfamily skus
    CREATE TABLE skus (
        sellerId varchar,
        productId varchar,
        skuId varchar,
        title text,
        listingId varchar,
        isListingCreated boolean,
        timeofskucreation text,
        PRIMARY KEY((sellerId,skuId), timeofskucreation, productId)
    );

    ## To create columnfamily listing
    CREATE TABLE listings (
        listingId varchar,
        sellerId varchar,
        skuId varchar,
        productId varchar,
        mrp int,
        ssp int,
        sla int,
        stock int,
        title text,
        PRIMARY KEY ((sellerid,skuId), stock, ssp)
    ); 

    ## To describe  column family product
    DESCRIBE product;

    ## to describe keyspace catalog
    DESCRIBE KEYSPACE catalog;

    ## Altering the replication index of keyspace
    ALTER KEYSPACE "products" 
    WITH REPLICATION = { 'class' : 'SimpleStrategy', 'replication_factor' : 2};




    ## To load data from file skus.csv into skus Column Family
    COPY skus(sellerid, productid, skuid, title,listingid, islistingcreated,timeofskucreation) FROM 'skus.csv' ;

    ## To show data of skus
    select * from skus;

    ## To do range query using > on token(partition key)
    select * from skus where token(sellerid,skuid) >= token('Chroma', 'ChromaSKU1') ;

    ## To query skus with restricted partition key
    select * from skus where seller IN ('Decor', 'Fab') and skuid IN ('FABSKU', 'DECORSKU');

    ## To query successfully with clustering keys with EQ on skus
    select * from skus where sellerid = 'Fab' and skuid in ('FABSKU', 'FABSKU1') and timeofskucreation = '2015-12-11 12:21' and productid = 'SOFA1';

    ## To query successfully with clustering keys with IN on skus
    select * from skus where sellerid in ('Decor', 'Fab') and skuid in ('FABSKU', 'DECORSKU') and timeofskucreation IN ('2015-12-11 12:21', '2016-07-01 22:30') AND 
    productid in ('SOFA1', 'SOFA2');

    ## To query by restricting the clustering columns with range operator
    select * from skus where sellerid in ('Decor', 'Fab') and skuid in ('FABSKU', 'DECORSKU') and timeofskucreation >= '2015-08-01 12:21' AND timeofskucreation
    <= '2016-07-01 22:30'; 

    ## To allow filtering
    select * from listings where productid > 'SOFA' ALLOW FILTERING;

    ## To add a role
    create role appUser WITH password = 'password' and LOGIN = true and superuser = true;

    ## To create role dummy NoSuperuser
    create role dummy with password = 'password1' and LOGIN = true;

    ## To list roles
    list roles;

    ## To drop role dummy
    drop role dummy;
```