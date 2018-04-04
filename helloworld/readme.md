# 1 - Hello World

Often when I attempt to reuse examples from github and other sources they tend to be complex and break in some small way that is difficult to determine. For this reason it is useful to start with the simplest case and build upon it.

### Step 1 - Create (simple) Sonar yaml

Getting Sonar launched with nothing but its required port exposed is a good starting point. Create a sonarqube-deployment.yaml file with the following contents:

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: sonar
spec:
  replicas: 1
  template:
    metadata:
      name: sonar
      labels:
        name: sonar
    spec:
      containers:
        - image: sonarqube:latest
          name: sonar
          ports:
            - containerPort: 9000
              name: sonar
```

### Step 2 - Launch Sonar on Kubernetes

A quick kubectl command will launch Sonar.

```
kubectl create -f sonarqube-deployment.yaml
```

### Step 3 - Verify Sonar on Kubernetes

Lets make sure the deployment is working before exposing it. The first step is to verify its running.

```
kubectl get pods
```

Which for me returned:

```
NAME                     READY     STATUS    RESTARTS   AGE
sonar-5d94d5d5bd-hm7f7   1/1       Running   1          1h
```

Great, lets check the logs to make sure it looks good. kubectl logs <pod name>. Here is my command:

```
kubectl logs sonar-5d94d5d5bd-hm7f7
```

The last line returned was reassuring and implies Sonar is probably running fine.

```
app[][o.s.a.SchedulerImpl] SonarQube is up
```

### Step 4 - Create Sonar service yaml on Kubernetes

#### Locally with Minikube

Now that we feel good about Sonar, lets expose it. If we are running it locally with [minikube](https://github.com/kubernetes/minikube) we will need to do so with [NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport). Create a sonarqube-nodeservice.yaml that looks something like this:

```
apiVersion: v1
kind: Service
metadata:
  labels:
    name: sonar
  name: sonar
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 9000
      name: sonarport
  selector:
    name: sonar
```

#### Public Cloud

If you happen to be going straight to public cloud, you can use type [LoadBalancer](https://kubernetes.io/docs/concepts/services-networking/service/#type-loadbalancer). This is what a sonarqube-lbservice.yaml would look like in this case:

```
apiVersion: v1
kind: Service
metadata:
  labels:
    name: sonar
  name: sonar
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 9000
      name: sonarport
  selector:
    name: sonar
```

### Step 5 - Launch service

This will be straight forward as well. Choose the appropriate service yaml (nodeservice for minikube/lbservice for public cloud)

```
kubectl create -f sonarqube-nodeservice.yaml
```

### Step 6 - Verify Service

Now verify the services and make sure they are up.

```
kubectl get services
```

#### Minikube

On minikube, you might observe:

```
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        25d
sonar        NodePort    10.99.147.104   <none>        80:30029/TCP   1h
```

Looks good, lets access it with our browser.

```
minikube service sonar
```

This should fire up a browser with a running Sonar instance.

#### Public Cloud

On Public Cloud, you will observe an external IP if all goes well. Sometimes it sites in a 'Pending' state for a few minutes before assigning.

### Summary

This is a very simple hello world Sonar. Useful for establishing a baseline before building more complexity (persistence, security, etc). This can be useful when launching on unfamilar infrastructure or unusual enterprise networking restrictions for example. 
