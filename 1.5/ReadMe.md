## 22.5 Сетевое взаимодействие в K8S. Часть 2 - Вячеслав Закариев

### Цель задания

В тестовой среде Kubernetes необходимо обеспечить доступ к двум приложениям снаружи кластера по разным путям.

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым Git-репозиторием.

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Инструкция](https://microk8s.io/docs/getting-started) по установке MicroK8S.
2. [Описание](https://kubernetes.io/docs/concepts/services-networking/service/) Service.
3. [Описание](https://kubernetes.io/docs/concepts/services-networking/ingress/) Ingress.
4. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

---

### Задание 1. Создать Deployment приложений backend и frontend

1. Создать Deployment приложения _frontend_ из образа nginx с количеством реплик 3 шт.
2. Создать Deployment приложения _backend_ из образа multitool. 
3. Добавить Service, которые обеспечат доступ к обоим приложениям внутри кластера. 
4. Продемонстрировать, что приложения видят друг друга с помощью Service.
5. Предоставить манифесты Deployment и Service в решении, а также скриншоты или вывод команды п.4.

---

### Задание 2. Создать Ingress и обеспечить доступ к приложениям снаружи кластера

1. Включить Ingress-controller в MicroK8S.
2. Создать Ingress, обеспечивающий доступ снаружи по IP-адресу кластера MicroK8S так, чтобы при запросе только по адресу открывался _frontend_ а при добавлении /api - _backend_.
3. Продемонстрировать доступ с помощью браузера или `curl` с локального компьютера.
4. Предоставить манифесты и скриншоты или вывод команды п.2.

---

### Правила приема работы

1. Домашняя работа оформляется в Git-репозитории в файле README.md. На выполненное задание пришлите ссылку на .md-файл.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl` и скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

---

### Решение 1

1. Написан манифест для [frontend](https://github.com/SlavaZakariev/netology-kuber/blob/d66eeb77242f8c041345678d71227b2a8f16bb94/1.5/yaml/deployment.front.netology.yml)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: netology
spec:
  replicas: 3
  selector:
    matchLabels:
      app: main-front
  template:
    metadata:
      labels:
        app: main-front
    spec:
      containers:
        - image: nginx:1.25.5
          name: nginx

---

apiVersion: v1
kind: Service
metadata:
  name: svc-frontend
  namespace: netology
spec:
  ports:
    - name: main-front
      port: 80
  selector:
    app: main-front
```

2. Написан манифест для [backend](https://github.com/SlavaZakariev/netology-kuber/blob/main/1.5/yaml/deployment.back.netology.yml)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: netology
spec:
  replicas: 1
  selector:
    matchLabels:
      app: main-back
  template:
    metadata:
      labels:
        app: main-back
    spec:
      containers:
        - image: wbitt/network-multitool
          name: multitool

---

apiVersion: v1
kind: Service
metadata:
  name: svc-backend
  namespace: netology
spec:
  ports:
    - name: main-back
      port: 80
  selector:
    app: main-back
```

4. Запускаем манифесты

![pods](https://github.com/SlavaZakariev/netology-kuber/blob/5d41b7473a12cf1cba55045bad3d6b37482330b0/1.5/resources/kub_2-5_1.1.jpg)

5. Проверка с помощью curl:
   - c пода front до IP сервиса backend
   - c пода back до IP сервиса frontend

![curl](https://github.com/SlavaZakariev/netology-kuber/blob/5d41b7473a12cf1cba55045bad3d6b37482330b0/1.5/resources/kub_2-5_1.2.jpg)

---

### Решение 2

1. Включен Ingress на Microk8s
 
![enable](https://github.com/SlavaZakariev/netology-kuber/blob/2b1da5d8d5b5e13d1ef57d0db332295c83f17fbf/1.5/resources/kub_2-5_2.1.jpg)

2. Проверяем статус после старта функции Ingress

![status](https://github.com/SlavaZakariev/netology-kuber/blob/2b1da5d8d5b5e13d1ef57d0db332295c83f17fbf/1.5/resources/kub_2-5_2.2.jpg)

3. Написан манифест [Ingress](https://github.com/SlavaZakariev/netology-kuber/blob/main/1.5/yaml/ingress.netology.yml)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-1.5
  namespace: netology
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host:
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: svc-frontend
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: svc-backend
            port:
              number: 80
```

4. Запускаем манифест Ingress

![apply-ing](https://github.com/SlavaZakariev/netology-kuber/blob/2b1da5d8d5b5e13d1ef57d0db332295c83f17fbf/1.5/resources/kub_2-5_2.3.jpg)

5. Проверка через браузер front

![curl-front](https://github.com/SlavaZakariev/netology-kuber/blob/2b1da5d8d5b5e13d1ef57d0db332295c83f17fbf/1.5/resources/kub_2-5_2.4.jpg)

6. Проверка через браузер back (/api)

![curl-back](https://github.com/SlavaZakariev/netology-kuber/blob/2b1da5d8d5b5e13d1ef57d0db332295c83f17fbf/1.5/resources/kub_2-5_2.5.jpg)
