# Kubernetes_Kafka_Mosquitto_MongoDB
This is a rework of a previous use case of Kafka data streaming from a Mosquitto MQTT broker to a Mongo database in Kubernetes. In particular, all the collaborating servers, e.g. Kafka, Zookeeper, Mongo DB and others, are deployed on K8s pods and accessible via K8s services. <br>
<br> 
0.  Make sure the Kubernetes cluster is set up with the nodes in the Ready state. Also make sure a specific user appuser exists on both nodes.
<pre>
$ kubectl get nodes
NAME        STATUS   ROLES           AGE     VERSION
k8smaster   Ready    control-plane   3h51m   v1.25.0
k8sworker   Ready    <none>          143m    v1.25.0

$ id appuser
uid=1000(appuser) gid=1000(appuser) groups=1000(appuser)
</pre>
<br>
1. Download and unzip both the Mossquitto and Mongo Kafa connectors to a designated directory on the K8s nodes, say /var/tmp/kafka-connect/java. It will be mounted by a Kafka-connect pod afterward to make the connectors available as plugins. The directory and its contents should be owned by appuser.
<pre>
mkdir -p /var/tmp/kafka-connect/java
cd /var/tmp/kafka-connect/java

wget https://d1i4a15mxbxib1.cloudfront.net/api/plugins/confluentinc/kafka-connect-mqtt/versions/1.5.1/confluentinc-kafka-connect-mqtt-1.5.1.zip
unzip confluentinc-kafka-connect-mqtt-1.5.1.zip
wget https://d1i4a15mxbxib1.cloudfront.net/api/plugins/mongodb/kafka-connect-mongodb/versions/1.7.0/mongodb-kafka-connect-mongodb-1.7.0.zip
unzip mongodb-kafka-connect-mongodb-1.7.0.zip
</pre>
</br>
