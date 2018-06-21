---
title: Стиль архитектуры для больших вычислений
description: В этой статье описаны преимущества и сложности архитектуры больших вычислений в Azure, а также содержатся рекомендации по ее разработке
author: MikeWasson
ms.openlocfilehash: b16be4133143d7d73062eeb280b44779c390f387
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/14/2017
ms.locfileid: "24539790"
---
# <a name="big-compute-architecture-style"></a><span data-ttu-id="806cc-103">Стиль архитектуры для больших вычислений</span><span class="sxs-lookup"><span data-stu-id="806cc-103">Big compute architecture style</span></span>

<span data-ttu-id="806cc-104">Термин *большие вычисления* означает крупномасштабные рабочие нагрузки, для которых нужно большое количество ядер — сотни или даже тысячи.</span><span class="sxs-lookup"><span data-stu-id="806cc-104">The term *big compute* describes large-scale workloads that require a large number of cores, often numbering in the hundreds or thousands.</span></span> <span data-ttu-id="806cc-105">Такая потребность возникает в разных сценариях, например при обработке изображений, оценке динамики жидкостей, моделировании финансовых рисков, в расчетах для нефтяной и фармацевтической отраслей и при анализе расчетного напряжения.</span><span class="sxs-lookup"><span data-stu-id="806cc-105">Scenarios include image rendering, fluid dynamics, financial risk modeling, oil exploration, drug design, and engineering stress analysis, among others.</span></span>

![](./images/big-compute-logical.png)

<span data-ttu-id="806cc-106">Ниже приведены стандартные характеристики приложений для больших вычислений:</span><span class="sxs-lookup"><span data-stu-id="806cc-106">Here are some typical characteristics of big compute applications:</span></span>

- <span data-ttu-id="806cc-107">Работу можно разделить на дискретные задачи, которые будут одновременно выполняться в множестве ядер.</span><span class="sxs-lookup"><span data-stu-id="806cc-107">The work can be split into discrete tasks, which can be run across many cores simultaneously.</span></span>
- <span data-ttu-id="806cc-108">Время выполнения каждой задачи ограничено.</span><span class="sxs-lookup"><span data-stu-id="806cc-108">Each task is finite.</span></span> <span data-ttu-id="806cc-109">Принимаются входные данные, выполняется обработка и выводятся результаты.</span><span class="sxs-lookup"><span data-stu-id="806cc-109">It takes some input, does some processing, and produces output.</span></span> <span data-ttu-id="806cc-110">Приложение в целом выполняется в течение конечного периода (от нескольких минут до нескольких дней).</span><span class="sxs-lookup"><span data-stu-id="806cc-110">The entire application runs for a finite amount of time (minutes to days).</span></span> <span data-ttu-id="806cc-111">Такие задачи обычно решаются единовременным выделением большого количества ядер, число которых постепенно сводится к нулю по мере выполнения приложения.</span><span class="sxs-lookup"><span data-stu-id="806cc-111">A common pattern is to provision a large number of cores in a burst, and then spin down to zero once the application completes.</span></span> 
- <span data-ttu-id="806cc-112">Такому приложению не обязательно работать круглосуточно и ежедневно.</span><span class="sxs-lookup"><span data-stu-id="806cc-112">The application does not need to stay up 24/7.</span></span> <span data-ttu-id="806cc-113">Но система должна быть готова к сбоям узлов или самого приложения.</span><span class="sxs-lookup"><span data-stu-id="806cc-113">However, the system must handle node failures or application crashes.</span></span>
- <span data-ttu-id="806cc-114">Задачи приложения могут быть независимыми и выполняться параллельно.</span><span class="sxs-lookup"><span data-stu-id="806cc-114">For some applications, tasks are independent and can run in parallel.</span></span> <span data-ttu-id="806cc-115">Или же они могут быть тесно взаимосвязаны, то есть должны взаимодействовать в процессе выполнения или обмениваться промежуточными результатами.</span><span class="sxs-lookup"><span data-stu-id="806cc-115">In other cases, tasks are tightly coupled, meaning they must interact or exchange intermediate results.</span></span> <span data-ttu-id="806cc-116">Для такого сценария можно применить технологии высокоскоростной сети, например InfiniBand и удаленный доступ к памяти (RDMA).</span><span class="sxs-lookup"><span data-stu-id="806cc-116">In that case, consider using high-speed networking technologies such as InfiniBand and remote direct memory access (RDMA).</span></span> 
- <span data-ttu-id="806cc-117">В зависимости от характера рабочей нагрузки вам могут потребоваться размеры виртуальных машин для больших объемов вычислений (H16r, H16mr или A9).</span><span class="sxs-lookup"><span data-stu-id="806cc-117">Depending on your workload, you might use compute-intensive VM sizes (H16r, H16mr, and A9).</span></span>

## <a name="when-to-use-this-architecture"></a><span data-ttu-id="806cc-118">Когда следует использовать эту архитектуру</span><span class="sxs-lookup"><span data-stu-id="806cc-118">When to use this architecture</span></span>

- <span data-ttu-id="806cc-119">Для ресурсоемких вычислительных задач, таких как моделирование и обработка больших объемов числовых данных.</span><span class="sxs-lookup"><span data-stu-id="806cc-119">Computationally intensive operations such as simulation and number crunching.</span></span>
- <span data-ttu-id="806cc-120">Для моделей с ресурсоемкими вычислениями, которые можно распределить между несколькими ЦП на нескольких компьютерах (от десятков до тысяч ядер).</span><span class="sxs-lookup"><span data-stu-id="806cc-120">Simulations that are computationally intensive and must be split across CPUs in multiple computers (10-1000s).</span></span>
- <span data-ttu-id="806cc-121">Для моделей, обработка которых невозможна в пределах оперативной памяти одного компьютера и которые необходимо распределить между несколькими компьютерами.</span><span class="sxs-lookup"><span data-stu-id="806cc-121">Simulations that require too much memory for one computer, and must be split across multiple computers.</span></span>
- <span data-ttu-id="806cc-122">Для длительных вычислений, выполнение которых на одном компьютере потребует слишком много времени.</span><span class="sxs-lookup"><span data-stu-id="806cc-122">Long-running computations that would take too long to complete on a single computer.</span></span>
- <span data-ttu-id="806cc-123">Для относительно небольших вычислений, которые нужно повторять сотни или тысячи раз, как, например, моделирование по методу Монте-Карло.</span><span class="sxs-lookup"><span data-stu-id="806cc-123">Smaller computations that must be run 100s or 1000s of times, such as Monte Carlo simulations.</span></span>

## <a name="benefits"></a><span data-ttu-id="806cc-124">Преимущества</span><span class="sxs-lookup"><span data-stu-id="806cc-124">Benefits</span></span>

- <span data-ttu-id="806cc-125">Высокая производительность при обработке [с усложненным параллелизмом][embarrassingly-parallel].</span><span class="sxs-lookup"><span data-stu-id="806cc-125">High performance with "[embarrassingly parallel][embarrassingly-parallel]" processing.</span></span>
- <span data-ttu-id="806cc-126">Возможность применить сразу сотни или тысячи ядер для быстрого решения больших проблем.</span><span class="sxs-lookup"><span data-stu-id="806cc-126">Can harness hundreds or thousands of computer cores to solve large problems faster.</span></span>
- <span data-ttu-id="806cc-127">Доступ к специализированному высокопроизводительному оборудованию с поддержкой высокоскоростной выделенной сети InfiniBand.</span><span class="sxs-lookup"><span data-stu-id="806cc-127">Access to specialized high-performance hardware, with dedicated high-speed InfiniBand networks.</span></span>
- <span data-ttu-id="806cc-128">Возможность подготовить виртуальные машины в необходимом для работы количестве и постепенно удалять их.</span><span class="sxs-lookup"><span data-stu-id="806cc-128">You can provision VMs as needed to do work, and then tear them down.</span></span> 

## <a name="challenges"></a><span data-ttu-id="806cc-129">Сложности</span><span class="sxs-lookup"><span data-stu-id="806cc-129">Challenges</span></span>

- <span data-ttu-id="806cc-130">Управление инфраструктурой виртуальных машин.</span><span class="sxs-lookup"><span data-stu-id="806cc-130">Managing the VM infrastructure.</span></span>
- <span data-ttu-id="806cc-131">Управление большими объемами обработки числовых данных.</span><span class="sxs-lookup"><span data-stu-id="806cc-131">Managing the volume of number crunching.</span></span> 
- <span data-ttu-id="806cc-132">Своевременная подготовка тысяч ядер.</span><span class="sxs-lookup"><span data-stu-id="806cc-132">Provisioning thousands of cores in a timely manner.</span></span>
- <span data-ttu-id="806cc-133">Для задач с высокой взаимозависимостью добавление ядер может привести к снижению эффективности.</span><span class="sxs-lookup"><span data-stu-id="806cc-133">For tightly coupled tasks, adding more cores can have diminishing returns.</span></span> <span data-ttu-id="806cc-134">Возможно, правильное количество ядер удастся определить лишь экспериментальным путем.</span><span class="sxs-lookup"><span data-stu-id="806cc-134">You may need to experiment to find the optimum number of cores.</span></span>

## <a name="big-compute-using-azure-batch"></a><span data-ttu-id="806cc-135">Большие вычисления с помощью пакетной службы Azure</span><span class="sxs-lookup"><span data-stu-id="806cc-135">Big compute using Azure Batch</span></span>

<span data-ttu-id="806cc-136">[Пакетная служба Azure][batch] — это управляемая служба для выполнения приложений высокопроизводительных крупномасштабных вычислений (HPC).</span><span class="sxs-lookup"><span data-stu-id="806cc-136">[Azure Batch][batch] is a managed service for running large-scale high-performance computing (HPC) applications.</span></span>

<span data-ttu-id="806cc-137">Сначала в пакетной службе Azure настраивается пул виртуальных машин, а затем в нее передаются файлы приложения и данных.</span><span class="sxs-lookup"><span data-stu-id="806cc-137">Using Azure Batch, you configure a VM pool, and upload the applications and data files.</span></span> <span data-ttu-id="806cc-138">Пакетная служба подготавливает виртуальные машины, распределяет между ними задачи, запускает задачи и отслеживает ход выполнения.</span><span class="sxs-lookup"><span data-stu-id="806cc-138">Then the Batch service provisions the VMs, assign tasks to the VMs, runs the tasks, and monitors the progress.</span></span> <span data-ttu-id="806cc-139">Пакетная служба может автоматически масштабировать виртуальные машины при изменении рабочей нагрузки.</span><span class="sxs-lookup"><span data-stu-id="806cc-139">Batch can automatically scale out the VMs in response to the workload.</span></span> <span data-ttu-id="806cc-140">Кроме того, пакетная служба поддерживает планирование заданий.</span><span class="sxs-lookup"><span data-stu-id="806cc-140">Batch also provides job scheduling.</span></span>

![](./images/big-compute-batch.png) 

## <a name="big-compute-running-on-virtual-machines"></a><span data-ttu-id="806cc-141">Выполнение больших вычислений на виртуальных машинах</span><span class="sxs-lookup"><span data-stu-id="806cc-141">Big compute running on Virtual Machines</span></span>

<span data-ttu-id="806cc-142">Для управления кластером виртуальных машин, а также планирования и мониторинга заданий HPC вы можете использовать [пакет Microsoft HPC][hpc-pack].</span><span class="sxs-lookup"><span data-stu-id="806cc-142">You can use [Microsoft HPC Pack][hpc-pack] to administer a cluster of VMs, and schedule and monitor HPC jobs.</span></span> <span data-ttu-id="806cc-143">Это позволит вам самостоятельно подготовить виртуальные машины и сетевую инфраструктуру, а также управлять ими.</span><span class="sxs-lookup"><span data-stu-id="806cc-143">With this approach, you must provision and manage the VMs and network infrastructure.</span></span> <span data-ttu-id="806cc-144">Рекомендуем этот подход, если у вас уже есть рабочие нагрузки HPC, но вы планируете полностью или частично переместить их в Azure.</span><span class="sxs-lookup"><span data-stu-id="806cc-144">Consider this approach if you have existing HPC workloads and want to move some or all it to Azure.</span></span> <span data-ttu-id="806cc-145">Вы можете переместить в Azure весь кластер HPC или сохранить локальный кластер HPC и использовать Azure для повышения производительности.</span><span class="sxs-lookup"><span data-stu-id="806cc-145">You can move the entire HPC cluster to Azure, or keep your HPC cluster on-premises but use Azure for burst capacity.</span></span> <span data-ttu-id="806cc-146">Дополнительные сведения см. в статье [Использование HPC, пакетной службы и больших вычислений][batch-hpc-solutions].</span><span class="sxs-lookup"><span data-stu-id="806cc-146">For more information, see [Batch and HPC solutions for large-scale computing workloads][batch-hpc-solutions].</span></span>

### <a name="hpc-pack-deployed-to-azure"></a><span data-ttu-id="806cc-147">Пакет HPC, развернутый в Azure</span><span class="sxs-lookup"><span data-stu-id="806cc-147">HPC Pack deployed to Azure</span></span>

<span data-ttu-id="806cc-148">В этом сценарии кластер HPC создается полностью на платформе Azure.</span><span class="sxs-lookup"><span data-stu-id="806cc-148">In this scenario, the HPC cluster is created entirely within Azure.</span></span>

![](./images/big-compute-iaas.png) 
 
<span data-ttu-id="806cc-149">В головном узле для кластера выполняются задачи управления и планирования заданий.</span><span class="sxs-lookup"><span data-stu-id="806cc-149">The head node provides management and job scheduling services to the cluster.</span></span> <span data-ttu-id="806cc-150">Для задач с высокой степенью взаимозависимости вы можете применить сеть RDMA, которая обеспечивает взаимодействие между виртуальными машинами с очень высокой пропускной способностью и низкими задержками.</span><span class="sxs-lookup"><span data-stu-id="806cc-150">For tightly coupled tasks, use an RDMA network that provides very high bandwidth, low latency communication between VMs.</span></span> <span data-ttu-id="806cc-151">Дополнительные сведения см. в статье [Развертывание кластера пакета HPC 2016 в Azure][deploy-hpc-azure].</span><span class="sxs-lookup"><span data-stu-id="806cc-151">For more information see [Deploy an HPC Pack 2016 cluster in Azure][deploy-hpc-azure].</span></span>

### <a name="burst-an-hpc-cluster-to-azure"></a><span data-ttu-id="806cc-152">Расширение кластера HPC в Azure</span><span class="sxs-lookup"><span data-stu-id="806cc-152">Burst an HPC cluster to Azure</span></span>

<span data-ttu-id="806cc-153">В этом сценарии пакет HPC выполняется в локальной среде организации к нему подключаются виртуальные машины Azure для повышения производительности.</span><span class="sxs-lookup"><span data-stu-id="806cc-153">In this scenario, an organization is running HPC Pack on-premises, and uses Azure VMs for burst capacity.</span></span> <span data-ttu-id="806cc-154">Головной узел кластера находится в локальной среде.</span><span class="sxs-lookup"><span data-stu-id="806cc-154">The cluster head node is on-premises.</span></span> <span data-ttu-id="806cc-155">Между локальной и виртуальной сетями Azure устанавливается подключение через ExpressRoute или VPN-шлюз.</span><span class="sxs-lookup"><span data-stu-id="806cc-155">ExpressRoute or VPN Gateway connects the on-premises network to the Azure VNet.</span></span>

![](./images/big-compute-hybrid.png) 


[batch]: /azure/batch/
[batch-hpc-solutions]: /azure/batch/batch-hpc-solutions
[deploy-hpc-azure]: /azure/virtual-machines/windows/hpcpack-2016-cluster
[embarrassingly-parallel]: https://en.wikipedia.org/wiki/Embarrassingly_parallel
[hpc-pack]: https://technet.microsoft.com/library/cc514029

 
