# Chaos Engineering Demo using Chaostoolkit - Litmus Chaos - Chaos Mesh

Chaos Engineering is the discipline to help proactively improve reliability of systems throught "experiments".

This demo shows how to run Chaos Engineering experiments against an app called Sock-Shop using Chaostoolkit - Litmus Chaos - Chaos Mesh

- [Sock Shop](https://microservices-demo.github.io/)

- [Chaos Toolkit](https://chaostoolkit.org/)

- [Litmus Chaos](https://litmuschaos.io/)

- [Chaos Mesh](https://chaos-mesh.org/)

## Environment Overview

This demo runs against a Kubernetes cluster. For this demo you can use Minikube.
On the client side you need Python 3.5 or above.


### Minikube Instalation

[Link](https://minikube.sigs.k8s.io/docs/start/) to install Minikube

Suggested settings to create the cluster:
```
minikube start --cni=cilium --memory=8192 --cpus=4
```

### Clone this repo
```
git clone https://github.com/hivp/chaos-engineering-demo.git

cd chaos-engineering-demo
```

### Install the Demo App

Install the demo app Sock-Shop. 

```
kubectl apply -f manifests/sock-shop.yaml
```

Verifiy the Sock Shop app is running:
```
kubectl get pods -n sock-shop
```

Access to Sock Shop web:
```
kubectl port-forward svc/front-end -n sock-shop 30001:80

http://localhost:30001
```

![imagen](https://user-images.githubusercontent.com/11853819/114205406-44fb6600-9928-11eb-9849-be6a40d2dc3e.png)



### Grafana and Prometheus

```
kubectl apply -f manifests/monitoring/grafana_prometheus.yaml
```

Once installed, you can review the services running after a couple of minutes:
```
kubectl -n monitoring get all
```

Access to Grafana (admin/admin):
```
kubectl port-forward -n monitoring svc/grafana 3000

http://localhost:3000
```

Add a data source as seen below:

![imagen](https://user-images.githubusercontent.com/11853819/114202777-88a0a080-9925-11eb-9c46-f82035214695.png)


Import grafana dashboard definition (manifests/monitoring/grafana-dashboard.json)

![imagen](https://user-images.githubusercontent.com/11853819/114203229-f6e56300-9925-11eb-997c-0375af7bbf41.png)


Go to Sock-Shop Performance Dashboard

![imagen](https://user-images.githubusercontent.com/11853819/114203505-3b70fe80-9926-11eb-8ba8-f35db7b97843.png)
![imagen](https://user-images.githubusercontent.com/11853819/114203831-83902100-9926-11eb-8213-a074d291f817.png)



### Install Litmus Chaos Infrastructue

Install the Litmus Chaos operator and CRDs:
```
kubectl apply -f https://litmuschaos.github.io/litmus/litmus-operator-v1.13.2.yaml
```

Install the litmus admin serviceaccount for centralized / admin-mode of chaos execution
```
kubectl apply -f https://litmuschaos.github.io/litmus/litmus-admin-rbac.yaml
```

Install the chaos experiments in litmus namespace
```
kubectl apply -f https://hub.litmuschaos.io/api/chaos/1.13.2?file=charts/generic/experiments.yaml -n litmus
```

Install the chaos experiment metrics exporter and chaos event exporter
```
kubectl apply -f manifests/monitoring/chaos-exporter.yaml
```

Verify the Litmus components:
```
kubectl get pods -n litmus
kubectl get crds | grep chaos
kubectl api-resources | grep chaos
```

### Install Chaos Toolkit and its dependencies
The [Chaos Toolkit](https://chaostoolkit.org/) is a Chaos Engineering automation framework, open source and written in Python. You should be able to install it as follows (according to your local machine use pip or pip3):

```
pip install chaostoolkit
```

You can verify it is now available by running:
```
chaos info core
```

Install the extensions
```
pip install chaostoolkit-prometheus chaostoolkit-kubernetes chaostoolkit-addons jsonpath2 chaostoolkit-reporting
```

You can verify the extensions:
```
chaos info extensions
```

### Install Chaos Mesh
```
curl -sSL https://mirrors.chaos-mesh.org/v1.1.2/install.sh | bash
```

See all its services running
```
kubectl -n chaos-testing get all
```

Access to Chaos Mesh dashboard
```
kubectl port-forward -n chaos-testing svc/chaos-dashboard 2333

http://localhost:2333
```

![imagen](https://user-images.githubusercontent.com/11853819/114204032-ba663700-9926-11eb-9a8f-e61a8fd66f3a.png)



## Define a Chaos Experiment Card (Card thanks to [Gremlin](https://www.gremlin.com/) team)

As a practice of the Chaos Engineering process we can define the Experiment Card with the hypothesis and conditions:

![imagen](https://user-images.githubusercontent.com/11853819/114228967-34a5b400-9945-11eb-999f-26343cc5ff08.png)


## Run the Latency Experiment with Litmus Chaos

Latency of 500ms during 60 seconds will be applied to catalogue app
```
kubectl apply -f manifests/catalogue-latency.yaml
```

Verify execution of the chaos latency experiment
```
kubectl describe chaosengine -n litmus catalogue-latency 
```

Verify that Catalogue of Sock Shop app is being impacted, we need to investigate.
![imagen](https://user-images.githubusercontent.com/11853819/114230008-96b2e900-9946-11eb-9d80-5d8b64c8fb4f.png)

Observe that the impact of chaos injection into Catalogue increases latency
![imagen](https://user-images.githubusercontent.com/11853819/114231620-a9c6b880-9948-11eb-8c4f-3e1f89741d92.png)


Update the Experiment Card 

![imagen](https://user-images.githubusercontent.com/11853819/114230954-c6162580-9947-11eb-8cc4-070259bcb079.png)


## Run the Latency Experiment with Chaostoolkit and Chaos Mesh

Inside chaostoolkit folder
```
chaos run --rollback-strategy=always experiment-latency.json
```

```
[2021-04-09 17:02:52 INFO] Validating the experiment's syntax
[2021-04-09 17:02:53 INFO] Experiment looks valid
[2021-04-09 17:02:53 INFO] Running experiment: We can tolerate a small latency from internal services
[2021-04-09 17:02:53 INFO] Steady-state strategy: default
[2021-04-09 17:02:53 INFO] Rollbacks strategy: always
[2021-04-09 17:02:53 INFO] Steady state hypothesis: n/a
[2021-04-09 17:02:53 INFO] Probe: front-must-respond-ok
[2021-04-09 17:02:53 INFO] Steady state hypothesis is met!
[2021-04-09 17:02:53 INFO] Playing your experiment's method now...
[2021-04-09 17:02:53 INFO] Pausing before next activity for 2s...
[2021-04-09 17:02:55 INFO] Action: inject-latency
[2021-04-09 17:02:55 INFO] Pausing after activity for 10s...
[2021-04-09 17:03:05 INFO] Steady state hypothesis: n/a
[2021-04-09 17:03:05 INFO] Probe: front-must-respond-ok
[2021-04-09 17:03:10 ERROR]   => failed: activity took too long to complete
[2021-04-09 17:03:10 WARNING] Probe terminated unexpectedly, so its tolerance could not be validated
[2021-04-09 17:03:10 CRITICAL] Steady state probe 'front-must-respond-ok' is not in the given tolerance so failing this experiment
[2021-04-09 17:03:10 WARNING] Rollbacks were explicitly requested to be played
[2021-04-09 17:03:10 INFO] Let's rollback...
[2021-04-09 17:03:10 INFO] Rollback: remove-latency
[2021-04-09 17:03:10 INFO] Action: remove-latency
[2021-04-09 17:03:10 INFO] Experiment ended with status: deviated
[2021-04-09 17:03:10 INFO] The steady-state has deviated, a weakness may have been discovered
```

Results are the same with Litmus Chaos and Chaostoolkit. Latency is observed in Grafana and Sock-Shop app is impacted.

Reports can be generated from the output journal.json 
```
chaos report --export-format=pdf journal.json report.pdf
```

Chaos Mesh dashboard can be accesed to review the experiments:
![imagen](https://user-images.githubusercontent.com/11853819/114242796-7b050e00-9959-11eb-9273-2372ef6c4ec3.png)


## Conclusions

- Experiments can be executed using Litmus Chaos or Chaostoolkit + Chaos Mesh , or both
- Results have to be observed to improve resilience
- In this demo, a weakness may have been discovered in Catalogue of Sock-App app with 500ms latency

## Notes
- Chaos Mesh has some problems to be used with Public Cloud K8s services, such as AKS, so Litmus is the option


## To-Do-List
- Improve access to Sock-shop, Grafana and Chaos Mesh portal, using Ingress or Traefik instead of port-forward.

## References
- Litmus Demo https://github.com/litmuschaos/litmus/tree/master/demo/sample-applications/sock-shop
- Gremlin https://www.gremlin.com/
- Sock-Shop demo app https://microservices-demo.github.io/
- Learning Chaos Engineering - Russ Miles [Book](https://www.amazon.com/Learning-Chaos-Engineering-Discovering-Experimentation/dp/1492051004/ref=sr_1_2?crid=1J401903WAESN&dchild=1&keywords=chaos+engineering&qid=1616421775&sprefix=chaos+engi%2Caps%2C349&sr=8-2)
