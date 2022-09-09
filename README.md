# Kubernetes_Kafka_Mosquitto_MongoDB
This is a rework of a previous use case of Kafka data streaming from a Mosquitto MQTT broker to a Mongo database in Kubernetes. In particular, all the collaborating servers, e.g. Kafka, Zookeeper, Mongo DB and others, are deployed on K8s pods and accessible via K8s services. <br>
<p> 
0.  Make sure the Kubernetes cluster is set up properly with the nodes in the Ready state. Also a specific user named appuser exists on both nodes.
<pre>
$ kubectl get nodes
NAME        STATUS   ROLES           AGE     VERSION
k8smaster   Ready    control-plane   3h51m   v1.25.0
k8sworker   Ready    <none>          143m    v1.25.0

$ id appuser
uid=1000(appuser) gid=1000(appuser) groups=1000(appuser)
</pre>
<p>
1. Download and unzip both the Mossquitto and Mongo Kafa connectors to a designated directory on the K8s nodes, say /var/tmp/kafka-connect/java. It will be mounted by a Kafka-connect pod afterward to make the connectors available as plugins. The directory and its contents should be owned by appuser.
<pre>
mkdir -p /var/tmp/kafka-connect/java
cd /var/tmp/kafka-connect/java

wget https://github.com/snpsuen/Kubernetes_Kafka_Mosquitto_MongoDB/raw/main/confluentinc-kafka-connect-mqtt-1.5.1.zip
unzip confluentinc-kafka-connect-mqtt-1.5.1.zip
wget https://github.com/snpsuen/Kubernetes_Kafka_Mosquitto_MongoDB/raw/main/mongodb-kafka-connect-mongodb-1.7.0.zip
unzip mongodb-kafka-connect-mongodb-1.7.0.zip
</pre>
<p>
2. Deploy MetalLB loadbancer, which 
<pre>
kubectl edit configmap -n kube-system kube-proxy
	apiVersion: kubeproxy.config.k8s.io/v1alpha1
	kind: KubeProxyConfiguration
	mode: "ipvs"
	ipvs:
	  strictARP: true <--- set to true
</pre>
<pre> kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.5/config/manifests/metallb-native.yaml </pre>

kubectl apply -f - <<END
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: myfirstpool
  namespace: metallb-system
spec:
  addresses:
  - 10.0.2.170-10.0.2.190
END

kubectl apply -f - <<END
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: example
  namespace: metallb-system
spec:
  ipAddressPools:
  - myfirstpool
END
</pre>



