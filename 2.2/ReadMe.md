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

1. Написан манифест для [deployment]()

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
2. Написаны манифесты для [PV]() и [PVC]()

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

4. Запускаем deployment
