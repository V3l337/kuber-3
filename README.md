Домашнее задание к занятию «Запуск приложений в K8S»

Задание 1. Создать Deployment и обеспечить доступ к репликам приложения из другого Pod

    Создать Deployment приложения, состоящего из двух контейнеров — nginx и multitool. Решить возникшую ошибку.
      ```yml
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
      Скриншот ПОСЛЕ
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
      

Задание 2. Создать Deployment и обеспечить старт основного контейнера при выполнении условий

    Создать Deployment приложения nginx и обеспечить старт контейнера только после того, как будет запущен сервис этого приложения.
    Убедиться, что nginx не стартует. В качестве Init-контейнера взять busybox.
    Создать и запустить Service. Убедиться, что Init запустился.
    Продемонстрировать состояние пода до и после запуска сервиса.

