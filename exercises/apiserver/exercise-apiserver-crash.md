# Apiserver Crash

## Configure a wrong argument

The idea here is to misconfigure the Apiserver in different ways, then check possible log locations for errors.
The core goal is to make the student comfortable with the situation where an apiserver is not coming back up.

### Exercise
#### 1 - Configure the Apiserver with a new argument `--this-is-very-wrong`
#### 2 - Check if the pod comes back up and what logs this causes. Save the logs into a file called `logs.txt` 
#### 3 - Fix the server again

### Solution
#### 1
```
# change current context
k config set-context --current --namespace kube-system

# make a backup
cp /etc/kubernetes/manifests/kube-apiserver.yaml ~/apiserver.yaml.ori

# make the change
vi /etc/kubernetes/manifests/kube-apiserver.yaml
```

#### 2 
```
# see if the api server comes back up
watch crictl ps

# check the pod
k get pod

# view the logs it caused
cat /var/log/pods/kube-syste_kube-apiserveri-controlplane__(some_id)/kube-apiserver/X.log >> ~/logs.txt
> 2022-01-26T10:41:12.401641185Z stderr F Error: unknown flag: --this-is-very-wrong
cat ~/logs.txt
```

#### 3 
```
# revert the change
cp ~/apiserver.yaml.ori /etc/kubernetes/manifests/kube-apiserver.yaml

# watch the pod come back
watch crictl ps 
k get po
```

## Misconfigure ETCD connection 

### Exercise
#### 1 - Change the existing Apiserver manifest argument to: `--etcd-servers=this-is-very-wrong`
#### 2 - Check what the logs say without using /var. Save the logs into a file called `logs.txt` 
#### 3 - Fix the apiserver again.

### Solution
#### 1
```
# change current context
k config set-context --current --namespace kube-system

# make a backup
cp /etc/kubernetes/manifests/kube-apiserver.yaml ~/apiserver.yaml.ori

# make the change
vi /etc/kubernetes/manifests/kube-apiserver.yaml
```

#### 2 
```
# see if the api server comes back up, since the api server is getting restarted a bunch of times
watch crictl ps

# check the logs of the container id  
crictl logs f669a6f3afda2
> Error while dialing dial tcp: address this-is-very-wrong: missing port in address. Reconnecting...

# try syslogs 
journalctl | grep apiserver # nothing specific
cat /var/log/syslog | grep apiserver # nothing specific
```

#### 3 
```
# revert the change
cp ~/apiserver.yaml.ori /etc/kubernetes/manifests/kube-apiserver.yaml

# watch the pod come back
watch crictl ps 
k get po
````

## Invalid Apiserver Manifest YAML 

### Exercise
#### 1 - Change the existing Apiserver manifest and add invalid YAML, something like this:
```
apiVersionTHIS IS VERY ::::: WRONG v1
kind: Pod
metadata:

```
#### 2 - Check what the logs say.
#### 3 - Fix the apiserver again.

### Solution
#### 1
```
# change current context
k config set-context --current --namespace kube-system

# make a backup
cp /etc/kubernetes/manifests/kube-apiserver.yaml ~/apiserver.yaml.ori

# make the change
vi /etc/kubernetes/manifests/kube-apiserver.yaml

# wait till container restarts
watch crictl ps

# check for apiserver pod
k get po
```

#### 2 
```
# seems like the kubelet can't even create the apiserver pod/container
/var/log/pods # nothing
crictl logs # nothing

# syslogs:
tail -f /var/log/syslog | grep apiserver
> Could not process manifest file err="/etc/kubernetes/manifests/kube-apiserver.yaml: couldn't parse as pod(yaml: mapping values are not allowed in this context), please check config file"

# or:
journalctl | grep apiserver
> Could not process manifest file" err="/etc/kubernetes/manifests/kube-apiserver.yaml: couldn't parse as pod(yaml: mapping values are not allowed in this context), please check config file
```

#### 3
```
# revert the change
cp ~/apiserver.yaml.ori /etc/kubernetes/manifests/kube-apiserver.yaml

# watch the pod come back
watch crictl ps 
k get po
````
