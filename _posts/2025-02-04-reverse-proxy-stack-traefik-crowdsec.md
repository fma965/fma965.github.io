---
layout: post
title: "Traefik & Crowdsec - Reverse Proxy Stack"
date: 2025-02-05 10:00:00 0000
categories: services networking
tags: homelab networking proxy traefik crowdsec reverse-proxy
image:
 path: /assets/img/thumbnails/traefik.webp
---

## Reverse Proxy Stack
- [🌐 Traefik *Reverse Proxy*](https://traefik.io/)
- [👮 Crowdsec *Behaviour detection and banning system*](https://www.crowdsec.net/)
- [😃 Traefik Crowdsec Bouncer *Links Traefik and Crowdsec*](https://github.com/fbonalair/traefik-crowdsec-bouncer)

[Traefik](https://traefik.io/) is an open-source Edge Router that makes publishing your services a fun and easy experience. it automatically discovers the right configuration for your services. The magic happens when Traefik inspects your infrastructure, where it finds relevant information and discovers which service serves which request.

I don't currently have Crowdsec implemented due to some issues, It will be returning soon.

~~**CrowdSec** is a free, modern & collaborative behavior detection engine, coupled with a global IP reputation network. CrowdSec is engineered for modern Cloud / Containers / VM-based infrastructures (by decoupling detection and remediation). Once detected you can remedy threats with various bouncers (firewall block, nginx http 403, Captchas, etc.) while the aggressive IP can be sent to CrowdSec for curation before being shared among all users to further improve everyone's security.~~

~~**Traefik Crowdsec Bouncer** is a http service to verify requests and bounce them according to decisions made by CrowdSec by parsing traefik logs~~

### Kubenetes Helm Chart
```yaml
globalArguments:
  - "--global.sendanonymoususage=false"
  - "--global.checknewversion=false"

additionalArguments:
  - "--serversTransport.insecureSkipVerify=true"
  - "--log.level=DEBUG"
  - "--accesslog=true"
  - "--providers.file.directory=/providers"
  - "--providers.file.watch=true"
deployment:
  enabled: true
  replicas: 3
  annotations: {}
  podAnnotations: {}
  additionalContainers: []
  initContainers: []
  additionalVolumes:
  - name: traefik
    persistentVolumeClaim:
      claimName: traefik
  - name: traefik-providers
    configMap:
      name: traefik-providers

additionalVolumeMounts:
  - name: traefik
    mountPath: /var/log/traefik     
    subPath: "log"  
  - name: traefik-providers
    mountPath: /providers 

logs:
  access:
    enabled: true
    filePath: "/var/log/traefik/access.log"

ports:
  web:
    redirectTo:
      port: websecure
      priority: 10
  websecure:
    tls:
      enabled: true
      
ingressRoute:
  dashboard:
    enabled: false

providers:
  kubernetesCRD:
    enabled: true
    ingressClass: traefik-external
    allowExternalNameServices: true
    allowCrossNamespace: true
  kubernetesIngress:
    enabled: true
    allowExternalNameServices: true
    publishedService:
      enabled: false
rbac:
  enabled: true

service:
  enabled: true
  type: LoadBalancer
  annotations: {}
  labels: {}
  spec:
    loadBalancerIP: 10.0.10.250 # this should be an IP in the MetalLB range
  loadBalancerSourceRanges: []
  externalIPs: []

nodeSelector:
  worker: 'true'
```

### Kubernetes Manifest (Authentik Middleware)
```
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
    name: authentik
    namespace: authentik
spec:
    forwardAuth:
        address: http://authentik-server.authentik:9000/outpost.goauthentik.io/auth/traefik
        trustForwardHeader: true
        authResponseHeaders:
            - X-authentik-username
            - X-authentik-groups
            - X-authentik-email
            - X-authentik-name
            - X-authentik-uid
            - X-authentik-jwt
            - X-authentik-meta-jwks
            - X-authentik-meta-outpost
            - X-authentik-meta-provider
            - X-authentik-meta-app
            - X-authentik-meta-version
```

### Kubernetes Manifest (Dashboard Ingress)
```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: traefik-dashboard
  namespace: default
  annotations: 
    kubernetes.io/ingress.class: traefik-external
spec:
  entryPoints:
    - websecure
  routes:
    - match: Host(`traefik.f9.casa`)
      kind: Rule
      middlewares:
        - name: authentik
          namespace: authentik
      services:
        - name: api@internal
          namespace: traefik
          kind: TraefikService
  tls:
    secretName: f9-casa-tls
```

### Docker Compose
> I use port mode "host" so i can get the correct IP Address for Crowdsec, due to this i force the container to node 1
I also use 2 different entry points, http/https and web/websecure, http/https is internal only, web/websecure is port forwarded
{: .prompt-info }

```yaml
version: '3.9'
services:
  traefik:
    image: traefik:v2.11.0
    ports:
      - target: 80
        published: 80
        protocol: tcp
        mode: host
      - target: 81
        published: 81
        protocol: tcp
        mode: host
      - target: 443
        published: 443
        protocol: tcp
        mode: host
      - target: 444
        published: 444
        protocol: tcp
        mode: host
    environment:
      CF_DNS_API_TOKEN: [REDACTED]

    deploy:
      labels:
        - traefik.enable=true
        - traefik.http.routers.traefik.rule=Host(`traefik.f9.casa`)
        - traefik.http.routers.traefik.entrypoints=https
        - traefik.http.routers.traefik.tls.certresolver=letsencrypt
        - traefik.http.routers.traefik.service=api@internal
        - traefik.http.routers.traefik.middlewares=strip
        - traefik.http.middlewares.strip.stripprefix.prefixes=/traefik
        - traefik.http.services.dummy-svc.loadbalancer.server.port=9999

        - "traefik.http.routers.traefik.middlewares=authentik@docker"
             
        - homepage.group=Networking
        - homepage.name=Traefik
        - homepage.icon=traefik.png
        - homepage.href=https://traefik.f9.casa
        - homepage.description=Reverse Proxy
        - homepage.siteMonitor=http://traefik:8080/ping
        - homepage.widget.type=traefik
        - homepage.widget.url=http://traefik:8080
      mode: replicated
      placement:
        constraints:
          # Make the traefik service run only on the node with this label
          # as the node with it has the volume for the certificates
          - node.labels.traefik == true
  
      update_config:
        order: stop-first   

    volumes:
      # Add Docker as a mounted volume, so that Traefik can read the labels of other services
      - /var/run/docker.sock:/var/run/docker.sock:ro
      # Make sure the volume folder is created
      - /srv/cephfs/docker/appdata/traefik/acme.json:/letsencrypt/acme.json
      - /srv/cephfs/docker/appdata/traefik/providers:/providers
      - /srv/cephfs/docker/appdata/traefik/logs:/var/log/traefik

    command:
      # Tell Traefik to discover containers using the Docker API
      - --providers.docker
      - --providers.docker.network=traefik-public
      - --providers.docker.exposedbydefault=false
      - --providers.docker.swarmmode

      - --providers.file.directory=/providers
      - --providers.file.watch=true
      
      # Enable the Trafik dashboard
      - --api=true
      - --api.insecure=true
      - --api.dashboard=true
      - --ping=true

      # Disable Backend Cert Verification
      - --serversTransport.insecureSkipVerify=true
      
      # Set up LetsEncrypt
      - --certificatesresolvers.letsencrypt.acme.dnschallenge=true
      - --certificatesresolvers.letsencrypt.acme.dnschallenge.provider=cloudflare
      - --certificatesresolvers.letsencrypt.acme.email=scottleeallen@gmail.com
      - --certificatesresolvers.letsencrypt.acme.storage=/letsencrypt/acme.json
      # !IMPORTANT - COMMENT OUT THE FOLLOWING LINE IN PRODUCTION!
      #- --certificatesresolvers.letsencrypt.acme.caserver=https://acme-staging-v02.api.letsencrypt.org/directory
       
      # Set up an insecure PUBLIC listener that redirects all traffic to TLS
      - --entrypoints.web.address=:81
      - --entrypoints.web.http.redirections.entrypoint.to=websecure
      - --entrypoints.web.http.redirections.entrypoint.scheme=https
      - --entryPoints.web.forwardedHeaders.trustedIPs=173.245.48.0/20,103.21.244.0/22,103.22.200.0/22,103.31.4.0/22,141.101.64.0/18,108.162.192.0/18,190.93.240.0/20,188.114.96.0/20,197.234.240.0/22,198.41.128.0/17,162.158.0.0/15,104.16.0.0/13,104.24.0.0/14,172.64.0.0/13,131.0.72.0/22

      - --entrypoints.websecure.address=:444
      
      # Set up an insecure PRIVATE listener that redirects all traffic to TLS
      - --entrypoints.http.address=:80
      - --entrypoints.http.http.redirections.entrypoint.to=https
      - --entrypoints.http.http.redirections.entrypoint.scheme=https
      - --entrypoints.https.address=:443
      
      # Set up the TLS configuration for our PUBLIC websecure listener
      - --entrypoints.websecure.http.tls=true
      - --entrypoints.websecure.http.tls.certResolver=letsencrypt
      - --entrypoints.websecure.http.tls.domains[0].main=f9.casa
      - --entrypoints.websecure.http.tls.domains[0].sans=*.f9.casa
      - --entryPoints.websecure.forwardedHeaders.trustedIPs=173.245.48.0/20,103.21.244.0/22,103.22.200.0/22,103.31.4.0/22,141.101.64.0/18,108.162.192.0/18,190.93.240.0/20,188.114.96.0/20,197.234.240.0/22,198.41.128.0/17,162.158.0.0/15,104.16.0.0/13,104.24.0.0/14,172.64.0.0/13,131.0.72.0/22
      
      # Set up the TLS configuration for our PRIVATE websecure listener
      - --entrypoints.https.http.tls=true
      - --entrypoints.https.http.tls.certResolver=letsencrypt
      - --entrypoints.https.http.tls.domains[0].main=f9.casa
      - --entrypoints.https.http.tls.domains[0].sans=*.f9.casa
      
      # # Set up the DNS TCP/UDP configuration for our dns-xxx listener
      # - --entryPoints.dns-udp.address=:53/udp
      # - --entryPoints.dns-tcp.address=:53/tcp
      
      - --log.level=DEBUG
      
      - --accesslog=true
      - --accesslog.filepath=/var/log/traefik/access.log

      - --entrypoints.http.http.middlewares=crowdsec-bouncer@file
      - --entrypoints.https.http.middlewares=crowdsec-bouncer@file
      - --entrypoints.web.http.middlewares=crowdsec-bouncer@file
      - --entrypoints.websecure.http.middlewares=crowdsec-bouncer@file
    networks:
      # Use the public network created to be shared between Traefik and
      # any other service that needs to be publicly available with HTTPS
      - traefik-public

  crowdsec:
    image: crowdsecurity/crowdsec:latest
    container_name: crowdsec
    environment:
     - GID=1000
     - COLLECTIONS=crowdsecurity/linux crowdsecurity/traefik
    volumes:
      - /srv/cephfs/docker/appdata/crowdsec:/etc/crowdsec
      - /srv/cephfs/docker/appdata/traefik/logs:/var/log/traefik/:ro
    networks:
      - traefik-public

  crowdsec-bouncer:
    image: fbonalair/traefik-crowdsec-bouncer
    container_name: bouncer-traefik
    hostname: bouncer-traefik
    environment:
      - CROWDSEC_BOUNCER_API_KEY=[REDACTED]
      - CROWDSEC_AGENT_HOST=crowdsec:8080
      - GIN_MODE=release
    networks:
      - traefik-public

networks:
  traefik-public:
    external: true

```

### Docker Configuration
crowdsec/acquis.yaml
```yaml
filenames:
  - /var/log/traefik/*
labels:
  type: traefik
```

crowdsec/config.yaml - PostegreSQL configuration
```yaml
db_config:
  log_level: info
  type: postgres
  user: crowdsec
  password: "[REDACTED]"
  db_name: crowdsec
  host: postgresql
  port: 5432
```
