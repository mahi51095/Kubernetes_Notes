# 🔐 ConfigMaps & Secrets Explained with Hotel Metaphor

In addition to lockers (PV/PVC), a guest may need to access:

* **Wi-Fi passwords** 🔐
* **Room service menus or hotel instructions** 📋

These are **ConfigMaps and Secrets** in Kubernetes!

---

## 📝 ConfigMap = Hotel Instruction Manual

* Contains **non-sensitive data** like:

  * Hotel rules
  * Room service hours
  * Emergency contacts
* All guests can safely read this.

✅ In Kubernetes:

* A ConfigMap stores **environment variables, app configs, URLs**, etc.
* Can be mounted to a pod as a **file** or **env variable**.

---

## 🔐 Secret = Wi-Fi Password Slip

* Contains **sensitive data** like:

  * Wi-Fi passwords
  * Room PIN
  * Payment token
* Shared **securely** with the guest.

✅ In Kubernetes:

* A Secret stores passwords, tokens, keys.
* Base64 encoded (not encrypted, but hidden).

⚠️ ConfigMaps and Secrets are stored in **etcd**. Secrets use base64, not true encryption.

---

## 🧩 How ConfigMap & Secret Are Used by Pods

| Real-World Analogy                          | Kubernetes                           |
| ------------------------------------------- | ------------------------------------ |
| Hotel instructions given to guest           | ConfigMap mounted in pod             |
| Passwords written on slip & handed securely | Secrets mounted or injected into pod |

### Usage:

* Pod **mounts** ConfigMap/Secret like it mounts PVC.
* Can also use them as **environment variables**.

---

## 🔗 Relationship to PV/PVC

* **PVC/PV** = used to store **files and persistent data**.
* **ConfigMap/Secret** = used to provide **startup configuration and credentials**.

They serve different purposes:

| Purpose   | Uses PersistentVolume? | Sensitive? | Scope                        |
| --------- | ---------------------- | ---------- | ---------------------------- |
| PVC/PV    | Yes                    | No         | File/data storage            |
| ConfigMap | No                     | No         | App config (safe info)       |
| Secret    | No                     | Yes        | Credentials & sensitive info |

Together, they enable a pod to:

* Store and retrieve data (via PV)
* Start with the right instructions (via ConfigMap)
* Stay secure (via Secret)

---

## 🧾 Sample YAML for ConfigMap and Secret

### 📄 ConfigMap Example with Files and Env

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-demo

data:
  player_initial_lives: "3"
  ui_properties_file_name: "user-interface.properties"
  game.properties: |
    enemy.types=aliens,monsters
    player.maximum-lives=5
  user-interface.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true
```

### 🧪 Pod Using ConfigMap as File and Env

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-demo-pod
spec:
  containers:
  - name: demo
    image: alpine
    command: ["sleep", "3600"]
    env:
    - name: PLAYER_INITIAL_LIVES
      valueFrom:
        configMapKeyRef:
          name: game-demo
          key: player_initial_lives
    - name: UI_PROPERTIES_FILE_NAME
      valueFrom:
        configMapKeyRef:
          name: game-demo
          key: ui_properties_file_name
    volumeMounts:
    - name: config
      mountPath: "/config"
      readOnly: true
  volumes:
  - name: config
    configMap:
      name: game-demo
      items:
      - key: "game.properties"
        path: "game.properties"
      - key: "user-interface.properties"
        path: "user-interface.properties"
```

---

## 🧰 Real-World Application Integration: MongoDB + Spring App

### 🗂️ ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: springappconfig
  namespace: prod

data:
  db_username: devdb
```

### 🔐 Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: springappsecret
  namespace: prod

type: Opaque
stringData:
  db_password: devdb@123
```

### 🍃 MongoDB ReplicaSet Using ConfigMap, Secret, and PVC

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: mongodb
  namespace: prod
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongocon
        image: mongo
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          valueFrom:
            configMapKeyRef:
              name: springappconfig
              key: db_username
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: springappsecret
              key: db_password
        volumeMounts:
        - name: mongonfsvol
          mountPath: /data/db
      volumes:
      - name: mongonfsvol
        persistentVolumeClaim:
          claimName: mongodb1-pvc
```

### 📡 MongoDB Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongosvc
  namespace: prod
spec:
  type: ClusterIP
  selector:
    app: mongodb
  ports:
  - port: 27017
    targetPort: 27017
```

### 🌱 Spring App Deployment with Hardcoded Credentials (For Comparison)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: springapp
  namespace: dev
spec:
  replicas: 2
  selector:
    matchLabels:
      app: springapp
  template:
    metadata:
      labels:
        app: springapp
    spec:
      containers:
      - name: springapp
        image: kkeducation12345/spring-app:1.0.0
        ports:
        - containerPort: 8080
        env:
        - name: MONGO_DB_HOSTNAME
          value: mongosvc
        - name: MONGO_DB_USERNAME
          value: devdb
        - name: MONGO_DB_PASSWORD
          value: devdb@123
        resources:
          requests:
            cpu: 300m
            memory: 256Mi
          limits:
            cpu: 500m
            memory: 512Mi
```

### 🛰️ Spring App Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: springappsvc
  namespace: dev
spec:
  type: NodePort
  selector:
    app: springapp
  ports:
  - port: 80
    targetPort: 8080
```

---

Let me know if you’d like me to modify the Spring App deployment to use the same ConfigMap/Secret style for credentials too!
