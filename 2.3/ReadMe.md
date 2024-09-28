## 22.8 Конфигурация приложений - Вячеслав Закариев

### Цель задания

В тестовой среде Kubernetes необходимо создать конфигурацию и продемонстрировать работу приложения.

### Чеклист готовности к домашнему заданию

1. Установленное K8s-решение (например, MicroK8s).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым GitHub-репозиторием.

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/concepts/configuration/secret/) Secret.
2. [Описание](https://kubernetes.io/docs/concepts/configuration/configmap/) ConfigMap.
3. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

---

### Задание 1. Создать Deployment приложения и решить возникшую проблему с помощью ConfigMap. Добавить веб-страницу

1. Создать Deployment приложения, состоящего из контейнеров nginx и multitool.
2. Решить возникшую проблему с помощью ConfigMap.
3. Продемонстрировать, что pod стартовал и оба контейнера работают.
4. Сделать веб-страницу и подключить её к Nginx с помощью ConfigMap. Подключить Service и показать вывод curl или в браузере.
5. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

---

### Задание 2. Создать приложение с вашей веб-страницей, доступной по HTTPS 

1. Создать Deployment приложения, состоящего из Nginx.
2. Создать собственную веб-страницу и подключить её как ConfigMap к приложению.
3. Выпустить самоподписной сертификат SSL. Создать Secret для использования сертификата.
4. Создать Ingress и необходимый Service, подключить к нему SSL в вид. Продемонстировать доступ к приложению по HTTPS. 
4. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

---

### Правила приёма работы

1. Домашняя работа оформляется в Git-репозитории в файле README.md. На выполненное задание пришлите ссылку на .md-файл.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl`, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

---

### Решение 1

1. Создадим манифест для [ConfigMap](https://github.com/SlavaZakariev/netology-kuber/blob/main/2.3/yaml/configmap.netology.yml) для начала, чтобы 2-й контейнер multitool выходил по альтернативному порту

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: configmap-nginx-multitool
  namespace: netology-2
data:
  HTTP_PORT: "8080"
  index.html: |
    <html>
    <h1>Welcome</h1>
    </br>
    <h1>This is Netology Homework - 22.8 Конфигурация приложений</h1>
    </html>
```

2. Запустим манифест **ConfigMap**

![configmap](https://github.com/SlavaZakariev/netology-kuber/blob/494a0301ebc6e4f2d1db58c270d988763a3803c5/2.3/resources/kub_2-8_1.1.jpg)

3. Написан манифест [Deployment](https://github.com/SlavaZakariev/netology-kuber/blob/main/2.3/yaml/deployment.nginx.multitool.netology.yml) для Nginx и Multitool. Привяжем к страницу из ConfigMap для nginx и порт для multitool

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-nginx-multitool
  namespace: netology-2
  labels:
    app: app-main
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
      - name: nginx
        image: nginx:1.25.5
        volumeMounts:
          - name: nginx-index-file
            mountPath: /usr/share/nginx/html/
      - name: multitool
        image: wbitt/network-multitool
        env:
        - name: HTTP_PORT
          valueFrom:
            configMapKeyRef:
              name: configmap-nginx-multitool
              key: HTTP_PORT

      volumes:
        - name: nginx-index-file
          configMap:
            name: configmap-nginx-multitool
```

4. Запустим манифест **Deployment**

![deploy](https://github.com/SlavaZakariev/netology-kuber/blob/494a0301ebc6e4f2d1db58c270d988763a3803c5/2.3/resources/kub_2-8_1.2.jpg)

5. Написан манифест [Service NodePort](https://github.com/SlavaZakariev/netology-kuber/blob/main/2.3/yaml/service.netology.yml), чтобы контейнеры были доступны вне сети кубера

```yaml
apiVersion: v1
kind: Service
metadata:
  name: service-deployment
  namespace: netology-2
spec:
  type: NodePort
  ports:
    - name: http-nginx
      port:  80
      nodePort: 30000
      protocol: TCP
      targetPort: 80
    - name: http-app-multitool
      port:  8080
      nodePort: 30001
      protocol: TCP
      targetPort: 8080
  selector:
    app: app-main
```

5. Запустим манифест **Service**

![svc](https://github.com/SlavaZakariev/netology-kuber/blob/494a0301ebc6e4f2d1db58c270d988763a3803c5/2.3/resources/kub_2-8_1.3.jpg)

6. Проверям через браузер nginx через заданный в svc порт 30000

![nginx](https://github.com/SlavaZakariev/netology-kuber/blob/494a0301ebc6e4f2d1db58c270d988763a3803c5/2.3/resources/kub_2-8_1.4.jpg)

7. Проверям через браузер multitool через заданный в svc порт 30001

![multitool](https://github.com/SlavaZakariev/netology-kuber/blob/494a0301ebc6e4f2d1db58c270d988763a3803c5/2.3/resources/kub_2-8_1.5.jpg)
