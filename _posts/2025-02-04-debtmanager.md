---
layout: post
title: "Debt Manager - Unreleased custom PHP dashboard for managing debts"
date: 2025-02-05 10:00:00 0000
categories: services finance
tags: homelab finance homebrew debtmanager

---

Debt Manager is a PHP Based application created by myself, not yet available publically, along with a API i use this to automatically track when friends have paid me any money they may owe me.

It uses the webdevops PHP and Nginx container

### Kubernetes Manifest (deployment.yaml)
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: debtmanager
    app.kubernetes.io/instance: debtmanager
    app.kubernetes.io/name: debtmanager
  name: debtmanager
  namespace: webdev
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: debtmanager
  template:
    metadata:
      labels:
        app: debtmanager
        app.kubernetes.io/name: debtmanager
    spec:
      nodeSelector:
        worker: "true"
      containers:
      - image: webdevops/php-nginx:8.1
        name: debtmanager
        ports:
        - containerPort: 81
          hostPort: 81
          protocol: TCP
        - containerPort: 80
          hostPort: 80
          protocol: TCP
        env:
        - name: APPLICATION_PATH
          value: /app
        - name: PHP_DATE_TIMEZONE
          value: Europe/London
        - name: SERVICE_NGINX_CLIENT_MAX_BODY_SIZE
          value: 50m
        - name: TZ
          value: Europe/London
        - name: WEB_DOCUMENT_INDEX
          value: index.php
        - name: WEB_DOCUMENT_ROOT
          value: /app/panel
        volumeMounts:
        - mountPath: "/app"
          readOnly: false
          name: data
          subPath: "app"
        - mountPath: "/opt/docker/etc/nginx/conf.d"
          name: debtmanager-config
        - mountPath: "/vendor"
          readOnly: false
          name: data
          subPath: "vendor"
      volumes:
        - name: data
          persistentVolumeClaim:
            claimName: debtmanager
        - name: debtmanager-config
          configMap:
            name: debtmanager-config
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: debtmanager
  name: debtmanager
  namespace: webdev
spec:
  ports:
  - name: "api"
    port: 81
    targetPort: 81
  - name: "web"
    port: 80
    targetPort: 80
  selector:
    app: debtmanager
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: debtmanager
  namespace: webdev
  annotations: 
    kubernetes.io/ingress.class: traefik-external
spec:
  entryPoints:
    - websecure
  routes:
    - match: Host(`debtmanager.f9.casa`)
      kind: Rule
      services:
        - name: debtmanager
          port: 80
      middlewares:
        - name: default-headers
          namespace: default
    - match: Host(`api.f9.casa`)
      kind: Rule
      services:
        - name: debtmanager
          port: 81
  tls:
    secretName: f9-casa-tls

```

### Kubernetes Manifest (configmap.yaml
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: debtmanager-config
  namespace: webdev
  labels:
    app: debtmanager
data:
  api.conf: |
    server {
        listen 81 default_server;
        listen [::]:81 default_server;

        server_name  _;

        root "/app/api";
        index index.php;

        rewrite ^/public/(.*)$ /public.php?u=$1 last;
        rewrite /webhook /webhook.php last;
        rewrite ^/webhook/(.*)$ /webhook.php last;

        include /opt/docker/etc/nginx/vhost.common.d/*.conf;
    }
  servers.conf: |
    server {
        listen 82 default_server;
        listen [::]:82 default_server;

        server_name  _;

        root "/app/servers";
        index index.php;

        include /opt/docker/etc/nginx/vhost.common.d/*.conf;
    }
  10-php.conf: |
    # Sets PHP upstream
    upstream php {
        server 127.0.0.1:9000;
    }
```

### Docker Compose
```yaml
version: '3.9'
services:
  debtmanager:
    image: webdevops/php-nginx:8.1
    hostname: debtmanager
    networks:
      - traefik-public
    environment:
      - APPLICATION_PATH=/app
      - WEB_DOCUMENT_ROOT=/app/panel
      - TZ=Europe/London
      - WEB_DOCUMENT_INDEX=index.php
      - SERVICE_NGINX_CLIENT_MAX_BODY_SIZE=50m
      - PHP_DATE_TIMEZONE=Europe/London
    volumes:
      - /srv/cephfs/docker/appdata/debtmanager/www:/app
      - /srv/cephfs/docker/appdata/debtmanager/vhost/:/opt/docker/etc/nginx/conf.d:rw
      - /srv/cephfs/docker/appdata/debtmanager/vendor/:/vendor:rw
    deploy:
      labels:
        - "traefik.enable=true"
        
        - "traefik.http.routers.debtmanager.rule=Host(`debtmanager.f9.casa`)"
        - "traefik.http.services.debtmanager.loadbalancer.server.port=80"
        - "traefik.http.routers.debtmanager.entrypoints=websecure"
        - "traefik.http.routers.debtmanager.tls=true"
        - "traefik.http.routers.debtmanager.tls.certresolver=letsencrypt"
        - "traefik.http.routers.debtmanager.service=debtmanager"
        - "traefik.http.routers.debtmanagerapi.rule=Host(`api.f9.casa`)"
        - "traefik.http.services.debtmanagerapi.loadbalancer.server.port=81"
        - "traefik.http.routers.debtmanagerapi.entrypoints=websecure"
        - "traefik.http.routers.debtmanagerapi.tls=true"
        - "traefik.http.routers.debtmanagerapi.tls.certresolver=letsencrypt"
        - "traefik.http.routers.debtmanagerapi.service=debtmanagerapi"

        - homepage.group=Finance
        - homepage.name=Debt Manager
        - homepage.icon=ihatemoney.png
        - homepage.href=https://debtmanager.f9.casa
        - homepage.description=Debt Management
        - homepage.siteMonitor=http://debtmanager
      mode: replicated
networks:
  traefik-public:
    external: true
```