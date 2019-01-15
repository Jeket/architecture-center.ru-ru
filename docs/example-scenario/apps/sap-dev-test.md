---
title: Среды разработки и тестирования для рабочих нагрузок SAP
titleSuffix: Azure Example Scenarios
description: Создание сред разработки и тестирования для рабочих нагрузок SAP.
author: AndrewDibbins
ms.date: 07/11/2018
ms.custom: fasttrack
ms.openlocfilehash: 9f9e8ec971373e4309703800c200ba2c62fe9a66
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54111025"
---
# <a name="devtest-environments-for-sap-workloads-on-azure"></a><span data-ttu-id="82c4c-103">Среды разработки и тестирования для рабочих нагрузок SAP в Azure</span><span class="sxs-lookup"><span data-stu-id="82c4c-103">Dev/test environments for SAP workloads on Azure</span></span>

<span data-ttu-id="82c4c-104">В этом примере показано, как установить среду разработки и тестирования для SAP NetWeaver в окружении Windows или Linux в Azure.</span><span class="sxs-lookup"><span data-stu-id="82c4c-104">This example shows how to establish a dev/test environment for SAP NetWeaver in a Windows or Linux environment on Azure.</span></span> <span data-ttu-id="82c4c-105">База данных сторонних поставщиков, или так называемая база данных AnyDB, — это термин SAP, который применяется к любой поддерживаемой СУБД, кроме SAP HANA.</span><span class="sxs-lookup"><span data-stu-id="82c4c-105">The database used is AnyDB, the SAP term for any supported DBMS (that isn't SAP HANA).</span></span> <span data-ttu-id="82c4c-106">Так как эта архитектура предназначена для нерабочих сред, она развертывается только с одной виртуальной машиной, и ее размер может быть изменен для удовлетворения потребностей вашей организации.</span><span class="sxs-lookup"><span data-stu-id="82c4c-106">Because this architecture is designed for non-production environments, it's deployed with just a single virtual machine (VM) and it's size can be changed to accommodate your organization's needs.</span></span>

<span data-ttu-id="82c4c-107">Для примеров использования в рабочей среде рассмотрите эталонные архитектуры SAP, доступные ниже:</span><span class="sxs-lookup"><span data-stu-id="82c4c-107">For production use cases review the SAP reference architectures available below:</span></span>

- <span data-ttu-id="82c4c-108">[Развертывание SAP NetWeaver (Windows) для баз сторонних поставщиков на виртуальных машинах Azure][sap-netweaver]</span><span class="sxs-lookup"><span data-stu-id="82c4c-108">[SAP NetWeaver for AnyDB][sap-netweaver]</span></span>
- <span data-ttu-id="82c4c-109">[SAP S/4HANA для виртуальных машин Linux в Azure][sap-hana]</span><span class="sxs-lookup"><span data-stu-id="82c4c-109">[SAP S/4HANA][sap-hana]</span></span>
- <span data-ttu-id="82c4c-110">[SAP HANA на крупных экземплярах Azure][sap-large]</span><span class="sxs-lookup"><span data-stu-id="82c4c-110">[SAP on Azure large instances][sap-large]</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="82c4c-111">Варианты соответствующего использования</span><span class="sxs-lookup"><span data-stu-id="82c4c-111">Relevant use cases</span></span>

<span data-ttu-id="82c4c-112">Другие варианты использования:</span><span class="sxs-lookup"><span data-stu-id="82c4c-112">Other relevant use cases include:</span></span>

- <span data-ttu-id="82c4c-113">Некритические нерабочие нагрузки SAP (песочница, разработка, тестирование, контроль качества).</span><span class="sxs-lookup"><span data-stu-id="82c4c-113">Non-critical SAP non-productive workloads (sandbox, development, test, quality assurance)</span></span>
- <span data-ttu-id="82c4c-114">Некритические рабочие нагрузки бизнес-процессов SAP.</span><span class="sxs-lookup"><span data-stu-id="82c4c-114">Non-critical SAP business workloads</span></span>

## <a name="architecture"></a><span data-ttu-id="82c4c-115">Архитектура</span><span class="sxs-lookup"><span data-stu-id="82c4c-115">Architecture</span></span>

![Схема архитектуры для сред разработки и тестирования для рабочих нагрузок SAP](./media/architecture-sap-dev-test.png)

<span data-ttu-id="82c4c-117">В этом сценарии показана подготовка отдельной базы данных системы SAP и сервера приложений SAP в одной виртуальной машине.</span><span class="sxs-lookup"><span data-stu-id="82c4c-117">This scenario demonstrates provisioning a single SAP system database and SAP application server on a single virtual machine.</span></span> <span data-ttu-id="82c4c-118">Данные передаются в сценарии следующим образом:</span><span class="sxs-lookup"><span data-stu-id="82c4c-118">The data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="82c4c-119">Для доступа к системе SAP на платформе Azure клиенты используют пользовательский интерфейс SAP или другие клиентские средства (Excel, веб-браузер или другие веб-приложения).</span><span class="sxs-lookup"><span data-stu-id="82c4c-119">Customers use the SAP user interface or other client tools (Excel, a web browser, or other web application) to access the Azure-based SAP system.</span></span>
2. <span data-ttu-id="82c4c-120">Связь обеспечивается с помощью установленного подключения ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="82c4c-120">Connectivity is provided through the use of an established ExpressRoute.</span></span> <span data-ttu-id="82c4c-121">Подключение ExpressRoute прерывается в Azure в шлюзе ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="82c4c-121">The ExpressRoute connection is terminated in Azure at the ExpressRoute gateway.</span></span> <span data-ttu-id="82c4c-122">Сетевой трафик проходит через шлюз Express Route к подсети шлюза, от подсети шлюза к подсети уровня приложения (см. шаблон [звездообразной топологии][hub-spoke]), а затем через сетевой шлюз безопасности к виртуальной машине приложения SAP.</span><span class="sxs-lookup"><span data-stu-id="82c4c-122">Network traffic routes through the ExpressRoute gateway to the gateway subnet, and from the gateway subnet to the application-tier spoke subnet (see the [hub-spoke network topology][hub-spoke]) and via a Network Security Gateway to the SAP application virtual machine.</span></span>
3. <span data-ttu-id="82c4c-123">Серверы управления идентификацией предоставляют услуги проверки подлинности.</span><span class="sxs-lookup"><span data-stu-id="82c4c-123">The identity management servers provide authentication services.</span></span>
4. <span data-ttu-id="82c4c-124">Переходная среда предоставляет возможности локального управления.</span><span class="sxs-lookup"><span data-stu-id="82c4c-124">The jump box provides local management capabilities.</span></span>

### <a name="components"></a><span data-ttu-id="82c4c-125">Компоненты</span><span class="sxs-lookup"><span data-stu-id="82c4c-125">Components</span></span>

- <span data-ttu-id="82c4c-126">[Виртуальные сети](/azure/virtual-network/virtual-networks-overview) служат основой для сетевого взаимодействия в Azure.</span><span class="sxs-lookup"><span data-stu-id="82c4c-126">[Virtual Networks](/azure/virtual-network/virtual-networks-overview) are the basis of network communication within Azure.</span></span>
- <span data-ttu-id="82c4c-127">[Виртуальная машина](/azure/virtual-machines/windows/overview). Виртуальные машины Azure предоставляют доступную по требованию, масштабируемую, безопасную виртуализированную инфраструктуру с помощью Windows Server или Linux Server.</span><span class="sxs-lookup"><span data-stu-id="82c4c-127">[Virtual Machine](/azure/virtual-machines/windows/overview) Azure Virtual Machines provides on-demand, high-scale, secure, virtualized infrastructure using Windows or Linux Server.</span></span>
- <span data-ttu-id="82c4c-128">[ExpressRoute](/azure/expressroute/expressroute-introduction) позволяет переносить локальные сети в Microsoft Cloud по частному подключению, которое обеспечивается поставщиком услуг подключения.</span><span class="sxs-lookup"><span data-stu-id="82c4c-128">[ExpressRoute](/azure/expressroute/expressroute-introduction) lets you extend your on-premises networks into the Microsoft cloud over a private connection facilitated by a connectivity provider.</span></span>
- <span data-ttu-id="82c4c-129">[Группы безопасности сети](/azure/virtual-network/security-overview) позволяют ограничить трафик к ресурсам в виртуальной сети.</span><span class="sxs-lookup"><span data-stu-id="82c4c-129">[Network Security Group](/azure/virtual-network/security-overview) lets you limit network traffic to resources in a virtual network.</span></span> <span data-ttu-id="82c4c-130">Группа безопасности сети содержит список правил безопасности, которые разрешают или запрещают входящий и исходящий трафик в зависимости от IP-адреса источника или назначения, порта и протокола.</span><span class="sxs-lookup"><span data-stu-id="82c4c-130">A network security group contains a list of security rules that allow or deny inbound or outbound network traffic based on source or destination IP address, port, and protocol.</span></span>
- <span data-ttu-id="82c4c-131">[Группы ресурсов](/azure/azure-resource-manager/resource-group-overview#resource-groups) действуют как логические контейнеры для ресурсов Azure.</span><span class="sxs-lookup"><span data-stu-id="82c4c-131">[Resource Groups](/azure/azure-resource-manager/resource-group-overview#resource-groups) act as logical containers for Azure resources.</span></span>

## <a name="considerations"></a><span data-ttu-id="82c4c-132">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="82c4c-132">Considerations</span></span>

### <a name="availability"></a><span data-ttu-id="82c4c-133">Доступность</span><span class="sxs-lookup"><span data-stu-id="82c4c-133">Availability</span></span>

<span data-ttu-id="82c4c-134">Корпорация Майкрософт предлагает Соглашение об уровне обслуживания (SLA) для отдельных экземпляров виртуальной машины.</span><span class="sxs-lookup"><span data-stu-id="82c4c-134">Microsoft offers a service level agreement (SLA) for single VM instances.</span></span> <span data-ttu-id="82c4c-135">Дополнительные сведения о Соглашении об уровне обслуживания Microsoft Azure для виртуальных машин см. в [этой статье](https://azure.microsoft.com/support/legal/sla/virtual-machines)</span><span class="sxs-lookup"><span data-stu-id="82c4c-135">For more information on Microsoft Azure Service Level Agreement for Virtual Machines [SLA For Virtual Machines](https://azure.microsoft.com/support/legal/sla/virtual-machines)</span></span>

### <a name="scalability"></a><span data-ttu-id="82c4c-136">Масштабируемость</span><span class="sxs-lookup"><span data-stu-id="82c4c-136">Scalability</span></span>

<span data-ttu-id="82c4c-137">Общие рекомендации по разработке масштабируемых решений см. в разделе [Контрольный список для обеспечения масштабируемости][scalability] в центре архитектуры Azure.</span><span class="sxs-lookup"><span data-stu-id="82c4c-137">For general guidance on designing scalable solutions, see the [scalability checklist][scalability] in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="82c4c-138">Безопасность</span><span class="sxs-lookup"><span data-stu-id="82c4c-138">Security</span></span>

<span data-ttu-id="82c4c-139">Общие рекомендации по разработке безопасных решений см. в разделе [Azure Security Documentation][security].</span><span class="sxs-lookup"><span data-stu-id="82c4c-139">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="82c4c-140">Устойчивость</span><span class="sxs-lookup"><span data-stu-id="82c4c-140">Resiliency</span></span>

<span data-ttu-id="82c4c-141">Общее руководство по проектированию устойчивых решений см. в разделе [Проектирование устойчивых приложений для Azure][resiliency].</span><span class="sxs-lookup"><span data-stu-id="82c4c-141">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="pricing"></a><span data-ttu-id="82c4c-142">Цены</span><span class="sxs-lookup"><span data-stu-id="82c4c-142">Pricing</span></span>

<span data-ttu-id="82c4c-143">Чтобы помочь вам изучить стоимость запуска этого сценария, все службы были предварительно сконфигурированы в приведенных ниже примерах калькулятора стоимости.</span><span class="sxs-lookup"><span data-stu-id="82c4c-143">To help you explore the cost of running this scenario, all of the services are pre-configured in the cost calculator examples below.</span></span> <span data-ttu-id="82c4c-144">Чтобы узнать, как изменится цена для вашего конкретного варианта использования, измените соответствующие переменные в соответствии с ожидаемым трафиком.</span><span class="sxs-lookup"><span data-stu-id="82c4c-144">To see how the pricing would change for your particular use case, change the appropriate variables to match your expected traffic.</span></span>

<span data-ttu-id="82c4c-145">Здесь предоставлено четыре примера профиля затрат в зависимости от объема трафика, который планируется принимать.</span><span class="sxs-lookup"><span data-stu-id="82c4c-145">We have provided four sample cost profiles based on amount of traffic you expect to receive:</span></span>

|<span data-ttu-id="82c4c-146">Размер</span><span class="sxs-lookup"><span data-stu-id="82c4c-146">Size</span></span>|<span data-ttu-id="82c4c-147">Протоколы SAP</span><span class="sxs-lookup"><span data-stu-id="82c4c-147">SAPs</span></span>|<span data-ttu-id="82c4c-148">Тип виртуальной машины</span><span class="sxs-lookup"><span data-stu-id="82c4c-148">VM Type</span></span>|<span data-ttu-id="82c4c-149">Хранилище</span><span class="sxs-lookup"><span data-stu-id="82c4c-149">Storage</span></span>|<span data-ttu-id="82c4c-150">Калькулятор стоимости Azure</span><span class="sxs-lookup"><span data-stu-id="82c4c-150">Azure Pricing Calculator</span></span>|
|----|----|-------|-------|---------------|
|<span data-ttu-id="82c4c-151">Малый</span><span class="sxs-lookup"><span data-stu-id="82c4c-151">Small</span></span>|<span data-ttu-id="82c4c-152">8000</span><span class="sxs-lookup"><span data-stu-id="82c4c-152">8000</span></span>|<span data-ttu-id="82c4c-153">D8s_v3</span><span class="sxs-lookup"><span data-stu-id="82c4c-153">D8s_v3</span></span>|<span data-ttu-id="82c4c-154">2xP20, 1xP10</span><span class="sxs-lookup"><span data-stu-id="82c4c-154">2xP20, 1xP10</span></span>|[<span data-ttu-id="82c4c-155">Малый</span><span class="sxs-lookup"><span data-stu-id="82c4c-155">Small</span></span>](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1)|
|<span data-ttu-id="82c4c-156">Средний</span><span class="sxs-lookup"><span data-stu-id="82c4c-156">Medium</span></span>|<span data-ttu-id="82c4c-157">16000</span><span class="sxs-lookup"><span data-stu-id="82c4c-157">16000</span></span>|<span data-ttu-id="82c4c-158">D16s_v3</span><span class="sxs-lookup"><span data-stu-id="82c4c-158">D16s_v3</span></span>|<span data-ttu-id="82c4c-159">3xP20, 1xP10</span><span class="sxs-lookup"><span data-stu-id="82c4c-159">3xP20, 1xP10</span></span>|[<span data-ttu-id="82c4c-160">Средний</span><span class="sxs-lookup"><span data-stu-id="82c4c-160">Medium</span></span>](https://azure.com/e/465bd07047d148baab032b2f461550cd)|
<span data-ttu-id="82c4c-161">большой</span><span class="sxs-lookup"><span data-stu-id="82c4c-161">Large</span></span>|<span data-ttu-id="82c4c-162">32000</span><span class="sxs-lookup"><span data-stu-id="82c4c-162">32000</span></span>|<span data-ttu-id="82c4c-163">E32s_v3</span><span class="sxs-lookup"><span data-stu-id="82c4c-163">E32s_v3</span></span>|<span data-ttu-id="82c4c-164">3xP20, 1xP10</span><span class="sxs-lookup"><span data-stu-id="82c4c-164">3xP20, 1xP10</span></span>|[<span data-ttu-id="82c4c-165">Крупный</span><span class="sxs-lookup"><span data-stu-id="82c4c-165">Large</span></span>](https://azure.com/e/ada2e849d68b41c3839cc976000c6931)|
<span data-ttu-id="82c4c-166">Очень крупный</span><span class="sxs-lookup"><span data-stu-id="82c4c-166">Extra Large</span></span>|<span data-ttu-id="82c4c-167">64000</span><span class="sxs-lookup"><span data-stu-id="82c4c-167">64000</span></span>|<span data-ttu-id="82c4c-168">M64s</span><span class="sxs-lookup"><span data-stu-id="82c4c-168">M64s</span></span>|<span data-ttu-id="82c4c-169">4xP20, 1xP10</span><span class="sxs-lookup"><span data-stu-id="82c4c-169">4xP20, 1xP10</span></span>|[<span data-ttu-id="82c4c-170">Очень крупный</span><span class="sxs-lookup"><span data-stu-id="82c4c-170">Extra Large</span></span>](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef)|

> [!NOTE]
> <span data-ttu-id="82c4c-171">Эти цены приведены в качестве руководства. В них указаны только затраты на виртуальные машины и хранилище.</span><span class="sxs-lookup"><span data-stu-id="82c4c-171">This pricing is a guide that only indicates the VMs and storage costs.</span></span> <span data-ttu-id="82c4c-172">Они не включают плату за сеть, хранилище резервных копий и передачу исходящих и входящих данных.</span><span class="sxs-lookup"><span data-stu-id="82c4c-172">It excludes networking, backup storage, and data ingress/egress charges.</span></span>

- <span data-ttu-id="82c4c-173">[Малый](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1). Небольшая система с виртуальной машиной типа D8s_v3 с восемью виртуальными ЦП, 32 ГБ оперативной памяти и 200 ГБ временного хранилища, а также дополнительными дисками хранения — двумя на 512 ГБ и одним на 128 ГБ класса "Премиум".</span><span class="sxs-lookup"><span data-stu-id="82c4c-173">[Small](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1): A small system consists of VM type D8s_v3 with 8x vCPUs, 32 GB RAM and 200 GB temp storage, additionally two 512 GB and one 128 GB premium storage disks.</span></span>
- <span data-ttu-id="82c4c-174">[Средний](https://azure.com/e/465bd07047d148baab032b2f461550cd). Средняя по производительности система с виртуальной машиной типа D16s_v3 с 16 виртуальными ЦП, 64 ГБ оперативной памяти и 400 ГБ временного хранилища, а также дополнительными дисками хранения — тремя на 512 ГБ и одним на 128 ГБ класса "Премиум".</span><span class="sxs-lookup"><span data-stu-id="82c4c-174">[Medium](https://azure.com/e/465bd07047d148baab032b2f461550cd): A medium system consists of VM type D16s_v3 with 16x vCPUs, 64 GB RAM and 400 GB temp storage, additionally three 512 GB and one 128 GB premium storage disks.</span></span>
- <span data-ttu-id="82c4c-175">[Большой](https://azure.com/e/ada2e849d68b41c3839cc976000c6931). Высокопроизводительная система с виртуальной машиной типа E32s_v3 с 32 виртуальными ЦП, 256 ГБ оперативной памяти и 512 ГБ временного хранилища, а также дополнительными дисками хранения — тремя на 512 ГБ и одним на 128 ГБ класса "Премиум".</span><span class="sxs-lookup"><span data-stu-id="82c4c-175">[Large](https://azure.com/e/ada2e849d68b41c3839cc976000c6931): A large system consists of VM type E32s_v3 with 32x vCPUs, 256 GB RAM and 512 GB temp storage, additionally three 512GB and one 128GB premium storage disks.</span></span>
- <span data-ttu-id="82c4c-176">[Сверхбольшой](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef). Исключительно высокопроизводительная система с виртуальной машиной типа M64s с 64 виртуальными ЦП, 1024 ГБ оперативной памяти и 2000 ГБ временного хранилища, а также дополнительными дисками хранения — четырьмя на 512 ГБ и одним на 128 ГБ класса "Премиум".</span><span class="sxs-lookup"><span data-stu-id="82c4c-176">[Extra Large](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef): An extra large system consists of a VM type M64s with 64x vCPUs, 1024 GB RAM and 2000 GB temp storage, additionally four 512 GB and one 128 GB premium storage disks.</span></span>

## <a name="deployment"></a><span data-ttu-id="82c4c-177">Развертывание</span><span class="sxs-lookup"><span data-stu-id="82c4c-177">Deployment</span></span>

<span data-ttu-id="82c4c-178">Щелкните здесь, чтобы развернуть базовую инфраструктуру для этого сценария.</span><span class="sxs-lookup"><span data-stu-id="82c4c-178">Click here to deploy the underlying infrastructure for this scenario.</span></span>

<!-- markdownlint-disable MD033 -->

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fsap-2tier%2Fazuredeploy.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>

<!-- markdownlint-enable MD033 -->

> [!NOTE]
> <span data-ttu-id="82c4c-179">В процессе этого развертывания SAP и Oracle не устанавливаются.</span><span class="sxs-lookup"><span data-stu-id="82c4c-179">SAP and Oracle are not installed during this deployment.</span></span> <span data-ttu-id="82c4c-180">Эти компоненты нужно будет развертывать отдельно.</span><span class="sxs-lookup"><span data-stu-id="82c4c-180">You will need to deploy these components separately.</span></span>

<!-- links -->
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[sap-netweaver]: /azure/architecture/reference-architectures/sap/sap-netweaver
[sap-hana]: /azure/architecture/reference-architectures/sap/sap-s4hana
[sap-large]: /azure/architecture/reference-architectures/sap/hana-large-instances
[hub-spoke]: /azure/architecture/reference-architectures/hybrid-networking/hub-spoke
