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

1. Написан манифест для [ConfigMap](https://github.com/SlavaZakariev/netology-kuber/blob/main/2.3/yaml/configmap.netology.yml) для начала, чтобы 2-й контейнер multitool выходил по альтернативному порту

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

6. Проверям через браузер nginx через заданный в svc порт 30000 (кириллицу не разпознало)

![nginx](https://github.com/SlavaZakariev/netology-kuber/blob/494a0301ebc6e4f2d1db58c270d988763a3803c5/2.3/resources/kub_2-8_1.4.jpg)

7. Проверям через браузер multitool через заданный в svc порт 30001

![multitool](https://github.com/SlavaZakariev/netology-kuber/blob/494a0301ebc6e4f2d1db58c270d988763a3803c5/2.3/resources/kub_2-8_1.5.jpg)

---

### Решение 2

1. Написан манифест для [Deployment](https://github.com/SlavaZakariev/netology-kuber/blob/main/2.3/yaml/deployment.nginx.netology.yml) для Nginx

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-nginx
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

      volumes:
        - name: nginx-index-file
          configMap:
            name: configmap-nginx
```
2. Написан манифест [ConfigMap](https://github.com/SlavaZakariev/netology-kuber/blob/2440b45256d797386fd4ccc52fc562109af4b3a8/2.3/yaml/configmap.nginx.netology.yml) для Nginx.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: configmap-nginx
  namespace: netology-2
data:
  index.html: |
    <html>
    <h1>Welcome</h1>
    </br>
    <h1>This is Netology Homework - The task №2</h1>
    </html>
```

3. Создаём самоподписные сертификаты **tls.crt** и **tls.key** для домена **app-nginx.ru** на 1 год

![cert](https://github.com/SlavaZakariev/netology-kuber/blob/eebb47d07df7e50e362b7c1ca1362e622fcc52b5/2.3/resources/kub_2-8_2.1.jpg)

4. Создаём **secret** и проверяем статус созданного объёкта в пространстве имён **netology-2**

![secret](https://github.com/SlavaZakariev/netology-kuber/blob/eebb47d07df7e50e362b7c1ca1362e622fcc52b5/2.3/resources/kub_2-8_2.2.jpg)

5. Проверяем secret в формате **yaml** для проверки наличия ключей и даты создания

```yaml
apiVersion: v1
items:
- apiVersion: v1
  data:
    tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUZEekNDQXZlZ0F3SUJBZ0lVV3NjUXZRSE9ralRVZm0rK2oxaHpaUDBaUnQ0d0RRWUpLb1pJaHZjTkFRRUwKQlFBd0Z6RVZNQk1HQTFVRUF3d01ZWEJ3TFc1bmFXNTRMbkoxTUI0WERUSTBNRGt5T0RFeU1qazBNbG9YRFRJMQpNRGt5T0RFeU1qazBNbG93RnpFVk1CTUdBMVVFQXd3TVlYQndMVzVuYVc1NExuSjFNSUlDSWpBTkJna3Foa2lHCjl3MEJBUUVGQUFPQ0FnOEFNSUlDQ2dLQ0FnRUFxS29XSUtvUWp2WWQwaEovVVZXby9ZVU5aMVRBYTIwUzl0M2EKUTRWRnhXOWlReWVoeEtnc1FHelZSZ3lVRXBienp4OVdjUndzTWlhelNNMkFOZno3S0llMExPNmxDSGJ0NWxESAovbVFuMVVOVXd4MHdPU1I4dGR4ODExd0VrQzIzYjdOYlM0c0p4TktoNkgwK0YxajFmcUVGUDE3SUZaaHRld0R6Ci9hcnBrdTYwK25wWW1JeWUrWUtDNVlmb1BFbGlNQldtZGxSU3F6Y2xQK1gwbHRuck9DQUJtamhxeWJ6K2ZOZEYKWVA0VTc5eTBoK010aVBHT1hkZHVsQVo4N0JDSXA4UkZlS3F1QW52U3R5dlRXdzZjaWdHLysxQkgyVTRKZ211ZQpFTVpWWDVTOEMrOWoyS3g1ck5rNlpSLytENVVNelU0b01ZdE52M1NrU0ppZCt2OUU0dzA5MVl4UjVSLzRTa1d0CkQ3ek91cVk2SmJnamV6WEduTXBvZGU2QU1Hb2NaQWVjYStVc21wOHY3NnIvWGFEOWl3OXUvcndBT1RkNjQvU24KSFg4ZE0rV1EybFZ3ZkJiemlYRzFEODMvS1lqM0JEZUlOeEFnL2JZcVlBK0Q5Qk41S01pZUJtV2xIZFE3VkxoSAp4UEJVR0huUlB5cGJSR2o4MkljZGJtNjZWeFdJOHVvejdXd1gxN1F5cG9NQVdwNVdiaWRuT3ZkTTZRdmtyNDVmClg0RjRIcHIvcEJ6MzRnaURjZGVsTldBRTV4VWZkbmdvMXRrYlIrd1p6MHpyNG9DQThia053TTBUcHRaUHI4dTEKNmk2dkpPQlowaEt4WmR2YVFEN1ZQYkVLcmFRb2VWRkJERVMvZTdEcENCK3ZXd2YwdE1pWWhSTTRvVWxYWkRKZgpkODZlQ2tVQ0F3RUFBYU5UTUZFd0hRWURWUjBPQkJZRUZGNDNmRXQ3RTZTb213L3E1em83NHV6c0paY21NQjhHCkExVWRJd1FZTUJhQUZGNDNmRXQ3RTZTb213L3E1em83NHV6c0paY21NQThHQTFVZEV3RUIvd1FGTUFNQkFmOHcKRFFZSktvWklodmNOQVFFTEJRQURnZ0lCQUNKbjBPUjh3S3EyeU5JT2RZcU94SXZMdkFhSTd1MXdkeitkb3QxagozMmszb1ZwYlZ0YzZRM0Q0SElDVDZ2Mm5STXJZU0JiTTRJQUpiTVB0bGM5M2dzeFkzZllaK3hZSXF4ekdScGlnCkhWSjIrUG5XVUJ6S3FjZ0JCc3YxdmVHaVVNaG5ycXBNYjZpakQvT2pJdy9GdktBd0h5enlPQkNMNkk4bHU4MkYKbXlxSjV4VlFrS2JuVnlOU1Q3Z3doZi9HNmJ0VUo2Lzl6MjJNM1kzV1pSMnE5WnloM2tEYmhNNVBqUFY4V1V3QgpsTnlibG5iL3Z6RldnOU5KYjNuS3FCNEpmSFBKWXBURmFXcUdvd1hHWW0yZG1Ed0tLQlBCdGdNaERTSG5WU2pjCk9pUmV1YWd4QVVVQ0RwaEhzMEtBcVBmUTk4R0pwOWVuU1d2b3UvVnYvSjNSckNJUitGOGd5QkRjRFZnVUtEZnAKdXdrYlM4QmEvOWZaenVQYmpsemxEOFNpQkFWclJCRzVDd2NnYVdROU9pOWU3NlQvZTVmMUsyeEhaaE9IZHNKZgp6QTcvdUtyNzE2NzRXL2lZb0c2YmVsb1JJT0x0cXQrNTFFWlc0Qkl4NGpTbUpZdDA1Z1VJLzByQnkwYlpvRitqCk9CRVBXVmorZ0pLVGdNTnhvWnZGQ3JSV3NpYnNjeURBakxKOEFsSG9EcXZUL1lyV3hIV2t2aXRzRm5tTGMxZjIKWmFqQjd0b25JaXd2SmpsaEd1QmxraEN6eTVkZXlueVc0d0UrNUVxTENzY1JsR1E1cWxidEpLMTcybnF0NUFvVQpUeVcxMDA5ekdZZlFaTTcxaEJjWWhkb0R5N3gvc0VRRGVST1drMXcwaHp6aTNKVGxZNmVLTmtnNjVlWDVHczVkCkN5VWoKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    tls.key: LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCk1JSUpRZ0lCQURBTkJna3Foa2lHOXcwQkFRRUZBQVNDQ1N3d2dna29BZ0VBQW9JQ0FRQ29xaFlncWhDTzloM1MKRW45UlZhajloUTFuVk1CcmJSTDIzZHBEaFVYRmIySkRKNkhFcUN4QWJOVkdESlFTbHZQUEgxWnhIQ3d5SnJOSQp6WUExL1Bzb2g3UXM3cVVJZHUzbVVNZitaQ2ZWUTFUREhUQTVKSHkxM0h6WFhBU1FMYmR2czF0TGl3bkUwcUhvCmZUNFhXUFYrb1FVL1hzZ1ZtRzE3QVBQOXF1bVM3clQ2ZWxpWWpKNzVnb0xsaCtnOFNXSXdGYVoyVkZLck55VS8KNWZTVzJlczRJQUdhT0dySnZQNTgxMFZnL2hUdjNMU0g0eTJJOFk1ZDEyNlVCbnpzRUlpbnhFVjRxcTRDZTlLMwpLOU5iRHB5S0FiLzdVRWZaVGdtQ2E1NFF4bFZmbEx3TDcyUFlySG1zMlRwbEgvNFBsUXpOVGlneGkwMi9kS1JJCm1KMzYvMFRqRFQzVmpGSGxIL2hLUmEwUHZNNjZwam9sdUNON05jYWN5bWgxN29Bd2FoeGtCNXhyNVN5YW55L3YKcXY5ZG9QMkxEMjcrdkFBNU4zcmo5S2NkZngwejVaRGFWWEI4RnZPSmNiVVB6ZjhwaVBjRU40ZzNFQ0Q5dGlwZwpENFAwRTNrb3lKNEdaYVVkMUR0VXVFZkU4RlFZZWRFL0tsdEVhUHpZaHgxdWJycFhGWWp5NmpQdGJCZlh0REttCmd3QmFubFp1SjJjNjkwenBDK1N2amw5ZmdYZ2VtditrSFBmaUNJTngxNlUxWUFUbkZSOTJlQ2pXMlJ0SDdCblAKVE92aWdJRHh1UTNBelJPbTFrK3Z5N1hxTHE4azRGblNFckZsMjlwQVB0VTlzUXF0cENoNVVVRU1STDk3c09rSQpINjliQi9TMHlKaUZFemloU1Zka01sOTN6cDRLUlFJREFRQUJBb0lDQUI1NWJ0cVRoNzdnNnJYdWFDa290a29xClBHZi9hTzN3RFFWa1E5Ky95SUc2RkpISUt5TW1kS3BtVDZtaXM2MWhMMmVzeEpoV3pBaDZ0Qm1USWRkL1lGenMKR2dxbFN0WEs4VVNVTTNKOEk3TlZnVTdvcXZJa0xORktKNWRjMWFrOFRXREdFaGlGNUd1MjJCQ3k4bUZURGpaRgpQemE2T0NJb3dwMC9LSmFjaHl2b2EvUTIzMk9ld3NtWHduRlkrMFhMUjQyakY4clpCUC9SREs2dExLS0YrTWxSCkJTRVprUGZtNlQ5aXdQTWJyV3BDWVM4TlJjaXd4bldmcTJmYy9UekU1d0FUQnk5Y2ZXak8rUjFsOG9BSkZaRWMKN0ZScTVyUjFkR1BuY0cyYy9FZHpEd0FFRHJFVFJyL0xseEMyOVZkWTZiRjlyZWIzd2NJMlA4bXJJNTQ2ZWJaVwpONFIvWEpMb1BaUVBCTUZXcHBoV0xLeVdlTWcrMEJGc2xtelJZaXR1bSszeHVvS2FPOFNkajBPNDYyTU5KMVFUCkNZL25kMm14eWdBWXRlRVRYeEsrUW5iSEFuMkpVRmVSZ1dJUUF2WjR4TXVVSnNVNWhOM1RCU3hrYnM3WUpTVnoKSWNXazNwak9ta0QyS3NQNm9DY1JHcFhiK3U1TXdrWkRXZ1Zick1LdHVLWjdIMVVnb3VpTmt4cGlrM00vMW4rQwpPSkZaUlR1WWhYcWE1elVKT3hNaFd5M1RtQ0pSbE8yeDhLdElnSTkrWUNQcldJU0lzak1VdjlaTGdWYzNFaWdlCldja3J0bHlIVGRNa21uWGJNeDJJbFBYRFlvVyt4RVdQanRqZWVMaXVUQlpCOFNqQmpsYmpTOGh1aWRvbXdkOS8KdUZRcUlwaTUxbVE5WFBza3JiSEpBb0lCQVFEZ3l3NFZQUGFnWU1FWlYyYXgySGZibTNkaDVDbVorTlBLSldmTQpOeTFpMGJVVHhlZGIvRWFhY1psVmRRd1dQa3dwbjhxWE1RYlovRml1UnQvQzhkazA1SHlUMHVwTEVwRTl2cDFmCkdCVU9wdnhTN2tCTmtYNVdzVFVkSFNRam0yRTY3Y3Y1YWhGRjN4R1V4QnlCc0JHU3VKSjI0MS9PdkIzS2UzdWwKajlSNys2QVh1M2tCdTFjSVdkTXd0L1BEeFdGTXhpN0NaWENXbkw5MEJXeHlEMmxTQ2srYjlvaEZtVlJsN2p2Vgo0WUp0S1pJN05pSTI3RjVkZGdwU3FOY1cybGxuK2VzUmZwUmcwbDgyOWkwVnJYOFViOXRocVZYOEdQOUJzQUNOCk8yRTNjaGNMcVBtaEsxM0l5TDRWVDZ1ZXlRTFNpSWNHdW5mbDZsdDRXVnZHcnVKNUFvSUJBUURBRkVQOVlTcWEKcTZRSWVrRVByU2twVFMvSGtBQzc2RDM2MnQ1RjJXcXlyL3d0d3pjZC8xdTJuU0o0aEozM0tPcUZNMDdTU1hINQp5dUpvTjFUSC8xQXRDaHRkSjh3VFdCbWRFbEY2RUNiVlA2UXFBM2x2NzFMQXlaUFBKd0NQVksyc3JNZUVzWEdMCjZTTk04VjRWN0VUc1RhaTBtTm5iNmlINHpHcEFBcW5ab1V1Z3VSb2JWTURleWlGdkFmYmV4Q2FkTjd1ajZZM20KSmdmWDhOa3RndlQvcVgyc0Mzbkw3S2VhL1M4a0cvSUd2Z0V0QlhsS0NxUFFkaW1ZN3lFM3ZBYkdjS3hiMklUcwpvd2xJTXJTdHkvZVprQWZSUVJ6VmdHWktrbHZPTUpmVHZTLzM2dVJvSWxkQ2lqZ1VKbjk1RlAxRHhZRmtlUFJVCm5wbWtGM2c5UzFNdEFvSUJBUUNGMThENTJrT0orREhoRWM0ZWhDSFJTdjdJOVYzanhHanR5bG5FR1BKWURUN0EKbUN3SjgrcGgyTk9RTTFIUUNLVzJmdUxVSktmTXNOaG9ZK0NsSlBUTDdtTlNiTmw5ZTMrcEFNNllxVEVZZVVweApZbFE5R3l6YkYwWGxvTTA4dWk0cE5SOG0wUVdaMFppWk9DODA5STF5QzUyQlZoNWNiRnRjalN0d3gzT2ZvcEdPCnI5djdzUHpBQnlPY3RWcFpyVE1pMERsVkc2cnVza080SSthTy95Z1paZlJDaTRaVjBsYVRIa2JZTVI1RU91VkwKaGc3WEh1T015RlNiSk5aMFQrdTk0ZXNaam9Gd0tMSHllcDhiY2lMaEd6ekhLRmorOGk5QmdEYkQ0S0Fnc0RpUwpnN2ltUXJqamJNcm95M2dHNGU1aFJsTUhLWTFzQXA4VnlEYkIxSzBaQW9JQkFFNGpzR2tTcjBkT3ppTll3TUVFCjI3cmVtTEpocGJyTHh2TVhPTmRIbEgxdTFITTFlR1d1clh2RHZ2TXlXQ1RsTTByT3phRUVtZVpabi9Odmx3RHAKbjQ5RERsQytVT29KckJuWEN1aFNTOG55NHZEb2l1MTdlYm5PQjJCOWFGL212ZVNDUVlSOHYwbUFwWWkycEdUYwp3a2t6YW85Vm8wTXdvM05ZalZ6TytKUDlad3ZTWVlsKzJCdUtOVUc0bGRxWUIzRnI2OXpKdFoxTUdXTENxMGMxCmdEL0Zqc2QvdjZPeStaZzJxWWZTQ29xdG4wTFdlRG9qeS9LUkwyajAxeG1hVjFOQklRMFlMek1wUEN4djNFcmIKc0RWN2Y0S2tMM3UzaVdXSzF1Z3hvb0pUODRDeXdRcVA5ZG11NnhOZmVmb2pETWtXUHdaRm9uZ09NVWlzOTVCUAo1VjBDZ2dFQVBOTm9taEhBWktUQjFyaUc4cHRvaUZPUlpNYkpib1lCRkN3NUsyNEdqL3YyRkE3MTVDbjdsVHBzClNNVVcvcHkwSEJoOWhUUlU3YWxmZ0YwU25rQnBXYzdMenlRSDBNZUY5ZEJkczl5NHE2dnBDbncxSUpOUWVsaHYKT0Vla24zWFdJVitlWEg5dkxyVnR1dkJvS1pSTmg5enVlVXBSRmI3MktXM1FNckRhTDdudjhUTEZpeWNORUl1TAplK0YwOUdDSE5kdk5zdnRMRzhGUE5MdXZmNGh0UkdpWVUrUjB4eUNPTy9XMFlaaS9yblNoTlVCaGIxekY0Z1M5CkRRZzR1TmtRTmN6eFRrd0RWL1FacTZ4ak1UUkNwdWlUV2lsaWVwUi91YjVFS09sS05Uait6N0dwOFAyZUcwcUsKWHJWVEVuR05tMGVOeExaYXhDbTJueHhnMDJWTmlBPT0KLS0tLS1FTkQgUFJJVkFURSBLRVktLS0tLQo=
  kind: Secret
  metadata:
    creationTimestamp: "2024-09-28T12:48:47Z"
    name: secret-tls
    namespace: netology-2
    resourceVersion: "549114"
    uid: 4c929be3-0868-4ab6-9b66-ae1502512879
  type: kubernetes.io/tls
kind: List
metadata:
  resourceVersion: ""
```

6. Написан манифест для [Ingress]()

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-https
  namespace: netology-2
spec:
  rules:
    - host: app-nginx.ru
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: service-https
                port:
                  number: 80

  tls:
  - hosts:
    - app-nginx.ru
      secretName: secret-tls
```

6. Написан манифест [Service]() для Nginx

```yaml
apiVersion: v1
kind: Service
metadata:
  name: service-nginx
  namespace: netology-2
spec:
  type: NodePort
  ports:
    - name: nginx-https
      port:  80
      nodePort: 30002
      protocol: TCP
      targetPort: 80
  selector:
    app: app-main
```

6. Запустим манифесты **Deployment**, **ConfigMap**, **Ingress**, **Service**

![pods]()
