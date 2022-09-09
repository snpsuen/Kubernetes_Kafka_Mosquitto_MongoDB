# Kubernetes_Kafka_Mosquitto_MongoDB
This is a rework of a previous use case of Kafka data streaming from a Mosquitto MQTT broker to a Mongo database in Kubernetes. All the collaborating servers are deployed on K8s pods and accessible via K8s services. <br>
<br> 
0.  Make sure the Kubernetes cluster is set up with the nodes in the Ready state.
<pre>
keyuser@k8smaster:~$ kubectl get nodes
NAME        STATUS   ROLES           AGE     VERSION
k8smaster   Ready    control-plane   3h51m   v1.25.0
k8sworker   Ready    <none>          143m    v1.25.0
</pre>
<br>
1. Download and unzip both the Mossquitto and Mongo Kafa connectors to a designated directory on the K8s nodes, say /var/tmp/kafka-connect/java. It will be mounted by a Kafka-connect pod adterwatd to use 
