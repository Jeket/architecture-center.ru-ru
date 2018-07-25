---
title: SAP для рабочих нагрузок разработки и тестирования
description: Сценарии SAP для среды разработки и тестирования
author: AndrewDibbins
ms.date: 7/11/18
ms.openlocfilehash: 675a5cb4b1ee4001ca50d24c145ce1a177f90da4
ms.sourcegitcommit: 71cbef121c40ef36e2d6e3a088cb85c4260599b9
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/14/2018
ms.locfileid: "39060965"
---
# <a name="sap-for-devtest-workloads"></a><span data-ttu-id="17033-103">SAP для рабочих нагрузок разработки и тестирования</span><span class="sxs-lookup"><span data-stu-id="17033-103">SAP for dev/test workloads</span></span>

<span data-ttu-id="17033-104">В этом примере приведены рекомендации по выполнению реализации SAP NetWeaver для разработки и тестирования в среде Windows или Linux в Azure.</span><span class="sxs-lookup"><span data-stu-id="17033-104">This example provides guidance for how to run a dev/test implementation of SAP NetWeaver in a Windows or Linux environment on Azure.</span></span> <span data-ttu-id="17033-105">База данных сторонних поставщиков, или так называемая база данных AnyDB, — это термин SAP, который применяется к любой поддерживаемой СУБД, кроме SAP HANA.</span><span class="sxs-lookup"><span data-stu-id="17033-105">The database used is AnyDB, the SAP term for any supported DBMS (that isn't SAP HANA).</span></span> <span data-ttu-id="17033-106">Так как эта архитектура предназначена для нерабочих сред, она развертывается только с одной виртуальной машиной, и ее размер может быть изменен для удовлетворения потребностей вашей организации.</span><span class="sxs-lookup"><span data-stu-id="17033-106">Because this architecture is designed for non-production environments, it's deployed with just a single virtual machine (VM) and it's size can be changed to accommodate your organization's needs.</span></span>

<span data-ttu-id="17033-107">Для примеров использования в рабочей среде рассмотрите эталонные архитектуры SAP, доступные ниже:</span><span class="sxs-lookup"><span data-stu-id="17033-107">For production use cases review the SAP reference architectures available below:</span></span>

* <span data-ttu-id="17033-108">[SAP NetWeaver для баз данных сторонних поставщиков (AnyDB)][sap-netweaver]</span><span class="sxs-lookup"><span data-stu-id="17033-108">[SAP netweaver for AnyDB][sap-netweaver]</span></span>
* <span data-ttu-id="17033-109">[SAP S/4 HANA][sap-hana]</span><span class="sxs-lookup"><span data-stu-id="17033-109">[SAP S/4Hana][sap-hana]</span></span>
* <span data-ttu-id="17033-110">[SAP HANA на крупных экземплярах Azure][sap-large]</span><span class="sxs-lookup"><span data-stu-id="17033-110">[SAP on Azure large instances][sap-large]</span></span>

## <a name="related-use-cases"></a><span data-ttu-id="17033-111">Связанные варианты использования</span><span class="sxs-lookup"><span data-stu-id="17033-111">Related use cases</span></span>

<span data-ttu-id="17033-112">Рассмотрите этот сценарий для следующих вариантов использования:</span><span class="sxs-lookup"><span data-stu-id="17033-112">Consider this scenario for the following use cases:</span></span>

* <span data-ttu-id="17033-113">Некритические нерабочие нагрузки SAP (песочница, разработка, тестирование, контроль качества).</span><span class="sxs-lookup"><span data-stu-id="17033-113">Non-critical SAP non-productive workloads (sandbox, development, test, quality assurance)</span></span>
* <span data-ttu-id="17033-114">Некритические рабочие нагрузки бизнес-процессов SAP.</span><span class="sxs-lookup"><span data-stu-id="17033-114">Non-critical SAP business one workloads</span></span>

## <a name="architecture"></a><span data-ttu-id="17033-115">Архитектура</span><span class="sxs-lookup"><span data-stu-id="17033-115">Architecture</span></span>

![Схема](media/sap-2tier/SAP-Infra-2Tier_finalversion.png)

<span data-ttu-id="17033-117">Этот сценарий охватывает предоставление единой системной базы и сервера приложений SAP на одной виртуальной машине, где данные передаются следующим образом:</span><span class="sxs-lookup"><span data-stu-id="17033-117">This scenario covers the provision of a single SAP system database and SAP application Server on a single virtual machine, the data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="17033-118">Клиенты из уровня презентации используют свой пользовательский интерфейс SAP или другие интерфейсы (Internet Explorer, Excel или другое веб-приложение) в локальной среде для доступа к системе SAP на основе Azure.</span><span class="sxs-lookup"><span data-stu-id="17033-118">Customers from the Presentation Tier use their SAP gui, or other user interfaces (Internet Explorer, Excel or other web application) on premise to access the Azure based SAP system.</span></span>
2. <span data-ttu-id="17033-119">Связь обеспечивается с помощью установленного подключения Express Route.</span><span class="sxs-lookup"><span data-stu-id="17033-119">Connectivity is provided through the use of the established Express Route.</span></span> <span data-ttu-id="17033-120">Express Route прерывается в Azure в шлюзе Express Route.</span><span class="sxs-lookup"><span data-stu-id="17033-120">The Express Route is terminated in Azure at the Express Route Gateway.</span></span> <span data-ttu-id="17033-121">Сетевой трафик проходит через шлюз Express Route к подсети шлюза и от подсети шлюза к подсети уровня приложения (см. шаблон [звездообразной топологии][hub-spoke]) и через сетевой шлюз безопасности виртуальной машины приложения SAP.</span><span class="sxs-lookup"><span data-stu-id="17033-121">Network traffic routes through the Express Route gateway to the Gateway Subnet and from the gateway subnet to the Application Tier Spoke subnet (see the [hub-spoke][hub-spoke] pattern) and via a Network Security Gateway to the SAP application virtual machine.</span></span>
3. <span data-ttu-id="17033-122">Серверы управления идентификацией предоставляют услуги проверки подлинности.</span><span class="sxs-lookup"><span data-stu-id="17033-122">The identity management servers provide authentication services.</span></span>
4. <span data-ttu-id="17033-123">Переходная среда предоставляет возможности локального управления.</span><span class="sxs-lookup"><span data-stu-id="17033-123">The Jump Box provides local management capabilities.</span></span>

### <a name="components"></a><span data-ttu-id="17033-124">Компоненты</span><span class="sxs-lookup"><span data-stu-id="17033-124">Components</span></span>

* <span data-ttu-id="17033-125">[Группа ресурсов](/azure/azure-resource-manager/resource-group-overview#resource-groups) — это логический контейнер для ресурсов Azure.</span><span class="sxs-lookup"><span data-stu-id="17033-125">[Resource Groups](/azure/azure-resource-manager/resource-group-overview#resource-groups) is a logical container for Azure resources.</span></span>
* <span data-ttu-id="17033-126">[Виртуальные сети](/azure/virtual-network/virtual-networks-overview) — это основа сетевых коммуникаций в Azure.</span><span class="sxs-lookup"><span data-stu-id="17033-126">[Virtual Networks](/azure/virtual-network/virtual-networks-overview) is the basis of network communications within Azure</span></span>
* <span data-ttu-id="17033-127">[Виртуальные машины](/azure/virtual-machines/windows/overview) Azure предоставляют по запросу безопасную высокомасштабируемую виртуализированную инфраструктуру с использованием сервера Windows или Linux.</span><span class="sxs-lookup"><span data-stu-id="17033-127">[Virtual Machine](/azure/virtual-machines/windows/overview) Azure Virtual Machines provides on-demand, high-scale, secure, virtualized infrastructure using Windows or Linux Server</span></span>
* <span data-ttu-id="17033-128">[ExpressRoute](/azure/expressroute/expressroute-introduction) позволяет переносить локальные сети в Microsoft Cloud по частному подключению, которое обеспечивается поставщиком услуг подключения.</span><span class="sxs-lookup"><span data-stu-id="17033-128">[Express Route](/azure/expressroute/expressroute-introduction) lets you extend your on-premises networks into the Microsoft cloud over a private connection facilitated by a connectivity provider.</span></span>
* <span data-ttu-id="17033-129">[Группы безопасности сети](/azure/virtual-network/security-overview) позволяют ограничить трафик к ресурсам в виртуальной сети.</span><span class="sxs-lookup"><span data-stu-id="17033-129">[Network Security Group](/azure/virtual-network/security-overview) lets you limit network traffic to resources in a virtual network.</span></span> <span data-ttu-id="17033-130">Группа безопасности сети содержит список правил безопасности, которые разрешают или запрещают входящий и исходящий трафик в зависимости от IP-адреса источника или назначения, порта и протокола.</span><span class="sxs-lookup"><span data-stu-id="17033-130">A network security group contains a list of security rules that allow or deny inbound or outbound network traffic based on source or destination IP address, port, and protocol.</span></span> 

## <a name="considerations"></a><span data-ttu-id="17033-131">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="17033-131">Considerations</span></span>

### <a name="availability"></a><span data-ttu-id="17033-132">Доступность</span><span class="sxs-lookup"><span data-stu-id="17033-132">Availability</span></span>

 <span data-ttu-id="17033-133">Корпорация Майкрософт предлагает Соглашение об уровне обслуживания (SLA) для отдельных экземпляров виртуальной машины.</span><span class="sxs-lookup"><span data-stu-id="17033-133">Microsoft offers a service level agreement (SLA) for single VM instances.</span></span> <span data-ttu-id="17033-134">Дополнительные сведения о Соглашении об уровне обслуживания Microsoft Azure для виртуальных машин см. в [этой статье](https://azure.microsoft.com/support/legal/sla/virtual-machines)</span><span class="sxs-lookup"><span data-stu-id="17033-134">For more information on Microsoft Azure Service Level Agreement for Virtual Machines [SLA For Virtual Machines](https://azure.microsoft.com/support/legal/sla/virtual-machines)</span></span>

### <a name="scalability"></a><span data-ttu-id="17033-135">Масштабируемость</span><span class="sxs-lookup"><span data-stu-id="17033-135">Scalability</span></span>

<span data-ttu-id="17033-136">Общие рекомендации по разработке масштабируемых решений см. в разделе [Контрольный список для обеспечения масштабируемости][scalability] в центре архитектуры Azure.</span><span class="sxs-lookup"><span data-stu-id="17033-136">For general guidance on designing scalable solutions, see the [scalability checklist][scalability] in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="17033-137">Безопасность</span><span class="sxs-lookup"><span data-stu-id="17033-137">Security</span></span>

<span data-ttu-id="17033-138">Общие рекомендации по разработке безопасных решений см. в разделе [Azure Security Documentation][security].</span><span class="sxs-lookup"><span data-stu-id="17033-138">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="17033-139">Устойчивость</span><span class="sxs-lookup"><span data-stu-id="17033-139">Resiliency</span></span>

<span data-ttu-id="17033-140">Общее руководство по проектированию устойчивых решений см. в разделе [Проектирование устойчивых приложений для Azure][resiliency].</span><span class="sxs-lookup"><span data-stu-id="17033-140">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="pricing"></a><span data-ttu-id="17033-141">Цены</span><span class="sxs-lookup"><span data-stu-id="17033-141">Pricing</span></span>

<span data-ttu-id="17033-142">Чтобы изучить стоимость выполнения этого сценария, все услуги были предварительно сконфигурированы в калькуляторе стоимости.</span><span class="sxs-lookup"><span data-stu-id="17033-142">Explore the cost of running this scenario, all of the services are pre-configured in the cost calculator.</span></span>  <span data-ttu-id="17033-143">Чтобы узнать, как изменится цена для вашего конкретного варианта использования, измените соответствующие переменные в соответствии с ожидаемым трафиком.</span><span class="sxs-lookup"><span data-stu-id="17033-143">To see how the pricing would change for your particular use case change the appropriate variables to match your expected traffic.</span></span>

<span data-ttu-id="17033-144">Здесь предоставлено четыре примера профиля затрат в зависимости от объема трафика, который планируется принимать.</span><span class="sxs-lookup"><span data-stu-id="17033-144">We have provided four sample cost profiles based on amount of traffic you expect to get:</span></span>

|<span data-ttu-id="17033-145">Размер</span><span class="sxs-lookup"><span data-stu-id="17033-145">Size</span></span>|<span data-ttu-id="17033-146">Протоколы SAP</span><span class="sxs-lookup"><span data-stu-id="17033-146">SAPs</span></span>|<span data-ttu-id="17033-147">Тип виртуальной машины</span><span class="sxs-lookup"><span data-stu-id="17033-147">VM Type</span></span>|<span data-ttu-id="17033-148">Служба хранилища</span><span class="sxs-lookup"><span data-stu-id="17033-148">Storage</span></span>|<span data-ttu-id="17033-149">Калькулятор стоимости Azure</span><span class="sxs-lookup"><span data-stu-id="17033-149">Azure Pricing Calculator</span></span>|
|----|----|-------|-------|---------------|
|<span data-ttu-id="17033-150">Малый</span><span class="sxs-lookup"><span data-stu-id="17033-150">Small</span></span>|<span data-ttu-id="17033-151">8000</span><span class="sxs-lookup"><span data-stu-id="17033-151">8000</span></span>|<span data-ttu-id="17033-152">D8s_v3</span><span class="sxs-lookup"><span data-stu-id="17033-152">D8s_v3</span></span>|<span data-ttu-id="17033-153">2xP20, 1xP10</span><span class="sxs-lookup"><span data-stu-id="17033-153">2xP20, 1xP10</span></span>|[<span data-ttu-id="17033-154">Малый</span><span class="sxs-lookup"><span data-stu-id="17033-154">Small</span></span>](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1)|
|<span data-ttu-id="17033-155">Средний</span><span class="sxs-lookup"><span data-stu-id="17033-155">Medium</span></span>|<span data-ttu-id="17033-156">16000</span><span class="sxs-lookup"><span data-stu-id="17033-156">16000</span></span>|<span data-ttu-id="17033-157">D16s_v3</span><span class="sxs-lookup"><span data-stu-id="17033-157">D16s_v3</span></span>|<span data-ttu-id="17033-158">3xP20, 1xP10</span><span class="sxs-lookup"><span data-stu-id="17033-158">3xP20, 1xP10</span></span>|[<span data-ttu-id="17033-159">Средний</span><span class="sxs-lookup"><span data-stu-id="17033-159">Medium</span></span>](https://azure.com/e/465bd07047d148baab032b2f461550cd)|
<span data-ttu-id="17033-160">большой</span><span class="sxs-lookup"><span data-stu-id="17033-160">Large</span></span>|<span data-ttu-id="17033-161">32000</span><span class="sxs-lookup"><span data-stu-id="17033-161">32000</span></span>|<span data-ttu-id="17033-162">E32s_v3</span><span class="sxs-lookup"><span data-stu-id="17033-162">E32s_v3</span></span>|<span data-ttu-id="17033-163">3xP20, 1xP10</span><span class="sxs-lookup"><span data-stu-id="17033-163">3xP20, 1xP10</span></span>|[<span data-ttu-id="17033-164">Крупный</span><span class="sxs-lookup"><span data-stu-id="17033-164">Large</span></span>](https://azure.com/e/ada2e849d68b41c3839cc976000c6931)|
<span data-ttu-id="17033-165">Очень крупный</span><span class="sxs-lookup"><span data-stu-id="17033-165">Extra Large</span></span>|<span data-ttu-id="17033-166">64000</span><span class="sxs-lookup"><span data-stu-id="17033-166">64000</span></span>|<span data-ttu-id="17033-167">M64s</span><span class="sxs-lookup"><span data-stu-id="17033-167">M64s</span></span>|<span data-ttu-id="17033-168">4xP20, 1xP10</span><span class="sxs-lookup"><span data-stu-id="17033-168">4xP20, 1xP10</span></span>|[<span data-ttu-id="17033-169">Очень крупный</span><span class="sxs-lookup"><span data-stu-id="17033-169">Extra Large</span></span>](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef)|

<span data-ttu-id="17033-170">Примечание. Приведенное ценообразование примерное и указывает только расходы на виртуальную машину и хранилища (исключая сетевые ресурсы, резервное хранилище и входящие и исходящие платежи).</span><span class="sxs-lookup"><span data-stu-id="17033-170">Note: pricing is a guide and only indicates the VMs and storage costs (excludes, networking, backup storage and data ingress/egress charges).</span></span>

* <span data-ttu-id="17033-171">[Малый](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1): небольшая система состоит из виртуальной машины типа D8s_v3 с 8 виртуальными ЦП, 32 ГБ оперативной памяти и 200 ГБ временного хранилища, а также дополнительно двух дисков хранения уровня "Премиум" на 512 ГБ и одного на 128 ГБ.</span><span class="sxs-lookup"><span data-stu-id="17033-171">[Small](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1): A small system consists of VM type D8s_v3 with 8x vCPUs, 32GB RAM and 200GB temp storage, additionally two 512GB and one 128GB premium storage disks.</span></span>
* <span data-ttu-id="17033-172">[Средний](https://azure.com/e/465bd07047d148baab032b2f461550cd): средняя система состоит из виртуальной машины типа D16s_v3 с 16 виртуальными ЦП, 64 ГБ оперативной памяти и 400 ГБ временного хранилища, а также дополнительно трех дисков хранения уровня "Премиум" на 512 ГБ и одного на 128 ГБ.</span><span class="sxs-lookup"><span data-stu-id="17033-172">[Medium](https://azure.com/e/465bd07047d148baab032b2f461550cd): A medium system consists of VM type D16s_v3 with 16x vCPUs, 64GB RAM and 400GB temp storage, additionally three 512GB and one 128GB premium storage disks.</span></span>
* <span data-ttu-id="17033-173">[Крупный](https://azure.com/e/ada2e849d68b41c3839cc976000c6931): крупная система состоит из виртуальной машины типа E32s_v3 с 32 виртуальными ЦП, 256 ГБ оперативной памяти и 512 ГБ временного хранилища, а также дополнительно трех дисков хранения уровня "Премиум" на 512 ГБ и одного на 128 ГБ.</span><span class="sxs-lookup"><span data-stu-id="17033-173">[Large](https://azure.com/e/ada2e849d68b41c3839cc976000c6931): A large system consists of VM type E32s_v3 with 32x vCPUs, 256GB RAM and 512GB temp storage, additionally three 512GB and one 128GB premium storage disks.</span></span>
* <span data-ttu-id="17033-174">[Очень крупный](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef): очень крупная система состоит из виртуальной машины типа M64s с 64 виртуальными ЦП, 1024 ГБ оперативной памяти и 2000 ГБ временного хранилища, а также дополнительно четырех дисков хранения уровня "Премиум" на 512 ГБ и одного на 128 ГБ.</span><span class="sxs-lookup"><span data-stu-id="17033-174">[Extra Large](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef): An extra large system consists of a VM type M64s with 64x vCPUs, 1024GB RAM and 2000GB temp storage, additionally four 512GB and one 128GB premium storage disks.</span></span>

## <a name="deployment"></a><span data-ttu-id="17033-175">Развертывание</span><span class="sxs-lookup"><span data-stu-id="17033-175">Deployment</span></span>

<span data-ttu-id="17033-176">Чтобы развернуть базовую инфраструктуру, аналогичную описанному выше сценарию, используйте кнопку развертывания</span><span class="sxs-lookup"><span data-stu-id="17033-176">To deploy the underlying infrastructure similar to the scenario above, please use the deploy button</span></span>

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fsap-2tier%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

<span data-ttu-id="17033-177">\* Среда SAP не будет установлена, вам нужно будет сделать это после того, как инфраструктура будет создана вручную.</span><span class="sxs-lookup"><span data-stu-id="17033-177">\* SAP will not be installed, you'll need to do this after the infrastructure is built manually.</span></span>

<!-- links -->
[reference architecture]:  /azure/architecture/reference-architectures/sap
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[sap-netweaver]: /azure/architecture/reference-architectures/sap/sap-netweaver
[sap-hana]: /azure/architecture/reference-architectures/sap/sap-s4hana
[sap-large]: /azure/architecture/reference-architectures/sap/hana-large-instances
[hub-spoke]: /azure/architecture/reference-architectures/hybrid-networking/hub-spoke