# ConfigMaps

ConfigMaps in Kubernetes can be created either using the **CLI** (from a file or literal values) or defined explicitly using **YAML files**. Let's explore both methods in depth to help you understand when and how to use them.

---

## **What is a ConfigMap?**
A ConfigMap is a Kubernetes object used to store configuration data as key-value pairs. Applications running in pods can consume this data as:
- **Environment variables**
- **Command-line arguments**
- **Mounted files**

---

## **Creating ConfigMaps**

### **1. Creating a ConfigMap Using the CLI**
#### **From a File**
If you have a configuration file, you can create a ConfigMap directly:
```bash
kubectl create configmap <configmap-name> --from-file=<path-to-file>
```
For example:
```bash
kubectl create configmap nginx-config --from-file=nginx.conf
```
This stores the file contents in the ConfigMap.

#### **From Literal Values**
You can also pass key-value pairs directly:
```bash
kubectl create configmap <configmap-name> --from-literal=<key>=<value>
```
Example:
```bash
kubectl create configmap app-settings --from-literal=ENV=production --from-literal=DEBUG=false
```

---

### **2. Creating a ConfigMap Using YAML**
Here’s an example of defining a ConfigMap explicitly in YAML:

#### Example: ConfigMap with Key-Value Pairs
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  ENV: production
  DEBUG: "false"
```

#### Example: ConfigMap from a File (Inline Data)
You can also embed file contents as values:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  nginx.conf: |
    server {
      listen 80;
      server_name localhost;

      location / {
        root /usr/share/nginx/html;
        index index.html;
      }
    }
```

This stores the `nginx.conf` contents directly in the ConfigMap.

---

## **Consuming ConfigMaps**

Once you create a ConfigMap, you can use it in pods in several ways:

### **1. As Environment Variables**
Add the ConfigMap data as environment variables for your container:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app
  template:
    metadata:
      labels:
        app: app
    spec:
      containers:
      - name: app-container
        image: my-app:latest
        env:
        - name: ENV
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: ENV
        - name: DEBUG
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: DEBUG
```

---

### **2. As a Mounted Volume**
Mount the ConfigMap as files inside the container:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        volumeMounts:
        - name: config-volume
          mountPath: /etc/nginx
      volumes:
      - name: config-volume
        configMap:
          name: nginx-config
```
Here:
- The `nginx-config` ConfigMap is mounted to `/etc/nginx` in the container.
- Each key in the ConfigMap becomes a file in the mount path (e.g., `/etc/nginx/nginx.conf`).

---

### **3. Command-Line Arguments**
You can also use the ConfigMap values as command-line arguments:
```yaml
containers:
- name: app
  image: my-app:latest
  args:
  - "--env=$(ENV)"
  - "--debug=$(DEBUG)"
  env:
  - name: ENV
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: ENV
  - name: DEBUG
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: DEBUG
```

---

## **Comparison: CLI vs YAML**
| Feature                | CLI                                            | YAML                               |
|------------------------|------------------------------------------------|------------------------------------|
| **Simplicity**         | Quick for one-off ConfigMaps                  | Better for version control        |
| **Flexibility**        | Limited to literals or file-based creation    | Full customization                |
| **Best Use Case**      | Quick testing or automation scripts           | Long-term configuration storage   |
