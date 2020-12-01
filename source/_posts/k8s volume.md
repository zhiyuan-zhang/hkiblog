---
title: k8s挂载问题
date: 2019/8/18
categories:
  - other
  - java
  - linux
  - db
  - 工具
typora-root-url: ../
typora-copy-images-to: ../static
abbrlink: 7
---





需求: k8s中的日志放到宿主机目录下

实现: k8s hostPath



详细请看
k8s volume存储卷
https://www.cnblogs.com/it-peng/p/11584439.html
Kubernetes进阶之hostpath及emptyDir数据卷
https://blog.51cto.com/14143894/2436199



```sh
拿到所有的pod
kubectl get pods

查看指定pod详情
kubectl describe pod iavatar-process-service-669d7f7f9-dhs82

进入pod bash
docker exec -it -u root bd04bf9d5f7dbb459cf88cbeb1088ceacd7ec4290ff613924be91297cc6b9036 /bin/bash
```





具体逻辑json

```json
{
  "kind": "Deployment",
  "apiVersion": "apps/v1beta1",
  "metadata": {
    "name": "iavatar-process-service",
    "namespace": "default"
  },
  "spec": {
    "replicas": 1,
    "selector": {
      "matchLabels": {
        "app": "iavatar-process-service"
      }
    },
    "template": {
        "metadata": {
        "creationTimestamp": null,
        "labels": {
          "app": "iavatar-process-service"
        }
      },
      "spec": {
        "containers": [{
            "name": "iavatar-process-service",
            "image": "iavatar.harbor.com/iavatar/isinonet:processservice",
            "resources": {},
            "imagePullPolicy": "Always",
            "volumeMounts": [
              {
                "mountPath": "/home/mars/ihunavi.log",
                "subPath": "ihunavi.log",
                "name": "biz-volume"
              }
            ],  
        "volumes": [
          {
            "name": "biz-volume",
            "hostPath": {
                "path": "/home/mars",
                "type": "DirectoryOrCreate"
                }
          }
        ],
        "restartPolicy": "Always",
        "terminationGracePeriodSeconds": 30
      }
    }
  }
}

```

