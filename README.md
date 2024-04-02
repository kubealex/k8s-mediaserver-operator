# k8s-mediaserver-operator

Your all-in-one resource for your media needs!

I am so happy to announce the first release of the **k8s-mediaserver-operator**, a project that mixes up some of the
mainstream tools for your media needs.

The Custom Resource that is implemented, allows you to create a fully working and complete media server on #kubernetes,
based on:

[Plex Media Server](https://www.plex.tv/ "Plex Media Server") - A complete and fully funtional mediaserver that allows
you to render in a webUI your movies, TV Series, podcasts, video streams.

[Jellyfin](https://jellyfin.org/ "JellyFin") - It is an alternative to the proprietary Emby and Plex, to provide media from a dedicated server to end-user devices via multiple apps

[Sonarr](https://sonarr.tv/ "Sonarr") - A TV series and show tracker, that allows the integration with download managers
for searching and retrieving TV Series, organizing them, schedule notifications when an episode comes up and much more.

[Radarr](https://radarr.video/ "Radarr") - The same a **Sonarr**, but for movies!

[Jackett](https://github.com/Jackett/Jackett "Jackett") - An API interface that keeps easy your life interacting with
trackers for torrents.

[Prowlarr](https://github.com/Prowlarr/Prowlarr "Prowlarr") - An indexer manager/proxy built on the popular \*arr .net/reactjs base stack to integrate with your various PVR apps. Prowlarr supports management of both Torrent Trackers and Usenet Indexers.

[Transmission](https://transmissionbt.com/ "Transmission") - A fast, easy and reliable torrent client.

[Sabnzbd](https://sabnzbd.org/ "Sabnzbd") - A free and easy binary newsreader.

All container images used by the operator are from [linuxserver](https://github.com/linuxserver)

- [linuxserver.io](https://www.linuxserver.io/ "linuxserver.io")

Each of the components can be **enabled** or **disabled** if you already have something in place in your lab!

## Introduction

I started working on this project because I was tired of using the 'containerized' version with docker/podman-compose,
and I wanted to experiment a bit both with helm and operators.

It is quite simple to use, and very minimalistic, with customizations that are strictly related to usability and access,
rather than complex customizations, even if, maybe in the future, we could add it!

Each container has its _init container_ in order to initialize configurations on the PV before starting the actual pod
and avoid to restart the pods.

## QuickStart

The operator and the CR are already configured with some defaults settings to make you jump and go with it.

All you need is:

- A namespace where you want to put your Custom Resource and all the pods that will spawn
- Being able to provision an RWX PV where to store configurations, downloads, and all related stuff (suggested > 200GB).
  Persistent Volume **or** StorageClasses for dynamically provisioned volumes are **REQUIRED** (See below for NFS)

1. First install the CRD and the operator:

AMD/Intel:

```shell
kubectl apply -f k8s-mediaserver-operator.yml
```

ARM - Raspberry Pi:

```shell
kubectl apply -f k8s-mediaserver-operator-arm64.yml
```

2. Install the custom resource with the default values:

```shell
kubectl apply -f k8s-mediaserver.yml
```

In seconds, you will be ready to use your applications!

With default settings, your applications will run in these paths:

| Service      | Link                                         |
| ------------ | -------------------------------------------- |
| Sonarr       | http://k8s-mediaserver.k8s.test/sonarr       |
| Radarr       | http://k8s-mediaserver.k8s.test/radarr       |
| Transmission | http://k8s-mediaserver.k8s.test/transmission |
| Jackett      | http://k8s-mediaserver.k8s.test/jackett      |
| Prowlarr     | http://k8s-mediaserver.k8s.test/prowlarr     |
| Jellyfin     | http://k8s-jelly.k8s.test/                   |
| PLEX         | http://k8s-plex.k8s.test/                    |

3. (Optional) Use custom values:

If you want to use your custom setup for all the services:

- Copy the default values files `cp ./helm-charts/k8s-mediaserver/values.yaml my-values.yaml`
- Make all the changes you want in the new file `./helm-charts/k8s-mediaserver/my-values.yaml`

With this value saved in the top level directory of this repo, running the below will add the resources to your cluster,
under the helm release name `k8s-mediaserver`

```shell
helm install -f my-values.yaml k8s-mediaserver ./helm-charts/k8s-mediaserver/
```

To make changes to the deploy

```shell
helm upgrade -f my-values.yaml k8s-mediaserver ./helm-charts/k8s-mediaserver/
```

This is equivalent to running `mount {SERVER-IP}:/mount/path/on/nfs/server ...` in each container, where `...` is
different per resource, as defined in the templates directory (for each resource).
In addition to the above, you should also edit your subpaths so that they when they are appended to your `path:`
specified in `values.yaml`, they map to the directories you intend.

## The mediaserver CR

The CR is quite simple to configure, and I tried to keep the number of parameters low to avoid confusion, but still
letting some customization to fit the resource inside your cluster.

## General config

| Config path                           | Meaning                                                                                                     | Default                                         |
| ------------------------------------- | ----------------------------------------------------------------------------------------------------------- | ----------------------------------------------- |
| general.ingress_host                  | The hostname to use in ingress definition, this will be the hostname where the applications will be exposed | k8s-mediaserver.k8s.test                        |
| general.plex_ingress_host             | The hostname to use for **PLEX** as it must be exposed on a / dedicated path                                | k8s-plex.k8s.test                               |
| general.jellyfin_ingress_host         | The hostname to use for **JellyFin** as it must be exposed on a / dedicated path                            | k8s-jelly.k8s.test                              |
| general.image_tag                     | The name of the image tag (arm32v7-latest, arm64v8-latest, development)                                     | latest                                          |
| general.pgid                          | The GID for the process                                                                                     | 1000                                            |
| general.puid                          | The UID for the process                                                                                     | 1000                                            |
| general.nodeSelector                  | Default Node Selector for all the pods. Per-service nodeSelectors are merged against this.                  | {}                                              |
| general.storage.customVolume          | Flag if you want to supply your own volume and not use a PVC                                                | false                                           |
| general.storage.pvcName               | Name of the persistenVolumeClaim configured in deployments                                                  | mediaserver-pvc                                 |
| general.storage.accessMode            | Access mode for mediaserver PVC in case of single nodes                                                     | ReadWriteMany                                   |
| general.storage.pvcStorageClass       | Specifies a storageClass for the PVC                                                                        | ""                                              |
| general.storage.size                  | Size of the persistenVolume                                                                                 | 50Gi                                            |
| general.storage.subPaths.tv           | Default subpath for tv series folder on used storage                                                        | media/tv                                        |
| general.storage.subPaths.movies       | Default subpath for movies folder on used storage                                                           | media/movies                                    |
| general.storage.subPaths.downloads    | Default root path for downloads for both sabnzbd and transmission on used storage                           | downloads                                       |
| general.storage.subPaths.transmission | Default subpath for transmission downloads on used storage                                                  | general.storage.subPaths.downloads/transmission |
| general.storage.subPaths.sabnzbd      | Default subpath for sabnzbd downloads on used storage                                                       | general.storage.subPaths.downloads/sabnzbd      |
| general.storage.subPaths.config       | Default subpath for all config files on used storage                                                        | config                                          |
| general.storage.volumes               | Supply custom volume to be mounted for all services                                                         | {}                                              |
| general.ingress.ingressClassName      | Reference an IngressClass resource that contains additional Ingress configuration                           | ""                                              |
| general.ingress.enableSubdomains      | If set to true each application will be accesible also as a subdomain. e.x. plex.domain.com                 | false                                           |

### Plex

| Config path                             | Meaning                                                                                                       | Default                    |
| --------------------------------------- | ------------------------------------------------------------------------------------------------------------- | -------------------------- |
| plex.enabled                            | Flag if you want to enable Plex                                                                               | true                       |
| plex.claim                              | **IMPORTANT** Token from your account, needed to claim the server                                             | CHANGEME                   |
| plex.replicaCount                       | Number of replicas serving Plex                                                                               | 1                          |
| plex.container.nodeSelector             | Node Selector for the Plex pods                                                                               | {}                         |
| plex.container.port                     | The port in use by the container                                                                              | 32400                      |
| plex.container.image                    | The image used by the container                                                                               | docker.io/linuxserver/plex |
| plex.container.tag                      | The tag used by the container                                                                                 | null                       |
| plex.service.type                       | The kind of Service (ClusterIP/NodePort/LoadBalancer)                                                         | ClusterIP                  |
| plex.service.port                       | The port assigned to the service                                                                              | 32400                      |
| plex.service.nodePort                   | In case of service.type NodePort, the nodePort to use                                                         | ""                         |
| plex.service.extraLBService             | If true, creates an additional LoadBalancer service with '-lb' suffix (requires a cloud provider or metalLB)  | false                      |
| plex.service.extraLBService.annotations | Instead of using extraLBService as a bool, you can use it as a map to define annotations on the loadbalancer  | null                       |
| plex.ingress.enabled                    | If true, creates the ingress resource for the application                                                     | true                       |
| plex.ingress.annotations                | Additional field for annotations, if needed                                                                   | {}                         |
| plex.ingress.path                       | The path where the application is exposed                                                                     | /plex                      |
| plex.ingress.tls.enabled                | If true, tls is enabled                                                                                       | false                      |
| plex.ingress.tls.secretName             | Name of the secret holding certificates for the secure ingress                                                | ""                         |
| plex.resources                          | Limits and Requests for the container                                                                         | {}                         |
| plex.volume                             | If set, Plex will create a PVC for it's config volume, else it will be put on general.storage.subPaths.config | {}                         |

### Jellyfin

| Config path                                 | Meaning                                                                                                           | Default                        |
| ------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- | ------------------------------ |
| jellyfin.enabled                            | Flag if you want to enable Plex                                                                                   | true                           |
| jellyfin.replicaCount                       | Number of replicas serving Plex                                                                                   | 1                              |
| jellyfin.container.nodeSelector             | Node Selector for the Plex pods                                                                                   | {}                             |
| jellyfin.container.port                     | The port in use by the container                                                                                  | 8096                           |
| jellyfin.container.image                    | The image used by the container                                                                                   | docker.io/linuxserver/jellyfin |
| jellyfin.container.tag                      | The tag used by the container                                                                                     | null                           |
| jellyfin.service.type                       | The kind of Service (ClusterIP/NodePort/LoadBalancer)                                                             | ClusterIP                      |
| jellyfin.service.port                       | The port assigned to the service                                                                                  | 8096                           |
| jellyfin.service.nodePort                   | In case of service.type NodePort, the nodePort to use                                                             | ""                             |
| jellyfin.service.extraLBService             | If true, creates an additional LoadBalancer service with '-lb' suffix (requires a cloud provider or metalLB)      | false                          |
| jellyfin.service.extraLBService.annotations | Instead of using extraLBService as a bool, you can use it as a map to define annotations on the loadbalancer      | null                           |
| jellyfin.ingress.enabled                    | If true, creates the ingress resource for the application                                                         | true                           |
| jellyfin.ingress.annotations                | Additional field for annotations, if needed                                                                       | {}                             |
| jellyfin.ingress.path                       | The path where the application is exposed                                                                         | /jellyfin                      |
| jellyfin.ingress.tls.enabled                | If true, tls is enabled                                                                                           | false                          |
| jellyfin.ingress.tls.secretName             | Name of the secret holding certificates for the secure ingress                                                    | ""                             |
| jellyfin.resources                          | Limits and Requests for the container                                                                             | {}                             |
| jellyfin.volume                             | If set, Jellyfin will create a PVC for it's config volume, else it will be put on general.storage.subPaths.config | {}                             |

### Sonarr

| Config path                               | Meaning                                                                                                       | Default                      |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------------- | ---------------------------- |
| sonarr.enabled                            | Flag if you want to enable Sonarr                                                                             | true                         |
| sonarr.container.port                     | The port in use by the container                                                                              | 8989                         |
| sonarr.container.nodeSelector             | Node Selector for the Sonarr pods                                                                             | {}                           |
| sonarr.container.image                    | The image used by the container                                                                               | docker.io/linuxserver/sonarr |
| sonarr.container.tag                      | The tag used by the container                                                                                 | null                         |
| sonarr.service.type                       | The kind of Service (ClusterIP/NodePort/LoadBalancer)                                                         | ClusterIP                    |
| sonarr.service.port                       | The port assigned to the service                                                                              | 8989                         |
| sonarr.service.nodePort                   | In case of service.type NodePort, the nodePort to use                                                         | ""                           |
| sonarr.service.extraLBService             | If true, creates an additional LoadBalancer service with '-lb' suffix (requires a cloud provider or metalLB)  | false                        |
| sonarr.service.extraLBService.annotations | Instead of using extraLBService as a bool, you can use it as a map to define annotations on the loadbalancer  | null                         |
| sonarr.ingress.enabled                    | If true, creates the ingress resource for the application                                                     | true                         |
| sonarr.ingress.annotations                | Additional field for annotations, if needed                                                                   | {}                           |
| sonarr.ingress.path                       | The path where the application is exposed                                                                     | /sonarr                      |
| sonarr.ingress.tls.enabled                | If true, tls is enabled                                                                                       | false                        |
| sonarr.ingress.tls.secretName             | Name of the secret holding certificates for the secure ingress                                                | ""                           |
| sonarr.resources                          | Limits and Requests for the container                                                                         | {}                           |
| sonarr.volume                             | If set, Plex will create a PVC for it's config volume, else it will be put on general.storage.subPaths.config | {}                           |

### Radarr

| Config path                               | Meaning                                                                                                       | Default                      |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------------- | ---------------------------- |
| radarr.enabled                            | Flag if you want to enable Radarr                                                                             | true                         |
| radarr.container.port                     | The port in use by the container                                                                              | 7878                         |
| radarr.container.nodeSelector             | Node Selector for the Radarr pods                                                                             | {}                           |
| radarr.container.image                    | The image used by the container                                                                               | docker.io/linuxserver/radarr |
| radarr.container.tag                      | The tag used by the container                                                                                 | null                         |
| radarr.service.type                       | The kind of Service (ClusterIP/NodePort/LoadBalancer)                                                         | ClusterIP                    |
| radarr.service.port                       | The port assigned to the service                                                                              | 7878                         |
| radarr.service.nodePort                   | In case of service.type NodePort, the nodePort to use                                                         | ""                           |
| radarr.service.extraLBService             | If true, creates an additional LoadBalancer service with '-lb' suffix (requires a cloud provider or metalLB)  | false                        |
| radarr.service.extraLBService.annotations | Instead of using extraLBService as a bool, you can use it as a map to define annotations on the loadbalancer  | null                         |
| radarr.ingress.enabled                    | If true, creates the ingress resource for the application                                                     | true                         |
| radarr.ingress.annotations                | Additional field for annotations, if needed                                                                   | {}                           |
| radarr.ingress.path                       | The path where the application is exposed                                                                     | /radarr                      |
| radarr.ingress.tls.enabled                | If true, tls is enabled                                                                                       | false                        |
| radarr.ingress.tls.secretName             | Name of the secret holding certificates for the secure ingress                                                | ""                           |
| radarr.resources                          | Limits and Requests for the container                                                                         | {}                           |
| radarr.volume                             | If set, Plex will create a PVC for it's config volume, else it will be put on general.storage.subPaths.config | {}                           |

### Jackett

| Config path                                | Meaning                                                                                                         | Default                       |
| ------------------------------------------ | --------------------------------------------------------------------------------------------------------------- | ----------------------------- |
| jackett.enabled                            | Flag if you want to enable Jackett                                                                              | true                          |
| jackett.container.port                     | The port in use by the container                                                                                | 9117                          |
| jackett.container.nodeSelector             | Node Selector for the Jackett pods                                                                              | {}                            |
| jackett.container.image                    | The image used by the container                                                                                 | docker.io/linuxserver/jackett |
| jackett.container.tag                      | The tag used by the container                                                                                   | null                          |
| jackett.service.type                       | The kind of Service (ClusterIP/NodePort/LoadBalancer)                                                           | ClusterIP                     |
| jackett.service.port                       | The port assigned to the service                                                                                | 9117                          |
| jackett.service.nodePort                   | In case of service.type NodePort, the nodePort to use                                                           | ""                            |
| jackett.service.extraLBService             | If true, it creates an additional LoadBalancer service with '-lb' suffix (requires a cloud provider or metalLB) | false                         |
| jackett.service.extraLBService.annotations | Instead of using extraLBService as a bool, you can use it as a map to define annotations on the loadbalancer    | null                          |
| jackett.ingress.enabled                    | If true, creates the ingress resource for the application                                                       | true                          |
| jackett.ingress.annotations                | Additional field for annotations, if needed                                                                     | {}                            |
| jackett.ingress.path                       | The path where the application is exposed                                                                       | /jackett                      |
| jackett.ingress.tls.enabled                | If true, tls is enabled                                                                                         | false                         |
| jackett.ingress.tls.secretName             | Name of the secret holding certificates for the secure ingress                                                  | ""                            |
| jackett.resources                          | Limits and Requests for the container                                                                           | {}                            |
| jackett.volume                             | If set, Plex will create a PVC for it's config volume, else it will be put on general.storage.subPaths.config   | {}                            |

### Prowlarr

| Config path                                 | Meaning                                                                                                         | Default                        |
| ------------------------------------------- | --------------------------------------------------------------------------------------------------------------- | ------------------------------ |
| prowlarr.enabled                            | Flag if you want to enable Prowlarr                                                                             | true                           |
| prowlarr.container.port                     | The port in use by the container                                                                                | 9117                           |
| prowlarr.container.nodeSelector             | Node Selector for the Prowlarr pods                                                                             | {}                             |
| prowlarr.container.image                    | The image used by the container                                                                                 | docker.io/linuxserver/prowlarr |
| prowlarr.container.tag                      | The tag used by the container                                                                                   | develop                        |
| prowlarr.service.type                       | The kind of Service (ClusterIP/NodePort/LoadBalancer)                                                           | ClusterIP                      |
| prowlarr.service.port                       | The port assigned to the service                                                                                | 9117                           |
| prowlarr.service.nodePort                   | In case of service.type NodePort, the nodePort to use                                                           | ""                             |
| prowlarr.service.extraLBService             | If true, it creates an additional LoadBalancer service with '-lb' suffix (requires a cloud provider or metalLB) | false                          |
| prowlarr.service.extraLBService.annotations | Instead of using extraLBService as a bool, you can use it as a map to define annotations on the loadbalancer    | null                           |
| prowlarr.ingress.enabled                    | If true, creates the ingress resource for the application                                                       | true                           |
| prowlarr.ingress.annotations                | Additional field for annotations, if needed                                                                     | {}                             |
| prowlarr.ingress.path                       | The path where the application is exposed                                                                       | /prowlarr                      |
| prowlarr.ingress.tls.enabled                | If true, tls is enabled                                                                                         | false                          |
| prowlarr.ingress.tls.secretName             | Name of the secret holding certificates for the secure ingress                                                  | ""                             |
| prowlarr.resources                          | Limits and Requests for the container                                                                           | {}                             |
| prowlarr.volume                             | If set, Plex will create a PVC for it's config volume, else it will be put on general.storage.subPaths.config   | {}                             |

### Transmission

| Config path                                     | Meaning                                                                                                                                                          | Default                            |
| ----------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------- |
| transmission.enabled                            | Flag if you want to enable Transmission                                                                                                                          | true                               |
| transmission.container.port.utp                 | The port in use by the container                                                                                                                                 | 9091                               |
| transmission.container.nodeSelector             | Node Selector for the Transmission pods                                                                                                                          | {}                                 |
| transmission.container.port.peer                | The port in use by the container for peer connection                                                                                                             | 51413                              |
| transmission.container.image                    | The image used by the container                                                                                                                                  | docker.io/linuxserver/transmission |
| transmission.container.tag                      | The tag used by the container                                                                                                                                    | null                               |
| transmission.service.utp.type                   | The kind of Service (ClusterIP/NodePort/LoadBalancer) for Transmission itself                                                                                    | ClusterIP                          |
| transmission.service.utp.port                   | The port assigned to the service for Transmission itself                                                                                                         | 9091                               |
| transmission.service.utp.nodePort               | In case of service.type NodePort, the nodePort to use for Transmission itself                                                                                    | ""                                 |
| transmission.service.utp.extraLBService         | If true, creates an additional LoadBalancer service with '-lb' suffix (requires a cloud provider or metalLB)                                                     | false                              |
| transmission.service.peer.type                  | The kind of Service (ClusterIP/NodePort/LoadBalancer) for peer port                                                                                              | ClusterIP                          |
| transmission.service.peer.port                  | The port assigned to the service for peer port                                                                                                                   | 51413                              |
| transmission.service.peer.nodePort              | In case of service.type NodePort, the nodePort to use for peer port                                                                                              | ""                                 |
| transmission.service.peer.nodePortUDP           | In case of service.type NodePort, the nodePort to use for peer port UDP service                                                                                  | ""                                 |
| transmission.service.peer.extraLBService        | If true, creates an additional LoadBalancer service with '-lb' suffix (requires a cloud provider or metalLB)                                                     | false                              |
| transmission.service.extraLBService.annotations | Instead of using extraLBService as a bool, you can use it as a map to define annotations on the loadbalancer                                                     | null                               |
| transmission.ingress.enabled                    | If true, creates the ingress resource for the application                                                                                                        | true                               |
| transmission.ingress.annotations                | Additional field for annotations, if needed                                                                                                                      | {}                                 |
| transmission.ingress.path                       | The path where the application is exposed                                                                                                                        | /transmission                      |
| transmission.ingress.tls.enabled                | If true, tls is enabled                                                                                                                                          | false                              |
| transmission.ingress.tls.secretName             | Name of the secret holding certificates for the secure ingress                                                                                                   | ""                                 |
| transmission.config.auth.enabled                | Enables authentication for Transmission                                                                                                                          | false                              |
| transmission.config.auth.username               | Username for Transmission                                                                                                                                        | ""                                 |
| transmission.config.auth.password               | Password for Transmission                                                                                                                                        | ""                                 |
| transmission.resources                          | Limits and Requests for the container                                                                                                                            | {}                                 |
| transmission.volume                             | If set, Plex will create a PVC for it's config volume, else it will be put on general.storage.subPaths.config                                                    | {}                                 |
| transmission.vpn.enabled                        | If set, a [gluetun](https://github.com/qdm12/gluetun-wiki) sidecar will be provisioned to route the traffic through a VPN. This requires a 3rd party VPN account | {}                                 |

### Sabnzbd

| Config path                                | Meaning                                                                                                       | Default                       |
| ------------------------------------------ | ------------------------------------------------------------------------------------------------------------- | ----------------------------- |
| sabnzbd.enabled                            | Flag if you want to enable Sabnzbd                                                                            | true                          |
| sabnzbd.container.nodeSelector             | Node Selector for the Sabnzbd pods                                                                            | {}                            |
| sabnzbd.container.port.http                | The port in use by the container                                                                              | 8080                          |
| sabnzbd.container.port.https               | The port in use by the container for peer connection                                                          | 9090                          |
| sabnzbd.container.image                    | The image used by the container                                                                               | docker.io/linuxserver/sabnzbd |
| sabnzbd.container.tag                      | The tag used by the container                                                                                 | null                          |
| sabnzbd.service.http.type                  | The kind of Service (ClusterIP/NodePort/LoadBalancer) for Sabnzbd itself                                      | ClusterIP                     |
| sabnzbd.service.http.port                  | The port assigned to the service for Sabnzbd itself                                                           | 9091                          |
| sabnzbd.service.http.nodePort              | In case of service.type NodePort, the nodePort to use for Sabnzbd itself                                      | ""                            |
| sabnzbd.service.http.extraLBService        | If true, creates an additional LoadBalancer service with '-lb' suffix (requires a cloud provider or metalLB)  | false                         |
| sabnzbd.service.https.type                 | The kind of Service (ClusterIP/NodePort/LoadBalancer) for https port                                          | ClusterIP                     |
| sabnzbd.service.https.port                 | The port assigned to the service for peer port                                                                | 51413                         |
| sabnzbd.service.https.nodePort             | In case of service.type NodePort, the nodePort to use for https port                                          | ""                            |
| sabnzbd.service.https.extraLBService       | If true, creates an additional LoadBalancer service with '-lb' suffix (requires a cloud provider or metalLB)  | false                         |
| sabnzbd.service.extraLBService.annotations | Instead of using extraLBService as a bool, you can use it as a map to define annotations on the loadbalancer  | null                          |
| sabnzbd.ingress.enabled                    | If true, creates the ingress resource for the application                                                     | true                          |
| sabnzbd.ingress.annotations                | Additional field for annotations, if needed                                                                   | {}                            |
| sabnzbd.ingress.path                       | The path where the application is exposed                                                                     | /sabnzbd                      |
| sabnzbd.ingress.tls.enabled                | If true, tls is enabled                                                                                       | false                         |
| sabnzbd.ingress.tls.secretName             | Name of the secret holding certificates for the secure ingress                                                | ""                            |
| sabnzbd.resources                          | Limits and Requests for the container                                                                         | {}                            |
| sabnzbd.volume                             | If set, Plex will create a PVC for it's config volume, else it will be put on general.storage.subPaths.config | {}                            |

## Helpful use-cases

### Using a cluster-external NFS server

This assumes that you have a pre-configured NFS server set up on your network that is accessible from all nodes. If it
is not accessible by all nodes, pods will not enter ready state when scheduled on nodes that do not have NFS access.

To add an NFS volume to each resource, edit the K8SMediaServer CR to match below snipttet. You should change the `server:`
and `path:` values to match your NFS.

```yaml
general:
  storage:
    customVolume: true
    volumes:
      nfs:
        server: { SERVER-IP }
        path: /mount/path/on/nfs/server/
```

### Adding annotations to the extra load balancer

If you need an extra load balancer on any service, you can either enable it like this:

```yaml
plex:
  service:
    extraLBService: true
```

or like this, if you need to add annotations to it (for use with cloud providers to configure the load balancer for example) :

```yaml
plex:
  service:
    extraLBService:
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-additional-resource-tags:
```

### Setting up the VPN

If you have enabled the VPN for transmission you will need to fullfill the rest of parameters related to it, currently only [Mullvad](https://mullvad.net) VPN is supported.

The following shows an example of the current settings for mullvard.

```yaml
vpn:
  enabled: true
  provider: mullvad
  type: openvpn
  user: "XXXXXX"
  city: zurich
```

## About the project

This project is intended as an exercise, and absolutely for fun.
This is not intended to promote piracy.

Also feel free to contribute and extend it!

### Uninstalling the helm chart

To fully remove all the resources created you should uninstall the helm deployment.

```bash
  helm uninstall k8s-mediaserver
```

This will not delete the Custom Resources like the operator.
