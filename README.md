# Kubernetes The Hard Way

Этот учебник проведёт вас через настройку Kubernetes «сложным путём». Это руководство не для тех, кто ищет полностью автоматизированный инструмент для развёртывания кластера Kubernetes. Kubernetes The Hard Way оптимизирован для обучения, а это значит, что мы идём длинным маршрутом, чтобы вы точно разобрались в каждой задаче, необходимой для начальной загрузки кластера Kubernetes.

> Результаты этого учебника не следует рассматривать как готовые к продакшену, и сообщество может оказывать им ограниченную поддержку, но пусть это не помешает вам учиться!

## Авторские права

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />Эта работа распространяется на условиях лицензии <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.


## Целевая аудитория

Целевая аудитория этого учебника — те, кто хочет понять основы Kubernetes и то, как взаимодействуют его ключевые компоненты.

## Детали кластера

Kubernetes The Hard Way проведёт вас через начальную загрузку базового кластера Kubernetes, в котором все компоненты control plane работают на одном узле, плюс два worker-узла — этого достаточно, чтобы изучить ключевые концепции.

Версии компонентов:

* [kubernetes](https://github.com/kubernetes/kubernetes) v1.32.x
* [containerd](https://github.com/containerd/containerd) v2.1.x
* [cni](https://github.com/containernetworking/cni) v1.6.x
* [etcd](https://github.com/etcd-io/etcd) v3.6.x

## Лабораторные работы

Для прохождения этого учебника требуется четыре (4) виртуальные или физические машины на архитектуре ARM64 или AMD64, подключённые к одной сети.

* [Предварительные требования](docs/01-prerequisites.md)
* [Настройка Jumpbox](docs/02-jumpbox.md)
* [Подготовка вычислительных ресурсов](docs/03-compute-resources.md)
* [Подготовка CA и генерация TLS-сертификатов](docs/04-certificate-authority.md)
* [Генерация конфигурационных файлов Kubernetes для аутентификации](docs/05-kubernetes-configuration-files.md)
* [Генерация конфигурации и ключа шифрования данных](docs/06-data-encryption-keys.md)
* [Начальная загрузка кластера etcd](docs/07-bootstrapping-etcd.md)
* [Начальная загрузка control plane Kubernetes](docs/08-bootstrapping-kubernetes-controllers.md)
* [Начальная загрузка worker-узлов Kubernetes](docs/09-bootstrapping-kubernetes-workers.md)
* [Настройка kubectl для удалённого доступа](docs/10-configuring-kubectl.md)
* [Подготовка маршрутов Pod-сети](docs/11-pod-network-routes.md)
* [Дымовой тест](docs/12-smoke-test.md)
* [Очистка](docs/13-cleanup.md)
