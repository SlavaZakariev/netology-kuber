## 22.2 Базовые объекты K8S - Вячеслав Закариев

### Цель задания

В тестовой среде для работы с Kubernetes, установленной в предыдущем ДЗ, необходимо развернуть Pod с приложением и подключиться к нему со своего локального компьютера. 

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключенным Git-репозиторием.

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. Описание [Pod](https://kubernetes.io/docs/concepts/workloads/pods/) и примеры манифестов.
2. Описание [Service](https://kubernetes.io/docs/concepts/services-networking/service/).

---

### Задание 1. Создать Pod с именем hello-world

1. Создать манифест (yaml-конфигурацию) Pod.
2. Использовать image - gcr.io/kubernetes-e2e-test-images/echoserver:2.2.
3. Подключиться локально к Pod с помощью `kubectl port-forward` и вывести значение (curl или в браузере).

---

### Задание 2. Создать Service и подключить его к Pod

1. Создать Pod с именем netology-web.
2. Использовать image — gcr.io/kubernetes-e2e-test-images/echoserver:2.2.
3. Создать Service с именем netology-svc и подключить к netology-web.
4. Подключиться локально к Service с помощью `kubectl port-forward` и вывести значение (curl или в браузере).

---

### Правила приёма работы

1. Домашняя работа оформляется в Git-репозитории в файле README.md. На выполненное задание пришлите ссылку на .md-файл.
2. Файл README.md должен содержать скриншоты вывода команд `kubectl get pods`, а также скриншот результата подключения.
3. Репозиторий должен содержать файлы манифестов и ссылки на них в файле README.md.

### Критерии оценки

Зачёт — выполнены все задания, ответы даны в развернутой форме, приложены соответствующие скриншоты и файлы проекта, в выполненных заданиях нет противоречий и нарушения логики.

На доработку — задание выполнено частично или не выполнено, в логике выполнения заданий есть противоречия, существенные недостатки.

---

### Решение 1

1. Написан манифест [pod.hello-world.yml](https://github.com/SlavaZakariev/netology-kuber/blob/main/1.2/yaml/pod.hello-world.yml)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello-world
spec:
  containers:
  - name: hello-world
    image: gcr.io/kubernetes-e2e-test-images/echoserver:2.2
    ports:
    - containerPort: 8080

```

2. Запускаем манифест с помощью команды `kubectl apply -f pod.hello-world.yml`

![pod](https://github.com/SlavaZakariev/netology-kuber/blob/e5b6850fe7cfc149ae95327fbe937bce215920dc/1.2/resorces/kub_2.1.jpg)

Проверяем статус созданного пода

![getpods](https://github.com/SlavaZakariev/netology-kuber/blob/8d398c239d04871833b3e2c76dd99391a49b1afc/1.2/resorces/kub_2.2.jpg)

3. Пробрасываем порт 8080 до пода

![port](https://github.com/SlavaZakariev/netology-kuber/blob/50f0bda95d88f1d33d4e07daea9e45d2905b9aa8/1.2/resorces/kub_2.3.jpg)

4. Проверям подключение через браузер на ПК

![curl](https://github.com/SlavaZakariev/netology-kuber/blob/50f0bda95d88f1d33d4e07daea9e45d2905b9aa8/1.2/resorces/kub_2.4.jpg)

---

### Решение 2

1. Написаны манифесты [pod.netology-web.yml](https://github.com/SlavaZakariev/netology-kuber/blob/main/1.2/yaml/pod.netology-web.yml) / [service.netology-web.yml](https://github.com/SlavaZakariev/netology-kuber/blob/main/1.2/yaml/service.netology-web.yml)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: netology-web
  labels:
    app: netology-web
spec:
  containers:
  - name: netology-web
    image: gcr.io/kubernetes-e2e-test-images/echoserver:2.2
```
```yaml
apiVersion: v1
kind: Service
metadata:
  name: netology-svc
spec:
  ports:
  - name: web
    protocol: TCP
    port: 8080
  selector:
    app: netology-web
```
