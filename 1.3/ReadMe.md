## 22.3 Запуск приложений в k8s - Вячеслав Закариев

### Цель задания

В тестовой среде для работы с Kubernetes, установленной в предыдущем ДЗ, необходимо развернуть Deployment с приложением, состоящим из нескольких контейнеров, и масштабировать его.

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым git-репозиторием.

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) Deployment и примеры манифестов.
2. [Описание](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/) Init-контейнеров.
3. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

---

### Задание 1. Создать Deployment и обеспечить доступ к репликам приложения из другого Pod

1. Создать Deployment приложения, состоящего из двух контейнеров — nginx и multitool. Решить возникшую ошибку.
2. После запуска увеличить количество реплик работающего приложения до 2.
3. Продемонстрировать количество подов до и после масштабирования.
4. Создать Service, который обеспечит доступ до реплик приложений из п.1.
5. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложений из п.1.

---

### Задание 2. Создать Deployment и обеспечить старт основного контейнера при выполнении условий

1. Создать Deployment приложения nginx и обеспечить старт контейнера только после запуска сервиса этого приложения.
2. Убедиться, что nginx не стартует. В качестве Init-контейнера взять busybox.
3. Создать и запустить Service. Убедиться, что Init запустился.
4. Продемонстрировать состояние пода до и после запуска сервиса.

---

### Правила приема работы

1. Домашняя работа оформляется в Git-репозитории в файле README.md. На выполненное задание пришлите ссылку на .md-файл.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl` и скриншоты результатов.
3. Репозиторий должен содержать файлы манифестов и ссылки на них в файле README.md.

---

### Решение 1

1. Создано отдельное пространство имён **namespace netology**

![ns](https://github.com/SlavaZakariev/netology-kuber/blob/70aef244e5d2a708d340a8a65e4d2c0a471e0dbb/1.3/resources/kub_2-3_1.1.jpg)

2. Написан манифест для **Deployment**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: netology-deployment
  labels:
    app: main
spec:
  replicas: 1
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
```

3. Запушен манифест в пространстве имён **netology**

![ns](https://github.com/SlavaZakariev/netology-kuber/blob/70aef244e5d2a708d340a8a65e4d2c0a471e0dbb/1.3/resources/kub_2-3_1.2.jpg)

4. Проверяем статус, один под в ошибке

![error](https://github.com/SlavaZakariev/netology-kuber/blob/70aef244e5d2a708d340a8a65e4d2c0a471e0dbb/1.3/resources/kub_2-3_1.3.jpg)

5. Смотрим логи и исправляем манифест, добавив альтернативный порт для **multitool**, так как по умолчанию оба стучатся на порт 80

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: netology-deployment
  labels:
    app: main
spec:
  replicas: 1
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

6. Перезапускаем манифест и проверяем статус, ошибка более не воспроизводится.

![fix](https://github.com/SlavaZakariev/netology-kuber/blob/70aef244e5d2a708d340a8a65e4d2c0a471e0dbb/1.3/resources/kub_2-3_1.5.jpg)

7. Увеличиваем количество реплик до 2-х, добавив в манифесте [Deployment](https://github.com/SlavaZakariev/netology-kuber/blob/main/1.3/yaml/deployment.netology.yml) цифру 2 в поле **replicas**

8. Проверяем статус подов

![status](https://github.com/SlavaZakariev/netology-kuber/blob/70aef244e5d2a708d340a8a65e4d2c0a471e0dbb/1.3/resources/kub_2-3_1.6.jpg)

9. Написан манифест для [Service](https://github.com/SlavaZakariev/netology-kuber/blob/main/1.3/yaml/service.netology.yml)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: deployment-service
spec:
  selector:
    app: main
  ports:
    - name: http-nginx
      port:  80
      protocol: TCP
      targetPort: 80
    - name: http-multitool
      port:  8080
      protocol: TCP
      targetPort: 8080
```

10. Запущен манифест в пространстве имён **netology**, проверяем статус

![svc](https://github.com/SlavaZakariev/netology-kuber/blob/666b52e4456ec2ac63097cf57358fdf56f6dd891/1.3/resources/kub_2-3_1.7.jpg)

11. Написан манифест для приложения [multitool](https://github.com/SlavaZakariev/netology-kuber/blob/main/1.3/yaml/pod.multitool.yml), в манифесте заранее указано пространство имён и порт

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multitool
  namespace: netology
spec:
  containers:
  - name: multitool
    image: wbitt/network-multitool
    ports:
    - containerPort: 8080
```

12. Запускаем манифест для **multitool**

![multitool](https://github.com/SlavaZakariev/netology-kuber/blob/666b52e4456ec2ac63097cf57358fdf56f6dd891/1.3/resources/kub_2-3_1.8.jpg)

13. Проверяем с помощью **curl** ответ от пода **multitool**

![curl](https://github.com/SlavaZakariev/netology-kuber/blob/666b52e4456ec2ac63097cf57358fdf56f6dd891/1.3/resources/kub_2-3_1.9.jpg)

---

### Решение 2

1. Написан манифест [Deployment](https://github.com/SlavaZakariev/netology-kuber/blob/main/1.3/yaml/deployment.init.yml) для приложения nginx

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-init
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web-init
  template:
    metadata:
      labels:
        app: web-init
    spec:
      containers:
      - name: nginx
        image: nginx:1.25.5
      initContainers:
      - name: busybox-init
        image: busybox
        command: ['sh', '-c', "until nslookup service-init.\
        $(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace)\
        .svc.cluster.local; do echo waiting for svc-nginx-init; sleep 2; done"]
```

2. Запушен манифест в пространстве имён **netology**, проверяем статус

![nginx](https://github.com/SlavaZakariev/netology-kuber/blob/c32164b11eb3137d3973135a2ea44680807ba399/1.3/resources/kub_2-3_2.1.jpg)

3. Написан манифест [Service](https://github.com/SlavaZakariev/netology-kuber/blob/main/1.3/yaml/service.init.yml)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: service-init
spec:
  ports:
    - name: web
      port: 80
  selector:
    app: web-init
```

4. Запушен манифест в пространстве имён **netology**, проверяем статус и наличие вновь созданного сервиса

![service-init](https://github.com/SlavaZakariev/netology-kuber/blob/c32164b11eb3137d3973135a2ea44680807ba399/1.3/resources/kub_2-3_2.2.jpg)

