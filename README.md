## k8s-mediaserver-operator - Your all-in-one resource for your media needs!

I am so happy to announce the first release of the **k8s-mediaserver-operator**, a project that mixes up some of the mainstream tools for your media needs.

The Custom Resource that is implemented, allows you to create a fully working and complete media server on #kubernetes, based on:

[Plex Media Server](https://www.plex.tv/ "Plex Media Server") - A complete and fully funtional mediaserver that allows you to render in a webUI your movies, TV Series, podcasts, video streams. 

[Sonarr](https://sonarr.tv/ "Sonarr") - A TV series and show tracker, that allows the integration with download managers for searching and retrieving TV Series, organizing them, schedule notifications when an episode comes up and much more.

[Radarr](https://radarr.video/ "Radarr") - The same a **Sonarr**, but for movies!

[Jackett](https://github.com/Jackett/Jackett "Jackett") - An API interface that keeps easy your life interacting with trackers for torrents.

[Transmission](https://transmissionbt.com/ "Transmission") - A fast, easy and reliable torrent client.

[Sabnzbd](https://sabnzbd.org/ "Sabnzbd") - A free and easy binary newsreader.

All container images used by the operator are from [linuxserver](https://github.com/linuxserver) - [linuxserver.io](https://www.linuxserver.io/ "linuxserver.io")

Each of the components can be **enabled** or **disabled** if you already have something in place in your lab!

## Introduction

I started working on this project because I was tired of using the 'containerized' version with docker/podman-compose, and I wanted to experiment a bit both with helm and operators.

It is quite simple to use, and very minimalistic, with customizations that are strictly related to usability and access, rather than complex customizations, even if, maybe in the future, we could add it!

Each container has its *init container* in order to initialize configurations on the PV before starting the actual pod and avoid to restart the pods.

## QuickStart

The operator and the CR are already configured with some defaults settings to make you jump and go with it.

All you need is:

- A namespace where you want to put your CR and all the pods that will spawn
- Being able to provision an RWX PV where to store configurations, downloads, and all related stuff (suggested > 200GB). 
Persistent Volume **or** StorageClasses for dynamically provisioned volumes are **REQUIRED**

First install the CRD and the operator:

AMD/Intel: 

    kubectl apply -f k8s-mediaserver-operator.yml

ARM - Raspberry Pi: 

    kubectl apply -f k8s-mediaserver-operator-arm64.yml

Then you are good to go with the CR:

    kubectl apply -f k8s-mediaserver.yml

In seconds, you will be ready to use your applications!

With default settings, your applications will run in these paths:

    http://k8s-mediaserver.k8s.test/sonarr
    http://k8s-mediaserver.k8s.test/radarr
    http://k8s-mediaserver.k8s.test/transmission
    http://k8s-mediaserver.k8s.test/jackett

    http://k8s-plex.k8s.test/

## The mediaserver CR
The CR is quite simple to configure, and I tried to keep the number of parameters low to avoid confusion, but still letting some customization to fit the resource inside your cluster.

# General config

| Config path | Meaning | Default | 
| ------------ | ------------ | ------------ |
| general.ingress_host | The hostname to use in ingress definition, this will be the hostname where the applications will be exposed  | k8s-mediaserver.k8s.test |
| general.plex_ingress_host | The hostname to use for **PLEX** as it must be exposed on a / dedicated path  | k8s-plex.k8s.test |
| general.image_tag | The name of the image tag (arm32v7-latest, arm64v8-latest, development)  | latest |
| general.pgid | The GID for the process | 1000 |
| general.puid | The UID for the process | 1000 |
| general.nodeSelector | Node Selector for all the pods | {} |
| general.storage.customVolume | Flag if you want to supply your own volume and not use a PVC | false |
| general.storage.pvcName  | Name of the persistenVolumeClaim configured in deployments | mediaserver-pvc |
| general.storage.pvcStorageClass  | Specifies a storageClass for the PVC | "" |
| general.storage.size | Size of the persistenVolume | 50Gi |
| general.storage.subPaths.tv | Default subpath for tv series folder on used storage | media/tv |
| general.storage.subPaths.movies | Default subpath for movies folder on used storage | media/movies |
| general.storage.subPaths.downloads | Default root path for downloads for both sabnzbd and transmission on used storage | downloads |
| general.storage.subPaths.transmission | Default subpath for transmission downloads on used storage | general.storage.subPaths.downloads/transmission |
| general.storage.subPaths.sabnzbd | Default subpath for sabnzbd downloads on used storage | general.storage.subPaths.downloads/sabnzbd |
| general.storage.subPaths.config | Default subpath for all config files on used storage | config |
| general.storage.volumes | Supply custom volume to be mounted for all services | {} |

# Plex

| Config path | Meaning | Default | 
| ------------ | ------------ | ------------ |
| plex.enabled | Flag if you want to enable plex | true | 
| plex.claim | **IMPORTANT** Token from your account, needed to claim the server | CHANGEME |
| plex.replicaCount | Number of replicas serving plex | 1 | 
| plex.container.port | The port in use by the container | 32400 | 
| plex.service.type | The kind of Service (ClusterIP/NodePort/LoadBalancer) | ClusterIP |
| plex.service.port | The port assigned to the service | 32400 |
| plex.service.nodePort | In case of service.type NodePort, the nodePort to use | "" |
| plex.service.extraLBService | If true, creates an additional LoadBalancer service with '-lb' suffix (requires a cloud provider or metalLB) | false | 
| plex.ingress.enabled | If true, creates the ingress resource for the application | true |
| plex.ingress.annotations | Additional field for annotations, if needed | {} |
| plex.ingress.path | The path where the application is exposed | /plex |
| plex.ingress.tls.enabled | If true, tls is enabled | false |
| plex.ingress.tls.secretName | Name of the secret holding certificates for the secure ingress | "" |
| plex.resources | Limits and Requests for the container | {} | 

# Sonarr

| Config path | Meaning | Default | 
| ------------ | ------------ | ------------ |
| sonarr.enabled | Flag if you want to enable sonarr | true | 
| sonarr.container.port | The port in use by the container | 8989 | 
| sonarr.service.type | The kind of Service (ClusterIP/NodePort/LoadBalancer) | ClusterIP |
| sonarr.service.port | The port assigned to the service | 8989 |
| sonarr.service.nodePort | In case of service.type NodePort, the nodePort to use | "" |
| sonarr.service.extraLBService | If true, creates an additional LoadBalancer service with '-lb' suffix (requires a cloud provider or metalLB) | false 
| sonarr.ingress.enabled | If true, creates the ingress resource for the application | true |
| sonarr.ingress.annotations | Additional field for annotations, if needed | {} |
| sonarr.ingress.path | The path where the application is exposed | /sonarr |
| sonarr.ingress.tls.enabled | If true, tls is enabled | false |
| sonarr.ingress.tls.secretName | Name of the secret holding certificates for the secure ingress | "" | 
| sonarr.resources | Limits and Requests for the container | {} |

# Radarr

| Config path | Meaning | Default | 
| ------------ | ------------ | ------------ |
| radarr.enabled | Flag if you want to enable radarr | true | 
| radarr.container.port | The port in use by the container | 7878 | 
| radarr.service.type | The kind of Service (ClusterIP/NodePort/LoadBalancer) | ClusterIP |
| radarr.service.port | The port assigned to the service | 7878 |
| radarr.service.nodePort | In case of service.type NodePort, the nodePort to use | "" |
| radarr.service.extraLBService | If true, creates an additional LoadBalancer service with '-lb' suffix (requires a cloud provider or metalLB) | false | 
| radarr.ingress.enabled | If true, creates the ingress resource for the application | true |
| radarr.ingress.annotations | Additional field for annotations, if needed | {} |
| radarr.ingress.path | The path where the application is exposed | /radarr  |
| radarr.ingress.tls.enabled | If true, tls is enabled | false |
| radarr.ingress.tls.secretName | Name of the secret holding certificates for the secure ingress | "" | 
| radarr.resources | Limits and Requests for the container | {} |

# Jackett

| Config path | Meaning | Default | 
| ------------ | ------------ | ------------ |
| jackett.enabled | Flag if you want to enable jackett | true | 
| jackett.container.port | The port in use by the container | 9117 | 
| jackett.service.type | The kind of Service (ClusterIP/NodePort/LoadBalancer) | ClusterIP |
| jackett.service.port | The port assigned to the service | 9117 |
| jackett.service.nodePort | In case of service.type NodePort, the nodePort to use | "" |
| jackett.service.extraLBService | If true, it creates an additional LoadBalancer service with '-lb' suffix (requires a cloud provider or metalLB) | false | 
| jackett.ingress.enabled | If true, creates the ingress resource for the application | true |
| jackett.ingress.annotations | Additional field for annotations, if needed | {} |
| jackett.ingress.path | The path where the application is exposed | /jackett |
| jackett.ingress.tls.enabled | If true, tls is enabled | false |
| jackett.ingress.tls.secretName | Name of the secret holding certificates for the secure ingress | "" | 
| jackett.resources | Limits and Requests for the container | {} |

# Transmission

| Config path | Meaning | Default | 
| ------------ | ------------ | ------------ |
| transmission.enabled | Flag if you want to enable transmission | true | 
| transmission.container.port.utp | The port in use by the container | 9091 | 
| transmission.container.port.peer | The port in use by the container for peer connection | 51413 | 
| transmission.service.utp.type | The kind of Service (ClusterIP/NodePort/LoadBalancer) for transmission itself | ClusterIP |
| transmission.service.utp.port | The port assigned to the service for transmission itself | 9091 |
| transmission.service.utp.nodePort | In case of service.type NodePort, the nodePort to use for transmission itself | "" |
| transmission.service.utp.extraLBService | If true, creates an additional LoadBalancer service with '-lb' suffix (requires a cloud provider or metalLB) | false | 
| transmission.service.peer.type | The kind of Service (ClusterIP/NodePort/LoadBalancer) for peer port | ClusterIP |
| transmission.service.peer.port | The port assigned to the service for peer port | 51413 |
| transmission.service.peer.nodePort | In case of service.type NodePort, the nodePort to use for peer port | "" |
| transmission.service.peer.nodePortUDP | In case of service.type NodePort, the nodePort to use for peer port UDP service | "" |
| transmission.service.peer.extraLBService | If true, creates an additional LoadBalancer service with '-lb' suffix (requires a cloud provider or metalLB) | false | 
| transmission.ingress.enabled | If true, creates the ingress resource for the application | true |
| transmission.ingress.annotations | Additional field for annotations, if needed | {} |
| transmission.ingress.path | The path where the application is exposed | /transmission |
| transmission.ingress.tls.enabled | If true, tls is enabled | false |
| transmission.ingress.tls.secretName | Name of the secret holding certificates for the secure ingress | "" |
| transmission.config.auth.enabled | Enables authentication for transmission | false |
| transmission.config.auth.username | Username for transmission | "" |
| transmission.config.auth.password | Password for transmission | "" | 
| transmission.resources | Limits and Requests for the container | {} |

# Sabnzbd

| Config path | Meaning | Default | 
| ------------ | ------------ | ------------ |
| sabnzbd.enabled | Flag if you want to enable sabnzbd | true | 
| sabnzbd.container.port.http | The port in use by the container | 8080 | 
| sabnzbd.container.port.https | The port in use by the container for peer connection | 9090 | 
| sabnzbd.service.http.type | The kind of Service (ClusterIP/NodePort/LoadBalancer) for sabnzbd itself | ClusterIP |
| sabnzbd.service.http.port | The port assigned to the service for sabnzbd itself | 9091 |
| sabnzbd.service.http.nodePort | In case of service.type NodePort, the nodePort to use for sabnzbd itself | "" |
| sabnzbd.service.http.extraLBService | If true, creates an additional LoadBalancer service with '-lb' suffix (requires a cloud provider or metalLB) | false | 
| sabnzbd.service.https.type | The kind of Service (ClusterIP/NodePort/LoadBalancer) for https port | ClusterIP |
| sabnzbd.service.https.port | The port assigned to the service for peer port | 51413 |
| sabnzbd.service.https.nodePort | In case of service.type NodePort, the nodePort to use for https port | "" |
| sabnzbd.service.https.extraLBService | If true, creates an additional LoadBalancer service with '-lb' suffix (requires a cloud provider or metalLB) | false | 
| sabnzbd.ingress.enabled | If true, creates the ingress resource for the application | true |
| sabnzbd.ingress.annotations | Additional field for annotations, if needed | {} |
| sabnzbd.ingress.path | The path where the application is exposed | /sabnzbd |
| sabnzbd.ingress.tls.enabled | If true, tls is enabled | false |
| sabnzbd.ingress.tls.secretName | Name of the secret holding certificates for the secure ingress | "" | 
| sabnzbd.resources | Limits and Requests for the container | {} |

## About the project

This project is intended as an exercise, and absolutely for fun. 
This is not intended to promote piracy. 

Also feel free to contribute and extend it!

