# Настройка SOPS

## Что такое SOPS

SOPS (Secrets OPerationS) - инструмент для шифрования файлов с секретами в Git репозиториях.

## Установка

```bash
# macOS
brew install sops

# Linux
curl -LO https://github.com/mozilla/sops/releases/latest/download/sops-v3.8.1.linux.amd64
sudo mv sops-v3.8.1.linux.amd64 /usr/local/bin/sops
sudo chmod +x /usr/local/bin/sops
```

## Конфигурация

### Настройки шифрования

Файл `.sops.yaml`:

```yaml
creation_rules:
  - path_regex: values\.yaml$
    pgp: <PGP-ключи>
    encrypted_regex: '^(<ключи секретов в yaml>)$'
```

## Использование

### Шифрование файла

```bash
sops -e -i charts/<my-chart>>/values.yaml
```

### Редактирование зашифрованного файла в vi-редакторе

```bash
sops charts/<my-chart>>/values.yaml
```

### Просмотр расшифрованного содержимого

```bash
sops -d charts/<my-chart>>/values.yaml
```

### Использование с Helm

```bash
helm install <my-chart> ./charts/<my-chart> \
  --values <(sops -d ./charts/<my-chart>>/values.yaml) \
  --namespace <my-chart-ns>
```
