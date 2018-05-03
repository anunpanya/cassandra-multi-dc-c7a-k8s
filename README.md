# multi-dc-db-k8s

These instructions are tested on Kubernetes version 1.8.4 and 1.8.5. StatefulSets in 1.8.x are in beta. In Kubernetes version 1.9, StatefulSets are stable GA. While same instructions work for 1.9, you are encouraged to change the StatefulSet api version to stable (apps/v1) or use the yaml files from 1.9 folder.

kubectl get nodes
<pre>
NAME       STATUS    ROLES     AGE       VERSION
kubenode1   Ready     master    7m        v1.8.4
kubenode2   Ready     <none>    3m        v1.8.4
</pre>

# kubectl label nodes kubenode1 dc=DC1
node "kubenode1" labeled
# kubectl label nodes kubenode2 dc=DC2
node "kubenode2" labeled


# kubectl get nodes --show-labels
<pre>
NAME       STATUS    ROLES     AGE       VERSION   LABELS
kubenode1   Ready     master    12m       v1.8.4    beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,dc=DC1,kubernetes.io/hostname=kubenode1,node-role.kubernetes.io/master=
kubenode2   Ready     <none>    8m        v1.8.4    beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,dc=DC2,kubernetes.io/hostname=kubenode2
</pre>

# kubectl create namespace db

#git clone https://github.com/ideagw/cassandra-multi-dc-db-k8s.git

cd mu*

# kubectl -n db create -f service.yaml
<pre>
service "cassandraa" created
service "cassandrab" created
</pre>

# kubectl -n db create -f local_pvs.yaml
<pre>
persistentvolume "cassandra-data-a" created
persistentvolume "cassandra-data-b" created
persistentvolume "cassandra-data-c" created
persistentvolume "cassandra-data-d" created
persistentvolume "cassandra-data-e" created
persistentvolume "cassandra-data-f" created
</pre>

# kubectl -n db create -f statefulset-a.yaml
<pre>
statefulset "cassandraa" created
</pre>

# kubectl -n db get pods
<pre>
NAME           READY     STATUS              RESTARTS   AGE
cassandraa-0   0/1       ContainerCreating   0          59s
</pre>

# kubectl -n db describe pod cassandraa-0

# kubectl -n db get pods
<pre>
NAME           READY     STATUS    RESTARTS   AGE
cassandraa-0   1/1       Running   0          1m
</pre>

# kubectl -n db create -f statefulset-b.yaml

# kubectl -n db get statefulsets
<pre>
NAME         DESIRED   CURRENT   AGE
cassandraa   1         1         3m
cassandrab   1         1         23s
</pre>

# kubectl -n db get pods -o wide
<pre>
NAME           READY     STATUS    RESTARTS   AGE       IP          NODE
cassandraa-0   1/1       Running   0          7m        10.32.0.3   kubenode1
cassandrab-0   1/1       Running   0          7s        10.44.0.1   kubenode2
</pre>

# kubectl -n db exec -ti cassandraa-0 -- nodetool status
<pre>
Datacenter: DC1
===============
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address    Load       Tokens       Owns (effective)  Host ID                               Rack
UN  10.32.0.3  82.34 KiB  256          100.0%            0e7af033-e505-4a98-b7ea-7c52b5fb07f7  Rack1
Datacenter: DC2
===============
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address    Load       Tokens       Owns (effective)  Host ID                               Rack
UN  10.44.0.1  106.36 KiB  256          100.0%            bc58184e-bc4a-4214-9e31-98328ea6f0b3  Rack1
</pre>

# kubectl -n db logs cassandraa-0

# kubectl -n db scale --replicas=3 statefulset/cassandraa
statefulset "cassandraa" scaled

# kubectl -n db get pods -o wide
<pre>
NAME           READY     STATUS    RESTARTS   AGE       IP          NODE
cassandraa-0   1/1       Running   0          13m       10.32.0.3   kubenode1
cassandraa-1   1/1       Running   0          15s       10.32.0.4   kubenode1
cassandraa-2   1/1       Running   0          11s       10.32.0.5   kubenode1
cassandrab-0   1/1       Running   0          6m        10.44.0.1   kubenode2
</pre>

# kubectl -n db scale --replicas=3 statefulset/cassandrab
statefulset "cassandrab" scaled

# kubectl -n db get pods -o wide
<pre>
NAME           READY     STATUS    RESTARTS   AGE       IP          NODE
cassandraa-0   1/1       Running   0          15m       10.32.0.3   kubenode1
cassandraa-1   1/1       Running   0          1m        10.32.0.4   kubenode1
cassandraa-2   1/1       Running   2          1m        10.32.0.5   kubenode1
cassandrab-0   1/1       Running   0          8m        10.44.0.1   kubenode2
cassandrab-1   1/1       Running   0          53s       10.44.0.2   kubenode2
cassandrab-2   1/1       Running   0          49s       10.44.0.3   kubenode2
</pre>

# kubectl -n db exec -ti cassandraa-0 -- nodetool status
<pre>
Datacenter: DC1
===============
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address    Load       Tokens       Owns (effective)  Host ID                               Rack
UN  10.32.0.3  87.29 KiB  256          32.6%             0e7af033-e505-4a98-b7ea-7c52b5fb07f7  Rack1
UN  10.32.0.4  106.08 KiB  256          32.0%             5d2f8d97-7c29-4c9e-986c-8590e9c8b790  Rack1
UN  10.32.0.5  163.95 KiB  256          35.8%             a1908f1b-1da6-4366-984a-7daf6773cd29  Rack1
Datacenter: DC2
===============
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address    Load       Tokens       Owns (effective)  Host ID                               Rack
UN  10.44.0.1  106.36 KiB  256          34.5%             bc58184e-bc4a-4214-9e31-98328ea6f0b3  Rack1
UN  10.44.0.2  103.29 KiB  256          32.3%             0292512e-f698-458b-b285-a323a7472127  Rack1
UN  10.44.0.3  84.25 KiB  256          32.9%             254ce3d5-455a-4176-a8c1-0468ba8baefe  Rack1
</pre>

# kubectl -n db get ep
<pre>
NAME         ENDPOINTS                                      AGE
cassandraa   10.32.0.3:9042,10.32.0.4:9042,10.32.0.5:9042   2m
cassandrab   10.44.0.1:9042,10.44.0.2:9042,10.44.0.3:9042   2m
</pre>

# kubectl -n db get pvc
<pre>
NAME                          STATUS    VOLUME             CAPACITY   ACCESS MODES   STORAGECLASS   AGE
cassandra-data-cassandraa-0   Bound     cassandra-data-a   10Gi       RWO                           21m
cassandra-data-cassandraa-1   Bound     cassandra-data-b   10Gi       RWO                           7m
cassandra-data-cassandraa-2   Bound     cassandra-data-f   10Gi       RWO                           7m
cassandra-data-cassandrab-0   Bound     cassandra-data-d   10Gi       RWO                           18m
cassandra-data-cassandrab-1   Bound     cassandra-data-c   10Gi       RWO                           6m
cassandra-data-cassandrab-2   Bound     cassandra-data-e   10Gi       RWO                           6m
</pre>

# kubectl -n db exec -ti cassandraa-0 -- cqlsh
<pre>
Connected to Cassandra at 127.0.0.1:9042.
[cqlsh 5.0.1 | Cassandra 3.11.1 | CQL spec 3.4.4 | Native protocol v4]
Use HELP for help.
cqlsh> create keyspace hr_keyspace with replication ={'class' : 'NetworkTopologyStrategy', 'DC1':2, 'DC2':2};
cqlsh> use hr_keyspace;
cqlsh> CREATE TABLE employee( emp_id int PRIMARY KEY, emp_name text, emp_city text, emp_sal varint, emp_phone varint);
#For asynchronous writes to other data center, set the consistency level to LOCAL_QUORUM
cqlsh:hr_keyspace> consistency LOCAL_QUORUM
Consistency level set to LOCAL_QUORUM.
cqlsh:hr_keyspace> 
cqlsh:hr_keyspace> consistency     
Current consistency level is LOCAL_QUORUM.
cqlsh:hr_keyspace> 


cqlsh:hr_keyspace> INSERT INTO employee (emp_id, emp_name, emp_city,emp_sal,emp_phone) VALUES(1,'David', 'San Francisco', 50000, 983210987);
cqlsh:hr_keyspace> INSERT INTO employee (emp_id, emp_name, emp_city,emp_sal,emp_phone) VALUES(2,'Robin', 'San Jose', 55000, 9848022339); cqlsh:hr_keyspace> INSERT INTO employee (emp_id, emp_name, emp_city,emp_sal,emp_phone) VALUES(3,'Bob', 'Austin', 45000, 9848022330);
cqlsh:hr_keyspace> INSERT INTO employee (emp_id, emp_name, emp_city,emp_sal,emp_phone) VALUES(4, 'Monica','San Jose', 55000, 9458022330);
cqlsh:hr_keyspace> 
cqlsh:hr_keyspace> 
cqlsh:hr_keyspace> select * from employee;

 emp_id | emp_city      | emp_name | emp_phone  | emp_sal
--------+---------------+----------+------------+---------
      1 | San Francisco |    David |  983210987 |   50000
      2 |      San Jose |    Robin | 9848022339 |   55000
      4 |      San Jose |   Monica | 9458022330 |   55000
      3 |        Austin |      Bob | 9848022330 |   45000

cqlsh:hr_keyspace> quit;
root@kubenode1:/home/ubuntu/multi-dc-db-k8s# 
root@kubenode1:/home/ubuntu/multi-dc-db-k8s# 
root@kubenode1:/home/ubuntu/multi-dc-db-k8s# 
root@kubenode1:/home/ubuntu/multi-dc-db-k8s# kubectl -n db exec -ti cassandrab-1 -- cqlsh
Connected to Cassandra at 127.0.0.1:9042.
[cqlsh 5.0.1 | Cassandra 3.11.1 | CQL spec 3.4.4 | Native protocol v4]
Use HELP for help.
cqlsh> 
cqlsh> 
cqlsh> use hr_keyspace;
cqlsh:hr_keyspace> 
cqlsh:hr_keyspace> select * from employee;

 emp_id | emp_city      | emp_name | emp_phone  | emp_sal
--------+---------------+----------+------------+---------
      1 | San Francisco |    David |  983210987 |   50000
      2 |      San Jose |    Robin | 9848022339 |   55000
      4 |      San Jose |   Monica | 9458022330 |   55000
      3 |        Austin |      Bob | 9848022330 |   45000

(4 rows)
cqlsh:hr_keyspace> 


</pre>
# Simulate site failure
# kubectl -n db delete statefulset cassandraa
statefulset "cassandraa" deleted

# kubectl -n db exec -ti cassandrab-1 -- cqlsh
<pre>
Connected to Cassandra at 127.0.0.1:9042.
[cqlsh 5.0.1 | Cassandra 3.11.1 | CQL spec 3.4.4 | Native protocol v4]
Use HELP for help.
cqlsh> use hr_keyspace;
cqlsh:hr_keyspace> select * from employee;

 emp_id | emp_city      | emp_name | emp_phone  | emp_sal
--------+---------------+----------+------------+---------
      1 | San Francisco |    David |  983210987 |   50000
      2 |      San Jose |    Robin | 9848022339 |   55000
      4 |      San Jose |   Monica | 9458022330 |   55000
      3 |        Austin |      Bob | 9848022330 |   45000

(4 rows)
cqlsh:hr_keyspace> quit;
</pre>

# kubectl -n db get pods -o wide
<pre>
NAME           READY     STATUS    RESTARTS   AGE       IP          NODE
cassandrab-0   1/1       Running   0          1h        10.44.0.1   kubenode2
cassandrab-1   1/1       Running   0          1h        10.44.0.2   kubenode2
cassandrab-2   1/1       Running   0          1h        10.44.0.3   kubenode2
</pre>
