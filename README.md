# haproxy-ingress-controller
A quick setup of a HAProxy Ingress Controller for Kubernetes

## Prerequisites
An ingress controller is required to satisfy an ingress, simply creating the resource will have no effect.

GCE/Google Kubernetes Engine deploys an ingress controller on the master. For other environments, you may need to deploy an [ingress controller](https://kubernetes.io/docs/concepts/services-networking/ingress/#ingress-controllers).

Assuming you have Kubernetes and Minikube (or Docker for Mac) installed, follow these steps to set up the HAProxy Ingress Controller on your local Minikube cluster.

## Installation Guide
- Enable the ingress add-on for Minikube
```shell
minikube addons enable ingress
```
- Create a TLS secret named tls-secret to be used as default TLS certificate
```shell
openssl req \
  -x509 -newkey rsa:2048 -nodes -days 365 \
  -keyout tls.key -out tls.crt -subj '/CN=sslexample.foo.bar.com'
```
```shell
kubectl create secret tls tls-secret --cert=tls.crt --key=tls.key
```
- Create the “mandatory” resources for HAProxy Ingress in your cluster which includes these elements:
  * Namespace
  * ServiceAccount
  * ClusterRole
  * Role
  * RoleBinding
  * ClusterRoleBinding
  * ConfigMap
  * Service serving as a default fallback
```shell
kubectl apply -f https://raw.githubusercontent.com/anouarchattouna/haproxy-ingress-controller/master/haproxy-mandatory.yaml
```
## Creating a Kubernetes Ingress
First, let's create two services to demonstrate how the Ingress routes our request.
```shell
kubectl apply -f https://raw.githubusercontent.com/anouarchattouna/haproxy-ingress-controller/master/echoserver.yaml
```
```shell
kubectl apply -f https://raw.githubusercontent.com/anouarchattouna/haproxy-ingress-controller/master/http-echo.yaml
```
Now, declare an Ingress to route requests to /appone to the first service, and requests to /apptwo to second service. Check out the Ingress’ rules field that declares how requests are passed along.
```shell
kubectl apply -f https://raw.githubusercontent.com/anouarchattouna/haproxy-ingress-controller/master/apps-ingress.yaml
```
Perfect! Let's check that it's working. As we re using Minikube, the cluster url is `192.168.99.100`.
```shell
curl -ki -H 'Host: sslexample.foo.bar.com' https://192.168.99.100/appone
HTTP/2 200
server: nginx/1.15.6
date: Mon, 29 Dec 2018 20:50:44 GMT
content-type: text/plain
vary: Accept-Encoding



Hostname: echoserver-7d46988f9-2b4vg

Pod Information:
	-no pod information available-

Server values:
	server_version=nginx: 1.13.3 - lua: 10008

Request Information:
	client_address=172.17.0.11
	method=GET
	real path=/appone
	query=
	request_version=1.1
	request_scheme=http
	request_uri=http://sslexample.foo.bar.com:8080/appone

Request Headers:
	accept=*/*
	host=sslexample.foo.bar.com
	user-agent=curl/7.54.0
	x-forwarded-for=192.168.99.1
	x-forwarded-host=sslexample.foo.bar.com
	x-forwarded-port=443
	x-forwarded-proto=https
	x-original-uri=/appone
	x-real-ip=192.168.99.1
	x-request-id=fce47da366dccee663a0d2e1de9c243f
	x-scheme=https

Request Body:
	-no body in request-
```
```shell
curl -ki -H 'Host: sslexample.foo.bar.com' https://192.168.99.100/apptwo'
HTTP/2 200
server: nginx/1.15.6
date: Mon, 29 Dec 2018 20:52:03 GMT
content-type: text/plain; charset=utf-8
content-length: 57
x-app-name: http-echo
x-app-version: 0.2.3

Happy learning HAProxy Ingress controller for kubernetes
```
```shell
curl -ki -H 'Host: sslexample.foo.bar.com' https://192.168.99.100
HTTP/2 404
server: nginx/1.15.6
date: Mon, 29 Dec 2018 20:52:14 GMT
content-type: text/plain; charset=utf-8
content-length: 21

default backend - 404%
```
## Credits

- [kubernetes.io](https://kubernetes.io/docs/concepts/services-networking/ingress/) 
- [jcmoraisjr/haproxy-ingress](https://github.com/jcmoraisjr/haproxy-ingress)
- [HAProxy blog post](https://www.haproxy.com/blog/haproxy_ingress_controller_for_kubernetes/)