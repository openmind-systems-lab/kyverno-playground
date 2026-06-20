# 🚀 Kyverno Proof of Concept (PoC)

## 📖 Overview

This Proof of Concept demonstrates how **Kyverno** acts as a Kubernetes **Admission Controller** to enforce security and governance policies before workloads are created in a cluster.

In this example, we'll create a policy that **prevents Pods from using the `latest` container image tag**.

---

# 🏗️ Architecture
![schema](img/schema.png)


---

# 🧠 How Kyverno Works

Kyverno integrates with Kubernetes using **Admission Webhooks**.

Whenever a user creates or modifies a Kubernetes resource, the request flows through Kyverno before Kubernetes accepts it.

The workflow looks like this:

```text
kubectl apply
      │
      ▼
Kubernetes API Server
      │
      ▼
Admission Webhook
      │
      ▼
Kyverno evaluates policies
      │
      ├──────────────┐
      │              │
      ▼              ▼
Allowed ✅       Denied ❌
      │              │
      ▼              ▼
Object Created   Error returned
```

Kyverno acts as a **security gate** for your cluster.

---

# 🎯 Objective

Prevent Pods from using the `latest` image tag.

Allowed:

```text
nginx:1.27
```

Blocked:

```text
nginx:latest
```

---

# ⚙️ Prerequisites

- Kubernetes Cluster
- kubectl
- Helm 3

---

# 📦 Install Kyverno

```bash
helm repo add kyverno https://kyverno.github.io/kyverno/
helm repo update

helm install kyverno kyverno/kyverno \
  -n kyverno \
  --create-namespace
```

Verify:

```bash
kubectl get pods -n kyverno
```

Expected:

```text
NAME                                      READY   STATUS
kyverno-admission-controller              1/1     Running
kyverno-background-controller             1/1     Running
kyverno-cleanup-controller                1/1     Running
kyverno-reports-controller                1/1     Running
```

---

# 🔒 Create the Policy

Create a file named **disallow-latest.yaml**

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: disallow-latest-tag

spec:
  validationFailureAction: Enforce
  background: true

  rules:
    - name: require-explicit-image-tag

      match:
        any:
          - resources:
              kinds:
                - Pod

      validate:
        message: "Images using the 'latest' tag are not allowed."

        pattern:
          spec:
            containers:
              - image: "!*:latest"
```

Apply the policy:

```bash
kubectl apply -f disallow-latest.yaml
```

---

# ❌ Test a Failing Deployment

```bash
kubectl run bad-nginx --image=nginx:latest
```

Expected output:

```text
Error from server:

admission webhook denied the request:

Images using the 'latest' tag are not allowed.
```

---

# ✅ Test a Successful Deployment

```bash
kubectl run good-nginx --image=nginx:1.27
```

Verify:

```bash
kubectl get pods
```

Expected:

```text
NAME          READY   STATUS
good-nginx    1/1     Running
```

---

# 🏛️ Kyverno Components

```text
+--------------------------------------------------+
| Admission Controller                             |
|--------------------------------------------------|
| Validates and mutates incoming requests          |
+--------------------------------------------------+

+--------------------------------------------------+
| Background Controller                            |
|--------------------------------------------------|
| Periodically scans existing resources            |
+--------------------------------------------------+

+--------------------------------------------------+
| Reports Controller                               |
|--------------------------------------------------|
| Generates PolicyReport and ClusterPolicyReport   |
+--------------------------------------------------+

+--------------------------------------------------+
| Cleanup Controller                               |
|--------------------------------------------------|
| Removes expired resources                        |
+--------------------------------------------------+
```

---

# 🌟 Why Kyverno?

✅ Kubernetes Native

✅ Policies written in YAML

✅ No Rego language required

✅ Easy to learn

✅ GitOps Friendly

✅ Admission Control

✅ Policy Auditing

✅ Resource Mutation

✅ Resource Generation

✅ Image Signature Verification

---

# 🎉 Conclusion

Kyverno provides a simple yet powerful way to enforce security, compliance, and operational best practices in Kubernetes.

Instead of writing custom admission webhooks or learning Rego, platform teams can define policies using familiar Kubernetes YAML manifests.

This PoC demonstrated how Kyverno prevents workloads using the `latest` image tag from entering the cluster, ensuring better deployment hygiene and repeatable governance.

---

# 🧹 Cleanup

Delete the demo Pod:

```bash
kubectl delete pod good-nginx
```

Delete the policy:

```bash
kubectl delete clusterpolicy disallow-latest-tag
```

Uninstall Kyverno:

```bash
helm uninstall kyverno -n kyverno
```

Delete the namespace:

```bash
kubectl delete namespace kyverno
```

---

# 📚 References

- https://kyverno.io
- https://kyverno.io/policies/

---

⭐ Happy Policy Engineering! 🚀
