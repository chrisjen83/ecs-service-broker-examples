###  ###

To deploy within Kubernetes a deployment which needs to consume ECS S3 bucket resources you can leverage the ECS Service Broker which has been installed onto your Kubernetes distribution

The Service Broker will automatically provision a bucket and object user then the accessKey and secretKey will be created and placed inside a Kubernetes Secret and mapped to the namespace where a deployment or pod can consume it.

To start the provisioning process follow the below steps.

### 1. Create a service instance

A service instance is a representation of an ECS bucket automatically provisioned against a published service plan.  You will need to create at least one instance (bucket) to connect to your application.

To create a bucket you will need to apply a yaml configuration similar to below.

```yaml
apiVersion: servicecatalog.k8s.io/v1beta1
kind: ServiceInstance
metadata:
  name: yaml-instance-v2
  namespace: default
spec:
  clusterServiceClassExternalName: ecs-bucket
  clusterServicePlanExternalName: 5gb
```

In the ServiceInstance yaml you will declare an instance name (name) and a kubernetes namespace (namespace) where your applications will run.  The kubernetes namespace needs to exist in the cluster.

In the spec part of the yaml you will declare the ECS cluster you want to provision a bucket in (clusterServiceClassExternalName) and select your plan (clusterServicePlanName).

To understand the ECS Clusters avalible via the broker issue the below command on your Kubernetes cluster.

```bash
root@csed204:/# kubectl get clusterserviceclass
NAME                                   EXTERNAL-NAME   BROKER                      AGE
f3cbab6a-5172-4ff1-a5c7-72990f0ce2aa   ecs-bucket      ecs-service-broker-client   4d

```

To understand the plans avaliable on your chosen ECS Cluster execute the below command.

```bash
root@csed204:/# kubectl get clusterserviceplan
NAME                                   EXTERNAL-NAME   BROKER                      CLASS                                  AGE
89d20694-9ab0-4a98-bc6a-868d6d4ecf31   unlimited       ecs-service-broker-client   f3cbab6a-5172-4ff1-a5c7-72990f0ce2aa   4d
8e777d49-0a78-4cf4-810a-b5f5173b019d   5gb             ecs-service-broker-client   f3cbab6a-5172-4ff1-a5c7-72990f0ce2aa   4d

```

Use the EXTERNAL-NAME column to select the plan to include in your serviceInstance YAML.



### Create service binding







```
 Environment:
      ecs_endpoint_url:   <set to the key 'endpoint' in secret 'yaml-instance-v2-binding'>   Optional: false
      ecs_access_key_id:  <set to the key 'accessKey' in secret 'yaml-instance-v2-binding'>  Optional: false
      ecs_secret_key:     <set to the key 'secretKey' in secret 'yaml-instance-v2-binding'>  Optional: false
      ecs_bucket_name:    <set to the key 'bucket' in secret 'yaml-instance-v2-binding'>     Optional: false
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-hs6wv (ro)

```





![](/Users/christopherjenkins/Desktop/Screen Shot 2020-06-22 at 2.41.52 pm.png)