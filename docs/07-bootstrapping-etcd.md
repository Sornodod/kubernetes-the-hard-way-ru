# Развёртывание кластера etcd

Компоненты Kubernetes не хранят состояние локально и сохраняют состояние кластера в [etcd](https://github.com/etcd-io/etcd). В этой лабораторной работе вы развернёте одноузловой кластер etcd.

## Предварительные требования

Скопируйте бинарные файлы `etcd` и systemd unit-файл на машину `server`:

```bash
scp \
  downloads/controller/etcd \
  downloads/client/etcdctl \
  units/etcd.service \
  root@server:~/
```

Команды из этой лабораторной работы необходимо выполнять на машине `server`. Подключитесь к ней по SSH. Пример:

```bash
ssh root@server
```

## Развёртывание кластера etcd

### Установка бинарных файлов etcd

Переместите и установите сервер `etcd` и утилиту командной строки `etcdctl`:

```bash
{
  mv etcd etcdctl /usr/local/bin/
}
```

### Настройка сервера etcd

```bash
{
  mkdir -p /etc/etcd /var/lib/etcd
  chmod 700 /var/lib/etcd
  cp ca.crt kube-api-server.key kube-api-server.crt \
    /etc/etcd/
}
```

Каждый участник etcd должен иметь уникальное имя внутри кластера etcd. Укажите для etcd имя, совпадающее с именем хоста текущего вычислительного экземпляра.

Создайте systemd unit-файл `etcd.service`:

```bash
mv etcd.service /etc/systemd/system/
```

### Запуск сервера etcd

```bash
{
  systemctl daemon-reload
  systemctl enable etcd
  systemctl start etcd
}
```

## Проверка

Выведите список участников кластера etcd:

```bash
etcdctl member list
```

```text
6702b0a34e2cfd39, started, controller, http://127.0.0.1:2380, http://127.0.0.1:2379, false
```

Далее: [Развёртывание control plane Kubernetes](08-bootstrapping-kubernetes-controllers.md)
