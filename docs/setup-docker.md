Lost? go [back](./../readme.md)

# 1. Setup the cluster

Again based on the [Cassandra docker tutorial](https://blog.digitalis.io/containerized-cassandra-cluster-for-local-testing-60d24d70dcc4). Slightly modified however to support the latest stable version of Cassandra. 

Follow the steps below to setup the cluster:

## 1.1 Create config folders

```
mkdir -p etc
cp -a etc_cassandra-4.1.3_vanilla etc/cass1
cp -a etc_cassandra-4.1.3_vanilla etc/cass2
cp -a etc_cassandra-4.1.3_vanilla etc/cass3
```

## 1.2 Fire up the cluster

```
docker-compose up -d
```

## 1.3 Enter the cluster CQLSH

```
docker exec -it cass1 cqlsh --connect-timeout="60" --request-timeout="60"
```

## 1.4 Adding the datasets to the cluster

You can find all of the datasets in the following [Google drive link](https://drive.google.com/drive/folders/1WYybo02Xz36sh7Fi5gcpUIEcjlE-E-nD?usp=sharing).

After downloading the datasets you can move the files to your mount link (data folder). The mount link will be used as absolute link to load the data from the CSV files into the cluster.

You can find the files in the node under the url: <br />
`/var/lib/cassandra/recommendation_one/{target_file}_{target_country}.csv` <br /> or <br /> `/var/lib/cassandra/recommendation_three/{target_file}_{target_country}.csv`


## 1.5 Stopping the cluster
```
docker-compose stop
```
