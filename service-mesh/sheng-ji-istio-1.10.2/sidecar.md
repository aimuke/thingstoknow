# sidecar

## ps -ef 

```text
node01 $ cat ps
UID        PID  PPID  C STIME TTY          TIME CMD
istio-p+     1     0  0 09:08 ?        00:00:00 /usr/local/bin/pilot-agent proxy sidecar --domain default.svc.cluster.local --serviceCluster productpage.default --proxyLogLevel=warning --proxyComponentLogLevel=misc:error --log_output_level=default:info --concurrency 2
istio-p+    11     1  0 09:08 ?        00:00:01 /usr/local/bin/envoy -c etc/istio/proxy/envoy-rev0.json --restart-epoch 0 --drain-time-s 45 --drain-strategy immediate --parent-shutdown-time-s 60 --service-cluster productpage.default --service-node sidecar~10.244.1.10~productpage-v1-5d9b4c9849-c6rbk.default~default.svc.cluster.local --local-address-ip-version v4 --bootstrap-version 3 --disable-hot-restart --log-format %Y-%m-%dT%T.%fZ.%l.envoy %n.%v -l warning --component-log-level misc:error --concurrency 2
istio-p+    57     0  0 09:14 ?        00:00:00 ps -ef
```

## netstat -anp

```text
node01 $ docker exec acc387522665 netstat -anp 
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:15006           0.0.0.0:*               LISTEN      11/envoy            
tcp        0      0 0.0.0.0:15021           0.0.0.0:*               LISTEN      11/envoy            
tcp        0      0 0.0.0.0:15090           0.0.0.0:*               LISTEN      11/envoy            
tcp        0      0 127.0.0.1:15000         0.0.0.0:*               LISTEN      11/envoy            
tcp        0      0 0.0.0.0:9080            0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:15001           0.0.0.0:*               LISTEN      11/envoy            
tcp        0      0 10.244.1.10:56470       10.103.91.213:15012     ESTABLISHED 1/pilot-agent       
tcp        0      0 127.0.0.1:35526         127.0.0.1:15020         ESTABLISHED 11/envoy            
tcp        0      0 10.244.1.10:56474       10.103.91.213:15012     ESTABLISHED 1/pilot-agent       
tcp        0      0 127.0.0.1:35542         127.0.0.1:15020         ESTABLISHED 11/envoy            
tcp6       0      0 :::15020                :::*                    LISTEN      1/pilot-agent       
tcp6       0      0 127.0.0.1:15020         127.0.0.1:35542         ESTABLISHED 1/pilot-agent       
tcp6       0      0 127.0.0.1:15020         127.0.0.1:35526         ESTABLISHED 1/pilot-agent       
Active UNIX domain sockets (servers and established)
Proto RefCnt Flags       Type       State         I-Node   PID/Program name     Path
unix  2      [ ACC ]     STREAM     LISTENING     77285    1/pilot-agent        ./etc/istio/proxy/SDS
unix  2      [ ACC ]     STREAM     LISTENING     77289    1/pilot-agent        ./etc/istio/proxy/XDS
unix  3      [ ]         STREAM     CONNECTED     77375    1/pilot-agent        ./etc/istio/proxy/SDS
unix  3      [ ]         STREAM     CONNECTED     75593    11/envoy             
unix  3      [ ]         STREAM     CONNECTED     75564    11/envoy             
unix  3      [ ]         STREAM     CONNECTED     77361    1/pilot-agent        ./etc/istio/proxy/XDS
node01 $ 
```

## docker inspect

```text
node01 $ docker inspect  acc387522665 
[
    {
        "Id": "acc3875226653de409b03962002c409068a605266d13c50a96e33af792e98b39",
        "Created": "2021-07-29T09:08:17.50977529Z",
        "Path": "/usr/local/bin/pilot-agent",
        "Args": [
            "proxy",
            "sidecar",
            "--domain",
            "default.svc.cluster.local",
            "--serviceCluster",
            "productpage.default",
            "--proxyLogLevel=warning",
            "--proxyComponentLogLevel=misc:error",
            "--log_output_level=default:info",
            "--concurrency",
            "2"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 10081,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2021-07-29T09:08:18.61387811Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:37f4e188f5b6f2e1455c0469e89c178c6e10a99e6310c9406fd5b00ea532c6d5",
        "ResolvConfPath": "/var/lib/docker/containers/6f7ed04d7db823421475c4364d020d892d8d72398c10ead9ef2eb36aefdc9cb0/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/6f7ed04d7db823421475c4364d020d892d8d72398c10ead9ef2eb36aefdc9cb0/hostname",
        "HostsPath": "/var/lib/kubelet/pods/548c4779-f884-40a1-afd7-3cb7ef4ca620/etc-hosts",
        "LogPath": "/var/lib/docker/containers/acc3875226653de409b03962002c409068a605266d13c50a96e33af792e98b39/acc3875226653de409b03962002c409068a605266d13c50a96e33af792e98b39-json.log",
        "Name": "/k8s_istio-proxy_productpage-v1-5d9b4c9849-c6rbk_default_548c4779-f884-40a1-afd7-3cb7ef4ca620_0",
        "RestartCount": 0,
        "Driver": "overlay",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "docker-default",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": [
                "/var/lib/kubelet/pods/548c4779-f884-40a1-afd7-3cb7ef4ca620/volumes/kubernetes.io~configmap/istiod-ca-cert:/var/run/secrets/istio:ro",
                "/var/lib/kubelet/pods/548c4779-f884-40a1-afd7-3cb7ef4ca620/volumes/kubernetes.io~empty-dir/istio-data:/var/lib/istio/data",
                "/var/lib/kubelet/pods/548c4779-f884-40a1-afd7-3cb7ef4ca620/volumes/kubernetes.io~empty-dir/istio-envoy:/etc/istio/proxy",
                "/var/lib/kubelet/pods/548c4779-f884-40a1-afd7-3cb7ef4ca620/volumes/kubernetes.io~downward-api/istio-podinfo:/etc/istio/pod:ro",
                "/var/lib/kubelet/pods/548c4779-f884-40a1-afd7-3cb7ef4ca620/volumes/kubernetes.io~secret/bookinfo-productpage-token-hwwqm:/var/run/secrets/kubernetes.io/serviceaccount:ro",
                "/var/lib/kubelet/pods/548c4779-f884-40a1-afd7-3cb7ef4ca620/etc-hosts:/etc/hosts",
                "/var/lib/kubelet/pods/548c4779-f884-40a1-afd7-3cb7ef4ca620/containers/istio-proxy/fd0ac660:/dev/termination-log"
            ],
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {
                    "max-size": "100m"
                }
            },
            "NetworkMode": "container:6f7ed04d7db823421475c4364d020d892d8d72398c10ead9ef2eb36aefdc9cb0",
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
            "IpcMode": "container:6f7ed04d7db823421475c4364d020d892d8d72398c10ead9ef2eb36aefdc9cb0",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 990,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": true,
            "SecurityOpt": [
                "no-new-privileges",
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
            "Memory": 1073741824,
            "NanoCpus": 0,
            "CgroupParent": "kubepods-burstable-pod548c4779_f884_40a1_afd7_3cb7ef4ca620.slice",
            "BlkioWeight": 0,
            "BlkioWeightDevice": null,
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 100000,
            "CpuQuota": 200000,
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
            "MemorySwap": -1,
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
                "LowerDir": "/var/lib/docker/overlay/8fab3ab014ac77e9bb61fff43282e966589b202f5f344bc5376c3ae76c423d47/root",
                "MergedDir": "/var/lib/docker/overlay/21b399bc888bb5f051e35aa929ad63a04a77873107093de7c1ca5915560b8813/merged",
                "UpperDir": "/var/lib/docker/overlay/21b399bc888bb5f051e35aa929ad63a04a77873107093de7c1ca5915560b8813/upper",
                "WorkDir": "/var/lib/docker/overlay/21b399bc888bb5f051e35aa929ad63a04a77873107093de7c1ca5915560b8813/work"
            },
            "Name": "overlay"
        },
        "Mounts": [
            {
                "Type": "bind",
                "Source": "/var/lib/kubelet/pods/548c4779-f884-40a1-afd7-3cb7ef4ca620/volumes/kubernetes.io~configmap/istiod-ca-cert",
                "Destination": "/var/run/secrets/istio",
                "Mode": "ro",
                "RW": false,
                "Propagation": "rprivate"
            },
            {
                "Type": "bind",
                "Source": "/var/lib/kubelet/pods/548c4779-f884-40a1-afd7-3cb7ef4ca620/volumes/kubernetes.io~empty-dir/istio-data",
                "Destination": "/var/lib/istio/data",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            },
            {
                "Type": "bind",
                "Source": "/var/lib/kubelet/pods/548c4779-f884-40a1-afd7-3cb7ef4ca620/volumes/kubernetes.io~empty-dir/istio-envoy",
                "Destination": "/etc/istio/proxy",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            },
            {
                "Type": "bind",
                "Source": "/var/lib/kubelet/pods/548c4779-f884-40a1-afd7-3cb7ef4ca620/volumes/kubernetes.io~downward-api/istio-podinfo",
                "Destination": "/etc/istio/pod",
                "Mode": "ro",
                "RW": false,
                "Propagation": "rprivate"
            },
            {
                "Type": "bind",
                "Source": "/var/lib/kubelet/pods/548c4779-f884-40a1-afd7-3cb7ef4ca620/volumes/kubernetes.io~secret/bookinfo-productpage-token-hwwqm",
                "Destination": "/var/run/secrets/kubernetes.io/serviceaccount",
                "Mode": "ro",
                "RW": false,
                "Propagation": "rprivate"
            },
            {
                "Type": "bind",
                "Source": "/var/lib/kubelet/pods/548c4779-f884-40a1-afd7-3cb7ef4ca620/etc-hosts",
                "Destination": "/etc/hosts",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            },
            {
                "Type": "bind",
                "Source": "/var/lib/kubelet/pods/548c4779-f884-40a1-afd7-3cb7ef4ca620/containers/istio-proxy/fd0ac660",
                "Destination": "/dev/termination-log",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            }
        ],
        "Config": {
            "Hostname": "productpage-v1-5d9b4c9849-c6rbk",
            "Domainname": "",
            "User": "1337:1337",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "HOST_IP=172.17.0.13",
                "CANONICAL_REVISION=v1",
                "ISTIO_META_INTERCEPTION_MODE=REDIRECT",
                "INSTANCE_IP=10.244.1.10",
                "ISTIO_META_APP_CONTAINERS=productpage",
                "ISTIO_META_WORKLOAD_NAME=productpage-v1",
                "TRUST_DOMAIN=cluster.local",
                "POD_NAME=productpage-v1-5d9b4c9849-c6rbk",
                "ISTIO_META_OWNER=kubernetes://apis/apps/v1/namespaces/default/deployments/productpage-v1",
                "ISTIO_META_MESH_ID=cluster.local",
                "ISTIO_META_POD_PORTS=[\n    {\"containerPort\":9080,\"protocol\":\"TCP\"}\n]",
                "JWT_POLICY=first-party-jwt",
                "PILOT_CERT_PROVIDER=istiod",
                "CA_ADDR=istiod.istio-system.svc:15012",
                "POD_NAMESPACE=default",
                "SERVICE_ACCOUNT=bookinfo-productpage",
                "CANONICAL_SERVICE=productpage",
                "PROXY_CONFIG={}\n",
                "ISTIO_META_CLUSTER_ID=Kubernetes",
                "REVIEWS_SERVICE_HOST=10.108.108.18",
                "REVIEWS_PORT=tcp://10.108.108.18:9080",
                "KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1",
                "PRODUCTPAGE_PORT_9080_TCP_ADDR=10.107.11.142",
                "DETAILS_SERVICE_HOST=10.103.196.48",
                "DETAILS_PORT_9080_TCP=tcp://10.103.196.48:9080",
                "RATINGS_PORT_9080_TCP_PORT=9080",
                "REVIEWS_SERVICE_PORT_HTTP=9080",
                "KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443",
                "PRODUCTPAGE_SERVICE_HOST=10.107.11.142",
                "DETAILS_PORT_9080_TCP_ADDR=10.103.196.48",
                "RATINGS_PORT_9080_TCP=tcp://10.109.175.138:9080",
                "KUBERNETES_PORT_443_TCP_PORT=443",
                "DETAILS_SERVICE_PORT_HTTP=9080",
                "REVIEWS_SERVICE_PORT=9080",
                "REVIEWS_PORT_9080_TCP_ADDR=10.108.108.18",
                "DETAILS_PORT=tcp://10.103.196.48:9080",
                "RATINGS_SERVICE_PORT=9080",
                "RATINGS_SERVICE_PORT_HTTP=9080",
                "REVIEWS_PORT_9080_TCP_PROTO=tcp",
                "KUBERNETES_SERVICE_HOST=10.96.0.1",
                "PRODUCTPAGE_SERVICE_PORT=9080",
                "PRODUCTPAGE_PORT_9080_TCP=tcp://10.107.11.142:9080",
                "DETAILS_SERVICE_PORT=9080",
                "KUBERNETES_PORT_443_TCP_PROTO=tcp",
                "PRODUCTPAGE_PORT=tcp://10.107.11.142:9080",
                "PRODUCTPAGE_PORT_9080_TCP_PROTO=tcp",
                "RATINGS_PORT_9080_TCP_PROTO=tcp",
                "KUBERNETES_SERVICE_PORT=443",
                "KUBERNETES_PORT=tcp://10.96.0.1:443",
                "DETAILS_PORT_9080_TCP_PROTO=tcp",
                "REVIEWS_PORT_9080_TCP=tcp://10.108.108.18:9080",
                "REVIEWS_PORT_9080_TCP_PORT=9080",
                "RATINGS_SERVICE_HOST=10.109.175.138",
                "RATINGS_PORT_9080_TCP_ADDR=10.109.175.138",
                "KUBERNETES_SERVICE_PORT_HTTPS=443",
                "PRODUCTPAGE_SERVICE_PORT_HTTP=9080",
                "PRODUCTPAGE_PORT_9080_TCP_PORT=9080",
                "DETAILS_PORT_9080_TCP_PORT=9080",
                "RATINGS_PORT=tcp://10.109.175.138:9080",
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "DEBIAN_FRONTEND=noninteractive",
                "ISTIO_META_ISTIO_PROXY_SHA=istio-proxy:4b528a87271e841bd64daf841a1a384ed4fcac68",
                "ISTIO_META_ISTIO_VERSION=1.10.3"
            ],
            "Cmd": [
                "proxy",
                "sidecar",
                "--domain",
                "default.svc.cluster.local",
                "--serviceCluster",
                "productpage.default",
                "--proxyLogLevel=warning",
                "--proxyComponentLogLevel=misc:error",
                "--log_output_level=default:info",
                "--concurrency",
                "2"
            ],
            "Healthcheck": {
                "Test": [
                    "NONE"
                ]
            },
            "Image": "sha256:37f4e188f5b6f2e1455c0469e89c178c6e10a99e6310c9406fd5b00ea532c6d5",
            "Volumes": null,
            "WorkingDir": "/",
            "Entrypoint": [
                "/usr/local/bin/pilot-agent"
            ],
            "OnBuild": null,
            "Labels": {
                "annotation.io.kubernetes.container.hash": "e5e745fa",
                "annotation.io.kubernetes.container.ports": "[{\"name\":\"http-envoy-prom\",\"containerPort\":15090,\"protocol\":\"TCP\"}]",
                "annotation.io.kubernetes.container.restartCount": "0",
                "annotation.io.kubernetes.container.terminationMessagePath": "/dev/termination-log",
                "annotation.io.kubernetes.container.terminationMessagePolicy": "File",
                "annotation.io.kubernetes.pod.terminationGracePeriod": "30",
                "io.kubernetes.container.logpath": "/var/log/pods/default_productpage-v1-5d9b4c9849-c6rbk_548c4779-f884-40a1-afd7-3cb7ef4ca620/istio-proxy/0.log",
                "io.kubernetes.container.name": "istio-proxy",
                "io.kubernetes.docker.type": "container",
                "io.kubernetes.pod.name": "productpage-v1-5d9b4c9849-c6rbk",
                "io.kubernetes.pod.namespace": "default",
                "io.kubernetes.pod.uid": "548c4779-f884-40a1-afd7-3cb7ef4ca620",
                "io.kubernetes.sandbox.id": "6f7ed04d7db823421475c4364d020d892d8d72398c10ead9ef2eb36aefdc9cb0"
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

## envoy-rev0.json

```text
node01 $ docker exec acc387522665 cat  /etc/istio/proxy/envoy-rev0.json
{
  "node": {
    "id": "sidecar~10.244.1.10~productpage-v1-5d9b4c9849-c6rbk.default~default.svc.cluster.local",
    "cluster": "productpage.default",
    "locality": {
    },
    "metadata": {"APP_CONTAINERS":"productpage","CLUSTER_ID":"Kubernetes","INSTANCE_IPS":"10.244.1.10","INTERCEPTION_MODE":"REDIRECT","ISTIO_PROXY_SHA":"istio-proxy:4b528a87271e841bd64daf841a1a384ed4fcac68","ISTIO_VERSION":"1.10.3","LABELS":{"app":"productpage","istio.io/rev":"default","pod-template-hash":"5d9b4c9849","security.istio.io/tlsMode":"istio","service.istio.io/canonical-name":"productpage","service.istio.io/canonical-revision":"v1","version":"v1"},"MESH_ID":"cluster.local","NAME":"productpage-v1-5d9b4c9849-c6rbk","NAMESPACE":"default","OWNER":"kubernetes://apis/apps/v1/namespaces/default/deployments/productpage-v1","PILOT_CERT_PROVIDER":"istiod","PILOT_SAN":["istiod.istio-system.svc"],"POD_PORTS":"[{\"containerPort\":9080,\"protocol\":\"TCP\"}]","PROV_CERT":"var/run/secrets/istio/root-cert.pem","PROXY_CONFIG":{"binaryPath":"/usr/local/bin/envoy","concurrency":2,"configPath":"./etc/istio/proxy","controlPlaneAuthPolicy":"MUTUAL_TLS","discoveryAddress":"istiod.istio-system.svc:15012","drainDuration":"45s","parentShutdownDuration":"60s","proxyAdminPort":15000,"serviceCluster":"productpage.default","statNameLength":189,"statusPort":15020,"terminationDrainDuration":"5s","tracing":{"zipkin":{"address":"zipkin.istio-system:9411"}}},"PROXY_VIA_AGENT":true,"SERVICE_ACCOUNT":"bookinfo-productpage","WORKLOAD_NAME":"productpage-v1"}
  },
  "layered_runtime": {
      "layers": [
          {
              "name": "deprecation",
              "static_layer": {
                  "envoy.deprecated_features:envoy.config.listener.v3.Listener.hidden_envoy_deprecated_use_original_dst": true,
                  "envoy.reloadable_features.strict_1xx_and_204_response_headers": false,
                  "re2.max_program_size.error_level": 1024,
                  "envoy.reloadable_features.treat_host_like_authority": false
              }
          },
          {
            "name": "global config",
            "static_layer": {
                "overload.global_downstream_max_connections": 2147483647
            }
          },
          {
              "name": "admin",
              "admin_layer": {}
          }
      ]
  },
  "stats_config": {
    "use_all_default_tags": false,
    "stats_tags": [
      {
        "tag_name": "cluster_name",
        "regex": "^cluster\\.((.+?(\\..+?\\.svc\\.cluster\\.local)?)\\.)"
      },
      {
        "tag_name": "tcp_prefix",
        "regex": "^tcp\\.((.*?)\\.)\\w+?$"
      },
      {
        "regex": "(response_code=\\.=(.+?);\\.;)|_rq(_(\\.d{3}))$",
        "tag_name": "response_code"
      },
      {
        "tag_name": "response_code_class",
        "regex": "_rq(_(\\dxx))$"
      },
      {
        "tag_name": "http_conn_manager_listener_prefix",
        "regex": "^listener(?=\\.).*?\\.http\\.(((?:[_.[:digit:]]*|[_\\[\\]aAbBcCdDeEfF[:digit:]]*))\\.)"
      },
      {
        "tag_name": "http_conn_manager_prefix",
        "regex": "^http\\.(((?:[_.[:digit:]]*|[_\\[\\]aAbBcCdDeEfF[:digit:]]*))\\.)"
      },
      {
        "tag_name": "listener_address",
        "regex": "^listener\\.(((?:[_.[:digit:]]*|[_\\[\\]aAbBcCdDeEfF[:digit:]]*))\\.)"
      },
      {
        "tag_name": "mongo_prefix",
        "regex": "^mongo\\.(.+?)\\.(collection|cmd|cx_|op_|delays_|decoding_)(.*?)$"
      },
      {
        "regex": "(reporter=\\.=(.*?);\\.;)",
        "tag_name": "reporter"
      },
      {
        "regex": "(source_namespace=\\.=(.*?);\\.;)",
        "tag_name": "source_namespace"
      },
      {
        "regex": "(source_workload=\\.=(.*?);\\.;)",
        "tag_name": "source_workload"
      },
      {
        "regex": "(source_workload_namespace=\\.=(.*?);\\.;)",
        "tag_name": "source_workload_namespace"
      },
      {
        "regex": "(source_principal=\\.=(.*?);\\.;)",
        "tag_name": "source_principal"
      },
      {
        "regex": "(source_app=\\.=(.*?);\\.;)",
        "tag_name": "source_app"
      },
      {
        "regex": "(source_version=\\.=(.*?);\\.;)",
        "tag_name": "source_version"
      },
      {
        "regex": "(source_cluster=\\.=(.*?);\\.;)",
        "tag_name": "source_cluster"
      },
      {
        "regex": "(destination_namespace=\\.=(.*?);\\.;)",
        "tag_name": "destination_namespace"
      },
      {
        "regex": "(destination_workload=\\.=(.*?);\\.;)",
        "tag_name": "destination_workload"
      },
      {
        "regex": "(destination_workload_namespace=\\.=(.*?);\\.;)",
        "tag_name": "destination_workload_namespace"
      },
      {
        "regex": "(destination_principal=\\.=(.*?);\\.;)",
        "tag_name": "destination_principal"
      },
      {
        "regex": "(destination_app=\\.=(.*?);\\.;)",
        "tag_name": "destination_app"
      },
      {
        "regex": "(destination_version=\\.=(.*?);\\.;)",
        "tag_name": "destination_version"
      },
      {
        "regex": "(destination_service=\\.=(.*?);\\.;)",
        "tag_name": "destination_service"
      },
      {
        "regex": "(destination_service_name=\\.=(.*?);\\.;)",
        "tag_name": "destination_service_name"
      },
      {
        "regex": "(destination_service_namespace=\\.=(.*?);\\.;)",
        "tag_name": "destination_service_namespace"
      },
      {
        "regex": "(destination_port=\\.=(.*?);\\.;)",
        "tag_name": "destination_port"
      },
      {
        "regex": "(destination_cluster=\\.=(.*?);\\.;)",
        "tag_name": "destination_cluster"
      },
      {
        "regex": "(request_protocol=\\.=(.*?);\\.;)",
        "tag_name": "request_protocol"
      },
      {
        "regex": "(request_operation=\\.=(.*?);\\.;)",
        "tag_name": "request_operation"
      },
      {
        "regex": "(request_host=\\.=(.*?);\\.;)",
        "tag_name": "request_host"
      },
      {
        "regex": "(response_flags=\\.=(.*?);\\.;)",
        "tag_name": "response_flags"
      },
      {
        "regex": "(grpc_response_status=\\.=(.*?);\\.;)",
        "tag_name": "grpc_response_status"
      },
      {
        "regex": "(connection_security_policy=\\.=(.*?);\\.;)",
        "tag_name": "connection_security_policy"
      },
      {
        "regex": "(source_canonical_service=\\.=(.*?);\\.;)",
        "tag_name": "source_canonical_service"
      },
      {
        "regex": "(destination_canonical_service=\\.=(.*?);\\.;)",
        "tag_name": "destination_canonical_service"
      },
      {
        "regex": "(source_canonical_revision=\\.=(.*?);\\.;)",
        "tag_name": "source_canonical_revision"
      },
      {
        "regex": "(destination_canonical_revision=\\.=(.*?);\\.;)",
        "tag_name": "destination_canonical_revision"
      },
      {
        "regex": "(cache\\.(.+?)\\.)",
        "tag_name": "cache"
      },
      {
        "regex": "(component\\.(.+?)\\.)",
        "tag_name": "component"
      },
      {
        "regex": "(tag\\.(.+?);\\.)",
        "tag_name": "tag"
      },
      {
        "regex": "(wasm_filter\\.(.+?)\\.)",
        "tag_name": "wasm_filter"
      },
      {
        "tag_name": "authz_enforce_result",
        "regex": "rbac(\\.(allowed|denied))"
      },
      {
        "tag_name": "authz_dry_run_action",
        "regex": "(\\.istio_dry_run_(allow|deny)_)"
      },
      {
        "tag_name": "authz_dry_run_result",
        "regex": "(\\.shadow_(allowed|denied))"
      }
    ],
    "stats_matcher": {
      "inclusion_list": {
        "patterns": [
          {
          "prefix": "reporter="
          },
          {
          "prefix": "cluster_manager"
          },
          {
          "prefix": "listener_manager"
          },
          {
          "prefix": "server"
          },
          {
          "prefix": "cluster.xds-grpc"
          },
          {
          "prefix": "wasm"
          },
          {
          "suffix": "rbac.allowed"
          },
          {
          "suffix": "rbac.denied"
          },
          {
          "suffix": "shadow_allowed"
          },
          {
          "suffix": "shadow_denied"
          },
          {
          "prefix": "component"
          }
        ]
      }
    }
  },
  "admin": {
    "access_log_path": "/dev/null",
    "profile_path": "/var/lib/istio/data/envoy.prof",
    "address": {
      "socket_address": {
        "address": "127.0.0.1",
        "port_value": 15000
      }
    }
  },
  "dynamic_resources": {
    "lds_config": {
      "ads": {},
      "initial_fetch_timeout": "0s",
      "resource_api_version": "V3"
    },
    "cds_config": {
      "ads": {},
      "initial_fetch_timeout": "0s",
      "resource_api_version": "V3"
    },
    "ads_config": {
      "api_type": "GRPC",
      "set_node_on_first_message_only": true,
      "transport_api_version": "V3",
      "grpc_services": [
        {
          "envoy_grpc": {
            "cluster_name": "xds-grpc"
          }
        }
      ]
    }
  },
  "static_resources": {
    "clusters": [
      {
        "name": "prometheus_stats",
        "type": "STATIC",
        "connect_timeout": "0.250s",
        "lb_policy": "ROUND_ROBIN",
        "load_assignment": {
          "cluster_name": "prometheus_stats",
          "endpoints": [{
            "lb_endpoints": [{
              "endpoint": {
                "address":{
                  "socket_address": {
                    "protocol": "TCP",
                    "address": "127.0.0.1",
                    "port_value": 15000
                  }
                }
              }
            }]
          }]
        }
      },
      {
        "name": "agent",
        "type": "STATIC",
        "connect_timeout": "0.250s",
        "lb_policy": "ROUND_ROBIN",
        "load_assignment": {
          "cluster_name": "agent",
          "endpoints": [{
            "lb_endpoints": [{
              "endpoint": {
                "address":{
                  "socket_address": {
                    "protocol": "TCP",
                    "address": "127.0.0.1",
                    "port_value": 15020
                  }
                }
              }
            }]
          }]
        }
      },
      {
        "name": "sds-grpc",
        "type": "STATIC",
        "http2_protocol_options": {},
        "connect_timeout": "1s",
        "lb_policy": "ROUND_ROBIN",
        "load_assignment": {
          "cluster_name": "sds-grpc",
          "endpoints": [{
            "lb_endpoints": [{
              "endpoint": {
                "address":{
                  "pipe": {
                    "path": "./etc/istio/proxy/SDS"
                  }
                }
              }
            }]
          }]
        }
      },
      {
        "name": "xds-grpc",
        "type" : "STATIC",
        "connect_timeout": "1s",
        "lb_policy": "ROUND_ROBIN",
        "load_assignment": {
          "cluster_name": "xds-grpc",
          "endpoints": [{
            "lb_endpoints": [{
              "endpoint": {
                "address":{
                  "pipe": {
                    "path": "./etc/istio/proxy/XDS"
                  }
                }
              }
            }]
          }]
        },
        "circuit_breakers": {
          "thresholds": [
            {
              "priority": "DEFAULT",
              "max_connections": 100000,
              "max_pending_requests": 100000,
              "max_requests": 100000
            },
            {
              "priority": "HIGH",
              "max_connections": 100000,
              "max_pending_requests": 100000,
              "max_requests": 100000
            }
          ]
        },
        "upstream_connection_options": {
          "tcp_keepalive": {
            "keepalive_time": 300
          }
        },
        "max_requests_per_connection": 1,
        "http2_protocol_options": { }
      }
      
      ,
      {
        "name": "zipkin",
        "type": "STRICT_DNS",
        "respect_dns_ttl": true,
        "dns_lookup_family": "V4_ONLY",
        "dns_refresh_rate": "30s",
        "connect_timeout": "1s",
        "lb_policy": "ROUND_ROBIN",
        "load_assignment": {
          "cluster_name": "zipkin",
          "endpoints": [{
            "lb_endpoints": [{
              "endpoint": {
                "address":{
                  "socket_address": {"address": "zipkin.istio-system", "port_value": 9411}
                }
              }
            }]
          }]
        }
      }
      
      
    ],
    "listeners":[
      {
        "address": {
          "socket_address": {
            "protocol": "TCP",
            "address": "0.0.0.0",
            "port_value": 15090
          }
        },
        "filter_chains": [
          {
            "filters": [
              {
                "name": "envoy.filters.network.http_connection_manager",
                "typed_config": {
                  "@type": "type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager",
                  "codec_type": "AUTO",
                  "stat_prefix": "stats",
                  "route_config": {
                    "virtual_hosts": [
                      {
                        "name": "backend",
                        "domains": [
                          "*"
                        ],
                        "routes": [
                          {
                            "match": {
                              "prefix": "/stats/prometheus"
                            },
                            "route": {
                              "cluster": "prometheus_stats"
                            }
                          }
                        ]
                      }
                    ]
                  },
                  "http_filters": [{
                    "name": "envoy.filters.http.router",
                    "typed_config": {
                      "@type": "type.googleapis.com/envoy.extensions.filters.http.router.v3.Router"
                    }
                  }]
                }
              }
            ]
          }
        ]
      },
      {
        "address": {
          "socket_address": {
            "protocol": "TCP",
            "address": "0.0.0.0",
            "port_value": 15021
          }
        },
        "filter_chains": [
          {
            "filters": [
              {
                "name": "envoy.filters.network.http_connection_manager",
                "typed_config": {
                  "@type": "type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager",
                  "codec_type": "AUTO",
                  "stat_prefix": "agent",
                  "route_config": {
                    "virtual_hosts": [
                      {
                        "name": "backend",
                        "domains": [
                          "*"
                        ],
                        "routes": [
                          {
                            "match": {
                              "prefix": "/healthz/ready"
                            },
                            "route": {
                              "cluster": "agent"
                            }
                          }
                        ]
                      }
                    ]
                  },
                  "http_filters": [{
                    "name": "envoy.filters.http.router",
                    "typed_config": {
                      "@type": "type.googleapis.com/envoy.extensions.filters.http.router.v3.Router"
                    }
                  }]
                }
              }
            ]
          }
        ]
      }
    ]
  }
  ,
  "tracing": {
    "http": {
      "name": "envoy.tracers.zipkin",
      "typed_config": {
        "@type": "type.googleapis.com/envoy.config.trace.v3.ZipkinConfig",
        "collector_cluster": "zipkin",
        "collector_endpoint": "/api/v2/spans",
        "collector_endpoint_version": "HTTP_JSON",
        "trace_id_128bit": true,
        "shared_span_context": false
      }
    }
  }
  
  
}
```



## docker env

```text
node01 $ docker exec acc387522665 env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=productpage-v1-5d9b4c9849-c6rbk
HOST_IP=172.17.0.13
CANONICAL_REVISION=v1
ISTIO_META_INTERCEPTION_MODE=REDIRECT
INSTANCE_IP=10.244.1.10
ISTIO_META_APP_CONTAINERS=productpage
ISTIO_META_WORKLOAD_NAME=productpage-v1
TRUST_DOMAIN=cluster.local
POD_NAME=productpage-v1-5d9b4c9849-c6rbk
ISTIO_META_OWNER=kubernetes://apis/apps/v1/namespaces/default/deployments/productpage-v1
ISTIO_META_MESH_ID=cluster.local
ISTIO_META_POD_PORTS=[
    {"containerPort":9080,"protocol":"TCP"}
]
JWT_POLICY=first-party-jwt
PILOT_CERT_PROVIDER=istiod
CA_ADDR=istiod.istio-system.svc:15012
POD_NAMESPACE=default
SERVICE_ACCOUNT=bookinfo-productpage
CANONICAL_SERVICE=productpage
PROXY_CONFIG={}

ISTIO_META_CLUSTER_ID=Kubernetes
REVIEWS_SERVICE_HOST=10.108.108.18
REVIEWS_PORT=tcp://10.108.108.18:9080
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
PRODUCTPAGE_PORT_9080_TCP_ADDR=10.107.11.142
DETAILS_SERVICE_HOST=10.103.196.48
DETAILS_PORT_9080_TCP=tcp://10.103.196.48:9080
RATINGS_PORT_9080_TCP_PORT=9080
REVIEWS_SERVICE_PORT_HTTP=9080
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
PRODUCTPAGE_SERVICE_HOST=10.107.11.142
DETAILS_PORT_9080_TCP_ADDR=10.103.196.48
RATINGS_PORT_9080_TCP=tcp://10.109.175.138:9080
KUBERNETES_PORT_443_TCP_PORT=443
DETAILS_SERVICE_PORT_HTTP=9080
REVIEWS_SERVICE_PORT=9080
REVIEWS_PORT_9080_TCP_ADDR=10.108.108.18
DETAILS_PORT=tcp://10.103.196.48:9080
RATINGS_SERVICE_PORT=9080
RATINGS_SERVICE_PORT_HTTP=9080
REVIEWS_PORT_9080_TCP_PROTO=tcp
KUBERNETES_SERVICE_HOST=10.96.0.1
PRODUCTPAGE_SERVICE_PORT=9080
PRODUCTPAGE_PORT_9080_TCP=tcp://10.107.11.142:9080
DETAILS_SERVICE_PORT=9080
KUBERNETES_PORT_443_TCP_PROTO=tcp
PRODUCTPAGE_PORT=tcp://10.107.11.142:9080
PRODUCTPAGE_PORT_9080_TCP_PROTO=tcp
RATINGS_PORT_9080_TCP_PROTO=tcp
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://10.96.0.1:443
DETAILS_PORT_9080_TCP_PROTO=tcp
REVIEWS_PORT_9080_TCP=tcp://10.108.108.18:9080
REVIEWS_PORT_9080_TCP_PORT=9080
RATINGS_SERVICE_HOST=10.109.175.138
RATINGS_PORT_9080_TCP_ADDR=10.109.175.138
KUBERNETES_SERVICE_PORT_HTTPS=443
PRODUCTPAGE_SERVICE_PORT_HTTP=9080
PRODUCTPAGE_PORT_9080_TCP_PORT=9080
DETAILS_PORT_9080_TCP_PORT=9080
RATINGS_PORT=tcp://10.109.175.138:9080
DEBIAN_FRONTEND=noninteractive
ISTIO_META_ISTIO_PROXY_SHA=istio-proxy:4b528a87271e841bd64daf841a1a384ed4fcac68
ISTIO_META_ISTIO_VERSION=1.10.3
HOME=/home/istio-proxy
node01 $ 
```

## docker logs

```text
2021-07-30T03:45:17.640296Z     info    FLAG: --concurrency="2"
2021-07-30T03:45:17.640332Z     info    FLAG: --domain="default.svc.cluster.local"
2021-07-30T03:45:17.640340Z     info    FLAG: --help="false"
2021-07-30T03:45:17.640345Z     info    FLAG: --log_as_json="false"
2021-07-30T03:45:17.640354Z     info    FLAG: --log_caller=""
2021-07-30T03:45:17.640359Z     info    FLAG: --log_output_level="default:info"
2021-07-30T03:45:17.640362Z     info    FLAG: --log_rotate=""
2021-07-30T03:45:17.640535Z     info    FLAG: --log_rotate_max_age="30"
2021-07-30T03:45:17.640543Z     info    FLAG: --log_rotate_max_backups="1000"
2021-07-30T03:45:17.640547Z     info    FLAG: --log_rotate_max_size="104857600"
2021-07-30T03:45:17.640552Z     info    FLAG: --log_stacktrace_level="default:none"
2021-07-30T03:45:17.640571Z     info    FLAG: --log_target="[stdout]"
2021-07-30T03:45:17.640578Z     info    FLAG: --meshConfig="./etc/istio/config/mesh"
2021-07-30T03:45:17.640580Z     info    FLAG: --outlierLogPath=""
2021-07-30T03:45:17.640583Z     info    FLAG: --proxyComponentLogLevel="misc:error"
2021-07-30T03:45:17.640685Z     info    FLAG: --proxyLogLevel="warning"
2021-07-30T03:45:17.640692Z     info    FLAG: --serviceCluster="productpage.default"
2021-07-30T03:45:17.640696Z     info    FLAG: --stsPort="0"
2021-07-30T03:45:17.640698Z     info    FLAG: --templateFile=""
2021-07-30T03:45:17.640701Z     info    FLAG: --tokenManagerPlugin="GoogleTokenExchange"
2021-07-30T03:45:17.640704Z     info    Version 1.10.3-61313778e0b785e401c696f5e92f47af069f96d0-Clean
2021-07-30T03:45:17.640900Z     info    Proxy role      ips=[10.244.1.9] type=sidecar id=productpage-v1-5d9b4c9849-h8dvn.default domain=default.svc.cluster.local
2021-07-30T03:45:17.641036Z     info    Apply proxy config from env {}

2021-07-30T03:45:17.641919Z     info    Effective config: binaryPath: /usr/local/bin/envoy
concurrency: 2
configPath: ./etc/istio/proxy
controlPlaneAuthPolicy: MUTUAL_TLS
discoveryAddress: istiod.istio-system.svc:15012
drainDuration: 45s
parentShutdownDuration: 60s
proxyAdminPort: 15000
serviceCluster: productpage.default
statNameLength: 189
statusPort: 15020
terminationDrainDuration: 5s
tracing:
  zipkin:
    address: zipkin.istio-system:9411

2021-07-30T03:45:17.641938Z     info    JWT policy is first-party-jwt
2021-07-30T03:45:17.641947Z     info    Pilot SAN: [istiod.istio-system.svc]
2021-07-30T03:45:17.641950Z     info    CA Endpoint istiod.istio-system.svc:15012, provider Citadel
2021-07-30T03:45:17.641990Z     info    Using CA istiod.istio-system.svc:15012 cert with certs: var/run/secrets/istio/root-cert.pem
2021-07-30T03:45:17.642176Z     info    citadelclient   Citadel client using custom root cert: istiod.istio-system.svc:15012
2021-07-30T03:45:17.672266Z     info    ads     All caches have been synced up in 34.33635ms, marking server ready
2021-07-30T03:45:17.672718Z     info    sds     SDS server for workload certificates started, listening on "./etc/istio/proxy/SDS"
2021-07-30T03:45:17.672869Z     info    xdsproxy        Initializing with upstream address "istiod.istio-system.svc:15012" and cluster "Kubernetes"
2021-07-30T03:45:17.673222Z     info    sds     Start SDS grpc server
2021-07-30T03:45:17.673407Z     info    Opening status port 15020
2021-07-30T03:45:17.678288Z     info    Starting proxy agent
2021-07-30T03:45:17.678312Z     info    Epoch 0 starting
2021-07-30T03:45:17.679646Z     info    Envoy command: [-c etc/istio/proxy/envoy-rev0.json --restart-epoch 0 --drain-time-s 45 --drain-strategy immediate --parent-shutdown-time-s 60 --service-cluster productpage.default --service-node sidecar~10.244.1.9~productpage-v1-5d9b4c9849-h8dvn.default~default.svc.cluster.local --local-address-ip-version v4 --bootstrap-version 3 --disable-hot-restart --log-format %Y-%m-%dT%T.%fZ      %l      envoy %n        %v -l warning --component-log-level misc:error --concurrency 2]
2021-07-30T03:45:17.798698Z     info    xdsproxy        connected to upstream XDS server: istiod.istio-system.svc:15012
2021-07-30T03:45:17.831655Z     info    ads     ADS: new connection for node:sidecar~10.244.1.9~productpage-v1-5d9b4c9849-h8dvn.default~default.svc.cluster.local-1
2021-07-30T03:45:17.832348Z     info    ads     ADS: new connection for node:sidecar~10.244.1.9~productpage-v1-5d9b4c9849-h8dvn.default~default.svc.cluster.local-2
2021-07-30T03:45:17.841421Z     info    cache   generated new workload certificate      latency=168.457775ms ttl=23h59m59.158589108s
2021-07-30T03:45:17.841450Z     info    cache   Root cert has changed, start rotating root cert
2021-07-30T03:45:17.841462Z     info    ads     XDS: Incremental Pushing:0 ConnectedEndpoints:2 Version:
2021-07-30T03:45:17.841499Z     info    cache   returned workload trust anchor from cache       ttl=23h59m59.158503284s
2021-07-30T03:45:17.841511Z     info    cache   returned workload trust anchor from cache       ttl=23h59m59.158490557s
2021-07-30T03:45:17.841728Z     info    sds     SDS: PUSH       resource=ROOTCA
2021-07-30T03:45:17.841837Z     info    cache   returned workload certificate from cache        ttl=23h59m59.158167965s
2021-07-30T03:45:17.841896Z     info    sds     SDS: PUSH       resource=default
2021-07-30T03:45:17.842023Z     info    cache   returned workload trust anchor from cache       ttl=23h59m59.157979952s
2021-07-30T03:45:17.842040Z     info    sds     SDS: PUSH       resource=ROOTCA
2021-07-30T03:45:19.575089Z     info    Initialization took 1.936605312s
2021-07-30T03:45:19.575111Z     info    Envoy proxy is ready
```



## References

