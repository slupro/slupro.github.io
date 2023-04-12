# Docker network

## default bridge -- docker0

Docker在安装完成后，会自动在系统中创建一个virtual bridge docker0。默认IP是`172.17.0.1`。

![](/assets/img/Docker-network/2023-04-12-19-39-01.png)

> 默认创建的docker都以该bridge为网关，IP会分配为`172.17.0.x`

创建两个container，如果不指定network，那么它们就会使用默认的`docker0`作为bridge，他们之间可以互相访问。

```
docker run -itd --rm --name box1 busybox
docker run -itd --rm --name box2 busybox
```

![](/assets/img/Docker-network/2023-04-12-20-45-45.png)

使用 `docker inspect bridge` 查询container使用的IP地址等信息。由于这两个container使用同一个bridge，所以 172.17.0.2 和 172.17.0.3 之间可以互相访问。

![](/assets/img/Docker-network/2023-04-12-20-51-56.png)

通过bridge这种 NAT 模式，container也可以访问外部网络，但外部网络不能访问172.17.0.2/3。如果需要通过外部IP访问内部网络，需要映射端口：`docker run -itd --rm -p 80:80 --name nginxbox nginx`。

## User-defined bridge

可以通过命令行`docker network create (bridge_name)`创建新的bridge，该bridge将使用和docker0不同的网段IP。当创建container时，可以通过`--network`指定bridge。在同一个bridge内的container可以互相访问，就像在同一个NAT后一样。

下面的命令创建了新的bridge，并创建了两个新的container使用该bridge。新bridge使用了IP `172.18.0.1/16`，不同于docker0使用的 `172.17.0.1/16`。

```
docker network create newbridge
docker run -itd --rm --network newbridge --name box3 busybox
docker run -itd --rm --network newbridge --name box4 busybox
```

![](/assets/img/Docker-network/2023-04-12-21-27-44.png)

![](/assets/img/Docker-network/2023-04-12-21-17-20.png)

## Host network

如果使用 host 作为网络的话，那么container的端口会直接暴露在host主机的端口中。通过`--network host`来指定使用host网络。

![](/assets/img/Docker-network/2023-04-12-22-02-23.png)

## Mac VLAN
