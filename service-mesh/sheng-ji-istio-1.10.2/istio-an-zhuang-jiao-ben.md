# istio 安装脚本

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

