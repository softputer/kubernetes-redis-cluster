# Redis Cluster on Kubernetes

This repo can help you the creation and running of redis cluster in a minute.

## Testing it out

#### Creation

```
$ kubectl create configmap redis-conf ./redis.conf
$ kubectl create -f ./redis-cluster.yaml
```

```
$ kubectl get po -o wide | grep redis
redis-cluster-0                1/1       Running   0          51m       10.1.94.6    node-01
redis-cluster-1                1/1       Running   0          51m       10.1.94.7    node-01
redis-cluster-2                1/1       Running   0          50m       10.1.94.8    node-01
redis-cluster-3                1/1       Running   0          50m       10.1.94.9    node-01
redis-cluster-4                1/1       Running   0          50m       10.1.94.10   node-01
redis-cluster-5                1/1       Running   0          49m       10.1.94.11   node-01
```

After all pods have been created, you can run commands to set cluster:

```
$ kubectl run -i --tty redis-trib --image=softputer/redis-trib \
  --restart=Never /bin/bash
```

In this terminal, you set 3 master and 3 slaves by:

```
$ ./redis-trib.rb create --replicas 1 \
  10.1.94.6:6379 \
  10.1.94.7:6379 \
  10.1.94.8:6379 \
  10.1.94.9:6379 \
  10.1.94.10:6379 \
  10.1.94.11:6379
```

After you have done that, if you are not going to use redis-trib any more, you should delete po redid-trib by:

```
$ kubectl delete po redis-trib
```



#### Result

Check Nodes.

```
$ kubectl exec -it redis-cluster-0 redis-cli cluster nodes
b6a2dc93e12ae14fd0236ad25d92448167b0e3e4 10.1.94.8:6379 master - 0 1477037899721 3 connected 10923-16383
2210f7698deb6710523ab3b03a3c1b59273acc08 10.1.94.9:6379 slave 66a93236e162f044bc99cd3dd79d37a9e3a530e4 0 1477037900665 4 connected
b7cd5974cc8cd7ec58613efaa41c4450c8b9a187 10.1.94.7:6379 master - 0 1477037900223 2 connected 5461-10922
c7a25f9e4eeb7da5ddb734b821c75aa1cf277d05 10.1.94.10:6379 slave b7cd5974cc8cd7ec58613efaa41c4450c8b9a187 0 1477037899217 5 connected
6dcfb31bdf46361cffd0687e201d7a00f7aed5f4 10.1.94.11:6379 slave b6a2dc93e12ae14fd0236ad25d92448167b0e3e4 0 1477037900768 6 connected
66a93236e162f044bc99cd3dd79d37a9e3a530e4 10.1.94.6:6379 myself,master - 0 0 1 connected 0-5460
```

We can see from the above data that, there are 3 masters  which share 16384 hash slots and 3 slaves.

## Further

You can reshard masters and create more slaves as you want.



