---
title: Моделирование сценариев CFD
titleSuffix: Azure Example Scenarios
description: Моделирование сценариев CFD в Azure.
author: mikewarr
ms.date: 09/20/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: fasttrack
social_image_url: /azure/architecture/example-scenario/infrastructure/media/architecture-hpc-cfd.png
ms.openlocfilehash: 6972b701a608351104c23459bc2c38029a5c8d3d
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58249589"
---
# <a name="running-computational-fluid-dynamics-cfd-simulations-on-azure"></a>Моделирование сценариев вычислительной гидродинамики (CFD) в Azure

Моделирование вычислительной гидродинамики (CFD) требует значительных временных затрат на вычисления, а также специализированного оборудования. По мере увеличения использования кластера время моделирования и общее использование сетки увеличиваются, вызывая проблемы со свободным местом и длительным временем в очереди. Добавление физического оборудования может быть затратным и не совпадать с колебаниями использования, которые испытывает компания. Используя преимущества Azure, многие из этих проблем можно преодолеть без капитальных затрат.

Azure предоставляет оборудование, необходимое для выполнения заданий CFD на обеих виртуальных машинах GPU. Размеры виртуальных машин с поддержкой RDMA (удаленного доступа к памяти) имеют сетевые подключения на основе FDR InfiniBand, которые обеспечивают низкую задержку обмена данными MPI (Message Passing Interface, интерфейса передачи сообщений). В сочетании с системой Avere vFXT, предоставляющей кластеризованную файловую систему корпоративного масштаба, клиенты могут обеспечить максимальную пропускную способность для операций чтения в Azure.

Чтобы упростить создание, управление и оптимизацию кластеров HPC, можно использовать Azure CycleCloud для подготовки кластеров и оркестрации данных в гибридных и облачных сценариях. Отслеживая задания в состоянии ожидания, CycleCloud автоматически запустит вычисления по запросу, подключенные к рабочей нагрузке предпочитаемого планировщика, при которых вы платите только за использование.

## <a name="relevant-use-cases"></a>Варианты соответствующего использования

Другие соответствующие отрасли для приложений CFD:

- аэронавтика;
- автомобили;
- построение систем отопления, вентиляции и кондиционирования;
- нефть и газ;
- медико-биологическая отрасль.

## <a name="architecture"></a>Архитектура

![Схема архитектуры][architecture]

На этой схеме показан общий обзор типичной гибридной структуры, обеспечивающей мониторинг заданий узлов по запросу в Azure.

1. Подключитесь к серверу Azure CycleCloud для настройки кластера.
2. Настройте и создайте головной узел кластера с помощью компьютеров с поддержкой RDMA для MPI.
3. Добавьте и настройте локальный головной узел.
4. При возникновении недостатка ресурсов, Azure CycleCloud увеличит (или уменьшит) масштаб вычислительных ресурсов в Azure. Во избежание чрезмерного распределения можно установить предопределенное ограничение.
5. Задачи, выделенные в узлах выполнения.
6. Данные, кэшированные в Azure с локального сервера NFS.
7. Данные, считанные из Avere vFXT для кэша Azure.
8. Сведения о заданиях и задачах, ретранслированных на сервер Azure CycleCloud.

### <a name="components"></a>Компоненты

- [Azure CycleCloud][cyclecloud] — инструмент для создания, управления, обработки и оптимизации HPC и кластеров больших вычислений в Azure.
- [Avere vFXT в Azure][avere] используется для предоставления кластеризованной файловой системы корпоративного масштаба, построенной для облака.
- [Виртуальные машины Azure][vms] позволяют создать статический набор вычислительных операций.
- [Масштабируемые наборы виртуальных машин][vmss] предоставляют группу идентичных виртуальных машин, поддерживающих масштабирование, проводимое в Azure CycleCloud.
- [Учетные записи хранения Azure](/azure/storage/common/storage-introduction) используются для хранения данных и синхронизации.
- [Виртуальные сети](/azure/virtual-network/virtual-networks-overview) позволяет ресурсам Azure различных типов (например, виртуальным машинам Azure) безопасно обмениваться данными через локальные сети и Интернет.

### <a name="alternatives"></a>Альтернативные варианты

Клиенты также могут использовать Azure CycleCloud, чтобы полностью создать сетку в Azure. В такой конфигурации сервер Azure CycleCloud запускается в подписке Azure.

Для современного подхода к приложениям, в котором управление планировщиком рабочей нагрузки не требуется, может пригодиться [пакетная служба Azure][batch]. Пакетная служба Azure может эффективно запускать приложения для крупномасштабных параллельных и высокопроизводительных вычислений (HPC) в облаке. Пакетная служба Azure позволяет определить вычислительные ресурсы Azure для выполнения приложений в параллельном режиме или с масштабированием без необходимости настраивать инфраструктуру и управлять ею вручную. Пакетная служба Azure планирует ресурсоемкие задачи и динамически добавляет и удаляет вычислительные ресурсы в зависимости от требований.

### <a name="scalability-and-security"></a>Масштабируемость и безопасность

Масштабирование узлов выполнения в Azure CycleCloud возможно либо вручную, либо с помощью автоматического масштабирования. Дополнительные сведения см. в статье [AutoScale Your Clusters][cycle-scale] (Автомасштабирование кластеров).

Общие рекомендации по разработке безопасных решений см. в разделе [Документация по системе безопасности Azure][security].

## <a name="deploy-the-scenario"></a>Развертывание сценария

### <a name="prerequisites"></a>Предварительные требования

Перед развертыванием шаблона Resource Manager выполните следующие действия.

1. Создайте [субъект-службу][cycle-svcprin] для получения appId, displayName, имени, пароля и клиента.
2. Создайте [пару ключей SSH][cycle-ssh] для безопасного входа на сервер CycleCloud.

    <!-- markdownlint-disable MD033 -->

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FCycleCloudCommunity%2Fcyclecloud_arm%2Fmaster%2Fazuredeploy.json" target="_blank">
        <img src="https://azuredeploy.net/deploybutton.png"/>
    </a>

    <!-- markdownlint-enable MD033 -->

3. [Войдите на сервер CycleCloud][cycle-login] для настройки и создания нового кластера.
4. [Создайте кластер][cycle-create].

Кэш Avere — это дополнительное решение, которое может существенно увеличить пропускную способность при чтении для данных заданий приложения. Avere vFXT для Azure позволяет решить проблему запуска этих корпоративных HPC-приложений в облаке, используя данные, хранящиеся локально или в хранилище BLOB-объектов Azure.

Для организаций, в которых запланировано использование гибридной инфраструктуры как с локальным хранилищем, так и с облачными вычислениями, HPC-приложения могут "прорываться" в Azure, используя данные, хранящиеся на NAS-устройствах, и при необходимости развертывать виртуальные ЦП. Набор данных никогда полностью не переносится в облако. Запрошенные байты временно кэшируются, используя кластер Avere во время обработки.

Чтобы начать и настроить процесс установки Avere vFXT, выполните инструкции, приведенные в [Avere Setup and Configuration guide][avere] (Руководство по установке и настройке Avere).

## <a name="pricing"></a>Цены

Стоимость запуска реализации HPC с помощью сервера CycleCloud будет зависеть от ряда факторов. Например, в CycleCloud оплачивается количество времени, затраченного на вычисления. При этом главный сервер и сервер CycleCloud обычно постоянно выделены и находятся в рабочем состоянии. Стоимость запуска узлов выполнения будет зависеть от того, как долго они выполняются, а также от используемого размера. Обычная плата Azure за использование хранилища и сетевые подключения также взимается.

В этом сценарии показано, как CFD-приложения могут выполняться в Azure, чтобы компьютеры требовали наличие функции RDMA, которая доступна только для определенных размерах виртуальных машин. Ниже приведены примеры платы, взымаемой за масштабируемый набор, который непрерывно выделяется на восемь часов в день на месяц с исходящим трафиком данных 1 ТБ. Сюда также входят цены на сервер Azure CycleCloud и vFXT Avere для установки Azure.

- Регион: Северная Европа
- Сервер Azure CycleCloud: 1 Standard D3 (4 ЦП, 14 ГБ памяти, диск HDD категории "Стандартный" на 32 ГБ).
- Главный сервер Azure CycleCloud: 1 Standard D12 (4 ЦП, 28 ГБ памяти, диск HDD категории "Стандартный" на 32 ГБ).
- Массив узла Azure CycleCloud: 10 Standard H16r (16 ЦП, 112 ГБ памяти).
- Avere vFXT в кластере Azure: 3 D16s v3 (200 ГБ ОС, диск SSD категории "Премиум" на 1 ТБ).
- Исходящий трафик данных: 1 TБ

Ознакомьтесь с [ценами][pricing] для оборудования, перечисленного выше.

## <a name="next-steps"></a>Дальнейшие действия

После развертывания примера подробнее изучите [Azure CycleCloud][cyclecloud].

## <a name="related-resources"></a>Связанные ресурсы

- [Экземпляры с поддержкой RDMA][rdma]
- [Customizing an RDMA Instance VM][rdma-custom] (Настройка RDMA-экземпляра виртуальной машины)

<!-- links -->
[architecture]: ./media/architecture-hpc-cfd.png
[calculator]: https://azure.com/e/
[availability]: /azure/architecture/checklist/availability
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[vmss]: /azure/virtual-machine-scale-sets/overview
[cyclecloud]: /azure/cyclecloud/
[rdma]: /azure/virtual-machines/windows/sizes-hpc#rdma-capable-instances
[gpu]: /azure/virtual-machines/windows/sizes-gpu
[hpcsizes]: /azure/virtual-machines/windows/sizes-hpc
[vms]: /azure/virtual-machines/
[low-pri]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-use-low-priority
[batch]: /azure/batch/
[avere]: https://github.com/Azure/Avere/blob/master/README.md
[cycle-prereq]: /azure/cyclecloud/quickstart-install-cyclecloud#prerequisites
[cycle-svcprin]: /azure/cyclecloud/quickstart-install-cyclecloud#service-principal
[cycle-ssh]: /azure/cyclecloud/quickstart-install-cyclecloud#ssh-keypair
[cycle-login]: /azure/cyclecloud/quickstart-install-cyclecloud#log-into-the-cyclecloud-application-server
[cycle-create]: /azure/cyclecloud/quickstart-create-and-run-cluster
[rdma]: /azure/virtual-machines/windows/sizes-hpc#rdma-capable-instances
[rdma-custom]: /azure/virtual-machines/linux/classic/rdma-cluster#customize-the-vm
[pricing]: https://azure.com/e/53030a04a2ab47a289156e2377a4247a
[cycle-scale]: /azure/cyclecloud/autoscale
