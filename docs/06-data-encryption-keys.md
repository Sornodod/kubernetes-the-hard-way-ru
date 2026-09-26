# Создание конфигурации и ключа шифрования данных

Kubernetes хранит различные данные, включая состояние кластера, конфигурации приложений и секреты. Kubernetes поддерживает возможность [шифрования](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data) данных кластера при хранении.

В этой лабораторной работе вы создадите ключ шифрования и [конфигурацию шифрования](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#understanding-the-encryption-at-rest-configuration), подходящую для шифрования Kubernetes Secrets.

## Ключ шифрования

Создайте ключ шифрования:

```bash
export ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
```

## Файл конфигурации шифрования

Создайте файл конфигурации шифрования `encryption-config.yaml`:

```bash
envsubst < configs/encryption-config.yaml \
  > encryption-config.yaml
```

Скопируйте файл конфигурации шифрования `encryption-config.yaml` на каждый экземпляр control plane:

```bash
scp encryption-config.yaml root@server:~/
```

Далее: [Развёртывание кластера etcd](07-bootstrapping-etcd.md)
