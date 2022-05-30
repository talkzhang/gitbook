# k8s相关命令

## kubectl cp 容器与宿主机之间复制

**使用前提**：Pod 中对应的 container 中安装了 tar 命令，该方式不用进入到 Pod 内

- 宿主机 –> Pod
```bash
kubectl cp /tmp/test_pod.txt default/mycentos-7b59b5b755-8rbgc:root
```

- Pod –> 宿主机
```bash
kubectl cp default/mycentos-7b59b5b755-8rbgc:/root/from_pod.txt  /tmp/from_pod.new
```

## 查看容器列表

kubectl get pods

## k8s容器和主机互相拷贝

1.kubectl cp /主机目录/文件路径 podName:/容器路径/xxx.datasource -n namespaces
这样可以把主机目录文件拷贝到容器内

2.kubectl cp podName:容器路径/xxx.datasource -n namespaces /主机目录
这样可以把容器内文件cp到主机目录

从容器拷贝文件到主机的时候podNname:这里不要加/ 之前加了/会一直报错