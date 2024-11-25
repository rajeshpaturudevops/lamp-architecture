a. Apache + PHP Deployment
yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apache-php
spec:
  replicas: 2
  selector:
    matchLabels:
      app: apache-php
  template:
    metadata:
      labels:
        app: apache-php
    spec:
      containers:
      - name: apache-php
        image: php:apache
        ports:
        - containerPort: 80
        volumeMounts:
        - name: app-code
          mountPath: /var/www/html
      volumes:
      - name: app-code
        configMap:
          name: app-config
b. MySQL StatefulSet
yaml
Copy code
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: root-password
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mysql-persistent-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
c. Service for Apache and MySQL
yaml
Copy code
apiVersion: v1
kind: Service
metadata:
  name: apache-service
spec:
  selector:
    app: apache-php
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer

---
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  selector:
    app: mysql
  ports:
  - protocol: TCP
    port: 3306
    targetPort: 3306
  clusterIP: None
