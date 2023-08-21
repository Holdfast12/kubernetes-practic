### minicube

minikube start  
kubectl get componentstatuses  
kubectl cluster-info  
kubectl get nodes показать все серверы K8s кластера  
minikube stop  
minikube delete  
minikube start --cpus=4 --memory=8gb --disk-size=25gb  

логин в виртуалке (в окне VirtualBox) - root без пароля  
или логин docker  
пароль tcuser  

или с консоли хоста:  
minikube ssh  


kubectl run hello --generator=run-pod/v1 --image=nginx:latest --port=80  
kubectl delete pods hello  

kubectl describe pods hello - информация о pod'е  

exec hello -- date  выполнить команду в pod'е  

kubectl exec -it hello -- sh зайти на под в интерактивном режиме  

kubectl logs hello - посмотреть лог-файл пода  

kubectl port-forward hello 8888:80 перенаправил 80 порт пода на 8888 порт локальной машины  

kubectl apply -f pod-myweb.yaml - запуск пода по манифест-файлу  
kubectl delete -f pod-myweb.yaml - удалить все что было запущено по манифест-файлу  

замена докер-образа делается без удаления пода, все остальное только после дестроя  
после редактирования манифеста еще раз - kubectl apply -f pod-myweb.yaml  
обязательно использовать в манифестах лейблы  
kubectl describe pods my-web  




kubectl get deployments  или  kubectl get deploy
kubectl get deploy  

kubectl create deployment mikhail-deployment --image nginx:latest  

kubectl describe pod mikhail-deployment-758dcccc87-5dmnr  

kubectl describe deployments mikhail-deployment  

kubectl scale deployment mikhail-deployment --replicas 4  

kubectl get pods  - число подов увеличилось в 4 раза  

C:\Users\michael\kubernetes-practic>kubectl get deploy  
NAME                 READY   UP-TO-DATE   AVAILABLE   AGE  
mikhail-deployment   4/4     4            4           13m  

 - replicas set
C:\Users\michael\kubernetes-practic>kubectl get rs  
NAME                            DESIRED   CURRENT   READY   AGE  
mikhail-deployment-758dcccc87   4         4         4       14m  

если удалить один из подов, сразу запустится новый  

kubectl autoscale deployment mikhail-deployment --min=4 --max=6 --cpu-percent=80  


horizontal pod autoscaling (hpa)  
C:\Users\michael\kubernetes-practic>kubectl get hpa  
NAME                 REFERENCE                       TARGETS         MINPODS   MAXPODS   REPLICAS   AGE  
mikhail-deployment   Deployment/mikhail-deployment   <unknown>/80%   4         6         4          73s  


меняем образ в деплойменте:

C:\Users\michael\kubernetes-practic>kubectl rollout history deployment/mikhail-deployment  
deployment.apps/mikhail-deployment  
REVISION  CHANGE-CAUSE  
1         <none>  

C:\Users\michael\kubernetes-practic>kubectl rollout status deployment/mikhail-deployment  
deployment "mikhail-deployment" successfully rolled out  

меняем контейнер в деплое, имя контейнера перед знаком "=" берем из describe-перечисления контейнеров в деплое  
C:\Users\michael\kubernetes-practic>kubectl set image deployment/mikhail-deployment nginx=nginx:1.14 --record  

просмотр статуса во время или в конце процесса деплоя  
C:\Users\michael\kubernetes-practic>kubectl rollout status deployment/mikhail-deployment  
deployment "mikhail-deployment" successfully rolled out  


просмотр истории деплоймента. none - это когда не было деплоя  
C:\Users\michael\kubernetes-practic>kubectl rollout history deployment/mikhail-deployment  
deployment.apps/mikhail-deployment  
REVISION  CHANGE-CAUSE  
1         <none>  
2         kubectl set image deployment/mikhail-deployment nginx=nginx:2.14 --record=true  
3         kubectl set image deployment/mikhail-deployment nginx=nginx:2.12 --record=true  
4         kubectl set image deployment/mikhail-deployment nginx=nginx:1.14 --record=true  


вернуться на предыдущую версию, я вернулся 2 раза (на 2ю версию), так как 2.12 версию образа взял с потолка и была ошибка :)  
C:\Users\michael\kubernetes-practic>kubectl rollout undo deployment/mikhail-deployment   
deployment.apps/mikhail-deployment rolled back  

сразу на нужную версию
C:\Users\michael\kubernetes-practic>kubectl rollout undo deployment/mikhail-deployment --to-revision=2
deployment.apps/mikhail-deployment rolled back

обновление контейнеров, чтобы загрузились актуальные latest версии !!!
C:\Users\michael\kubernetes-practic>kubectl rollout restart deployment/mikhail-deployment
deployment.apps/mikhail-deployment restarted

создаем деплой по манифест-файлу
C:\Users\michael\kubernetes-practic\deployments>kubectl apply -f deployment-first.yaml
deployment.apps/my-web-deployment created

деплой по манифест-файлу с репликами
C:\Users\michael\kubernetes-practic\deployments>kubectl apply -f deployment-replicas.yaml
deployment.apps/my-web-deployment-replicas created

C:\Users\michael\kubernetes-practic\deployments>kubectl get deploy
NAME                         READY   UP-TO-DATE   AVAILABLE   AGE  
mikhail-deployment           4/4     4            4           7h59m
my-web-deployment            1/1     1            1           87m  
my-web-deployment-replicas   3/3     3            3           50s 

перенаправил 80 порт одного пода из реплики на локалхост, зашел - работает :)
C:\Users\michael\kubernetes-practic\deployments>kubectl port-forward my-web-deployment-replicas-55bb54494-rkf2f 8888:80 
Forwarding from 127.0.0.1:8888 -> 80
Forwarding from [::1]:8888 -> 80
Handling connection for 8888

посмотрел какие версии API автоскейлинга у меня есть, поправил манифест-файл
C:\Users\michael\kubernetes-practic\deployments>kubectl api-versions | findstr autoscaling
autoscaling/v1
autoscaling/v2


запустил деплой с автоскейлингом с манифест файла
C:\Users\michael\kubernetes-practic\deployments>kubectl apply -f deployment-autoscaling.yaml
deployment.apps/my-web-deployment-autoscaling unchanged
horizontalpodautoscaler.autoscaling/my-autoscaling created


смотрю инф по автоскейлингу
C:\Users\michael\kubernetes-practic\deployments>kubectl get hpa
NAME                 REFERENCE                                  TARGETS                        MINPODS   MAXPODS   REPLICAS   AGE
mikhail-deployment   Deployment/mikhail-deployment              <unknown>/80%                  4         6         4          8h     
my-autoscaling       Deployment/my-web-deployment-autoscaling   <unknown>/70%, <unknown>/80%   3         6         3          9m58s 

еще информация будет по describe deployment

для замены образа редактирую манифест-файл, потом снова apply

для удаления - delete с указанием манифест-файла

снести все деплойменты
kubectl delete deployment --all

сервисы

ClusterIP - ip только внутри кластера
NodePort - определенный порт на каждой ноде
ExternalName - DNS CNAME Record
LoadBalancer - только в облачных k8s кластерах (AWS, GCP, Azure)

C:\Users\michael\kubernetes-practic\deployments>kubectl apply -f deployment-autoscaling.yaml         
deployment.apps/my-web-deployment-autoscaling created
horizontalpodautoscaler.autoscaling/my-autoscaling unchanged

беру действующий деплоймент и применяю expose
C:\Users\michael\kubernetes-practic\deployments>kubectl expose deployment my-web-deployment-autoscaling --type=ClusterIP --port 80
service/my-web-deployment-autoscaling exposed

смотрю сервисы на кластере (services можно сократить до svc - его алиас)
C:\Users\michael\kubernetes-practic\deployments>kubectl get services
NAME                            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
kubernetes                      ClusterIP   10.96.0.1        <none>        443/TCP   7d16h
my-web-deployment-autoscaling   ClusterIP   10.108.184.200   <none>        80/TCP    8m5s

теперь находясь на сервере ноды можно получить доступ к страничке на адресе 10.108.184.200
при ClusterIP происходит балансировка по внутренним ip адресам сервиса в кластере.






