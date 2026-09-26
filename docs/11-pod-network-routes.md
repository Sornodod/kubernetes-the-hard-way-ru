# Настройка маршрутов сети pod'ов

Pod'ы, назначенные на узел, получают IP-адрес из диапазона Pod CIDR этого узла. На данном этапе pod'ы не могут взаимодействовать с pod'ами, работающими на других узлах, из-за отсутствующих сетевых [маршрутов](https://cloud.google.com/compute/docs/vpc/routes).

В этой лабораторной работе вы создадите маршрут для каждого рабочего узла, который сопоставляет диапазон Pod CIDR узла с его внутренним IP-адресом.

> Существуют [другие способы](https://kubernetes.io/docs/concepts/cluster-administration/networking/#how-to-achieve-this) реализации сетевой модели Kubernetes.

## Таблица маршрутизации

В этом разделе вы получите сведения, необходимые для создания маршрутов в VPC-сети `kubernetes-the-hard-way`.

Получите внутренний IP-адрес и диапазон Pod CIDR для каждого рабочего узла:

```bash
{
  SERVER_IP=$(grep server machines.txt | cut -d " " -f 1)
  NODE_0_IP=$(grep node-0 machines.txt | cut -d " " -f 1)
  NODE_0_SUBNET=$(grep node-0 machines.txt | cut -d " " -f 4)
  NODE_1_IP=$(grep node-1 machines.txt | cut -d " " -f 1)
  NODE_1_SUBNET=$(grep node-1 machines.txt | cut -d " " -f 4)
}
```

Добавьте маршруты на машине `server`:

```bash
ssh root@server <<EOF
  ip route add ${NODE_0_SUBNET} via ${NODE_0_IP}
  ip route add ${NODE_1_SUBNET} via ${NODE_1_IP}
EOF
```

Добавьте маршрут к Pod CIDR узла `node-1` на машине `node-0`:

```bash
ssh root@node-0 <<EOF
  ip route add ${NODE_1_SUBNET} via ${NODE_1_IP}
EOF
```

Добавьте маршрут к Pod CIDR узла `node-0` на машине `node-1`:

```bash
ssh root@node-1 <<EOF
  ip route add ${NODE_0_SUBNET} via ${NODE_0_IP}
EOF
```

## Проверка

Проверьте таблицу маршрутизации на машине `server`:

```bash
ssh root@server ip route
```

```text
default via XXX.XXX.XXX.XXX dev ens160 
10.200.0.0/24 via XXX.XXX.XXX.XXX dev ens160 
10.200.1.0/24 via XXX.XXX.XXX.XXX dev ens160 
XXX.XXX.XXX.0/24 dev ens160 proto kernel scope link src XXX.XXX.XXX.XXX 
```

Проверьте таблицу маршрутизации на машине `node-0`:

```bash
ssh root@node-0 ip route
```

```text
default via XXX.XXX.XXX.XXX dev ens160 
10.200.1.0/24 via XXX.XXX.XXX.XXX dev ens160 
XXX.XXX.XXX.0/24 dev ens160 proto kernel scope link src XXX.XXX.XXX.XXX 
```

Проверьте таблицу маршрутизации на машине `node-1`:

```bash
ssh root@node-1 ip route
```

```text
default via XXX.XXX.XXX.XXX dev ens160 
10.200.0.0/24 via XXX.XXX.XXX.XXX dev ens160 
XXX.XXX.XXX.0/24 dev ens160 proto kernel scope link src XXX.XXX.XXX.XXX 
```

Далее: [Финальная проверка](12-smoke-test.md)
