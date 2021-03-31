# Deployment.spec.selector.matchLables 实验解释

**作者: 张首富 时间: 2019-02-23**

## 正确配置

正确的 `Deployment` , 让 `matchLabels` 和 `template.metadata.lables` 完全比配不报错 

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      app: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
```

pod创建成功

```text
[root@rke test_yaml]# kubectl get pods
NAME                      READY   STATUS    RESTARTS   AGE
my-nginx-9b44d8f5-d6n8z   1/1     Running   0          3s
my-nginx-9b44d8f5-zzv52   1/1     Running   0          3s
```

## 错误配置

### 缺少 spec.selector

直接不写spec.mathlabels创建直接报错缺少缺少必要字段

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
```

运行报错结果如下:

```text
[root@rke test_yaml]# kubectl create -f test_pod_svc.yaml
error: error validating "test_pod_svc.yaml": error validating data: ValidationError(Deployment.spec): missing required field "selector" in io.k8s.api.apps.v1.DeploymentSpec; if you choose to ignore these errors, turn validation off with --validate=false
```

### 标签不匹配

spec.selector.matchLables 和 pod模板不相对应会直接报错

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      app: my-nginx-add
  replicas: 2
  template:
    metadata:
      labels:
        app: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx:1.14
        ports:
        - containerPort: 80
```

运行报错结果如下:

```bash
The Deployment "my-nginx" is invalid: spec.template.metadata.labels: Invalid value: map[string]string{"app":"my-nginx"}: `selector` does not match template `labels`
```

查看帮助手册

```text
kubectl explain Deployment.spec

    selector  <Object>
     Label selector for pods. Existing ReplicaSets whose pods are selected by
     this will be the ones affected by this deployment.
```

## 总结

* 在 Deployment 中 matchLabels 是必填字段
* template.metadata.labels 必填，因为 `Deployment.spec.selector`是必须字段,而他又必须和 template.labels 对应
* template.metadata.labels 会应用到下面所有的副本集里,在 `template.spec.containers` 里面不能定义labels标签.

## References

[原文 Deployment.spec.selector.matchLables实验解释](https://cloud.tencent.com/developer/article/1394657) , 张首富

