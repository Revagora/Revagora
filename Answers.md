Installed docker and minikube for this exercise. Cloned the repo to local using git clone.
Used visual studio code to modify the yaml files and kubectl cmd to run the script.

1.	Why will the service fail to bind to the deployment in the same manifest?
The service defined in manifest does not bind to the deployment because the selectors used in the service's selector field do not match the labels defined in the pods. For the service to bind to the deployment, the labels used in the service's selector must match the labels used in the deployment's selector.
In below manifest we have specified the selector like below.
selector:
  matchLabels:
    type: example
    color: red
This means that the pods managed by the deployment have the labels type: example and color: red.
But in the service manifest, we have specified the selector as 
selector:
  type: example
  color: blue
for this to work, need to update the service's selector to match the labels used in the deployment's selector.
Below is the updated yaml file that worked.

apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  selector:
    matchLabels:
      type: example
      color: red
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 10%
  template:
    metadata:
      labels:
        type: example
        color: red
    spec:
      containers:
      - name: echocolor
        image: reselbob/echocolor:v0.1
        resources: {}
        ports:
        - containerPort: 3000
        env:
        - name: COLOR_ECHO_COLOR
          value: RED
        - name: COLOR_ECHO_VERSION
          value: V1
---
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    type: example
    color: red
  ports:
  -
    protocol: TCP
    port: 3000
    targetPort: 3000
  type: NodePort



2.	These two yaml files deploy a service account and an nginx pod.
Setup the pod so that the nginx pod uses the "gmfuser" service account.

To make the nginx pod use the gmfuser service account, you need to specify the serviceAccountName field in the pod's spec. Here's an updated version of your nginx-pod.yaml file:
Below is the modified script that tested and working.
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: nginx
spec:
  serviceAccountName: gmfuser  
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

when you create the pod using this updated manifest, it will use the gmfuser service account.
 


3.	This simple nginx pod will be stuck on ContainerCreating upon deployment.
What other resource (not defined here) must be created on the cluster for this nginx pod to be able to start successfully?
This manifest referencing a secret named mygmfsecret2 using a volume, but we haven't defined that secret in the same YAML file. To resolve the issue and allow the nginx pod to start successfully, need to create the secret resource in cluster with the name mygmfsecret2
apiVersion: v1
kind: Secret
metadata:
  name: mygmfsecret2
type: Opaque
data:
  username: <base64-encoded-username>
  password: <base64-encoded-password>
After creating this secret resource in your cluster, your nginx pod will be able to use it as a volume, and it should no longer be stuck in the ‘ContainerCreating state.

4.	This yaml deploys a basic nginx pod.
	Please modify the deployment so that the liveness probe starts kicking in after 8 seconds.
	Also set the interval between probes to be 10 seconds.
To modify the liveness probe to start after 8 seconds and set the probe interval to 10 seconds, update the pod manifest like below.
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    livenessProbe:
      initialDelaySeconds: 8  
      periodSeconds: 10        
      exec:
        command:
        - ls
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
 

5.	The redis deployment manifest in this folder is likely to get stuck in a pending state upon deployment.
What is the probable reason?
The reason for the Redis deployment manifest to get stuck in a pending state is that the resource requests for memory are set to an extremely high value, specifically "100Ti" (100 terabytes of memory). This is an unrealistic and excessively high memory request that may exceed the available resources in your Kubernetes cluster. To resolve this issue and allow the deployment to run successfully, need to adjust the memory request to a more reasonable value.
resources:
  requests:
    cpu: 100m
    memory: 1024Mi  # Adjust this value based on your actual memory needs

6.	This manifest deploys pod that contains a busybox image. The container uses a simple script to increment a counter every second. It then writes a counter value to a log file called server.log.
Create a sidecar container that outputs the server.log file to stdout.
To create a sidecar container that outputs the server.log file to stdout, we can add another container to the same Pod. Sidecar containers are containers that are needed to run alongside the main container. The two containers share resources like pod storage and network interfaces. The sidecar containers can also share storage volumes with the main containers, allowing the main containers to access the data in the sidecars.
In below updated manifest, added a second container named log-reader that runs the tail command to continuously output the contents of the server.log file to stdout. Both containers share the same volume named logdir, which allows them to access the same log file.
apiVersion: v1
kind: Pod
metadata:
  name: logging-example
spec:
  containers:
  - name: busybox-simple-container
    image: busybox
    args:
    - /bin/sh
    - -c
    - >
      i=0;
      while true;
      do
        echo "$i: $(date)" >> /var/log/server.log;
        i=$((i+1));
        sleep 5;
      done
    volumeMounts:
    - name: logdir
      mountPath: /var/log
    resources: {}

  - name: log-reader
    image: busybox
    command: ["sh", "-c"]
    args:
    - tail -f /var/log/server.log
    volumeMounts:
    - name: logdir
      mountPath: /var/log
    resources: {}

  volumes:
  - name: logdir
    emptyDir: {}

 


7.	This folder contains two yaml files and a total of 4 Kubernetes manifest objects.
	What prerequisite(s) must be deployed in order to use these specific objects in your cluster?
	Please describe how the manifests in this folder depend on one another, and what will end up being created by them when applied.

We need to have Cert-Manager installed and properly configured in your Kubernetes cluster. Cert-Manager is a Kubernetes add-on for managing TLS certificates.
Manifest Dependencies:
1.	ca.yaml:
•	This manifest creates a ClusterIssuer named example-selfsigned-cluster-issuer. It configures a self-signed certificate issuer, which is used for self-signing certificates.
2.	example-selfsigned-ca:
•	This manifest creates a self-signed Certificate named example-selfsigned-ca in the cert-manager namespace. It generates a self-signed CA certificate.
3.	example-intermicrosvccom-ca-issuer:
•	This manifest creates a ClusterIssuer named example-intermicrosvccom-ca-issuer. It configures the issuer to use a CA certificate stored in a secret named ca-tls.
4.	Certificate in auth-tls:
•	This manifest creates a Certificate named auth-tls in the example-system namespace.
•	It specifies the example-intermicrosvccom-ca-issuer as the issuer for this certificate.
•	The certificate is configured for server and client authentication (usages field).
•	It defines various DNS names and the common name for the certificate.
•	It sets a duration and renewal policy for the certificate.
When you apply these manifests to your cluster, Cert-Manager will create the specified resources based on the definitions in these YAML files.
The self-signed CA certificate, ClusterIssuer, and Certificate for auth-tls will be created and managed by Cert-Manager.
The self-signed CA certificate and ClusterIssuer for example-intermicrosvccom-ca-issuer are prerequisites for the auth-tls certificate to be issued.
The auth-tls certificate is intended to be used for securing authentication between services using TLS.
Make sure that Cert-Manager is properly installed and running in your cluster, and then apply these manifests to set up the necessary certificates and issuers.


8.	This folder contains two yaml documents that will deploy nginx along with a network policy.
	What cluster configuration prerequisites must be in place to use this network policy?
	What configuration would need to be present on a pod so that it could connect to the nginx service with the policy applied?
NOTE:
You can run a pod to test the connection using the following command:
kubectl run busybox --image=busybox --rm -it --restart=Never -- wget -O- http://nginx:80 --timeout 2
Hint, you can supply the required configuration with a modification to the above test command

We need network policy support and network policy controller installed and running in cluster.
To allow a pod to connect to the Nginx service with the applied network policy, we need to label the pod with access: granted. This label is specified in the Networkpolicy.yaml file as the podSelector for the allowed ingress traffic.

 


9.	This folder contains a copy of Hashicorp's Vault Helm chart.
	If you supply the argument --set injector.enabled=yes when running helm install/upgrade will the vault agent injector be deployed?
Hint, take a close look at how the chart decides which components are enabled.
	What if you supply no additional arguments at install/upgrade time? Will the vault agent injector be deployed in that case?
when you run helm install or helm upgrade without specifying any additional arguments, the default values from the chart's values.yaml file will be used. we should check the values.yaml file to see if injector.enabled is set to yes or true by default. If it is, then the Vault Agent Injector will be deployed by default.

If we specifically run helm install or helm upgrade with the --set injector.enabled=yes argument, you are overriding the default value and explicitly enabling the Vault Agent Injector component, regardless of its default value in values.yaml. So, in this case, the Vault Agent Injector will be deployed.


