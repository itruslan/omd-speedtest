# OMD Speedtest

Проект для развертывания LibreSpeed speedtest приложения в Kubernetes кластере с MySQL базой данных.

## Структура проекта

```bash
├── charts/
│   ├── proxmox-csi/          # Helm чарт для CSI
│   ├── metallb/              # Helm чарт для MetalLB LoadBalancer
│   └── nginx-ingress/        # Helm чарт для Ingress Controller
├── manifests/
│   ├── db/                   # MySQL манифесты
│   └── application/          # LibreSpeed манифесты
│   └── metallb/              # MetalLB манифесты
└── .pre-commit-config.yaml   # Pre-commit хуки
└── .sops.yaml                # SOPS конфиг
```

## Быстрый старт

### 1. Подготовка кластера (single-node)

```bash
# Удаление control-plane taint
kubectl taint nodes --all node-role.kubernetes.io/control-plane-

# Удаление лейбла exclude-from-external-load-balancers (для работы MetalLB)
kubectl label node omd-1 node.kubernetes.io/exclude-from-external-load-balancers-

# Настройка topology labels (zone=pve-1 означает что ВМ развернута в proxmox кластере на ноде pve-1)
kubectl label node omd-1 topology.kubernetes.io/region=homelab topology.kubernetes.io/zone=pve-1

# Установка CSI драйвера Proxmox
helm dependency update charts/proxmox-csi/
helm install proxmox-csi ./charts/proxmox-csi \
  --values <(sops -d ./charts/proxmox-csi/values.yaml) \
  --namespace proxmox-csi \
  --create-namespace

# Установка MetalLB LoadBalancer
helm dependency update charts/metallb/
helm install metallb ./charts/metallb \
  --namespace metallb-system \
  --create-namespace

# Установка nginx-ingress-controller
helm dependency update charts/nginx-ingress/
helm install nginx-ingress ./charts/nginx-ingress \
  --namespace ingress-nginx \
  --create-namespace
```

### 2. Развертывание базы данных

```bash
kubectl apply \
-f manifests/db/00-mysql-namespace.yaml \
-f manifests/db/mysql-configmap.yaml \
-f manifests/db/mysql-headless.yaml \
-f manifests/db/mysql-pvc.yaml \
-f manifests/db/mysql-service.yaml \
-f manifests/db/mysql.yaml
sops -d manifests/db/mysql-secret.yaml | kubectl apply -f -
```

### 3. Развертывание приложения

```bash
kubectl apply \
-f manifests/application/00-librespeed-namespace.yaml \
-f manifests/application/librespeed-env.yaml \
-f manifests/application/librespeed-ingress.yaml \
-f manifests/application/librespeed-servers.yaml \
-f manifests/application/librespeed-service.yaml \
-f manifests/application/librespeed.yaml
sops -d manifests/application/librespeed-secret.yaml | kubectl apply -f -
```

## Доступ к приложению

- **Host-based URL**: <http://speedtest.omd.itruslan.ru>
- **Статистика**: <http://speedtest.omd.itruslan.ru/results/stats.php> (пароль: `speedtest123`)
