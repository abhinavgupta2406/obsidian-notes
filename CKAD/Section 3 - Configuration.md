# Commands and Arguments in Docker
```
# To create a container (this will create and exits immediately)
docker run ubuntu

# List running containers
docker ps

# List all containers including stopped
docker ps -a

# To override default command mentioned in Docker image
docker run ubuntu [COMMAND]
docker run ubuntu sleep 5
```

```
# Correct commands
CMD sleep 5
CMD ["sleep", "5"]

# Incorrect commands
CMD ["sleep 5"]
```

```
# Dockerfile
FROM ubuntu

CMD sleep 5
```

```
# Build the image
docker build -t ubuntu-sleeper .

docker run ubuntu-sleeper
```

What if I want to change the number of seconds to sleep?
```
# One way is
docker run ubuntu-sleeper sleep 10

# But we want to pass this value is argument
docker run ubuntu-sleeper 10
```

That is where ENTRYPOINT instructions come into play
```
# Docker file
FROM ubuntu

ENTRYPOINT ["sleep"]
```

```
docker run ubuntu-sleeper 10

# The value which we are appending (here 10) will be added in front of the ENTRYPOINT and will work as whole command. Means, command will be sleep 10
```

If no value is passed (like here 10), there will be no arguments passed to the command. So how to configure default value if no value is passed?
So we will use combination of ENTRYPOINT and CMD

```
FROM ubuntu

ENTRYPOINT ["sleep"]

CMD ["5"]
```
Here default value is 5. So if you do not pass any value in the docker command, this will be executed `sleep 5`

If you pass a variable, that will override the CMD value. For example, if you write `docker run ubuntu-sleeper 10`, it will execute `sleep 10`

What if you want to update ENTRYPOINT at runtime?
```
docker run --entrypoint sleep2.0 ubuntu-sleeper 10
```
Above command will execute `sleep2.0 10`

---
# Command and Arguments in Kubernetes pod

`args` in pod definition file is used to override CMD parameters in Docker image. 
`command` in pod definition file is used to override ENTRYPOINT in Docker image.

Continuing from last example:

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      command: ["sleep2.0"] # This is replacing ENTRYPOINT in Docker
      args: ["10"] #This is replacing CMD parameters. This will run `sleep2.0 10`
```

---
# Editing pods and deployments
## Editing pod
We CANNOT edit specification of an existing POD. For example, you cannot edit environment variables, service accounts, resource limits of a running pod. So you have 2 options:

### Option 1
Run `kubectl edit pod <pod-name>` command which will give option to edit. Once you try to save it, you will get error because you are trying to edit a field that is not editable. But a copy of the file with your changes is saved in temporary location which will be shown in the error.

Once you get the file, delete existing pod and create a new pod with your changes in temporary file.

### Option 2
Extract pod definition in YAML format using `-o yaml` and add in a new YAML file. Edit the YAML file as per the requirement. Delete the existing pod and use new YAML file to create the pod.

## Editing deployments
With deployments you can easily edit any field or property of the POD template. Since pod template is child of the deployment specification, with every change the deployment  will automatically delete and create a new pod with new changes.
So if we have to edit a property of pod which is part of deployment, we can do that simply by running: `kubectl edit deployment my-deployment`

---
# Environment Variables

```
# Docker command to add environment variables
docker run -e APP_COLOR=pink simple-webapp-color
```

To add in pod definition file:
```
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
        - containerPort: 8080
      env:
        - name: APP_COLOR
          value: pink
```

Different ways to add environment variables:

Plain Key Value:
```
env:
  - name: APP_COLOR
    value: pink
```

ConfigMap:
```
env:
  - name: APP_COLOR
    valueFrom:
      configMapKeyRef:
```

Secrets:
```
env:
  - name: APP_COLOR
    valueFrom:
      secretKeyRef:
```