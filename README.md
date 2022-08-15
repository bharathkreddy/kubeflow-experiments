# BRK experiments with kubernete objects 

### Experiment 1
1. svc-apache2.yml creates a service which selects from apache1.yml pod and apache2.yml pod and exposes the same service ip:port, so randomly one application is rendered on the webpage. 
2. The difference between apache1.yml and apache2 is:
    - apache1.yml : Renders the default "it works!" webpage from httpd:alpine image
    - apache2.yml : Renders an index.html mounted from a local drive. 
3. steps 
    - kubectl create -f apache1.yml
    - kubectl create -f apache2.yml
    - kubectl create -f svc-apache2.yml
    - kubectl describe scv svc-apache2 : this gives endpoints matching ip add of both the pods which it is selecting.
    - curl http://localhost:{port to which svc is exposed}
    - kubectl exec --namespace brk pod/po1 -- wget --spider -q -S http://10.1.0.27:80 to exec into a running pod and see server resonce.

### Experiment 2 : SERVICE
1. Types of service:
    - **ClusterIP** : Creates a service which gets an IP and port but is accessible only via from inside the cluster, using *service IP and port.*
    - **NodePort** : Creates a service with IP and port which can be reached via any of the nodes from outside cluster by using *node's IP address and port of the service.* The etcd server knows how to redirect the traffic from node to service and from service to hit the pod. 
    - **LoadBalancer** : uses external load balancer 

### Experiment 3 : DEPLOYMENTS
1. Deployment creates a pod and a replicaset by itself without explicit definitions of pod or replicaset.
    - **kubectl create deployment dep1 --image=httpd:alpine -n brk**
        Above will create a replicaset and a pod.
    - as of now there are no ports which are exposed so cannot reach this app. To expose the port do below, this creates a service exposed at port specified as *ClusterIP* by default. 
        **kubectl expose deployment dep1 --port=80 -n brk** 
        Access the application from inside a pod since this is a *ClusterIP* by reaching to service IP & port from inside a pod, by : 
        **kubectl exec -n brk dep1-54568b4b49-hcmn8 -- wget --spider -q -S http://10.111.218.181:80**
    - if we want to see loadbalancing hapen do these:
        - Exec into a container: **kubectl exec --namespace brk -it dep1-54568b4b49-hcmn8 sh**
        - change the index.html
        - hit the service from within a pod and see random selection of pods.
    - if we delete the pod, the deployment would recreate the pod.
        **kubectl delete pod dep1-54568b4b49-xqwq4 -n brk** : Deltes the pod.
        **kubectl get all -o wide -n brk** : you will see another pod gets created with a new name.
    - Deployment first creates a replicaset and replicaset matches the no of pods to target number.
    - Scaling the deployment: **kubectl scale deployment dep1 --replicas=5 -n brk**
        - we can scale the deployment backwards also.
    - **kubectl get replicaset -o wide -n brk** to get replicasets in a namespace.
    - Deleteing a deployment deletes the pods, replicaset and deployment, but not the services !!

### Experiment 4: Replicaset
    1. Create replicaset by deploying the rc.yml
        - **kubectl create -f rc.yml -n brk**
    2. We see the RS creates all the pods by itself. But does not create any service like deployments do.
        - **kubectl get all -o wide -n brk**
    3. Deleting RC deletes the Pods as well.


