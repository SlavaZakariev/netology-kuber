## 22.9 Управление доступом - Вячеслав Закариев

### Цель задания

В тестовой среде Kubernetes нужно предоставить ограниченный доступ пользователю.

### Чеклист готовности к домашнему заданию

1. Установлено k8s-решение, например MicroK8S.
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым github-репозиторием.

### Инструменты / дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/).
2. [Пользователи и авторизация RBAC в Kubernetes](https://habr.com/ru/company/flant/blog/470503/).
3. [RBAC with Kubernetes in Minikube](https://medium.com/@HoussemDellai/rbac-with-kubernetes-in-minikube-4deed658ea7b).

---

### Задание 1. Создайте конфигурацию для подключения пользователя

1. Создайте и подпишите SSL-сертификат для подключения к кластеру.
2. Настройте конфигурационный файл kubectl для подключения.
3. Создайте роли и все необходимые настройки для пользователя.
4. Предусмотрите права пользователя. Пользователь может просматривать логи подов и их конфигурацию \
   (`kubectl logs pod <pod_id>`, `kubectl describe pod <pod_id>`).
6. Предоставьте манифесты и скриншоты и/или вывод необходимых команд.

---

### Правила приёма работы

1. Домашняя работа оформляется в Git-репозитории в файле README.md. На выполненное задание пришлите ссылку на .md-файл.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl`, скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

---

### Решение 1

1. С помощью **OpenSSL** создаём закрытый ключ **tls.key** и запрос на подписание для пользователя **netology**

![genrsa](https://github.com/SlavaZakariev/netology-kuber/blob/9429a72bb7213e948bdca97c5b4640b0119386f0/2.4/resources/kub_2-9_1.1.jpg)

2. Генерируем самоподписной файл сертификата на **365 дней**

![x509](https://github.com/SlavaZakariev/netology-kuber/blob/9429a72bb7213e948bdca97c5b4640b0119386f0/2.4/resources/kub_2-9_1.2.jpg)

3. Создаём пользователя **netology** на сервере

![useradd](https://github.com/SlavaZakariev/netology-kuber/blob/9429a72bb7213e948bdca97c5b4640b0119386f0/2.4/resources/kub_2-9_1.3.jpg)

4. Настраиваем конфигурационный файл **kubectl** для пользователя **netology**

![ctl](https://github.com/SlavaZakariev/netology-kuber/blob/9429a72bb7213e948bdca97c5b4640b0119386f0/2.4/resources/kub_2-9_1.4.jpg)

5. Создаём контекс с именем **netology-context** и проверяем его наличие

![context](https://github.com/SlavaZakariev/netology-kuber/blob/9429a72bb7213e948bdca97c5b4640b0119386f0/2.4/resources/kub_2-9_1.5.jpg)

6. Включаем функцию RBAC на нашем microk8s и проверем его статус

![rbac](https://github.com/SlavaZakariev/netology-kuber/blob/9429a72bb7213e948bdca97c5b4640b0119386f0/2.4/resources/kub_2-9_1.6.jpg)

7. Создадим роль [Role](https://github.com/SlavaZakariev/netology-kuber/blob/main/2.4/yaml/role.netology.yml) и привязку роли к субъекту [RoleBinding](https://github.com/SlavaZakariev/netology-kuber/blob/main/2.4/yaml/rolebinding.netology.yml)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-logs-describe
  namespace: default
rules:
  - apiGroups: [""]
    resources: ["pods", "pods/log"]
    verbs: ["get", "watch", "list"]
```
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader
  namespace: default
subjects:
- kind: User
  name: netology
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-logs-describe
  apiGroup: rbac.authorization.k8s.io
```
![role](https://github.com/SlavaZakariev/netology-kuber/blob/9429a72bb7213e948bdca97c5b4640b0119386f0/2.4/resources/kub_2-9_1.7.jpg)
