# Client-go 结构

![](<../.gitbook/assets/image (48).png>)

## RESTClient

RESTClient 是所有客户端的父类，底层调用了Go语言 `net\http` 库，访问API Server的RESTful接口。

RESTClient的操作相对原始，使用样例如下：

```go
// 构建config对象，通常会存放在~/.kube/config的路径；如果运行在集群中，会有所不同
config, err := clientcmd.BuildConfigFromFlags("", clientcmd.RecommendedHomeFile)
// 封装error判断
mustSuccess(err)
config.APIPath = "api"
config.GroupVersion = &corev1.SchemeGroupVersion
config.NegotiatedSerializer = scheme.Codecs

restClient, err := rest.RESTClientFor(config)
mustSuccess(err)

result := &corev1.PodList{}
// 实际是在Do方法里调用了底层的net/http库向api-server发送request，最后将结果解析出放入result中
err = restClient.Get()
    .Namespace("sandbox")
    .Resource("pods")
    .VersionedParams(&metav1.ListOptions{Limit: 40}, scheme.ParameterCodec)
    .Do(context.TODO())
    .Into(result)
mustSuccess(err)

for _, d := range result.Items {
    fmt.Printf("NameSpace: %v \t Name: %v \t Status: %+v \n", d.Namespace, d.Name, d.Status.Phase)
}

```

## ClientSet

`ClientSet` 是使用最多的客户端，它继承自 `RESTClient`，使用K8s的代码生成机制(client-gen机制)，在编译过程中，会根据目前K8s内置的资源信息，自动生成他们的客户端代码(前提是需要添加适当的注解)，使用者可以通过builder pattern进行初始化，得到自己在意的目标资源类型的客户端。`ClientSet` 如同它的名字一样，代表的是一组内置资源的客户端。例如：

```go
clientset, err := kubernetes.NewForConfig(config) // 根据config对象创建clientSet对象
mustSuccess(err)

// 根据Pod资源的Group、Version、Recource Name创建资源定制客户端，传入的字符串表示资源所在的ns；
// podClient对象具有List\Update\Delete\Patch\Get等curd接口
podClient := clientset.CoreV1().Pods("development") 

```

## DynamiClient

`DynamiClient` 动态客户端，可以根据传入的 `GVR(group, version, resource)` 生成一个可以操作特定资源的客户端。但是不是内存安全的客户端，返回的结果通常是非结构化的。需要额外经过一次类型转换才能变为目标资源类型的对象，这一步存在内存安全的风险。相比 `ClientSet` ,动态客户端不局限于K8s的内置资源，可以用于处理 `CRD(custome resource define)` 自定义资源，但是缺点在于安全性不高。`DynamicClient` 使用的样例代码如下：

> 结构化的类型通常属于k8s runtime object的子类型；非结构化的对象通常是 `map[string]interface{}` 的形式，通过一个字典存储对象的属性；K8s所有的内置资源都可以通过代码生成机制，拥有默认的资源转换方法

```go
dynamicClient, err := dynamic.NewForConfig(config)
mustSuccess(err)

gvr := schema.GroupVersionResource{Version: "v1", Resource: "pods"}
// 返回非结构化的对象
unstructObj, err := dynamicClient.Resource(gvr).Namespace("sandbox").List(context.TODO(), metav1.ListOptions{Limit: 40})
mustSuccess(err)

podList := &corev1.PodList{}
// 额外做一次类型转换,如果这里传错类型，就会有类型安全的风险
err = runtime.DefaultUnstructuredConverter.FromUnstructured(unstructObj.UnstructuredContent(), podList)
mustSuccess(err)

for _, po := range podList.Items {
	fmt.Printf("NAMESPACE: %v \t NAME: %v \t STATUS: %v \n", po.Namespace, po.Name, po.Status)
}

```

## DiscoveryClient

`DiscoveryClient` 发现客户端，主要用于处理向服务端请求当前集群支持的资源信息，例如命令 `kubectl api-resources` 使用的就是发现客户端，由于发现客户端获取的数据量比较大，并且集群的资源信息变更并不频繁，因此发现客户端会在本地建立文件缓存，默认十分钟之内的请求，使用本地缓存，超过十分钟之后则重新请求服务端。`DiscoveryClient` 的使用样例代码如下：

```go
discoveryClient, err := discovery.NewDiscoveryClientForConfig(config)
mustSuccess(err)

_, APIResourceList, err := discoveryClient.ServerGroupsAndResources()
mustSuccess(err)

for _, list := range APIResourceList {
    gv, err := schema.ParseGroupVersion(list.GroupVersion)
    mustSuccess(err)

    for _, resource := range list.APIResources {
        fmt.Printf("name: %v \t group: %v \t verison: %v \n",
                   resource.Name, gv.Group, gv.Version)
    }
}
```

本地缓存路径：

![](<../.gitbook/assets/image (119).png>)

本地存储了`serverresources.json`文件，感兴趣的可以打开看下，是json格式化后的资源信息。

参考代码文件 `pkg/kubectl/cmd/apiresources/apiresources.go`，可以看到 `kubectl api-resources` 命令里确实使用了 `discoveryClient` :

```go
func (o *APIResourceOptions) RunAPIResources(cmd *cobra.Command, f cmdutil.Factory) error {
 ...
    // discoveryCilent
	discoveryclient, err := f.ToDiscoveryClient()
	if err != nil {
		return err
	}

    // 是否可以读本地缓存
	if !o.Cached {
		// Always request fresh data from the server
		discoveryclient.Invalidate()
	}

	errs := []error{}
    
	lists, err := discoveryclient.ServerPreferredResources()	
...
}
```

## 总结

| 客户端名称            | 源码目录                  | 简单描述                                                                                                                              |
| ---------------- | --------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| RESTClient       | client-go/rest/       | 基础客户端，对HTTP Request封装                                                                                                             |
| ClientSet        | client-go/kubernetes/ | 在RESTClient基础上封装了对Resource和Version，也就是说我们使用ClientSet的话是必须要知道Resource和Version， 例如AppsV1().Deployments或者CoreV1.Pods，缺点是不能访问CRD自定义资源 |
| DynamicClient    | client-go/dynamic/    | 包含一组动态的客户端，可以对任意的K8S API对象执行通用操作，包括CRD自定义资源                                                                                       |
| DiscoveryClient  | client-go/discovery/  | ClientSet必须要知道Resource和Version, 但使用者通常很难记住所有的GVR信息，这个DiscoveryClient是提供一个发现客户端，发现API Server支持的资源组，资源版本和资源信息                       |

## References

* [Programing In K8s 1：Client-go 实现分析与二次开发](https://blog.csdn.net/King_DJF/article/details/108307735)
