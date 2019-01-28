---
title: Реализация звездообразной топологии сети
titleSuffix: Azure Reference Architectures
description: Реализация звездообразной топологии сети с помощью общих служб в Azure.
author: telmosampaio
ms.date: 10/09/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18, networking
ms.openlocfilehash: dd7632c3a84f6a0cee5d8b35e6a943ab8c52caf8
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54488313"
---
# <a name="implement-a-hub-spoke-network-topology-with-shared-services-in-azure"></a><span data-ttu-id="5d7c6-103">Реализация звездообразной топологии сети с помощью общих служб в Azure</span><span class="sxs-lookup"><span data-stu-id="5d7c6-103">Implement a hub-spoke network topology with shared services in Azure</span></span>

<span data-ttu-id="5d7c6-104">Эта эталонная архитектура создана на основе [звездообразной][guidance-hub-spoke] эталонной архитектуры. Это позволяет включить в концентраторе общие службы, которые можно использовать во всех периферийных зонах.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-104">This reference architecture builds on the [hub-spoke][guidance-hub-spoke] reference architecture to include shared services in the hub that can be consumed by all spokes.</span></span> <span data-ttu-id="5d7c6-105">Чтобы приступить к перемещению центра обработки данных в облако и создать [виртуальный центр обработки данных], необходимо предоставить общий доступ к службам удостоверений и безопасности.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-105">As a first step toward migrating a datacenter to the cloud, and building a [virtual datacenter], the first services you need to share are identity and security.</span></span> <span data-ttu-id="5d7c6-106">Эта эталонная архитектура содержит сведения о том, как расширить службы Active Directory из локального центра обработки данных в Azure и как добавить виртуальные сетевые модули (NVA), которые могут действовать как брандмауэры, в звездообразную топологию.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-106">This reference architecture shows you how to extend your Active Directory services from your on-premises datacenter to Azure, and how to add a network virtual appliance (NVA) that can act as a firewall, in a hub-spoke topology.</span></span>  <span data-ttu-id="5d7c6-107">[**Разверните это решение**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="5d7c6-107">[**Deploy this solution**](#deploy-the-solution).</span></span>

> [!NOTE]
> <span data-ttu-id="5d7c6-108">Этот сценарий также может выполняться с помощью [Брандмауэра Azure](/azure/firewall/) — облачной службы безопасности сети.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-108">This scenario can also be accomplished using [Azure Firewall](/azure/firewall/), a cloud-based network security service.</span></span>

![Топология общих служб в Azure](./images/shared-services.png)

<span data-ttu-id="5d7c6-110">*Скачайте [файл Visio][visio-download] этой архитектуры*</span><span class="sxs-lookup"><span data-stu-id="5d7c6-110">*Download a [Visio file][visio-download] of this architecture*</span></span>

<span data-ttu-id="5d7c6-111">До преимуществ этой топологии можно отнести:</span><span class="sxs-lookup"><span data-stu-id="5d7c6-111">The benefits of this topology include:</span></span>

- <span data-ttu-id="5d7c6-112">**Сокращение затрат** путем централизации служб, которые могут совместно использоваться несколькими рабочими нагрузками, такими как сетевые виртуальные устройства (NVA) и DNS-серверы в одном расположении.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-112">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
- <span data-ttu-id="5d7c6-113">**Превышение лимитов подписки** путем пиринга виртуальных сетей из разных подписок к центральному концентратору.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-113">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
- <span data-ttu-id="5d7c6-114">**Разделение областей ответственности** между центральным ИТ-отделом (SecOps, InfraOps) и рабочими нагрузками (DevOps).</span><span class="sxs-lookup"><span data-stu-id="5d7c6-114">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="5d7c6-115">Типичные способы использования этой архитектуры:</span><span class="sxs-lookup"><span data-stu-id="5d7c6-115">Typical uses for this architecture include:</span></span>

- <span data-ttu-id="5d7c6-116">Рабочая нагрузка, развернутая в разных средах, например в среде для разработки, тестирования и в рабочей среде, для которых требуются общие службы, например DNS, IDS, NTP или AD DS.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-116">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="5d7c6-117">Общие службы размещаются в виртуальной сети концентратора, в то время как каждая среда развертывается в периферийной зоне для обеспечения изоляции.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-117">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
- <span data-ttu-id="5d7c6-118">Рабочие нагрузки, которые не требуют подключения друг к другу, но требуют доступа к общим службам.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-118">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
- <span data-ttu-id="5d7c6-119">Предприятия, которые требуют централизованного контроля над аспектами безопасности, такими как брандмауэр в концентраторе, таком как сеть периметра, и отдельного управления для рабочих нагрузок в каждой периферийной зоне.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-119">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="5d7c6-120">Архитектура</span><span class="sxs-lookup"><span data-stu-id="5d7c6-120">Architecture</span></span>

<span data-ttu-id="5d7c6-121">Архитектура состоит из следующих компонентов:</span><span class="sxs-lookup"><span data-stu-id="5d7c6-121">The architecture consists of the following components.</span></span>

- <span data-ttu-id="5d7c6-122">**Локальная сеть.**</span><span class="sxs-lookup"><span data-stu-id="5d7c6-122">**On-premises network**.</span></span> <span data-ttu-id="5d7c6-123">Частная локальная сеть, работающая внутри организации.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-123">A private local-area network running within an organization.</span></span>

- <span data-ttu-id="5d7c6-124">**VPN-устройство**.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-124">**VPN device**.</span></span> <span data-ttu-id="5d7c6-125">Устройство или служба, предоставляющая возможность внешнего подключения к локальной сети.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-125">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="5d7c6-126">VPN-устройство может быть аппаратным устройством или программным решением, таким как служба маршрутизации и удаленного доступа (RRAS) в Windows Server 2012.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-126">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="5d7c6-127">Список поддерживаемых VPN-устройств и информацию о настройке выбранных VPN-устройств для подключения к Azure см. в статье [VPN-устройства и параметры IPsec/IKE для подключений типа "сеть — сеть" через VPN-шлюз][vpn-appliance].</span><span class="sxs-lookup"><span data-stu-id="5d7c6-127">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

- <span data-ttu-id="5d7c6-128">**Сетевой шлюз виртуальной сети VPN или шлюз ExpressRoute**.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-128">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="5d7c6-129">Шлюз виртуальной сети позволяет виртуальной сети подключаться к VPN-устройству или к каналу ExpressRoute, которые используются для подключения к вашей локальной сети.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-129">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="5d7c6-130">Дополнительные сведения см. в статье [Подключение локальной сети к виртуальной сети Microsoft Azure][connect-to-an-Azure-vnet].</span><span class="sxs-lookup"><span data-stu-id="5d7c6-130">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="5d7c6-131">В сценариях развертывания для этой эталонной архитектуры используется VPN-шлюз для подключения, а виртуальная сеть в Azure используется для имитации вашей локальной сети.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-131">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

- <span data-ttu-id="5d7c6-132">**Виртуальная сеть концентратора**.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-132">**Hub VNet**.</span></span> <span data-ttu-id="5d7c6-133">Виртуальная сеть Azure используется в качестве концентратора в звездообразной топологии.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-133">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="5d7c6-134">Концентратор является центральной точкой подключения к вашей локальной сети и местом для размещения служб, которые могут использоваться различными рабочими нагрузками, размещенными в виртуальных сетях.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-134">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

- <span data-ttu-id="5d7c6-135">**Подсеть шлюза**.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-135">**Gateway subnet**.</span></span> <span data-ttu-id="5d7c6-136">Шлюзы виртуальных сетей хранятся в одной подсети.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-136">The virtual network gateways are held in the same subnet.</span></span>

- <span data-ttu-id="5d7c6-137">**Подсеть общих служб**.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-137">**Shared services subnet**.</span></span> <span data-ttu-id="5d7c6-138">Подсеть в концентраторе виртуальной сети используется для размещения служб, которые могут использоваться совместно во всех периферийных зонах, например DNS или AD DS.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-138">A subnet in the hub VNet used to host services that can be shared among all spokes, such as DNS or AD DS.</span></span>

- <span data-ttu-id="5d7c6-139">**Подсеть сети периметра.**</span><span class="sxs-lookup"><span data-stu-id="5d7c6-139">**DMZ subnet**.</span></span> <span data-ttu-id="5d7c6-140">Подсеть в виртуальной сети концентратора, используемая для размещения виртуальных сетевых модулей, которые могут действовать как устройства безопасности, например как брандмауэры.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-140">A subnet in the hub VNet used to host NVAs that can act as security appliances, such as firewalls.</span></span>

- <span data-ttu-id="5d7c6-141">**Виртуальные сети периферийных зон**.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-141">**Spoke VNets**.</span></span> <span data-ttu-id="5d7c6-142">Одна или несколько виртуальных сетей Azure, которые используются в качестве периферийных зон в звездообразной топологии.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-142">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="5d7c6-143">Периферийные зоны можно использовать для изоляции рабочих нагрузок в их виртуальных сетях, управляемых отдельно от других периферийных зон.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-143">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="5d7c6-144">Каждая рабочая нагрузка может содержать несколько уровней с несколькими подсетями, подключенными через подсистемы балансировки нагрузки Azure.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-144">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="5d7c6-145">Дополнительные сведения об инфраструктуре приложений см. в статьях [Запуск рабочих нагрузок на виртуальных машинах Windows][windows-vm-ra] и [Запуск рабочих нагрузок на виртуальной машине Linux][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="5d7c6-145">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

- <span data-ttu-id="5d7c6-146">**Пиринговая связь между виртуальными сетями**.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-146">**VNet peering**.</span></span> <span data-ttu-id="5d7c6-147">Две виртуальные сети в одном регионе Azure можно подключить с помощью [пиринга][vnet-peering].</span><span class="sxs-lookup"><span data-stu-id="5d7c6-147">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="5d7c6-148">Пиринговые подключения представляют собой нетранзитивные подключения между виртуальными подсетями с низкой задержкой.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-148">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="5d7c6-149">После установки пирингового подключения виртуальные сети обмениваются трафиком по магистрали Azure без необходимости использования маршрутизатора.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-149">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="5d7c6-150">В звездообразной топологии сети пиринг виртуальной сети используется для подключения концентратора к каждой периферийной зоне.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-150">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="5d7c6-151">В этой статье рассматриваются только [развертывания Resource Manager](/azure/azure-resource-manager/resource-group-overview), но вы также можете подключить классическую виртуальную сеть к виртуальной сети Resource Manager в той же подписке.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-151">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="5d7c6-152">Таким образом, ваши периферийные зоны могут размещать классические развертывания и по-прежнему пользоваться преимуществами общих служб в концентраторе.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-152">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>

## <a name="recommendations"></a><span data-ttu-id="5d7c6-153">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="5d7c6-153">Recommendations</span></span>

<span data-ttu-id="5d7c6-154">Все рекомендации для [звездообразной][guidance-hub-spoke] эталонной архитектуры также можно применить к эталонной архитектуре общих служб.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-154">All the recommendations for the [hub-spoke][guidance-hub-spoke] reference architecture also apply to the shared services reference architecture.</span></span>

<span data-ttu-id="5d7c6-155">Кроме того, следующие рекомендации применимы для большинства сценариев с общими службами.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-155">Also, the following recommendations apply for most scenarios under shared services.</span></span> <span data-ttu-id="5d7c6-156">Следуйте этим рекомендациям, если они не противоречат особым требованиям для вашего случая.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-156">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="identity"></a><span data-ttu-id="5d7c6-157">Удостоверение</span><span class="sxs-lookup"><span data-stu-id="5d7c6-157">Identity</span></span>

<span data-ttu-id="5d7c6-158">В локальном центре обработки данных большинства коммерческих организаций есть среда Active Directory Services (ADDS).</span><span class="sxs-lookup"><span data-stu-id="5d7c6-158">Most enterprise organizations have an Active Directory Directory Services (ADDS) environment in their on-premises datacenter.</span></span> <span data-ttu-id="5d7c6-159">Чтобы упростить управление ресурсами, перемещенными в Azure из локальной сети, которая зависит от ADDS, рекомендуется разместить контроллеры домена ADDS в Azure.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-159">To facilitate management of assets moved to Azure from your on-premises network that depend on ADDS, it is recommended to host ADDS domain controllers in Azure.</span></span>

<span data-ttu-id="5d7c6-160">Если вы используете объекты групповой политики, которыми вы хотите управлять отдельно для Azure и локальной среды, для каждого региона Azure необходимо использовать разные сайты AD.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-160">If you make use of Group Policy Objects, that you want to control separately for Azure and your on-premises environment, use a different AD site for each Azure region.</span></span> <span data-ttu-id="5d7c6-161">Разместите контроллеры домена в центральной виртуальной сети (концентраторе), доступ к которой могут получить зависимые рабочие нагрузки.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-161">Place your domain controllers in a central VNet (hub) that dependent workloads can access.</span></span>

### <a name="security"></a><span data-ttu-id="5d7c6-162">Безопасность</span><span class="sxs-lookup"><span data-stu-id="5d7c6-162">Security</span></span>

<span data-ttu-id="5d7c6-163">При перемещении рабочих нагрузок из локальной среды в Azure некоторые из них потребуется разместить на виртуальных машинах.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-163">As you move workloads from your on-premises environment to Azure, some of these workloads will require to be hosted in VMs.</span></span> <span data-ttu-id="5d7c6-164">В целях обеспечения соответствия может потребоваться наложить ограничения на передачу трафика этих рабочих нагрузок.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-164">For compliance reasons, you may need to enforce restrictions on traffic traversing those workloads.</span></span>

<span data-ttu-id="5d7c6-165">Вы можете использовать виртуальные сетевые модули в Azure для размещения разных типов служб безопасности и производительности.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-165">You can use network virtual appliances (NVAs) in Azure to host different types of security and performance services.</span></span> <span data-ttu-id="5d7c6-166">Если вы знакомы с заданным набором устройств в локальной среде, рекомендуется использовать те же виртуальные модули в Azure, где это применимо.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-166">If you are familiar with a given set of appliances on-premises today, it is recommended to use the same virtualized appliances in Azure, where applicable.</span></span>

> [!NOTE]
> <span data-ttu-id="5d7c6-167">В скриптах развертывания этой эталонной архитектуры используется виртуальная машина Ubuntu с IP-пересылкой, которая позволяет сымитировать виртуальный сетевой модуль.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-167">The deployment scripts for this reference architecture use an Ubuntu VM with IP forwarding enabled to mimic a network virtual appliance.</span></span>

## <a name="considerations"></a><span data-ttu-id="5d7c6-168">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="5d7c6-168">Considerations</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="5d7c6-169">Преодоление ограничения пиринговых подключений виртуальной сети</span><span class="sxs-lookup"><span data-stu-id="5d7c6-169">Overcoming VNet peering limits</span></span>

<span data-ttu-id="5d7c6-170">Убедитесь, что вы учли [ограничение числа пиринговых подключений виртуальных сетей на одну виртуальную сеть][vnet-peering-limit] в Azure.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-170">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="5d7c6-171">Если вам нужно превысить лимит по периферийным зонам, подумайте о создании вложенной звездообразной топологии, где первый уровень периферийных зон также выступает в качестве концентраторов.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-171">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="5d7c6-172">Этот подход показан на схеме ниже.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-172">The following diagram shows this approach.</span></span>

![Вложенная звездообразная топология в Azure](./images/hub-spokehub-spoke.svg)

<span data-ttu-id="5d7c6-174">Кроме того, следует учитывать, какие службы используются совместно в концентраторе, чтобы убедиться, что концентратор масштабируется в соответствии с большим количеством периферийных зон.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-174">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="5d7c6-175">Например, если ваш концентратор предоставляет службы брандмауэра, рассмотрите возможность ограничения пропускной способности решения брандмауэра при добавлении нескольких периферийных зон.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-175">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="5d7c6-176">Возможно, вам захочется перенести некоторые из этих общих служб на второй уровень концентраторов.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-176">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="5d7c6-177">Развертывание решения</span><span class="sxs-lookup"><span data-stu-id="5d7c6-177">Deploy the solution</span></span>

<span data-ttu-id="5d7c6-178">Пример развертывания для этой архитектуры можно найти на портале [GitHub][ref-arch-repo].</span><span class="sxs-lookup"><span data-stu-id="5d7c6-178">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="5d7c6-179">Развертывание создает в вашей подписке следующие группы ресурсов:</span><span class="sxs-lookup"><span data-stu-id="5d7c6-179">The deployment creates the following resource groups in your subscription:</span></span>

- <span data-ttu-id="5d7c6-180">hub-adds-rg</span><span class="sxs-lookup"><span data-stu-id="5d7c6-180">hub-adds-rg</span></span>
- <span data-ttu-id="5d7c6-181">hub-nva-rg</span><span class="sxs-lookup"><span data-stu-id="5d7c6-181">hub-nva-rg</span></span>
- <span data-ttu-id="5d7c6-182">hub-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="5d7c6-182">hub-vnet-rg</span></span>
- <span data-ttu-id="5d7c6-183">onprem-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="5d7c6-183">onprem-vnet-rg</span></span>
- <span data-ttu-id="5d7c6-184">spoke1-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="5d7c6-184">spoke1-vnet-rg</span></span>
- <span data-ttu-id="5d7c6-185">spoke2-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="5d7c6-185">spoke2-vnet-rg</span></span>

<span data-ttu-id="5d7c6-186">Файлы параметров шаблона ссылаются на эти имена, поэтому, если вы их изменяете, соответствующим образом обновите файлы параметров.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-186">The template parameter files refer to these names, so if you change them, update the parameter files to match.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="5d7c6-187">Предварительные требования</span><span class="sxs-lookup"><span data-stu-id="5d7c6-187">Prerequisites</span></span>

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-simulated-on-premises-datacenter-using-azbb"></a><span data-ttu-id="5d7c6-188">Развертывание имитации локального центра обработки данных с помощью azbb</span><span class="sxs-lookup"><span data-stu-id="5d7c6-188">Deploy the simulated on-premises datacenter using azbb</span></span>

<span data-ttu-id="5d7c6-189">Чтобы развернуть имитацию локального центра обработки данных в виртуальной сети Azure, сделайте следующее:</span><span class="sxs-lookup"><span data-stu-id="5d7c6-189">This step deploys the simulated on-premises datacenter as an Azure VNet.</span></span>

1. <span data-ttu-id="5d7c6-190">Перейдите в папку `hybrid-networking\shared-services-stack\` в репозитории GitHub.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-190">Navigate to the `hybrid-networking\shared-services-stack\` folder of the GitHub repository.</span></span>

2. <span data-ttu-id="5d7c6-191">Откройте файл `onprem.json` .</span><span class="sxs-lookup"><span data-stu-id="5d7c6-191">Open the `onprem.json` file.</span></span>

3. <span data-ttu-id="5d7c6-192">Найдите все экземпляры `UserName`, `adminUserName`,`Password` и `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-192">Search for all instances of `UserName`, `adminUserName`,`Password`, and `adminPassword`.</span></span> <span data-ttu-id="5d7c6-193">Введите значения для имени пользователя и пароля в параметрах и сохраните файл.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-193">Enter values for the user name and password in the parameters and save the file.</span></span>

4. <span data-ttu-id="5d7c6-194">Выполните следующую команду:</span><span class="sxs-lookup"><span data-stu-id="5d7c6-194">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g onprem-vnet-rg -l <location> -p onprem.json --deploy
   ```

5. <span data-ttu-id="5d7c6-195">Дождитесь завершения развертывания.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-195">Wait for the deployment to finish.</span></span> <span data-ttu-id="5d7c6-196">При развертывании создается виртуальная сеть, виртуальная машина под управлением Windows и VPN-шлюз.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-196">This deployment creates a virtual network, a virtual machine running Windows, and a VPN gateway.</span></span> <span data-ttu-id="5d7c6-197">Создание VPN-шлюза может занять более 40 минут.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-197">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="deploy-the-hub-vnet"></a><span data-ttu-id="5d7c6-198">Развертывание концентратора виртуальной сети</span><span class="sxs-lookup"><span data-stu-id="5d7c6-198">Deploy the hub VNet</span></span>

<span data-ttu-id="5d7c6-199">Чтобы развернуть концентратор виртуальной сети и подключить его к имитации локальной виртуальной сети, сделайте следующее:</span><span class="sxs-lookup"><span data-stu-id="5d7c6-199">This step deploys the hub VNet and connects it to the simulated on-premises VNet.</span></span>

1. <span data-ttu-id="5d7c6-200">Откройте файл `hub-vnet.json` .</span><span class="sxs-lookup"><span data-stu-id="5d7c6-200">Open the `hub-vnet.json` file.</span></span>

2. <span data-ttu-id="5d7c6-201">Найдите `adminPassword` и введите имя пользователя и пароль в параметрах.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-201">Search for `adminPassword` and enter a user name and password in the parameters.</span></span>

3. <span data-ttu-id="5d7c6-202">Найдите все экземпляры `sharedKey` и введите значение для общего ключа.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-202">Search for all instances of `sharedKey` and enter a value for a shared key.</span></span> <span data-ttu-id="5d7c6-203">Сохраните файл.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-203">Save the file.</span></span>

   ```json
   "sharedKey": "abc123",
   ```

4. <span data-ttu-id="5d7c6-204">Выполните следующую команду:</span><span class="sxs-lookup"><span data-stu-id="5d7c6-204">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet.json --deploy
   ```

5. <span data-ttu-id="5d7c6-205">Дождитесь завершения развертывания.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-205">Wait for the deployment to finish.</span></span> <span data-ttu-id="5d7c6-206">В ходе этого развертывания создается виртуальная сеть, виртуальная машина, VPN-шлюз и подключение к шлюзу, созданному в предыдущем разделе.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-206">This deployment creates a virtual network, a virtual machine, a VPN gateway, and a connection to the gateway created in the previous section.</span></span> <span data-ttu-id="5d7c6-207">Создание VPN-шлюза может занять более 40 минут.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-207">The VPN gateway can take more than 40 minutes to complete.</span></span>

### <a name="deploy-ad-ds-in-azure"></a><span data-ttu-id="5d7c6-208">Развертывание AD DS в Azure</span><span class="sxs-lookup"><span data-stu-id="5d7c6-208">Deploy AD DS in Azure</span></span>

<span data-ttu-id="5d7c6-209">Чтобы развернуть контроллеры AD DS в Azure, сделайте следующее:</span><span class="sxs-lookup"><span data-stu-id="5d7c6-209">This step deploys AD DS domain controllers in Azure.</span></span>

1. <span data-ttu-id="5d7c6-210">Откройте файл `hub-adds.json` .</span><span class="sxs-lookup"><span data-stu-id="5d7c6-210">Open the `hub-adds.json` file.</span></span>

2. <span data-ttu-id="5d7c6-211">Найдите все экземпляры `Password` и `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-211">Search for all instances of `Password` and `adminPassword`.</span></span> <span data-ttu-id="5d7c6-212">Введите значения для имени пользователя и пароля в параметрах и сохраните файл.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-212">Enter values for the user name and password in the parameters and save the file.</span></span>

3. <span data-ttu-id="5d7c6-213">Выполните следующую команду:</span><span class="sxs-lookup"><span data-stu-id="5d7c6-213">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-adds-rg -l <location> -p hub-adds.json --deploy
   ```

<span data-ttu-id="5d7c6-214">Этот этап развертывания может занять несколько минут, так как для него требуется присоединение двух виртуальных машин к домену, размещенному в имитации локального центра обработки данных, с последующей установкой на них AD DS.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-214">This deployment step may take several minutes, because it joins the two VMs to the domain hosted in the simulated on-premises datacenter, and installs AD DS on them.</span></span>

### <a name="deploy-the-spoke-vnets"></a><span data-ttu-id="5d7c6-215">Развертывание виртуальных сетей периферийных зон</span><span class="sxs-lookup"><span data-stu-id="5d7c6-215">Deploy the spoke VNets</span></span>

<span data-ttu-id="5d7c6-216">На этом этапе выполняется развертывание виртуальных сетей периферийных зон.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-216">This step deploys the spoke VNets.</span></span>

1. <span data-ttu-id="5d7c6-217">Откройте файл `spoke1.json` .</span><span class="sxs-lookup"><span data-stu-id="5d7c6-217">Open the `spoke1.json` file.</span></span>

2. <span data-ttu-id="5d7c6-218">Найдите `adminPassword` и введите имя пользователя и пароль в параметрах.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-218">Search for `adminPassword` and enter a user name and password in the parameters.</span></span>

3. <span data-ttu-id="5d7c6-219">Выполните следующую команду:</span><span class="sxs-lookup"><span data-stu-id="5d7c6-219">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg -l <location> -p spoke1.json --deploy
   ```

4. <span data-ttu-id="5d7c6-220">Повторите шаги 1–2 для файла `spoke2.json`.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-220">Repeat steps 1 and 2 for the file `spoke2.json`.</span></span>

5. <span data-ttu-id="5d7c6-221">Выполните следующую команду:</span><span class="sxs-lookup"><span data-stu-id="5d7c6-221">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg -l <location> -p spoke2.json --deploy
   ```

### <a name="peer-the-hub-vnet-to-the-spoke-vnets"></a><span data-ttu-id="5d7c6-222">Пиринговое подключение виртуальной сети концентратора к виртуальным сетям периферийных зон</span><span class="sxs-lookup"><span data-stu-id="5d7c6-222">Peer the hub VNet to the spoke VNets</span></span>

<span data-ttu-id="5d7c6-223">Чтобы создать пиринговое подключение между виртуальной сетью концентратора и виртуальными сетями периферийных зон, выполните следующую команду:</span><span class="sxs-lookup"><span data-stu-id="5d7c6-223">To create a peering connection from the hub VNet to the spoke VNets, run the following command:</span></span>

```bash
azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet-peering.json --deploy
```

### <a name="deploy-the-nva"></a><span data-ttu-id="5d7c6-224">Развертывание виртуального сетевого модуля</span><span class="sxs-lookup"><span data-stu-id="5d7c6-224">Deploy the NVA</span></span>

<span data-ttu-id="5d7c6-225">На этом этапе выполняется развертывание виртуального сетевого модуля в подсети `dmz`.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-225">This step deploys an NVA in the `dmz` subnet.</span></span>

1. <span data-ttu-id="5d7c6-226">Откройте файл `hub-nva.json` .</span><span class="sxs-lookup"><span data-stu-id="5d7c6-226">Open the `hub-nva.json` file.</span></span>

2. <span data-ttu-id="5d7c6-227">Найдите `adminPassword` и введите имя пользователя и пароль в параметрах.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-227">Search for `adminPassword` and enter a user name and password in the parameters.</span></span>

3. <span data-ttu-id="5d7c6-228">Выполните следующую команду:</span><span class="sxs-lookup"><span data-stu-id="5d7c6-228">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-nva-rg -l <location> -p hub-nva.json --deploy
   ```

### <a name="test-connectivity"></a><span data-ttu-id="5d7c6-229">Проверка подключения</span><span class="sxs-lookup"><span data-stu-id="5d7c6-229">Test connectivity</span></span>

<span data-ttu-id="5d7c6-230">Протестируйте подключение от моделируемой локальной среды к концентратору виртуальной сети.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-230">Test conectivity from the simulated on-premises environment to the hub VNet.</span></span>

1. <span data-ttu-id="5d7c6-231">Используйте портал Azure для поиска в группе ресурсов `onprem-jb-rg` виртуальной машины с именем `jb-vm1`.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-231">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="5d7c6-232">Нажмите `Connect`, чтобы открыть сеанс удаленного рабочего стола для виртуальной машины.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-232">Click `Connect` to open a remote desktop session to the VM.</span></span> <span data-ttu-id="5d7c6-233">Используйте пароль, указанный в файле параметров `onprem.json`.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-233">Use the password that you specified in the `onprem.json` parameter file.</span></span>

3. <span data-ttu-id="5d7c6-234">Откройте консоль PowerShell в виртуальной машине и используйте командлет `Test-NetConnection`, чтобы убедиться, что вы можете подключиться к виртуальной машине Jumpbox в виртуальной сети концентратора.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-234">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the jumpbox VM in the hub VNet.</span></span>

   ```powershell
   Test-NetConnection 10.0.0.68 -CommonTCPPort RDP
   ```

<span data-ttu-id="5d7c6-235">Результат должен выглядеть следующим образом.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-235">The output should look similar to the following:</span></span>

```powershell
ComputerName     : 10.0.0.68
RemoteAddress    : 10.0.0.68
RemotePort       : 3389
InterfaceAlias   : Ethernet 2
SourceAddress    : 192.168.1.000
TcpTestSucceeded : True
```

> [!NOTE]
> <span data-ttu-id="5d7c6-236">По умолчанию виртуальные машины Windows Server не разрешают трафик ICMP в Azure.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-236">By default, Windows Server VMs do not allow ICMP responses in Azure.</span></span> <span data-ttu-id="5d7c6-237">Если вы хотите использовать команду `ping` для проверки возможности подключения, включите трафик ICMP в расширенном брандмауэре Windows для каждой виртуальной машины.</span><span class="sxs-lookup"><span data-stu-id="5d7c6-237">If you want to use `ping` to test connectivity, you need to enable ICMP traffic in the Windows Advanced Firewall for each VM.</span></span>

<span data-ttu-id="5d7c6-238">Повторите эти же шаги, чтобы проверить возможность подключения к виртуальным сетям периферийных зон:</span><span class="sxs-lookup"><span data-stu-id="5d7c6-238">Repeat the sames steps to test connectivity to the spoke VNets:</span></span>

```powershell
Test-NetConnection 10.1.0.68 -CommonTCPPort RDP
Test-NetConnection 10.2.0.68 -CommonTCPPort RDP
```

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[guidance-hub-spoke]: ./hub-spoke.md
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[best-practices-security]: /azure/best-practices-network-securit
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[linux-vm-ra]: ../virtual-machines-linux/index.md
[hybrid-ha]: ./expressroute-vpn-failover.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[Виртуальный центр обработки данных]: https://aka.ms/vdc
[virtual datacenter]: https://aka.ms/vdc
[vnet-peering]: /azure/virtual-network/virtual-network-peering-overview
[vnet-peering-limit]: /azure/azure-subscription-service-limits#networking-limits
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
