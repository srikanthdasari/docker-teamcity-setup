# Teamcity setup with docker stack

### NOTE: using the mysql as backend for teamcity

#### step 1: create the network
```sh
docker network create --driver overlay appnet
```
#### step 2: creat the secrets
```sh
echo "password123" | docker secret create db_root_password -
echo "password456" | docker secret create db_dba_password -
```
#### step 3: create the below volume folder in the host (if you are replicating in other workers, make sure this structure is in workers as well) 
```sh
./data/server/datadir
./data/server/logs
./data/agent/
```
#### step 4: deploy the docker stack
```sh 
docker stack deploy -c ci-stack.yml ci
```
#### step 5: make sure your services are up and running
```sh
docker stack ps ci
```
```sh
ID                  NAME                  IMAGE                              NODE                DESIRED STATE       CURRENT STATE               
tiqs09jezgbv        ci_teamcity.1         jetbrains/teamcity-server:latest   manager             Running             Running about an hour ago
2wodvnxzzak8        ci_viz.1              dockersamples/visualizer:latest    manager             Running             Running about an hour ago
8sqiq8zl4lp4        ci_db.1               mysql:latest                       manager             Running             Running about an hour ago
jxffrtfsk6p0        ci_teamcity-agent.1   jetbrains/teamcity-agent:latest    worker2             Running             Running about an hour ago
ndffqvyo0mab        ci_teamcity-agent.2   jetbrains/teamcity-agent:latest    worker1             Running             Running about an hour ago
```
#### step 6: Check your database access (if you want)
```sh
docker exec -it $(docker ps -f name=ci_db.1 -q) mysql -u root -p
```
#### step 7: access the visulize container 
```sh
http://{node ip}:8090
```
#### step 8: access the teamcity server
```sh
http://{node ip}:8111
```
#### step 9: during the teamcity initial setup (jdbc driver for mysql), this needs to be done once the docker services (teamcity) up
```sh
sudo wget "https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-8.0.13.tar.gz" -O mysql-connector-java-8.0.13.tar.gz
sudo tar -xzvf mysql-connector-java-8.0.13.tar.gz 
sudo cp ./mysql-connector-java-8.0.13.jar ./data/server/datadir/lib/jdbc/
```
