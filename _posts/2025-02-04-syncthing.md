---
layout: post
title: "SyncThing continous file synchronization"
date: 2025-02-05 10:00:00 0000
categories: services backup
tags: homelab backup syncthing

---

## [Syncthing](https://syncthing.net/)

### Info
Syncthing is a continuous file synchronization program. It synchronizes files between two or more computers in real time, safely protected from prying eyes. Your data is your data alone and you deserve to choose where it is stored, whether it is shared with some third party, and how it’s transmitted over the internet.

~~I have syncthing configured to sync my "appdata" of my Docker Swarm to 2 of my friends home servers, this provides a simple but effective offsite backup solution~~

> Since a lot of migration stuff this isn't currently configured, other backups are in place though, like PBS, DBBackup and Longhorn Backups.
{: .prompt-warning }


### Kubernetes Manifest
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: syncthing
    app.kubernetes.io/instance: syncthing
    app.kubernetes.io/name: syncthing
  name: syncthing
  namespace: syncthing
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: syncthing
  template:
    metadata:
      labels:
        app: syncthing
        app.kubernetes.io/name: syncthing
    spec:
      nodeSelector:
        nas: "true"
      containers:
      - image: linuxserver/syncthing
        name: syncthing
        ports:
          - containerPort: 22000
            hostPort: 22000
            protocol: TCP
          - containerPort: 22000
            hostPort: 22000
            protocol: UDP
          - containerPort: 21027
            hostPort: 21027
            protocol: UDP
          - containerPort: 8384
            hostPort: 8384
            protocol: TCP
        env:
        - name: TZ
          value: Europe/London
        volumeMounts:
        - mountPath: "/backup"
          readOnly: false
          name: smb
        - mountPath: "/config"
          readOnly: false
          name: data
      volumes:
        - name: smb
          persistentVolumeClaim:
            claimName: pvc-syncthing-smb
        - name: data
          persistentVolumeClaim:
            claimName: syncthing
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: syncthing
  name: syncthing
  namespace: syncthing 
spec:
  ports:
  - name: web-tcp
    port: 8384
    protocol: TCP
    targetPort: 8384
  - name: tcp
    port: 22000
    protocol: TCP
    targetPort: 22000
  - name: udp
    port: 22000
    protocol: UDP
    targetPort: 22000
  - name: udp2
    port: 21027
    protocol: UDP
    targetPort: 21027
  type: LoadBalancer
  loadBalancerIP: 10.0.10.203
  selector:
    app: syncthing
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: syncthing
  namespace: syncthing
  annotations: 
    kubernetes.io/ingress.class: traefik-external
spec:
  entryPoints:
    - websecure
  routes:
    - match: Host(`syncthing.f9.casa`)
      kind: Rule
      services:
        - name: syncthing
          port: 8384
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
version: '3.9'
services:
  syncthing:
    image: syncthing/syncthing
    hostname: syncthing
    networks:
      - traefik-public
    environment:
      - TZ=Europe/London
      - PUID=1000
      - PGID=1000
    volumes:
      - /srv/cephfs/docker/appdata/syncthing:/var/syncthing
      - /srv/cephfs/docker:/userdata
      - /srv/backup/Syncthing:/backup
    ports:
      - 22000:22000/tcp # TCP file transfers
      - 22000:22000/udp # QUIC file transfers
      - 21027:21027/udp # Receive local discovery broadcasts
    deploy:
      labels:
        - "traefik.enable=true"
        - "traefik.http.routers.syncthing.rule=Host(`syncthing.f9.casa`)"
        - "traefik.http.services.syncthing.loadbalancer.server.port=8384"
        - "traefik.http.routers.syncthing.entrypoints=websecure"
        - "traefik.http.routers.syncthing.tls=true"
        - "traefik.http.routers.syncthing.tls.certresolver=letsencrypt"

        - "traefik.http.routers.syncthing.middlewares=authentik@docker"
      mode: replicated
networks:
  traefik-public:
    external: true
```
