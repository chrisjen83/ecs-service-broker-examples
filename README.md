###  

## Using the ECS Service Broker inside Kubernetes YAML Files

To deploy a micro-services deployment which needs to consume Dell Technologies ECS S3 bucket resources, you can leverage the ECS Service Broker who has been installed on your Kubernetes distribution.
The Service Broker will automatically provision a bucket and object user then the accessKey and secretKey will be created and placed inside a Kubernetes Secret and mapped to the namespace where deployment or pod can consume.

To start the provisioning process, follow the below steps.

### 1. Create a service instance

A service instance is a representation of an ECS bucket automatically provisioned against a published service plan. You will need to create at least one instance (bucket) to connect to your application.

To create an instance, you will need to apply a YAML configuration similar to below.

```yaml
apiVersion: servicecatalog.k8s.io/v1beta1
kind: ServiceInstance
metadata:
  name: yaml-instance-v2
  namespace: default
spec:
  clusterServiceClassExternalName: ecs-bucket
  clusterServicePlanExternalName: 5gb
  parameters:
   reclaim-policy: Delete
```

In the ServiceInstance YAML, you will declare an instance name (name) and a Kubernetes namespace (namespace) where your applications will run. The Kubernetes namespace needs to exist in the cluster before running this YAML.

In the spec section of the YAML, you will declare the ECS cluster you want to provision a bucket in (clusterServiceClassExternalName) and select your plan (clusterServicePlanName).

After running the Service Instance YAML, you will have a bucket created on the ECS Cluster, but there will be no access to the bucket. To gain access, you will need to bind your created instance to your namespace. Follow section two to complete an instance bind.

#### Note

To understand the ECS Clusters available via the Service Broker issue the below command on your Kubernetes cluster.

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

Use the EXTERNAL-NAME column to select the plan to include in your serviceInstance YAML. If there is a plan which is not listed, you will need to contact your administration team.

In the ServiceInstance YAML, there are a few custom parameters which can be applied to your bucket. These parameters deal with reclaim policies, Access During Outage and bucket encryption. These parameters are unique to the ECS object storage solution. Below is a list of optional parameters which can be included in the ServiceInstance YAML

- reclaim-policy: Delete, Reclaim, Fail

### 2. Create service binding

Binding an Instance to a Kubernetes cluster will tell the service broker to create an object user with full control rights to the instance (bucket) and then create a secret configuration and place the secret into the namespace you configured in the Instance creation step.

Below is the YAML structure you will use to initiate binding.

```yaml
kind: ServiceBinding
metadata:
  name: yaml-instance-v2-binding
  namespace: default
spec:
  instanceRef:
    name: yaml-instance-v2
  secretName: shhh-my-secret  
```

In the metadata name line, you will add a name to identify your binding, in the namespace line add in the Kubernetes namespace which you want to use the secret in or where your application resides. The namespace has to exist at the time of running this YAML.

In spec, instanceeRef the name line needs to match the instance you created in the previous YAML, this is so the binding know which bucket to create the object user for. In the secretName this allows you to apply a custom name to your Kubernetes secret file.

Below is a sample Kubernetes secret file. To use this secret file inside a pod you have two options:

- Import the secret data as individual enviroment variables into your pod.
- Mount the secret file as a volume definition in your pod YAML file.

For more information please refer to [link]:https://kubernetes.io/docs/concepts/configuration/secret/



```bash
root@csed204:~# kubectl describe secret shhh-my-secret
Name:         shhh-my-secret
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
accessKey:          48 bytes
bucket:             48 bytes
endpoint:           25 bytes
path-style-access:  4 bytes
s3Url:              166 bytes
secretKey:          40 bytes

```



Below are the JSON Path for all of the areas of the secret file.

- {.data.accessKey}
- {.data.secretKey}
- {.data.bucket}
- {.data.endpoint}
- {.data.s3Url}
- {.data.path-style-access}

To decode the base64 encoded secrets use the below example and subsittute the JSON path fields.

``` bash
root@csed204:~# kubectl get secrets <SECRET_NAME> -o jsonpath='{.data.accessKey}' -n <NAMESPACE> | base64 -d
```