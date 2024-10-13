## 22.10 «Helm» - Вячеслав Закариев

### Цель задания

В тестовой среде Kubernetes необходимо установить и обновить приложения с помощью Helm.

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение, например, MicroK8S.
2. Установленный локальный kubectl.
3. Установленный локальный Helm.
4. Редактор YAML-файлов с подключенным репозиторием GitHub.

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Инструкция](https://helm.sh/docs/intro/install/) по установке Helm. [Helm completion](https://helm.sh/docs/helm/helm_completion/).

---

### Задание 1. Подготовить Helm-чарт для приложения

1. Необходимо упаковать приложение в чарт для деплоя в разные окружения. 
2. Каждый компонент приложения деплоится отдельным deployment’ом или statefulset’ом.
3. В переменных чарта измените образ приложения для изменения версии.

---

### Задание 2. Запустить две версии в разных неймспейсах

1. Подготовив чарт, необходимо его проверить. Запуститe несколько копий приложения.
2. Одну версию в namespace=app1, вторую версию в том же неймспейсе, третью версию в namespace=app2.
3. Продемонстрируйте результат.

---

### Правила приёма работы

1. Домашняя работа оформляется в Git-репозитории в файле README.md. На выполненное задание пришлите ссылку на .md-файл.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl`, `helm`, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

---

### Решение 1

1. Скачиваем и устанавливаем Helm

![helm](https://github.com/SlavaZakariev/netology-kuber/blob/e0756dfb73b82bd4bc3d8a81d2a989b0966cdfff/2.5/resources/kub_2-10_1.1.jpg)

2. Подготавливаем манифесты
   - Chart.yaml
   - values.yaml
   - templates\deployment.netology.yml
   - templates\service.netology.yml

```yaml
apiVersion: v2
name: netology

type: application

version: 1.0.0
appVersion: 1.25.5
```
```yaml
replicaCount: 1
image: nginx
port: 80
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mainApp
  labels:
    app: main
spec:
  replicas: {{ .Values.replicaCount }}
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
          image: "{{ .Values.image }}:{{ .Chart.appVersion }}"
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: {{ .Values.port }}
              protocol: TCP
```
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mainApp
  labels:
    app: main
spec:
  ports:
    - port: {{ .Values.port }}
      name: http
  selector:
    app: main
```

3. Создаём [helm-nginx](https://github.com/SlavaZakariev/netology-kuber/tree/main/2.5/helm-nginx)

![create](https://github.com/SlavaZakariev/netology-kuber/blob/e0756dfb73b82bd4bc3d8a81d2a989b0966cdfff/2.5/resources/kub_2-10_1.2.jpg)

---

### Решение 2

1. Устанавливаем **helm chart** в под названием **netology-01**

![n-01](https://github.com/SlavaZakariev/netology-kuber/blob/e0756dfb73b82bd4bc3d8a81d2a989b0966cdfff/2.5/resources/kub_2-10_2.1.jpg)

2. Увеличиваем до 2-х реплик **chart** **netology-01**

![upgrade-01](https://github.com/SlavaZakariev/netology-kuber/blob/e0756dfb73b82bd4bc3d8a81d2a989b0966cdfff/2.5/resources/kub_2-10_2.2.jpg)

3. Удаляем объект **netology-01**

![uninstall-01](https://github.com/SlavaZakariev/netology-kuber/blob/d84a21fd05b82010033bdc06e243a3bf1a49ad48/2.5/resources/kub_2-10_2.3.jpg)
