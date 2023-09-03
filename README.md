# MongoDb Replica Set

- Useful Links
- https://hub.docker.com/_/mongo
- https://www.mongodb.com/docs/v4.4/

- MongoAtlas Live Migration
- https://www.mongodb.com/docs/atlas/migration-from-com/

- https://docs.docker.com/engine/reference/commandline/network_connect
- Connect a running container to a network🔗
```bash
docker network connect <network> <container>
```

- create a network
```bash
docker network create poc --attachable
```

- When you enable replication and security, MongoDB expects you to provide a shared key file that each member uses to authenticate to other members in the replica set. This key file must be the same for all members.
- Create a keyfile
```bash
openssl rand -base64 756 > ${PWD}/mongo-keyfile
chmod 400 ${PWD}/mongo-keyfile
chown 999:999 ${PWD}/mongo-keyfile
```


- Note: mongosh will not be available in this 4.4.minor version

- Container 1
```bash
docker run -id --network poc --name mongo-node1 \
    -v ${PWD}/mongo-keyfile:/etc/mongo/mongo-keyfile \
    -v mongo-node1-config:/etc/mongo \
    -v mongo-node1-data:/data/db \
    -e MONGO_INITDB_ROOT_USERNAME=mongoadmin \
    -e MONGO_INITDB_ROOT_PASSWORD=secret \
    mongo:4.4.23 --replSet myrs --keyFile /etc/mongo/mongo-keyfile
```

- Container 2
```bash
docker run -id --network poc --name mongo-node2 \
    -v ${PWD}/mongo-keyfile:/etc/mongo/mongo-keyfile \
    -v mongo-node2-config:/etc/mongo \
	-v mongo-node2-data:/data/db \
	-e MONGO_INITDB_ROOT_USERNAME=mongoadmin \
	-e MONGO_INITDB_ROOT_PASSWORD=secret \
	mongo:4.4.23 --replSet myrs --keyFile /etc/mongo/mongo-keyfile
```

- Container 3
```bash
docker run -id --network poc --name mongo-node3 \
    -v ${PWD}/mongo-keyfile:/etc/mongo/mongo-keyfile \
    -v mongo-node3-config:/etc/mongo \
	-v mongo-node3-data:/data/db \
	-e MONGO_INITDB_ROOT_USERNAME=mongoadmin \
	-e MONGO_INITDB_ROOT_PASSWORD=secret \
	mongo:4.4.23 --replSet myrs --keyFile /etc/mongo/mongo-keyfile
```

- Initiate the Replica Set:
- Connect to the primary container or any container
```bash
docker exec -it mongo-node1 mongo -u mongoadmin -p secret
```

- login to mongo
```bash
mongo -u mongoadmin -p secret --authenticationDatabase admin
```
- Initiate the Replica Set:
```bash
rs.initiate({
   _id: "myrs",
   members: [
       { _id: 0, host: "mongo-node1:27017" },
       { _id: 1, host: "mongo-node2:27017" },
       { _id: 2, host: "mongo-node3:27017" }
   ]
});
```
- to retrieve the status of the replica set
```bash
rs.status() 
```

- Import Collection
```bash
git clone https://github.com/neelabalan/mongodb-sample-dataset.git

docker cp listingsAndReviews.json main-mongo-source:/listingsAndReviews.json

mongoimport --host main-mongo-source --db poc --collection collectionName --file listingsAndReviews.json \
--username mongoadmin --password secret --authenticationDatabase admin
```

- Mongo Express
```bash
docker run --name mongo-express --network poc \
-p 8081:8081 \
--restart unless-stopped \
-e ME_CONFIG_MONGODB_ADMINUSERNAME='mongoadmin' \
-e ME_CONFIG_MONGODB_ADMINPASSWORD='secret' \
-e ME_CONFIG_MONGODB_URL='mongodb://mongoadmin:secret@main-mongo-source:27017/' \
-id mongo-express:latest
```
- All Volumes Remove
```bash
docker volume ls -q | xargs docker volume rm
```
- All Containers Removed
```bash
docker rm -f $(docker ps -aq)
```

- MongoDB Shell
```bash
mongo "mongodb+srv://USERNAME:PASSWORD@ENDPOINT/DATABASENAME?readPreference=secondary"

mongosh "mongodb+srv://USERNAME:PASSWORD@ENDPOINT/DATABASENAME?readPreference=secondary"
```
  
- 1. List Databases: To list databases, use the command
```bash
 show dbs
```
- 2. List Collections:
```bash
use your_database_name
```

- 3. show collections

- AWS EC2 HA Mode

- aws private hosted zone policy
```bash
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowChangeResourceRecordSets",
      "Effect": "Allow",
      "Action": [
        "route53:ChangeResourceRecordSets",
        "route53:ListResourceRecordSets"
      ],
      "Resource": "arn:aws:route53:::hostedzone/YOUR_PRIVATE_HOSTED_ZONE_ID"
    },
    {
      "Sid": "AllowViewHostedZone",
      "Effect": "Allow",
      "Action": [
        "route53:GetHostedZone",
        "route53:ListHostedZones"
      ],
      "Resource": "*"
    }
  ]
}
```

- When you enable replication and security, MongoDB expects you to provide a shared key file that each member uses to authenticate to other members in the replica set. This key file must be the same for all members.
- In case of multiple aws ec2 Create a keyfile locally and copy to all the nodes
```bash
openssl rand -base64 756 > ${PWD}/mongo-keyfile
chmod 400 ${PWD}/mongo-keyfile
chown 999:999 ${PWD}/mongo-keyfile
```


- Note: mongosh will not be available in this 4.4.minor version

- Container 1
```bash
docker run -id --network host --name mongo-node1 \
    --hostname mongo-node1.cloudgeeks.mongo \
    -v ${PWD}/mongo-keyfile:/etc/mongo/mongo-keyfile \
    -v mongo-node1-config:/etc/mongo \
    -v mongo-node1-data:/data/db \
    -e MONGO_INITDB_ROOT_USERNAME=mongoadmin \
    -e MONGO_INITDB_ROOT_PASSWORD=secret \
    mongo:4.4.23 --replSet myrs --keyFile /etc/mongo/mongo-keyfile
```

- Container 2
```bash
docker run -id --network host --name mongo-node2 \
    --hostname mongo-node2.cloudgeeks.mongo \
    -v ${PWD}/mongo-keyfile:/etc/mongo/mongo-keyfile \
    -v mongo-node2-config:/etc/mongo \
	-v mongo-node2-data:/data/db \
	-e MONGO_INITDB_ROOT_USERNAME=mongoadmin \
	-e MONGO_INITDB_ROOT_PASSWORD=secret \
	mongo:4.4.23 --replSet myrs --keyFile /etc/mongo/mongo-keyfile
```

- Container 3
```bash
docker run -id --network host --name mongo-node3 \
    --hostname mongo-node3.cloudgeeks.mongo \
    -v ${PWD}/mongo-keyfile:/etc/mongo/mongo-keyfile \
    -v mongo-node3-config:/etc/mongo \
	-v mongo-node3-data:/data/db \
	-e MONGO_INITDB_ROOT_USERNAME=mongoadmin \
	-e MONGO_INITDB_ROOT_PASSWORD=secret \
	mongo:4.4.23 --replSet myrs --keyFile /etc/mongo/mongo-keyfile
```
- login to mongo
```bash
mongo -u mongoadmin -p secret --authenticationDatabase admin
```
- Initiate the Replica Set:
```bash
rs.initiate({
   _id: "myrs",
   members: [
       { _id: 0, host: "mongo-node1.cloudgeeks.mongo:27017" },
       { _id: 1, host: "mongo-node2.cloudgeeks.mongo:27017" },
       { _id: 2, host: "mongo-node3.cloudgeeks.mongo:27017" }
   ]
});
```

- to retrieve the status of the replica set
```bash
rs.status() 
```


- Kubernetes Bitnami Helm Chart Deployment
```bash
helm repo ls
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo bitnami
helm search repo bitnami/mongodb
helm search repo bitnami/mongodb --versions
helm show values bitnami/mongodb --version 13.18.1
helm upgrade --install mongodb bitnami/mongodb \
--namespace mongodb \
--create-namespace \
--set architecture=replicaset \
--set auth.enabled=true \
--set auth.rootPassword=rootpassword \
--set auth.username=my-user \
--set auth.password=my-password \
--set auth.database=my-database \
--set replicaSet.enabled=true \
--set replicaSet.replicas.secondary=2 \
--set persistence.size=50Gi \
--wait --timeout 600s
```

- 3 Worker Nodes (--set replicaSet.replicas.secondary=2 and mongodb-arbiter-0 )

- mongodb-arbiter
```bash
The MongoDB arbiter is part of a MongoDB replica set, and its primary role is to participate in elections to decide which member of the replica set will become the primary. Arbiters do not hold any data and are not involved in regular read and write operations. Because they do not hold a copy of the data set, they use fewer system resources.
``````
```bash
kubectl get pods -n mongodb -o wide
NAME                READY   STATUS    RESTARTS   AGE     IP            NODE                 NOMINATED NODE   READINESS GATES
mongodb-0           1/1     Running   0          3m7s    10.244.2.13   cloudgeeks-worker    <none>           <none>
mongodb-1           1/1     Running   0          2m49s   10.244.3.12   cloudgeeks-worker3   <none>           <none>
mongodb-arbiter-0   1/1     Running   0          3m7s    10.244.1.5    cloudgeeks-worker2   <none>           <none>
```

