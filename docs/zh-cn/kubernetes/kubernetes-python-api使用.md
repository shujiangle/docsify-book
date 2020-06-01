# kubernetes python api使用

```
>>> from kubernetes import client,config

>>> config.load_kube_config()
>>> v1=client.CoreV1Api()
```

从API对象V1中，我们可以使用命名为spaced的列表列出所有正在运行的Pod
接收两个参数的pod方法，一个是命名空间，另一个是用于继续监视对象或立即进行一次检索
```
>>> ret = v1.list_namespaced_pod('default', watch=False)
```

master节点，命令查看pod信息
```
[root@k8s-master ~]# kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
volume-pod              2/2     Running   4          2d14h
volume2-pod             2/2     Running   4          2d14h
volume3-pod             2/2     Running   2          42h
web1-858cff4454-wpc8z   1/1     Running   1          39h
```

通过它使用python list comporehension 存储所有Pod名称
```
>>> pods = ret.items
>>> pods_names = [pod.metadata.name for pod in pods]
```
```
>>> pods_names
['volume-pod', 'volume2-pod', 'volume3-pod', 'web1-858cff4454-wpc8z']
```

```
>>> pods_status = [pod.status.phase for pod in pods]
>>> pods_status
['Running', 'Running', 'Running', 'Running']
```

将其捆绑在一起，并以更友好的输出显示
 ```
 list(zip(pods_names, pods_status))
```

```
>>> list(zip(pods_names, pods_status))
[('volume-pod', 'Running'), ('volume2-pod', 'Running'), ('volume3-pod', 'Running'), ('web1-858cff4454-wpc8z', 'Running')]
```

```
>>> from prettytable import PrettyTable
```

图表的方式展示
```
>>> from prettytable import PrettyTable
>>> t = PrettyTable(['Pod Name', 'Status'])
>>> for i in range(len(pods)):
...   t.add_row(list(zip(pods_names, pods_status))[i])
...
>>> print(t)
+-----------------------+---------+
|        Pod Name       |  Status |
+-----------------------+---------+
|       volume-pod      | Running |
|      volume2-pod      | Running |
|      volume3-pod      | Running |
| web1-858cff4454-wpc8z | Running |
+-----------------------+---------+
>>>
````
