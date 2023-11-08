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
docker exec -it cass1 cqlsh
```

## 1.4 Stopping the cluster
```
docker-compose stop
```
