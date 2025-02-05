---
layout: post
title: "Pterodactyl Panel - Manage game servers for various games easily"
date: 2025-02-05 10:00:00 0000
categories: services gaming
tags: homelab gaming games pterodactyl

---

Pterodactyl® is a free, open-source game server management panel built with PHP, React, and Go. Designed with security in mind, Pterodactyl runs all game servers in isolated Docker containers while exposing a beautiful and intuitive UI to end users.

I run the Pterodactyl Game Server panel on Kubernetes, this panel controls "Wings" instances on 4 different machines, 3 are internal at my house and another that is remote. 

### Kubernetes Manifest
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: panel
    app.kubernetes.io/instance: panel
    app.kubernetes.io/name: panel
  name: panel
  namespace: pterodactyl
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: panel
  template:
    metadata:
      labels:
        app: panel
        app.kubernetes.io/name: panel
    spec:
      nodeSelector:
        worker: "true"
      containers:
      - image: ghcr.io/pterodactyl/panel:latest
        name: panel
        ports:
        - containerPort: 80
          name: http
          protocol: TCP
        - containerPort: 443
          name: https
          protocol: TCP
        env:
        - name: APP_URL
          value: "https://[REDACTED].f9.casa"
        - name: APP_TIMEZONE
          value: Europe/London
        - name: LETSENCRYPT_EMAIL
          value: "[REDACTED]"
        - name: CACHE_DRIVER
          value: redis         
        - name: SESSION_DRIVER
          value: redis                
        - name: QUEUE_DRIVER
          value: redis                   
        - name: REDIS_HOST
          value: redis.redis
        - name: REDIS_PASSWORD
          value: "[REDACTED]"     
        - name: DB_HOST
          value: mariadb.mariadb                    
        - name: DB_PORT
          value: "3306" 
        - name: DB_DATABASE
          value: pterodactyl
        - name: DB_USERNAME
          value: pterodactyl
        - name: DB_PASSWORD
          value: "[REDACTED]"
        volumeMounts:
        - mountPath: "/app/var"
          readOnly: false
          name: data
          subPath: "var"
        - mountPath: "/etc/nginx/http.d/"
          readOnly: false
          name: data
          subPath: "nginx"
        - mountPath: "/etc/letsencrypt/"
          readOnly: false
          name: data
          subPath: "letsencrypt"
        - mountPath: "/app/storage/logs"
          readOnly: false
          name: data
          subPath: "logs"
      hostAliases:
      - ip: "10.0.10.31"
        hostnames:
        - "[REDACTED].f9.casa"
      - ip: "10.0.10.32"
        hostnames:
        - "[REDACTED].f9.casa"
      - ip: "10.0.10.33"
        hostnames:
        - "[REDACTED].f9.casa"
      volumes:
        - name: data
          persistentVolumeClaim:
            claimName: pterodactyl-panel
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: panel
  name: panel
  namespace: pterodactyl
spec:
  ports:
  - name: https
    port: 443
    protocol: TCP
    targetPort: 443
  - name: http
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: panel
---
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: panel
  namespace: pterodactyl
  annotations: 
    kubernetes.io/ingress.class: traefik-external
    gethomepage.dev/href: "https://[REDACTED].f9.casa"
    gethomepage.dev/enabled: "true"
    gethomepage.dev/description: Game Servers
#    gethomepage.dev/group: Media Management
    gethomepage.dev/icon: pterodactyl.png
    gethomepage.dev/name: Pterodactyl
    gethomepage.dev/widget.type: pterodactyl
    gethomepage.dev/widget.url: "http://panel.pterodactyl" 
    gethomepage.dev/widget.key: "[REDACTED]"
spec:
  entryPoints:
    - websecure
  routes:
    - match: Host(`[REDACTED].f9.casa`)
      kind: Rule
      services:
        - name: panel
          port: 80
      middlewares:
        - name: default-headers
          namespace: default
        - name: panel-headers
          namespace: pterodactyl
  tls:
    secretName: f9-casa-tls

```