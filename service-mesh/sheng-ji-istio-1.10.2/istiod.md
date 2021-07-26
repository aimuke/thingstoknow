# istiod

## Env

```bash
docker exec a25e4a30fda8 env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=istiod-cbcb9f67b-ls56l
POD_NAMESPACE=istio-system
PILOT_TRACE_SAMPLING=100
PILOT_ENABLE_PROTOCOL_SNIFFING_FOR_OUTBOUND=true
PILOT_ENABLE_PROTOCOL_SNIFFING_FOR_INBOUND=true
ISTIOD_ADDR=istiod.istio-system.svc:15012
JWT_POLICY=first-party-jwt
PILOT_CERT_PROVIDER=istiod
POD_NAME=istiod-cbcb9f67b-ls56l
SERVICE_ACCOUNT=istiod-service-account
KUBECONFIG=/var/run/secrets/remote/config
PILOT_ENABLE_ANALYSIS=false
CLUSTER_ID=Kubernetes
REVISION=default
ISTIOD_SERVICE_PORT_HTTPS_DNS=15012
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
ISTIOD_SERVICE_PORT_HTTPS_WEBHOOK=443
ISTIOD_SERVICE_PORT_HTTP_MONITORING=15014
ISTIOD_PORT_15012_TCP_PORT=15012
ISTIOD_PORT_15012_TCP_ADDR=10.96.106.222
ISTIOD_PORT_15014_TCP_PROTO=tcp
ISTIOD_PORT_15014_TCP_PORT=15014
ISTIOD_PORT_443_TCP_PROTO=tcp
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
ISTIOD_SERVICE_HOST=10.96.106.222
ISTIOD_PORT=tcp://10.96.106.222:15010
ISTIOD_PORT_15010_TCP_PROTO=tcp
ISTIOD_PORT_15010_TCP_ADDR=10.96.106.222
ISTIOD_PORT_15012_TCP_PROTO=tcp
ISTIOD_PORT_15014_TCP=tcp://10.96.106.222:15014
ISTIOD_SERVICE_PORT_GRPC_XDS=15010
ISTIOD_PORT_443_TCP_PORT=443
ISTIOD_PORT_443_TCP_ADDR=10.96.106.222
ISTIOD_PORT_15014_TCP_ADDR=10.96.106.222
KUBERNETES_SERVICE_PORT_HTTPS=443
ISTIOD_SERVICE_PORT=15010
ISTIOD_PORT_15010_TCP=tcp://10.96.106.222:15010
ISTIOD_PORT_15010_TCP_PORT=15010
ISTIOD_PORT_443_TCP=tcp://10.96.106.222:443
KUBERNETES_PORT_443_TCP_PORT=443
ISTIOD_PORT_15012_TCP=tcp://10.96.106.222:15012
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_PORT=tcp://10.96.0.1:443
DEBIAN_FRONTEND=noninteractive
HOME=/home/istio-proxy
```

## docker inspect

```bash
$ docker inspect a25e4a30fda8
[
    {
        "Id": "a25e4a30fda86ddc5236b2d7948002602fab3060a200930bd5f1678817a484d1",
        "Created": "2021-07-26T03:23:04.175832174Z",
        "Path": "/usr/local/bin/pilot-discovery",
        "Args": [
            "discovery",
            "--monitoringAddr=:15014",
            "--log_output_level=default:info",
            "--domain",
            "cluster.local",
            "--keepaliveMaxServerConnectionAge",
            "30m"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 9067,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2021-07-26T03:23:05.133000386Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:8c8ea32730d4f1e9ffdfc30d2711a0db796568a9c70a2f60a1893e9425b999d2",
        "ResolvConfPath": "/var/lib/docker/containers/ba0bd848f50aba66f518b9ea113fe9b2c7aa41624194c320022ce3ef18cb6566/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/ba0bd848f50aba66f518b9ea113fe9b2c7aa41624194c320022ce3ef18cb6566/hostname",
        "HostsPath": "/var/lib/kubelet/pods/c89f03bb-0e48-4a7e-b628-af39a8f49559/etc-hosts",
        "LogPath": "/var/lib/docker/containers/a25e4a30fda86ddc5236b2d7948002602fab3060a200930bd5f1678817a484d1/a25e4a30fda86ddc5236b2d7948002602fab3060a200930bd5f1678817a484d1-json.log",
        "Name": "/k8s_discovery_istiod-cbcb9f67b-ls56l_istio-system_c89f03bb-0e48-4a7e-b628-af39a8f49559_0",
        "RestartCount": 0,
        "Driver": "overlay",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "docker-default",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": [
                "/var/lib/kubelet/pods/c89f03bb-0e48-4a7e-b628-af39a8f49559/volumes/kubernetes.io~empty-dir/local-certs:/var/run/secrets/istio-dns",
                "/var/lib/kubelet/pods/c89f03bb-0e48-4a7e-b628-af39a8f49559/volumes/kubernetes.io~secret/cacerts:/etc/cacerts:ro",
                "/var/lib/kubelet/pods/c89f03bb-0e48-4a7e-b628-af39a8f49559/volumes/kubernetes.io~secret/istio-kubeconfig:/var/run/secrets/remote:ro",
                "/var/lib/kubelet/pods/c89f03bb-0e48-4a7e-b628-af39a8f49559/volumes/kubernetes.io~secret/istiod-service-account-token-q4gdv:/var/run/secrets/kubernetes.io/serviceaccount:ro",
                "/var/lib/kubelet/pods/c89f03bb-0e48-4a7e-b628-af39a8f49559/etc-hosts:/etc/hosts",
                "/var/lib/kubelet/pods/c89f03bb-0e48-4a7e-b628-af39a8f49559/containers/discovery/5ea09b51:/dev/termination-log"
            ],
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {
                    "max-size": "100m"
                }
            },
            "NetworkMode": "container:ba0bd848f50aba66f518b9ea113fe9b2c7aa41624194c320022ce3ef18cb6566",
            "PortBindings": null,
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": [
                "ALL"
            ],
            "Capabilities": null,
            "Dns": null,
            "DnsOptions": null,
            "DnsSearch": null,
            "ExtraHosts": null,
            "GroupAdd": [
                "1337"
            ],
            "IpcMode": "container:ba0bd848f50aba66f518b9ea113fe9b2c7aa41624194c320022ce3ef18cb6566",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 975,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": [
                "seccomp=unconfined"
            ],
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 10,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "kubepods-burstable-podc89f03bb_0e48_4a7e_b628_af39a8f49559.slice",
            "BlkioWeight": 0,
            "BlkioWeightDevice": null,
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 100000,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "KernelMemory": 0,
            "KernelMemoryTCP": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/asound",
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay/3f56ec6c2bbe7914d821864e864615a7343d78decf1800d1b5a7164a86c6cca3/root",
                "MergedDir": "/var/lib/docker/overlay/eacf8844e04f78904b23b2d5b8df5afb600598bce8a7aee5351993ddb823a12b/merged",
                "UpperDir": "/var/lib/docker/overlay/eacf8844e04f78904b23b2d5b8df5afb600598bce8a7aee5351993ddb823a12b/upper",
                "WorkDir": "/var/lib/docker/overlay/eacf8844e04f78904b23b2d5b8df5afb600598bce8a7aee5351993ddb823a12b/work"
            },
            "Name": "overlay"
        },
        "Mounts": [
            {
                "Type": "bind",
                "Source": "/var/lib/kubelet/pods/c89f03bb-0e48-4a7e-b628-af39a8f49559/volumes/kubernetes.io~empty-dir/local-certs",
                "Destination": "/var/run/secrets/istio-dns",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            },
            {
                "Type": "bind",
                "Source": "/var/lib/kubelet/pods/c89f03bb-0e48-4a7e-b628-af39a8f49559/volumes/kubernetes.io~secret/cacerts",
                "Destination": "/etc/cacerts",
                "Mode": "ro",
                "RW": false,
                "Propagation": "rprivate"
            },
            {
                "Type": "bind",
                "Source": "/var/lib/kubelet/pods/c89f03bb-0e48-4a7e-b628-af39a8f49559/volumes/kubernetes.io~secret/istio-kubeconfig",
                "Destination": "/var/run/secrets/remote",
                "Mode": "ro",
                "RW": false,
                "Propagation": "rprivate"
            },
            {
                "Type": "bind",
                "Source": "/var/lib/kubelet/pods/c89f03bb-0e48-4a7e-b628-af39a8f49559/volumes/kubernetes.io~secret/istiod-service-account-token-q4gdv",
                "Destination": "/var/run/secrets/kubernetes.io/serviceaccount",
                "Mode": "ro",
                "RW": false,
                "Propagation": "rprivate"
            },
            {
                "Type": "bind",
                "Source": "/var/lib/kubelet/pods/c89f03bb-0e48-4a7e-b628-af39a8f49559/etc-hosts",
                "Destination": "/etc/hosts",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            },
            {
                "Type": "bind",
                "Source": "/var/lib/kubelet/pods/c89f03bb-0e48-4a7e-b628-af39a8f49559/containers/discovery/5ea09b51",
                "Destination": "/dev/termination-log",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            }
        ],
        "Config": {
            "Hostname": "istiod-cbcb9f67b-ls56l",
            "Domainname": "",
            "User": "1337:1337",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "POD_NAMESPACE=istio-system",
                "PILOT_TRACE_SAMPLING=100",
                "PILOT_ENABLE_PROTOCOL_SNIFFING_FOR_OUTBOUND=true",
                "PILOT_ENABLE_PROTOCOL_SNIFFING_FOR_INBOUND=true",
                "ISTIOD_ADDR=istiod.istio-system.svc:15012",
                "JWT_POLICY=first-party-jwt",
                "PILOT_CERT_PROVIDER=istiod",
                "POD_NAME=istiod-cbcb9f67b-ls56l",
                "SERVICE_ACCOUNT=istiod-service-account",
                "KUBECONFIG=/var/run/secrets/remote/config",
                "PILOT_ENABLE_ANALYSIS=false",
                "CLUSTER_ID=Kubernetes",
                "REVISION=default",
                "ISTIOD_SERVICE_PORT_HTTPS_DNS=15012",
                "KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443",
                "KUBERNETES_PORT_443_TCP_PROTO=tcp",
                "ISTIOD_SERVICE_PORT_HTTPS_WEBHOOK=443",
                "ISTIOD_SERVICE_PORT_HTTP_MONITORING=15014",
                "ISTIOD_PORT_15012_TCP_PORT=15012",
                "ISTIOD_PORT_15012_TCP_ADDR=10.96.106.222",
                "ISTIOD_PORT_15014_TCP_PROTO=tcp",
                "ISTIOD_PORT_15014_TCP_PORT=15014",
                "ISTIOD_PORT_443_TCP_PROTO=tcp",
                "KUBERNETES_SERVICE_PORT=443",
                "KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1",
                "ISTIOD_SERVICE_HOST=10.96.106.222",
                "ISTIOD_PORT=tcp://10.96.106.222:15010",
                "ISTIOD_PORT_15010_TCP_PROTO=tcp",
                "ISTIOD_PORT_15010_TCP_ADDR=10.96.106.222",
                "ISTIOD_PORT_15012_TCP_PROTO=tcp",
                "ISTIOD_PORT_15014_TCP=tcp://10.96.106.222:15014",
                "ISTIOD_SERVICE_PORT_GRPC_XDS=15010",
                "ISTIOD_PORT_443_TCP_PORT=443",
                "ISTIOD_PORT_443_TCP_ADDR=10.96.106.222",
                "ISTIOD_PORT_15014_TCP_ADDR=10.96.106.222",
                "KUBERNETES_SERVICE_PORT_HTTPS=443",
                "ISTIOD_SERVICE_PORT=15010",
                "ISTIOD_PORT_15010_TCP=tcp://10.96.106.222:15010",
                "ISTIOD_PORT_15010_TCP_PORT=15010",
                "ISTIOD_PORT_443_TCP=tcp://10.96.106.222:443",
                "KUBERNETES_PORT_443_TCP_PORT=443",
                "ISTIOD_PORT_15012_TCP=tcp://10.96.106.222:15012",
                "KUBERNETES_SERVICE_HOST=10.96.0.1",
                "KUBERNETES_PORT=tcp://10.96.0.1:443",
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "DEBIAN_FRONTEND=noninteractive"
            ],
            "Cmd": [
                "discovery",
                "--monitoringAddr=:15014",
                "--log_output_level=default:info",
                "--domain",
                "cluster.local",
                "--keepaliveMaxServerConnectionAge",
                "30m"
            ],
            "Healthcheck": {
                "Test": [
                    "NONE"
                ]
            },
            "Image": "istio/pilot@sha256:bda5d84cf86568f6578b0c5ffdd36c6ba8ee6bbf06230e0eb452af4f1a98f7d9",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": [
                "/usr/local/bin/pilot-discovery"
            ],
            "OnBuild": null,
            "Labels": {
                "annotation.io.kubernetes.container.hash": "23a82394",
                "annotation.io.kubernetes.container.ports": "[{\"containerPort\":8080,\"protocol\":\"TCP\"},{\"containerPort\":15010,\"protocol\":\"TCP\"},{\"containerPort\":15017,\"protocol\":\"TCP\"}]",
                "annotation.io.kubernetes.container.restartCount": "0",
                "annotation.io.kubernetes.container.terminationMessagePath": "/dev/termination-log",
                "annotation.io.kubernetes.container.terminationMessagePolicy": "File",
                "annotation.io.kubernetes.pod.terminationGracePeriod": "30",
                "io.kubernetes.container.logpath": "/var/log/pods/istio-system_istiod-cbcb9f67b-ls56l_c89f03bb-0e48-4a7e-b628-af39a8f49559/discovery/0.log",
                "io.kubernetes.container.name": "discovery",
                "io.kubernetes.docker.type": "container",
                "io.kubernetes.pod.name": "istiod-cbcb9f67b-ls56l",
                "io.kubernetes.pod.namespace": "istio-system",
                "io.kubernetes.pod.uid": "c89f03bb-0e48-4a7e-b628-af39a8f49559",
                "io.kubernetes.sandbox.id": "ba0bd848f50aba66f518b9ea113fe9b2c7aa41624194c320022ce3ef18cb6566"
            }
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {},
            "SandboxKey": "",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "",
            "Gateway": "",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "",
            "IPPrefixLen": 0,
            "IPv6Gateway": "",
            "MacAddress": "",
            "Networks": {}
        }
    }
]
```



## netstat -anp

```bash
node01 $ docker exec a25e4a30fda8 netstat -anp
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.1:9876          0.0.0.0:*               LISTEN      1/pilot-discovery   
tcp        0      0 127.0.0.1:41130         127.0.0.1:15017         ESTABLISHED 1/pilot-discovery   
tcp        0      0 10.244.1.3:54900        10.96.0.1:443           ESTABLISHED 1/pilot-discovery   
tcp6       0      0 :::15010                :::*                    LISTEN      1/pilot-discovery   
tcp6       0      0 :::15012                :::*                    LISTEN      1/pilot-discovery   
tcp6       0      0 :::15014                :::*                    LISTEN      1/pilot-discovery   
tcp6       0      0 :::15017                :::*                    LISTEN      1/pilot-discovery   
tcp6       0      0 :::8080                 :::*                    LISTEN      1/pilot-discovery   
tcp6       0      0 10.244.1.3:8080         10.244.1.1:53348        TIME_WAIT   -                   
tcp6       0      0 10.244.1.3:15012        10.244.1.11:48496       ESTABLISHED 1/pilot-discovery   
tcp6       0      0 10.244.1.3:8080         10.244.1.1:53404        TIME_WAIT   -                   
tcp6       0      0 10.244.1.3:8080         10.244.1.1:53084        TIME_WAIT   -                   
tcp6       0      0 10.244.1.3:15012        10.244.1.5:34690        ESTABLISHED 1/pilot-discovery   
tcp6       0      0 10.244.1.3:15012        10.244.1.4:47102        ESTABLISHED 1/pilot-discovery   
tcp6       0      0 10.244.1.3:15012        10.244.1.7:36294        ESTABLISHED 1/pilot-discovery   
tcp6       0      0 10.244.1.3:8080         10.244.1.1:53266        TIME_WAIT   -                   
tcp6       0      0 10.244.1.3:15012        10.244.1.9:50946        ESTABLISHED 1/pilot-discovery   
tcp6       0      0 10.244.1.3:8080         10.244.1.1:53050        TIME_WAIT   -                   
tcp6       0      0 10.244.1.3:15012        10.244.1.4:47838        ESTABLISHED 1/pilot-discovery   
tcp6       0      0 10.244.1.3:15012        10.244.1.8:33734        ESTABLISHED 1/pilot-discovery   
tcp6       0      0 10.244.1.3:8080         10.244.1.1:53244        TIME_WAIT   -                   
tcp6       0      0 10.244.1.3:8080         10.244.1.1:53214        TIME_WAIT   -                   
tcp6       0      0 10.244.1.3:15012        10.244.1.9:50950        ESTABLISHED 1/pilot-discovery   
tcp6       0      0 10.244.1.3:8080         10.244.1.1:53374        TIME_WAIT   -                   
tcp6       0      0 10.244.1.3:8080         10.244.1.1:53318        TIME_WAIT   -                   
tcp6       0      0 10.244.1.3:15012        10.244.1.7:36308        ESTABLISHED 1/pilot-discovery   
tcp6       0      0 10.244.1.3:15012        10.244.1.8:33740        ESTABLISHED 1/pilot-discovery   
tcp6       0      0 10.244.1.3:8080         10.244.1.1:53480        TIME_WAIT   -                   
tcp6       0      0 127.0.0.1:15017         127.0.0.1:41130         ESTABLISHED 1/pilot-discovery   
tcp6       0      0 10.244.1.3:8080         10.244.1.1:53136        TIME_WAIT   -                   
tcp6       0      0 10.244.1.3:15012        10.244.1.10:58904       ESTABLISHED 1/pilot-discovery   
tcp6       0      0 10.244.1.3:8080         10.244.1.1:53106        TIME_WAIT   -                   
tcp6       0      0 10.244.1.3:15012        10.244.1.6:37232        ESTABLISHED 1/pilot-discovery   
tcp6       0      0 10.244.1.3:8080         10.244.1.1:53028        TIME_WAIT   -                   
tcp6       0      0 10.244.1.3:15012        10.244.1.5:34892        ESTABLISHED 1/pilot-discovery   
tcp6       0      0 10.244.1.3:8080         10.244.1.1:53456        TIME_WAIT   -                   
tcp6       0      0 10.244.1.3:15012        10.244.1.11:35226       ESTABLISHED 1/pilot-discovery   
tcp6       0      0 10.244.1.3:15012        10.244.1.10:45508       ESTABLISHED 1/pilot-discovery   
tcp6       0      0 10.244.1.3:15012        10.244.1.6:37138        TIME_WAIT   -                   
tcp6       0      0 10.244.1.3:15012        10.244.1.6:50928        ESTABLISHED 1/pilot-discovery   
tcp6       0      0 10.244.1.3:8080         10.244.1.1:53296        TIME_WAIT   -                   
Active UNIX domain sockets (servers and established)
Proto RefCnt Flags       Type       State         I-Node   PID/Program name     Path
node01 $ 
```



## ps -ef 

```bash
$ docker exec a25e4a30fda8 ps -ef 
UID        PID  PPID  C STIME TTY          TIME CMD
istio-p+     1     0  0 03:23 ?        00:00:12 /usr/local/bin/pilot-discovery discovery --monitoringAddr=:15014 --log_output_level=default:info --domain cluster.local --keepaliveMaxServerConnectionAge 30m
istio-p+    28     0  0 03:54 ?        00:00:00 ps -ef
```





