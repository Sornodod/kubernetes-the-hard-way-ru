# Настройка kubectl для удалённого доступа

В этой лабораторной работе вы создадите kubeconfig-файл для утилиты командной строки `kubectl` на основе учётных данных пользователя `admin`.

> Выполняйте команды из этой лабораторной работы на машине `jumpbox`.

## Конфигурационный файл Kubernetes для admin

Для каждого kubeconfig требуется Kubernetes API Server, к которому будет выполняться подключение.

Вы должны иметь возможность отправить запрос к `server.kubernetes.local` благодаря DNS-записи в `/etc/hosts`, созданной в одной из предыдущих лабораторных работ.

```bash
curl --cacert ca.crt \
  https://server.kubernetes.local:6443/version
```

```text
{
  "major": "1",
  "minor": "32",
  "gitVersion": "v1.32.3",
  "gitCommit": "32cc146f75aad04beaaa245a7157eb35063a9f99",
  "gitTreeState": "clean",
  "buildDate": "2025-03-11T19:52:21Z",
  "goVersion": "go1.23.6",
  "compiler": "gc",
  "platform": "linux/arm64"
}
```

Создайте kubeconfig-файл, подходящий для аутентификации от имени пользователя `admin`:

```bash
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://server.kubernetes.local:6443

  kubectl config set-credentials admin \
    --client-certificate=admin.crt \
    --client-key=admin.key

  kubectl config set-context kubernetes-the-hard-way \
    --cluster=kubernetes-the-hard-way \
    --user=admin

  kubectl config use-context kubernetes-the-hard-way
}
```

В результате выполнения команды выше kubeconfig-файл будет создан в расположении по умолчанию — `~/.kube/config`, которое использует утилита командной строки `kubectl`. Это также означает, что вы сможете запускать команду `kubectl`, не указывая конфигурационный файл явно.

## Проверка

Проверьте версию удалённого кластера Kubernetes:

```bash
kubectl version
```

```text
Client Version: v1.32.3
Kustomize Version: v5.5.0
Server Version: v1.32.3
```

Выведите список узлов в удалённом кластере Kubernetes:

```bash
kubectl get nodes
```

```text
NAME     STATUS   ROLES    AGE    VERSION
node-0   Ready    <none>   10m   v1.32.3
node-1   Ready    <none>   10m   v1.32.3
```

Далее: [Настройка маршрутов сети pod'ов](11-pod-network-routes.md)
