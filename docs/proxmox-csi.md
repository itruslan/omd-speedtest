# Настройка Proxmox CSI

## Требования

- Kubernetes кластер
- Настроен SOPS с PGP ключом
- Установлены helm и kubectl

## Шаги установки

### 1. Настройка секретов

Отредактируйте значения с вашими данными Proxmox:

```bash
sops charts/proxmox-csi/values.yaml
```

### 2. Скачивание зависимостей

Обновите Helm зависимости:

```bash
helm dependency update charts/proxmox-csi/
```

### 3. Настройка topology labels

Примените labels на ноды для topology awareness:

```bash
kubectl label node omd-1 topology.kubernetes.io/region=homelab topology.kubernetes.io/zone=pve-2
```

### 4. Установка CSI плагина

Установите через Helm:

```bash
helm install proxmox-csi ./charts/proxmox-csi \
  --values <(sops -d ./charts/proxmox-csi/values.yaml) \
  --namespace proxmox-csi \
  --create-namespace
```

### 5. Проверка установки

Проверьте что поды запущены:

```bash
kubectl get pods -n proxmox-csi
kubectl get storageclass
kubectl get pv
```

## Конфигурация

### Topology Labels

Для корректной работы CSI с топологией на ноды применяются labels:

```bash
kubectl label node omd-1 topology.kubernetes.io/region=homelab topology.kubernetes.io/zone=pve-2
```

Где:

- `omd-1` - имя вашей ноды
- `region=homelab` - регион для topology awareness
- `zone=pve-2` - зона (соответствует Proxmox ноде)
