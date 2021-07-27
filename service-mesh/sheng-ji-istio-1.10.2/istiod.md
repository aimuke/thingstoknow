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
istio-proxy@istiod-6dff7cdd47-rlgk7:~$ cat ps
UID        PID  PPID  C STIME TTY          TIME CMD
istio-p+     1     0  1 03:10 ?        00:00:02 /usr/local/bin/pilot-discovery discovery --monitoringAddr=:15014 --log_output_level=default:info --domain cluster.local --keepaliveMaxServerConnectionAge 30m
istio-p+    14     0  0 03:12 pts/0    00:00:00 /bin/bash
istio-p+    26    14  0 03:13 pts/0    00:00:00 ps -ef
```



## docker logs

```bash
node01 $ cat pilot.log 
2021-07-27T03:10:59.961992Z     info    FLAG: --appNamespace=""
2021-07-27T03:10:59.962083Z     info    FLAG: --caCertFile=""
2021-07-27T03:10:59.962107Z     info    FLAG: --clusterID="Kubernetes"
2021-07-27T03:10:59.962121Z     info    FLAG: --clusterRegistriesNamespace="istio-system"
2021-07-27T03:10:59.962136Z     info    FLAG: --configDir=""
2021-07-27T03:10:59.962175Z     info    FLAG: --ctrlz_address="localhost"
2021-07-27T03:10:59.962217Z     info    FLAG: --ctrlz_port="9876"
2021-07-27T03:10:59.962225Z     info    FLAG: --domain="cluster.local"
2021-07-27T03:10:59.962228Z     info    FLAG: --grpcAddr=":15010"
2021-07-27T03:10:59.962232Z     info    FLAG: --help="false"
2021-07-27T03:10:59.962265Z     info    FLAG: --httpAddr=":8080"
2021-07-27T03:10:59.962284Z     info    FLAG: --httpsAddr=":15017"
2021-07-27T03:10:59.962300Z     info    FLAG: --keepaliveInterval="30s"
2021-07-27T03:10:59.962315Z     info    FLAG: --keepaliveMaxServerConnectionAge="30m0s"
2021-07-27T03:10:59.962326Z     info    FLAG: --keepaliveTimeout="10s"
2021-07-27T03:10:59.962359Z     info    FLAG: --kubeconfig=""
2021-07-27T03:10:59.962375Z     info    FLAG: --kubernetesApiBurst="160"
2021-07-27T03:10:59.962390Z     info    FLAG: --kubernetesApiQPS="80"
2021-07-27T03:10:59.962408Z     info    FLAG: --log_as_json="false"
2021-07-27T03:10:59.962413Z     info    FLAG: --log_caller=""
2021-07-27T03:10:59.962415Z     info    FLAG: --log_output_level="default:info"
2021-07-27T03:10:59.962418Z     info    FLAG: --log_rotate=""
2021-07-27T03:10:59.962420Z     info    FLAG: --log_rotate_max_age="30"
2021-07-27T03:10:59.962449Z     info    FLAG: --log_rotate_max_backups="1000"
2021-07-27T03:10:59.962452Z     info    FLAG: --log_rotate_max_size="104857600"
2021-07-27T03:10:59.962455Z     info    FLAG: --log_stacktrace_level="default:none"
2021-07-27T03:10:59.962461Z     info    FLAG: --log_target="[stdout]"
2021-07-27T03:10:59.962464Z     info    FLAG: --meshConfig="./etc/istio/config/mesh"
2021-07-27T03:10:59.962466Z     info    FLAG: --monitoringAddr=":15014"
2021-07-27T03:10:59.962469Z     info    FLAG: --namespace="istio-system"
2021-07-27T03:10:59.962471Z     info    FLAG: --networksConfig="/etc/istio/config/meshNetworks"
2021-07-27T03:10:59.962489Z     info    FLAG: --plugins="[ext_authz,authn,authz]"
2021-07-27T03:10:59.962495Z     info    FLAG: --profile="true"
2021-07-27T03:10:59.962500Z     info    FLAG: --registries="[Kubernetes]"
2021-07-27T03:10:59.962506Z     info    FLAG: --resync="1m0s"
2021-07-27T03:10:59.962569Z     info    FLAG: --secureGRPCAddr=":15012"
2021-07-27T03:10:59.962593Z     info    FLAG: --tls-cipher-suites="[]"
2021-07-27T03:10:59.962597Z     info    FLAG: --tlsCertFile=""
2021-07-27T03:10:59.962601Z     info    FLAG: --tlsKeyFile=""
2021-07-27T03:11:00.003393Z     info    klog    Config not found: /var/run/secrets/remote/config
2021-07-27T03:11:00.005173Z     info    initializing mesh configuration ./etc/istio/config/mesh
2021-07-27T03:11:00.106577Z     info    Loaded MeshNetworks config from Kubernetes API server.
2021-07-27T03:11:00.106704Z     info    mesh networks configuration updated to: {
    "networks": {
    }
}
2021-07-27T03:11:00.107764Z     info    Loaded MeshConfig config from Kubernetes API server.
2021-07-27T03:11:00.107993Z     info    mesh configuration updated to: (*v1alpha1.MeshConfig)(0xc000c6ad80)(proxy_listen_port:15001 connect_timeout:<seconds:10 > protocol_detection_timeout:<> ingress_class:"istio" ingress_service:"istio-ingressgateway" ingress_controller_mode:STRICT enable_tracing:true access_log_file:"/dev/stdout" default_config:<config_path:"./etc/istio/proxy" binary_path:"/usr/local/bin/envoy" service_cluster:"istio-proxy" drain_duration:<seconds:45 > parent_shutdown_duration:<seconds:60 > discovery_address:"istiod.istio-system.svc:15012" proxy_admin_port:15000 control_plane_auth_policy:MUTUAL_TLS stat_name_length:189 concurrency:<value:2 > tracing:<zipkin:<address:"zipkin.istio-system:9411" > > status_port:15020 termination_drain_duration:<seconds:5 > > outbound_traffic_policy:<mode:ALLOW_ANY > enable_auto_mtls:<value:true > trust_domain:"cluster.local" default_service_export_to:"*" default_virtual_service_export_to:"*" default_destination_rule_export_to:"*" root_namespace:"istio-system" locality_lb_setting:<enabled:<value:true > > dns_refresh_rate:<seconds:5 > thrift_config:<> enable_prometheus_merge:<value:true > )

2021-07-27T03:11:00.209291Z     info    mesh configuration: {
    "proxyListenPort": 15001,
    "connectTimeout": "10s",
    "protocolDetectionTimeout": "0s",
    "ingressClass": "istio",
    "ingressService": "istio-ingressgateway",
    "ingressControllerMode": "STRICT",
    "enableTracing": true,
    "accessLogFile": "/dev/stdout",
    "defaultConfig": {
        "configPath": "./etc/istio/proxy",
        "binaryPath": "/usr/local/bin/envoy",
        "serviceCluster": "istio-proxy",
        "drainDuration": "45s",
        "parentShutdownDuration": "60s",
        "discoveryAddress": "istiod.istio-system.svc:15012",
        "proxyAdminPort": 15000,
        "controlPlaneAuthPolicy": "MUTUAL_TLS",
        "statNameLength": 189,
        "concurrency": 2,
        "tracing": {
            "zipkin": {
                "address": "zipkin.istio-system:9411"
            }
        },
        "proxyMetadata": {
        },
        "statusPort": 15020,
        "terminationDrainDuration": "5s"
    },
    "outboundTrafficPolicy": {
        "mode": "ALLOW_ANY"
    },
    "enableAutoMtls": true,
    "trustDomain": "cluster.local",
    "trustDomainAliases": [
    ],
    "defaultServiceExportTo": [
        "*"
    ],
    "defaultVirtualServiceExportTo": [
        "*"
    ],
    "defaultDestinationRuleExportTo": [
        "*"
    ],
    "rootNamespace": "istio-system",
    "localityLbSetting": {
        "enabled": true
    },
    "dnsRefreshRate": "5s",
    "certificates": [
    ],
    "thriftConfig": {

    },
    "serviceSettings": [
    ],
    "enablePrometheusMerge": true
}
2021-07-27T03:11:00.209326Z     info    version: 1.10.3-61313778e0b785e401c696f5e92f47af069f96d0-Clean
2021-07-27T03:11:00.210192Z     info    flags: {
   "ServerOptions": {
      "HTTPAddr": ":8080",
      "HTTPSAddr": ":15017",
      "GRPCAddr": ":15010",
      "MonitoringAddr": ":15014",
      "EnableProfiling": true,
      "TLSOptions": {
         "CaCertFile": "",
         "CertFile": "",
         "KeyFile": "",
         "TLSCipherSuites": null,
         "CipherSuits": null
      },
      "SecureGRPCAddr": ":15012"
   },
   "InjectionOptions": {
      "InjectionDirectory": "./var/lib/istio/inject"
   },
   "PodName": "istiod-6dff7cdd47-rlgk7",
   "Namespace": "istio-system",
   "Revision": "default",
   "MeshConfigFile": "./etc/istio/config/mesh",
   "NetworksConfigFile": "/etc/istio/config/meshNetworks",
   "RegistryOptions": {
      "FileDir": "",
      "Registries": [
         "Kubernetes"
      ],
      "KubeOptions": {
         "SystemNamespace": "",
         "WatchedNamespaces": "",
         "ResyncPeriod": 60000000000,
         "DomainSuffix": "cluster.local",
         "ClusterID": "Kubernetes",
         "Metrics": null,
         "XDSUpdater": null,
         "NetworksWatcher": null,
         "MeshWatcher": null,
         "EndpointMode": 0,
         "KubernetesAPIQPS": 80,
         "KubernetesAPIBurst": 160,
         "SyncInterval": 0,
         "SyncTimeout": null,
         "DiscoveryNamespacesFilter": null
      },
      "ClusterRegistriesNamespace": "istio-system",
      "KubeConfig": "",
      "DistributionCacheRetention": 60000000000,
      "DistributionTrackingEnabled": true
   },
   "CtrlZOptions": {
      "Port": 9876,
      "Address": "localhost"
   },
   "Plugins": [
      "ext_authz",
      "authn",
      "authz"
   ],
   "KeepaliveOptions": {
      "Time": 30000000000,
      "Timeout": 10000000000,
      "MaxServerConnectionAge": 1800000000000,
      "MaxServerConnectionAgeGrace": 10000000000
   },
   "ShutdownDuration": 0,
   "JwtRule": ""
}
2021-07-27T03:11:00.210238Z     info    initializing mesh networks from mesh config watcher
2021-07-27T03:11:00.210245Z     info    initializing mesh handlers
2021-07-27T03:11:00.210278Z     info    creating CA and initializing public key
2021-07-27T03:11:00.210342Z     info    Use self-signed certificate as the CA certificate
2021-07-27T03:11:00.214194Z     info    pkica   Failed to get secret (error: secrets "istio-ca-secret" not found), will create one
2021-07-27T03:11:00.277223Z     info    pkica   Using self-generated public key: -----BEGIN CERTIFICATE-----
MIIC/TCCAeWgAwIBAgIRAJm9VS5W2PXcga/GjYMc++gwDQYJKoZIhvcNAQELBQAw
GDEWMBQGA1UEChMNY2x1c3Rlci5sb2NhbDAeFw0yMTA3MjcwMzExMDBaFw0zMTA3
MjUwMzExMDBaMBgxFjAUBgNVBAoTDWNsdXN0ZXIubG9jYWwwggEiMA0GCSqGSIb3
DQEBAQUAA4IBDwAwggEKAoIBAQCh0xafVwuB7ChOtBpTvS+LgFos6K0WCB/Rf970
siOxrMgcgWGvUoWo0hYV7mCa+mQQO1R7kXx7bU757nUR+9ZGvVjlVtPWtn6Z0RXF
96u0uOOw8yjJwYAF7IPDb6Q72SaHLXyqo2R+mvbNwb2vhhV5QKCF5kjjOTqKPkVj
pR0bOKmlAupO5fk8ZGQs5k5wkFBTm/Q9wgWnQDAv/wMVjE39i2NIAO/BDuDfF3Pl
XSqQ0vCOtgePaIHqn7Yls2Bj3aK+Q4TbUVtmLyCi7HgCuv876GDtndkeH+R9O48u
xMEp+S/t9jV2ZBQ2RdQK8hbfJdSaJjH1D7O9l8zLkOzCTDavAgMBAAGjQjBAMA4G
A1UdDwEB/wQEAwICBDAPBgNVHRMBAf8EBTADAQH/MB0GA1UdDgQWBBSGbqmJQDY4
4Jl1KsCbBUV1qG0YsTANBgkqhkiG9w0BAQsFAAOCAQEAk3Dv187lAbmds+z21MqJ
+8FK3YBR6t9oMtVJoKCdnrH57wSDpwreIXfylhBoMyMK0MOaboe/IRvGsbpiqsnd
UdT1s3jBYVVLIBKGu/ueGb1aWWoHIHcXMHz1NgPWvm9kDIPyl3YSF83AcYJWv8pe
xXg7QxvVGCokEwtjfWz0OWg/IY1ndsAermXDspc8oRTlcFTukmkrZJ8dHogVm+4Q
XFePhFltC5nVLZ1DSn2EGJjsUkglAVdF4CqJwy9gZnjk9V9GqgzbvboO0LcauuaJ
wsg4ROZgy/bWSkrpevY56lGp+3oC2h0tWcR7OZTTMn+7ehYKFCEsDQy5e+Quc7Rv
xw==
-----END CERTIFICATE-----

2021-07-27T03:11:00.277447Z     info    rootcertrotator Set up back off time 35m31s to start rotator.
2021-07-27T03:11:00.277613Z     info    initializing controllers
2021-07-27T03:11:00.277708Z     info    No certificates specified, skipping K8S DNS certificate controller
2021-07-27T03:11:00.277802Z     info    rootcertrotator Jitter is enabled, wait 35m31s before starting root cert rotator.
2021-07-27T03:11:00.324844Z     warn    kube    Skipping CRD networking.x-k8s.io/v1alpha1/BackendPolicy as it is not present
2021-07-27T03:11:00.325062Z     warn    kube    Skipping CRD networking.x-k8s.io/v1alpha1/GatewayClass as it is not present
2021-07-27T03:11:00.325133Z     warn    kube    Skipping CRD networking.x-k8s.io/v1alpha1/Gateway as it is not present
2021-07-27T03:11:00.325212Z     warn    kube    Skipping CRD networking.x-k8s.io/v1alpha1/HTTPRoute as it is not present
2021-07-27T03:11:00.325289Z     warn    kube    Skipping CRD networking.x-k8s.io/v1alpha1/TCPRoute as it is not present
2021-07-27T03:11:00.325351Z     warn    kube    Skipping CRD networking.x-k8s.io/v1alpha1/TLSRoute as it is not present
2021-07-27T03:11:00.327386Z     info    Adding Kubernetes registry adapter
2021-07-27T03:11:00.327432Z     info    initializing Istiod DNS certificates host: istiod.istio-system.svc, custom host: 
2021-07-27T03:11:00.696156Z     info    Generating istiod-signed cert for [istiod.istio-system.svc istiod-remote.istio-system.svc istio-pilot.istio-system.svc]:
 -----BEGIN CERTIFICATE-----
MIIDaTCCAlGgAwIBAgIQQDooYkPiVoEffkxN7IUBXDANBgkqhkiG9w0BAQsFADAY
MRYwFAYDVQQKEw1jbHVzdGVyLmxvY2FsMB4XDTIxMDcyNzAzMTEwMFoXDTMxMDcy
NTAzMTEwMFowADCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAMTU7IG8
TNsseC9S5jQaLmv4ccf2OmfWpDQH18HX0ufyzP6TDqaGL2LDhPTS7SzyhWuYtczc
rktBq4voFrrDow94vbqeqsyTH9iNsHXdmixGtoMhC5hU76s4hI6vh1qRxciea4eq
dF2IwVckf/7vtbvcgTFP3VoqOGHrXcvxX4Z/zkkTDFGolURF3euGHP9Dgqnv/edL
41H1nBSKy4I+wRyxyMnwFRiR/qklCd/zOhe3Fvtne9njoOPYyKT3Epcx8uo0VfMC
QR5BIm5uqCfEHjmebrejMkslytjS+lmezNR6EIteFPtbVPsiWBBj/yUskpjjiscd
KcF87GZ1bfh6OScCAwEAAaOBxjCBwzAOBgNVHQ8BAf8EBAMCBaAwHQYDVR0lBBYw
FAYIKwYBBQUHAwEGCCsGAQUFBwMCMAwGA1UdEwEB/wQCMAAwHwYDVR0jBBgwFoAU
hm6piUA2OOCZdSrAmwVFdahtGLEwYwYDVR0RAQH/BFkwV4IXaXN0aW9kLmlzdGlv
LXN5c3RlbS5zdmOCHmlzdGlvZC1yZW1vdGUuaXN0aW8tc3lzdGVtLnN2Y4IcaXN0
aW8tcGlsb3QuaXN0aW8tc3lzdGVtLnN2YzANBgkqhkiG9w0BAQsFAAOCAQEAiFYJ
kdTUu+MPrcTkaDvq3mj7rGqNly6cPiSzUg0n+2BLsd2fyRQsvWpFZ10R1NEiESSj
9TGFgQTRQzf8vnyyl/yX2G++vuI7Ae9PvHtKS1LotWb/0vfDkYgl3HPYXsUkmkOJ
fbcPcxhNknwDbWL10Y2OD0b1CIGgrrxtmkPN6WOVb1ul/Lg8RjuZhxzZYKPXSyey
teDjU21CReXGgYzA8Lxe+KzW9TGXK9MIYizRfQlMO2W3nttP2z8ZPDZQU+Ufpb+N
6dVHnK+P3IHyUZlxnfwDw+l3CsPp3z+vXtRjnUe/dQ3oQaUuPB7eybcXEhtDbhG5
dUWLsIxi3sPLJzTbXA==
-----END CERTIFICATE-----

2021-07-27T03:11:00.696234Z     info    No plugged-in cert at etc/cacerts/ca-key.pem; self-signed cert is used
2021-07-27T03:11:00.696519Z     info    x509 cert - Issuer: "O=cluster.local", Subject: "", SN: 403a286243e256811f7e4c4dec85015c, NotBefore: "2021-07-27T03:11:00Z", NotAfter: "2031-07-25T03:11:00Z"
2021-07-27T03:11:00.696530Z     info    Istiod certificates are reloaded
2021-07-27T03:11:00.696617Z     info    spiffe  Added 1 certs to trust domain cluster.local in peer cert verifier
2021-07-27T03:11:00.696629Z     info    initializing secure discovery service
2021-07-27T03:11:00.696641Z     info    using max conn age of 30m0s
2021-07-27T03:11:00.696720Z     info    initializing secure webhook server for istiod webhooks
2021-07-27T03:11:00.711278Z     info    initializing sidecar injector
2021-07-27T03:11:00.720674Z     info    initializing config validator
2021-07-27T03:11:00.720828Z     info    initializing Istiod admin server
2021-07-27T03:11:00.721025Z     info    initializing registry event handlers
2021-07-27T03:11:00.721131Z     info    starting discovery service
2021-07-27T03:11:00.721189Z     info    using max conn age of 30m0s
2021-07-27T03:11:00.724291Z     info    Starting Istiod Server with primary cluster Kubernetes
2021-07-27T03:11:00.724434Z     info    ControlZ available at 127.0.0.1:9876
2021-07-27T03:11:00.724419Z     info    kube    Initializing Kubernetes service registry "Kubernetes"
2021-07-27T03:11:00.724602Z     info    klog    attempting to acquire leader lease istio-system/istio-leader...
2021-07-27T03:11:00.725042Z     info    kube    Starting Pilot K8S CRD controller
2021-07-27T03:11:00.725088Z     info    kube    controller "networking.istio.io/v1alpha3/Sidecar" is syncing...
2021-07-27T03:11:00.725403Z     info    Setting up event handlers
2021-07-27T03:11:00.725526Z     info    Starting Secrets controller
2021-07-27T03:11:00.725807Z     info    Starting ADS server
2021-07-27T03:11:00.725901Z     info    initializing Kubernetes credential reader for cluster Kubernetes
2021-07-27T03:11:00.725980Z     info    Setting up event handlers
2021-07-27T03:11:00.726101Z     info    Starting IstioD CA
2021-07-27T03:11:00.726151Z     info    JWT policy is first-party-jwt
2021-07-27T03:11:00.726382Z     info    Istiod CA has started
2021-07-27T03:11:00.726615Z     info    Starting Secrets controller
2021-07-27T03:11:00.726754Z     info    klog    attempting to acquire leader lease istio-system/istio-validation-controller-election...
2021-07-27T03:11:00.732749Z     info    New webhook config added, patching MutatingWebhookConfiguration for istio-sidecar-injector
2021-07-27T03:11:00.748570Z     info    klog    successfully acquired lease istio-system/istio-leader
2021-07-27T03:11:00.767135Z     info    klog    successfully acquired lease istio-system/istio-validation-controller-election
2021-07-27T03:11:00.767568Z     info    Starting validation controller
2021-07-27T03:11:00.830503Z     info    kube    controller "networking.istio.io/v1alpha3/DestinationRule" is syncing...
2021-07-27T03:11:00.830839Z     info    ads     Full push, new service kube-system/kube-dns.kube-system.svc.cluster.local
2021-07-27T03:11:00.831026Z     info    ads     Full push, new service default/kubernetes.default.svc.cluster.local
2021-07-27T03:11:00.831412Z     info    ads     Incremental push, service istiod.istio-system.svc.cluster.local has no endpoints
2021-07-27T03:11:00.831491Z     info    kube    Handle EDS endpoint: skip collecting workload entry endpoints, service cloud-controller-manager/kube-system has not been populated
2021-07-27T03:11:00.831521Z     info    ads     Incremental push, service cloud-controller-manager.kube-system.svc.cluster.local has no endpoints
2021-07-27T03:11:00.831621Z     info    kube    Handle EDS endpoint: skip collecting workload entry endpoints, service kube-controller-manager/kube-system has not been populated
2021-07-27T03:11:00.831684Z     info    ads     Incremental push, service kube-controller-manager.kube-system.svc.cluster.local has no endpoints
2021-07-27T03:11:00.831804Z     info    kube    Handle EDS endpoint: skip collecting workload entry endpoints, service kube-scheduler/kube-system has not been populated
2021-07-27T03:11:00.831874Z     info    ads     Incremental push, service kube-scheduler.kube-system.svc.cluster.local has no endpoints
2021-07-27T03:11:00.832276Z     info    kube    Handle EDS endpoint: skip collecting workload entry endpoints, service cloud-controller-manager/kube-system has not been populated
2021-07-27T03:11:00.832354Z     info    ads     Incremental push, service cloud-controller-manager.kube-system.svc.cluster.local has no endpoints
2021-07-27T03:11:00.832500Z     info    kube    Handle EDS endpoint: skip collecting workload entry endpoints, service kube-controller-manager/kube-system has not been populated
2021-07-27T03:11:00.832580Z     info    ads     Incremental push, service kube-controller-manager.kube-system.svc.cluster.local has no endpoints
2021-07-27T03:11:00.832656Z     info    kube    Handle EDS endpoint: skip collecting workload entry endpoints, service kube-scheduler/kube-system has not been populated
2021-07-27T03:11:00.832754Z     info    ads     Incremental push, service kube-scheduler.kube-system.svc.cluster.local has no endpoints
2021-07-27T03:11:00.832974Z     info    ads     Incremental push, service istiod.istio-system.svc.cluster.local has no endpoints
2021-07-27T03:11:00.833057Z     info    kube    Handle EDS endpoint: skip collecting workload entry endpoints, service cloud-controller-manager/kube-system has not been populated
2021-07-27T03:11:00.833129Z     info    ads     Incremental push, service cloud-controller-manager.kube-system.svc.cluster.local has no endpoints
2021-07-27T03:11:00.833329Z     info    kube    Handle EDS endpoint: skip collecting workload entry endpoints, service kube-controller-manager/kube-system has not been populated
2021-07-27T03:11:00.833400Z     info    ads     Incremental push, service kube-controller-manager.kube-system.svc.cluster.local has no endpoints
2021-07-27T03:11:00.833472Z     info    kube    Handle EDS endpoint: skip collecting workload entry endpoints, service kube-scheduler/kube-system has not been populated
2021-07-27T03:11:00.833549Z     info    ads     Incremental push, service kube-scheduler.kube-system.svc.cluster.local has no endpoints
2021-07-27T03:11:00.833671Z     info    ads     Incremental push, service istiod.istio-system.svc.cluster.local has no endpoints
2021-07-27T03:11:00.928882Z     info    kube    controller "networking.istio.io/v1alpha3/Sidecar" is syncing...
2021-07-27T03:11:00.934269Z     info    ads     Push debounce stable[1] 67: 100.474545ms since last change, 169.04487ms since last push, full=true
2021-07-27T03:11:00.939866Z     info    ads     XDS: Pushing:2021-07-27T03:11:00Z/1 Services:3 ConnectedEndpoints:0  Version:2021-07-27T03:11:00Z/1
2021-07-27T03:11:01.025969Z     info    kube    controller "networking.istio.io/v1alpha3/WorkloadGroup" is syncing...
2021-07-27T03:11:01.125747Z     info    kube    controller "networking.istio.io/v1alpha3/Gateway" is syncing...
2021-07-27T03:11:01.225558Z     info    kube    controller "networking.istio.io/v1alpha3/Sidecar" is syncing...
2021-07-27T03:11:01.250855Z     info    Starting ingress controller
2021-07-27T03:11:01.250990Z     warn    Missing ingress, skip status updates
2021-07-27T03:11:01.269363Z     info    validationController    Reconcile(enter): add event (admissionregistration.k8s.io/v1, Kind=ValidatingWebhookConfiguration) istiod-istio-system
2021-07-27T03:11:01.293713Z     info    ads     Push debounce stable[2] 1: 101.109226ms since last change, 101.109033ms since last push, full=false
2021-07-27T03:11:01.293961Z     info    ads     XDS: Incremental Pushing:2021-07-27T03:11:00Z/1 ConnectedEndpoints:0 Version:2021-07-27T03:11:00Z/1
2021-07-27T03:11:01.326063Z     info    kube    Pilot K8S CRD controller synced 601.00649ms
2021-07-27T03:11:01.332261Z     info    kube    joining leader-election for istio-namespace-controller-election in istio-system on cluster Kubernetes
2021-07-27T03:11:01.332409Z     info    klog    attempting to acquire leader lease istio-system/istio-namespace-controller-election...
2021-07-27T03:11:01.342149Z     info    klog    successfully acquired lease istio-system/istio-namespace-controller-election
2021-07-27T03:11:01.342627Z     info    kube    starting namespace controller for cluster Kubernetes
2021-07-27T03:11:01.428025Z     info    ads     Push debounce stable[3] 8: 100.322546ms since last change, 101.35054ms since last push, full=true
2021-07-27T03:11:01.434296Z     info    ads     XDS: Pushing:2021-07-27T03:11:01Z/2 Services:3 ConnectedEndpoints:0  Version:2021-07-27T03:11:01Z/2
2021-07-27T03:11:01.443914Z     info    kube    Namespace controller started
2021-07-27T03:11:01.532654Z     info    ads     All caches have been synced up in 1.571909921s, marking server ready
2021-07-27T03:11:01.533107Z     info    starting secure gRPC discovery service at [::]:15012
2021-07-27T03:11:01.533231Z     info    starting HTTP service at [::]:8080
2021-07-27T03:11:01.533435Z     info    starting gRPC discovery service at [::]:15010
2021-07-27T03:11:01.533579Z     info    starting webhook service at [::]:15017
2021-07-27T03:11:02.274388Z     info    ads     Full push, new service istio-system/istiod.istio-system.svc.cluster.local
2021-07-27T03:11:02.299789Z     info    validationController    Not ready to switch validation to fail-closed: dummy invalid config not rejected
2021-07-27T03:11:02.321931Z     info    validationController    Successfully updated validatingwebhookconfiguration istiod-istio-system (failurePolicy=Ignore,resourceVersion=913)
2021-07-27T03:11:02.322241Z     info    validationController    Reconcile(enter): CABundle changed
2021-07-27T03:11:02.375478Z     info    ads     Push debounce stable[4] 1: 101.014547ms since last change, 101.014345ms since last push, full=true
2021-07-27T03:11:02.376187Z     info    ads     XDS: Pushing:2021-07-27T03:11:02Z/3 Services:3 ConnectedEndpoints:0  Version:2021-07-27T03:11:02Z/3
2021-07-27T03:11:02.779611Z     info    ads     Push debounce stable[5] 2: 100.772595ms since last change, 188.474036ms since last push, full=false
2021-07-27T03:11:02.779736Z     info    ads     XDS: Incremental Pushing:2021-07-27T03:11:02Z/3 ConnectedEndpoints:0 Version:2021-07-27T03:11:02Z/3
2021-07-27T03:11:03.104221Z     info    ads     Incremental push, service istio-egressgateway.istio-system.svc.cluster.local has no endpoints
2021-07-27T03:11:03.129339Z     info    ads     Incremental push, service istio-ingressgateway.istio-system.svc.cluster.local has no endpoints
2021-07-27T03:11:03.262901Z     info    ads     Push debounce stable[6] 5: 100.365255ms since last change, 175.621921ms since last push, full=true
2021-07-27T03:11:03.263631Z     info    ads     XDS: Pushing:2021-07-27T03:11:03Z/4 Services:5 ConnectedEndpoints:0  Version:2021-07-27T03:11:03Z/4
2021-07-27T03:11:03.369265Z     info    validationServer        configuration is invalid: gateway must have at least one server
2021-07-27T03:11:03.375685Z     info    validationController    Endpoint successfully rejected invalid config. Switching to fail-close.
2021-07-27T03:11:03.384503Z     info    validationController    Successfully updated validatingwebhookconfiguration istiod-istio-system (failurePolicy=Fail,resourceVersion=975)
2021-07-27T03:11:03.384539Z     info    validationController    Reconcile(enter): update event (admissionregistration.k8s.io/v1, Kind=ValidatingWebhookConfiguration) istiod-istio-system
2021-07-27T03:11:03.385095Z     info    validationController    validatingwebhookconfiguration istiod-istio-system (failurePolicy=Fail, resourceVersion=975) is up-to-date. No change required.
2021-07-27T03:11:03.385114Z     info    validationController    Reconcile(enter): retry dry-run creation of invalid config
2021-07-27T03:11:03.385478Z     info    validationController    validatingwebhookconfiguration istiod-istio-system (failurePolicy=Fail, resourceVersion=975) is up-to-date. No change required.
2021-07-27T03:11:03.385496Z     info    validationController    Reconcile(enter): update event (admissionregistration.k8s.io/v1, Kind=ValidatingWebhookConfiguration) istiod-istio-system
2021-07-27T03:11:03.385759Z     info    validationController    validatingwebhookconfiguration istiod-istio-system (failurePolicy=Fail, resourceVersion=975) is up-to-date. No change required.
2021-07-27T03:11:12.124543Z     info    ads     ADS: new connection for node:router~10.244.1.5~istio-ingressgateway-6ffdc56559-gf97t.istio-system~istio-system.svc.cluster.local-1
2021-07-27T03:11:12.125431Z     info    ads     ADS: new connection for node:router~10.244.1.6~istio-egressgateway-7f8f879549-hxfls.istio-system~istio-system.svc.cluster.local-2
2021-07-27T03:11:12.130977Z     info    ads     CDS: PUSH request for node:istio-ingressgateway-6ffdc56559-gf97t.istio-system resources:15 size:14.3kB
2021-07-27T03:11:12.132769Z     info    ads     CDS: PUSH request for node:istio-egressgateway-7f8f879549-hxfls.istio-system resources:15 size:14.3kB
2021-07-27T03:11:12.157835Z     info    ads     EDS: PUSH request for node:istio-ingressgateway-6ffdc56559-gf97t.istio-system resources:14 size:1.7kB empty:7 cached:0/14
2021-07-27T03:11:12.158044Z     info    ads     EDS: PUSH request for node:istio-egressgateway-7f8f879549-hxfls.istio-system resources:14 size:1.7kB empty:0 cached:14/14
2021-07-27T03:11:12.234546Z     info    ads     LDS: PUSH request for node:istio-ingressgateway-6ffdc56559-gf97t.istio-system resources:0 size:0B
2021-07-27T03:11:12.265919Z     info    ads     LDS: PUSH request for node:istio-egressgateway-7f8f879549-hxfls.istio-system resources:0 size:0B
2021-07-27T03:11:12.292204Z     info    ads     CDS: PUSH for node:istio-egressgateway-7f8f879549-hxfls.istio-system resources:15 size:14.3kB
2021-07-27T03:11:12.292304Z     info    ads     EDS: PUSH for node:istio-egressgateway-7f8f879549-hxfls.istio-system resources:14 size:1.7kB empty:0 cached:14/14
2021-07-27T03:11:12.292420Z     info    ads     LDS: PUSH for node:istio-egressgateway-7f8f879549-hxfls.istio-system resources:0 size:0B
2021-07-27T03:11:12.295342Z     info    ads     Incremental push, service istio-egressgateway.istio-system.svc.cluster.local has no endpoints
2021-07-27T03:11:12.306977Z     info    ads     CDS: PUSH for node:istio-ingressgateway-6ffdc56559-gf97t.istio-system resources:15 size:14.3kB
2021-07-27T03:11:12.307103Z     info    ads     EDS: PUSH for node:istio-ingressgateway-6ffdc56559-gf97t.istio-system resources:14 size:1.7kB empty:0 cached:14/14
2021-07-27T03:11:12.307212Z     info    ads     LDS: PUSH for node:istio-ingressgateway-6ffdc56559-gf97t.istio-system resources:0 size:0B
2021-07-27T03:11:12.311854Z     info    ads     Incremental push, service istio-ingressgateway.istio-system.svc.cluster.local has no endpoints
2021-07-27T03:11:12.412443Z     info    ads     Push debounce stable[7] 2: 100.535301ms since last change, 117.039588ms since last push, full=false
2021-07-27T03:11:12.412620Z     info    ads     XDS: Incremental Pushing:2021-07-27T03:11:03Z/4 ConnectedEndpoints:2 Version:2021-07-27T03:11:03Z/4
2021-07-27T03:11:13.328170Z     info    ads     Full push, new service istio-system/istio-egressgateway.istio-system.svc.cluster.local
2021-07-27T03:11:13.372822Z     info    ads     Full push, new service istio-system/istio-ingressgateway.istio-system.svc.cluster.local
2021-07-27T03:11:13.473849Z     info    ads     Push debounce stable[8] 2: 100.611911ms since last change, 145.599157ms since last push, full=true
2021-07-27T03:11:13.474572Z     info    ads     XDS: Pushing:2021-07-27T03:11:13Z/5 Services:5 ConnectedEndpoints:2  Version:2021-07-27T03:11:13Z/5
2021-07-27T03:11:13.475461Z     info    ads     CDS: PUSH for node:istio-egressgateway-7f8f879549-hxfls.istio-system resources:15 size:14.9kB
2021-07-27T03:11:13.475735Z     info    ads     EDS: PUSH for node:istio-egressgateway-7f8f879549-hxfls.istio-system resources:14 size:2.6kB empty:0 cached:7/14
2021-07-27T03:11:13.475840Z     info    ads     CDS: PUSH for node:istio-ingressgateway-6ffdc56559-gf97t.istio-system resources:15 size:14.9kB
2021-07-27T03:11:13.475948Z     info    ads     LDS: PUSH for node:istio-egressgateway-7f8f879549-hxfls.istio-system resources:0 size:0B
2021-07-27T03:11:13.476003Z     info    ads     EDS: PUSH for node:istio-ingressgateway-6ffdc56559-gf97t.istio-system resources:14 size:2.6kB empty:0 cached:14/14
2021-07-27T03:11:13.476228Z     info    ads     LDS: PUSH for node:istio-ingressgateway-6ffdc56559-gf97t.istio-system resources:0 size:0B
2021-07-27T03:11:17.117477Z     info    ads     Incremental push, service details.default.svc.cluster.local has no endpoints
2021-07-27T03:11:17.166387Z     info    Sidecar injection request for default/details-v1-66b6955995-***** (actual name not yet known)
2021-07-27T03:11:17.177030Z     info    ads     Incremental push, service ratings.default.svc.cluster.local has no endpoints
2021-07-27T03:11:17.268132Z     info    ads     Incremental push, service reviews.default.svc.cluster.local has no endpoints
2021-07-27T03:11:17.343945Z     info    Sidecar injection request for default/reviews-v1-6549ddccc5-***** (actual name not yet known)
2021-07-27T03:11:17.362852Z     info    Sidecar injection request for default/reviews-v2-76c4865449-***** (actual name not yet known)
2021-07-27T03:11:17.407903Z     info    Sidecar injection request for default/reviews-v3-6b554c875-***** (actual name not yet known)
2021-07-27T03:11:17.423390Z     info    ads     Push debounce stable[9] 9: 100.216149ms since last change, 317.984751ms since last push, full=true
2021-07-27T03:11:17.424359Z     info    ads     XDS: Pushing:2021-07-27T03:11:17Z/6 Services:8 ConnectedEndpoints:2  Version:2021-07-27T03:11:17Z/6
2021-07-27T03:11:17.427363Z     info    ads     CDS: PUSH for node:istio-egressgateway-7f8f879549-hxfls.istio-system resources:18 size:17.5kB
2021-07-27T03:11:17.427576Z     info    ads     EDS: PUSH for node:istio-egressgateway-7f8f879549-hxfls.istio-system resources:14 size:2.6kB empty:0 cached:14/14
2021-07-27T03:11:17.427743Z     info    ads     LDS: PUSH for node:istio-egressgateway-7f8f879549-hxfls.istio-system resources:0 size:0B
2021-07-27T03:11:17.428798Z     info    ads     CDS: PUSH for node:istio-ingressgateway-6ffdc56559-gf97t.istio-system resources:18 size:17.5kB
2021-07-27T03:11:17.429325Z     info    ads     EDS: PUSH for node:istio-ingressgateway-6ffdc56559-gf97t.istio-system resources:14 size:2.6kB empty:0 cached:14/14
2021-07-27T03:11:17.429629Z     info    ads     LDS: PUSH for node:istio-ingressgateway-6ffdc56559-gf97t.istio-system resources:0 size:0B
2021-07-27T03:11:17.443050Z     info    ads     EDS: PUSH request for node:istio-egressgateway-7f8f879549-hxfls.istio-system resources:17 size:2.8kB empty:3 cached:14/17
2021-07-27T03:11:17.465664Z     info    ads     Incremental push, service productpage.default.svc.cluster.local has no endpoints
2021-07-27T03:11:17.479499Z     info    ads     EDS: PUSH request for node:istio-ingressgateway-6ffdc56559-gf97t.istio-system resources:17 size:2.8kB empty:0 cached:17/17
2021-07-27T03:11:17.535626Z     info    Sidecar injection request for default/productpage-v1-5d9b4c9849-***** (actual name not yet known)
2021-07-27T03:11:17.601697Z     info    ads     Push debounce stable[10] 3: 102.942299ms since last change, 170.334457ms since last push, full=true
2021-07-27T03:11:17.602216Z     info    ads     XDS: Pushing:2021-07-27T03:11:17Z/7 Services:9 ConnectedEndpoints:2  Version:2021-07-27T03:11:17Z/7
2021-07-27T03:11:17.603091Z     info    ads     CDS: PUSH for node:istio-ingressgateway-6ffdc56559-gf97t.istio-system resources:19 size:18.5kB
2021-07-27T03:11:17.603144Z     info    ads     EDS: PUSH for node:istio-ingressgateway-6ffdc56559-gf97t.istio-system resources:17 size:2.8kB empty:0 cached:17/17
2021-07-27T03:11:17.603205Z     info    ads     LDS: PUSH for node:istio-ingressgateway-6ffdc56559-gf97t.istio-system resources:0 size:0B
2021-07-27T03:11:17.603276Z     info    ads     CDS: PUSH for node:istio-egressgateway-7f8f879549-hxfls.istio-system resources:19 size:18.5kB
2021-07-27T03:11:17.603313Z     info    ads     EDS: PUSH for node:istio-egressgateway-7f8f879549-hxfls.istio-system resources:17 size:2.8kB empty:0 cached:17/17
2021-07-27T03:11:17.603355Z     info    ads     LDS: PUSH for node:istio-egressgateway-7f8f879549-hxfls.istio-system resources:0 size:0B
2021-07-27T03:11:17.609394Z     info    ads     EDS: PUSH request for node:istio-ingressgateway-6ffdc56559-gf97t.istio-system resources:18 size:2.8kB empty:1 cached:17/18
2021-07-27T03:11:17.614076Z     info    ads     EDS: PUSH request for node:istio-egressgateway-7f8f879549-hxfls.istio-system resources:18 size:2.8kB empty:0 cached:18/18
2021-07-27T03:11:18.209058Z     info    Sidecar injection request for default/ratings-v1-fd78f799f-***** (actual name not yet known)
2021-07-27T03:11:19.902549Z     info    ads     Incremental push, service reviews.default.svc.cluster.local has no endpoints
2021-07-27T03:11:20.002721Z     info    ads     Push debounce stable[11] 1: 100.111596ms since last change, 100.111449ms since last push, full=false
2021-07-27T03:11:20.002802Z     info    ads     XDS: Incremental Pushing:2021-07-27T03:11:17Z/7 ConnectedEndpoints:2 Version:2021-07-27T03:11:17Z/7
2021-07-27T03:11:21.537664Z     info    ads     Incremental push, service productpage.default.svc.cluster.local has no endpoints
2021-07-27T03:11:21.592601Z     info    ads     Incremental push, service details.default.svc.cluster.local has no endpoints
2021-07-27T03:11:21.694175Z     info    ads     Push debounce stable[12] 2: 101.489923ms since last change, 156.429511ms since last push, full=false
2021-07-27T03:11:21.694503Z     info    ads     XDS: Incremental Pushing:2021-07-27T03:11:17Z/7 ConnectedEndpoints:2 Version:2021-07-27T03:11:17Z/7
2021-07-27T03:11:21.899379Z     info    ads     Incremental push, service ratings.default.svc.cluster.local has no endpoints
2021-07-27T03:11:22.000457Z     info    ads     Push debounce stable[13] 1: 100.756419ms since last change, 100.756176ms since last push, full=false
2021-07-27T03:11:22.000729Z     info    ads     XDS: Incremental Pushing:2021-07-27T03:11:17Z/7 ConnectedEndpoints:2 Version:2021-07-27T03:11:17Z/7
2021-07-27T03:11:51.865944Z     info    ads     ADS: new connection for node:sidecar~10.244.1.10~reviews-v1-6549ddccc5-6xwlr.default~default.svc.cluster.local-3
2021-07-27T03:11:51.866859Z     info    ads     CDS: PUSH request for node:reviews-v1-6549ddccc5-6xwlr.default resources:22 size:19.6kB
2021-07-27T03:11:51.940965Z     info    ads     EDS: PUSH request for node:reviews-v1-6549ddccc5-6xwlr.default resources:18 size:2.8kB empty:0 cached:18/18
2021-07-27T03:11:51.994968Z     info    ads     LDS: PUSH request for node:reviews-v1-6549ddccc5-6xwlr.default resources:16 size:74.3kB
2021-07-27T03:11:52.092039Z     info    ads     RDS: PUSH request for node:reviews-v1-6549ddccc5-6xwlr.default resources:6 size:6.1kB
2021-07-27T03:11:53.036359Z     info    ads     Full push, new service default/reviews.default.svc.cluster.local
2021-07-27T03:11:53.136889Z     info    ads     Push debounce stable[14] 1: 100.443762ms since last change, 100.443444ms since last push, full=true
2021-07-27T03:11:53.137940Z     info    ads     XDS: Pushing:2021-07-27T03:11:53Z/8 Services:9 ConnectedEndpoints:3  Version:2021-07-27T03:11:53Z/8
2021-07-27T03:11:53.142721Z     info    ads     CDS: PUSH for node:reviews-v1-6549ddccc5-6xwlr.default resources:22 size:19.7kB
2021-07-27T03:11:53.143049Z     info    ads     EDS: PUSH for node:reviews-v1-6549ddccc5-6xwlr.default resources:18 size:3.0kB empty:0 cached:17/18
2021-07-27T03:11:53.144379Z     info    ads     CDS: PUSH for node:istio-ingressgateway-6ffdc56559-gf97t.istio-system resources:19 size:18.5kB
2021-07-27T03:11:53.144447Z     info    ads     EDS: PUSH for node:istio-ingressgateway-6ffdc56559-gf97t.istio-system resources:18 size:3.0kB empty:0 cached:18/18
2021-07-27T03:11:53.145148Z     info    ads     LDS: PUSH for node:istio-ingressgateway-6ffdc56559-gf97t.istio-system resources:0 size:0B
2021-07-27T03:11:53.149863Z     info    ads     CDS: PUSH for node:istio-egressgateway-7f8f879549-hxfls.istio-system resources:19 size:18.5kB
2021-07-27T03:11:53.149940Z     info    ads     EDS: PUSH for node:istio-egressgateway-7f8f879549-hxfls.istio-system resources:18 size:3.0kB empty:0 cached:18/18
2021-07-27T03:11:53.150010Z     info    ads     LDS: PUSH for node:istio-egressgateway-7f8f879549-hxfls.istio-system resources:0 size:0B
2021-07-27T03:11:53.158212Z     info    ads     LDS: PUSH for node:reviews-v1-6549ddccc5-6xwlr.default resources:16 size:74.3kB
2021-07-27T03:11:53.166806Z     info    ads     RDS: PUSH for node:reviews-v1-6549ddccc5-6xwlr.default resources:6 size:6.1kB
2021-07-27T03:12:11.056488Z     info    ads     ADS: new connection for node:sidecar~10.244.1.11~productpage-v1-5d9b4c9849-z4sx6.default~default.svc.cluster.local-4
2021-07-27T03:12:11.057993Z     info    ads     CDS: PUSH request for node:productpage-v1-5d9b4c9849-z4sx6.default resources:22 size:19.7kB
2021-07-27T03:12:11.145031Z     info    ads     EDS: PUSH request for node:productpage-v1-5d9b4c9849-z4sx6.default resources:18 size:3.0kB empty:0 cached:18/18
2021-07-27T03:12:11.165717Z     info    ads     ADS: new connection for node:sidecar~10.244.1.8~reviews-v3-6b554c875-7wtc7.default~default.svc.cluster.local-5
2021-07-27T03:12:11.166821Z     info    ads     CDS: PUSH request for node:reviews-v3-6b554c875-7wtc7.default resources:22 size:19.7kB
2021-07-27T03:12:11.238676Z     info    ads     EDS: PUSH request for node:reviews-v3-6b554c875-7wtc7.default resources:18 size:3.0kB empty:0 cached:18/18
2021-07-27T03:12:11.346278Z     info    ads     LDS: PUSH request for node:productpage-v1-5d9b4c9849-z4sx6.default resources:16 size:74.3kB
2021-07-27T03:12:11.501795Z     info    ads     RDS: PUSH request for node:productpage-v1-5d9b4c9849-z4sx6.default resources:6 size:6.1kB
2021-07-27T03:12:11.769987Z     info    ads     LDS: PUSH request for node:reviews-v3-6b554c875-7wtc7.default resources:16 size:74.3kB
2021-07-27T03:12:11.908781Z     info    ads     RDS: PUSH request for node:reviews-v3-6b554c875-7wtc7.default resources:6 size:6.1kB
2021-07-27T03:12:11.986829Z     info    ads     ADS: new connection for node:sidecar~10.244.1.7~reviews-v2-76c4865449-mq9ml.default~default.svc.cluster.local-6
2021-07-27T03:12:11.987806Z     info    ads     CDS: PUSH request for node:reviews-v2-76c4865449-mq9ml.default resources:22 size:19.7kB
2021-07-27T03:12:12.013042Z     info    ads     EDS: PUSH request for node:reviews-v2-76c4865449-mq9ml.default resources:18 size:3.0kB empty:0 cached:18/18
2021-07-27T03:12:12.332039Z     info    ads     LDS: PUSH request for node:reviews-v2-76c4865449-mq9ml.default resources:16 size:74.3kB
2021-07-27T03:12:12.420866Z     info    ads     RDS: PUSH request for node:reviews-v2-76c4865449-mq9ml.default resources:6 size:6.1kB
2021-07-27T03:12:12.762272Z     info    ads     Push debounce stable[15] 1: 102.430591ms since last change, 102.430315ms since last push, full=false
2021-07-27T03:12:12.762342Z     info    ads     XDS: Incremental Pushing:2021-07-27T03:11:53Z/8 ConnectedEndpoints:6 Version:2021-07-27T03:11:53Z/8
2021-07-27T03:12:13.654779Z     info    ads     Full push, new service default/productpage.default.svc.cluster.local
2021-07-27T03:12:13.754988Z     info    ads     Push debounce stable[16] 2: 100.109474ms since last change, 200.554867ms since last push, full=true
2021-07-27T03:12:13.755540Z     info    ads     XDS: Pushing:2021-07-27T03:12:13Z/9 Services:9 ConnectedEndpoints:6  Version:2021-07-27T03:12:13Z/9
2021-07-27T03:12:13.756441Z     info    ads     CDS: PUSH for node:reviews-v2-76c4865449-mq9ml.default resources:22 size:19.7kB
2021-07-27T03:12:13.756566Z     info    ads     EDS: PUSH for node:reviews-v2-76c4865449-mq9ml.default resources:18 size:3.5kB empty:0 cached:16/18
2021-07-27T03:12:13.760209Z     info    ads     LDS: PUSH for node:reviews-v2-76c4865449-mq9ml.default resources:16 size:74.3kB
2021-07-27T03:12:13.762383Z     info    ads     CDS: PUSH for node:productpage-v1-5d9b4c9849-z4sx6.default resources:22 size:19.8kB
2021-07-27T03:12:13.762441Z     info    ads     EDS: PUSH for node:productpage-v1-5d9b4c9849-z4sx6.default resources:18 size:3.5kB empty:0 cached:18/18
2021-07-27T03:12:13.769950Z     info    ads     CDS: PUSH for node:reviews-v3-6b554c875-7wtc7.default resources:22 size:19.7kB
2021-07-27T03:12:13.770028Z     info    ads     EDS: PUSH for node:reviews-v3-6b554c875-7wtc7.default resources:18 size:3.5kB empty:0 cached:18/18
2021-07-27T03:12:13.772580Z     info    ads     LDS: PUSH for node:productpage-v1-5d9b4c9849-z4sx6.default resources:16 size:74.3kB
2021-07-27T03:12:13.773877Z     info    ads     LDS: PUSH for node:reviews-v3-6b554c875-7wtc7.default resources:16 size:74.3kB
2021-07-27T03:12:13.774379Z     info    ads     RDS: PUSH for node:reviews-v2-76c4865449-mq9ml.default resources:6 size:6.1kB
2021-07-27T03:12:13.774424Z     info    ads     CDS: PUSH for node:istio-ingressgateway-6ffdc56559-gf97t.istio-system resources:19 size:18.6kB
2021-07-27T03:12:13.774481Z     info    ads     EDS: PUSH for node:istio-ingressgateway-6ffdc56559-gf97t.istio-system resources:18 size:3.5kB empty:0 cached:18/18
2021-07-27T03:12:13.774527Z     info    ads     LDS: PUSH for node:istio-ingressgateway-6ffdc56559-gf97t.istio-system resources:0 size:0B
2021-07-27T03:12:13.774758Z     info    ads     RDS: PUSH for node:reviews-v3-6b554c875-7wtc7.default resources:6 size:6.1kB
2021-07-27T03:12:13.789670Z     info    ads     CDS: PUSH for node:istio-egressgateway-7f8f879549-hxfls.istio-system resources:19 size:18.6kB
2021-07-27T03:12:13.789758Z     info    ads     EDS: PUSH for node:istio-egressgateway-7f8f879549-hxfls.istio-system resources:18 size:3.5kB empty:0 cached:18/18
2021-07-27T03:12:13.789853Z     info    ads     LDS: PUSH for node:istio-egressgateway-7f8f879549-hxfls.istio-system resources:0 size:0B
2021-07-27T03:12:13.789971Z     info    ads     CDS: PUSH for node:reviews-v1-6549ddccc5-6xwlr.default resources:22 size:19.7kB
2021-07-27T03:12:13.790028Z     info    ads     EDS: PUSH for node:reviews-v1-6549ddccc5-6xwlr.default resources:18 size:3.5kB empty:0 cached:18/18
2021-07-27T03:12:13.791319Z     info    ads     LDS: PUSH for node:reviews-v1-6549ddccc5-6xwlr.default resources:16 size:74.3kB
2021-07-27T03:12:13.802210Z     info    ads     RDS: PUSH for node:productpage-v1-5d9b4c9849-z4sx6.default resources:6 size:6.1kB
2021-07-27T03:12:13.802550Z     info    ads     RDS: PUSH for node:reviews-v1-6549ddccc5-6xwlr.default resources:6 size:6.1kB
2021-07-27T03:12:20.170339Z     info    ads     ADS: new connection for node:sidecar~10.244.1.9~details-v1-66b6955995-nxqnz.default~default.svc.cluster.local-7
2021-07-27T03:12:20.171402Z     info    ads     CDS: PUSH request for node:details-v1-66b6955995-nxqnz.default resources:22 size:19.7kB
2021-07-27T03:12:20.392916Z     info    ads     EDS: PUSH request for node:details-v1-66b6955995-nxqnz.default resources:18 size:3.5kB empty:0 cached:18/18
2021-07-27T03:12:20.739282Z     info    ads     LDS: PUSH request for node:details-v1-66b6955995-nxqnz.default resources:16 size:74.3kB
2021-07-27T03:12:20.861268Z     info    ads     RDS: PUSH request for node:details-v1-66b6955995-nxqnz.default resources:6 size:6.1kB
2021-07-27T03:12:21.251783Z     info    ads     Full push, new service default/details.default.svc.cluster.local
2021-07-27T03:12:21.353262Z     info    ads     Push debounce stable[17] 1: 101.406837ms since last change, 101.406646ms since last push, full=true
2021-07-27T03:12:21.353870Z     info    ads     XDS: Pushing:2021-07-27T03:12:21Z/10 Services:9 ConnectedEndpoints:7  Version:2021-07-27T03:12:21Z/10
2021-07-27T03:12:21.354608Z     info    ads     CDS: PUSH for node:productpage-v1-5d9b4c9849-z4sx6.default resources:22 size:19.8kB
2021-07-27T03:12:21.354726Z     info    ads     EDS: PUSH for node:productpage-v1-5d9b4c9849-z4sx6.default resources:18 size:3.6kB empty:0 cached:17/18
2021-07-27T03:12:21.355849Z     info    ads     LDS: PUSH for node:productpage-v1-5d9b4c9849-z4sx6.default resources:16 size:74.3kB
2021-07-27T03:12:21.376036Z     info    ads     CDS: PUSH for node:reviews-v1-6549ddccc5-6xwlr.default resources:22 size:19.8kB
2021-07-27T03:12:21.376148Z     info    ads     EDS: PUSH for node:reviews-v1-6549ddccc5-6xwlr.default resources:18 size:3.6kB empty:0 cached:18/18
2021-07-27T03:12:21.377624Z     info    ads     LDS: PUSH for node:reviews-v1-6549ddccc5-6xwlr.default resources:16 size:74.3kB
2021-07-27T03:12:21.377986Z     info    ads     CDS: PUSH for node:details-v1-66b6955995-nxqnz.default resources:22 size:19.8kB
2021-07-27T03:12:21.378038Z     info    ads     EDS: PUSH for node:details-v1-66b6955995-nxqnz.default resources:18 size:3.6kB empty:0 cached:18/18
2021-07-27T03:12:21.379086Z     info    ads     LDS: PUSH for node:details-v1-66b6955995-nxqnz.default resources:16 size:74.3kB
2021-07-27T03:12:21.394459Z     info    ads     CDS: PUSH for node:istio-ingressgateway-6ffdc56559-gf97t.istio-system resources:19 size:18.6kB
2021-07-27T03:12:21.394585Z     info    ads     EDS: PUSH for node:istio-ingressgateway-6ffdc56559-gf97t.istio-system resources:18 size:3.6kB empty:0 cached:18/18
2021-07-27T03:12:21.394747Z     info    ads     LDS: PUSH for node:istio-ingressgateway-6ffdc56559-gf97t.istio-system resources:0 size:0B
2021-07-27T03:12:21.394940Z     info    ads     CDS: PUSH for node:istio-egressgateway-7f8f879549-hxfls.istio-system resources:19 size:18.6kB
2021-07-27T03:12:21.395007Z     info    ads     EDS: PUSH for node:istio-egressgateway-7f8f879549-hxfls.istio-system resources:18 size:3.6kB empty:0 cached:18/18
2021-07-27T03:12:21.395077Z     info    ads     LDS: PUSH for node:istio-egressgateway-7f8f879549-hxfls.istio-system resources:0 size:0B
2021-07-27T03:12:21.395541Z     info    ads     RDS: PUSH for node:details-v1-66b6955995-nxqnz.default resources:6 size:6.1kB
2021-07-27T03:12:21.417656Z     info    ads     RDS: PUSH for node:productpage-v1-5d9b4c9849-z4sx6.default resources:6 size:6.1kB
2021-07-27T03:12:21.418045Z     info    ads     CDS: PUSH for node:reviews-v3-6b554c875-7wtc7.default resources:22 size:19.8kB
2021-07-27T03:12:21.418587Z     info    ads     EDS: PUSH for node:reviews-v3-6b554c875-7wtc7.default resources:18 size:3.6kB empty:0 cached:18/18
2021-07-27T03:12:21.427716Z     info    ads     CDS: PUSH for node:reviews-v2-76c4865449-mq9ml.default resources:22 size:19.8kB
2021-07-27T03:12:21.427816Z     info    ads     EDS: PUSH for node:reviews-v2-76c4865449-mq9ml.default resources:18 size:3.6kB empty:0 cached:18/18
2021-07-27T03:12:21.445650Z     info    ads     LDS: PUSH for node:reviews-v3-6b554c875-7wtc7.default resources:16 size:74.3kB
2021-07-27T03:12:21.447623Z     info    ads     RDS: PUSH for node:reviews-v1-6549ddccc5-6xwlr.default resources:6 size:6.1kB
2021-07-27T03:12:21.447803Z     info    ads     RDS: PUSH for node:reviews-v3-6b554c875-7wtc7.default resources:6 size:6.1kB
2021-07-27T03:12:21.450406Z     info    ads     LDS: PUSH for node:reviews-v2-76c4865449-mq9ml.default resources:16 size:74.3kB
2021-07-27T03:12:21.450663Z     info    ads     RDS: PUSH for node:reviews-v2-76c4865449-mq9ml.default resources:6 size:6.1kB
2021-07-27T03:12:37.407315Z     info    ads     ADS: new connection for node:sidecar~10.244.1.12~ratings-v1-fd78f799f-5hc75.default~default.svc.cluster.local-8
2021-07-27T03:12:37.408270Z     info    ads     CDS: PUSH request for node:ratings-v1-fd78f799f-5hc75.default resources:22 size:19.8kB
2021-07-27T03:12:37.434136Z     info    ads     EDS: PUSH request for node:ratings-v1-fd78f799f-5hc75.default resources:18 size:3.6kB empty:0 cached:18/18
2021-07-27T03:12:37.947589Z     info    ads     LDS: PUSH request for node:ratings-v1-fd78f799f-5hc75.default resources:16 size:74.3kB
2021-07-27T03:12:37.998286Z     info    ads     RDS: PUSH request for node:ratings-v1-fd78f799f-5hc75.default resources:6 size:6.1kB
2021-07-27T03:12:39.200259Z     info    ads     Full push, new service default/ratings.default.svc.cluster.local
2021-07-27T03:12:39.300802Z     info    ads     Push debounce stable[18] 1: 100.460936ms since last change, 100.460759ms since last push, full=true
2021-07-27T03:12:39.301933Z     info    ads     XDS: Pushing:2021-07-27T03:12:39Z/11 Services:9 ConnectedEndpoints:8  Version:2021-07-27T03:12:39Z/11
2021-07-27T03:12:39.302942Z     info    ads     CDS: PUSH for node:productpage-v1-5d9b4c9849-z4sx6.default resources:22 size:19.9kB
2021-07-27T03:12:39.304761Z     info    ads     CDS: PUSH for node:istio-egressgateway-7f8f879549-hxfls.istio-system resources:19 size:18.7kB
2021-07-27T03:12:39.336566Z     info    ads     EDS: PUSH for node:istio-egressgateway-7f8f879549-hxfls.istio-system resources:18 size:3.8kB empty:0 cached:17/18
2021-07-27T03:12:39.336758Z     info    ads     LDS: PUSH for node:istio-egressgateway-7f8f879549-hxfls.istio-system resources:0 size:0B
2021-07-27T03:12:39.304862Z     info    ads     CDS: PUSH for node:reviews-v1-6549ddccc5-6xwlr.default resources:22 size:19.9kB
2021-07-27T03:12:39.336870Z     info    ads     EDS: PUSH for node:reviews-v1-6549ddccc5-6xwlr.default resources:18 size:3.8kB empty:0 cached:18/18
2021-07-27T03:12:39.337647Z     info    ads     EDS: PUSH for node:productpage-v1-5d9b4c9849-z4sx6.default resources:18 size:3.8kB empty:0 cached:17/18
2021-07-27T03:12:39.339386Z     info    ads     LDS: PUSH for node:productpage-v1-5d9b4c9849-z4sx6.default resources:16 size:74.3kB
2021-07-27T03:12:39.339849Z     info    ads     RDS: PUSH for node:productpage-v1-5d9b4c9849-z4sx6.default resources:6 size:6.1kB
2021-07-27T03:12:39.305088Z     info    ads     CDS: PUSH for node:istio-ingressgateway-6ffdc56559-gf97t.istio-system resources:19 size:18.7kB
2021-07-27T03:12:39.339987Z     info    ads     EDS: PUSH for node:istio-ingressgateway-6ffdc56559-gf97t.istio-system resources:18 size:3.8kB empty:0 cached:18/18
2021-07-27T03:12:39.340045Z     info    ads     LDS: PUSH for node:istio-ingressgateway-6ffdc56559-gf97t.istio-system resources:0 size:0B
2021-07-27T03:12:39.305191Z     info    ads     CDS: PUSH for node:reviews-v3-6b554c875-7wtc7.default resources:22 size:19.9kB
2021-07-27T03:12:39.340163Z     info    ads     EDS: PUSH for node:reviews-v3-6b554c875-7wtc7.default resources:18 size:3.8kB empty:0 cached:18/18
2021-07-27T03:12:39.345760Z     info    ads     LDS: PUSH for node:reviews-v1-6549ddccc5-6xwlr.default resources:16 size:74.3kB
2021-07-27T03:12:39.346176Z     info    ads     RDS: PUSH for node:reviews-v1-6549ddccc5-6xwlr.default resources:6 size:6.1kB
2021-07-27T03:12:39.313300Z     info    ads     CDS: PUSH for node:reviews-v2-76c4865449-mq9ml.default resources:22 size:19.9kB
2021-07-27T03:12:39.346442Z     info    ads     EDS: PUSH for node:reviews-v2-76c4865449-mq9ml.default resources:18 size:3.8kB empty:0 cached:18/18
2021-07-27T03:12:39.347621Z     info    ads     LDS: PUSH for node:reviews-v2-76c4865449-mq9ml.default resources:16 size:74.3kB
2021-07-27T03:12:39.313471Z     info    ads     CDS: PUSH for node:ratings-v1-fd78f799f-5hc75.default resources:22 size:19.9kB
2021-07-27T03:12:39.347861Z     info    ads     EDS: PUSH for node:ratings-v1-fd78f799f-5hc75.default resources:18 size:3.8kB empty:0 cached:18/18
2021-07-27T03:12:39.353878Z     info    ads     LDS: PUSH for node:reviews-v3-6b554c875-7wtc7.default resources:16 size:74.3kB
2021-07-27T03:12:39.354719Z     info    ads     RDS: PUSH for node:reviews-v3-6b554c875-7wtc7.default resources:6 size:6.1kB
2021-07-27T03:12:39.361335Z     info    ads     RDS: PUSH for node:reviews-v2-76c4865449-mq9ml.default resources:6 size:6.1kB
2021-07-27T03:12:39.325142Z     info    ads     CDS: PUSH for node:details-v1-66b6955995-nxqnz.default resources:22 size:19.9kB
2021-07-27T03:12:39.361768Z     info    ads     EDS: PUSH for node:details-v1-66b6955995-nxqnz.default resources:18 size:3.8kB empty:0 cached:18/18
2021-07-27T03:12:39.363737Z     info    ads     LDS: PUSH for node:ratings-v1-fd78f799f-5hc75.default resources:16 size:74.3kB
2021-07-27T03:12:39.364349Z     info    ads     RDS: PUSH for node:ratings-v1-fd78f799f-5hc75.default resources:6 size:6.1kB
2021-07-27T03:12:39.365248Z     info    ads     LDS: PUSH for node:details-v1-66b6955995-nxqnz.default resources:16 size:74.3kB
2021-07-27T03:12:39.365463Z     info    ads     RDS: PUSH for node:details-v1-66b6955995-nxqnz.default resources:6 size:6.1kB
node01 $ 
```

