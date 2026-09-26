# Создание конфигурационных файлов Kubernetes для аутентификации

В этой лабораторной работе вы создадите [конфигурационные файлы клиентов Kubernetes](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/), обычно называемые kubeconfig. Они настраивают подключение и аутентификацию клиентов Kubernetes к API-серверам Kubernetes.

## Конфигурации аутентификации клиентов

В этом разделе вы создадите kubeconfig-файлы для `kubelet` и пользователя `admin`.

### Конфигурационный файл Kubernetes для kubelet

При создании kubeconfig-файлов для Kubelet необходимо использовать клиентский сертификат, соответствующий имени узла Kubelet. Это гарантирует, что Kubelet будет корректно авторизован с помощью [Node Authorizer](https://kubernetes.io/docs/reference/access-authn-authz/node/) Kubernetes.

> Следующие команды необходимо выполнять в той же директории, которая использовалась для создания SSL-сертификатов в лабораторной работе [Создание TLS-сертификатов](04-certificate-authority.md).

Создайте kubeconfig-файлы для рабочих узлов `node-0` и `node-1`:

```bash
for host in node-0 node-1; do
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=[https://server.kubernetes.local:6443](https://server.kubernetes.local:6443) \
    --kubeconfig=${host}.kubeconfig

  kubectl config set-credentials system:node:${host} \
    --client-certificate=${host}.crt \
    --client-key=${host}.key \
    --embed-certs=true \
    --kubeconfig=${host}.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:node:${host} \
    --kubeconfig=${host}.kubeconfig

  kubectl config use-context default \
    --kubeconfig=${host}.kubeconfig
done
```

Результат:

```text
node-0.kubeconfig
node-1.kubeconfig
```

### Конфигурационный файл Kubernetes для kube-proxy

Создайте kubeconfig-файл для сервиса `kube-proxy`:

```bash
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=[https://server.kubernetes.local:6443](https://server.kubernetes.local:6443) \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-credentials system:kube-proxy \
    --client-certificate=kube-proxy.crt \
    --client-key=kube-proxy.key \
    --embed-certs=true \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-proxy \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config use-context default \
    --kubeconfig=kube-proxy.kubeconfig
}
```

Результат:

```text
kube-proxy.kubeconfig
```

### Конфигурационный файл Kubernetes для kube-controller-manager

Создайте kubeconfig-файл для сервиса `kube-controller-manager`:

```bash
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=[https://server.kubernetes.local:6443](https://server.kubernetes.local:6443) \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=kube-controller-manager.crt \
    --client-key=kube-controller-manager.key \
    --embed-certs=true \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-controller-manager \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config use-context default \
    --kubeconfig=kube-controller-manager.kubeconfig
}
```

Результат:

```text
kube-controller-manager.kubeconfig
```

### Конфигурационный файл Kubernetes для kube-scheduler

Создайте kubeconfig-файл для сервиса `kube-scheduler`:

```bash
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=[https://server.kubernetes.local:6443](https://server.kubernetes.local:6443) \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-credentials system:kube-scheduler \
    --client-certificate=kube-scheduler.crt \
    --client-key=kube-scheduler.key \
    --embed-certs=true \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-scheduler \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config use-context default \
    --kubeconfig=kube-scheduler.kubeconfig
}
```

Результат:

```text
kube-scheduler.kubeconfig
```

### Конфигурационный файл Kubernetes для admin

Создайте kubeconfig-файл для пользователя `admin`:

```bash
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=[https://127.0.0.1:6443](https://127.0.0.1:6443) \
    --kubeconfig=admin.kubeconfig

  kubectl config set-credentials admin \
    --client-certificate=admin.crt \
    --client-key=admin.key \
    --embed-certs=true \
    --kubeconfig=admin.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=admin \
    --kubeconfig=admin.kubeconfig

  kubectl config use-context default \
    --kubeconfig=admin.kubeconfig
}
```

Результат:

```text
admin.kubeconfig
```

## Распространение конфигурационных файлов Kubernetes

Скопируйте kubeconfig-файлы `kubelet` и `kube-proxy` на машины `node-0` и `node-1`:

```bash
for host in node-0 node-1; do
  ssh root@${host} "mkdir -p /var/lib/{kube-proxy,kubelet}"

  scp kube-proxy.kubeconfig \
    root@${host}:/var/lib/kube-proxy/kubeconfig \

  scp ${host}.kubeconfig \
    root@${host}:/var/lib/kubelet/kubeconfig
done
```

Скопируйте kubeconfig-файлы `kube-controller-manager` и `kube-scheduler` на машину `server`:

```bash
scp admin.kubeconfig \
  kube-controller-manager.kubeconfig \
  kube-scheduler.kubeconfig \
  root@server:~/
```

Далее: [Создание конфигурации и ключа шифрования данных](06-data-encryption-keys.md)
