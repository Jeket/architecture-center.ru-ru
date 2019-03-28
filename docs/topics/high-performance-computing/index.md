---
title: Высокопроизводительные вычисления (HPC) в Azure
description: Руководство по созданию выполняющихся рабочих нагрузок HPC в Azure
author: adamboeglin
ms.date: 2/4/2019
---
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD026 -->

# <a name="high-performance-computing-hpc-on-azure"></a><span data-ttu-id="cca09-103">Высокопроизводительные вычисления (HPC) в Azure</span><span class="sxs-lookup"><span data-stu-id="cca09-103">High Performance Computing (HPC) on Azure</span></span>

## <a name="introduction-to-hpc"></a><span data-ttu-id="cca09-104">Общие сведения об HPC</span><span class="sxs-lookup"><span data-stu-id="cca09-104">Introduction to HPC</span></span>

<!-- markdownlint-disable MD034 -->

> [!VIDEO https://www.youtube.com/embed/rKURT32faJk]

<!-- markdownlint-enable MD034 -->

<span data-ttu-id="cca09-105">Высокопроизводительные вычисления (HPC) (т. н. большие вычисления) используют большое количество ЦП или компьютеров на основе GPU для решения сложных математических задач.</span><span class="sxs-lookup"><span data-stu-id="cca09-105">High Performance Computing (HPC), also called "Big Compute", uses a large number of CPU or GPU-based computers to solve complex mathematical tasks.</span></span>

<span data-ttu-id="cca09-106">Во многих отраслях HPC используются для решения самых сложных проблем.</span><span class="sxs-lookup"><span data-stu-id="cca09-106">Many industries use HPC to solve some of their most difficult problems.</span></span>  <span data-ttu-id="cca09-107">Сюда входят следующие рабочие нагрузки:</span><span class="sxs-lookup"><span data-stu-id="cca09-107">These include workloads such as:</span></span>

- <span data-ttu-id="cca09-108">Genomics</span><span class="sxs-lookup"><span data-stu-id="cca09-108">Genomics</span></span>
- <span data-ttu-id="cca09-109">модели для нефтяной и газовой промышленности;</span><span class="sxs-lookup"><span data-stu-id="cca09-109">Oil and gas simulations</span></span>
- <span data-ttu-id="cca09-110">Finance.</span><span class="sxs-lookup"><span data-stu-id="cca09-110">Finance</span></span>
- <span data-ttu-id="cca09-111">разработка полупроводников;</span><span class="sxs-lookup"><span data-stu-id="cca09-111">Semiconductor design</span></span>
- <span data-ttu-id="cca09-112">Engineering</span><span class="sxs-lookup"><span data-stu-id="cca09-112">Engineering</span></span>
- <span data-ttu-id="cca09-113">моделирование погоды.</span><span class="sxs-lookup"><span data-stu-id="cca09-113">Weather modeling</span></span>

### <a name="how-is-hpc-different-on-the-cloud"></a><span data-ttu-id="cca09-114">Чем отличается HPC в облаке?</span><span class="sxs-lookup"><span data-stu-id="cca09-114">How is HPC different on the cloud?</span></span>

<span data-ttu-id="cca09-115">Одним из основных различий между локальной и облачной системами HPC является возможность динамического добавления и удаления ресурсов по мере необходимости.</span><span class="sxs-lookup"><span data-stu-id="cca09-115">One of the primary differences between an on-premise HPC system and one in the cloud is the ability for resources to dynamically be added and removed as they're needed.</span></span>  <span data-ttu-id="cca09-116">Динамическое масштабирование исключает избыточную вычислительную емкость, предоставляя клиентам инфраструктуру требуемого в соответствии с поставленными задачами размера.</span><span class="sxs-lookup"><span data-stu-id="cca09-116">Dynamic scaling removes compute capacity as a bottleneck and instead allow customers to right size their infrastructure for the requirements of their jobs.</span></span>

<span data-ttu-id="cca09-117">Следующие материалы содержат дополнительные сведения об этой возможности динамического масштабирования.</span><span class="sxs-lookup"><span data-stu-id="cca09-117">The following articles provide more detail about this dynamic scaling capability.</span></span>

- [<span data-ttu-id="cca09-118">Стиль архитектуры для больших вычислений</span><span class="sxs-lookup"><span data-stu-id="cca09-118">Big Compute Architecture Style</span></span>](/azure/architecture/guide/architecture-styles/big-compute?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="cca09-119">Рекомендации по автомасштабированию</span><span class="sxs-lookup"><span data-stu-id="cca09-119">Autoscaling best practices</span></span>](/azure/architecture/best-practices/auto-scaling?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

## <a name="implementation-checklist"></a><span data-ttu-id="cca09-120">Контрольный список для реализации</span><span class="sxs-lookup"><span data-stu-id="cca09-120">Implementation checklist</span></span>

<span data-ttu-id="cca09-121">Если вам нужно реализовать собственное решение HPC в Azure, см. следующие материалы:</span><span class="sxs-lookup"><span data-stu-id="cca09-121">As you're looking to implement your own HPC solution on Azure, ensure you're reviewed the following topics:</span></span>

<!-- markdownlint-disable MD032 -->

> [!div class="checklist"]
> - <span data-ttu-id="cca09-122">Выбор соответствующей [архитектуры](#infrastructure) на основе требований.</span><span class="sxs-lookup"><span data-stu-id="cca09-122">Choose the appropriate [architecture](#infrastructure) based on your requirements</span></span>
> - <span data-ttu-id="cca09-123">Выбор [вычислительных ресурсов](#compute) для рабочей нагрузки.</span><span class="sxs-lookup"><span data-stu-id="cca09-123">Know which [compute](#compute) options is right for your workload</span></span>
> - <span data-ttu-id="cca09-124">Определение правильного решения для [хранения](#storage) в соответствии с потребностями.</span><span class="sxs-lookup"><span data-stu-id="cca09-124">Identify the right [storage](#storage) solution that meets your needs</span></span>
> - <span data-ttu-id="cca09-125">Выбор вариантов [управления](#management) всеми ресурсами.</span><span class="sxs-lookup"><span data-stu-id="cca09-125">Decide how you're going to [manage](#management) all your resources</span></span>
> - <span data-ttu-id="cca09-126">Оптимизация [приложений](#hpc-applications) для использования в облаке.</span><span class="sxs-lookup"><span data-stu-id="cca09-126">Optimize your [application](#hpc-applications) for the cloud</span></span>
> - <span data-ttu-id="cca09-127">[Защита](#security) инфраструктуры.</span><span class="sxs-lookup"><span data-stu-id="cca09-127">[Secure](#security) your Infrastructure</span></span>

<!-- markdownlint-enable MD032 -->

## <a name="infrastructure"></a><span data-ttu-id="cca09-128">Инфраструктура</span><span class="sxs-lookup"><span data-stu-id="cca09-128">Infrastructure</span></span>

<span data-ttu-id="cca09-129">Для создания системы HPC используется ряд компонентов инфраструктуры.</span><span class="sxs-lookup"><span data-stu-id="cca09-129">There are a number of infrastructure components necessary to build an HPC system.</span></span>  <span data-ttu-id="cca09-130">Вычислительные ресурсы, а также ресурсы хранилища и сети предоставляют базовые компоненты независимо от выбранного способа управления рабочими нагрузками HPC.</span><span class="sxs-lookup"><span data-stu-id="cca09-130">Compute, Storage, and Networking provide the underlying components, no matter how you choose to manage your HPC workloads.</span></span>

### <a name="example-hpc-architectures"></a><span data-ttu-id="cca09-131">Примеры архитектуры HPC</span><span class="sxs-lookup"><span data-stu-id="cca09-131">Example HPC architectures</span></span>

<span data-ttu-id="cca09-132">Есть разные подходы к проектированию и реализации архитектуры HPC в Azure.</span><span class="sxs-lookup"><span data-stu-id="cca09-132">There are a number of different ways to design and implement your HPC architecture on Azure.</span></span>  <span data-ttu-id="cca09-133">Приложения HPC позволяют масштабировать тысячи вычислительных ядер, расширять локальные кластеры и выполнять полностью облачные решения.</span><span class="sxs-lookup"><span data-stu-id="cca09-133">HPC applications can scale to thousands of compute cores, extend on-premises clusters, or run as a 100% cloud native solution.</span></span>

<span data-ttu-id="cca09-134">Приведенные ниже сценарии описывают некоторые распространенные способы создания решений HPC.</span><span class="sxs-lookup"><span data-stu-id="cca09-134">The following scenarios outline a few of the common ways HPC solutions are built.</span></span>

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/example-scenario/apps/hpc-saas?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="../../example-scenario/apps/media/architecture-hpc-saas.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="cca09-135">Службы автоматизированного проектирования в Azure</span><span class="sxs-lookup"><span data-stu-id="cca09-135">Computer-aided engineering services on Azure</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="cca09-136">Предоставление платформы SaaS (программное обеспечение как услуга) для автоматизированного проектирования (CAE) в Azure.</span><span class="sxs-lookup"><span data-stu-id="cca09-136">Provide a software-as-a-service (SaaS) platform for computer-aided engineering (CAE) on Azure.</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/example-scenario/infrastructure/hpc-cfd?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="../../example-scenario/infrastructure/media/architecture-hpc-cfd.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="cca09-137">Моделирование сценариев вычислительной гидродинамики (CFD) в Azure.</span><span class="sxs-lookup"><span data-stu-id="cca09-137">Computational fluid dynamics (CFD) simulations on Azure</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="cca09-138">Моделирование сценариев CFD в Azure.</span><span class="sxs-lookup"><span data-stu-id="cca09-138">Execute computational fluid dynamics (CFD) simulations on Azure.</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/example-scenario/infrastructure/video-rendering?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="../../example-scenario/infrastructure/media/architecture-video-rendering.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="cca09-139">Отрисовка трехмерного видео на портале Azure</span><span class="sxs-lookup"><span data-stu-id="cca09-139">3D video rendering on Azure</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="cca09-140">Выполнение собственных рабочих нагрузок HPC в Azure с использованием пакетной службы Azure.</span><span class="sxs-lookup"><span data-stu-id="cca09-140">Run native HPC workloads in Azure using the Azure Batch service</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

### <a name="compute"></a><span data-ttu-id="cca09-141">Службы вычислений</span><span class="sxs-lookup"><span data-stu-id="cca09-141">Compute</span></span>

<span data-ttu-id="cca09-142">Azure предлагает решения разных размеров. Все они оптимизированы для рабочих нагрузок, которые потребляют много ресурсов ЦП и GPU.</span><span class="sxs-lookup"><span data-stu-id="cca09-142">Azure offers a range of sizes that are optimized for both CPU & GPU intensive workloads.</span></span>

#### <a name="cpu-based-virtual-machines"></a><span data-ttu-id="cca09-143">Виртуальные машины на основе ЦП</span><span class="sxs-lookup"><span data-stu-id="cca09-143">CPU-based virtual machines</span></span>

- [<span data-ttu-id="cca09-144">Виртуальные машины Linux</span><span class="sxs-lookup"><span data-stu-id="cca09-144">Linux VMs</span></span>](/azure/virtual-machines/linux/sizes-hpc?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- <span data-ttu-id="cca09-145">[Виртуальные машины Windows](/azure/virtual-machines/windows/sizes-hpc?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)</span><span class="sxs-lookup"><span data-stu-id="cca09-145">[Windows VM's](/azure/virtual-machines/windows/sizes-hpc?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) VMs</span></span>
  
#### <a name="gpu-enabled-virtual-machines"></a><span data-ttu-id="cca09-146">Виртуальные машины с поддержкой GPU</span><span class="sxs-lookup"><span data-stu-id="cca09-146">GPU-enabled virtual machines</span></span>

<span data-ttu-id="cca09-147">Виртуальные машины серии N оснащены графическими процессорами NVIDIA и предназначены для приложений с ресурсоемкими вычислениями или графикой, в том числе для обучения искусственного интеллекта (AI) и визуализации.</span><span class="sxs-lookup"><span data-stu-id="cca09-147">N-series VMs feature NVIDIA GPUs designed for compute-intensive or graphics-intensive applications including artificial intelligence (AI) learning and visualization.</span></span>

- [<span data-ttu-id="cca09-148">Виртуальные машины Linux</span><span class="sxs-lookup"><span data-stu-id="cca09-148">Linux VMs</span></span>](/azure/virtual-machines/linux/sizes-gpu?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="cca09-149">Виртуальные машины Windows</span><span class="sxs-lookup"><span data-stu-id="cca09-149">Windows VMs</span></span>](/azure/virtual-machines/windows/sizes-gpu?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

### <a name="storage"></a><span data-ttu-id="cca09-150">Хранилище</span><span class="sxs-lookup"><span data-stu-id="cca09-150">Storage</span></span>

<span data-ttu-id="cca09-151">Масштабные рабочие нагрузки пакетной службы и HPC требуют ресурсов для хранения данных и доступа, которые превышают возможности традиционных файловых систем в облаке.</span><span class="sxs-lookup"><span data-stu-id="cca09-151">Large-scale Batch and HPC workloads have demands for data storage and access that exceed the capabilities of traditional cloud file systems.</span></span>  <span data-ttu-id="cca09-152">Управлять скоростью и емкостью приложений HPC в Azure можно разными способами.</span><span class="sxs-lookup"><span data-stu-id="cca09-152">There are a number of solutions to manage both the speed and capacity needs of HPC applications on Azure</span></span>

- <span data-ttu-id="cca09-153">[Avere vFXT](https://azure.microsoft.com/services/storage/avere-vfxt/) — быстрое и доступное хранилище данных для высокопроизводительных вычислений на пограничном устройстве.</span><span class="sxs-lookup"><span data-stu-id="cca09-153">[Avere vFXT](https://azure.microsoft.com/services/storage/avere-vfxt/) for faster, more accessible data storage for high-performance computing at the edge</span></span>
- <span data-ttu-id="cca09-154">[BeeGFS](https://azure.microsoft.com/resources/implement-glusterfs-on-azure/).</span><span class="sxs-lookup"><span data-stu-id="cca09-154">[BeeGFS](https://azure.microsoft.com/resources/implement-glusterfs-on-azure/)</span></span>
- <span data-ttu-id="cca09-155">[Виртуальные машины, оптимизированные для операций в хранилище](/azure/virtual-machines/windows/sizes-storage?context=/azure/architecture/topics/high-performance-computing/context/hpc-context).</span><span class="sxs-lookup"><span data-stu-id="cca09-155">[Storage Optimized Virtual Machines](/azure/virtual-machines/windows/sizes-storage?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)</span></span>
- [<span data-ttu-id="cca09-156">Хранилище BLOB-объектов, таблиц и очередей</span><span class="sxs-lookup"><span data-stu-id="cca09-156">Blob, table, and queue storage</span></span>](/azure/storage/storage-introduction?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- <span data-ttu-id="cca09-157">[Хранилище файлов SMB Azure](/azure/storage/files/storage-files-introduction?context=/azure/architecture/topics/high-performance-computing/context/hpc-context).</span><span class="sxs-lookup"><span data-stu-id="cca09-157">[Azure SMB File storage](/azure/storage/files/storage-files-introduction?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)</span></span>
- <span data-ttu-id="cca09-158">[Intel Cloud Edition Lustre](https://azuremarketplace.microsoft.com/marketplace/apps/intel.intel-cloud-edition-gs).</span><span class="sxs-lookup"><span data-stu-id="cca09-158">[Intel Cloud Edition Lustre](https://azuremarketplace.microsoft.com/marketplace/apps/intel.intel-cloud-edition-gs)</span></span>

<span data-ttu-id="cca09-159">Дополнительные сведения об использовании Lustre, GlusterFS, и BeeGFS в Azure см. в [электронной книге о параллельных файловых системах в Azure](https://blogs.msdn.microsoft.com/azurecat/2018/06/11/azurecat-ebook-parallel-virtual-file-systems-on-microsoft-azure/).</span><span class="sxs-lookup"><span data-stu-id="cca09-159">For more information comparing Lustre, GlusterFS, and BeeGFS on Azure, review the [Parallel Files Systems on Azure eBook](https://blogs.msdn.microsoft.com/azurecat/2018/06/11/azurecat-ebook-parallel-virtual-file-systems-on-microsoft-azure/)</span></span>

### <a name="networking"></a><span data-ttu-id="cca09-160">Сеть</span><span class="sxs-lookup"><span data-stu-id="cca09-160">Networking</span></span>

<span data-ttu-id="cca09-161">Виртуальные машины H16r, H16mr, A8 и A9 могут подключаться к сети RDMA серверной части с высокой пропускной способностью.</span><span class="sxs-lookup"><span data-stu-id="cca09-161">H16r, H16mr, A8, and A9 VMs can connect to a high throughput back-end RDMA network.</span></span> <span data-ttu-id="cca09-162">Эта сеть может улучшить производительность тесно связанных параллельных приложений, выполняемых на базе Microsoft MPI или Intel MPI.</span><span class="sxs-lookup"><span data-stu-id="cca09-162">This network can improve the performance of tightly coupled parallel applications running under Microsoft MPI or Intel MPI.</span></span>

- [<span data-ttu-id="cca09-163">Экземпляры с поддержкой RDMA</span><span class="sxs-lookup"><span data-stu-id="cca09-163">RDMA Capable Instances</span></span>](/azure/virtual-machines/windows/sizes-hpc?context=/azure/architecture/topics/high-performance-computing/context/hpc-context#rdma-capable-instances)
- [<span data-ttu-id="cca09-164">Виртуальная сеть</span><span class="sxs-lookup"><span data-stu-id="cca09-164">Virtual Network</span></span>](/azure/virtual-network/virtual-networks-overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="cca09-165">ExpressRoute</span><span class="sxs-lookup"><span data-stu-id="cca09-165">ExpressRoute</span></span>](/azure/expressroute/expressroute-introduction?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

## <a name="management"></a><span data-ttu-id="cca09-166">управления</span><span class="sxs-lookup"><span data-stu-id="cca09-166">Management</span></span>

### <a name="do-it-yourself"></a><span data-ttu-id="cca09-167">Модель "Сделай сам"</span><span class="sxs-lookup"><span data-stu-id="cca09-167">Do-it-yourself</span></span>

<span data-ttu-id="cca09-168">Создание системы HPC в Azure с нуля обеспечивает значительную гибкость, но зачастую при этом требуется интенсивное обслуживание.</span><span class="sxs-lookup"><span data-stu-id="cca09-168">Building an HPC system from scratch on Azure offers a significant amount of flexibility, but is often very maintenance intensive.</span></span>  

1. <span data-ttu-id="cca09-169">Настройка собственной кластерной среды на виртуальных машинах Azure или в [масштабируемых наборах виртуальных машин](/azure/virtual-machine-scale-sets/overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context).</span><span class="sxs-lookup"><span data-stu-id="cca09-169">Set up your own cluster environment in Azure virtual machines or [virtual machine scale sets](/azure/virtual-machine-scale-sets/overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context).</span></span>
2. <span data-ttu-id="cca09-170">Использование шаблонов Azure Resource Manager для развертывания лучших [диспетчеров рабочих нагрузок](#workload-managers), инфраструктуры и [приложений](#hpc-applications).</span><span class="sxs-lookup"><span data-stu-id="cca09-170">Use Azure Resource Manager templates to deploy leading [workload managers](#workload-managers), infrastructure, and [applications](#hpc-applications).</span></span>
3. <span data-ttu-id="cca09-171">Выбор [размеров виртуальной машины](#compute) с поддержкой графического процессора и HPC, которые включают специальное оборудование и сетевые подключения для рабочих нагрузок графического процессора или MPI.</span><span class="sxs-lookup"><span data-stu-id="cca09-171">Choose HPC and GPU [VM sizes](#compute) that include specialized hardware and network connections for MPI or GPU workloads.</span></span>
4. <span data-ttu-id="cca09-172">Добавление [высокопроизводительного хранилища](#storage) для рабочих нагрузок с большим количеством операций ввода-вывода.</span><span class="sxs-lookup"><span data-stu-id="cca09-172">Add [high performance storage](#storage) for I/O-intensive workloads.</span></span>

### <a name="hybrid-and-cloud-bursting"></a><span data-ttu-id="cca09-173">Переход в гибридную и облачную среды</span><span class="sxs-lookup"><span data-stu-id="cca09-173">Hybrid and cloud Bursting</span></span>

<span data-ttu-id="cca09-174">Если у вас есть локальная система HPC, которую вы хотите подключить к Azure, вы можете воспользоваться доступными ресурсами, чтобы приступить к работе.</span><span class="sxs-lookup"><span data-stu-id="cca09-174">If you have an existing on-premise HPC system that you'd like to connect to Azure, there are a number of resources to help get you started.</span></span>

<span data-ttu-id="cca09-175">Для начала ознакомьтесь с [вариантами подключения к локальной сети в Azure](/azure/architecture/reference-architectures/hybrid-networking/?context=/azure/architecture/topics/high-performance-computing/context/hpc-context).</span><span class="sxs-lookup"><span data-stu-id="cca09-175">First, review the [Options for connecting an on-premises network to Azure](/azure/architecture/reference-architectures/hybrid-networking/?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) article in the documentation.</span></span>  <span data-ttu-id="cca09-176">После этого вы можете перейти к изучению сведений о доступных возможностях подключения:</span><span class="sxs-lookup"><span data-stu-id="cca09-176">From there, you may want information on these connectivity options:</span></span>

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/reference-architectures/hybrid-networking/vpn?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="/azure/architecture/reference-architectures/hybrid-networking/images/vpn.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="cca09-177">Подключение локальной сети к Azure с помощью VPN-шлюза</span><span class="sxs-lookup"><span data-stu-id="cca09-177">Connect an on-premises network to Azure using a VPN gateway</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="cca09-178">На схеме этой эталонной архитектуры представлены сведения о том, как расширить локальную сеть в Azure с помощью подключения VPN "сеть — сеть".</span><span class="sxs-lookup"><span data-stu-id="cca09-178">This reference architecture shows how to extend an on-premises network to Azure, using a site-to-site virtual private network (VPN).</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/reference-architectures/hybrid-networking/expressroute?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="/azure/architecture/reference-architectures/hybrid-networking/images/expressroute.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="cca09-179">Подключение локальной сети к Azure с помощью ExpressRoute</span><span class="sxs-lookup"><span data-stu-id="cca09-179">Connect an on-premises network to Azure using ExpressRoute</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="cca09-180">Подключения ExpressRoute создают закрытые выделенные подключения через сети сторонних поставщиков услуг подключения.</span><span class="sxs-lookup"><span data-stu-id="cca09-180">ExpressRoute connections use a private, dedicated connection through a third-party connectivity provider.</span></span> <span data-ttu-id="cca09-181">Такое частное подключение расширяет вашу локальную сеть в Azure.</span><span class="sxs-lookup"><span data-stu-id="cca09-181">The private connection extends your on-premises network into Azure.</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/reference-architectures/hybrid-networking/expressroute-vpn-failover?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="/azure/architecture/reference-architectures/hybrid-networking/images/expressroute-vpn-failover.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="cca09-182">Подключение локальной сети к Azure с помощью ExpressRoute с отработкой отказа VPN</span><span class="sxs-lookup"><span data-stu-id="cca09-182">Connect an on-premises network to Azure using ExpressRoute with VPN failover</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="cca09-183">Реализуйте архитектуру высокодоступной защищенной сети типа "сеть — сеть", которая охватывает виртуальную сеть Azure и локальную сеть, подключенные с помощью ExpressRoute с отработкой отказа VPN-шлюза.</span><span class="sxs-lookup"><span data-stu-id="cca09-183">Implement a highly available and secure site-to-site network architecture that spans an Azure virtual network and an on-premises network connected using ExpressRoute with VPN gateway failover.</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

<span data-ttu-id="cca09-184">Установив безопасное подключение к сети, вы можете начать работу, используя облачные вычислительные ресурсы по требованию и возможности расширения, предоставляемые доступным [диспетчером рабочих нагрузок](#workload-managers).</span><span class="sxs-lookup"><span data-stu-id="cca09-184">Once network connectivity is securely established, you can start using cloud compute resources on-demand with the bursting capabilities of your existing [workload manager](#workload-managers).</span></span>

### <a name="marketplace-solutions"></a><span data-ttu-id="cca09-185">Решения Marketplace</span><span class="sxs-lookup"><span data-stu-id="cca09-185">Marketplace solutions</span></span>

<span data-ttu-id="cca09-186">В [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/) предлагаются разные диспетчеры рабочих нагрузок.</span><span class="sxs-lookup"><span data-stu-id="cca09-186">There are a number of workload managers offered in the [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/).</span></span>

- [<span data-ttu-id="cca09-187">HPC RogueWave на основе CentOS</span><span class="sxs-lookup"><span data-stu-id="cca09-187">RogueWave CentOS-based HPC</span></span>](https://azuremarketplace.microsoft.com/marketplace/apps/RogueWave.CentOSbased73HPC?tab=Overview)
- [<span data-ttu-id="cca09-188">SUSE Linux Enterprise Server для HPC</span><span class="sxs-lookup"><span data-stu-id="cca09-188">SUSE Linux Enterprise Server for HPC</span></span>](https://azure.microsoft.com/marketplace/partners/suse/suselinuxenterpriseserver12optimizedforhighperformancecompute/)
- <span data-ttu-id="cca09-189">[TIBCO Grid Server Engine](https://azuremarketplace.microsoft.com/marketplace/apps/tibco-software.gridserverlinuxengine?tab=Overview);</span><span class="sxs-lookup"><span data-stu-id="cca09-189">[TIBCO Grid Server Engine](https://azuremarketplace.microsoft.com/marketplace/apps/tibco-software.gridserverlinuxengine?tab=Overview)</span></span>
- [<span data-ttu-id="cca09-190">Виртуальные машины для анализа и обработки данных для Windows и Linux</span><span class="sxs-lookup"><span data-stu-id="cca09-190">Azure Data Science VM for Windows and Linux</span></span>](/azure/machine-learning/data-science-virtual-machine/overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- <span data-ttu-id="cca09-191">[d3View](https://azuremarketplace.microsoft.com/marketplace/apps/xfinityinc.d3view-v5?tab=Overview);</span><span class="sxs-lookup"><span data-stu-id="cca09-191">[D3View](https://azuremarketplace.microsoft.com/marketplace/apps/xfinityinc.d3view-v5?tab=Overview)</span></span>
- [<span data-ttu-id="cca09-192">UberCloud</span><span class="sxs-lookup"><span data-stu-id="cca09-192">UberCloud</span></span>](https://azure.microsoft.com/search/marketplace/?q=ubercloud)

### <a name="azure-batch"></a><span data-ttu-id="cca09-193">Пакетная служба Azure</span><span class="sxs-lookup"><span data-stu-id="cca09-193">Azure Batch</span></span>

<span data-ttu-id="cca09-194">[Пакетная служба Azure](/azure/batch/batch-technical-overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) — это служба платформы, которая позволяет эффективно работать с приложениями для крупномасштабных параллельных и высокопроизводительных вычислений (HPC) в облаке.</span><span class="sxs-lookup"><span data-stu-id="cca09-194">[Azure Batch](/azure/batch/batch-technical-overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) is a platform service for running large-scale parallel and high-performance computing (HPC) applications efficiently in the cloud.</span></span> <span data-ttu-id="cca09-195">Пакетная служба Azure планирует запуск ресурсоемких вычислительных задач в управляемом пуле виртуальных машин и автоматически масштабирует вычислительные ресурсы, учитывая требования заданий.</span><span class="sxs-lookup"><span data-stu-id="cca09-195">Azure Batch schedules compute-intensive work to run on a managed pool of virtual machines, and can automatically scale compute resources to meet the needs of your jobs.</span></span>

<span data-ttu-id="cca09-196">Разработчики или поставщики SaaS могут использовать пакеты SDK для пакетной службы и средства для интеграции приложений HPC или контейнерных рабочих нагрузок с Azure, промежуточного хранения данных в Azure и создания конвейеров выполнения заданий.</span><span class="sxs-lookup"><span data-stu-id="cca09-196">SaaS providers or developers can use the Batch SDKs and tools to integrate HPC applications or container workloads with Azure, stage data to Azure, and build job execution pipelines.</span></span>

### <a name="azure-cyclecloud"></a><span data-ttu-id="cca09-197">Azure CycleCloud</span><span class="sxs-lookup"><span data-stu-id="cca09-197">Azure CycleCloud</span></span>

<span data-ttu-id="cca09-198">[Azure CycleCloud](/azure/cyclecloud/?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) — самый простой способ управлять рабочими нагрузками HPC в Azure с помощью любого планировщика, например Slurm, Grid Engine, HPC Pack, HTCondor, LSF, PBS Pro или Symphony.</span><span class="sxs-lookup"><span data-stu-id="cca09-198">[Azure CycleCloud](/azure/cyclecloud/?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) Provides the simplest way to manage HPC workloads using any scheduler (like Slurm, Grid Engine, HPC Pack, HTCondor, LSF, PBS Pro, or Symphony), on Azure</span></span>

<span data-ttu-id="cca09-199">CycleCloud позволяет:</span><span class="sxs-lookup"><span data-stu-id="cca09-199">CycleCloud allows you to:</span></span>

- <span data-ttu-id="cca09-200">развертывать полные кластеры и другие ресурсы, например планировщики, вычислительные виртуальные машины, хранилища, сети и кэш;</span><span class="sxs-lookup"><span data-stu-id="cca09-200">Deploy full clusters and other resources, including scheduler, compute VMs, storage, networking, and cache</span></span>
- <span data-ttu-id="cca09-201">координировать рабочие процессы, связанные с заданиями, данными и облаком;</span><span class="sxs-lookup"><span data-stu-id="cca09-201">Orchestrate job, data, and cloud workflows</span></span>
- <span data-ttu-id="cca09-202">предоставлять администраторам полный контроль над тем, какие пользователи могут запускать задания, а также где они могут это делать и с какими затратами;</span><span class="sxs-lookup"><span data-stu-id="cca09-202">Give admins full control over which users can run jobs, as well as where and at what cost</span></span>
- <span data-ttu-id="cca09-203">настраивать и оптимизировать кластеры с помощью расширенных политик и системы управления, включая интеграцию с Active Directory, средства контроля затрат, а также инструменты для мониторинга и отчетности;</span><span class="sxs-lookup"><span data-stu-id="cca09-203">Customize and optimize clusters through advanced policy and governance features, including cost controls, Active Directory integration, monitoring, and reporting</span></span>
- <span data-ttu-id="cca09-204">использовать доступные планировщики и приложения, не внося в них какие-либо изменения;</span><span class="sxs-lookup"><span data-stu-id="cca09-204">Use your current job scheduler and applications without modification</span></span>
- <span data-ttu-id="cca09-205">использовать встроенные возможности автомасштабирования, а также проверенные на практике эталонные архитектуры для целого ряда отраслей и рабочих нагрузок HPC.</span><span class="sxs-lookup"><span data-stu-id="cca09-205">Take advantage of built-in autoscaling and battle-tested reference architectures for a wide range of HPC workloads and industries</span></span>

### <a name="workload-managers"></a><span data-ttu-id="cca09-206">Диспетчеры рабочих нагрузок</span><span class="sxs-lookup"><span data-stu-id="cca09-206">Workload managers</span></span>

<span data-ttu-id="cca09-207">Ниже приведены примеры диспетчеров рабочих нагрузок и кластеров, которые могут выполняться в инфраструктуре Azure.</span><span class="sxs-lookup"><span data-stu-id="cca09-207">The following are examples of cluster and workload managers that can run in Azure infrastructure.</span></span> <span data-ttu-id="cca09-208">Создавайте автономные кластеры на виртуальных машинах Azure или переносите нагрузки из локального кластера на виртуальные машины Azure.</span><span class="sxs-lookup"><span data-stu-id="cca09-208">Create stand-alone clusters in Azure VMs or burst to Azure VMs from an on-premises cluster.</span></span>

- [<span data-ttu-id="cca09-209">Alces Flight Compute</span><span class="sxs-lookup"><span data-stu-id="cca09-209">Alces Flight Compute</span></span>](https://azuremarketplace.microsoft.com/marketplace/apps/alces-flight-limited.alces-flight-compute-solo?tab=Overview)
- [<span data-ttu-id="cca09-210">TIBCO DataSynapse GridServer</span><span class="sxs-lookup"><span data-stu-id="cca09-210">TIBCO DataSynapse GridServer</span></span>](https://azure.microsoft.com/blog/tibco-datasynapse-comes-to-the-azure-marketplace/)
- [<span data-ttu-id="cca09-211">Bright Cluster Manager</span><span class="sxs-lookup"><span data-stu-id="cca09-211">Bright Cluster Manager</span></span>](http://www.brightcomputing.com/technology-partners/microsoft)
- [<span data-ttu-id="cca09-212">IBM Spectrum Symphony и Symphony LSF</span><span class="sxs-lookup"><span data-stu-id="cca09-212">IBM Spectrum Symphony and Symphony LSF</span></span>](https://azure.microsoft.com/blog/ibm-and-microsoft-azure-support-spectrum-symphony-and-spectrum-lsf/)
- <span data-ttu-id="cca09-213">[PBS Pro](http://pbspro.org);</span><span class="sxs-lookup"><span data-stu-id="cca09-213">[PBS Pro](http://pbspro.org)</span></span>
- [<span data-ttu-id="cca09-214">Altair</span><span class="sxs-lookup"><span data-stu-id="cca09-214">Altair</span></span>](http://www.altair.com/)
- [<span data-ttu-id="cca09-215">Rescale</span><span class="sxs-lookup"><span data-stu-id="cca09-215">Rescale</span></span>](https://www.rescale.com/azure/)
- [<span data-ttu-id="cca09-216">Пакет высокопроизводительных вычислений Microsoft</span><span class="sxs-lookup"><span data-stu-id="cca09-216">Microsoft HPC Pack</span></span>](https://technet.microsoft.com/library/mt744885.aspx)
  - [<span data-ttu-id="cca09-217">Пакет HPC для Windows</span><span class="sxs-lookup"><span data-stu-id="cca09-217">HPC Pack for Windows</span></span>](/azure/virtual-machines/windows/hpcpack-cluster-options?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
  - [<span data-ttu-id="cca09-218">Пакет HPC для Linux</span><span class="sxs-lookup"><span data-stu-id="cca09-218">HPC Pack for Linux</span></span>](/azure/virtual-machines/linux/hpcpack-cluster-options?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

#### <a name="containers"></a><span data-ttu-id="cca09-219">Контейнеры</span><span class="sxs-lookup"><span data-stu-id="cca09-219">Containers</span></span>

<span data-ttu-id="cca09-220">Для управления некоторыми рабочими нагрузками HPC также можно использовать контейнеры.</span><span class="sxs-lookup"><span data-stu-id="cca09-220">Containers can also be used to manage some HPC workloads.</span></span>  <span data-ttu-id="cca09-221">Такие решения, как Служба Azure Kubernetes (AKS), упрощают развертывание управляемого кластера Kubernetes в Azure.</span><span class="sxs-lookup"><span data-stu-id="cca09-221">Services like the Azure Kubernetes Service (AKS) makes it simple to deploy a managed Kubernetes cluster in Azure.</span></span>

- [<span data-ttu-id="cca09-222">Служба Azure Kubernetes (AKS)</span><span class="sxs-lookup"><span data-stu-id="cca09-222">Azure Kubernetes Service (AKS)</span></span>](/azure/aks/intro-kubernetes?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="cca09-223">Реестр контейнеров</span><span class="sxs-lookup"><span data-stu-id="cca09-223">Container Registry</span></span>](/azure/container-registry/container-registry-intro?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

## <a name="cost-management"></a><span data-ttu-id="cca09-224">управления затратами;</span><span class="sxs-lookup"><span data-stu-id="cca09-224">Cost management</span></span>

<span data-ttu-id="cca09-225">Управление затратами HPC в Azure может осуществляться разными способами.</span><span class="sxs-lookup"><span data-stu-id="cca09-225">Managing your HPC cost on Azure can be done through a few different ways.</span></span>  <span data-ttu-id="cca09-226">Чтобы определить наиболее подходящий для вас способ, ознакомьтесь с [вариантами приобретения Azure](https://azure.microsoft.com/pricing/purchase-options/).</span><span class="sxs-lookup"><span data-stu-id="cca09-226">Ensure you've reviewed the [Azure purchasing options](https://azure.microsoft.com/pricing/purchase-options/) to find the method that works best for your organization.</span></span>

<span data-ttu-id="cca09-227">[Низкоприоритетные виртуальные машины](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-use-low-priority?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) позволяют работать с нашей неиспользуемой емкостью при значительной экономии затрат.</span><span class="sxs-lookup"><span data-stu-id="cca09-227">[Low priority VMs](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-use-low-priority?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) allow you to take advantage of our unutilized capacity at a significant cost savings.</span></span>

## <a name="security"></a><span data-ttu-id="cca09-228">Безопасность</span><span class="sxs-lookup"><span data-stu-id="cca09-228">Security</span></span>

<span data-ttu-id="cca09-229">Общие сведения об обеспечении безопасности в Azure см. в [документации по системе безопасности Azure](/azure/security/azure-security?context=/azure/architecture/topics/high-performance-computing/context/hpc-context).</span><span class="sxs-lookup"><span data-stu-id="cca09-229">For an overview of security best practices on Azure, review the [Azure Security Documentation](/azure/security/azure-security?context=/azure/architecture/topics/high-performance-computing/context/hpc-context).</span></span>  

<span data-ttu-id="cca09-230">В дополнение к сетевым конфигурациям, описанным в разделе о [выходе в облако](#hybrid-and-cloud-bursting), вам может потребоваться реализовать звездообразную топологию для изоляции вычислительных ресурсов:</span><span class="sxs-lookup"><span data-stu-id="cca09-230">In addition to the network configurations available in the [Cloud Bursting](#hybrid-and-cloud-bursting) section, you may want to implement a hub/spoke configuration to isolate your compute resources:</span></span>

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/reference-architectures/hybrid-networking/hub-spoke?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="/azure/architecture/reference-architectures/hybrid-networking/images/hub-spoke.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="cca09-231">Реализация звездообразной топологии сети в Azure</span><span class="sxs-lookup"><span data-stu-id="cca09-231">Implement a hub-spoke network topology in Azure</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="cca09-232">Концентратор (центр топологии) — это виртуальная сеть в Azure, которая выступает в качестве центральной точки подключения к локальной сети.</span><span class="sxs-lookup"><span data-stu-id="cca09-232">The hub is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="cca09-233">Периферийные зоны — это виртуальные сети, которые устанавливают пиринг с концентратором и могут использоваться для изоляции рабочих нагрузок.</span><span class="sxs-lookup"><span data-stu-id="cca09-233">The spokes are VNets that peer with the hub, and can be used to isolate workloads.</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/reference-architectures/hybrid-networking/shared-services?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="/azure/architecture/reference-architectures/hybrid-networking/images/shared-services.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="cca09-234">Реализация звездообразной топологии сети с помощью общих служб в Azure</span><span class="sxs-lookup"><span data-stu-id="cca09-234">Implement a hub-spoke network topology with shared services in Azure</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="cca09-235">Эта эталонная архитектура создана на основе типовой звездообразной топологии. Она позволяет настроить в концентраторе общие службы, которые можно использовать во всех периферийных зонах.</span><span class="sxs-lookup"><span data-stu-id="cca09-235">This reference architecture builds on the hub-spoke reference architecture to include shared services in the hub that can be consumed by all spokes.</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

## <a name="hpc-applications"></a><span data-ttu-id="cca09-236">Приложения HPC</span><span class="sxs-lookup"><span data-stu-id="cca09-236">HPC applications</span></span>

<span data-ttu-id="cca09-237">Запустите пользовательские или коммерческие приложения HPC в Azure.</span><span class="sxs-lookup"><span data-stu-id="cca09-237">Run custom or commercial HPC applications in Azure.</span></span> <span data-ttu-id="cca09-238">Некоторые приложения из этого раздела могут эффективно масштабироваться с помощью дополнительных виртуальных машин или вычислительных ядер.</span><span class="sxs-lookup"><span data-stu-id="cca09-238">Several examples in this section are benchmarked to scale efficiently with additional VMs or compute cores.</span></span> <span data-ttu-id="cca09-239">Чтобы получить готовые к развертыванию решения, посетите [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace).</span><span class="sxs-lookup"><span data-stu-id="cca09-239">Visit the [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace) for ready-to-deploy solutions.</span></span>

> [!NOTE]
> <span data-ttu-id="cca09-240">Проконсультируйтесь с поставщиками всех коммерческих приложений насчет лицензирования или иных ограничений на запуск приложений в облаке.</span><span class="sxs-lookup"><span data-stu-id="cca09-240">Check with the vendor of any commercial application for licensing or other restrictions for running in the cloud.</span></span> <span data-ttu-id="cca09-241">Не все поставщики предлагают лицензирование с оплатой по мере использования.</span><span class="sxs-lookup"><span data-stu-id="cca09-241">Not all vendors offer pay-as-you-go licensing.</span></span> <span data-ttu-id="cca09-242">Для вашего решения может потребоваться сервер лицензий в облаке или локальный сервер лицензий.</span><span class="sxs-lookup"><span data-stu-id="cca09-242">You might need a licensing server in the cloud for your solution, or connect to an on-premises license server.</span></span>

### <a name="engineering-applications"></a><span data-ttu-id="cca09-243">Проектирование приложений</span><span class="sxs-lookup"><span data-stu-id="cca09-243">Engineering applications</span></span>

- [<span data-ttu-id="cca09-244">Altair RADIOSS</span><span class="sxs-lookup"><span data-stu-id="cca09-244">Altair RADIOSS</span></span>](https://azure.microsoft.com/blog/availability-of-altair-radioss-rdma-on-microsoft-azure/)
- [<span data-ttu-id="cca09-245">ANSYS CFD</span><span class="sxs-lookup"><span data-stu-id="cca09-245">ANSYS CFD</span></span>](https://azure.microsoft.com/blog/ansys-cfd-and-microsoft-azure-perform-the-best-hpc-scalability-in-the-cloud/)
- [<span data-ttu-id="cca09-246">MATLAB Distributed Computing Server</span><span class="sxs-lookup"><span data-stu-id="cca09-246">MATLAB Distributed Computing Server</span></span>](/azure/virtual-machines/windows/matlab-mdcs-cluster?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="cca09-247">StarCCM+</span><span class="sxs-lookup"><span data-stu-id="cca09-247">StarCCM+</span></span>](https://blogs.msdn.microsoft.com/azurecat/2017/07/07/run-star-ccm-in-an-azure-hpc-cluster/)
- [<span data-ttu-id="cca09-248">OpenFOAM</span><span class="sxs-lookup"><span data-stu-id="cca09-248">OpenFOAM</span></span>](https://simulation.azure.com/casestudies/Team-182-ABB-UC-Final.pdf)

### <a name="graphics-and-rendering"></a><span data-ttu-id="cca09-249">Графика и отрисовка</span><span class="sxs-lookup"><span data-stu-id="cca09-249">Graphics and rendering</span></span>

- <span data-ttu-id="cca09-250">[Autodesk Maya, 3ds Max и Arnold](/azure/batch/batch-rendering-service?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) в пакетной службе Azure</span><span class="sxs-lookup"><span data-stu-id="cca09-250">[Autodesk Maya, 3ds Max, and Arnold](/azure/batch/batch-rendering-service?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) on Azure Batch</span></span>

### <a name="ai-and-deep-learning"></a><span data-ttu-id="cca09-251">Искусственный интеллект и глубокое обучение</span><span class="sxs-lookup"><span data-stu-id="cca09-251">AI and deep learning</span></span>

- [<span data-ttu-id="cca09-252">Microsoft Cognitive Toolkit</span><span class="sxs-lookup"><span data-stu-id="cca09-252">Microsoft Cognitive Toolkit</span></span>](/cognitive-toolkit/cntk-on-azure)
- [<span data-ttu-id="cca09-253">Виртуальная машина для глубокого обучения</span><span class="sxs-lookup"><span data-stu-id="cca09-253">Deep Learning VM</span></span>](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-ads.dsvm-deep-learning)
- [<span data-ttu-id="cca09-254">Инструкции Batch Shipyard для глубокого обучения</span><span class="sxs-lookup"><span data-stu-id="cca09-254">Batch Shipyard recipes for deep learning</span></span>](https://github.com/Azure/batch-shipyard/tree/master/recipes#deeplearning)

### <a name="mpi-providers"></a><span data-ttu-id="cca09-255">Поставщики MPI</span><span class="sxs-lookup"><span data-stu-id="cca09-255">MPI Providers</span></span>

- [<span data-ttu-id="cca09-256">Microsoft MPI</span><span class="sxs-lookup"><span data-stu-id="cca09-256">Microsoft MPI</span></span>](/message-passing-interface/microsoft-mpi)

## <a name="remote-visualization"></a><span data-ttu-id="cca09-257">Удаленная визуализация</span><span class="sxs-lookup"><span data-stu-id="cca09-257">Remote visualization</span></span>

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/example-scenario/infrastructure/linux-vdi-citrix?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="../../example-scenario/infrastructure/media/azure-citrix-sample-diagram.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary"><span data-ttu-id="cca09-258">Виртуальный рабочий стол Linux с использованием Citrix</span><span class="sxs-lookup"><span data-stu-id="cca09-258">Linux virtual desktops with Citrix</span></span></h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p><span data-ttu-id="cca09-259">Создание среды VDI для настольных компьютеров Linux с помощью Citrix в Azure.</span><span class="sxs-lookup"><span data-stu-id="cca09-259">Build a VDI environment for Linux Desktops using Citrix on Azure.</span></span></p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

## <a name="performance-benchmarks"></a><span data-ttu-id="cca09-260">Тесты производительности</span><span class="sxs-lookup"><span data-stu-id="cca09-260">Performance Benchmarks</span></span>

- [<span data-ttu-id="cca09-261">Тесты вычислительных ресурсов</span><span class="sxs-lookup"><span data-stu-id="cca09-261">Compute benchmarks</span></span>](/azure/virtual-machines/windows/compute-benchmark-scores?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

## <a name="customer-stories"></a><span data-ttu-id="cca09-262">Истории клиентов</span><span class="sxs-lookup"><span data-stu-id="cca09-262">Customer stories</span></span>

<span data-ttu-id="cca09-263">Многие клиенты достигли значительного успеха при использовании Azure для рабочих нагрузок HPC.</span><span class="sxs-lookup"><span data-stu-id="cca09-263">There are a number of customers who have seen great success by using Azure for their HPC workloads.</span></span>  <span data-ttu-id="cca09-264">Некоторые примеры представлены ниже.</span><span class="sxs-lookup"><span data-stu-id="cca09-264">You can find a few of these customer case studies below:</span></span>

- [<span data-ttu-id="cca09-265">ANEO</span><span class="sxs-lookup"><span data-stu-id="cca09-265">ANEO</span></span>](https://customers.microsoft.com/story/it-provider-finds-highly-scalable-cloud-based-hpc-redu)
- [<span data-ttu-id="cca09-266">AXA Global P&C</span><span class="sxs-lookup"><span data-stu-id="cca09-266">AXA Global P&C</span></span>](https://customers.microsoft.com/story/axa-global-p-and-c)
- [<span data-ttu-id="cca09-267">Axioma</span><span class="sxs-lookup"><span data-stu-id="cca09-267">Axioma</span></span>](https://customers.microsoft.com/story/axioma-delivers-fintechs-first-born-in-the-cloud-multi-asset-class-enterprise-risk-solution)
- [<span data-ttu-id="cca09-268">d3View</span><span class="sxs-lookup"><span data-stu-id="cca09-268">d3View</span></span>](https://customers.microsoft.com/story/big-data-solution-provider-adopts-new-cloud-gains-thou)
- [<span data-ttu-id="cca09-269">EFS</span><span class="sxs-lookup"><span data-stu-id="cca09-269">EFS</span></span>](https://customers.microsoft.com/story/efs-professionalservices-azure)
- [<span data-ttu-id="cca09-270">Hymans Robertson</span><span class="sxs-lookup"><span data-stu-id="cca09-270">Hymans Robertson</span></span>](https://customers.microsoft.com/story/hymans-robertson)
- [<span data-ttu-id="cca09-271">MetLife</span><span class="sxs-lookup"><span data-stu-id="cca09-271">MetLife</span></span>](https://enterprise.microsoft.com/customer-story/industries/insurance/metlife/)
- [<span data-ttu-id="cca09-272">Microsoft Research</span><span class="sxs-lookup"><span data-stu-id="cca09-272">Microsoft Research</span></span>](https://customers.microsoft.com/doclink/fast-lmm-and-windows-azure-put-genetics-research-on-fa)
- [<span data-ttu-id="cca09-273">Milliman</span><span class="sxs-lookup"><span data-stu-id="cca09-273">Milliman</span></span>](https://customers.microsoft.com/story/actuarial-firm-works-to-transform-insurance-industry-w)
- [<span data-ttu-id="cca09-274">Mitsubishi UFJ Securities International</span><span class="sxs-lookup"><span data-stu-id="cca09-274">Mitsubishi UFJ Securities International</span></span>](https://customers.microsoft.com/story/powering-risk-compute-grids-in-the-cloud)
- [<span data-ttu-id="cca09-275">NeuroInitiative</span><span class="sxs-lookup"><span data-stu-id="cca09-275">NeuroInitiative</span></span>](https://customers.microsoft.com/story/neuroinitiative-health-provider-azure)
- [<span data-ttu-id="cca09-276">Schlumberger</span><span class="sxs-lookup"><span data-stu-id="cca09-276">Schlumberger</span></span>](https://azure.microsoft.com/blog/big-compute-for-large-engineering-simulations)
- [<span data-ttu-id="cca09-277">Towers Watson</span><span class="sxs-lookup"><span data-stu-id="cca09-277">Towers Watson</span></span>](https://customers.microsoft.com/story/insurance-tech-provider-delivers-disruptive-solutions)

## <a name="other-important-information"></a><span data-ttu-id="cca09-278">Другие важные сведения</span><span class="sxs-lookup"><span data-stu-id="cca09-278">Other important information</span></span>

- <span data-ttu-id="cca09-279">Убедитесь, что ваша [квота на виртуальные ЦП](/azure/virtual-machines/linux/quotas?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) увеличена, прежде чем выполнять крупномасштабные рабочие нагрузки.</span><span class="sxs-lookup"><span data-stu-id="cca09-279">Ensure your [vCPU quota](/azure/virtual-machines/linux/quotas?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) has been increased before attempting to run large-scale workloads.</span></span>

## <a name="next-steps"></a><span data-ttu-id="cca09-280">Дополнительная информация</span><span class="sxs-lookup"><span data-stu-id="cca09-280">Next steps</span></span>

<span data-ttu-id="cca09-281">См. последние объявления:</span><span class="sxs-lookup"><span data-stu-id="cca09-281">For the latest announcements, see:</span></span>

- [<span data-ttu-id="cca09-282">Блог группы Microsoft HPC и пакетной службы</span><span class="sxs-lookup"><span data-stu-id="cca09-282">Microsoft HPC and Batch team blog</span></span>](http://blogs.technet.com/b/windowshpc/)
- <span data-ttu-id="cca09-283">Ознакомьтесь с [блогом, посвященным Azure](https://azure.microsoft.com/blog/tag/hpc/).</span><span class="sxs-lookup"><span data-stu-id="cca09-283">Visit the [Azure blog](https://azure.microsoft.com/blog/tag/hpc/).</span></span>

### <a name="microsoft-batch-examples"></a><span data-ttu-id="cca09-284">Примеры заданий пакетной службы Microsoft</span><span class="sxs-lookup"><span data-stu-id="cca09-284">Microsoft Batch Examples</span></span>

<span data-ttu-id="cca09-285">Эти руководства содержат сведения о выполняющихся приложениях в пакетной службе Microsoft</span><span class="sxs-lookup"><span data-stu-id="cca09-285">These tutorials will provide you with details on running applications on Microsoft Batch</span></span>

- [<span data-ttu-id="cca09-286">Сведения о начале разработки с помощью пакетной службы</span><span class="sxs-lookup"><span data-stu-id="cca09-286">Get started developing with Batch</span></span>](/azure/batch/quick-run-dotnet?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="cca09-287">Примеры использования кода пакетной службы</span><span class="sxs-lookup"><span data-stu-id="cca09-287">Use Azure Batch code samples</span></span>](https://github.com/Azure/azure-batch-samples)
- [<span data-ttu-id="cca09-288">Использование низкоприоритетных виртуальных машин в пакетной службе</span><span class="sxs-lookup"><span data-stu-id="cca09-288">Use low-priority VMs with Batch</span></span>](/azure/batch/batch-low-pri-vms?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [<span data-ttu-id="cca09-289">Сведения о запуске контейнерных рабочих нагрузок HPC с помощью Batch Shipyard</span><span class="sxs-lookup"><span data-stu-id="cca09-289">Run containerized HPC workloads with Batch Shipyard</span></span>](https://github.com/Azure/batch-shipyard)
- [<span data-ttu-id="cca09-290">Выполнение параллельных рабочих нагрузок R в пакетной службе</span><span class="sxs-lookup"><span data-stu-id="cca09-290">Run parallel R workloads on Batch</span></span>](https://github.com/Azure/doAzureParallel)
- [<span data-ttu-id="cca09-291">Запуск заданий Spark по требованию в пакетной службе</span><span class="sxs-lookup"><span data-stu-id="cca09-291">Run on-demand Spark jobs on Batch</span></span>](https://github.com/Azure/aztk)
- [<span data-ttu-id="cca09-292">Использование экземпляров с поддержкой RDMA или графического процессора (GPU) в пулах пакетной службы</span><span class="sxs-lookup"><span data-stu-id="cca09-292">Use compute-intensive VMs in Batch pools</span></span>](/azure/batch/batch-pool-compute-intensive-sizes?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)