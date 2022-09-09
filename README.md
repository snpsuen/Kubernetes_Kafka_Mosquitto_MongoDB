# Kubernetes_Kafka_Mosquitto_MongoDB
This is a rework of a previous use case of Kafka data streaming from a Mosquitto MQTT broker to a Mongo database in Kubernetes. In particular, all the collaborating servers, e.g. Kafka, Zookeeper, Mongo DB and others, are deployed on K8s pods and accessible via K8s services. <br>
<p>
  (0) Make sure the Kubernetes cluster is set up properly with the nodes in the Ready state. Also a specific user named appuser exists on both nodes.
  
  ~~~
  $ kubectl get nodes -o wide
  NAME        STATUS   ROLES           AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
  k8smaster   Ready    control-plane   25h   v1.25.0   10.0.2.220    <none>        Ubuntu 20.04.1 LTS   5.4.0-125-generic   containerd://1.5.9
  k8sworker   Ready    <none>          24h   v1.25.0   10.0.2.222    <none>        Ubuntu 20.04.1 LTS   5.4.0-125-generic   containerd://1.5.9
  
  $ id appuser
  uid=1000(appuser) gid=1000(appuser) groups=1000(appuser)
  ~~~
  
<p>
  (1) Download and unzip both the Mossquitto and Mongo Kafa connectors to a designated directory on the K8s nodes, say /var/tmp/kafka-connect/java. It will be mounted by a Kafka-connect pod afterward to make the connectors available as plugins. The directory and its contents should be owned by appuser.
  
  ~~~
  mkdir -p /var/tmp/kafka-connect/java
  cd /var/tmp/kafka-connect/java

  wget https://github.com/snpsuen/Kubernetes_Kafka_Mosquitto_MongoDB/raw/main/confluentinc-kafka-connect-mqtt-1.5.1.zip
  unzip confluentinc-kafka-connect-mqtt-1.5.1.zip
  wget https://github.com/snpsuen/Kubernetes_Kafka_Mosquitto_MongoDB/raw/main/mongodb-kafka-connect-mongodb-1.7.0.zip
  unzip mongodb-kafka-connect-mongodb-1.7.0.zip
  ~~~

<p>
(2) Deploy MetalLB loadbancer, which will listen at VIPs assigned from the address pool 10.0.2.170-10.0.2.190.
    
    ~~~
    kubectl edit configmap -n kube-system kube-proxy
  apiVersion: kubeproxy.config.k8s.io/v1alpha1
  kind: KubeProxyConfiguration
  mode: "ipvs"
  ipvs:
    strictARP: true <--- set to true

kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.5/config/manifests/metallb-native.yaml

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
  ~~~
                        
<p>
    (3) Deploy Mosquitto MQTT broker.
    
    ~~~
kubectl apply -f https://raw.githubusercontent.com/snpsuen/Kubernetes_Kafka_Mosquitto_MongoDB/main/mosquitto.yaml
    ~~~
    
<p>
(4) Deploy Kafka Zookeeper. We will start with one zookeeper pod in this case.

~~~
kubectl apply -f https://raw.githubusercontent.com/snpsuen/Kubernetes_Kafka_Mosquitto_MongoDB/main/zookeeper.yaml
~~~
<p>
(5) Deploy Kafka broker. We will start with one Kafka pod in this case. There are two important points to note.
  
1. Create a sub-directory in the path /var/tmp/kafa/data and assign its ownership to appuser (uid 1000). It will be mounted by the Kafaka broker afterward.
2. Avoid using the name "kafka" to denote the K8s service for the Kafka pod. For example, the service is named "kafka-svc" here. Otherwise, the pod will fail to run and produce this error message "port is deprecated. Please use KAFKA_ADVERTISED_LISTENERS instead."
  
~~~
sudo mkdir -p /var/tmp/kafka/data
sudo chown -R 1000:1000 /var/tmp/kafka/data

kubectl apply -f https://raw.githubusercontent.com/snpsuen/Kubernetes_Kafka_Mosquitto_MongoDB/main/kafka.yaml
~~~
<p> 
  (6) Deploy Kafka connect
  ~~~
  kubectl apply -f https://raw.githubusercontent.com/snpsuen/Kubernetes_Kafka_Mosquitto_MongoDB/main/kafka-connect.yaml
  ~~~





