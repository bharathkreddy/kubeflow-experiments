# My notes on kubernetes objects

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
        - ClusterIP : Creates a service which gets an IP and port but is accessible only via from inside the cluster, using *service IP and port.*
        - NodePort : Creates a service with IP and port which can be reached via any of the nodes from outside cluster by using *node's IP address and port of the service.* The etcd server knows how to redirect the traffic from node to service and from service to hit the pod. 
        - LoadBalancer : uses external load balancer 

### Experiment 3 : DEPLOYMENTS
    1. Deployment creates a pod and a replicaset by itself without explicit definitions of pod or replicaset.
        - kubectl create deployment dep1 --image=httpd:alpine -n brk
            - Above will create a replicaset and a pod.
        - as of now there are no ports which are exposed so cannot reach this app. To expose the port do below, this creates a service exposed at port specified as ClusterIP by default. Exposing creates a service.
            - kubectl expose deployment dep1 --port=80 -n brk
            - Access the application from inside a pod since this is a ClusterIP by reaching to service IP & port from inside a pod, by : 
                - kubectl exec -n brk dep1-54568b4b49-hcmn8 -- wget --spider -q -S http://10.111.218.181:80
        - if we want to see loadbalancing hapen do these:
            - Exec into a container: kubectl exec --namespace brk -it dep1-54568b4b49-hcmn8 sh
            - change the index.html
            - hit the service from within a pod and see random selection of pods.
        - if we delete the pod, the deployment would recreate the pod.
            kubectl delete pod dep1-54568b4b49-xqwq4 -n brk : Deltes the pod.
            kubectl get all -o wide -n brk : you will see another pod gets created with a new name.
        - Deployment first creates a replicaset and replicaset matches the no of pods to target number.
        - Scaling the deployment: kubectl scale deployment dep1 --replicas=5 -n brk
            - we can scale the deployment backwards also.
        - kubectl get replicaset -o wide -n brk to get replicasets in a namespace.
        - Deleteing a deployment deletes the pods, replicaset and deployment, but not the services !!

### Experiment 4: Replicaset
    1. Create replicaset by deploying the rc.yml
        - kubectl create -f rc.yml -n brk
    2. We see the RS creates all the pods by itself. But does not create any service like deployments do.
        - kubectl get all -o wide -n brk
    3. Deleting RC deletes the Pods as well.

### Experiment 5: Selectors
    1: Create pod1 to pod6 using create -f command like : 
        - kubectl create -n brk -f pod6.yml
    2: List all pods with labels : 
        - kubectl get pods -n brk --show-labels
    3: Equity based selectors:
        Filter pods with : 
            - kubectl get pods -n brk -l color=blue --show-labels
    4: Set based selectors:
        Filter pods with : 
            - kubectl get pod -n brk -l 'color in (blue)'
            - kubectl get pod -n brk -l 'color in (blue,red)' --show-labels
            - kubectl get pod -n brk -l 'color notin (blue,red)' --show-labels

### Experiment 6: Resources and limits
    1. Deploy the resource.yml
        - kubectl create -f resource.yml -n brk
    2. Describe the pod to see Limits and Requests
        - kubectl describe pod resource-example -n brk
    3. Take NODE value from kubectl get pods and describe the node 
        - kubectl describe node docker-desktop -n brk
        - this gives if there is enough memory and memory left, the CPU and memory Requests and limits for the pod.
    4. CPU UNITS :
        - 1 CPU in k8's = 1 Azure vCore / 1 GCP core / 1 AWS vCPU / 1 Hyperthread on baremetal intel processor
        - ex: 0.5 CPU = 500m CPU / 500 milli CPU
        - ex: 0.1 CPU = 100m CPU / 100 milli CPU

### Experiment 7: DaemonSets
    1. DaemonSets ensure numeber of pods is same as no of nodes i.e. one pod per node, even in there are new joined nodes, the demonset will replicate the pod over the new joined node. Every node will run just 1 pod, no less no more. 
    2. Usually run for node monitoring, node log collection, storage cluster etc. 
    3. Create DaemonSet by using
        - kubectl apply -f daemonset.yml -n brk
    4. Check DaemonSets by using 
        - kubectl get daemonset -o wide -n brk ,or
        - kubectl get ds -o wide -n brk 
    5. Check the pods and check pod logs to see if everything is working
        - kubectl logs daemonset-example-6sl8k -n brk
    6. Delete the pod and see the pod spawn again.
    7. Delete the DS & pods created by DS are deleted as well.
        - kubectl delete ds daemonset-example -n brk
    8. 

![Demonset](Demonset_usecases.jpeg)

### Experiment 8: Making apps schedulable on Master node
    1. kubectl get node -o wide
    2. kubectl edit node {server/master}
    3. on vi editor remove the taint and effect
    4. Now master has become schedulable * but this is not adviced.*

### Experiment 9: Environment Variables
    1. Run mysql as a deployment, this dry-run with -o gives a yaml output and saves it to output.yaml which can be used as template for further deployments.
        - kubectl create deployment mydep1 --image=docker.io/mysql:5.6 --dry-run=client -o yaml > output.yaml
        - kubectl create -f output.yaml -n brk
    2. Above will fail, lets diagnose by first running the docker image of mysql and then traslating the docker command to .yml file.
        - <docker commands.>
    3. Observations and changes to output.yaml
    4. Deploy the services. 

### Experiment 10: Deploy a 2 tier application, wordpress powered by mysql. 

![mysql](2tier-mysql.png)


