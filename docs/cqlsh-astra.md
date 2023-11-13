# Creation

You can create a free account, valid for 2 weeks at https://astra.datastax.com.

# Connection

To use the Astra CLI, you need to create a token with the "Organization Administrator" role: [https://astra.datastax.com/org/\<your-org-id\>/settings/tokens]()

Follow this document to install and connect to an online Astra DB https://awesome-astra.github.io/docs/pages/astra/astra-cli/, or try `curl -Ls https://dtsx.io/get-astra-cli | bash`.

Once set up and connected, you can use `$ astra db cqlsh \<name of database\> --token=AstraCS:\<token\>` to start a cqlsh.

# Working

just like [./local-usecase.md](local-usecase), but changing the import path as in the following example:

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