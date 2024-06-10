# minikube-squid-proxy

## Easy proxy pod.
1. Create file proxy-pod.yaml
```
# proxy-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: proxy-pod
spec:
  containers:
  - name: squid
    image: sameersbn/squid:3.5.27-2
    ports:
    - containerPort: 3128
      protocol: TCP
```
2. Usage
```
 1. kubectl apply -f proxy-pod.yaml
 2. kubectl get pods (Make sure the pod is running)
 3. kubectl port-forward pod/proxy-pod 20000:3128
 20000 is forward port, you can change any port you like .
 4. In chrome, use SwitchOmega to add proxy 127.0.0.1:20000 and then do your test
```
3. Advantage-Disavantage
** Advantage
   This is easy to build and usage.
** Disavantage
   Because it use forward traffic, so some time it has lost connect and make your internet discontinuity.

## Build proxy-pod with proxy-service.
** Describe:
We will build a service on kubernetes to handle get traffic coming and return result. This server help proxy-pod stronger connected.
1. Create proxy-server
```
# proxy-server.yaml
apiVersion: v1
kind: Service
metadata:
  name: proxy-service
spec:
  type: NodePort
  ports:
  - port: 3128
    targetPort: 3128
    nodePort: 32000 # You can choose any available port in the range 30000-32767
  selector:
    app: proxy
```

2. Create proxy-pod
```
# proxy-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: proxy-pod
  labels:
    app: proxy
spec:
  containers:
  - name: squid
    image: sameersbn/squid:3.5.27-2
    ports:
    - containerPort: 3128
      protocol: TCP
    volumeMounts:
    - name: squid-config
      mountPath: /etc/squid/squid.conf
      subPath: squid.conf
  volumes:
  - name: squid-config
    configMap:
      name: squid-config
```
We have to create file squid.conf to config the squid access.
```
http_access allow all
http_port 3128
```
3. Run
** Node: if you have run proxy-pod before, you should delete it to make it work correctly (kubectl delete pod proxy-pod)
```
kubectl apply -f proxy-server.yaml
kubectl apply -f proxy-pod.yaml

--Do some check and test:
kubectl get pods
kubectl logs proxy-pod
kubectl describe service proxy-service

kubectl run test-pod --rm -it --image=appropriate/curl -- sh
curl 172.24.245.196:32000
```
   
