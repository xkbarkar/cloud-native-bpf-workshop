## 02 - Traceloop

Traceloop is a tool that allows us to see the last syscalls executed by a
process, without adding the slowness that strace adds.

### Look at the already stored information

The traceloop gadget is constantly monitoring pods, so whatever our
cluster has been running since we've added the gadget pod will be already
stored there. You can see all the currently available traces by listing
them for all namespaces, using the `list` subcommand.

```
kubectl gadget traceloop list -A
```

If you completed the network advisor policy exercise, this list will
include all the pods that were started for that. For a freshly started
cluster, it could be that there's only a few pods that have information.

Notice that there's an index column in the list that you get. You'll need
that index when retrieving the syscalls for a running pod.

To look at the syscalls of a pod that's currently running, we'll use the
`pod` subcommand, we need to pass it the namespace, the pod name, and the
index. For example, to look at the first trace for pod
`coredns-66bff467f8-ntxqt` in the `kube-system` namespace, we need to do:

```
kubectl gadget traceloop pod kube-system coredns-66bff467f8-ntxqt 0
```

We can see the last calls that were executed in the pod.

### Inspect a failing pod

The most interesting part of the traceloop gadget is that it allows us to
debug a pod that crashed, even after it's gone. To see that in action,
let's start a pod that will terminate with an error.

```
kubectl run --restart=Never -ti --image=busybox multiplication \
    -- sh -c \
    'RANDOM=output ; echo "3*7*2" | bc > /tmp/file-$RANDOM ; cat /tmp/file-$RANDOM'
```

This pod performed a multiplication, saved the result in a file and attempted
to display the result but failed to do so because of a mistake in the shell
script. The pod is no longer running, so we cannot read the file anymore.

Let's go further and delete the pod:
```
kubectl delete pod multiplication
```

It is possible to retrieve the result of the multiplication using the traceloop
gadget. You can first identify the relevent trace from the list:
```
kubectl gadget traceloop list -n default
```
And then print the trace:
```
gadget traceloop show XXXXXX
```
