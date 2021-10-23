# redis-k8s
This repository contains a manifest file for creating a Redis cluster with 3 masters and 3 slaves, using a configmap and setting the log level to debug, as requested in the assignment.

**NOTE: The manifest file is written for AKS (Azure), this is manifested in the selected storage class ("default").**

**NOTE: comments in the manifest file (deploy-redis.yml) will be used for its explanations, as this is the heart of the project and is fairly complex.**

**Read the following after running `kubectl apply -f deploy-redis.yml`****


To verify the deployment, you can use `kubectl get statefulset`, and make sure that all of the pods are running.

In my case, the pod creation got stuck and upon using `kubectl describe pvc` I found out that I needed to create an additional node because I have maxed out the per-node pvc limit.

to create the cluster we need to run: 
```
kubectl exec -it redis-cluster-0 -- redis-cli \
--cluster create $(kubectl get pods -l app=redis-cluster -o jsonpath='{range.items[*]}{.status.podIP}:6379 ') \
--cluster-replicas 1 \
-a 'NXYdun&KYKzxVMCm&LPUPKtw5wJFu@kmpDKUF3X%zmX6EZtPaChk#L#6E3&hnGY%'
```

This will check the IP addresses of the pods in the statefulset feed them to the cluster creation command.
We're mapping 1 master to 1 slave and `-a` is used to pass the password to redis-cli.
If everything is fine we should be prompted to type `yes` in the shell to confirm the creation of the cluster.

**NOTE: It since we're supplying a password, it might be beneficial to add a space before the command so it won't save it to history,  
This is the deault behavior in bash but it _can_ be changed so make sure your HISTCONTROL is set to `ignorespace` or `ignoreboth`)**

After that, we can check that we have 3 masters and 3 slaves:
```
for x in $(seq 0 5); do echo "redis-cluster-$x"; kubectl exec redis-cluster-$x -- redis-cli \
-a 'NXYdun&KYKzxVMCm&LPUPKtw5wJFu@kmpDKUF3X%zmX6EZtPaChk#L#6E3&hnGY%' role; echo; done
```