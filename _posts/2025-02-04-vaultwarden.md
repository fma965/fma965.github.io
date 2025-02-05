---
layout: post
title: "VaultWarden - Self Hosted Password Management with 2FA And Bitwarden Client Support"
date: 2025-02-05 10:00:00 0000
categories: services security
tags: homelab security password-management password 2fa bitwarden vaultwarden

---

[Vaultwarden - Password Manager](https://github.com/dani-garcia/vaultwarden) is an alternative implementation of the Bitwarden server API written in Rust and compatible with upstream Bitwarden clients*, perfect for self-hosted deployment where running the official resource-heavy service might not be ideal.

I use PostgreSQL instead of SQLite

### Kubernetes Manifest
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: vaultwarden
    app.kubernetes.io/instance: vaultwarden
    app.kubernetes.io/name: vaultwarden
  name: vaultwarden
  namespace: vaultwarden
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: vaultwarden
  template:
    metadata:
      labels:
        app: vaultwarden
        app.kubernetes.io/name: vaultwarden
    spec:
      nodeSelector:
        worker: "true"
      containers:
      - image: vaultwarden/server:latest
        name: vaultwarden
        ports:
        - containerPort: 80
          name: web
          protocol: TCP
        env:
        - name: DATABASE_URL
          value: "[REDACTED]"
        - name: TZ
          value: Europe/London
        volumeMounts:
        - mountPath: "/data"
          readOnly: false
          name: data
        - mountPath: "/data/config.json"
          name: vaultwarden-config
          subPath: config.json
      volumes:
        - name: data
          persistentVolumeClaim:
            claimName: vaultwarden
        - name: vaultwarden-config
          configMap:
            name: vaultwarden-config
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: vaultwarden
  name: vaultwarden
  namespace: vaultwarden
spec:
  ports:
  - name: web-tcp
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: vaultwarden
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: vaultwarden
  namespace: vaultwarden
  annotations: 
    kubernetes.io/ingress.class: traefik-external
    gethomepage.dev/href: "https://pw.f9.casa"
    gethomepage.dev/enabled: "true"
    gethomepage.dev/description: Password Management
    gethomepage.dev/group: Authentication
    gethomepage.dev/icon: vaultwarden.png
    gethomepage.dev/name: Vaultwarden
spec:
  entryPoints:
    - websecure
  routes:
    - match: Host(`pw.f9.casa`)
      kind: Rule
      services:
        - name: vaultwarden
          port: 80
      middlewares:
        - name: default-headers
          namespace: default
  tls:
    secretName: f9-casa-tls
```

### Docker Compose
```yaml
version: '3.9'
services:
  vaultwarden:
    image: vaultwarden/server:latest
    hostname: vaultwarden
    networks:
      - traefik-public
    environment:
      - WEBSOCKET_ENABLED=true
      - INVITATIONS_ALLOWED=false
      - TZ=Europe/London
      - ADMIN_TOKEN=[REDACTED]
      - DATABASE_URL=mysql://vaultwarden:[REDACTED]@mariadb/vaultwarden
    volumes:
      - /srv/cephfs/docker/appdata/vaultwarden:/data
    deploy:
      labels:
        - "traefik.enable=true"
        - "traefik.http.routers.vaultwarden.rule=Host(`pw.f9.casa`)"
        - "traefik.http.services.vaultwarden.loadbalancer.server.port=80"
        - "traefik.http.routers.vaultwarden.entrypoints=https"
        - "traefik.http.routers.vaultwarden.tls=true"
        - "traefik.http.routers.vaultwarden.tls.certresolver=letsencrypt"

        - homepage.group=Authentication
        - homepage.name=Vaultwarden
        - homepage.icon=vaultwarden.png
        - homepage.href=https://pw.f9.casa
        - homepage.description=Password Management
        - homepage.siteMonitor=http://vaultwarden:80
      mode: replicated
networks:
  traefik-public:
    external: true
```