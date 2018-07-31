---
title: Отрисовка трехмерного видео на портале Azure
description: Выполнение собственных рабочих нагрузок HPC в Azure с использованием пакетной службы Azure
author: adamboeglin
ms.date: 07/13/2018
ms.openlocfilehash: e629e2ba0b9490e534057fee33f7bededa9656af
ms.sourcegitcommit: c704d5d51c8f9bbab26465941ddcf267040a8459
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/24/2018
ms.locfileid: "39229190"
---
# <a name="3d-video-rendering-on-azure"></a><span data-ttu-id="a5ed9-103">Отрисовка трехмерного видео на портале Azure</span><span class="sxs-lookup"><span data-stu-id="a5ed9-103">3D video rendering on Azure</span></span>

<span data-ttu-id="a5ed9-104">Трехмерная отрисовка — это длительный процесс, для его завершения требуется значительное количество времени ЦП.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-104">3D rendering is a time consuming process that requires a significant amount of CPU time to complete.</span></span>  <span data-ttu-id="a5ed9-105">На одном компьютере процесс создания видеофайла из статических ресурсов может занять часы или даже дни в зависимости от длины и сложности видео.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-105">On a single machine, the process of generating a video file from static assets can take hours or even days depending on the length and complexity of the video you are producing.</span></span>  <span data-ttu-id="a5ed9-106">Многие компании будут покупать либо ресурсоемкие высокопроизводительные настольные компьютеры для выполнения этих задач, либо инвестировать в крупные фермы рендеринга, в которые они могут отправлять задания.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-106">Many companies will purchase either expensive high end desktop computers to perform these tasks, or invest in large render farms that they can submit jobs to.</span></span>  <span data-ttu-id="a5ed9-107">Однако при использовании пакетной службы Azure ресурсы доступны по требованию и они завершают работу, если в них нет надобности. И все это без каких-либо капиталовложений.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-107">However, by taking advantage of Azure Batch, that power is available to you when you need it and shuts itself down when you don't, all without any capital investment.</span></span>

<span data-ttu-id="a5ed9-108">Пакетная служба предоставляет согласованные средства планирования заданий и управления ими для вычислительных узлов как с Windows Server, так и с Linux.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-108">Batch gives you a consistent management experience and job scheduling, whether you select Windows Server or Linux compute nodes.</span></span> <span data-ttu-id="a5ed9-109">С помощью пакетной службы вы можете использовать имеющиеся приложения Windows или Linux, включая AutoDesk Maya и Blender, чтобы выполнить крупномасштабную отрисовку заданий в Azure.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-109">With Batch, you can use your existing Windows or Linux applications, including AutoDesk Maya and Blender, to run large-scale render jobs in Azure.</span></span>

## <a name="related-use-cases"></a><span data-ttu-id="a5ed9-110">Связанные варианты использования</span><span class="sxs-lookup"><span data-stu-id="a5ed9-110">Related use cases</span></span>

<span data-ttu-id="a5ed9-111">Рассмотрите этот сценарий для похожих вариантов использования:</span><span class="sxs-lookup"><span data-stu-id="a5ed9-111">Consider this scenario for these similar use cases:</span></span>

* <span data-ttu-id="a5ed9-112">3D-моделирование.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-112">3D Modeling</span></span>
* <span data-ttu-id="a5ed9-113">Отрисовка Visual FX (VFX).</span><span class="sxs-lookup"><span data-stu-id="a5ed9-113">Visual FX (VFX) Rendering</span></span>
* <span data-ttu-id="a5ed9-114">Перекодирование видео.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-114">Video transcoding</span></span>
* <span data-ttu-id="a5ed9-115">Обработка изображений, цветокоррекция и изменение размера.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-115">Image processing, color correction, & resizing</span></span>

## <a name="architecture"></a><span data-ttu-id="a5ed9-116">Архитектура</span><span class="sxs-lookup"><span data-stu-id="a5ed9-116">Architecture</span></span>

![Общие сведения об архитектуре компонентов, используемых в собственном облачном решении HPC с применением пакетной службы Azure][architecture]

<span data-ttu-id="a5ed9-118">В этом примере сценария описывается рабочий процесс при использовании пакетной службы Azure. Поток данных выглядит следующим образом:</span><span class="sxs-lookup"><span data-stu-id="a5ed9-118">This sample scenario covers the workflow when using Azure Batch, the data flows as follows:</span></span>

1. <span data-ttu-id="a5ed9-119">Передача входных файлов и приложений для обработки этих файлов в учетную запись службы хранилища Azure.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-119">Upload input files and the applications to process those files to your Azure Storage account</span></span>
2. <span data-ttu-id="a5ed9-120">Создание пула вычислительных узлов пакетной службы в своей учетной записи пакетной службы, задание для выполнения рабочей нагрузки в пуле и задачи в задании.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-120">Create a Batch pool of compute nodes in your Batch account, a job to run the workload on the pool, and tasks in the job.</span></span>
3. <span data-ttu-id="a5ed9-121">Скачивание входных файлов и приложений для пакетной службы.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-121">Download input files and the applications to Batch</span></span>
4. <span data-ttu-id="a5ed9-122">Мониторинг выполнения задач.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-122">Monitor task execution</span></span>
5. <span data-ttu-id="a5ed9-123">Отправка выходных данных задачи.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-123">Upload task output</span></span>
6. <span data-ttu-id="a5ed9-124">Скачивание выходных файлов.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-124">Download output files</span></span>

<span data-ttu-id="a5ed9-125">Чтобы упростить этот процесс, можно также использовать [подключаемые модули пакетной службы для Maya и 3DS Max][batch-plugins]</span><span class="sxs-lookup"><span data-stu-id="a5ed9-125">To simplify this process, you could also use the [Batch Plugins for Maya & 3ds Max][batch-plugins]</span></span>

### <a name="components"></a><span data-ttu-id="a5ed9-126">Компоненты</span><span class="sxs-lookup"><span data-stu-id="a5ed9-126">Components</span></span>

<span data-ttu-id="a5ed9-127">Пакетная служба Azure построена на основе следующих технологий Azure.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-127">Azure Batch builds upon the following Azure technologies:</span></span>

* <span data-ttu-id="a5ed9-128">[Группа ресурсов][resource-groups] — это логический контейнер для ресурсов Azure.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-128">[Resource Groups][resource-groups] is a logical container for Azure resources.</span></span>
* <span data-ttu-id="a5ed9-129">[Виртуальные сети][vnet] используются для головного узла и вычислительных ресурсов.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-129">[Virtual Networks][vnet] are used to for both the Head Node and Compute resources</span></span>
* <span data-ttu-id="a5ed9-130">Учетные записи [хранения][storage] используются для хранения данных и синхронизации.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-130">[Storage][storage] accounts are used for the synchronisation and data retention</span></span>
* <span data-ttu-id="a5ed9-131">[Масштабируемые наборы виртуальных машин][vmss] используются CycleCloud для вычислительных ресурсов.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-131">[Virtual Machine Scale Sets][vmss] are utilised by CycleCloud for compute resources</span></span>

## <a name="considerations"></a><span data-ttu-id="a5ed9-132">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="a5ed9-132">Considerations</span></span>

### <a name="machine-sizes-available-for-azure-batch"></a><span data-ttu-id="a5ed9-133">Доступный размер компьютеров для пакетной службы Azure</span><span class="sxs-lookup"><span data-stu-id="a5ed9-133">Machine Sizes available for Azure Batch</span></span>
<span data-ttu-id="a5ed9-134">Хотя большинство клиентов, которые заказывают услуги рендеринга, выбирают ресурсы с высокой мощностью ЦП, для других рабочих нагрузок, использующих Масштабируемые наборы виртуальных машин, можно выбрать разные виртуальные машины. Их выбор зависит от ряда факторов:</span><span class="sxs-lookup"><span data-stu-id="a5ed9-134">While most rendering customers whill choose resources with high CPU power, other workloads using VM Scale Sets may choose VM's differently and will depend on a number of factors:</span></span>
  - <span data-ttu-id="a5ed9-135">Связано ли приложение с выполняющейся памятью?</span><span class="sxs-lookup"><span data-stu-id="a5ed9-135">Is the Application being run memory bound?</span></span>
  - <span data-ttu-id="a5ed9-136">Требуется ли использование графического процессора в приложении?</span><span class="sxs-lookup"><span data-stu-id="a5ed9-136">Does the Application need to use GPU's?</span></span> 
  - <span data-ttu-id="a5ed9-137">Требуют ли типы заданий высокого уровня параллелизма или подключения Infiniband для тесно связанных заданий?</span><span class="sxs-lookup"><span data-stu-id="a5ed9-137">Are the job types embarassingly parallel or require Infiniband connectivity for tightly coupled jobs?</span></span>
  - <span data-ttu-id="a5ed9-138">Требуется ли быстрый ввод-вывод в службе хранилища на вычислительных узлах?</span><span class="sxs-lookup"><span data-stu-id="a5ed9-138">Require fast I/O to Storage on the Compute Nodes</span></span>

<span data-ttu-id="a5ed9-139">Azure имеет широкий диапазон размеров виртуальных машин, которые могут удовлетворить каждое из перечисленных выше требований к приложениям. Некоторые из них относятся к HPC, но даже виртуальные машины самых маленьких размеров могут использоваться для обеспечения эффективной реализации сетки:</span><span class="sxs-lookup"><span data-stu-id="a5ed9-139">Azure has a wide range of VM sizes that can address each and every one of the above application requirements, some are specific to HPC, but even the smallest sizes can be utilised to provide an effective grid implementation:</span></span>

  - <span data-ttu-id="a5ed9-140">[Размеры виртуальных машин с высокопроизводительными вычислительными системами (HPC)][compute-hpc]. Из-за особенностей рендеринга, связанных с центральным процессором, корпорация Майкрософт обычно рекомендует использовать виртуальные машины Azure серии H.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-140">[HPC VM sizes][compute-hpc] Due to the CPU bound nature of rendering, Microsoft typically suggests the Azure H-Series VM's.</span></span>  <span data-ttu-id="a5ed9-141">Они специально созданы для высокопроизводительных вычислений, предоставляют виртуальные ЦП с 8 или 16 ядрами, а также имеют оперативную память DDR4, временное хранилище SSD и технологию Haswell E5 Intel.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-141">These are built specifically available for high end computational needs, have 8 and 16 core vCPU sizes available, and feature DDR4 memory, SSD temporary storage and Haswell E5 Intel technology.</span></span>
  - <span data-ttu-id="a5ed9-142">[Размеры виртуальных машин с GPU][compute-gpu]. Размеры виртуальных машин с оптимизацией GPU — это специализированные виртуальные машины с одним или несколькими GPU NVIDIA.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-142">[GPU VM sizes][compute-gpu] GPU optimized VM sizes are specialized virtual machines available with single or multiple NVIDIA GPUs.</span></span> <span data-ttu-id="a5ed9-143">Эти размеры предназначены для рабочих нагрузок с большим объемом вычислений, графической обработки и визуализаций.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-143">These sizes are designed for compute-intensive, graphics-intensive, and visualization workloads.</span></span>
    - <span data-ttu-id="a5ed9-144">Размеры NC, NCv2, NCv3 и ND оптимизированы для приложений и алгоритмов, требующих ресурсоемких вычислений и высокой пропускной способности сети, включая приложения и моделирование на базе CUDA и OpenCL, а также ИИ и глубокое обучение.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-144">NC, NCv2, NCv3, and ND sizes are optimized for compute-intensive and network-intensive applications and algorithms, including CUDA and OpenCL-based applications and simulations, AI, and Deep Learning.</span></span> <span data-ttu-id="a5ed9-145">Размеры NV оптимизированы и предназначены для удаленной визуализации, потоковой передачи, игр, кодирования и сценариев VDI, которые используют такие платформы, как OpenGL и DirectX.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-145">NV sizes are optimized and designed for remote visualization, streaming, gaming, encoding, and VDI scenarios utilizing frameworks such as OpenGL and DirectX.</span></span>
  - <span data-ttu-id="a5ed9-146">[Размеры виртуальных машин, оптимизированных для операций в памяти][compute-memory]. Когда требуется больше памяти, в размерах виртуальных машин, оптимизированных для операций в памяти, предоставляется высокое соотношение ресурсов памяти и ядра.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-146">[Memory optmised VM sizes][compute-memory] When more memory is required, the memory optimized VM sizes offer a higher memory-to-CPU ratio.</span></span>
  - <span data-ttu-id="a5ed9-147">[Размеры виртуальных машин общего назначения][compute-general]. Размеры виртуальных машин общего назначения также доступны и предоставляют сбалансированное соотношение ресурсов ЦП и памяти.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-147">[General purposes VM sizes][compute-general] General purpose VM sizes are also available and provide balanced CPU-to-memory ratio.</span></span>

### <a name="alternatives"></a><span data-ttu-id="a5ed9-148">Альтернативные варианты</span><span class="sxs-lookup"><span data-stu-id="a5ed9-148">Alternatives</span></span>

<span data-ttu-id="a5ed9-149">Если вам требуется больше контроля над средой отрисовки в Azure или нужна гибридная реализация, тогда вычисления CycleCloud могут помочь оркестрировать сетку IaaS в облаке.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-149">If you require more control over your rendering environment in Azure or need a hybrid implementation, then CycleCloud computing can help orchestrate an IaaS grid in the cloud.</span></span> <span data-ttu-id="a5ed9-150">Используя те же базовые технологии Azure, что и пакетная служба Azure, они позволяют эффективно создавать и поддерживать сетку IaaS.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-150">Using the same underlying Azure technologies as Azure Batch, it makes building and maintaining an IaaS grid an efficient process.</span></span> <span data-ttu-id="a5ed9-151">Чтобы узнать об этом побольше и изучить принципы проектирования, используйте следующую ссылку.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-151">To find out more and learn about the design principles, please use the following link:</span></span>

<span data-ttu-id="a5ed9-152">Полный обзор всех решений HPC, доступных в Azure, см. в статье [Использование HPC, пакетной службы и больших вычислений][hpc-alt-solutions]</span><span class="sxs-lookup"><span data-stu-id="a5ed9-152">For a complete overview of all the HPC solutions that are available to you in Azure, please see the article [HPC, Batch, and Big Compute solutions using Azure VMs][hpc-alt-solutions]</span></span>

### <a name="availability"></a><span data-ttu-id="a5ed9-153">Доступность</span><span class="sxs-lookup"><span data-stu-id="a5ed9-153">Availability</span></span>

<span data-ttu-id="a5ed9-154">Мониторинг компонентов пакетной службы Azure доступен в целом ряде служб, средств и API.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-154">Monitoring of the Azure Batch components is available through a range of services, tools and APIs.</span></span> <span data-ttu-id="a5ed9-155">Это описано далее в статье [Мониторинг решений пакетной службы][batch-monitor].</span><span class="sxs-lookup"><span data-stu-id="a5ed9-155">This is discussed further in the [Monitor Batch solutions][batch-monitor] article.</span></span>

### <a name="scalability"></a><span data-ttu-id="a5ed9-156">Масштабируемость</span><span class="sxs-lookup"><span data-stu-id="a5ed9-156">Scalability</span></span>

<span data-ttu-id="a5ed9-157">Пулы учетной записи можно масштабировать автоматически по формуле на основе метрик пакетной службы Azure, или же вручную.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-157">Pools within an Azure Batch account can either scale through manual intervention or, by using an formula based on Azure Batch metrics, be scaled automatically.</span></span> <span data-ttu-id="a5ed9-158">Дополнительные сведения см. в статье [Создание формулы автоматического масштабирования для масштабирования вычислительных узлов в пуле пакетной службы][batch-scaling].</span><span class="sxs-lookup"><span data-stu-id="a5ed9-158">For more information on this, please see the article [Create an automatic scaling formula for scaling nodes in a Batch pool][batch-scaling].</span></span>

### <a name="security"></a><span data-ttu-id="a5ed9-159">Безопасность</span><span class="sxs-lookup"><span data-stu-id="a5ed9-159">Security</span></span>

<span data-ttu-id="a5ed9-160">Общие рекомендации по разработке безопасных решений см. в разделе [Azure Security Documentation][security].</span><span class="sxs-lookup"><span data-stu-id="a5ed9-160">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="a5ed9-161">Устойчивость</span><span class="sxs-lookup"><span data-stu-id="a5ed9-161">Resiliency</span></span>

<span data-ttu-id="a5ed9-162">В настоящее время в пакетной службе Azure отсутствует возможность отработки отказа, поэтому рекомендуется выполнить следующие действия для обеспечения доступности в случае незапланированного сбоя:</span><span class="sxs-lookup"><span data-stu-id="a5ed9-162">While there is currently no failover capability in Azure Batch, we recommend using the following steps to ensure availbility in case of an unplanned outage:</span></span>

* <span data-ttu-id="a5ed9-163">Создайте учетную запись пакетной службы Azure в альтернативном расположение Azure с помощью альтернативной учетной записи хранения.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-163">Create a Azure Batch account in an alternate Azure location with an alternate Storage Account</span></span>
* <span data-ttu-id="a5ed9-164">Создайте одни и те же узлы пулов с тем же именем без выделенных узлов.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-164">Create the same node pools with the same name, with 0 nodes allocated</span></span>
* <span data-ttu-id="a5ed9-165">Убедитесь, что приложения создаются и обновляются в альтернативной учетной записи хранения.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-165">Ensure Applications are created and updated to the alternate Storage Account</span></span>
* <span data-ttu-id="a5ed9-166">Отправьте входные файлы и задания в альтернативную учетную запись пакетной службы Azure.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-166">Upload input files and submit jobs to the alternate Azure Batch account</span></span>

## <a name="deploy-this-scenario"></a><span data-ttu-id="a5ed9-167">Развертывание этого сценария</span><span class="sxs-lookup"><span data-stu-id="a5ed9-167">Deploy this scenario</span></span>

### <a name="creating-an-azure-batch-account-and-pools-manually"></a><span data-ttu-id="a5ed9-168">Создание учетной записи и пулов пакетной службы Azure вручную</span><span class="sxs-lookup"><span data-stu-id="a5ed9-168">Creating an Azure Batch account and pools manually</span></span>

<span data-ttu-id="a5ed9-169">В этом примере сценария предоставлена справка о работе пакетной службы Azure при демонстрации Azure Batch Labs. В качестве примера выступает решение SaaS, которое может быть разработано для ваших клиентов:</span><span class="sxs-lookup"><span data-stu-id="a5ed9-169">This sample scenario will provide help in learning how Azure Batch works while showcasing Azure Batch Labs as an example SaaS solution that can be developed for your own customers:</span></span>

<span data-ttu-id="a5ed9-170">[Демонстрация с пакетной службой Azure][batch-labs-masterclass]</span><span class="sxs-lookup"><span data-stu-id="a5ed9-170">[Azure Batch Masterclass][batch-labs-masterclass]</span></span>

### <a name="deploying-the-sample-scenario-using-an-azure-resource-manager-arm-template"></a><span data-ttu-id="a5ed9-171">Развертывание сценария с помощью шаблона Azure Resource Manager (ARM)</span><span class="sxs-lookup"><span data-stu-id="a5ed9-171">Deploying the sample scenario using an Azure Resource Manager (ARM) template</span></span>

<span data-ttu-id="a5ed9-172">Шаблон развертывает:</span><span class="sxs-lookup"><span data-stu-id="a5ed9-172">The template will deploy:</span></span>
  - <span data-ttu-id="a5ed9-173">Новую учетную запись пакетной службы Azure.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-173">A new Azure Batch account</span></span>
  - <span data-ttu-id="a5ed9-174">Учетная запись хранения.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-174">A storage account</span></span>
  - <span data-ttu-id="a5ed9-175">Пул узла, связанный с учетной записью пакетной службы.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-175">A node pool associated with the batch account</span></span>
  - <span data-ttu-id="a5ed9-176">Пул узла, который будет настроен для использования виртуальных машин A2 v2 с образами Canonical Ubuntu.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-176">The node pool will be configured to use A2 v2 VMs with Canonical Ubuntu images</span></span>
  - <span data-ttu-id="a5ed9-177">Пул узла, который изначально не будет содержать виртуальных машин и потребует масштабирование вручную для их добавления.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-177">The node pool will contain 0 VMs initially and will require scaling manually to add VMs</span></span>

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fhpc%2Fbatchcreatewithpools.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

<span data-ttu-id="a5ed9-178">[Дополнительные сведения о шаблонах ARM][azure-arm-templates]</span><span class="sxs-lookup"><span data-stu-id="a5ed9-178">[Learn more about ARM templates][azure-arm-templates]</span></span>

## <a name="pricing"></a><span data-ttu-id="a5ed9-179">Цены</span><span class="sxs-lookup"><span data-stu-id="a5ed9-179">Pricing</span></span>

<span data-ttu-id="a5ed9-180">Затраты на использование пакетной службы Azure будут зависеть от размеров виртуальных машин, которые используются для пулов, и от того, как долго они выделяются и выполняются. Плата за создание учетной записи пакетной службы Azure отсутствует.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-180">The cost of using Azure Batch will depend on the VM sizes that are used for the pools and how long these are allocated and running, there is no cost associated with an Azure Batch account creation.</span></span> <span data-ttu-id="a5ed9-181">Исходящий трафик хранилища и данных также следует учитывать, так как за них взимается дополнительная плата.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-181">Storage and data egress should also be taken into account as these will apply additional costs.</span></span>

<span data-ttu-id="a5ed9-182">Ниже приведены примеры взимаемых затрат для задания, выполняющегося за 8 часов с разным количеством серверов:</span><span class="sxs-lookup"><span data-stu-id="a5ed9-182">The following are examples of costs that could be incurred for a job that completes in 8 hours using a different number of servers:</span></span>


- <span data-ttu-id="a5ed9-183">100 виртуальных машин с высокопроизводительным ЦП: [оценка затрат][hpc-est-high]</span><span class="sxs-lookup"><span data-stu-id="a5ed9-183">100 High Performance CPU VMs: [Cost Estimate][hpc-est-high]</span></span>

  <span data-ttu-id="a5ed9-184">100 x H16m (16 ядер, 225 ГБ ОЗУ, хранилище класса Premium 512 ГБ), хранилище BLOB-объектов — 2 ТБ, 1 ТБ исходящего трафика</span><span class="sxs-lookup"><span data-stu-id="a5ed9-184">100 x H16m (16 cores, 225GB RAM, Premium Storage 512GB), 2 TB Blob Storage, 1 TB egress</span></span>

- <span data-ttu-id="a5ed9-185">50 виртуальных машин с высокопроизводительным ЦП: [оценка затрат][hpc-est-med]</span><span class="sxs-lookup"><span data-stu-id="a5ed9-185">50 High Performance CPU VMs: [Cost Estimate][hpc-est-med]</span></span>

  <span data-ttu-id="a5ed9-186">50 x H16m (16 ядер, 225 ГБ ОЗУ, хранилище класса Premium 512 ГБ), хранилище BLOB-объектов — 2 ТБ, 1 ТБ исходящего трафика</span><span class="sxs-lookup"><span data-stu-id="a5ed9-186">50 x H16m (16 cores, 225GB RAM, Premium Storage 512GB), 2 TB Blob Storage, 1 TB egress</span></span>

- <span data-ttu-id="a5ed9-187">10 виртуальных машин с высокопроизводительным ЦП: [оценка затрат][hpc-est-low]</span><span class="sxs-lookup"><span data-stu-id="a5ed9-187">10 High Performance CPU VMs: [Cost Estimate][hpc-est-low]</span></span>
  
  <span data-ttu-id="a5ed9-188">10 x H16m (16 ядер, 225 ГБ ОЗУ, хранилище класса Premium 512 ГБ), хранилище BLOB-объектов — 2 ТБ, 1 ТБ исходящего трафика</span><span class="sxs-lookup"><span data-stu-id="a5ed9-188">10 x H16m (16 cores, 225GB RAM, Premium Storage 512GB), 2 TB Blob Storage, 1 TB egress</span></span>

### <a name="low-priority-vm-pricing"></a><span data-ttu-id="a5ed9-189">Виртуальные машины с низким приоритетом (цены)</span><span class="sxs-lookup"><span data-stu-id="a5ed9-189">Low Priority VM Pricing</span></span>

<span data-ttu-id="a5ed9-190">Пакетная служба Azure также поддерживает использование в пулах узлов виртуальных машин с низким приоритетом\*, которые фундаментально уменьшают затраты.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-190">Azure Batch also supports the use of Low Priority VMs\* in the node pools, which can potentially provide a substantial cost saving.</span></span> <span data-ttu-id="a5ed9-191">Сравнение цен стандартных виртуальных машин и виртуальных машин с низким приоритетом, а также дополнительные сведения о виртуальных машинах с низким приоритетом см. в статье [Цены на пакетную службу][batch-pricing].</span><span class="sxs-lookup"><span data-stu-id="a5ed9-191">For a price comparison between standard VMs and Low Priority VMs, and to find out more about Low Priority VMs, please see [Batch Pricing][batch-pricing].</span></span>

<span data-ttu-id="a5ed9-192">\* Обратите внимание, что только некоторые приложения и рабочие нагрузки подходят для запуска на виртуальных машинах с низким приоритетом.</span><span class="sxs-lookup"><span data-stu-id="a5ed9-192">\* Please note that only certain applications and workloads will be suitable to run on Low Priority VMs.</span></span>

## <a name="related-resources"></a><span data-ttu-id="a5ed9-193">Связанные ресурсы</span><span class="sxs-lookup"><span data-stu-id="a5ed9-193">Related Resources</span></span>

<span data-ttu-id="a5ed9-194">[Пакетная служба][batch-overview]</span><span class="sxs-lookup"><span data-stu-id="a5ed9-194">[Azure Batch Overview][batch-overview]</span></span>

<span data-ttu-id="a5ed9-195">[Документация по пакетной службе][batch-doc]</span><span class="sxs-lookup"><span data-stu-id="a5ed9-195">[Azure Batch Documentation][batch-doc]</span></span>

<span data-ttu-id="a5ed9-196">[Использование контейнеров в пакетной службе Azure][batch-containers]</span><span class="sxs-lookup"><span data-stu-id="a5ed9-196">[Using containers on Azure Batch][batch-containers]</span></span>

<!-- links -->
[architecture]: ./media/native-hpc-ref-arch.png
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[security]: /azure/security/
[resiliency]: /azure/architecture/resiliency/
[scalability]: /azure/architecture/checklist/scalability
[vmss]: /azure/virtual-machine-scale-sets/overview
[vnet]: /azure/virtual-network/virtual-networks-overview
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
[batch-pricing]: https://azure.microsoft.com/en-gb/pricing/details/batch/
[batch-doc]: /azure/batch/
[batch-overview]: https://azure.microsoft.com/services/batch/
[batch-containers]: https://github.com/Azure/batch-shipyard
[azure-arm-templates]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[batch-plugins]: /azure/batch/batch-rendering-service#options-for-submitting-a-render-job
