# BRK experiments with kubernete objects 

### Experiment 1
1. svc-apache2.yml creates a service which selects from apache1.yml pod and apache2.yml pod and exposes the same service ip:port, so randomly one application is rendered on the webpage. 
2. The difference between apache1.yml and apache2 is:
    - apache1.yml : Renders the default "it works!" webpage from httpd:alpine image
    - apache2.yml : Renders an index.html mounted from a local drive. 

### Experiment 2
