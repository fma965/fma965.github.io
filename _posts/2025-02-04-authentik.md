---
layout: post
title: "Authentik - SSO Identity Provider For All Your Services"
date: 2025-02-05 10:00:00 0000
categories: services security
tags: homelab security password-management password 2fa authentication sso authentik auth

---

[Authentik - OAuth](https://goauthentik.io/) is an open source Identity Provider focused on flexibility and versatility. You can use authentik in an existing environment to add support for new protocols, implement sign-up/recovery/etc. in your application so you don't have to deal with it, and many other things.

Authentik is configured as a Middleware for [Traefik](/posts/reverse-proxy-stack-traefik-crowdsec/)

### Kubernetes Manifest (Authentik-Server)
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: authentik-server
    app.kubernetes.io/instance: authentik-server
    app.kubernetes.io/name: authentik-server
  name: authentik-server
  namespace: authentik
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: authentik-server
  template:
    metadata:
      labels:
        app: authentik-server
        app.kubernetes.io/name: authentik-server
    spec:
      nodeSelector:
        nas: "true"
      containers:
        - args:
            - server
          env:
            - name: AUTHENTIK_ERROR_REPORTING__ENABLED
              value: "false"
            - name: AUTHENTIK_POSTGRESQL__HOST
              value: postgresql.postgresql
            - name: AUTHENTIK_POSTGRESQL__NAME
              value: authentik
            - name: AUTHENTIK_POSTGRESQL__PASSWORD
              value: [REDACTED]
            - name: AUTHENTIK_POSTGRESQL__USER
              value: authentik
            - name: AUTHENTIK_REDIS__HOST
              value: redis.redis
            - name: AUTHENTIK_REDIS__PASSWORD
              value: [REDACTED]
            - name: AUTHENTIK_SECRET_KEY
              value: [REDACTED]
            - name: TZ
              value: Europe/London
          image: ghcr.io/goauthentik/server:latest
          name: authentik-server
          ports:
            - containerPort: 9000
              hostPort: 9000
              protocol: TCP
          volumeMounts:
            - mountPath: /backups
              name: config
              subPath: backups
            - mountPath: /media
              name: config
              subPath: media
            - mountPath: /certs
              name: config
              subPath: certs
            - mountPath: /templates
              name: config
              subPath: templates
      restartPolicy: Always
      volumes:
        - name: config
          persistentVolumeClaim:
            claimName: authentik
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: authentik-server
  name: authentik-server
  namespace: authentik 
spec:
  ports:
  - name: web-tcp
    port: 9000
    protocol: TCP
    targetPort: 9000
  selector:
    app: authentik-server
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: authentik-server
  namespace: authentik
  annotations: 
    kubernetes.io/ingress.class: traefik-external
    gethomepage.dev/href: "https://auth.f9.casa"
    gethomepage.dev/enabled: "true"
    gethomepage.dev/description: SSO Authentication
    gethomepage.dev/group: Authentication
    gethomepage.dev/icon: authentik.png
    gethomepage.dev/name: Authentik
    gethomepage.dev/widget.type: authentik
    gethomepage.dev/widget.url: "http://authentik-server.authentik:9000"
    gethomepage.dev/widget.key: "[REDACTED]"
spec:
  entryPoints:
    - websecure
  routes:
    - match: Host(`auth.f9.casa`)
      kind: Rule
      services:
        - name: authentik-server
          port: 9000
      middlewares:
        - name: default-headers
          namespace: default
  tls:
    secretName: f9-casa-tls
```

### Kubernetes Manifest (Authentik-Worker)
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: authentik-worker
    app.kubernetes.io/instance: authentik-worker
    app.kubernetes.io/name: authentik-worker
  name: authentik-worker
  namespace: authentik
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: authentik-worker
  template:
    metadata:
      labels:
        app: authentik-worker
        app.kubernetes.io/name: authentik-worker
    spec:
      nodeSelector:
        nas: "true"
      containers:
        - args:
            - worker
          env:
            - name: AUTHENTIK_ERROR_REPORTING__ENABLED
              value: "false"
            - name: AUTHENTIK_POSTGRESQL__HOST
              value: postgresql.postgresql
            - name: AUTHENTIK_POSTGRESQL__NAME
              value: authentik
            - name: AUTHENTIK_POSTGRESQL__PASSWORD
              value: [REDACTED]
            - name: AUTHENTIK_POSTGRESQL__USER
              value: authentik
            - name: AUTHENTIK_REDIS__HOST
              value: redis.redis
            - name: AUTHENTIK_REDIS__PASSWORD
              value: [REDACTED]
            - name: AUTHENTIK_SECRET_KEY
              value: [REDACTED]
            - name: TZ
              value: Europe/London
          image: ghcr.io/goauthentik/server:latest
          name: authentik-worker
          volumeMounts:
            - mountPath: /backups
              name: config
              subPath: backups
            - mountPath: /media
              name: config
              subPath: media
            - mountPath: /certs
              name: config
              subPath: certs
            - mountPath: /templates
              name: config
              subPath: templates
      restartPolicy: Always
      volumes:
        - name: config
          persistentVolumeClaim:
            claimName: authentik
```

### Docker Compose
```yaml
version: '3.2'
services:
  server:
    image: ghcr.io/goauthentik/server:latest
    command: server
    hostname: authentik
    networks:
      - traefik-public
    environment:
      - TZ=Europe/London
      - AUTHENTIK_POSTGRESQL__HOST=postgresql
      - AUTHENTIK_POSTGRESQL__USER=authentik
      - AUTHENTIK_POSTGRESQL__NAME=authentik
      - AUTHENTIK_POSTGRESQL__PASSWORD=[REDACTED]
      - AUTHENTIK_REDIS__HOST=redis
      - AUTHENTIK_REDIS__PASSWORD=[REDACTED]
      - AUTHENTIK_SECRET_KEY=[REDACTED]
      - AUTHENTIK_ERROR_REPORTING__ENABLED=false
    volumes:
      - /srv/cephfs/docker/appdata/authentik/templates:/templates:rw
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /srv/cephfs/docker/appdata/authentik/media:/media:rw
    deploy:
      labels:
        - traefik.enable=true

        - traefik.http.routers.authentik.entrypoints=websecure
        - traefik.http.routers.authentik.tls=true
        - traefik.http.routers.authentik.tls.certresolver=letsencrypt
        - traefik.http.services.authentik.loadbalancer.server.port=9000
       
        - traefik.http.routers.authentik.rule=Host(`auth.f9.casa`) || HostRegexp(`{subdomain:[a-z0-9]+}.f9.casa`) && PathPrefix(`/outpost.goauthentik.io/`)
        - traefik.http.middlewares.authentik.forwardauth.address=http://authentik:9000/outpost.goauthentik.io/auth/traefik
        - traefik.http.middlewares.authentik.forwardauth.trustForwardHeader=true
        - traefik.http.middlewares.authentik.forwardauth.authResponseHeaders=X-authentik-username,X-authentik-groups,X-authentik-email,X-authentik-name,X-authentik-uid,X-authentik-jwt,X-authentik-meta-jwks,X-authentik-meta-outpost,X-authentik-meta-provider,X-authentik-meta-app,X-authentik-meta-version
      
        - homepage.group=Authentication
        - homepage.name=Authentik
        - homepage.icon=authentik.png
        - homepage.href=https://auth.f9.casa
        - homepage.description=SSO Authentication
        - homepage.siteMonitor=http://authentik:9000/-/health/ready/
        - homepage.widget.type=authentik
        - homepage.widget.url=http://authentik:9000
        - homepage.widget.key=[REDACTED]
      mode: replicated
      placement:
        constraints: [node.role == manager]
  worker:
    image: ghcr.io/goauthentik/server:latest
    command: worker
    networks:
      - traefik-public
    environment:
      - TZ=Europe/London
      - AUTHENTIK_POSTGRESQL__HOST=postgresql
      - AUTHENTIK_POSTGRESQL__USER=authentik
      - AUTHENTIK_POSTGRESQL__NAME=authentik
      - AUTHENTIK_POSTGRESQL__PASSWORD=[REDACTED]
      - AUTHENTIK_REDIS__HOST=redis
      - AUTHENTIK_REDIS__PASSWORD=[REDACTED]
      - AUTHENTIK_SECRET_KEY=[REDACTED]
      - AUTHENTIK_ERROR_REPORTING__ENABLED=false
    volumes:
      - /srv/cephfs/docker/appdata/authentik/backups:/backups:rw
      - /srv/cephfs/docker/appdata/authentik/media:/media:rw
      - /srv/cephfs/docker/appdata/authentik/certs:/certs:rw
      - /var/run/docker.sock:/var/run/docker.sock:rw
      - /srv/cephfs/docker/appdata/authentik/templates:/templates:rw
    deploy:
      mode: replicated
      placement:
        constraints: [node.role == manager]
networks:
  traefik-public:
    external: true
```