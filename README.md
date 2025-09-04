# OMD Speedtest

Проект для развертывания Kubernetes кластера с приложением [speedtest](https://github.com/librespeed/speedtest)

## Структура проекта

```bash
├── docs/                   # Документация
│   ├── proxmox-csi.md      # Настройка CSI драйвера
│   ├── sops-setup.md       # Конфигурация SOPS
│   └── troubleshooting.md  # Решение проблем
├── charts/proxmox-csi/     # Helm чарт для CSI
├── manifests/              # Kubernetes манифесты
└── .pre-commit-config.yaml # Хуки для безопасности
```

## Быстрый старт

### 1. Настройка SOPS

См. подробные инструкции: [docs/sops-setup.md](docs/sops-setup.md)

### 2. Подготовка CSI драйвера

См. подробные инструкции: [docs/proxmox-csi.md](docs/proxmox-csi.md)

```bash
# Настройка секретов
sops charts/proxmox-csi/values.yaml

# Установка
helm install proxmox-csi ./charts/proxmox-csi \
  --values <(sops -d ./charts/proxmox-csi/values.yaml) \
  --namespace proxmox-csi \
  --create-namespace
```
