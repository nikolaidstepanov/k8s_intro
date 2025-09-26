# Практика: Введение в Kubernetes (через Minikube)

Этот документ содержит шаги для live-сессии по работе с Kubernetes. Все команды можно выполнять локально через Minikube.

---

## 0) Запуск кластера

```bash
# поднять кластер (если minikube установлен)
minikube start
minikube status
kubectl cluster-info
kubectl get nodes -o wide

# если нужен драйвер (например docker):
minikube start --driver=docker
```

---

## 1) Pod + манифесты

`pod.yaml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo-nginx
  labels: { app: demo-nginx }
spec:
  containers:
    - name: nginx
      image: nginx:1.25
      ports: [{ containerPort: 80 }]
```

```bash
kubectl apply -f pod.yaml
kubectl get pods
kubectl describe pod demo-nginx
```

---

## 2) Быстрый тур по kubectl

```bash
kubectl logs demo-nginx                    # логи
kubectl exec -it demo-nginx -- sh          # зайти внутрь
# внутри пода:
#   wget -qO- localhost
exit
kubectl get pod demo-nginx -o wide
kubectl get events --sort-by=.lastTimestamp
```

---

## 3) Service + доступ

`service.yaml` (NodePort)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: demo-nginx-svc
spec:
  type: NodePort
  selector: { app: demo-nginx }
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

```bash
kubectl apply -f service.yaml
kubectl get svc demo-nginx-svc

# Вариант 1: port-forward (быстро и локально)
kubectl port-forward pod/demo-nginx 8080:80
# открыть http://localhost:8080

# Вариант 2: NodePort через Minikube
minikube service demo-nginx-svc --url
# открыть выданный URL
```

---

## 4) Helm Charts

```bash
# установить helm (если нет)
# mac:  brew install helm
# linux: curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

helm version
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# развернуть redis
helm install my-redis bitnami/redis
kubectl get pods -l app.kubernetes.io/name=redis
helm status my-redis

# показать отличие: одна команда развернула кучу ресурсов (svc, sts, cm, pvc)
helm uninstall my-redis   # (по желанию) очистка
```

---

## 5) Namespaces

```bash
kubectl create ns dev
kubectl create ns qa

# развернём Pod в dev
kubectl apply -n dev -f pod.yaml
kubectl get pods -A | grep demo-nginx

# сервис в qa (сначала Pod)
kubectl apply -n qa -f pod.yaml
kubectl apply -n qa -f service.yaml

kubectl get pods --all-namespaces
```

> одинаковые имена ресурсов могут жить в разных `ns`.

---

## 6) Lens (визуализация)

1. Открыть Lens → **Add Cluster** → выбрать `~/.kube/config`.
2. Подключить Minikube.
3. Визуально посмотреть Pods, Services, Events в разных namespaces.
4. Открыть логи Pod через Lens.

---

## Очистка (в конце)

```bash
kubectl delete -f service.yaml
kubectl delete -f pod.yaml
kubectl delete ns dev qa
helm uninstall my-redis || true

minikube stop
# minikube delete    # полностью удалить кластер (по желанию)
```

---
