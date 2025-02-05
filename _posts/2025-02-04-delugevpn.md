---
layout: post
title: "DelugeVPN - Deluge torrent client with built in VPN support"
date: 2025-02-05 10:00:00 0000
categories: services downloader
tags: homelab downloader deluge delugevpn

---

## [DelugeVPN](https://www.deluge-torrent.org/)
ghcr.io/binhex/arch-delugevpn

Deluge BitTorrent Client is a free and open-source, cross-platform BitTorrent client written in Python. Deluge uses a front and back end architecture where libtorrent, a software library written in C++ which provides the application's networking logic, is connected to one of various front ends including a text console, the web interface and a graphical desktop interface using GTK through the project's own Python bindings. 

This service has a built in Wireguard VPN.

> Since this uses media from my NAS i have simply decided to run this as a Docker Container on UnRAID on my NAS instead of having it as part of my kubernetes cluster to remove the need to access the NAS media remotely
{: .prompt-info }