
## Instala K3d
wget -q -O - https://raw.githubusercontent.com/rancher/k3d/main/install.sh | bash

## Cria clustes
k3d cluster create -p "8000:30000@loadbalancer" --agents 2

## Instala Istio
curl -L https://istio.io/downloadIstio | sh -

## istioctl fica na pasta bin

## instala istio no cluster
istioctl install

## instala plugins
https://raw.githubusercontent.com/istio/istio/release-1.12/samples/addons/prometheus.yaml

kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.12/samples/addons/kiali.yaml

kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.12/samples/addons/jaeger.yaml

kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.12/samples/addons/grafana.yaml

## Roda kiali
istioctl dashboard kiali

## cria label
kubectl label namespace default istio-injection=enabled

## POD do fortio:
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.12/samples/httpbin/sample-client/fortio-deploy.yaml

## Nome do POD
export FORTIO_POD=$(kubectl get pods -l app=fortio -o 'jsonpath={.items[0].metadata.name}')

## Executa FORTIO
kubectl exec "$FORTIO_POD" -c fortio -- /usr/bin/fortio load -c 2 -qps 0 -t 200s -loglevel Warning http://nginx-service:8000
**** -timeout 20s para fault-injection

## Executa FORTIO - Circuit Breaker
kubectl exec "$FORTIO_POD" -c fortio -- /usr/bin/fortio load -c 2 -qps 0 -n 20 -loglevel Warning http://servicex-service

## Edita svc para mudar porta redirecionada da 80
kubectl edit svc istio-ingressgateway -n istio-system
