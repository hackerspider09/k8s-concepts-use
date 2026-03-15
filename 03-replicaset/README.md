# 03 Replicaset

| Feature                               | **ReplicaSet**                                              | **Deployment**                                                                         |
| ------------------------------------- | ----------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| **Purpose**                           | Ensures a specified number of pod replicas are running      | Manages ReplicaSets and provides declarative updates (rolling updates, rollback, etc.) |
| **Control Level**                     | Low-level (directly controls Pods)                          | High-level (controls ReplicaSets which control Pods)                                   |
| **Rolling Updates**                   | ❌ **Not Supported**                                         | ✅ **Supported** automatically                                                          |
| **Rollback**                          | ❌ **Not Supported**                                         | ✅ **Supported** with `kubectl rollout undo`                                            |
| **Version Management**                | ❌ No version tracking                                       | ✅ Tracks versions via ReplicaSet revisions                                             |
| **Recommended Usage**                 | Rarely used directly (unless you need fine-grained control) | ✅ **Use this for all production workloads**                                            |
| **Creates ReplicaSet Automatically?** | You manually create one                                     | Deployment automatically creates a ReplicaSet internally                               |
| **Upgrade Strategy**                  | Delete and recreate pods manually                           | Uses rolling update strategy by default (`maxUnavailable`, `maxSurge`)                 |


| Action                    | ReplicaSet      | Deployment     |
| ------------------------- | --------------- | -------------- |
| `kubectl set image`       | Manual recreate | Rolling update |
| `kubectl rollout status`  | ❌               | ✅              |
| `kubectl rollout undo`    | ❌               | ✅              |
| `kubectl rollout history` | ❌               | ✅              |
| Manages ReplicaSets       | ❌               | ✅              |
