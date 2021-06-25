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