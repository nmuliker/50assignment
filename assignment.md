Question : what happens after voter pod deletion?

Answer:
After voter pod deletion 
-> new voter pod is up
-> no changes in UI for voting or result
-> new votes registered


[root@ip-172-31-39-16 k8s-specifications]# kubectl delete pod vote-94849dc97-7fbbs
pod "vote-94849dc97-7fbbs" deleted
[root@ip-172-31-39-16 k8s-specifications]#  kubectl get po
NAME                      READY   STATUS    RESTARTS   AGE
db-b54cd94f4-md4m6        1/1     Running   0          35m
redis-868d64d78-mch68     1/1     Running   0          35m
result-5d57b59f4b-xvd67   1/1     Running   0          35m
vote-94849dc97-l7d58      1/1     Running   0          4s
worker-dd46d7584-fbcm2    1/1     Running   0          35m


[root@ip-172-31-39-16 k8s-specifications]# kubectl logs worker-dd46d7584-fbcm2
Waiting for db
Waiting for db
Waiting for db
Connected to db
Found redis at 10.97.13.18
Connecting to redis
Processing vote for 'a' by 'ee4bfe5ac646f91'
Processing vote for 'b' by 'ee4bfe5ac646f91'
Processing vote for 'b' by 'ee4bfe5ac646f91'
Processing vote for 'b' by '18e92d8bd02a84'
Processing vote for 'a' by '18e92d8bd02a84'
Processing vote for 'b' by '18e92d8bd02a84'
Processing vote for 'a' by '308bc7753288263'

---------------------------------------------------------------------------------------------

Question : what happens after worker pod deletion?

Answer:
After worker pod deletion
-> logs for older votes lost
-> new votes logs captured in worker node
-> UI for both voter and result working
-> older votes also accounted for


[root@ip-172-31-39-16 k8s-specifications]# kubectl delete pod worker-dd46d7584-fbcm2
pod "worker-dd46d7584-fbcm2" deleted
[root@ip-172-31-39-16 k8s-specifications]#  kubectl get po
NAME                      READY   STATUS    RESTARTS   AGE
db-b54cd94f4-md4m6        1/1     Running   0          36m
redis-868d64d78-mch68     1/1     Running   0          36m
result-5d57b59f4b-xvd67   1/1     Running   0          36m
vote-94849dc97-l7d58      1/1     Running   0          70s
worker-dd46d7584-npvj8    1/1     Running   0          9s
[root@ip-172-31-39-16 k8s-specifications]# kubectl logs worker-dd46d7584-npvj8
Connected to db
Found redis at 10.97.13.18
Connecting to redis
Processing vote for 'b' by 'ee4bfe5ac646f91'
Processing vote for 'b' by '2b0e84827e94a46'

-----------------------------------------------------------------------------------------------------

Question :  what happens after db pod deletion?

Answer:
After DB pod deletion
-> voter UI working
-> result UI showing 0 votes - no data retained as pod is stateless
-> Once the db pod was deleted there was no data stored about previous votes
-> Observed restart in result pod
-> To show vote result, have to enter votes again


[root@ip-172-31-39-16 result]# kubectl delete pod db-b54cd94f4-md4m6
pod "db-b54cd94f4-md4m6" deleted
[root@ip-172-31-39-16 result]#  kubectl get po
NAME                      READY   STATUS    RESTARTS   AGE
db-b54cd94f4-pchpg        1/1     Running   0          42s
redis-868d64d78-mch68     1/1     Running   0          45m
result-5d57b59f4b-xvd67   1/1     Running   1          45m
vote-94849dc97-l7d58      1/1     Running   0          10m
worker-dd46d7584-npvj8    1/1     Running   1          9m6s


[root@ip-172-31-39-16 result]# kubectl logs worker-dd46d7584-npvj8
Connected to db
Found redis at 10.97.13.18
Connecting to redis
Processing vote for 'a' by 'ee4bfe5ac646f91'
Processing vote for 'b' by '63ec568826d887c'
Processing vote for 'a' by '63ec568826d887c'


[root@ip-172-31-39-16 result]# kubectl logs result-5d57b59f4b-xvd67
Tue, 18 Jul 2023 06:25:20 GMT body-parser deprecated bodyParser: use individual json/urlencoded middlewares at server.js:73:9
Tue, 18 Jul 2023 06:25:20 GMT body-parser deprecated undefined extended: provide extended option at ../node_modules/body-parser/index.js:104:29
App running on port 80
Connected to db
Error performing query: error: relation "votes" does not exist
Error performing query: error: relation "votes" does not exist
Error performing query: error: relation "votes" does not exist


----------------------------------------------------------------------------------------------------------

Question : 5. Some jargons (just names are enough- dont need sentences) that you learnt from the  Training session

Answer:
-> Virtualization/Containerzation
-> Containers/Pods/Images/Layers
-> Dockerfile
-> Docker Swarm/ Dcoker Hub
-> Microservices(Polyglot, Resource allocation, patches)
-> Kubernetes setup - master worker relationship
-> Labels, namespaces
-> Tainting
-> Circuit Breaking
-> Services (ClusterIP,NodePort,Ingress, Load Balancer)
-> Deployment
-> RS, RC , DS
-> PV , PVC

------------------------------------------------------------------------------------------------------------------

Replica Set Question: Write a common use-case, where you will use a daemon set instead of replica set

Answer:
DS is used when same application needs to run in all nodes of the cluster. 
The most common use case is for logging, when each of the pod needs to have a logger instance
so then there will be exactly one pod per node.

------------------------------------------------------------------------------------------------------------------

Deployment Question: Suppose you have deployed your application using a deployment controller. 
Assume the initial number of replicas is one. 
Write the steps needed to update a container's image using deployment, making sure that there is zero downtime.

Answer:
1. Update the container image in the deployment configuration.
2. Create a new deployment with the updated image.
3. Scale up the new deployment gradually.
4. Verify the progress and test the new deployment.
5. Update the service to point to the new deployment.
6. Scale down the original deployment gradually.
7. Verify the successful rolling update.

---------------------------------------------------------------------------------------------------------------------

Service Question: You have deployed an application, that is listening at port 8000. 
You used a replica-set to deploy it and created a NodePort service to make it accessible. 
But when you test it, somehow the application is not reachable at the port. 
Write down your approach and sequentially all the steps that you will take to find out the issue.

Answer:
1. Check the yaml file used to deploy the NodePort service if it has set the port needed
2. Check the docker file if the port is exposed in docker file
3. Check for port mapping
4. Check by running curl command <ip>:8000 if its reachable

------------------------------------------------------------------------------------------------------------------------
