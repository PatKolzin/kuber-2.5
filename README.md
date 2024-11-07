# Домашнее задание к занятию "`Helm`"



---

### Задание 1. Подготовить Helm-чарт для приложения

1. Необходимо упаковать приложение в чарт для деплоя в разные окружения. 
2. Каждый компонент приложения деплоится отдельным deployment’ом или statefulset’ом.
3. В переменных чарта измените образ приложения для изменения версии.

#### Решение

Проверяем установку ```helm```:
![image](https://github.com/user-attachments/assets/07bef095-06fb-45ce-ba6c-77900cf9fd74)

Создаём собственный чарт с именем `nginx-chart`
![image](https://github.com/user-attachments/assets/c39af3aa-94e4-40da-a43f-7a92170f453a)

Подготовим ```deployment``` ```service``` и разместим их в директории ```templates/```:

- Манифест [templates/deployment.yaml](https://github.com/PatKolzin/kuber-2.5/blob/main/src/nginx-chart/templates/deployment.yaml)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "nginx-chart.fullname" . }}
  labels:
    app: {{ include "nginx-chart.name" . }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ include "nginx-chart.name" . }}
  template:
    metadata:
      labels:
        app: {{ include "nginx-chart.name" . }}
    spec:
      containers:
        - name: nginx
          image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
          ports:
            - containerPort: 80
```
- Манифест [templates/service.yaml](https://github.com/PatKolzin/kuber-2.5/blob/main/src/nginx-chart/templates/service.yaml)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "nginx-chart.fullname" . }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: 80
  selector:
    app: {{ include "nginx-chart.name" . }}
```
Подготовим ```Chart```: 
- Манифест [Chart.yaml](https://github.com/PatKolzin/kuber-2.5/blob/main/src/nginx-chart/Chart.yaml)
```yaml
  apiVersion: v2
  name: nginx-chart

  type: application

  version: 0.1.0
  appVersion: "1.16.0"
```
Подготовим ```values```:
- Манифест [values.yaml](https://github.com/PatKolzin/kuber-2.5/blob/main/src/nginx-chart/values.yaml)
```yaml
image:
  repository: nginx
  tag: stable

replicaCount: 2

service:
  enabled: true
  type: ClusterIP
  port: 80
```

Поменяем номер версии приложения в файле ```Chart.yaml```:
```yaml
  apiVersion: v2
  name: netology

  type: application

  version: 0.1.2
  appVersion: "1.20.0"
```
![image](https://github.com/user-attachments/assets/41434ed0-2d87-4479-9688-10951bf19a91)

Видим, что версия образа поменялась на ```1.20.0```




------
### Задание 2. Запустить две версии в разных неймспейсах

1. Подготовив чарт, необходимо его проверить. Запуститe несколько копий приложения.
2. Одну версию в namespace=app1, вторую версию в том же неймспейсе, третью версию в namespace=app2.
3. Продемонстрируйте результат.

#### Решение

Запускаем несколько копий приложения в namespace `app1` и `app2`
![image](https://github.com/user-attachments/assets/53431d26-e6de-41be-b882-f8f509b2ce4f)
![image](https://github.com/user-attachments/assets/eb690afd-cb3f-405c-9a2d-e8a73d060d12)

Проверим установку:

![image](https://github.com/user-attachments/assets/8dff0fc4-cf04-4e75-a8ae-ccd98b4f0333)

Приложение также успешно развернуто.

Между собой приложения отличаются версиями образов, указанных в тегах.
