## 22.7 Хранение в K8s. Часть 2 - Вячеслав Закариев

### Цель задания

В тестовой среде Kubernetes нужно создать PV и продемострировать запись и хранение файлов.

### Чеклист готовности к домашнему заданию

1. Установленное K8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключенным GitHub-репозиторием.

### Дополнительные материалы для выполнения задания

1. [Инструкция по установке NFS в MicroK8S](https://microk8s.io/docs/nfs). 
2. [Описание Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/). 
3. [Описание динамического провижининга](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/). 
4. [Описание Multitool](https://github.com/wbitt/Network-MultiTool).

---

### Задание 1

**Что нужно сделать**

Создать Deployment приложения, использующего локальный PV, созданный вручную.

1. Создать Deployment приложения, состоящего из контейнеров busybox и multitool.
2. Создать PV и PVC для подключения папки на локальной ноде, которая будет использована в поде.
3. Продемонстрировать, что multitool может читать файл, в который busybox пишет каждые пять секунд в общей директории. 
4. Удалить Deployment и PVC. Продемонстрировать, что после этого произошло с PV. Пояснить, почему.
5. Продемонстрировать, что файл сохранился на локальном диске ноды. Удалить PV.  Продемонстрировать что произошло с файлом после удаления PV. Пояснить, почему.
5. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

---

### Задание 2

**Что нужно сделать**

Создать Deployment приложения, которое может хранить файлы на NFS с динамическим созданием PV.

1. Включить и настроить NFS-сервер на MicroK8S.
2. Создать Deployment приложения состоящего из multitool, и подключить к нему PV, созданный автоматически на сервере NFS.
3. Продемонстрировать возможность чтения и записи файла изнутри пода. 
4. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

---

### Правила приёма работы

1. Домашняя работа оформляется в Git-репозитории в файле README.md. На выполненное задание пришлите ссылку на .md-файл.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl`, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

---

### Решение 1

1. Написан манифест для [Deployment](https://github.com/SlavaZakariev/netology-kuber/blob/main/2.2/yaml/deployment.busybox.multitool.netology.yml)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-multitool-busybox
  namespace: netology-2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app-main
  template:
    metadata:
      labels:
        app: app-main
    spec:
      containers:
      - name: busybox
        image: busybox
        command: ['sh', '-c', 'while true; do echo "current date = $(date)" >> /busybox_data/date.log; sleep 5; done']
        volumeMounts:
          - name: app-vol-pvc
            mountPath: /busybox_data
      - name: multitool
        image: wbitt/network-multitool
        volumeMounts:
          - name: app-vol-pvc
            mountPath: /multitool_data
      volumes:
        - name: app-vol-pvc
          persistentVolumeClaim:
            claimName: pvc1-vol
```
2. Написаны манифесты для [PV](https://github.com/SlavaZakariev/netology-kuber/blob/main/2.2/yaml/pv1.netology.yml) и [PVC](https://github.com/SlavaZakariev/netology-kuber/blob/main/2.2/yaml/pvc1.netology.yml)

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv1
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data/pv1
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage

```
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc1-vol
  namespace: netology-2
spec:
  storageClassName: local-storage
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi

```

3. Запускаем PV и PVC

![pv-pvc](https://github.com/SlavaZakariev/netology-kuber/blob/f841399913e53d9a00c9232835e661636e83433f/2.2/resources/kub_2-7_1.1.jpg)

4. Запускаем Deployment

![dep](https://github.com/SlavaZakariev/netology-kuber/blob/f841399913e53d9a00c9232835e661636e83433f/2.2/resources/kub_2-7_1.2.jpg)

5. Проверяем загрузку и доступность данных в контейнере

![data](https://github.com/SlavaZakariev/netology-kuber/blob/f841399913e53d9a00c9232835e661636e83433f/2.2/resources/kub_2-7_1.3.jpg)

6. Удаляем Deployment и PV

![del](https://github.com/SlavaZakariev/netology-kuber/blob/f841399913e53d9a00c9232835e661636e83433f/2.2/resources/kub_2-7_1.4.jpg)

7. Проверяем наличие данных в хранилище

![data](https://github.com/SlavaZakariev/netology-kuber/blob/f841399913e53d9a00c9232835e661636e83433f/2.2/resources/kub_2-7_1.5.jpg)

8. Проверяем статус PVC

![PVC](https://github.com/SlavaZakariev/netology-kuber/blob/f841399913e53d9a00c9232835e661636e83433f/2.2/resources/kub_2-7_1.6.jpg)

При написании манифеста **PV** использовался режим ReclaimPolicy: Retain. Данный режим после удаления PV не удаляет ресурсы из внешних провайдеров автоматически. В случае удаления PV, файлы всё равно сохранятся по указанному пути.

---

### Решение 2

1. Установлен плагин **csidrivers**

![csi](https://github.com/SlavaZakariev/netology-kuber/blob/6f19c56bcfa96e1f8d337f39e9f24676df866716/2.2/resources/kub_2-7_2.1.jpg)

2. Написан манифест для [Deployment](https://github.com/SlavaZakariev/netology-kuber/blob/main/2.2/yaml/deployment.multitool.netology.yml)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: netology-app-multitool
  namespace: netology-2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app-main
  template:
    metadata:
      labels:
        app: app-main
    spec:
      containers:
      - name: multitool
        image: wbitt/network-multitool
        volumeMounts:
          - name: app-pvc-nfs
            mountPath: /multitool_data
      volumes:
        - name: app-pvc-nfs
          persistentVolumeClaim:
            claimName: app-pvc-nfs
```

3. Запущен Deployment, статус пода в Pending, так как нет созданного хранилища

![dep-pen](https://github.com/SlavaZakariev/netology-kuber/blob/b60b814e015d7cf1ede09d0cefdbaacbc325b6ab/2.2/resources/kub_2-7_2.2.jpg)

4. Написан манифест для [SC NFS](https://github.com/SlavaZakariev/netology-kuber/blob/main/2.2/yaml/sc.nfs.netology.yml)

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-csi
provisioner: nfs.csi.k8s.io
parameters:
  server: 172.27.169.38
  share: /srv/nfs
reclaimPolicy: Delete
volumeBindingMode: Immediate
mountOptions:
  - hard
  - nfsvers=4.1
```

5. Написан манифест для [PVC NFS](https://github.com/SlavaZakariev/netology-kuber/blob/main/2.2/yaml/pvc1.nfs.netology.yml)

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-pvc-nfs
  namespace: netology-2
spec:
  storageClassName: nfs-csi
  accessModes: [ReadWriteOnce]
  resources:
    requests:
      storage: 2Gi
```

6. Запускаем манифесты SC и PVC

![sc-pvc](https://github.com/SlavaZakariev/netology-kuber/blob/b60b814e015d7cf1ede09d0cefdbaacbc325b6ab/2.2/resources/kub_2-7_2.2.jpg)

7. Проверяем снова статус пода - сменился с Pending на Running

![sc-pvc](https://github.com/SlavaZakariev/netology-kuber/blob/b60b814e015d7cf1ede09d0cefdbaacbc325b6ab/2.2/resources/kub_2-7_2.3.jpg)



