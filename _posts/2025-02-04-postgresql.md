---
layout: post
title: "PostgreSQL - Database Engine"
date: 2025-02-05 10:00:00 0000
categories: services database
tags: homelab database postgres postgresql
image:
 path: /assets/img/thumbnails/postgresql.webp
---

[PostgreSQL](https://www.postgresql.org/) is a powerful, open source object-relational database system with over 35 years of active development that has earned it a strong reputation for reliability, feature robustness, and performance.

This stack also includes pgAdmin which is the most popular and feature rich Open Source administration and development platform for PostgreSQL

> I am using tensorchord/pgvecto-rs for Immich
{: .prompt-info }

### Kubernetes Manifest (PostgreSQL)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: postgresql
    app.kubernetes.io/instance: postgresql
    app.kubernetes.io/name: postgresql
  name: postgresql
  namespace: postgresql
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: postgresql
  template:
    metadata:
      labels:
        app: postgresql
        app.kubernetes.io/name: postgresql
    spec:
      nodeSelector:
        worker: "true"
      containers:
        - image: tensorchord/pgvecto-rs:pg16-v0.2.0
          name: postgresql
          ports:
            - containerPort: 5432
              hostPort: 5432
              protocol: TCP
          env:
            - name: PGADMIN_CONFIG_SERVER_MODE
              value: "false"
            - name: PGDATA
              value: /var/lib/postgresql/data/pgdata
            - name: POSTGRES_PASSWORD
              value: [REDACTED]
            - name: POSTGRES_USER
              value: admin
            - name: TZ
              value: Europe/London
          volumeMounts:
            - mountPath: /var/lib/postgresql/data
              name: data
      hostname: postgresql
      restartPolicy: Always
      volumes:
        - name: data
          persistentVolumeClaim:
            claimName: postgresql
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: postgresql
  name: postgresql
  namespace: postgresql 
spec:
  type: LoadBalancer
  loadBalancerIP: 10.0.10.201
  ports:
    - name: "postgres"
      port: 5432
      targetPort: 5432
  selector:
    app: postgresql
```

### Kubernetes Manifest (PGAdmin)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: pgadmin
    app.kubernetes.io/instance: pgadmin
    app.kubernetes.io/name: pgadmin
  name: pgadmin
  namespace: postgresql
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: pgadmin
  template:
    metadata:
      labels:
        app: pgadmin
        app.kubernetes.io/name: pgadmin
    spec:
      nodeSelector:
        worker: "true"
      containers:
        - image: dpage/pgadmin4
          name: pgadmin
          securityContext:
            runAsUser: 0
            runAsGroup: 0
          env:
            - name: PGADMIN_DEFAULT_EMAIL
              value: [REDACTED]
            - name: PGADMIN_DEFAULT_PASSWORD
              value: [REDACTED]
          volumeMounts:
            - mountPath: /var/lib/pgadmin
              name: pgadmin
      volumes:
        - name: pgadmin
          persistentVolumeClaim:
            claimName: pgadmin
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: pgadmin
  name: pgadmin
  namespace: postgresql 
spec:
  ports:
    - name: "pgadmin"
      port: 80
      targetPort: 80
  selector:
    app: pgadmin
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: pgadmin
  namespace: postgresql
  annotations: 
    kubernetes.io/ingress.class: traefik-external
spec:
  entryPoints:
    - websecure
  routes:
    - match: Host(`pgadmin.f9.casa`)
      kind: Rule
      services:
        - name: pgadmin
          port: 80
      middlewares:
        - name: default-headers
          namespace: default
        - name: authentik
          namespace: authentik
  tls:
    secretName: f9-casa-tls
```

### Docker Compose
```yaml
version: '3.2'
services:
  postgresql:
    image: tensorchord/pgvecto-rs:pg16-v0.2.0
    hostname: postgresql
    networks:
      - traefik-public
    environment:
      - TZ=Europe/London
      - POSTGRES_PASSWORD=[REDACTED]
      - POSTGRES_USER=admin
      - POSTGRES_DB=postgres
      - PGDATA=/var/lib/postgresql/data
      - PGADMIN_CONFIG_SERVER_MODE=false
    ports:
      - 5432:5432/tcp
    volumes:
      - type: bind
        source: /srv/cephfs/docker/appdata/postgresql16
        target: /var/lib/postgresql/data
    deploy:
      mode: replicated
      placement:
        constraints: [node.role == manager]
        
  pgadmin:
    image: dpage/pgadmin4
    hostname: pgadmin
    networks:
      - traefik-public
    environment:
      PGADMIN_DEFAULT_EMAIL: [REDACTED]
      PGADMIN_DEFAULT_PASSWORD: [REDACTED]
    volumes:
      - /srv/cephfs/docker/appdata/pgadmin:/var/lib/pgadmin
    deploy:
      labels:
        - "traefik.enable=true"
        - "traefik.http.routers.pgadmin.rule=Host(`pgadmin.f9.casa`)"
        - "traefik.http.services.pgadmin.loadbalancer.server.port=80"
        - "traefik.http.routers.pgadmin.entrypoints=https"
        - "traefik.http.routers.pgadmin.tls=true"
        - "traefik.http.routers.pgadmin.tls.certresolver=letsencrypt"
        - "traefik.http.routers.pgadmin.middlewares=authentik@docker"
      mode: replicated
      placement:
        constraints: [node.role == manager]
        
networks:
  traefik-public:
    external: true
```