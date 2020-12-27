## 使用 StatefulSet 部署 nginx

https://kubernetes.io/zh/docs/concepts/workloads/controllers/statefulset

statefulSet，有状态集。

当在同一个集合中，每个 pod 拥有自己的状态，可以使用。

statefulSet 必须配合 headless service 使用。

### Headless service

https://kubernetes.io/zh/docs/concepts/services-networking/service/#headless-services

通常的 service 拥有一个 cluster ip，充当对 pod 的负载均衡器，但是其他服务访问 service 的时候，不关心调用的是那个 pod，所以其他服务通过 service cluster ip 再访问到 pod。当需要关注与调用不同 pod 时候，就需要使用 headless service。

headless service 没有 cluster ip，而是通过把 dns 放在 pods 上。

- service

  ```yml
  apiVersion: v1
  kind: Service
  metadata:
    name: nginx
    labels:
      app: nginx
  spec:
    ports:
    - port: 80
      name: web
    clusterIP: None
    selector:
      app: nginx
  ```

- statefulSet

  ```yml
  apiVersion: apps/v1
  kind: StatefulSet
  metadata:
    name: web
  spec:
    selector:
      matchLabels:
        app: nginx # has to match .spec.template.metadata.labels
    serviceName: "nginx"
    replicas: 2 # by default is 1
    template:
      metadata:
        labels:
          app: nginx # has to match .spec.selector.matchLabels
      spec:
        containers:
        - name: nginx
          image: "nginx:1.19.6"
          ports:
          - containerPort: 80
            name: web
  ```

  