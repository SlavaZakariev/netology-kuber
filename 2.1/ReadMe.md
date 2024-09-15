## 22.6 Хранение в K8s. Часть 1 - Вячеслав Закариев

### Цель задания

В тестовой среде Kubernetes нужно обеспечить обмен файлами между контейнерам пода и доступ к логам ноды.

### Чеклист готовности к домашнему заданию

1. Установленное K8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключенным GitHub-репозиторием.

### Дополнительные материалы для выполнения задания

1. [Инструкция по установке MicroK8S](https://microk8s.io/docs/getting-started).
2. [Описание Volumes](https://kubernetes.io/docs/concepts/storage/volumes/).
3. [Описание Multitool](https://github.com/wbitt/Network-MultiTool).

---

### Задание 1 

Создать Deployment приложения, состоящего из двух контейнеров и обменивающихся данными.

1. Создать Deployment приложения, состоящего из контейнеров busybox и multitool.
2. Сделать так, чтобы busybox писал каждые пять секунд в некий файл в общей директории.
3. Обеспечить возможность чтения файла контейнером multitool.
4. Продемонстрировать, что multitool может читать файл, который периодоически обновляется.
5. Предоставить манифесты Deployment в решении, а также скриншоты или вывод команды из п. 4.

---

### Задание 2

Создать DaemonSet приложения, которое может прочитать логи ноды.

1. Создать DaemonSet приложения, состоящего из multitool.
2. Обеспечить возможность чтения файла `/var/log/syslog` кластера MicroK8S.
3. Продемонстрировать возможность чтения файла изнутри пода.
4. Предоставить манифесты Deployment, а также скриншоты или вывод команды из п. 2.

---

### Правила приёма работы

1. Домашняя работа оформляется в Git-репозитории в файле README.md. На выполненное задание пришлите ссылку на .md-файл.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl`, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

---

### Решение 1

1. Написан манифест для [Deployment](https://github.com/SlavaZakariev/netology-kuber/blob/42bd25a7a2b75f297bc4c38ebdea3b81575ae855/2.1/yaml/daemonset.multitool.netology.yml)

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
        command: ['sh', '-c', 'while true; do echo "current date = $(date)" >> /busybox_data/date.log; sleep 10; done']
        volumeMounts:
          - mountPath: "/busybox_data"
            name: deployment-volume
      - name: multitool
        image: wbitt/network-multitool
        volumeMounts:
          - name: deployment-volume
            mountPath: "/multitool_data"
      volumes:
        - name: deployment-volume
          emptyDir: {}
```

2. Развёрнуто пространство имён netology-2

![ns](https://github.com/SlavaZakariev/netology-kuber/blob/42bd25a7a2b75f297bc4c38ebdea3b81575ae855/2.1/resources/kub_2-6_1.1.jpg)

2. Развёрнут Deployment

![depl](https://github.com/SlavaZakariev/netology-kuber/blob/42bd25a7a2b75f297bc4c38ebdea3b81575ae855/2.1/resources/kub_2-6_1.2.jpg)

3. Проверяем наличии тома в контейнерах

![vol](https://github.com/SlavaZakariev/netology-kuber/blob/42bd25a7a2b75f297bc4c38ebdea3b81575ae855/2.1/resources/kub_2-6_1.3.jpg)

4. Проверяем доступность данных из тома для busybox

![vol](https://github.com/SlavaZakariev/netology-kuber/blob/42bd25a7a2b75f297bc4c38ebdea3b81575ae855/2.1/resources/kub_2-6_1.4.jpg)

5. Проверяем доступность данных из тома для multitool

![vol](https://github.com/SlavaZakariev/netology-kuber/blob/42bd25a7a2b75f297bc4c38ebdea3b81575ae855/2.1/resources/kub_2-6_1.5.jpg)
