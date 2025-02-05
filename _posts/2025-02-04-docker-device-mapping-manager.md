---
layout: post
title: "Device Mapping Manager - Mount Devices In Docker Swarm"
date: 2025-02-05 10:00:00 0000
categories: services utility
tags: homelab utility docker swarm devicemapper device-mapper-manager

---
[Device Mapping Manager](https://github.com/allfro/device-mapping-manager) maps and enables devices into containers running on docker swarm. It is currently only compatible with linux systems that use cgroup v1 and v2.

### Docker Compose
```yaml
version: "3.8"
services:
  dmm:
    image: docker
    entrypoint: docker
    command: |
      run 
      -i
      --restart always 
      --privileged 
      --cgroupns=host 
      --pid=host 
      --userns=host 
      -v /sys:/host/sys 
      -v /var/run/docker.sock:/var/run/docker.sock 
      ghcr.io/allfro/allfro/device-mapping-manager:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    deploy:
      mode: global
```