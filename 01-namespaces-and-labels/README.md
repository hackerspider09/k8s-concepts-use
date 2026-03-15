# 01 Namespaces And Labels

Got it 🙂 — you want me to **explain Kubernetes default labels** (`app.kubernetes.io/*`), their **types**, **when to use them**, and **how to know which one to pick**.
Let’s break this down **step by step** so it’s crystal clear.

---

# **1. What Are Default Kubernetes Labels?**

Kubernetes defines a **recommended set of labels** called **common labels**, all under the namespace:

```
app.kubernetes.io/*
```

These labels are **not mandatory**, but they are **best practice** because:

* They **standardize** how you describe apps.
* They’re **understood by many tools** (Helm, ArgoCD, Prometheus, Grafana, Kustomize).
* They make **filtering, debugging, and monitoring** easier.

---

# **2. Default Kubernetes Labels and When to Use Them**

| **Label**                          | **Purpose**                           | **Example Value**                          | **When to Use**                                           |
| ---------------------------------- | ------------------------------------- | ------------------------------------------ | --------------------------------------------------------- |
| **`app.kubernetes.io/name`**       | **Name of the application**           | `nginx` / `postgresql` / `orders-api`      | ✅ Always — identifies what this workload is               |
| **`app.kubernetes.io/instance`**   | **Unique deployment name**            | `nginx-prod` / `orders-api-test`           | ✅ Use when you deploy **multiple copies** of the same app |
| **`app.kubernetes.io/version`**    | **Version of the app**                | `1.25.3` / `v2.0.0`                        | ✅ Use when versioning matters, especially for upgrades    |
| **`app.kubernetes.io/component`**  | **Role in the architecture**          | `frontend`, `backend`, `database`, `cache` | ✅ Use to classify the app’s function                      |
| **`app.kubernetes.io/part-of`**    | **Larger system this app belongs to** | `ecommerce-app`, `analytics-platform`      | ✅ Use for grouping services into one system               |
| **`app.kubernetes.io/managed-by`** | **Tool managing the resource**        | `helm`, `argocd`, `kustomize`              | ✅ Use if you're using Helm, ArgoCD, Flux, etc.            |

---

# **3. How to Decide Which Label to Use**

### **A) Start with `app.kubernetes.io/name`**

* **Always required**.
* Describes **what the app is**.
* Example:

  ```yaml
  app.kubernetes.io/name: postgresql
  app.kubernetes.io/name: orders-api
  ```

---

### **B) Use `app.kubernetes.io/instance` When You Have Multiple Deployments**

Example: Two PostgreSQL DBs for two apps:

```yaml
app.kubernetes.io/name: postgresql
app.kubernetes.io/instance: orders-db
```

```yaml
app.kubernetes.io/name: postgresql
app.kubernetes.io/instance: users-db
```

This makes it easy to query:

```bash
kubectl get pods -l app.kubernetes.io/instance=orders-db
```

---

### **C) Use `app.kubernetes.io/component` for Microservices**

* Helps classify **what the service does**.
* Example:

  ```yaml
  app.kubernetes.io/name: orders-api
  app.kubernetes.io/component: backend
  ```

  ```yaml
  app.kubernetes.io/name: webshop-ui
  app.kubernetes.io/component: frontend
  ```

---

### **D) Use `app.kubernetes.io/part-of` for Grouping**

When your system has **multiple components** (frontend, backend, DB, cache), group them:

```yaml
app.kubernetes.io/part-of: ecommerce-app
```

Then you can query **all services** of the ecommerce app:

```bash
kubectl get pods -l app.kubernetes.io/part-of=ecommerce-app
```

---

### **E) Use `app.kubernetes.io/version` for Upgrades**

* Helps track which version is running.
* Especially useful in monitoring dashboards.
* Example:

  ```yaml
  app.kubernetes.io/name: nginx
  app.kubernetes.io/version: 1.25.3
  ```

---

### **F) Use `app.kubernetes.io/managed-by` for Deployment Tools**

* If you use Helm, ArgoCD, Flux, etc.
* Example:

  ```yaml
  app.kubernetes.io/managed-by: helm
  ```

---

# **4. Practical Examples**

### **Example 1 — Backend Service (Orders API)**

```yaml
metadata:
  labels:
    app.kubernetes.io/name: orders-api
    app.kubernetes.io/instance: orders-api-prod
    app.kubernetes.io/component: backend
    app.kubernetes.io/part-of: ecommerce-app
    app.kubernetes.io/version: "1.2.3"
    app.kubernetes.io/managed-by: helm
```

---

### **Example 2 — Database (PostgreSQL)**

```yaml
metadata:
  labels:
    app.kubernetes.io/name: postgresql
    app.kubernetes.io/instance: ecommerce-db
    app.kubernetes.io/component: database
    app.kubernetes.io/part-of: ecommerce-app
    app.kubernetes.io/version: "15.3"
    app.kubernetes.io/managed-by: helm
```

---

### **Example 3 — Frontend (Webshop UI)**

```yaml
metadata:
  labels:
    app.kubernetes.io/name: webshop-ui
    app.kubernetes.io/instance: webshop-ui-prod
    app.kubernetes.io/component: frontend
    app.kubernetes.io/part-of: ecommerce-app
    app.kubernetes.io/version: "2.0.0"
    app.kubernetes.io/managed-by: argocd
```

---

# **5. How to Remember Which Label to Use**

* **Always** → `app.kubernetes.io/name`
* **If multiple copies** → `app.kubernetes.io/instance`
* **If classifying service** → `app.kubernetes.io/component`
* **If grouping services** → `app.kubernetes.io/part-of`
* **If version tracking** → `app.kubernetes.io/version`
* **If using Helm/ArgoCD** → `app.kubernetes.io/managed-by`

---

# **6. Quick Cheatsheet**

| **Scenario**           | **Label**                      | **Example**       |
| ---------------------- | ------------------------------ | ----------------- |
| What app is it?        | `app.kubernetes.io/name`       | `orders-api`      |
| Multiple deployments   | `app.kubernetes.io/instance`   | `orders-api-prod` |
| Which role in system?  | `app.kubernetes.io/component`  | `backend`         |
| Part of which system?  | `app.kubernetes.io/part-of`    | `ecommerce-app`   |
| Which version?         | `app.kubernetes.io/version`    | `1.2.3`           |
| Managed by which tool? | `app.kubernetes.io/managed-by` | `helm`            |

---

If you want, I can prepare a **visual flowchart** showing **how to choose the right label** based on the type of service — **frontend, backend, database, cache, monitoring**.
It’ll make this whole thing much easier to understand at a glance.

Do you want me to make that?


Yes 🙂, you’ve **almost** got it right, but let me explain it **step by step** with an example so there’s **no confusion**.

Let's assume you have an **application** called **ecommerce-app**, and inside it, you have **multiple databases**:

* **PostgreSQL** → for orders
* **MongoDB** → for products
* **Redis** → for caching

Now, we need to assign **default Kubernetes labels** properly.

---

## **1. How to Set Labels for Multiple Databases**

| **Label**                         | **What It Represents**                            | **Example for PostgreSQL** | **Example for MongoDB** | **Example for Redis** |
| --------------------------------- | ------------------------------------------------- | -------------------------- | ----------------------- | --------------------- |
| **`app.kubernetes.io/name`**      | The **product name** (DB type)                    | `postgresql`               | `mongodb`               | `redis`               |
| **`app.kubernetes.io/instance`**  | A **unique name** for this specific DB deployment | `orders-db`                | `products-db`           | `cache-db`            |
| **`app.kubernetes.io/component`** | The role in the architecture                      | `database`                 | `database`              | `database`            |
| **`app.kubernetes.io/part-of`**   | The larger app/system this belongs to             | `ecommerce-app`            | `ecommerce-app`         | `ecommerce-app`       |

---

## **2. Labeling Rules for Your Case**

### **A) `app.kubernetes.io/name` → Database Type**

* This is **not any random name**, it should ideally be the **DB product name**.
* Why?

  * Tools like Helm, ArgoCD, and monitoring dashboards can **identify the software**.
  * If you write `app.kubernetes.io/name: postgresql`, dashboards know it’s a PostgreSQL pod.

✅ **Example:**

```yaml
app.kubernetes.io/name: postgresql
```

❌ Avoid random names like:

```yaml
app.kubernetes.io/name: orders-db
```

That should go into `instance`, not `name`.

---

### **B) `app.kubernetes.io/instance` → Unique DB Deployment Name**

* This differentiates **multiple databases** of the same type.
* Example:

  * If you have **two PostgreSQL DBs** (orders and payments):

    ```yaml
    app.kubernetes.io/name: postgresql
    app.kubernetes.io/instance: orders-db
    ```

    ```yaml
    app.kubernetes.io/name: postgresql
    app.kubernetes.io/instance: payments-db
    ```

---

### **C) `app.kubernetes.io/component` → Always “database”**

* Use `database` for **all DBs**, regardless of type.
* This makes it easy to query **all databases** in your cluster:

```bash
kubectl get pods -l app.kubernetes.io/component=database
```

---

### **D) `app.kubernetes.io/part-of` → The Main Application**

* All DBs belong to the **same ecommerce app**, so they share:

```yaml
app.kubernetes.io/part-of: ecommerce-app
```

---

## **3. Complete Example for PostgreSQL (Orders DB)**

```yaml
metadata:
  labels:
    app.kubernetes.io/name: postgresql
    app.kubernetes.io/instance: orders-db
    app.kubernetes.io/component: database
    app.kubernetes.io/part-of: ecommerce-app
    app.kubernetes.io/version: "15.3"
    app.kubernetes.io/managed-by: helm
```

---

## **4. Quick Summary**

| **If you have multiple DBs** | **Label to Use**              | **Example**                |
| ---------------------------- | ----------------------------- | -------------------------- |
| **DB product/type**          | `app.kubernetes.io/name`      | `postgresql`, `mongodb`    |
| **Unique DB name**           | `app.kubernetes.io/instance`  | `orders-db`, `products-db` |
| **DB role**                  | `app.kubernetes.io/component` | Always `database`          |
| **Parent application**       | `app.kubernetes.io/part-of`   | `ecommerce-app`            |

---

So your understanding is **almost correct**: ✅

* `component` → **always** `database`
* `name` → the **DB product name** (e.g., `postgresql`, **not** `orders-db`)
* `instance` → the **unique deployment name** (e.g., `orders-db`)

---

If you want, I can create a **visual cheat sheet** showing exactly **which labels go where** for **multi-DB apps**, **backend services**, and **frontends** — so you’ll **never get confused again**.

Do you want me to make that? It’ll make things 10× clearer.
