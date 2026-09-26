# Развёртывание рабочих узлов Kubernetes

В этой лабораторной работе вы развернёте два рабочих узла Kubernetes. Будут установлены следующие компоненты: [runc](https://github.com/opencontainers/runc), [плагины контейнерной сети](https://github.com/containernetworking/cni), [containerd](https://github.com/containerd/containerd), [kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet) и [kube-proxy](https://kubernetes.io/docs/concepts/cluster-administration/proxies).

## Предварительные требования

Команды этого раздела необходимо выполнять с `jumpbox`.

Скопируйте бинарные файлы Kubernetes и systemd unit-файлы на каждый рабочий узел:

```bash
for HOST in node-0 node-1; do
  SUBNET=$(grep ${HOST} machines.txt | cut -d " " -f 4)
  sed "s|SUBNET|$SUBNET|g" \
    configs/10-bridge.conf > 10-bridge.conf

  sed "s|SUBNET|$SUBNET|g" \
    configs/kubelet-config.yaml > kubelet-config.yaml

  scp 10-bridge.conf kubelet-config.yaml \
  root@${HOST}:~/
done
```

```bash
for HOST in node-0 node-1; do
  scp \
    downloads/worker/* \
    downloads/client/kubectl \
    configs/99-loopback.conf \
    configs/containerd-config.toml \
    configs/kube-proxy-config.yaml \
    units/containerd.service \
    units/kubelet.service \
    units/kube-proxy.service \
    root@${HOST}:~/
done
```

```bash
for HOST in node-0 node-1; do
  scp \
    downloads/cni-plugins/* \
    root@${HOST}:~/cni-plugins/
done
```

Команды следующего раздела необходимо выполнить на каждом рабочем узле: `node-0` и `node-1`. Подключитесь к рабочему узлу по SSH. Пример:

```bash
ssh root@node-0
```

## Настройка рабочего узла Kubernetes

Установите зависимости операционной системы:

```bash
{
  apt-get update
  apt-get -y install socat conntrack ipset kmod
}
```

> Бинарный файл `socat` обеспечивает поддержку команды `kubectl port-forward`.

### Отключение swap

Kubernetes ограниченно поддерживает использование swap-памяти, поскольку при задействованном swap сложно гарантировать и учитывать потребление памяти pod'ами.

Проверьте, отключён ли swap:

```bash
swapon --show
```

Если вывод пустой, swap отключён. Если swap включён, выполните следующую команду, чтобы отключить его немедленно:

```bash
swapoff -a
```

> Чтобы swap оставался отключённым после перезагрузки, обратитесь к документации вашего дистрибутива Linux.

Создайте каталоги для установки:

```bash
mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes
```

Установите бинарные файлы рабочего узла:

```bash
{
  mv crictl kube-proxy kubelet runc \
    /usr/local/bin/
  mv containerd containerd-shim-runc-v2 containerd-stress /bin/
  mv cni-plugins/* /opt/cni/bin/
}
```

### Настройка сети CNI

Создайте файл конфигурации сети `bridge`:

```bash
mv 10-bridge.conf 99-loopback.conf /etc/cni/net.d/
```

Чтобы сетевой трафик, проходящий через CNI-сеть `bridge`, обрабатывался `iptables`, загрузите и настройте модуль ядра `br-netfilter`:

```bash
{
  modprobe br-netfilter
  echo "br-netfilter" >> /etc/modules-load.d/modules.conf
}
```

```bash
{
  echo "net.bridge.bridge-nf-call-iptables = 1" \
    >> /etc/sysctl.d/kubernetes.conf
  echo "net.bridge.bridge-nf-call-ip6tables = 1" \
    >> /etc/sysctl.d/kubernetes.conf
  sysctl -p /etc/sysctl.d/kubernetes.conf
}
```

### Настройка containerd

Установите конфигурационные файлы `containerd`:

```bash
{
  mkdir -p /etc/containerd/
  mv containerd-config.toml /etc/containerd/config.toml
  mv containerd.service /etc/systemd/system/
}
```

### Настройка Kubelet

Создайте конфигурационный файл `kubelet-config.yaml`:

```bash
{
  mv kubelet-config.yaml /var/lib/kubelet/
  mv kubelet.service /etc/systemd/system/
}
```

### Настройка Kubernetes Proxy

```bash
{
  mv kube-proxy-config.yaml /var/lib/kube-proxy/
  mv kube-proxy.service /etc/systemd/system/
}
```

### Запуск сервисов рабочего узла

```bash
{
  systemctl daemon-reload
  systemctl enable containerd kubelet kube-proxy
  systemctl start containerd kubelet kube-proxy
}
```

Проверьте, что сервис kubelet работает:

```bash
systemctl is-active kubelet
```

```text
active
```

Перед переходом к следующему разделу обязательно выполните шаги этого раздела на каждом рабочем узле: `node-0` и `node-1`.

## Проверка

Выполните следующие команды на машине `jumpbox`.

Выведите список зарегистрированных узлов Kubernetes:

```bash
ssh root@server \
  "kubectl get nodes \
  --kubeconfig admin.kubeconfig"
```

```text
NAME     STATUS   ROLES    AGE    VERSION
node-0   Ready    <none>   1m     v1.32.3
node-1   Ready    <none>   10s    v1.32.3
```

Далее: [Настройка kubectl для удалённого доступа](10-configuring-kubectl.md)
