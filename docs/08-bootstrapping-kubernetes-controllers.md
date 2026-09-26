# Развёртывание control plane Kubernetes

В этой лабораторной работе вы развернёте control plane Kubernetes. На машине `server` будут установлены следующие компоненты: Kubernetes API Server, Scheduler и Controller Manager.

## Предварительные требования

Подключитесь к `jumpbox` и скопируйте бинарные файлы Kubernetes, а также systemd unit-файлы на машину `server`:

```bash
scp \
  downloads/controller/kube-apiserver \
  downloads/controller/kube-controller-manager \
  downloads/controller/kube-scheduler \
  downloads/client/kubectl \
  units/kube-apiserver.service \
  units/kube-controller-manager.service \
  units/kube-scheduler.service \
  configs/kube-scheduler.yaml \
  configs/kube-apiserver-to-kubelet.yaml \
  root@server:~/
```

Команды этой лабораторной работы необходимо выполнять на машине `server`. Подключитесь к ней по SSH. Пример:

```bash
ssh root@server
```

## Подготовка control plane Kubernetes

Создайте каталог конфигурации Kubernetes:

```bash
mkdir -p /etc/kubernetes/config
```

### Установка бинарных файлов компонентов управления Kubernetes

Установите бинарные файлы Kubernetes:

```bash
{
  mv kube-apiserver \
    kube-controller-manager \
    kube-scheduler kubectl \
    /usr/local/bin/
}
```

### Настройка Kubernetes API Server

```bash
{
  mkdir -p /var/lib/kubernetes/

  mv ca.crt ca.key \
    kube-api-server.key kube-api-server.crt \
    service-accounts.key service-accounts.crt \
    encryption-config.yaml \
    /var/lib/kubernetes/
}
```

Создайте systemd unit-файл `kube-apiserver.service`:

```bash
mv kube-apiserver.service \
  /etc/systemd/system/kube-apiserver.service
```

### Настройка Kubernetes Controller Manager

Переместите kubeconfig `kube-controller-manager` в требуемое расположение:

```bash
mv kube-controller-manager.kubeconfig /var/lib/kubernetes/
```

Создайте systemd unit-файл `kube-controller-manager.service`:

```bash
mv kube-controller-manager.service /etc/systemd/system/
```

### Настройка Kubernetes Scheduler

Переместите kubeconfig `kube-scheduler` в требуемое расположение:

```bash
mv kube-scheduler.kubeconfig /var/lib/kubernetes/
```

Создайте конфигурационный файл `kube-scheduler.yaml`:

```bash
mv kube-scheduler.yaml /etc/kubernetes/config/
```

Создайте systemd unit-файл `kube-scheduler.service`:

```bash
mv kube-scheduler.service /etc/systemd/system/
```

### Запуск сервисов control plane

```bash
{
  systemctl daemon-reload

  systemctl enable kube-apiserver \
    kube-controller-manager kube-scheduler

  systemctl start kube-apiserver \
    kube-controller-manager kube-scheduler
}
```

> Подождите до 10 секунд, пока Kubernetes API Server полностью инициализируется.

Проверить, активны ли компоненты control plane, можно с помощью команды `systemctl`. Например, чтобы проверить, что `kube-apiserver` полностью инициализирован и находится в активном состоянии, выполните:

```bash
systemctl is-active kube-apiserver
```

Для более подробной проверки состояния, включая дополнительную информацию о процессе и сообщения журналов, используйте команду `systemctl status`:

```bash
systemctl status kube-apiserver
```

Если возникли ошибки или необходимо просмотреть журналы любого компонента control plane, используйте команду `journalctl`. Например, для просмотра журналов `kube-apiserver` выполните:

```bash
journalctl -u kube-apiserver
```

### Проверка

На этом этапе компоненты control plane Kubernetes должны быть запущены. Проверьте это с помощью утилиты командной строки `kubectl`:

```bash
kubectl cluster-info \
  --kubeconfig admin.kubeconfig
```

```text
Kubernetes control plane is running at [https://127.0.0.1:6443](https://127.0.0.1:6443)
```

## RBAC для авторизации Kubelet

В этом разделе вы настроите разрешения RBAC, позволяющие Kubernetes API Server обращаться к Kubelet API на каждом рабочем узле. Доступ к Kubelet API требуется для получения метрик и логов, а также выполнения команд в pod'ах.

> В этом руководстве для Kubelet устанавливается флаг `--authorization-mode` со значением `Webhook`. Режим Webhook использует API [SubjectAccessReview](https://kubernetes.io/docs/reference/access-authn-authz/authorization/#checking-api-access) для определения прав доступа.

Команды из этого раздела влияют на весь кластер, поэтому их необходимо выполнить на машине `server` только один раз.

```bash
ssh root@server
```

Создайте [ClusterRole](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#role-and-clusterrole) `system:kube-apiserver-to-kubelet` с разрешениями для доступа к Kubelet API и выполнения большинства распространённых задач по управлению pod'ами:

```bash
kubectl apply -f kube-apiserver-to-kubelet.yaml \
  --kubeconfig admin.kubeconfig
```

### Проверка

На этом этапе control plane Kubernetes должен быть запущен. Выполните следующие команды на машине `jumpbox`, чтобы убедиться, что он работает.

Отправьте HTTP-запрос для получения информации о версии Kubernetes:

```bash
curl --cacert ca.crt \
  [https://server.kubernetes.local:6443/version](https://server.kubernetes.local:6443/version)
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

Далее: [Развёртывание рабочих узлов Kubernetes](09-bootstrapping-kubernetes-workers.md)
