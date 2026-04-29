# Отчет по практической работе №2
### Студент: Синицина Полина Владимировна
### Группа: БСБО-16-23
### 1. Информация о кластере
#### 1.1 Статус Minikube
```
containers-lab-1 git:(lab2/kubernetes-basics) ✗ minikube status  
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

#### 1.2 Узлы кластера
```
kubectl get nodes -o wide
NAME       STATUS   ROLES           AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE                         KERNEL-VERSION     CONTAINER-RUNTIME
minikube   Ready    control-plane   49m   v1.35.1   192.168.49.2   <none>        Debian GNU/Linux 12 (bookworm)   6.12.72-linuxkit   docker://29.2.1
```

### 2. Созданные ресурсы
#### 2.1 Pods
```
 kubectl get pods -o wide
NAME                                   READY   STATUS             RESTARTS   AGE     IP             NODE       NOMINATED NODE   READINESS GATES
go-app-deployment-9574dd56b-dlsf7      1/1     Running            0          23m     10.244.0.121   minikube   <none>           <none>
go-app-deployment-9574dd56b-l2bgg      1/1     Running            0          23m     10.244.0.123   minikube   <none>           <none>
my-go                                  0/1     CrashLoopBackOff   0          4m43s   10.244.0.131   minikube   <none>           <none>
nginx-deployment-74fcddd4b-699fk       1/1     Running            0          4m41s   10.244.0.136   minikube   <none>           <none>
nginx-deployment-74fcddd4b-vdtd9       1/1     Running            0          4m43s   10.244.0.134   minikube   <none>           <none>
postgres-deployment-54c59b77b5-4njl8   1/1     Running            0          25m     10.244.0.112   minikube   <none>           <none>
postgres-deployment-54c59b77b5-lbqnq   1/1     Running            0          4m41s   10.244.0.135   minikube   <none>           <none>
postgres-deployment-54c59b77b5-njl87   1/1     Running            0          4m43s   10.244.0.133   minikube   <none>           <none>

```

#### 2.2 Deployments

```
kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
go-app-deployment     2/2     2            2           26m
nginx-deployment      2/2     2            2           26m
postgres-deployment   3/3     3            3           26m
```

#### 2.3 Services
```
kubectl get services
NAME               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
go-app-service     ClusterIP   10.111.139.141   <none>        8080/TCP       26m
kubernetes         ClusterIP   10.96.0.1        <none>        443/TCP        27m
nginx-service      NodePort    10.109.249.187   <none>        80:30080/TCP   26m
postgres-service   ClusterIP   10.105.198.178   <none>        5432/TCP       26m
```
### 3. Скриншоты работы приложения
#### 3.1 Главная страница
![Главная страница](screenshots/lab2_1.png)
![Запрос](screenshots/lab2_2.png)
### 4. Эксперименты с масштабированием
#### 4.1 Масштабирование до 5 реплик
\`\`\`
kubectl scale deployment/nginx-deployment --replicas=5
deployment.apps/nginx-deployment scaled

kubectl get deployments nginx-deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   5/5     5            5           29м
\`\`\`
#### 4.2 Проверка распределения нагрузки
\`\`\`
Сделано: в кластере отправлено 50 запросов на `http://nginx-service/api/health` (curl из временного pod).

Количество обработанных запросов `GET /api/health` по nginx pod’ам (интервал 21:02):
nginx-deployment-846d7bc598-5cjlj: 10
nginx-deployment-846d7bc598-6n96l:  9
nginx-deployment-846d7bc598-csdq8:  15
nginx-deployment-846d7bc598-dv99h:  10
nginx-deployment-846d7bc598-rt6lp:  6

Примеры строк из access-лога nginx на 21:02 (GET /api/health):
nginx-deployment-846d7bc598-5cjlj:
10.244.0.95 - - [26/Mar/2026:21:02:19 +0000] "GET /api/health HTTP/1.1" 200 21 "-" "curl/8.7.1"
nginx-deployment-846d7bc598-6n96l:
10.244.0.95 - - [26/Mar/2026:21:02:19 +0000] "GET /api/health HTTP/1.1" 200 21 "-" "curl/8.7.1"
nginx-deployment-846d7bc598-csdq8:
10.244.0.95 - - [26/Mar/2026:21:02:19 +0000] "GET /api/health HTTP/1.1" 200 21 "-" "curl/8.7.1"
nginx-deployment-846d7bc598-dv99h:
10.244.0.95 - - [26/Mar/2026:21:02:19 +0000] "GET /api/health HTTP/1.1" 200 21 "-" "curl/8.7.1"
nginx-deployment-846d7bc598-rt6lp:
10.244.0.95 - - [26/Mar/2026:21:02:19 +0000] "GET /api/health HTTP/1.1" 200 21 "-" "curl/8.7.1"
\`\`\`
### 5. GitHub Actions
#### 5.1 Успешная валидация манифестов

### 6. Ответы на контрольные вопросы
1. В чем разница между Pod и Deployment?
```
Pod - минимальная сущность k8s, которая запускается в единственном экземпляре с одни или несколькоми контейнерами внутри. 
Deployment - это констроллер, который управляет жизненным циклом репликов подов. В нем описаывается желаемые ресурсы и их количество, чего нет в api pod.
```
2. Для чего нужен Service типа ClusterIP?
```
Для предоставления стабильного ip на уровне кластера k8s, без возможности подключения из вне.
```
3. Как ReplicaSet обеспечивает самовосстановление?
```
Манифесты k8s - декларативный код, то есть мы просим некий ожидания, k8s реализовывает их. В ReplicaSet указываются количество реплик, желаемые ресурсы, необходимый образ. Если одно из условий не выполняются, то ReplicaSet пытается прийти к этому результату. Если под упадется, он не будет удолетворять условию, и ReplicaSet пересоздаст упавший под.
```
4. Что произойдет с приложением, если удалить под PostgreSQL?
```
Если не настроены StatefulSet, то у нас не оснанутся данные приложения и самовостановится под. Если он есть, то под восстановится и временно нагрузку возмут другие поды, если настроены реплики. 
```
### 7. Выводы
Развернул Minikube и на практике разобрался с основными сущностями Kubernetes: Pod, Deployment, ReplicaSet и Service. В ходе лабораторной развернул приложение с nginx в виде Deployment, проверил доступность через Service и убедился, что масштабирование увеличивает число работающих pod’ов и распределяет нагрузку. Также закрепил работу с манифестами и GitHub Actions в рамках контроля корректности Kubernetes-развёртываний.
