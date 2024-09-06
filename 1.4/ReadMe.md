## 22.4 Сетевое взаимодействие в K8S. Часть 1 - Вячеслав Закариев

### Цель задания

В тестовой среде Kubernetes необходимо обеспечить доступ к приложению, установленному в предыдущем ДЗ и состоящему из двух контейнеров, по разным портам в разные контейнеры как внутри кластера, так и снаружи.

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым Git-репозиторием.

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) Deployment и примеры манифестов.
2. [Описание](https://kubernetes.io/docs/concepts/services-networking/service/) Описание Service.
3. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

---

### Задание 1. Создать Deployment и обеспечить доступ к контейнерам приложения по разным портам из другого Pod внутри кластера

1. Создать Deployment приложения, состоящего из двух контейнеров (nginx и multitool), с количеством реплик 3 шт.
2. Создать Service, который обеспечит доступ внутри кластера до контейнеров приложения из п.1 по порту 9001 — nginx 80, по 9002 — multitool 8080.
3. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложения из п.1 по разным портам в разные контейнеры.
4. Продемонстрировать доступ с помощью `curl` по доменному имени сервиса.
5. Предоставить манифесты Deployment и Service в решении, а также скриншоты или вывод команды п.4.

---

### Задание 2. Создать Service и обеспечить доступ к приложениям снаружи кластера

1. Создать отдельный Service приложения из Задания 1 с возможностью доступа снаружи кластера к nginx, используя тип NodePort.
2. Продемонстрировать доступ с помощью браузера или `curl` с локального компьютера.
3. Предоставить манифест и Service в решении, а также скриншоты или вывод команды п.2.

---

### Правила приёма работы

1. Домашняя работа оформляется в Git-репозитории в файле README.md. На выполненное задание пришлите ссылку на .md-файл.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl` и скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

---

### Решение 1

1. Написан манифест для [Deployment](https://github.com/SlavaZakariev/netology-kuber/blob/main/1.4/yaml/deployment.netology.yml)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: netology-deployment
  namespace: netology
  labels:
    app: main
spec:
  replicas: 3
  selector:
    matchLabels:
      app: main
  template:
    metadata:
      labels:
        app: main
    spec:
      containers:
      - name: nginx
        image: nginx:1.25.5
      - name: multitool
        image: wbitt/network-multitool
        env:
          - name: HTTP_PORT
            value: "8080"
```

2. Созданы 3 пода

![dep](https://github.com/SlavaZakariev/netology-kuber/blob/5368ba469f74dfdbc007bf59ac8805cd569b52bd/1.4/resources/kub_2-4_1.1.jpg)

3. Написан манифест для [Service](https://github.com/SlavaZakariev/netology-kuber/blob/main/1.4/yaml/service.netology.yml)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: deployment-service
  namespace: netology
spec:
  ports:
    - name: http-app
      port:  9001
      protocol: TCP
      targetPort: 80
    - name: http-app-multi
      port:  9002
      protocol: TCP
      targetPort: 8080
  selector:
    app: main
```

4. Создан сервис

![svc](https://github.com/SlavaZakariev/netology-kuber/blob/5368ba469f74dfdbc007bf59ac8805cd569b52bd/1.4/resources/kub_2-4_1.2.jpg)

5. Написан манифест для [Pod](https://github.com/SlavaZakariev/netology-kuber/blob/main/1.4/yaml/pod.netology.yml)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-multitool
  namespace: netology
spec:
  containers:
    - image: wbitt/network-multitool
      name: app-multitool
```

6. Создан под app-multitool

![pod](https://github.com/SlavaZakariev/netology-kuber/blob/5368ba469f74dfdbc007bf59ac8805cd569b52bd/1.4/resources/kub_2-4_1.3.jpg)

7. Проверяем доступность по 80 порту 1-й и 2-й под.

![curl1](https://github.com/SlavaZakariev/netology-kuber/blob/5368ba469f74dfdbc007bf59ac8805cd569b52bd/1.4/resources/kub_2-4_1.4.jpg)

8. Проверяем доступность по 9001 и 9002 порту 3-й под.

![curl2](https://github.com/SlavaZakariev/netology-kuber/blob/5368ba469f74dfdbc007bf59ac8805cd569b52bd/1.4/resources/kub_2-4_1.5.jpg)

---

### Решение 2

1. Написан манифест для [Service Node]()

```yaml
apiVersion: v1
kind: Service
metadata:
  name: deployment-service-nodeport
  namespace: netology
spec:
  ports:
    - name: http-app
      port:  80
      nodePort: 30000
    - name: http-app-multi
      port:  8080
      nodePort: 30001
  selector:
    app: main
  type: NodePort
```

2. Создан сервис

![svc-node]()

3. Доступ через браузер с локального ПК

![browser]()
