# pod-basic
files=> pod-basic.yaml

to run pod : `$ kubectl apply -f pod-basic.yaml`

as app running on port 5000 to access it need to use port forwarding 
`$ kubectl port-forward pod/app-pod 8080:5000`


# init-pod
file => pod-init-containers.yaml

* A **Pod** in Kubernetes is made up of one or more **containers**.
* Each container has its **own isolated filesystem** (coming from the image + writable layer).
* When an **initContainer** finishes, its container is destroyed along with its filesystem.
* The main container then starts fresh, in its own filesystem.

So even though both containers run “inside the same Pod,” they don’t automatically share files unless you explicitly mount a shared volume (like `emptyDir`, hostPath, or a PVC).


> If you want initContainers to leave behind data for the main containers, you must use a shared volume.


### Analogy

Think of it like this:

* The initContainer is a “setup worker” who uses a **scratchpad**.
* Once he’s done, his scratchpad gets thrown away.
* If you want the main worker to see it, you need to tell them both to write/read from the **same whiteboard** (a volume).


### Why you don’t see it on the worker node directly

Yes, technically Docker (or containerd) is running inside the worker node, and the container filesystems exist there. But they are sandboxed and tied to each container. Kubernetes does not automatically “copy” data from an initContainer’s filesystem into the main container.

The only Kubernetes-supported way to persist and share that data within a Pod is: **mount a volume in both containers**.

---

✅ That’s why you need the `emptyDir` (or another volume) if you want `/testDir/test.txt` created by the initContainer to show up in your main app container.

`$ kubectl exec -it pods/app-pod -- sh`
to see file is present in main pod

# Side-car
file=> pod-sidecar.yaml

to get into specific pod
`$ kubectl exec -it <pod-name> -c logshipper -- sh`

to check logs
`$ kubectl logs <pod-name> -c logshipper`







