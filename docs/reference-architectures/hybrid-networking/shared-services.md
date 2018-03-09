---
title: "Реализация звездообразной топологии сети с помощью общих служб в Azure"
description: "Сведения о реализации звездообразной топологии сети с помощью общих служб в Azure."
author: telmosampaio
ms.date: 02/25/2018
pnp.series.title: Implement a hub-spoke network topology with shared services in Azure
pnp.series.prev: hub-spoke
ms.openlocfilehash: c0fb1d1ddd7c70ed914d58e7c73b10475b91aedf
ms.sourcegitcommit: 2123c25b1a0b5501ff1887f98030787191cf6994
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/08/2018
---
# <a name="implement-a-hub-spoke-network-topology-with-shared-services-in-azure"></a><span data-ttu-id="5a792-103">Реализация звездообразной топологии сети с помощью общих служб в Azure</span><span class="sxs-lookup"><span data-stu-id="5a792-103">Implement a hub-spoke network topology with shared services in Azure</span></span>

<span data-ttu-id="5a792-104">Эта эталонная архитектура создана на основе [звездообразной][guidance-hub-spoke] эталонной архитектуры. Это позволяет включить в концентраторе общие службы, которые можно использовать во всех периферийных зонах.</span><span class="sxs-lookup"><span data-stu-id="5a792-104">This reference architecture builds on the [hub-spoke][guidance-hub-spoke] reference architecture to include shared services in the hub that can be consumed by all spokes.</span></span> <span data-ttu-id="5a792-105">Чтобы приступить к перемещению центра обработки данных в облако и создать [виртуальный центр обработки данных], необходимо предоставить общий доступ к службам удостоверений и безопасности.</span><span class="sxs-lookup"><span data-stu-id="5a792-105">As a first step toward migrating a datacenter to the cloud, and building a [virtual datacenter], the first services you need to share are identity and security.</span></span> <span data-ttu-id="5a792-106">Эта эталонная архитектура содержит сведения о том, как расширить службы Active Directory из локального центра обработки данных в Azure и как добавить виртуальные сетевые модули (NVA), которые могут действовать как брандмауэры, в звездообразную топологию.</span><span class="sxs-lookup"><span data-stu-id="5a792-106">This reference archiecture shows you how to extend your Active Directory services from your on-premises datacenter to Azure, and how to add a network virtual appliance (NVA) that can act as a firewall, in a hub-spoke topology.</span></span>  <span data-ttu-id="5a792-107">[**Разверните это решение**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="5a792-107">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="5a792-108">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="5a792-108">![[0]][0]</span></span>

<span data-ttu-id="5a792-109">*Скачайте [файл Visio][visio-download] этой архитектуры*</span><span class="sxs-lookup"><span data-stu-id="5a792-109">*Download a [Visio file][visio-download] of this architecture*</span></span>

<span data-ttu-id="5a792-110">Преимущества этой топологии:</span><span class="sxs-lookup"><span data-stu-id="5a792-110">The benefits of this toplogy include:</span></span>

* <span data-ttu-id="5a792-111">**Сокращение затрат** путем централизации служб, которые могут совместно использоваться несколькими рабочими нагрузками, такими как сетевые виртуальные устройства (NVA) и DNS-серверы в одном расположении.</span><span class="sxs-lookup"><span data-stu-id="5a792-111">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
* <span data-ttu-id="5a792-112">**Превышение лимитов подписки** путем пиринга виртуальных сетей из разных подписок к центральному концентратору.</span><span class="sxs-lookup"><span data-stu-id="5a792-112">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
* <span data-ttu-id="5a792-113">**Разделение областей ответственности** между центральным ИТ-отделом (SecOps, InfraOps) и рабочими нагрузками (DevOps).</span><span class="sxs-lookup"><span data-stu-id="5a792-113">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="5a792-114">Типичные способы использования этой архитектуры:</span><span class="sxs-lookup"><span data-stu-id="5a792-114">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="5a792-115">Рабочая нагрузка, развернутая в разных средах, например в среде для разработки, тестирования и в рабочей среде, для которых требуются общие службы, например DNS, IDS, NTP или AD DS.</span><span class="sxs-lookup"><span data-stu-id="5a792-115">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="5a792-116">Общие службы размещаются в виртуальной сети концентратора, в то время как каждая среда развертывается в периферийной зоне для обеспечения изоляции.</span><span class="sxs-lookup"><span data-stu-id="5a792-116">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
* <span data-ttu-id="5a792-117">Рабочие нагрузки, которые не требуют подключения друг к другу, но требуют доступа к общим службам.</span><span class="sxs-lookup"><span data-stu-id="5a792-117">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
* <span data-ttu-id="5a792-118">Предприятия, которые требуют централизованного контроля над аспектами безопасности, такими как брандмауэр в концентраторе, таком как сеть периметра, и отдельного управления для рабочих нагрузок в каждой периферийной зоне.</span><span class="sxs-lookup"><span data-stu-id="5a792-118">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="5a792-119">Архитектура</span><span class="sxs-lookup"><span data-stu-id="5a792-119">Architecture</span></span>

<span data-ttu-id="5a792-120">Архитектура состоит из следующих компонентов:</span><span class="sxs-lookup"><span data-stu-id="5a792-120">The architecture consists of the following components.</span></span>

* <span data-ttu-id="5a792-121">**Локальная сеть.**</span><span class="sxs-lookup"><span data-stu-id="5a792-121">**On-premises network**.</span></span> <span data-ttu-id="5a792-122">Частная локальная сеть, работающая внутри организации.</span><span class="sxs-lookup"><span data-stu-id="5a792-122">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="5a792-123">**VPN-устройство**.</span><span class="sxs-lookup"><span data-stu-id="5a792-123">**VPN device**.</span></span> <span data-ttu-id="5a792-124">Устройство или служба, предоставляющая возможность внешнего подключения к локальной сети.</span><span class="sxs-lookup"><span data-stu-id="5a792-124">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="5a792-125">VPN-устройство может быть аппаратным устройством или программным решением, таким как служба маршрутизации и удаленного доступа (RRAS) в Windows Server 2012.</span><span class="sxs-lookup"><span data-stu-id="5a792-125">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="5a792-126">Список поддерживаемых VPN-устройств и информацию о настройке выбранных VPN-устройств для подключения к Azure см. в статье [VPN-устройства и параметры IPsec/IKE для подключений типа "сеть — сеть" через VPN-шлюз][vpn-appliance].</span><span class="sxs-lookup"><span data-stu-id="5a792-126">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="5a792-127">**Сетевой шлюз виртуальной сети VPN или шлюз ExpressRoute**.</span><span class="sxs-lookup"><span data-stu-id="5a792-127">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="5a792-128">Шлюз виртуальной сети позволяет виртуальной сети подключаться к VPN-устройству или к каналу ExpressRoute, которые используются для подключения к вашей локальной сети.</span><span class="sxs-lookup"><span data-stu-id="5a792-128">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="5a792-129">Дополнительные сведения см. в статье [Подключение локальной сети к виртуальной сети Microsoft Azure][connect-to-an-Azure-vnet].</span><span class="sxs-lookup"><span data-stu-id="5a792-129">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="5a792-130">В сценариях развертывания для этой эталонной архитектуры используется VPN-шлюз для подключения, а виртуальная сеть в Azure используется для имитации вашей локальной сети.</span><span class="sxs-lookup"><span data-stu-id="5a792-130">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

* <span data-ttu-id="5a792-131">**Виртуальная сеть концентратора**.</span><span class="sxs-lookup"><span data-stu-id="5a792-131">**Hub VNet**.</span></span> <span data-ttu-id="5a792-132">Виртуальная сеть Azure используется в качестве концентратора в звездообразной топологии.</span><span class="sxs-lookup"><span data-stu-id="5a792-132">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="5a792-133">Концентратор является центральной точкой подключения к вашей локальной сети и местом для размещения служб, которые могут использоваться различными рабочими нагрузками, размещенными в виртуальных сетях.</span><span class="sxs-lookup"><span data-stu-id="5a792-133">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

* <span data-ttu-id="5a792-134">**Подсеть шлюза**.</span><span class="sxs-lookup"><span data-stu-id="5a792-134">**Gateway subnet**.</span></span> <span data-ttu-id="5a792-135">Шлюзы виртуальных сетей хранятся в одной подсети.</span><span class="sxs-lookup"><span data-stu-id="5a792-135">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="5a792-136">**Подсеть общих служб**.</span><span class="sxs-lookup"><span data-stu-id="5a792-136">**Shared services subnet**.</span></span> <span data-ttu-id="5a792-137">Подсеть в концентраторе виртуальной сети используется для размещения служб, которые могут использоваться совместно во всех периферийных зонах, например DNS или AD DS.</span><span class="sxs-lookup"><span data-stu-id="5a792-137">A subnet in the hub VNet used to host services that can be shared among all spokes, such as DNS or AD DS.</span></span>

* <span data-ttu-id="5a792-138">**Подсеть сети периметра.**</span><span class="sxs-lookup"><span data-stu-id="5a792-138">**DMZ subnet**.</span></span> <span data-ttu-id="5a792-139">Подсеть в виртуальной сети концентратора, используемая для размещения виртуальных сетевых модулей, которые могут действовать как устройства безопасности, например как брандмауэры.</span><span class="sxs-lookup"><span data-stu-id="5a792-139">A subnet in the hub VNet used to host NVAs that can act as security appliances, such as firewalls.</span></span>

* <span data-ttu-id="5a792-140">**Виртуальные сети периферийных зон**.</span><span class="sxs-lookup"><span data-stu-id="5a792-140">**Spoke VNets**.</span></span> <span data-ttu-id="5a792-141">Одна или несколько виртуальных сетей Azure, которые используются в качестве периферийных зон в звездообразной топологии.</span><span class="sxs-lookup"><span data-stu-id="5a792-141">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="5a792-142">Периферийные зоны можно использовать для изоляции рабочих нагрузок в их виртуальных сетях, управляемых отдельно от других периферийных зон.</span><span class="sxs-lookup"><span data-stu-id="5a792-142">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="5a792-143">Каждая рабочая нагрузка может содержать несколько уровней с несколькими подсетями, подключенными через подсистемы балансировки нагрузки Azure.</span><span class="sxs-lookup"><span data-stu-id="5a792-143">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="5a792-144">Дополнительные сведения об инфраструктуре приложений см. в статьях [Запуск рабочих нагрузок на виртуальных машинах Windows][windows-vm-ra] и [Запуск рабочих нагрузок на виртуальной машине Linux][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="5a792-144">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="5a792-145">**Пиринговая связь между виртуальными сетями**.</span><span class="sxs-lookup"><span data-stu-id="5a792-145">**VNet peering**.</span></span> <span data-ttu-id="5a792-146">Две виртуальные сети в одном регионе Azure можно подключить с помощью [пиринга][vnet-peering].</span><span class="sxs-lookup"><span data-stu-id="5a792-146">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="5a792-147">Пиринговые подключения представляют собой нетранзитивные подключения между виртуальными подсетями с низкой задержкой.</span><span class="sxs-lookup"><span data-stu-id="5a792-147">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="5a792-148">После установки пирингового подключения виртуальные сети обмениваются трафиком по магистрали Azure без необходимости использования маршрутизатора.</span><span class="sxs-lookup"><span data-stu-id="5a792-148">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="5a792-149">В звездообразной топологии сети пиринг виртуальной сети используется для подключения концентратора к каждой периферийной зоне.</span><span class="sxs-lookup"><span data-stu-id="5a792-149">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="5a792-150">В этой статье рассматриваются только [развертывания Resource Manager](/azure/azure-resource-manager/resource-group-overview), но вы также можете подключить классическую виртуальную сеть к виртуальной сети Resource Manager в той же подписке.</span><span class="sxs-lookup"><span data-stu-id="5a792-150">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="5a792-151">Таким образом, ваши периферийные зоны могут размещать классические развертывания и по-прежнему пользоваться преимуществами общих служб в концентраторе.</span><span class="sxs-lookup"><span data-stu-id="5a792-151">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>

## <a name="recommendations"></a><span data-ttu-id="5a792-152">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="5a792-152">Recommendations</span></span>

<span data-ttu-id="5a792-153">Все рекомендации для [звездообразной][guidance-hub-spoke] эталонной архитектуры также можно применить к эталонной архитектуре общих служб.</span><span class="sxs-lookup"><span data-stu-id="5a792-153">All the recommendations for the [hub-spoke][guidance-hub-spoke] reference architecture also apply to the shared services reference architecture.</span></span> 

<span data-ttu-id="5a792-154">Кроме того, следующие рекомендации применимы для большинства ситуаций с общими службами.</span><span class="sxs-lookup"><span data-stu-id="5a792-154">ALso, the following recommendations apply for most scenarios under shared services.</span></span> <span data-ttu-id="5a792-155">Следуйте этим рекомендациям, если они не противоречат особым требованиям для вашего случая.</span><span class="sxs-lookup"><span data-stu-id="5a792-155">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="identity"></a><span data-ttu-id="5a792-156">Удостоверение</span><span class="sxs-lookup"><span data-stu-id="5a792-156">Identity</span></span>

<span data-ttu-id="5a792-157">В локальном центре обработки данных большинства коммерческих организаций есть среда Active Directory Services (ADDS).</span><span class="sxs-lookup"><span data-stu-id="5a792-157">Most enterprise organizations have an Active Directory Directory Services (ADDS) environment in their on-premises datacenter.</span></span> <span data-ttu-id="5a792-158">Чтобы упростить управление ресурсами, перемещенными в Azure из локальной сети, которая зависит от ADDS, рекомендуется разместить контроллеры домена ADDS в Azure.</span><span class="sxs-lookup"><span data-stu-id="5a792-158">To facilitate management of assets moved to Azure from your on-premises network that depend on ADDS, it is recommended to host ADDS domain controllers in Azure.</span></span>

<span data-ttu-id="5a792-159">Если вы используете объекты групповой политики, которыми вы хотите управлять отдельно для Azure и локальной среды, для каждого региона Azure необходимо использовать разные сайты AD.</span><span class="sxs-lookup"><span data-stu-id="5a792-159">If you make use of Group Policy Objects, that you want to control separately for Azure and your on-premises environment, use a different AD site for each Azure region.</span></span> <span data-ttu-id="5a792-160">Разместите контроллеры домена в центральной виртуальной сети (концентраторе), доступ к которой могут получить зависимые рабочие нагрузки.</span><span class="sxs-lookup"><span data-stu-id="5a792-160">Place your domain controllers in a central VNet (hub) that dependent workloads can access.</span></span>

### <a name="security"></a><span data-ttu-id="5a792-161">Безопасность</span><span class="sxs-lookup"><span data-stu-id="5a792-161">Security</span></span>

<span data-ttu-id="5a792-162">При перемещении рабочих нагрузок из локальной среды в Azure некоторые из них потребуется разместить на виртуальных машинах.</span><span class="sxs-lookup"><span data-stu-id="5a792-162">As you move workloads from your on-premises environment to Azure, some of these workloads will require to be hosted in VMs.</span></span> <span data-ttu-id="5a792-163">В целях обеспечения соответствия может потребоваться наложить ограничения на передачу трафика этих рабочих нагрузок.</span><span class="sxs-lookup"><span data-stu-id="5a792-163">For compliance reasons, you may need to enforce restrictions on traffic traversing those workloads.</span></span> 

<span data-ttu-id="5a792-164">Вы можете использовать виртуальные сетевые модули в Azure для размещения разных типов служб безопасности и производительности.</span><span class="sxs-lookup"><span data-stu-id="5a792-164">You can use network virtual appliances (NVAs) in Azure to host different types of security and performance services.</span></span> <span data-ttu-id="5a792-165">Если вы знакомы с заданным набором устройств в локальной среде, рекомендуется использовать те же виртуальные модули в Azure, где это применимо.</span><span class="sxs-lookup"><span data-stu-id="5a792-165">If you are familiar with a given set of appliances on-premises today, it is recommended to use the same virtualized appliances in Azure, where applicable.</span></span>

> [!NOTE]
> <span data-ttu-id="5a792-166">В скриптах развертывания этой эталонной архитектуры используется виртуальная машина Ubuntu с IP-пересылкой, которая позволяет сымитировать виртуальный сетевой модуль.</span><span class="sxs-lookup"><span data-stu-id="5a792-166">The deployment scripts for this reference architecture use an Ubuntu VM with IP forwarding enbaled to mimic a network virtual appliance.</span></span>

## <a name="considerations"></a><span data-ttu-id="5a792-167">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="5a792-167">Considerations</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="5a792-168">Преодоление ограничения пиринговых подключений виртуальной сети</span><span class="sxs-lookup"><span data-stu-id="5a792-168">Overcoming VNet peering limits</span></span>

<span data-ttu-id="5a792-169">Убедитесь, что вы учли [ограничение числа пиринговых подключений виртуальных сетей на одну виртуальную сеть][vnet-peering-limit] в Azure.</span><span class="sxs-lookup"><span data-stu-id="5a792-169">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="5a792-170">Если вам нужно превысить лимит по периферийным зонам, подумайте о создании вложенной звездообразной топологии, где первый уровень периферийных зон также выступает в качестве концентраторов.</span><span class="sxs-lookup"><span data-stu-id="5a792-170">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="5a792-171">Этот подход показан на схеме ниже.</span><span class="sxs-lookup"><span data-stu-id="5a792-171">The following diagram shows this approach.</span></span>

<span data-ttu-id="5a792-172">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="5a792-172">![[3]][3]</span></span>

<span data-ttu-id="5a792-173">Кроме того, следует учитывать, какие службы используются совместно в концентраторе, чтобы убедиться, что концентратор масштабируется в соответствии с большим количеством периферийных зон.</span><span class="sxs-lookup"><span data-stu-id="5a792-173">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="5a792-174">Например, если ваш концентратор предоставляет службы брандмауэра, рассмотрите возможность ограничения пропускной способности решения брандмауэра при добавлении нескольких периферийных зон.</span><span class="sxs-lookup"><span data-stu-id="5a792-174">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="5a792-175">Возможно, вам захочется перенести некоторые из этих общих служб на второй уровень концентраторов.</span><span class="sxs-lookup"><span data-stu-id="5a792-175">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="5a792-176">Развертывание решения</span><span class="sxs-lookup"><span data-stu-id="5a792-176">Deploy the solution</span></span>

<span data-ttu-id="5a792-177">Пример развертывания для этой архитектуры можно найти на портале [GitHub][ref-arch-repo].</span><span class="sxs-lookup"><span data-stu-id="5a792-177">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="5a792-178">В нем используются виртуальные машины Ubuntu в каждой виртуальной сети для проверки возможности подключения.</span><span class="sxs-lookup"><span data-stu-id="5a792-178">It uses Ubuntu VMs in each VNet to test connectivity.</span></span> <span data-ttu-id="5a792-179">В подсети **shared-services** в **виртуальной сети концентратора** нет настоящих служб.</span><span class="sxs-lookup"><span data-stu-id="5a792-179">There are no actual services hosted in the **shared-services** subnet in the **hub VNet**.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="5a792-180">предварительным требованиям</span><span class="sxs-lookup"><span data-stu-id="5a792-180">Prerequisites</span></span>

<span data-ttu-id="5a792-181">Перед развертыванием эталонной архитектуры в собственной подписке необходимо выполнить следующие действия.</span><span class="sxs-lookup"><span data-stu-id="5a792-181">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="5a792-182">Клонируйте и скачайте ZIP-файл [с эталонными архитектурами AzureCAT][ref-arch-repo] в репозитории GitHub, а также выполните его разветвление.</span><span class="sxs-lookup"><span data-stu-id="5a792-182">Clone, fork, or download the zip file for the [AzureCAT reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="5a792-183">Убедитесь, что Azure CLI 2.0 установлен на компьютере.</span><span class="sxs-lookup"><span data-stu-id="5a792-183">Make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="5a792-184">См. инструкции по [установке Azure CLI 2.0][azure-cli-2].</span><span class="sxs-lookup"><span data-stu-id="5a792-184">For CLI installation instructions, see [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="5a792-185">Установите пакет npm [стандартных блоков Azure][azbb].</span><span class="sxs-lookup"><span data-stu-id="5a792-185">Install the [Azure building blocks][azbb] npm package.</span></span>

4. <span data-ttu-id="5a792-186">Из командной строки, строки bash или PowerShell войдите в свою учетную запись Azure с помощью приведенной ниже команды и следуйте инструкциям.</span><span class="sxs-lookup"><span data-stu-id="5a792-186">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using the command below, and follow the prompts.</span></span>

  ```bash
  az login
  ```

### <a name="deploy-the-simulated-on-premises-datacenter-using-azbb"></a><span data-ttu-id="5a792-187">Развертывание имитации локального центра обработки данных с помощью azbb</span><span class="sxs-lookup"><span data-stu-id="5a792-187">Deploy the simulated on-premises datacenter using azbb</span></span>

<span data-ttu-id="5a792-188">Чтобы развернуть имитацию локального центра обработки данных в виртуальной сети Azure, следуйте этим инструкциям:</span><span class="sxs-lookup"><span data-stu-id="5a792-188">To deploy the simulated on-premises datacenter as an Azure VNet, follow these steps:</span></span>

1. <span data-ttu-id="5a792-189">Перейдите в папку `hybrid-networking\shared-services-stack\` репозитория, скачанного на предыдущем шаге.</span><span class="sxs-lookup"><span data-stu-id="5a792-189">Navigate to the `hybrid-networking\shared-services-stack\` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="5a792-190">Откройте файл `onprem.json` и введите имя пользователя и пароль в кавычках в строках 45 и 46, как показано ниже, а затем сохраните файл.</span><span class="sxs-lookup"><span data-stu-id="5a792-190">Open the `onprem.json` file and enter a username and password between the quotes in line 45 and 46, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. <span data-ttu-id="5a792-191">Выполните команду `azbb`, чтобы развернуть имитированную локальную среду, как показано ниже.</span><span class="sxs-lookup"><span data-stu-id="5a792-191">Run `azbb` to deploy the simulated onprem environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g onprem-vnet-rg - l <location> -p onoprem.json --deploy
  ```
  > [!NOTE]
  > <span data-ttu-id="5a792-192">Если вы решили использовать другое имя группы ресурсов (отличное от `onprem-vnet-rg`), обязательно найдите все файлы параметров, которые используют это имя, и отредактируйте их, чтобы использовать собственное имя группы ресурсов.</span><span class="sxs-lookup"><span data-stu-id="5a792-192">If you decide to use a different resource group name (other than `onprem-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

4. <span data-ttu-id="5a792-193">Дождитесь завершения развертывания.</span><span class="sxs-lookup"><span data-stu-id="5a792-193">Wait for the deployment to finish.</span></span> <span data-ttu-id="5a792-194">При развертывании создается виртуальная сеть, виртуальная машина под управлением Windows и VPN-шлюз.</span><span class="sxs-lookup"><span data-stu-id="5a792-194">This deployment creates a virtual network, a virtual machine running Windows, and a VPN gateway.</span></span> <span data-ttu-id="5a792-195">Создание VPN-шлюза может занять более 40 минут.</span><span class="sxs-lookup"><span data-stu-id="5a792-195">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="azure-hub-vnet"></a><span data-ttu-id="5a792-196">Виртуальная сеть концентратора Azure</span><span class="sxs-lookup"><span data-stu-id="5a792-196">Azure hub VNet</span></span>

<span data-ttu-id="5a792-197">Чтобы развернуть виртуальную сеть концентратора Azure и подключиться к имитации локальной виртуальной сети, созданной выше, сделайте следующее.</span><span class="sxs-lookup"><span data-stu-id="5a792-197">To deploy the hub VNet, and connect to the simulated on-premises VNet created above, perform the following steps.</span></span>

1. <span data-ttu-id="5a792-198">Откройте файл `hub-vnet.json` и введите имя пользователя и пароль в кавычках в строках 50 и 51, как показано ниже.</span><span class="sxs-lookup"><span data-stu-id="5a792-198">Open the `hub-vnet.json` file and enter a username and password between the quotes in line 50 and 51, as shown below.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

2. <span data-ttu-id="5a792-199">В строке 52 для параметра `osType` введите `Windows` или `Linux`, чтобы установить Windows Server 2016 Datacenter или Ubuntu 16.04 в качестве операционной системы для jumpbox.</span><span class="sxs-lookup"><span data-stu-id="5a792-199">On line 52, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

3. <span data-ttu-id="5a792-200">Введите общий ключ в кавычках в строке 83, как показано ниже, а затем сохраните файл.</span><span class="sxs-lookup"><span data-stu-id="5a792-200">Enter a shared key between the quotes in line 83, as shown below, then save the file.</span></span>

  ```bash
  "sharedKey": "",
  ```

4. <span data-ttu-id="5a792-201">Выполните команду `azbb`, чтобы развернуть имитированную локальную среду, как показано ниже.</span><span class="sxs-lookup"><span data-stu-id="5a792-201">Run `azbb` to deploy the simulated onprem environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet.json --deploy
  ```
  > [!NOTE]
  > <span data-ttu-id="5a792-202">Если вы решили использовать другое имя группы ресурсов (отличное от `hub-vnet-rg`), обязательно найдите все файлы параметров, которые используют это имя, и отредактируйте их, чтобы использовать собственное имя группы ресурсов.</span><span class="sxs-lookup"><span data-stu-id="5a792-202">If you decide to use a different resource group name (other than `hub-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="5a792-203">Дождитесь завершения развертывания.</span><span class="sxs-lookup"><span data-stu-id="5a792-203">Wait for the deployment to finish.</span></span> <span data-ttu-id="5a792-204">В ходе этого развертывания создается виртуальная сеть, виртуальная машина, VPN-шлюз и подключение к шлюзу, созданному в предыдущем разделе.</span><span class="sxs-lookup"><span data-stu-id="5a792-204">This deployment creates a virtual network, a virtual machine, a VPN gateway, and a connection to the gateway created in the previous section.</span></span> <span data-ttu-id="5a792-205">Создание VPN-шлюза может занять более 40 минут.</span><span class="sxs-lookup"><span data-stu-id="5a792-205">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="adds-in-azure"></a><span data-ttu-id="5a792-206">ADDS в Azure</span><span class="sxs-lookup"><span data-stu-id="5a792-206">ADDS in Azure</span></span>

<span data-ttu-id="5a792-207">Чтобы развернуть контроллеры домена ADDS в Azure, сделайте следующее.</span><span class="sxs-lookup"><span data-stu-id="5a792-207">To deploy the ADDS domain controllers in Azure, perform the following steps.</span></span>

1. <span data-ttu-id="5a792-208">Откройте файл `hub-adds.json` и введите имя пользователя и пароль в кавычках в строках 14 и 15, как показано ниже, а затем сохраните файл.</span><span class="sxs-lookup"><span data-stu-id="5a792-208">Open the `hub-adds.json` file and enter a username and password between the quotes in lines 14 and 15, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

2. <span data-ttu-id="5a792-209">Запустите `azbb`, чтобы развернуть контроллеры домена ADDS, как показано ниже.</span><span class="sxs-lookup"><span data-stu-id="5a792-209">Run `azbb` to deploy the ADDS domain controllers as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g hub-adds-rg - l <location> -p hub-adds.json --deploy
  ```
  
  > [!NOTE]
  > <span data-ttu-id="5a792-210">Если вы решили использовать другое имя группы ресурсов (отличное от `hub-adds-rg`), обязательно найдите все файлы параметров, которые используют это имя, и отредактируйте их, чтобы использовать собственное имя группы ресурсов.</span><span class="sxs-lookup"><span data-stu-id="5a792-210">If you decide to use a different resource group name (other than `hub-adds-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

  > [!NOTE]
  > <span data-ttu-id="5a792-211">Этот этап развертывания может занять несколько минут, так как для него требуется подключение двух виртуальных машин к домену, размещенному в имитации локального центра обработки данных, а затем необходимо установить на них AD DS.</span><span class="sxs-lookup"><span data-stu-id="5a792-211">This part of the deployment may take several minutes, since it requires joining the two VMs to the domain hosted int he simulated on-premises datacenter, then installing AD DS on them.</span></span>

### <a name="nva"></a><span data-ttu-id="5a792-212">Виртуальные сетевые модули</span><span class="sxs-lookup"><span data-stu-id="5a792-212">NVA</span></span>

<span data-ttu-id="5a792-213">Чтобы развернуть виртуальный сетевой модуль в подсети `dmz`, сделайте следующее.</span><span class="sxs-lookup"><span data-stu-id="5a792-213">To deploy an NVA in the `dmz` subnet, perform the following steps:</span></span>

1. <span data-ttu-id="5a792-214">Откройте файл `hub-nva.json` и введите имя пользователя и пароль в кавычках в строках 13 и 14, как показано ниже, а затем сохраните файл.</span><span class="sxs-lookup"><span data-stu-id="5a792-214">Open the `hub-nva.json` file and enter a username and password between the quotes in lines 13 and 14, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```
2. <span data-ttu-id="5a792-215">Выполните команду `azbb`, чтобы развернуть виртуальную машину NVA и определяемые пользователем маршруты.</span><span class="sxs-lookup"><span data-stu-id="5a792-215">Run `azbb` to deploy the NVA VM and user defined routes.</span></span>

  ```bash
  azbb -s <subscription_id> -g hub-nva-rg - l <location> -p hub-nva.json --deploy
  ```
  > [!NOTE]
  > <span data-ttu-id="5a792-216">Если вы решили использовать другое имя группы ресурсов (отличное от `hub-nva-rg`), обязательно найдите все файлы параметров, которые используют это имя, и отредактируйте их, чтобы использовать собственное имя группы ресурсов.</span><span class="sxs-lookup"><span data-stu-id="5a792-216">If you decide to use a different resource group name (other than `hub-nva-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

### <a name="azure-spoke-vnets"></a><span data-ttu-id="5a792-217">Виртуальные сети периферийных зон Azure</span><span class="sxs-lookup"><span data-stu-id="5a792-217">Azure spoke VNets</span></span>

<span data-ttu-id="5a792-218">Чтобы развернуть виртуальные сети периферийных зон, сделайте следующее.</span><span class="sxs-lookup"><span data-stu-id="5a792-218">To deploy the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="5a792-219">Откройте файл `spoke1.json` и введите имя пользователя и пароль в кавычках в строках 52 и 53, как показано ниже, а затем сохраните файл.</span><span class="sxs-lookup"><span data-stu-id="5a792-219">Open the `spoke1.json` file and enter a username and password between the quotes in lines 52 and 53, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

2. <span data-ttu-id="5a792-220">В строке 54 для параметра `osType` введите `Windows` или `Linux`, чтобы установить Windows Server 2016 Datacenter или Ubuntu 16.04 в качестве операционной системы для jumpbox.</span><span class="sxs-lookup"><span data-stu-id="5a792-220">On line 54, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

3. <span data-ttu-id="5a792-221">Выполните команду `azbb`, чтобы развернуть первую среду виртуальной сети периферийных зон, как показано ниже.</span><span class="sxs-lookup"><span data-stu-id="5a792-221">Run `azbb` to deploy the first spoke VNet environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g spoke1-vnet-rg - l <location> -p spoke1.json --deploy
  ```
  
  > [!NOTE]
  > <span data-ttu-id="5a792-222">Если вы решили использовать другое имя группы ресурсов (отличное от `spoke1-vnet-rg`), обязательно найдите все файлы параметров, которые используют это имя, и отредактируйте их, чтобы использовать собственное имя группы ресурсов.</span><span class="sxs-lookup"><span data-stu-id="5a792-222">If you decide to use a different resource group name (other than `spoke1-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

3. <span data-ttu-id="5a792-223">Повторите шаг 1 для файла `spoke2.json`.</span><span class="sxs-lookup"><span data-stu-id="5a792-223">Repeat step 1 above for file `spoke2.json`.</span></span>

4. <span data-ttu-id="5a792-224">Выполните команду `azbb`, чтобы развернуть вторую среду виртуальной сети периферийных зон, как показано ниже.</span><span class="sxs-lookup"><span data-stu-id="5a792-224">Run `azbb` to deploy the second spoke VNet environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g spoke2-vnet-rg - l <location> -p spoke2.json --deploy
  ```
  > [!NOTE]
  > <span data-ttu-id="5a792-225">Если вы решили использовать другое имя группы ресурсов (отличное от `spoke2-vnet-rg`), обязательно найдите все файлы параметров, которые используют это имя, и отредактируйте их, чтобы использовать собственное имя группы ресурсов.</span><span class="sxs-lookup"><span data-stu-id="5a792-225">If you decide to use a different resource group name (other than `spoke2-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a><span data-ttu-id="5a792-226">Пиринговое подключение виртуальной сети концентратора Azure к виртуальным сетям периферийных зон</span><span class="sxs-lookup"><span data-stu-id="5a792-226">Azure hub VNet peering to spoke VNets</span></span>

<span data-ttu-id="5a792-227">Чтобы создать пиринговую связь между виртуальной сетью концентратора и виртуальными сетями периферийных зон, сделайте следующее.</span><span class="sxs-lookup"><span data-stu-id="5a792-227">To create a peering connection from the hub VNet to the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="5a792-228">Откройте файл `hub-vnet-peering.json`, чтобы проверить правильность имени группы ресурсов и имени виртуальной сети для каждого пиринга виртуальных сетей, начиная с 29 строки.</span><span class="sxs-lookup"><span data-stu-id="5a792-228">Open the `hub-vnet-peering.json` file and verify that the resource group name, and virtual network name for each of the virtual network peerings starting in line 29 are correct.</span></span>

2. <span data-ttu-id="5a792-229">Выполните команду `azbb`, чтобы развернуть первую среду виртуальной сети периферийных зон, как показано ниже.</span><span class="sxs-lookup"><span data-stu-id="5a792-229">Run `azbb` to deploy the first spoke VNet environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet-peering.json --deploy
  ```

  > [!NOTE]
  > <span data-ttu-id="5a792-230">Если вы решили использовать другое имя группы ресурсов (отличное от `hub-vnet-rg`), обязательно найдите все файлы параметров, которые используют это имя, и отредактируйте их, чтобы использовать собственное имя группы ресурсов.</span><span class="sxs-lookup"><span data-stu-id="5a792-230">If you decide to use a different resource group name (other than `hub-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

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
[виртуальный центр обработки данных]: https://aka.ms/vdc
[vnet-peering]: /azure/virtual-network/virtual-network-peering-overview
[vnet-peering-limit]: /azure/azure-subscription-service-limits#networking-limits
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.azureedge.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/shared-services.png "Топология общих служб в Azure"
[3]: ./images/hub-spokehub-spoke.svg "Вложенная звездообразная топология в Azure"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
