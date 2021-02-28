# kubernetes-k8s-lab09

In this scenario, you'll learn how manage secrets using Kubernetes. Kubernetes allows you to create secrets that are mounted to a pod via environment variables or as a volume.

This allows secrets, such as SSL certificates or passwords, to only be managed via an infrastructure team in a secure way instead of having the passwords stored within the application's deployment artefacts. 

######

Step 2 - Create Secrets

Kubernetes requires secrets to be encoded as Base64 strings.

Using the command line tool we can create the Base64 strings and store them as variables to use in a file. 
username=$(echo -n "admin" | base64)
password=$(echo -n "a62fjbd37942dcs" | base64)

The secret is defined using yaml. Below we'd using the variables defined above and providing them with friendly labels which our application can use. This will create a collection of key/value secrets that can be accessed via the name, in this case test-secret

echo "apiVersion: v1
kind: Secret
metadata:
  name: test-secret
type: Opaque
data:
  username: $username
  password: $password" >> secret.yaml

This yaml file can be used to with Kubectl to create our secret. When launching pods that require access to the secret we'll refer to the collection via the friendly-name.
Task: Create the secret

Use kubectl to create our secret.

kubectl create -f secret.yaml

The following command allows you to view all the secret collections defined.

kubectl get secrets

In the next step we'll use these secrets via a Pod.
######

Step 3 - Consume via Environment Variables

In the file secret-env.yaml we've defined a Pod which has environment variables populated from the previously created secret.

View the file using cat secret-env.yaml

To populate the environment variable we define the name, in this case SECRET_USERNAME, along with the name of the secrets collection and the key which containers the data.

The structure looks like this:

- name: SECRET_USERNAME
valueFrom:
 secretKeyRef:
   name: test-secret
   key: username

Task

Launch the Pod using 
kubectl create -f secret-env.yaml

Once the Pod started, you output the populated environment variables. 

kubectl exec -it secret-env-pod env | grep SECRET_

Kubernetes decodes the base64 value when populating the environment variables. You should see the original username/password combination we defined. These variables can now be used for accessing APIs, Databases etc.

You can check the status of a Pod using kubectl get pods.

In the next step we'll mount the secrets as files.

######

Step 4 - Consume via Volumes

The use of environment variables for storing secrets in memory can result in them accidentally leaking. The recommend approach is to use mount them as a Volume.

The Pod specification can be viewed using cat secret-pod.yaml.

To mount the secrets as volumes we first define a volume with a well-known name, in this case, secret-volume, and provide it with our stored secret.

volumes:
 - name: secret-volume
   secret:
     secretName: test-secret

When we define the container we mount our created volume to a particular directory. Applications will read the secrets as files from this path.

volumeMounts:
 - name: secret-volume
   mountPath: /etc/secret-volume

Task

Create our new Pod using kubectl create -f secret-pod.yaml

Once started you can interact with the mounted secrets. For example, you can list all the secrets available as if they're regular data. For example 

kubectl exec -it secret-vol-pod ls /etc/secret-volume

Reading the files allows us to access the decoded secret value. To access username we'd use 

kubectl exec -it secret-vol-pod cat /etc/secret-volume/username

For the password, we'd read the password file 

kubectl exec -it secret-vol-pod cat /etc/secret-volume/password
######
