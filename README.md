# Лабораторная работа: Микросервисный мессенджер в Kubernetes

Данный репозиторий содержит конфигурацию для развертывания микросервисного приложения (Frontend, BFF, User Service, Message Service, PostgreSQL, MinIO) в кластере Kubernetes. 

Реализованы следующие требования:
- Монтирование S3-совместимого хранилища через CSI-драйвер для загрузки файлов.
- Разделение конфигураций для окружений `dev` и `prod` с помощью Kustomize.
- Настройка правил `nodeAffinity` для распределения нагрузки между системными и прикладными узлами.
- GitOps-деплой и автоматическая синхронизация через Argo CD.

---

## 1. Подготовка кластера Minikube

Для корректной работы правил `nodeAffinity` требуется кластер минимум из двух узлов.

```bash
# Запуск кластера с 2 узлами (профиль lab-cluster)
minikube start --driver=docker --nodes 2 -p lab-cluster --memory=4096 --cpus=2

kubectl label nodes lab-cluster workload=system
kubectl label nodes lab-cluster-m02 workload=app disk=fast

# Проверка применения меток
kubectl get nodes --show-labels | grep workload
```

---

## 2. Установка компонентов инфраструктуры

### Ingress Controller
Необходим для работы манифеста Ingress:
```bash
minikube addons enable ingress
```

### S3 CSI Driver
Был выбран драйвер ch.ctrox.csi.s3-driver
```bash
helm repo add csi-s3 https://ctrox.github.io/charts
helm repo update
helm install csi-s3 csi-s3/csi-s3 --namespace kube-system
kubectl get pods -n kube-system | grep csi-s3
```


### Argo CD
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

kubectl wait --for=condition=available deployment/argocd-server -n argocd --timeout=300s
```

---

## 3. Развертывание приложения

### Через Argo CD 
   ```bash
   kubectl apply -f argocd/application.yaml
   ```
### Прямой запуск через Kustomize (для локальной отладки)
```bash
kubectl apply -k k8s/overlays/dev
```

---

## 4. Инициализация S3 Bucket в MinIO


   ```bash
   kubectl port-forward svc/minio -n messager-dev 9001:9001
   ```
1. Откройте в браузере: `http://localhost:9001`
2. Авторизуйтесь: логин `minioadmin`, пароль `minioadmin`.
3. Нажмите **Create Bucket** и задайте имя: `uploads` (строго так, как указано в `s3-csi-pv.yaml`)

---

## 5. Проверка работоспособности и эксплуатация

### 1. Проверка распределения нодов
```bash
kubectl get pods -n messager-dev -o wide
```

### 2. Проверка миграций и хранилища
```bash
kubectl get jobs -n messager-dev

kubectl get pvc -n messager-dev
```

### 3. Доступ к интерфейсу мессенджера
```bash
kubectl port-forward svc/frontend -n messager-dev 8080:80
```
Откройте в браузере: `http://localhost:8080`

### 4. Доступ к панели управления Argo CD
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo

kubectl port-forward svc/argocd-server -n argocd 8443:443
```
Откройте в браузере: `https://localhost:8443` (логин: `admin`)
