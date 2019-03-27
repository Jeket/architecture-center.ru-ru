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
# <a name="running-computational-fluid-dynamics-cfd-simulations-on-azure"></a><span data-ttu-id="1d4b3-103">Моделирование сценариев вычислительной гидродинамики (CFD) в Azure</span><span class="sxs-lookup"><span data-stu-id="1d4b3-103">Running computational fluid dynamics (CFD) simulations on Azure</span></span>

<span data-ttu-id="1d4b3-104">Моделирование вычислительной гидродинамики (CFD) требует значительных временных затрат на вычисления, а также специализированного оборудования.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-104">Computational Fluid Dynamics (CFD) simulations require significant compute time along with specialized hardware.</span></span> <span data-ttu-id="1d4b3-105">По мере увеличения использования кластера время моделирования и общее использование сетки увеличиваются, вызывая проблемы со свободным местом и длительным временем в очереди.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-105">As cluster usage increases, simulation times and overall grid use grow, leading to issues with spare capacity and long queue times.</span></span> <span data-ttu-id="1d4b3-106">Добавление физического оборудования может быть затратным и не совпадать с колебаниями использования, которые испытывает компания.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-106">Adding physical hardware can be expensive, and may not align to the usage peaks and valleys that a business goes through.</span></span> <span data-ttu-id="1d4b3-107">Используя преимущества Azure, многие из этих проблем можно преодолеть без капитальных затрат.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-107">By taking advantage of Azure, many of these challenges can be overcome with no capital expenditure.</span></span>

<span data-ttu-id="1d4b3-108">Azure предоставляет оборудование, необходимое для выполнения заданий CFD на обеих виртуальных машинах GPU.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-108">Azure provides the hardware you need to run your CFD jobs on both GPU and CPU virtual machines.</span></span> <span data-ttu-id="1d4b3-109">Размеры виртуальных машин с поддержкой RDMA (удаленного доступа к памяти) имеют сетевые подключения на основе FDR InfiniBand, которые обеспечивают низкую задержку обмена данными MPI (Message Passing Interface, интерфейса передачи сообщений).</span><span class="sxs-lookup"><span data-stu-id="1d4b3-109">RDMA (Remote Direct Memory Access) enabled VM sizes have FDR InfiniBand-based networking which allows for low latency MPI (Message Passing Interface) communication.</span></span> <span data-ttu-id="1d4b3-110">В сочетании с системой Avere vFXT, предоставляющей кластеризованную файловую систему корпоративного масштаба, клиенты могут обеспечить максимальную пропускную способность для операций чтения в Azure.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-110">Combined with the Avere vFXT, which provides an enterprise-scale clustered file system, customers can ensure maximum throughput for read operations in Azure.</span></span>

<span data-ttu-id="1d4b3-111">Чтобы упростить создание, управление и оптимизацию кластеров HPC, можно использовать Azure CycleCloud для подготовки кластеров и оркестрации данных в гибридных и облачных сценариях.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-111">To simplify the creation, management, and optimization of HPC clusters, Azure CycleCloud can be used to provision clusters and orchestrate data in both hybrid and cloud scenarios.</span></span> <span data-ttu-id="1d4b3-112">Отслеживая задания в состоянии ожидания, CycleCloud автоматически запустит вычисления по запросу, подключенные к рабочей нагрузке предпочитаемого планировщика, при которых вы платите только за использование.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-112">By monitoring the pending jobs, CycleCloud will automatically launch on-demand compute, where you only pay for what you use, connected to the workload scheduler of your choice.</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="1d4b3-113">Варианты соответствующего использования</span><span class="sxs-lookup"><span data-stu-id="1d4b3-113">Relevant use cases</span></span>

<span data-ttu-id="1d4b3-114">Другие соответствующие отрасли для приложений CFD:</span><span class="sxs-lookup"><span data-stu-id="1d4b3-114">Other relevant industries for CFD applications include:</span></span>

- <span data-ttu-id="1d4b3-115">аэронавтика;</span><span class="sxs-lookup"><span data-stu-id="1d4b3-115">Aeronautics</span></span>
- <span data-ttu-id="1d4b3-116">автомобили;</span><span class="sxs-lookup"><span data-stu-id="1d4b3-116">Automotive</span></span>
- <span data-ttu-id="1d4b3-117">построение систем отопления, вентиляции и кондиционирования;</span><span class="sxs-lookup"><span data-stu-id="1d4b3-117">Building HVAC</span></span>
- <span data-ttu-id="1d4b3-118">нефть и газ;</span><span class="sxs-lookup"><span data-stu-id="1d4b3-118">Oil and gas</span></span>
- <span data-ttu-id="1d4b3-119">медико-биологическая отрасль.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-119">Life sciences</span></span>

## <a name="architecture"></a><span data-ttu-id="1d4b3-120">Архитектура</span><span class="sxs-lookup"><span data-stu-id="1d4b3-120">Architecture</span></span>

![Схема архитектуры][architecture]

<span data-ttu-id="1d4b3-122">На этой схеме показан общий обзор типичной гибридной структуры, обеспечивающей мониторинг заданий узлов по запросу в Azure.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-122">This diagram shows a high-level overview of a typical hybrid design providing job monitoring of the on-demand nodes in Azure:</span></span>

1. <span data-ttu-id="1d4b3-123">Подключитесь к серверу Azure CycleCloud для настройки кластера.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-123">Connect to the Azure CycleCloud server to configure the cluster.</span></span>
2. <span data-ttu-id="1d4b3-124">Настройте и создайте головной узел кластера с помощью компьютеров с поддержкой RDMA для MPI.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-124">Configure and create the cluster head node, using RDMA enabled machines for MPI.</span></span>
3. <span data-ttu-id="1d4b3-125">Добавьте и настройте локальный головной узел.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-125">Add and configure the on-premises head node.</span></span>
4. <span data-ttu-id="1d4b3-126">При возникновении недостатка ресурсов, Azure CycleCloud увеличит (или уменьшит) масштаб вычислительных ресурсов в Azure.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-126">If there are insufficient resources, Azure CycleCloud will scale up (or down) compute resources in Azure.</span></span> <span data-ttu-id="1d4b3-127">Во избежание чрезмерного распределения можно установить предопределенное ограничение.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-127">A predetermined limit can be defined to prevent over allocation.</span></span>
5. <span data-ttu-id="1d4b3-128">Задачи, выделенные в узлах выполнения.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-128">Tasks allocated to the execute nodes.</span></span>
6. <span data-ttu-id="1d4b3-129">Данные, кэшированные в Azure с локального сервера NFS.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-129">Data cached in Azure from on-premises NFS server.</span></span>
7. <span data-ttu-id="1d4b3-130">Данные, считанные из Avere vFXT для кэша Azure.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-130">Data read in from the Avere vFXT for Azure cache.</span></span>
8. <span data-ttu-id="1d4b3-131">Сведения о заданиях и задачах, ретранслированных на сервер Azure CycleCloud.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-131">Job and task information relayed to the Azure CycleCloud server.</span></span>

### <a name="components"></a><span data-ttu-id="1d4b3-132">Компоненты</span><span class="sxs-lookup"><span data-stu-id="1d4b3-132">Components</span></span>

- <span data-ttu-id="1d4b3-133">[Azure CycleCloud][cyclecloud] — инструмент для создания, управления, обработки и оптимизации HPC и кластеров больших вычислений в Azure.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-133">[Azure CycleCloud][cyclecloud] a tool for creating, managing, operating, and optimizing HPC and Big Compute clusters in Azure.</span></span>
- <span data-ttu-id="1d4b3-134">[Avere vFXT в Azure][avere] используется для предоставления кластеризованной файловой системы корпоративного масштаба, построенной для облака.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-134">[Avere vFXT on Azure][avere] is used to provide an enterprise-scale clustered file system built for the cloud.</span></span>
- <span data-ttu-id="1d4b3-135">[Виртуальные машины Azure][vms] позволяют создать статический набор вычислительных операций.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-135">[Azure Virtual Machines (VMs)][vms] are used to create a static set of compute instances.</span></span>
- <span data-ttu-id="1d4b3-136">[Масштабируемые наборы виртуальных машин][vmss] предоставляют группу идентичных виртуальных машин, поддерживающих масштабирование, проводимое в Azure CycleCloud.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-136">[Virtual Machine Scale Sets (virtual machine scale set)][vmss] provide a group of identical VMs capable of being scaled up or down by Azure CycleCloud.</span></span>
- <span data-ttu-id="1d4b3-137">[Учетные записи хранения Azure](/azure/storage/common/storage-introduction) используются для хранения данных и синхронизации.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-137">[Azure Storage accounts](/azure/storage/common/storage-introduction) are used for synchronization and data retention.</span></span>
- <span data-ttu-id="1d4b3-138">[Виртуальные сети](/azure/virtual-network/virtual-networks-overview) позволяет ресурсам Azure различных типов (например, виртуальным машинам Azure) безопасно обмениваться данными через локальные сети и Интернет.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-138">[Virtual Networks](/azure/virtual-network/virtual-networks-overview) enable many types of Azure resources, such as Azure Virtual Machines (VMs), to securely communicate with each other, the internet, and on-premises networks.</span></span>

### <a name="alternatives"></a><span data-ttu-id="1d4b3-139">Альтернативные варианты</span><span class="sxs-lookup"><span data-stu-id="1d4b3-139">Alternatives</span></span>

<span data-ttu-id="1d4b3-140">Клиенты также могут использовать Azure CycleCloud, чтобы полностью создать сетку в Azure.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-140">Customers can also use Azure CycleCloud to create a grid entirely in Azure.</span></span> <span data-ttu-id="1d4b3-141">В такой конфигурации сервер Azure CycleCloud запускается в подписке Azure.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-141">In this setup, the Azure CycleCloud server is run within your Azure subscription.</span></span>

<span data-ttu-id="1d4b3-142">Для современного подхода к приложениям, в котором управление планировщиком рабочей нагрузки не требуется, может пригодиться [пакетная служба Azure][batch].</span><span class="sxs-lookup"><span data-stu-id="1d4b3-142">For a modern application approach where management of a workload scheduler is not needed, [Azure Batch][batch] can help.</span></span> <span data-ttu-id="1d4b3-143">Пакетная служба Azure может эффективно запускать приложения для крупномасштабных параллельных и высокопроизводительных вычислений (HPC) в облаке.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-143">Azure Batch can run large-scale parallel and high-performance computing (HPC) applications efficiently in the cloud.</span></span> <span data-ttu-id="1d4b3-144">Пакетная служба Azure позволяет определить вычислительные ресурсы Azure для выполнения приложений в параллельном режиме или с масштабированием без необходимости настраивать инфраструктуру и управлять ею вручную.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-144">Azure Batch allows you to define the Azure compute resources to execute your applications in parallel or at scale without manually configuring or managing infrastructure.</span></span> <span data-ttu-id="1d4b3-145">Пакетная служба Azure планирует ресурсоемкие задачи и динамически добавляет и удаляет вычислительные ресурсы в зависимости от требований.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-145">Azure Batch schedules compute-intensive tasks and dynamically adds and removes compute resources based on your requirements.</span></span>

### <a name="scalability-and-security"></a><span data-ttu-id="1d4b3-146">Масштабируемость и безопасность</span><span class="sxs-lookup"><span data-stu-id="1d4b3-146">Scalability, and Security</span></span>

<span data-ttu-id="1d4b3-147">Масштабирование узлов выполнения в Azure CycleCloud возможно либо вручную, либо с помощью автоматического масштабирования.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-147">Scaling the execute nodes on Azure CycleCloud can be accomplished either manually or using autoscaling.</span></span> <span data-ttu-id="1d4b3-148">Дополнительные сведения см. в статье [AutoScale Your Clusters][cycle-scale] (Автомасштабирование кластеров).</span><span class="sxs-lookup"><span data-stu-id="1d4b3-148">For more information, see [CycleCloud Autoscaling][cycle-scale].</span></span>

<span data-ttu-id="1d4b3-149">Общие рекомендации по разработке безопасных решений см. в разделе [Документация по системе безопасности Azure][security].</span><span class="sxs-lookup"><span data-stu-id="1d4b3-149">For general guidance on designing secure solutions, see the [Azure security documentation][security].</span></span>

## <a name="deploy-the-scenario"></a><span data-ttu-id="1d4b3-150">Развертывание сценария</span><span class="sxs-lookup"><span data-stu-id="1d4b3-150">Deploy the scenario</span></span>

### <a name="prerequisites"></a><span data-ttu-id="1d4b3-151">Предварительные требования</span><span class="sxs-lookup"><span data-stu-id="1d4b3-151">Prerequisites</span></span>

<span data-ttu-id="1d4b3-152">Перед развертыванием шаблона Resource Manager выполните следующие действия.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-152">Follow these steps before deploying the Resource Manager template:</span></span>

1. <span data-ttu-id="1d4b3-153">Создайте [субъект-службу][cycle-svcprin] для получения appId, displayName, имени, пароля и клиента.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-153">Create a [service principal][cycle-svcprin] for retrieving the appId, displayName, name, password, and tenant.</span></span>
2. <span data-ttu-id="1d4b3-154">Создайте [пару ключей SSH][cycle-ssh] для безопасного входа на сервер CycleCloud.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-154">Generate an [SSH key pair][cycle-ssh] to sign in securely to the CycleCloud server.</span></span>

    <!-- markdownlint-disable MD033 -->

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FCycleCloudCommunity%2Fcyclecloud_arm%2Fmaster%2Fazuredeploy.json" target="_blank">
        <img src="https://azuredeploy.net/deploybutton.png"/>
    </a>

    <!-- markdownlint-enable MD033 -->

3. <span data-ttu-id="1d4b3-155">[Войдите на сервер CycleCloud][cycle-login] для настройки и создания нового кластера.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-155">[Log into the CycleCloud server][cycle-login] to configure and create a new cluster.</span></span>
4. <span data-ttu-id="1d4b3-156">[Создайте кластер][cycle-create].</span><span class="sxs-lookup"><span data-stu-id="1d4b3-156">[Create a cluster][cycle-create].</span></span>

<span data-ttu-id="1d4b3-157">Кэш Avere — это дополнительное решение, которое может существенно увеличить пропускную способность при чтении для данных заданий приложения.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-157">The Avere Cache is an optional solution that can drastically increase read throughput for the application job data.</span></span> <span data-ttu-id="1d4b3-158">Avere vFXT для Azure позволяет решить проблему запуска этих корпоративных HPC-приложений в облаке, используя данные, хранящиеся локально или в хранилище BLOB-объектов Azure.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-158">Avere vFXT for Azure solves the problem of running these enterprise HPC applications in the cloud while leveraging data stored on-premises or in Azure Blob storage.</span></span>

<span data-ttu-id="1d4b3-159">Для организаций, в которых запланировано использование гибридной инфраструктуры как с локальным хранилищем, так и с облачными вычислениями, HPC-приложения могут "прорываться" в Azure, используя данные, хранящиеся на NAS-устройствах, и при необходимости развертывать виртуальные ЦП.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-159">For organizations that are planning for a hybrid infrastructure with both on-premises storage and cloud computing, HPC applications can “burst” into Azure using data stored in NAS devices and spin up virtual CPUs as needed.</span></span> <span data-ttu-id="1d4b3-160">Набор данных никогда полностью не переносится в облако.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-160">The data set is never moved completely into the cloud.</span></span> <span data-ttu-id="1d4b3-161">Запрошенные байты временно кэшируются, используя кластер Avere во время обработки.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-161">The requested bytes are temporarily cached using an Avere cluster during processing.</span></span>

<span data-ttu-id="1d4b3-162">Чтобы начать и настроить процесс установки Avere vFXT, выполните инструкции, приведенные в [Avere Setup and Configuration guide][avere] (Руководство по установке и настройке Avere).</span><span class="sxs-lookup"><span data-stu-id="1d4b3-162">To set up and configure an Avere vFXT installation, follow the [Avere Setup and Configuration guide][avere].</span></span>

## <a name="pricing"></a><span data-ttu-id="1d4b3-163">Цены</span><span class="sxs-lookup"><span data-stu-id="1d4b3-163">Pricing</span></span>

<span data-ttu-id="1d4b3-164">Стоимость запуска реализации HPC с помощью сервера CycleCloud будет зависеть от ряда факторов.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-164">The cost of running an HPC implementation using CycleCloud server will vary depending on a number of factors.</span></span> <span data-ttu-id="1d4b3-165">Например, в CycleCloud оплачивается количество времени, затраченного на вычисления. При этом главный сервер и сервер CycleCloud обычно постоянно выделены и находятся в рабочем состоянии.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-165">For example, CycleCloud is charged by the amount of compute time that is used, with the Master and CycleCloud server typically being constantly allocated and running.</span></span> <span data-ttu-id="1d4b3-166">Стоимость запуска узлов выполнения будет зависеть от того, как долго они выполняются, а также от используемого размера.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-166">The cost of running the Execute nodes will depend on how long these are up and running as well as what size is used.</span></span> <span data-ttu-id="1d4b3-167">Обычная плата Azure за использование хранилища и сетевые подключения также взимается.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-167">The normal Azure charges for storage and networking also apply.</span></span>

<span data-ttu-id="1d4b3-168">В этом сценарии показано, как CFD-приложения могут выполняться в Azure, чтобы компьютеры требовали наличие функции RDMA, которая доступна только для определенных размерах виртуальных машин.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-168">This scenario shows how CFD applications can be run in Azure, so the machines will require RDMA functionality, which is only available on specific VM sizes.</span></span> <span data-ttu-id="1d4b3-169">Ниже приведены примеры платы, взымаемой за масштабируемый набор, который непрерывно выделяется на восемь часов в день на месяц с исходящим трафиком данных 1 ТБ.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-169">The following are examples of costs that could be incurred for a scale set that is allocated continuously for eight hours per day for one month, with data egress of 1 TB.</span></span> <span data-ttu-id="1d4b3-170">Сюда также входят цены на сервер Azure CycleCloud и vFXT Avere для установки Azure.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-170">It also includes pricing for the Azure CycleCloud server and the Avere vFXT for Azure install:</span></span>

- <span data-ttu-id="1d4b3-171">Регион: Северная Европа</span><span class="sxs-lookup"><span data-stu-id="1d4b3-171">Region: North Europe</span></span>
- <span data-ttu-id="1d4b3-172">Сервер Azure CycleCloud: 1 Standard D3 (4 ЦП, 14 ГБ памяти, диск HDD категории "Стандартный" на 32 ГБ).</span><span class="sxs-lookup"><span data-stu-id="1d4b3-172">Azure CycleCloud Server: 1 x Standard D3 (4 x CPUs, 14 GB Memory, Standard HDD 32 GB)</span></span>
- <span data-ttu-id="1d4b3-173">Главный сервер Azure CycleCloud: 1 Standard D12 (4 ЦП, 28 ГБ памяти, диск HDD категории "Стандартный" на 32 ГБ).</span><span class="sxs-lookup"><span data-stu-id="1d4b3-173">Azure CycleCloud Master Server: 1 x Standard D12 v (4 x CPUs, 28 GB Memory, Standard HDD 32 GB)</span></span>
- <span data-ttu-id="1d4b3-174">Массив узла Azure CycleCloud: 10 Standard H16r (16 ЦП, 112 ГБ памяти).</span><span class="sxs-lookup"><span data-stu-id="1d4b3-174">Azure CycleCloud Node Array: 10 x Standard H16r (16 x CPUs, 112 GB Memory)</span></span>
- <span data-ttu-id="1d4b3-175">Avere vFXT в кластере Azure: 3 D16s v3 (200 ГБ ОС, диск SSD категории "Премиум" на 1 ТБ).</span><span class="sxs-lookup"><span data-stu-id="1d4b3-175">Avere vFXT on Azure Cluster: 3 x D16s v3 (200 GB OS, Premium SSD 1-TB data disk)</span></span>
- <span data-ttu-id="1d4b3-176">Исходящий трафик данных: 1 TБ</span><span class="sxs-lookup"><span data-stu-id="1d4b3-176">Data Egress: 1 TB</span></span>

<span data-ttu-id="1d4b3-177">Ознакомьтесь с [ценами][pricing] для оборудования, перечисленного выше.</span><span class="sxs-lookup"><span data-stu-id="1d4b3-177">Review this [price estimate][pricing] for the hardware listed above.</span></span>

## <a name="next-steps"></a><span data-ttu-id="1d4b3-178">Дальнейшие действия</span><span class="sxs-lookup"><span data-stu-id="1d4b3-178">Next Steps</span></span>

<span data-ttu-id="1d4b3-179">После развертывания примера подробнее изучите [Azure CycleCloud][cyclecloud].</span><span class="sxs-lookup"><span data-stu-id="1d4b3-179">Once you've deployed the sample, learn more about [Azure CycleCloud][cyclecloud].</span></span>

## <a name="related-resources"></a><span data-ttu-id="1d4b3-180">Связанные ресурсы</span><span class="sxs-lookup"><span data-stu-id="1d4b3-180">Related resources</span></span>

- <span data-ttu-id="1d4b3-181">[Экземпляры с поддержкой RDMA][rdma]</span><span class="sxs-lookup"><span data-stu-id="1d4b3-181">[RDMA Capable Machine Instances][rdma]</span></span>
- <span data-ttu-id="1d4b3-182">[Customizing an RDMA Instance VM][rdma-custom] (Настройка RDMA-экземпляра виртуальной машины)</span><span class="sxs-lookup"><span data-stu-id="1d4b3-182">[Customizing an RDMA Instance VM][rdma-custom]</span></span>

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
