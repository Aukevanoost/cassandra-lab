# How to

https://blog.digitalis.io/containerized-cassandra-cluster-for-local-testing-60d24d70dcc4

## Setup the docker nodes
```
mkdir -p etc
cp -a etc_cassandra-4.1.3_vanilla etc/cass1
cp -a etc_cassandra-4.1.3_vanilla etc/cass2
cp -a etc_cassandra-4.1.3_vanilla etc/cass3
```

## Fire up the cluster

```
docker-compose up -d
```