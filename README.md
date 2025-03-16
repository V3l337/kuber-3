# Домашнее задание к занятию «Запуск приложений в K8S»

## Задание 1. Создать Deployment и обеспечить доступ к репликам приложения из другого Pod

### 1.1. Создание Deployment приложения

Создайте Deployment приложения, состоящего из двух контейнеров: `nginx` и `multitool`.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-multitool-depl
  labels:
    app: mvb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mvb
  template:
    metadata:
      labels:
        app: mvb
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
      - name: multitool
        image: wbitt/network-multitool
        ports:
        - containerPort: 8080
        command: ["/bin/sh", "-c", "httpd -p 8080 -h / & sleep infinity"]
```

После запуска увеличить количество реплик работающего приложения до 2.
```
replicas: 2
```
или
```
kubectl scale deployment nginx-multitool-depl --replicas=2
```
Продемонстрировать количество подов до и после масштабирования.
Скриншот ДО 

![1ДО](https://github.com/user-attachments/assets/efcef164-4884-4c5e-a18a-aa2fe0e330c6)

Скриншот ПОСЛЕ

![1ПОСЛЕ](https://github.com/user-attachments/assets/ad33bc7a-e3cb-413f-932e-4ad0584e5402)


Создать Service, который обеспечит доступ до реплик приложений из п.1.
```yml
    apiVersion: v1
    kind: Service
    metadata:
      name: nginx-multitool-service
    spec:
      selector:
        app: mvb  
      ports:
      - name: nginx-port
        protocol: TCP
        port: 8181          # Порт, на котором сервис принимает запросы
        targetPort: 80    # Порт, на который пересылает внутри пода (nginx)
      - name: multitool-port
        protocol: TCP
        port: 8080        # Порт сервиса для multitool
        targetPort: 8080  # Внутренний порт multitool
      type: ClusterIP
```
Создать отдельный Pod с приложением multitool и убедиться с помощью curl, что из пода есть доступ до приложений из п.1.
```yml
      apiVersion: v1
      kind: Pod
      metadata:
        name: multitool-test
        # labels:
        #   app: mvb
      spec:
        containers:
        - name: multitool
          image: wbitt/network-multitool
          command: ["/bin/sh", "-c", "sleep infinity"]
 ```
 Подключился 
 ```
 kubectl exec -it multitool-test -- /bin/bash
 ```
 
 Скриншот проверки

![Curl доступ до приложений из п 1](https://github.com/user-attachments/assets/7de107b6-a783-4ce8-9293-9184de8d42c5)
    
Задание 2. Создать Deployment и обеспечить старт основного контейнера при выполнении условий

Создать Deployment приложения nginx и обеспечить старт контейнера только после того, как будет запущен сервис этого приложения.
Убедиться, что nginx не стартует. В качестве Init-контейнера взять busybox.
```yml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-depl
      labels:
        app: mvb
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: mvb
      template:
        metadata:
          labels:
            app: mvb
        spec:
          containers:
          - name: nginx
            image: nginx:latest
            ports:
            - containerPort: 80
          initContainers:
          - name: wait-for-service
            image: busybox 
            command: ['sh', '-c', 'until nslookup nginx-service; do echo waiting for nginx-service; sleep 2; done;']
```
Создать и запустить Service. Убедиться, что Init запустился.
```yml
    apiVersion: v1
    kind: Service
    metadata:
      name: nginx-service
    spec:
      selector:
        app: mvb
      ports:
        - protocol: TCP
          port: 8080      # Порт, на котором будет доступен сервис
          targetPort: 80 # Порт, на котором контейнер слушает внутри Pod
      type: ClusterIP
```
Продемонстрировать состояние пода до и после запуска сервиса.
```
kubectl get svc 
```
```
kubectl get deployments
```
```
kubectl get pods
```
```
kubectl run -i --tty --rm debug --image=busybox --restart=Never -- nslookup nginx-service.default.svc.cluster.local
```
Исправил в deployment (nginx-depl)  Так как не запускался под, потому что требовалось указать подробнее расположение сервиса.
```
command: ['sh', '-c', 'until nslookup nginx-service; do echo waiting for nginx-service; sleep 2; done;'
```
На
```
command: ['sh', '-c', 'until nslookup nginx-service.default.svc.cluster.local; do echo waiting for nginx-service; sleep 2; done;']
```
Все заработало 

Скрин ДО

![ДО](https://github.com/user-attachments/assets/494e4a02-a726-42a2-8d09-066d74c9a45e)

Скрин ПОСЛЕ


![после](https://github.com/user-attachments/assets/d28de394-f8b2-4a50-a5d5-a5cb8abab620)
