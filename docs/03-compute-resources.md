# Подготовка вычислительных ресурсов

Kubernetes требует набора машин для размещения control plane Kubernetes и worker-узлов, на которых в конечном итоге запускаются контейнеры. В этой лабораторной работе вы подготовите машины, необходимые для настройки кластера Kubernetes.

## База данных машин

В этом учебнике будет использоваться текстовый файл, который послужит базой данных машин и будет хранить различные атрибуты машин, используемые при настройке control plane Kubernetes и worker-узлов. Следующая схема описывает записи в базе данных машин, по одной записи на строку:

```text
IPV4_ADDRESS FQDN HOSTNAME POD_SUBNET
```

Каждый из столбцов соответствует IP-адресу машины `IPV4_ADDRESS`, полному доменному имени `FQDN`, имени хоста `HOSTNAME` и IP-подсети `POD_SUBNET`. Kubernetes назначает по одному IP-адресу на каждый `pod`, а `POD_SUBNET` представляет уникальный диапазон IP-адресов, выделенный каждой машине в кластере для этой цели.

Вот пример базы данных машин, похожий на тот, что использовался при создании этого учебника. Обратите внимание, что IP-адреса замаскированы. Вашим машинам можно назначить любые IP-адреса, если каждая машина доступна с любой другой машины и с `jumpbox`.

```bash
cat machines.txt
```

```text
XXX.XXX.XXX.XXX server.kubernetes.local server
XXX.XXX.XXX.XXX node-0.kubernetes.local node-0 10.200.0.0/24
XXX.XXX.XXX.XXX node-1.kubernetes.local node-1 10.200.1.0/24
```

Теперь ваша очередь создать файл `machines.txt` с данными для трёх машин, которые вы будете использовать для создания кластера Kubernetes. Используйте пример базы данных машин выше и добавьте данные для своих машин.

## Настройка SSH-доступа

SSH будет использоваться для настройки машин в кластере. Убедитесь, что у вас есть SSH-доступ от имени `root` к каждой машине, перечисленной в вашей базе данных машин. Возможно, вам потребуется включить root-доступ по SSH на каждом узле, обновив файл sshd_config и перезапустив SSH-сервер.

### Включение root-доступа по SSH

Если root-доступ по SSH уже включён для каждой из ваших машин, этот раздел можно пропустить.

По умолчанию новая установка `debian` отключает SSH-доступ для пользователя `root`. Это сделано в целях безопасности, поскольку пользователь `root` обладает полным административным контролем над unix-подобными системами. Если на машине, подключённой к интернету, используется слабый пароль, что ж, скажем так — это лишь вопрос времени, когда ваша машина перейдёт к кому-то другому. Как упоминалось ранее, мы включим доступ `root` по SSH, чтобы упростить шаги в этом учебнике. Безопасность — это компромисс, и в данном случае мы оптимизируем удобство. Войдите на каждую машину по SSH под своей учётной записью, затем переключитесь на пользователя `root` с помощью команды `su`:

```bash
su - root
```

Отредактируйте файл конфигурации SSH-демона `/etc/ssh/sshd_config` и установите для параметра `PermitRootLogin` значение `yes`:
```bash
sed -i \
  's/^#*PermitRootLogin.*/PermitRootLogin yes/' \
  /etc/ssh/sshd_config
```

Перезапустите SSH-сервер `sshd`, чтобы он подхватил обновлённый файл конфигурации:

```bash
systemctl restart sshd
```

### Генерация и распространение SSH-ключей

В этом разделе вы сгенерируете пару SSH-ключей и распространите её на машины `server`, `node-0` и `node-1`, которые будут использоваться для запуска команд на этих машинах на протяжении всего учебника. Выполните следующие команды с машины `jumpbox`.

Сгенерируйте новый SSH-ключ:

```bash
ssh-keygen
```

```text
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa
Your public key has been saved in /root/.ssh/id_rsa.pub
```

Скопируйте открытый SSH-ключ на каждую машину:

```bash
while read IP FQDN HOST SUBNET; do
  ssh-copy-id root@${IP}
done < machines.txt
```

После добавления каждого ключа проверьте, что доступ по открытому SSH-ключу работает:

```bash
while read IP FQDN HOST SUBNET; do
  ssh -n root@${IP} hostname
done < machines.txt
```

```text
server
node-0
node-1
```

## Имена хостов

В этом разделе вы назначите имена хостов машинам `server`, `node-0` и `node-1`. Имя хоста будет использоваться при выполнении команд с `jumpbox` на каждую машину. Имя хоста также играет важную роль внутри кластера. Вместо того чтобы клиенты Kubernetes использовали IP-адрес для отправки команд API-серверу Kubernetes, эти клиенты будут использовать имя хоста `server`. Имена хостов также используются каждой worker-машиной, `node-0` и `node-1`, при регистрации в заданном кластере Kubernetes.

Чтобы настроить имя хоста для каждой машины, выполните следующие команды на `jumpbox`.

Установите имя хоста на каждой машине, перечисленной в файле `machines.txt`:

```bash
while read IP FQDN HOST SUBNET; do
    CMD="sed -i 's/^127.0.1.1.*/127.0.1.1\t${FQDN} ${HOST}/' /etc/hosts"
    ssh -n root@${IP} "$CMD"
    ssh -n root@${IP} hostnamectl set-hostname ${HOST}
    ssh -n root@${IP} systemctl restart systemd-hostnamed
done < machines.txt
```

Проверьте, что имя хоста установлено на каждой машине:

```bash
while read IP FQDN HOST SUBNET; do
  ssh -n root@${IP} hostname --fqdn
done < machines.txt
```

```text
server.kubernetes.local
node-0.kubernetes.local
node-1.kubernetes.local
```

## Таблица соответствия хостов

В этом разделе вы сгенерируете файл `hosts`, который будет добавлен к файлу `/etc/hosts` на `jumpbox` и к файлам `/etc/hosts` на всех трёх членах кластера, используемых в этом учебнике. Это позволит обращаться к каждой машине по имени хоста, например `server`, `node-0` или `node-1`.

Создайте новый файл `hosts` и добавьте заголовок для идентификации добавляемых машин:

```bash
echo "" > hosts
echo "# Kubernetes The Hard Way" >> hosts
```

Сгенерируйте запись хоста для каждой машины в файле `machines.txt` и добавьте её в файл `hosts`:

```bash
while read IP FQDN HOST SUBNET; do
    ENTRY="${IP} ${FQDN} ${HOST}"
    echo $ENTRY >> hosts
done < machines.txt
```

Просмотрите записи хостов в файле `hosts`:

```bash
cat hosts
```

```text

# Kubernetes The Hard Way
XXX.XXX.XXX.XXX server.kubernetes.local server
XXX.XXX.XXX.XXX node-0.kubernetes.local node-0
XXX.XXX.XXX.XXX node-1.kubernetes.local node-1
```

## Добавление записей в /etc/hosts на локальной машине

В этом разделе вы добавите DNS-записи из файла `hosts` в локальный файл `/etc/hosts` на машине `jumpbox`.

Добавьте DNS-записи из `hosts` в `/etc/hosts`:

```bash
cat hosts >> /etc/hosts
```

Проверьте, что файл `/etc/hosts` был обновлён:

```bash
cat /etc/hosts
```

```text
127.0.0.1       localhost
127.0.1.1       jumpbox

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

# Kubernetes The Hard Way
XXX.XXX.XXX.XXX server.kubernetes.local server
XXX.XXX.XXX.XXX node-0.kubernetes.local node-0
XXX.XXX.XXX.XXX node-1.kubernetes.local node-1
```

На этом этапе вы должны иметь возможность подключиться по SSH к каждой машине, перечисленной в файле `machines.txt`, используя имя хоста.

```bash
for host in server node-0 node-1
   do ssh root@${host} hostname
done
```

```text
server
node-0
node-1
```

## Добавление записей в `/etc/hosts` на удалённых машинах

В этом разделе вы добавите записи хостов из файла hosts в /etc/hosts на каждой машине, перечисленной в текстовом файле machines.txt.

Скопируйте файл `hosts` на каждую машину и добавьте его содержимое в `/etc/hosts`:

```bash
while read IP FQDN HOST SUBNET; do
  scp hosts root@${HOST}:~/
  ssh -n \
    root@${HOST} "cat hosts >> /etc/hosts"
done < machines.txt
```

На этом этапе имена хостов можно использовать при подключении к машинам с вашей машины `jumpbox` или с любой из трёх машин в кластере Kubernetes. Вместо IP-адресов теперь можно подключаться к машинам, используя имя хоста, например `server`, `node-0` или `node-1`.

Next: [Подготовка CA и генерация TLS-сертификатов](04-certificate-authority.md)
