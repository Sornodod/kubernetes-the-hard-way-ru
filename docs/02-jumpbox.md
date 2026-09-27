# Настройка Jumpbox

В этой лабораторной работе вы настроите одну из четырёх машин в качестве `jumpbox`. Эта машина будет использоваться для запуска команд на протяжении всего учебника. Хотя для обеспечения единообразия используется выделенная машина, эти команды можно выполнять практически с любой машины, включая вашу личную рабочую станцию под управлением macOS или Linux.

Воспринимайте `jumpbox` как машину администратора, которая будет вашей базой при построении кластера Kubernetes с нуля. Прежде чем мы начнём, нужно установить несколько утилит командной строки и клонировать git-репозиторий Kubernetes The Hard Way, который содержит дополнительные конфигурационные файлы, используемые для настройки различных компонентов Kubernetes на протяжении всего учебника.

Войдите на `jumpbox`:

```bash
ssh root@jumpbox
```

Все команды будут выполняться от имени пользователя root. Это сделано для удобства и поможет сократить количество команд, необходимых для настройки всего окружения.

### Установка утилит командной строки

Теперь, когда вы вошли на машину jumpbox как пользователь root, вы установите утилиты командной строки, которые будут использоваться для выполнения различных задач на протяжении учебника.

```bash
{
  apt-get update
  apt-get -y install wget curl vim openssl git
}
```

### Синхронизация GitHub-репозитория

Теперь пора скачать копию этого учебника, содержащую конфигурационные файлы и шаблоны, которые будут использоваться для построения вашего кластера Kubernetes с нуля. Клонируйте git-репозиторий Kubernetes The Hard Way с помощью команды git:

```bash
git clone --depth 1 \
  https://github.com/kelseyhightower/kubernetes-the-hard-way.git
```

Перейдите в каталог kubernetes-the-hard-way:
```bash
cd kubernetes-the-hard-way
```

Это будет рабочим каталогом для всего оставшегося учебника. Если вы когда-нибудь заблудитесь, выполните команду `pwd`, чтобы убедиться, что находитесь в правильном каталоге при выполнении команд на `jumpbox`:

```bash
pwd
```

```text
/root/kubernetes-the-hard-way
```

### Загрузка бинарных файлов

В этом разделе вы скачаете бинарные файлы для различных компонентов Kubernetes. Бинарные файлы будут храниться в каталоге `downloads` на `jumpbox`, что позволит сократить объём интернет-трафика, необходимого для прохождения учебника, поскольку мы избегаем многократной загрузки бинарных файлов для каждой машины в нашем кластере Kubernetes.

Бинарные файлы, которые будут загружены, перечислены в файле `downloads-amd64.txt` или `downloads-arm64.txt` в зависимости от архитектуры вашего оборудования; просмотреть его можно с помощью команды cat:

```bash
cat downloads-$(dpkg --print-architecture).txt
```

Скачайте бинарные файлы в каталог с именем `downloads` с помощью команды wget:

```bash
wget -q --show-progress \
  --https-only \
  --timestamping \
  -P downloads \
  -i downloads-$(dpkg --print-architecture).txt
```

В зависимости от скорости вашего интернет-соединения загрузка более `500` мегабайт бинарных файлов может занять некоторое время; когда загрузка завершится, вы можете вывести их список с помощью команды `ls`:

```bash
ls -oh downloads
```

Извлеките бинарные файлы компонентов из архивов релизов и разложите их по каталогу `downloads`.

```bash
{
  ARCH=$(dpkg --print-architecture)
  mkdir -p downloads/{client,cni-plugins,controller,worker}
  tar -xvf downloads/crictl-v1.32.0-linux-${ARCH}.tar.gz \
    -C downloads/worker/
  tar -xvf downloads/containerd-2.1.0-beta.0-linux-${ARCH}.tar.gz \
    --strip-components 1 \
    -C downloads/worker/
  tar -xvf downloads/cni-plugins-linux-${ARCH}-v1.6.2.tgz \
    -C downloads/cni-plugins/
  tar -xvf downloads/etcd-v3.6.0-rc.3-linux-${ARCH}.tar.gz \
    -C downloads/ \
    --strip-components 1 \
    etcd-v3.6.0-rc.3-linux-${ARCH}/etcdctl \
    etcd-v3.6.0-rc.3-linux-${ARCH}/etcd
  mv downloads/{etcdctl,kubectl} downloads/client/
  mv downloads/{etcd,kube-apiserver,kube-controller-manager,kube-scheduler} \
    downloads/controller/
  mv downloads/{kubelet,kube-proxy} downloads/worker/
  mv downloads/runc.${ARCH} downloads/worker/runc
}
```

```bash
rm -rf downloads/*gz
```

Сделайте бинарные файлы исполняемыми.

```bash
{
  chmod +x downloads/{client,cni-plugins,controller,worker}/*
}
```

### Установка kubectl

В этом разделе вы установите `kubectl` — официальный клиент командной строки Kubernetes — на машину `jumpbox`. `kubectl` будет использоваться для взаимодействия с control plane Kubernetes после того, как ваш кластер будет развёрнут далее в этом учебнике.

Используйте команду `chmod`, чтобы сделать бинарный файл `kubectl` исполняемым, и переместите его в каталог `/usr/local/bin/`:

```bash
{
  cp downloads/client/kubectl /usr/local/bin/
}
```

На этом этапе `kubectl` установлен, и это можно проверить, выполнив команду kubectl:

```bash
kubectl version --client
```

```text
Client Version: v1.32.3
Kustomize Version: v5.5.0
```

На этом этапе `jumpbox` настроен со всеми инструментами и утилитами командной строки, необходимыми для прохождения лабораторных работ этого учебника.

Далее: [Подготовка вычислительных ресурсов](03-compute-resources.md)
