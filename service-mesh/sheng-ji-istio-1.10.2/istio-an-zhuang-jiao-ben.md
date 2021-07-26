# istio 安装脚本

## install.sh

```text
curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.10.3 TARGET_ARCH=x86_64 sh -

cd istio-1.10.3
export PATH=$PWD/bin:$PATH

# install istio
istioctl install --set profile=demo -y
kubectl label namespace default istio-injection=enabled


# install bookinfo
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml

# check
kubectl get services

kubectl get pods
```



## Docker images

```text
istio/proxyv2    1.10.3   37f4e188f5b6        11 days ago         286MB
istio/pilot      1.10.3   5fd1e1693a85        11 days ago         221MB
```

