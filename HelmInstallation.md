## Installing the ECS Service Broker with Helm

The ECS Service broker is based on the open-source project Open Service Broker for Kubernetes. The easiest way to install the ECS Service Broker into your Kubernetes distribution is to use a local Helm installation.

This guide assumes you have Helm installed on a client machine which can access your Kubernetes Master Server, a Kubernetes cluster setup and the service catalogue pods deployed in your Kubernetes cluster.

If you have not deployed the service catalogue into your Kubernetes Cluster before, please refer to the below link on how to install Service Catalog on popular Kubernetes distributions.

[Roll you own Kubernetes]:https://kubernetes.io/docs/tasks/service-catalog/install-service-catalog-using-helm/
[ Redhat OpenShift ]: https://docs.openshift.com/container-platform/4.2/applications/service_brokers/installing-service-catalog.html



### 1. Installing ECS Service Broker

On your client machine perform the below steps:

1. Clone the ECS Service Broker GitHub repository to your local machine.

   [ Link ]:https://github.com/thecodeteam/ecs-cf-service-broker.git

2. Change directory into the now clones repository to the path **ecs-cf-service-broker/charts**.

3. Once in the charts directory you will need to cutomise the **values.yaml** file to reflect the ECS infrastructure installed on your premises. 

   Please see below the modifiable sections of the **values.yaml** file with comments on what you need to include.

   ```yaml
   namespace: "131701420476682255"	# ECS namespace FOR ALL SC created Buckets
   prefix: "kubetesting-"	# You can customise the bucket prefix to whatever you want
   replicationGroup: "ecstestdrivegeo"	# Copy the human readable name of the ECS replication group to associate with your SC created buckets
   ```

   ```yaml
   # Management SSL Custom CA Trust Certificate
   certificate: |
     -----BEGIN CERTIFICATE-----
   # Copy your ECS Managment certificate into this section.  This is essentual for connecting to your management API endpoint
     -----END CERTIFICATE-----
   ```

   ```yaml
   # ECS Object API
   api:
     name: ecs-broker m
     namespace: 131701420476682255	# Copy the same namespace as above
     endpoint: "https://FQDN_ECS_DATA_ENDPOINT"
     username: ****************	# Namespace Administrator
     password: ****************	# Namespace Administrator Password
   
   # ECS Management Endpoint
   ecsConnection:
     name: ecs-broker-connection  j
     endpoint: "https://FQDN_ECS_Mgnt_ENDPOINT"
     username: ****************	# ECS Management User
     password: ****************	# ECS Management Password
   ```

   ```yaml
   # The default ReclaimPolicy to use if one has not been explicitly specified (valid values are Fail, Detach, Delete)
   defaultReclaimPolicy: Detach	#You can alter the default delete policy when an instance has been unbound.  The options are Detach, Delete or Fail.
   ```

   Below is a complete values.yaml file ready for installation. Your **values.yaml** should look similar.

   ```yaml
   # Default values for charts.
   # This is a YAML-formatted file.
   # Declare variables to be passed into your templates.
   
   replicaCount: 1
   
   namespace: "131701420476682255"
   prefix: "kubetesting-"
   replicationGroup: "ecstestdrivegeo"
   
   # Management SSL Custom CA Trust Certificate
   certificate: |
     -----BEGIN CERTIFICATE-----
   
     -----END CERTIFICATE-----
   
   # ECS Object API
   api:
     name: ecs-broker m
     namespace: 131701420476682255
     endpoint: "https://FQDN_ECS_DATA_ENDPOINT"
     username: ****************	
     password: ****************
   
   # ECS Management Endpoint
   ecsConnection:
     name: ecs-broker-connection  j
     endpoint: "https://FQDN_ECS_Mgnt_ENDPOINT"
     username: ****************
     password: ****************
   
   image:
     repository: objectscale/ecs-service-broker
     tag: latest
     pullPolicy: Always
   
   imagePullSecrets: []
   nameOverride: ""
   fullnameOverride: ""
   
   service:
     type: ClusterIP
     port: 8080
   
   ingress:
     enabled: true
     annotations: {}
       # kubernetes.io/ingress.class: nginx
       # kubernetes.io/tls-acme: "true"
     hosts:
       - host: chart-example.local
         paths: []
   
     tls: []
     #  - secretName: chart-example-tls
     #    hosts:
     #      - chart-example.local
   
   resources: {}
     # We usually recommend not to specify default resources and to leave this as a conscious
     # choice for the user. This also increases chances charts run on environments with little
     # resources, such as Minikube. If you do want to specify resources, uncomment the following
     # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
     # limits:
     #   cpu: 100m
     #   memory: 128Mi
     # requests:
     #   cpu: 100m
     #   memory: 128Mi
   
   nodeSelector: {}
   
   tolerations: []
   
   affinity: {}
   
   # The default ReclaimPolicy to use if one has not been explicitly specified (valid values are Fail, Detach, Delete)
   defaultReclaimPolicy: Detach
   
   # Indicates this should be registered as a ServiceCatalog Broker
   serviceCatalog: true
   
   ```

   4. Staying in the same directory **ecs-cf-service-broker/charts** you are now ready to install the ECS Serivce Broker via Helm.

      Issue the following command at the terminal prompt to start the install.

      ```bash
      root@csed204: helm install -f values.yaml ecs-service-broker ./charts
      ```



5. When Helm returns an installation complete, you can now inspect your Kubernetes Cluster to see what state the ecs-service-broker pod is in.

   Issue the below command to see, please note that for **-n** you need to specify the namespace you have installed the service broker into.

   ```bash
   root@csed204:~/ecs-cf-service-broker/charts# kubectl get pods -n catalog
   NAME                                                  READY   STATUS    RESTARTS   AGE
   catalog-catalog-controller-manager-75fffdcf57-b6bf9   1/1     Running   1          28d
   catalog-catalog-webhook-7d8497cdf6-tszz6              1/1     Running   0          28d
   ecs-service-broker-76f565ff84-mjr6m                   1/1     Running   0          16d
   
   ```

   Once the **ecs-service-broker-<UUID>** is in the running state you have then completed the Helm installation of ECS Service Broker.