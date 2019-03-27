---
title: Трехмерный рендеринг видео
titleSuffix: Azure Example Scenarios
description: Выполнение собственных рабочих нагрузок HPC в Azure с использованием пакетной службы Azure.
author: adamboeglin
ms.date: 07/13/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: fasttrack
social_image_url: /azure/architecture/example-scenario/infrastructure/media/architecture-video-rendering.png
ms.openlocfilehash: 39b411eacaa1e0f400b59a099e9aa21b1159c921
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58245755"
---
# <a name="3d-video-rendering-on-azure"></a><span data-ttu-id="0f490-103">Отрисовка трехмерного видео на портале Azure</span><span class="sxs-lookup"><span data-stu-id="0f490-103">3D video rendering on Azure</span></span>

<span data-ttu-id="0f490-104">Отрисовка трехмерных видео — это длительный процесс, для его завершения требуется значительное количество времени ЦП.</span><span class="sxs-lookup"><span data-stu-id="0f490-104">3D video rendering is a time consuming process that requires a significant amount of CPU time to complete.</span></span> <span data-ttu-id="0f490-105">На одном компьютере процесс создания видеофайла из статических ресурсов может занять часы или даже дни в зависимости от длины и сложности видео.</span><span class="sxs-lookup"><span data-stu-id="0f490-105">On a single machine, the process of generating a video file from static assets can take hours or even days depending on the length and complexity of the video you are producing.</span></span> <span data-ttu-id="0f490-106">Многие компании будут покупать либо ресурсоемкие высокопроизводительные настольные компьютеры для выполнения этих задач, либо инвестировать в крупные фермы рендеринга, в которые они могут отправлять задания.</span><span class="sxs-lookup"><span data-stu-id="0f490-106">Many companies will purchase either expensive high end desktop computers to perform these tasks, or invest in large render farms that they can submit jobs to.</span></span> <span data-ttu-id="0f490-107">Однако при использовании пакетной службы Azure ресурсы доступны по требованию и они завершают работу, если в них нет надобности. И все это без каких-либо капиталовложений.</span><span class="sxs-lookup"><span data-stu-id="0f490-107">However, by taking advantage of Azure Batch, that power is available to you when you need it and shuts itself down when you don't, all without any capital investment.</span></span>

<span data-ttu-id="0f490-108">Пакетная служба предоставляет согласованные средства планирования заданий и управления ими для вычислительных узлов как с Windows Server, так и с Linux.</span><span class="sxs-lookup"><span data-stu-id="0f490-108">Batch gives you a consistent management experience and job scheduling, whether you select Windows Server or Linux compute nodes.</span></span> <span data-ttu-id="0f490-109">С помощью пакетной службы вы можете использовать имеющиеся приложения Windows или Linux, включая AutoDesk Maya и Blender, чтобы выполнить крупномасштабную отрисовку заданий в Azure.</span><span class="sxs-lookup"><span data-stu-id="0f490-109">With Batch, you can use your existing Windows or Linux applications, including AutoDesk Maya and Blender, to run large-scale render jobs in Azure.</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="0f490-110">Варианты соответствующего использования</span><span class="sxs-lookup"><span data-stu-id="0f490-110">Relevant use cases</span></span>

<span data-ttu-id="0f490-111">Другие варианты использования:</span><span class="sxs-lookup"><span data-stu-id="0f490-111">Other relevant use cases include:</span></span>

- <span data-ttu-id="0f490-112">3D-моделирование.</span><span class="sxs-lookup"><span data-stu-id="0f490-112">3D modeling</span></span>
- <span data-ttu-id="0f490-113">Отрисовка визуальных эффектов (VFX).</span><span class="sxs-lookup"><span data-stu-id="0f490-113">Visual FX (VFX) rendering</span></span>
- <span data-ttu-id="0f490-114">Перекодирование видео.</span><span class="sxs-lookup"><span data-stu-id="0f490-114">Video transcoding</span></span>
- <span data-ttu-id="0f490-115">Обработка изображений, цветокоррекция и изменение размера.</span><span class="sxs-lookup"><span data-stu-id="0f490-115">Image processing, color correction, and resizing</span></span>

## <a name="architecture"></a><span data-ttu-id="0f490-116">Архитектура</span><span class="sxs-lookup"><span data-stu-id="0f490-116">Architecture</span></span>

![Общие сведения об архитектуре компонентов, используемых в собственном облачном решении HPC с применением пакетной службы Azure][architecture]

<span data-ttu-id="0f490-118">Этот сценарий показывает рабочий процесс, в котором используется пакетная служба Azure.</span><span class="sxs-lookup"><span data-stu-id="0f490-118">This scenario shows a workflow that uses Azure Batch.</span></span> <span data-ttu-id="0f490-119">Путь потока данных выглядит следующим образом:</span><span class="sxs-lookup"><span data-stu-id="0f490-119">The data flows as follows:</span></span>

1. <span data-ttu-id="0f490-120">Передача входных файлов и приложений для обработки этих файлов в учетную запись службы хранилища Azure.</span><span class="sxs-lookup"><span data-stu-id="0f490-120">Upload input files and the applications to process those files to your Azure Storage account.</span></span>
2. <span data-ttu-id="0f490-121">Создание пула вычислительных узлов пакетной службы в своей учетной записи пакетной службы, задание для выполнения рабочей нагрузки в пуле и задачи в задании.</span><span class="sxs-lookup"><span data-stu-id="0f490-121">Create a Batch pool of compute nodes in your Batch account, a job to run the workload on the pool, and tasks in the job.</span></span>
3. <span data-ttu-id="0f490-122">Скачивание входных файлов и приложений для пакетной службы.</span><span class="sxs-lookup"><span data-stu-id="0f490-122">Download input files and the applications to Batch.</span></span>
4. <span data-ttu-id="0f490-123">Мониторинг выполнения задач.</span><span class="sxs-lookup"><span data-stu-id="0f490-123">Monitor task execution.</span></span>
5. <span data-ttu-id="0f490-124">Отправка выходных данных задачи.</span><span class="sxs-lookup"><span data-stu-id="0f490-124">Upload task output.</span></span>
6. <span data-ttu-id="0f490-125">Скачивание выходных файлов.</span><span class="sxs-lookup"><span data-stu-id="0f490-125">Download output files.</span></span>

<span data-ttu-id="0f490-126">Чтобы упростить этот процесс, можно также использовать [подключаемые модули пакетной службы для Maya и 3DS Max][batch-plugins]</span><span class="sxs-lookup"><span data-stu-id="0f490-126">To simplify this process, you could also use the [Batch Plugins for Maya and 3ds Max][batch-plugins]</span></span>

### <a name="components"></a><span data-ttu-id="0f490-127">Компоненты</span><span class="sxs-lookup"><span data-stu-id="0f490-127">Components</span></span>

<span data-ttu-id="0f490-128">Пакетная служба Azure построена на основе следующих технологий Azure.</span><span class="sxs-lookup"><span data-stu-id="0f490-128">Azure Batch builds upon the following Azure technologies:</span></span>

- <span data-ttu-id="0f490-129">[Виртуальные сети](/azure/virtual-network/virtual-networks-overview) используются как для головного узла, так и для вычислительных ресурсов.</span><span class="sxs-lookup"><span data-stu-id="0f490-129">[Virtual Networks](/azure/virtual-network/virtual-networks-overview) are used for both the head node and the compute resources.</span></span>
- <span data-ttu-id="0f490-130">[Учетные записи хранения Azure](/azure/storage/common/storage-introduction) используются для хранения данных и синхронизации.</span><span class="sxs-lookup"><span data-stu-id="0f490-130">[Azure Storage accounts](/azure/storage/common/storage-introduction) are used for synchronization and data retention.</span></span>
- <span data-ttu-id="0f490-131">[Масштабируемые наборы виртуальных машин][vmss] используются CycleCloud для вычислительных ресурсов.</span><span class="sxs-lookup"><span data-stu-id="0f490-131">[Virtual Machine Scale Sets][vmss] are used by CycleCloud for compute resources.</span></span>

## <a name="considerations"></a><span data-ttu-id="0f490-132">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="0f490-132">Considerations</span></span>

### <a name="machine-sizes-available-for-azure-batch"></a><span data-ttu-id="0f490-133">Доступный размер компьютеров для пакетной службы Azure</span><span class="sxs-lookup"><span data-stu-id="0f490-133">Machine Sizes available for Azure Batch</span></span>

<span data-ttu-id="0f490-134">Хотя большинство клиентов, которым требуется рендеринг, выбирают ресурсы с ЦП большой мощности, для других рабочих нагрузок, использующих масштабируемые наборы виртуальных машин, им подойдут виртуальные машины с иными характеристиками. Выбор зависит от ряда факторов:</span><span class="sxs-lookup"><span data-stu-id="0f490-134">While most rendering customers will choose resources with high CPU power, other workloads using virtual machine scale sets may choose VMs differently and will depend on a number of factors:</span></span>

- <span data-ttu-id="0f490-135">Потребляет ли выполняемое приложение много памяти?</span><span class="sxs-lookup"><span data-stu-id="0f490-135">Is the application being run memory bound?</span></span>
- <span data-ttu-id="0f490-136">Необходим ли GPU для работы приложения?</span><span class="sxs-lookup"><span data-stu-id="0f490-136">Does the application need to use GPUs?</span></span>
- <span data-ttu-id="0f490-137">Требуется ли высокая степень параллелизма для выполнения заданий или организация подключения InfiniBand для тесно связанных заданий?</span><span class="sxs-lookup"><span data-stu-id="0f490-137">Are the job types embarrassingly parallel or require infiniband connectivity for tightly coupled jobs?</span></span>
- <span data-ttu-id="0f490-138">Требуется ли быстрый ввод-вывод для доступа к службе хранилища на вычислительных узлах?</span><span class="sxs-lookup"><span data-stu-id="0f490-138">Require fast I/O to access storage on the compute Nodes.</span></span>

<span data-ttu-id="0f490-139">В Azure доступны виртуальные машины разных размеров, способные удовлетворить любое из перечисленных выше требований, предъявляемых приложениями. Некоторые из этих виртуальных машин предназначены для высокопроизводительных вычислений, но даже виртуальные машины самых маленьких размеров могут быть эффективными для реализации сетки:</span><span class="sxs-lookup"><span data-stu-id="0f490-139">Azure has a wide range of VM sizes that can address each and every one of the above application requirements, some are specific to HPC, but even the smallest sizes can be utilized to provide an effective grid implementation:</span></span>

- <span data-ttu-id="0f490-140">[Размеры виртуальных машин для высокопроизводительных вычислений][compute-hpc]. Из-за высоких требований к мощности ЦП при рендеринге Майкрософт рекомендует для этих целей использовать виртуальные машины Azure серии H.</span><span class="sxs-lookup"><span data-stu-id="0f490-140">[HPC VM sizes][compute-hpc] Due to the CPU bound nature of rendering, Microsoft typically suggests the Azure H-Series VMs.</span></span> <span data-ttu-id="0f490-141">Они предназначены специально для высокопроизводительных вычислений и оснащены виртуальными ЦП с 8 или 16 ядрами на основе ЦП Intel Haswell E5, оперативной памятью DDR4 и временным хранилищем на SSD-дисках.</span><span class="sxs-lookup"><span data-stu-id="0f490-141">This type of VM is built specifically for high end computational needs, they have 8 and 16 core vCPU sizes available, and features DDR4 memory, SSD temporary storage, and Haswell E5 Intel technology.</span></span>
- <span data-ttu-id="0f490-142">[Размеры виртуальных машин с GPU][compute-gpu]. Размеры виртуальных машин с оптимизацией GPU — это специализированные виртуальные машины с одним или несколькими GPU NVIDIA.</span><span class="sxs-lookup"><span data-stu-id="0f490-142">[GPU VM sizes][compute-gpu] GPU optimized VM sizes are specialized virtual machines available with single or multiple NVIDIA GPUs.</span></span> <span data-ttu-id="0f490-143">Эти размеры предназначены для рабочих нагрузок с большим объемом вычислений, графической обработки и визуализаций.</span><span class="sxs-lookup"><span data-stu-id="0f490-143">These sizes are designed for compute-intensive, graphics-intensive, and visualization workloads.</span></span>
- <span data-ttu-id="0f490-144">Размеры NC, NCv2, NCv3 и ND оптимизированы для приложений и алгоритмов, требующих ресурсоемких вычислений и высокой пропускной способности сети, включая приложения и моделирование на базе CUDA и OpenCL, а также ИИ и глубокое обучение.</span><span class="sxs-lookup"><span data-stu-id="0f490-144">NC, NCv2, NCv3, and ND sizes are optimized for compute-intensive and network-intensive applications and algorithms, including CUDA and OpenCL-based applications and simulations, AI, and Deep Learning.</span></span> <span data-ttu-id="0f490-145">Размеры NV оптимизированы и предназначены для удаленной визуализации, потоковой передачи, игр, кодирования и сценариев VDI, которые используют такие платформы, как OpenGL и DirectX.</span><span class="sxs-lookup"><span data-stu-id="0f490-145">NV sizes are optimized and designed for remote visualization, streaming, gaming, encoding, and VDI scenarios utilizing frameworks such as OpenGL and DirectX.</span></span>
- <span data-ttu-id="0f490-146">[Размеры виртуальных машин, оптимизированных для операций в памяти][compute-memory]. Такие виртуальные машины имеют высокий показатель объема памяти в расчете на одно ядро ЦП и подходят для операций, требующих большого объема памяти.</span><span class="sxs-lookup"><span data-stu-id="0f490-146">[Memory optimized VM sizes][compute-memory] When more memory is required, the memory optimized VM sizes offer a higher memory-to-CPU ratio.</span></span>
- <span data-ttu-id="0f490-147">[Размеры виртуальных машин общего назначения][compute-general]. Такие виртуальные машины имеют сбалансированный показатель объема памяти на ядро ЦП.</span><span class="sxs-lookup"><span data-stu-id="0f490-147">[General purposes VM sizes][compute-general] General-purpose VM sizes are also available and provide balanced CPU-to-memory ratio.</span></span>

### <a name="alternatives"></a><span data-ttu-id="0f490-148">Альтернативные варианты</span><span class="sxs-lookup"><span data-stu-id="0f490-148">Alternatives</span></span>

<span data-ttu-id="0f490-149">Если вам требуется больше контроля над средой отрисовки в Azure или нужна гибридная реализация, тогда вычисления CycleCloud могут помочь оркестрировать сетку IaaS в облаке.</span><span class="sxs-lookup"><span data-stu-id="0f490-149">If you require more control over your rendering environment in Azure or need a hybrid implementation, then CycleCloud computing can help orchestrate an IaaS grid in the cloud.</span></span> <span data-ttu-id="0f490-150">Используя те же базовые технологии Azure, что и пакетная служба Azure, они позволяют эффективно создавать и поддерживать сетку IaaS.</span><span class="sxs-lookup"><span data-stu-id="0f490-150">Using the same underlying Azure technologies as Azure Batch, it makes building and maintaining an IaaS grid an efficient process.</span></span> <span data-ttu-id="0f490-151">Для получения дополнительных сведений, в том числе о принципах проектирования, перейдите по этой ссылке:</span><span class="sxs-lookup"><span data-stu-id="0f490-151">To find out more and learn about the design principles use the following link:</span></span>

<span data-ttu-id="0f490-152">Полный обзор всех доступных в Azure решений для высокопроизводительных вычислений см. в статье [Использование HPC, пакетной службы и больших вычислений][hpc-alt-solutions].</span><span class="sxs-lookup"><span data-stu-id="0f490-152">For a complete overview of all the HPC solutions that are available to you in Azure, see the article [HPC, Batch, and Big Compute solutions using Azure VMs][hpc-alt-solutions]</span></span>

### <a name="availability"></a><span data-ttu-id="0f490-153">Доступность</span><span class="sxs-lookup"><span data-stu-id="0f490-153">Availability</span></span>

<span data-ttu-id="0f490-154">Мониторинг компонентов пакетной службы Azure доступен в целом ряде служб, средств и API.</span><span class="sxs-lookup"><span data-stu-id="0f490-154">Monitoring of the Azure Batch components is available through a range of services, tools, and APIs.</span></span> <span data-ttu-id="0f490-155">Дополнительные сведения см. в статье [Мониторинг решений пакетной службы][batch-monitor].</span><span class="sxs-lookup"><span data-stu-id="0f490-155">Monitoring is discussed further in the [Monitor Batch solutions][batch-monitor] article.</span></span>

### <a name="scalability"></a><span data-ttu-id="0f490-156">Масштабируемость</span><span class="sxs-lookup"><span data-stu-id="0f490-156">Scalability</span></span>

<span data-ttu-id="0f490-157">Пулы учетной записи пакетной службы Azure можно масштабировать автоматически по формуле на основе метрик службы или вручную.</span><span class="sxs-lookup"><span data-stu-id="0f490-157">Pools within an Azure Batch account can either scale through manual intervention or, by using a formula based on Azure Batch metrics, be scaled automatically.</span></span> <span data-ttu-id="0f490-158">Дополнительные сведения см. в статье [Создание формулы автоматического масштабирования для масштабирования вычислительных узлов в пуле пакетной службы][batch-scaling].</span><span class="sxs-lookup"><span data-stu-id="0f490-158">For more information on scalability, see the article [Create an automatic scaling formula for scaling nodes in a Batch pool][batch-scaling].</span></span>

### <a name="security"></a><span data-ttu-id="0f490-159">Безопасность</span><span class="sxs-lookup"><span data-stu-id="0f490-159">Security</span></span>

<span data-ttu-id="0f490-160">Общие рекомендации по разработке безопасных решений см. в разделе [Azure Security Documentation][security].</span><span class="sxs-lookup"><span data-stu-id="0f490-160">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="0f490-161">Устойчивость</span><span class="sxs-lookup"><span data-stu-id="0f490-161">Resiliency</span></span>

<span data-ttu-id="0f490-162">Пакетная служба Azure сейчас не поддерживает отработку отказа, поэтому рекомендуется выполнить следующие действия для обеспечения доступности в случае сбоя:</span><span class="sxs-lookup"><span data-stu-id="0f490-162">While there is currently no failover capability in Azure Batch, we recommend using the following steps to ensure availability if there is an unplanned outage:</span></span>

- <span data-ttu-id="0f490-163">Создайте альтернативную учетную запись пакетной службы Azure в другом расположении Azure, используя другую учетную запись хранения.</span><span class="sxs-lookup"><span data-stu-id="0f490-163">Create an Azure Batch account in an alternate Azure location with an alternate Storage Account</span></span>
- <span data-ttu-id="0f490-164">Создайте аналогичные узлы пулов с такими же именами, но без выделенных узлов.</span><span class="sxs-lookup"><span data-stu-id="0f490-164">Create the same node pools with the same name, with zero nodes allocated</span></span>
- <span data-ttu-id="0f490-165">Убедитесь, что приложения создаются и обновляются в альтернативной учетной записи хранения.</span><span class="sxs-lookup"><span data-stu-id="0f490-165">Ensure Applications are created and updated to the alternate Storage Account</span></span>
- <span data-ttu-id="0f490-166">Отправьте входные файлы и задания в альтернативную учетную запись пакетной службы Azure.</span><span class="sxs-lookup"><span data-stu-id="0f490-166">Upload input files and submit jobs to the alternate Azure Batch account</span></span>

## <a name="deploy-the-scenario"></a><span data-ttu-id="0f490-167">Развертывание сценария</span><span class="sxs-lookup"><span data-stu-id="0f490-167">Deploy the scenario</span></span>

### <a name="create-an-azure-batch-account-and-pools-manually"></a><span data-ttu-id="0f490-168">Создание учетной записи и пулов пакетной службы Azure вручную</span><span class="sxs-lookup"><span data-stu-id="0f490-168">Create an Azure Batch account and pools manually</span></span>

<span data-ttu-id="0f490-169">Этот сценарий демонстрирует работу пакетной службы Azure и возможность разработки пользовательского SaaS-решения в лабораториях пакетной службы Azure:</span><span class="sxs-lookup"><span data-stu-id="0f490-169">This scenario demonstrates how Azure Batch works while showcasing Azure Batch Labs as an example SaaS solution that can be developed for your own customers:</span></span>

<span data-ttu-id="0f490-170">[Демонстрация с пакетной службой Azure][batch-labs-masterclass]</span><span class="sxs-lookup"><span data-stu-id="0f490-170">[Azure Batch Masterclass][batch-labs-masterclass]</span></span>

### <a name="deploy-the-components"></a><span data-ttu-id="0f490-171">Развертывание компонентов</span><span class="sxs-lookup"><span data-stu-id="0f490-171">Deploy the components</span></span>

<span data-ttu-id="0f490-172">Шаблон развертывает:</span><span class="sxs-lookup"><span data-stu-id="0f490-172">The template will deploy:</span></span>

- <span data-ttu-id="0f490-173">Новую учетную запись пакетной службы Azure.</span><span class="sxs-lookup"><span data-stu-id="0f490-173">A new Azure Batch account</span></span>
- <span data-ttu-id="0f490-174">Учетная запись хранения.</span><span class="sxs-lookup"><span data-stu-id="0f490-174">A storage account</span></span>
- <span data-ttu-id="0f490-175">Пул узла, связанный с учетной записью пакетной службы.</span><span class="sxs-lookup"><span data-stu-id="0f490-175">A node pool associated with the batch account</span></span>
- <span data-ttu-id="0f490-176">Пул узла, который будет настроен для использования виртуальных машин A2 v2 с образами Canonical Ubuntu.</span><span class="sxs-lookup"><span data-stu-id="0f490-176">The node pool will be configured to use A2 v2 VMs with Canonical Ubuntu images</span></span>
- <span data-ttu-id="0f490-177">Пул узла изначально не будет содержать виртуальных машин. Вам нужно будет выполнить масштабирование вручную для их добавления.</span><span class="sxs-lookup"><span data-stu-id="0f490-177">The node pool will contain zero VMs initially and will require you to manually scale to add VMs</span></span>

<!-- markdownlint-disable MD033 -->

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fhpc%2Fbatchcreatewithpools.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>
<!-- markdownlint-enable MD033 -->

<span data-ttu-id="0f490-178">[См. дополнительные сведения о шаблонах Resource Manager][azure-arm-templates]</span><span class="sxs-lookup"><span data-stu-id="0f490-178">[Learn more about Resource Manager templates][azure-arm-templates]</span></span>

## <a name="pricing"></a><span data-ttu-id="0f490-179">Цены</span><span class="sxs-lookup"><span data-stu-id="0f490-179">Pricing</span></span>

<span data-ttu-id="0f490-180">Стоимость использования пакетной службы Azure будет зависеть от размеров виртуальных машин в пулах и времени их фактической работы. Учетная запись пакетной службы Azure создается бесплатно.</span><span class="sxs-lookup"><span data-stu-id="0f490-180">The cost of using Azure Batch will depend on the VM sizes that are used for the pools and how long these VMs are allocated and running, there is no cost associated with an Azure Batch account creation.</span></span> <span data-ttu-id="0f490-181">На стоимость также влияет объем исходящего трафика данных, в т. ч. хранилища.</span><span class="sxs-lookup"><span data-stu-id="0f490-181">Storage and data egress should be taken into account as these will apply additional costs.</span></span>

<span data-ttu-id="0f490-182">Ниже приведены примеры взимаемых затрат для задания, выполняющегося за 8 часов с разным количеством серверов:</span><span class="sxs-lookup"><span data-stu-id="0f490-182">The following are examples of costs that could be incurred for a job that completes in 8 hours using a different number of servers:</span></span>

- <span data-ttu-id="0f490-183">100 виртуальных машин с высокопроизводительным ЦП. [Примерная стоимость][hpc-est-high]</span><span class="sxs-lookup"><span data-stu-id="0f490-183">100 High-Performance CPU VMs: [Cost Estimate][hpc-est-high]</span></span>

  <span data-ttu-id="0f490-184">100 x H16m (16 ядер, 225 ГБ ОЗУ, хранилище класса Premium на 512 ГБ), хранилище BLOB-объектов на 2 ТБ, 1 ТБ исходящего трафика</span><span class="sxs-lookup"><span data-stu-id="0f490-184">100 x H16m (16 cores, 225 GB RAM, Premium Storage 512 GB), 2 TB Blob Storage, 1-TB egress</span></span>

- <span data-ttu-id="0f490-185">50 виртуальных машин с высокопроизводительным ЦП. [Примерная стоимость][hpc-est-med]</span><span class="sxs-lookup"><span data-stu-id="0f490-185">50 High-Performance CPU VMs: [Cost Estimate][hpc-est-med]</span></span>

  <span data-ttu-id="0f490-186">50 x H16m (16 ядер, 225 ГБ ОЗУ, хранилище класса Premium на 512 ГБ), хранилище BLOB-объектов на 2 ТБ, 1 ТБ исходящего трафика</span><span class="sxs-lookup"><span data-stu-id="0f490-186">50 x H16m (16 cores, 225 GB RAM, Premium Storage 512 GB), 2 TB Blob Storage, 1-TB egress</span></span>

- <span data-ttu-id="0f490-187">10 виртуальных машин с высокопроизводительным ЦП. [Примерная стоимость][hpc-est-low]</span><span class="sxs-lookup"><span data-stu-id="0f490-187">10 High-Performance CPU VMs: [Cost Estimate][hpc-est-low]</span></span>

  <span data-ttu-id="0f490-188">10 x H16m (16 ядер, 225 ГБ ОЗУ, хранилище класса Premium на 512 ГБ), хранилище BLOB-объектов на 2 ТБ, 1 ТБ исходящего трафика</span><span class="sxs-lookup"><span data-stu-id="0f490-188">10 x H16m (16 cores, 225 GB RAM, Premium Storage 512 GB), 2 TB Blob Storage, 1-TB egress</span></span>

### <a name="pricing-for-low-priority-vms"></a><span data-ttu-id="0f490-189">Цены на низкоприоритетные виртуальные машины</span><span class="sxs-lookup"><span data-stu-id="0f490-189">Pricing for low-priority VMs</span></span>

<span data-ttu-id="0f490-190">В пулах узлов пакетной службы Azure также можно использовать виртуальные машины с низким приоритетом, что может существенно снизить расходы.</span><span class="sxs-lookup"><span data-stu-id="0f490-190">Azure Batch also supports the use of low-priority VMs in the node pools, which can potentially provide a substantial cost saving.</span></span> <span data-ttu-id="0f490-191">Дополнительные сведения, в том числе сравнение стоимости использования стандартных виртуальных машин и виртуальных машин с низким приоритетом см. на странице [расценок на пакетную службу Azure][batch-pricing].</span><span class="sxs-lookup"><span data-stu-id="0f490-191">For more information, including a price comparison between standard VMs and low-priority VMs, see [Azure Batch Pricing][batch-pricing].</span></span>

> [!NOTE]
> <span data-ttu-id="0f490-192">Низкоприоритетные виртуальные машины подходят только для определенных приложений и рабочих нагрузок.</span><span class="sxs-lookup"><span data-stu-id="0f490-192">Low-priority VMs are only suitable for certain applications and workloads.</span></span>

## <a name="related-resources"></a><span data-ttu-id="0f490-193">Связанные ресурсы</span><span class="sxs-lookup"><span data-stu-id="0f490-193">Related resources</span></span>

<span data-ttu-id="0f490-194">[Пакетная служба][batch-overview]</span><span class="sxs-lookup"><span data-stu-id="0f490-194">[Azure Batch Overview][batch-overview]</span></span>

<span data-ttu-id="0f490-195">[Документация по пакетной службе][batch-doc]</span><span class="sxs-lookup"><span data-stu-id="0f490-195">[Azure Batch Documentation][batch-doc]</span></span>

<span data-ttu-id="0f490-196">[Использование контейнеров в пакетной службе Azure][batch-containers]</span><span class="sxs-lookup"><span data-stu-id="0f490-196">[Using containers on Azure Batch][batch-containers]</span></span>

<!-- links -->
[architecture]: ./media/architecture-video-rendering.png
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[security]: /azure/security/
[resiliency]: /azure/architecture/resiliency/
[scalability]: /azure/architecture/checklist/scalability
[vmss]: /azure/virtual-machine-scale-sets/overview
[storage]: https://azure.microsoft.com/services/storage/
[batch]: https://azure.microsoft.com/services/batch/
[batch-arch]: https://azure.microsoft.com/solutions/architecture/big-compute-with-azure-batch/
[compute-hpc]: /azure/virtual-machines/windows/sizes-hpc
[compute-gpu]: /azure/virtual-machines/windows/sizes-gpu
[compute-compute]: /azure/virtual-machines/windows/sizes-compute
[compute-memory]: /azure/virtual-machines/windows/sizes-memory
[compute-general]: /azure/virtual-machines/windows/sizes-general
[compute-storage]: /azure/virtual-machines/windows/sizes-storage
[compute-acu]: /azure/virtual-machines/windows/acu
[compute=benchmark]: /azure/virtual-machines/windows/compute-benchmark-scores
[hpc-est-high]: https://azure.com/e/9ac25baf44ef49c3a6b156935ee9544c
[hpc-est-med]: https://azure.com/e/0286f1d6f6784310af4dcda5aec8c893
[hpc-est-low]: https://azure.com/e/e39afab4e71949f9bbabed99b428ba4a
[batch-labs-masterclass]: https://github.com/azurebigcompute/BigComputeLabs/tree/master/Azure%20Batch%20Masterclass%20Labs
[batch-scaling]: /azure/batch/batch-automatic-scaling
[hpc-alt-solutions]: /azure/virtual-machines/linux/high-performance-computing?toc=%2fazure%2fbatch%2ftoc.json
[batch-monitor]: /azure/batch/monitoring-overview
[batch-pricing]: https://azure.microsoft.com/pricing/details/batch/
[batch-doc]: /azure/batch/
[batch-overview]: https://azure.microsoft.com/services/batch/
[batch-containers]: https://github.com/Azure/batch-shipyard
[azure-arm-templates]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[batch-plugins]: /azure/batch/batch-rendering-service#options-for-submitting-a-render-job