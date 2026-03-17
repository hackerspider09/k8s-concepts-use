# Port Forward

## How `kubectl port-forward` Works

`kubectl port-forward` allows you to access a Pod or Service inside a Kubernetes cluster from your local machine.

It creates a **temporary tunnel** from your local port to a port inside the cluster.

### Command Example

```
kubectl port-forward service/django-service 9376:5000
```

This means:

```
Local Machine:9376 → Kubernetes Service:5000 → Pod:targetPort
```

### Flow Diagram

```
Browser / Client
      |
      |  http://localhost:9376
      ▼
kubectl port-forward
      |
      |  (creates TCP tunnel)
      ▼
Kubernetes Service (port 5000)
      |
      ▼
One Backend Pod
      |
      ▼
Container Port (5000)
```

### Important Behavior

Even if the Service has multiple Pods, `kubectl port-forward` connects to **only one Pod**.

Example cluster:

```
ReplicaSet
 ├── Pod-1
 ├── Pod-2
 └── Pod-3
```

Normal Service routing:

```
Client
   |
   ▼
Service
   |
   ├── Pod-1
   ├── Pod-2
   └── Pod-3
```

With `kubectl port-forward`:

```
Client
   |
   ▼
kubectl tunnel
   |
   ▼
Only One Pod (selected endpoint)
```

So **load balancing does not happen** when using `port-forward`.

### Why This Happens

`kubectl port-forward` creates a **direct TCP tunnel** from your local machine to a specific Pod endpoint, bypassing the normal Service load-balancing performed by kube-proxy.

### When to Use Port Forwarding

Port forwarding is mainly used for:

* Local debugging
* Accessing internal services temporarily
* Testing applications without exposing them externally

### Verify Service Endpoints

To see which Pods are behind the Service:

```
kubectl get endpoints django-service
```

Example output:

```
10.244.0.5:5000
10.244.0.6:5000
10.244.0.7:5000
```

These are the Pods that the Service can route traffic to.

### Port Forwarding with Custom Address

By default, `kubectl port-forward` only binds to **localhost (127.0.0.1)**.
This means the forwarded port can only be accessed from the same machine.

If you want the forwarded port to be accessible from **other machines using your system's private IP**, you can use the `--address` flag.

#### Command

```
kubectl port-forward --address=0.0.0.0 <resource>/<name> <local-port>:<target-port>
```

Example:

```
kubectl port-forward --address=0.0.0.0 service/django-service 9376:5000
```

#### What This Does

```text
External Client (same network)
        |
        | http://<your-private-ip>:9376
        ▼
Your Machine (0.0.0.0:9376)
        |
        | kubectl port-forward tunnel
        ▼
Kubernetes Service (port 5000)
        |
        ▼
Backend Pod (container port)
```

#### Key Points

* `--address=0.0.0.0` binds the forwarded port to **all network interfaces**.
* This allows access using your **private IP address** instead of only `localhost`.
* Useful when testing from:

  * another machine in the same network
  * mobile device
  * remote system connected to your network

Without `--address`, the port is only accessible via:

```text
http://localhost:<local-port>
```

With `--address=0.0.0.0`, it becomes accessible via:

```text
http://<your-private-ip>:<local-port>
```
