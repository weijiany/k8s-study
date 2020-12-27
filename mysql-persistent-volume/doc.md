## 在 k8s 部署 mysql

- 创建 pv

  ```yml
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: mysql-pv
    labels:
      type: local
  spec:
    capacity:
      storage: 1Gi
    storageClassName: manual
    accessModes:
      - ReadWriteOnce
    persistentVolumeReclaimPolicy: Recycle # https://kubernetes.io/docs/concepts/storage/persistent-volumes/#reclaim-policy
    local:
      path: "/Users/weijian/code/infrastructure/mysql-pv"
    nodeAffinity:
      required:
        nodeSelectorTerms:
        - matchExpressions:
          - key: node-type
            operator: In
            values:
            - mysql
  ```

  类型：

  - local：在 node 上申请一块磁盘作为 k8s persistent volume，必须指定 `nodeAffinity` 节点亲和性

  - hostpath：在 node 上申请一块磁盘作为 k8s persistent volume

    __*区别*__：<font color=#FF0000>带验证</font>

    ​	由于每次重新部署 pod 会随机部署在不同 node 上。hostpath 会在部署的 node 上申请一块 pv，而 local 只能部署在 pv 所指定的 node 上。

    ​	应该也可以通过在 deployment 上指定 `nodeAffinity` 实现相同的功能。

- 创建pvc

  ```yml
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: mysql-pvc
  spec:
    storageClassName: manual
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 1Gi
  ```

  根据 storageClassName 选择 pv

- 创建 deployment

  ```yml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: mysql
  spec:
    selector:
      matchLabels:
        app: mysql
    strategy:
      type: Recreate
    template:
      metadata:
        labels:
          app: mysql
      spec:
        containers:
        - image: mysql:5.7
          name: mysql
          env:
          - name: MYSQL_ROOT_PASSWORD
            value: "123456"
          ports:
          - containerPort: 3306
            name: mysql
          volumeMounts:
          - name: mysql-persistent-storage
            mountPath: /var/lib/mysql
        volumes:
        - name: mysql-persistent-storage
          persistentVolumeClaim:
            claimName: mysql-pvc
  ```

  