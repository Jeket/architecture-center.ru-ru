---
title: "Реализация звездообразной топологии сети в Azure"
description: "Как реализовывать звездообразную топологию сети в Azure."
author: telmosampaio
ms.date: 05/05/2017
pnp.series.title: Implement a hub-spoke network topology in Azure
pnp.series.prev: expressroute
ms.openlocfilehash: e6f07a7962dd5728226b023700268340590d97a3
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/14/2017
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a><span data-ttu-id="57412-103">Реализация звездообразной топологии сети в Azure</span><span class="sxs-lookup"><span data-stu-id="57412-103">Implement a hub-spoke network topology in Azure</span></span>

<span data-ttu-id="57412-104">На схеме эталонной архитектуры представлены сведения о том, как реализовать звездообразную топологию в Azure.</span><span class="sxs-lookup"><span data-stu-id="57412-104">This reference architecture shows how to implement a hub-spoke topology in Azure.</span></span> <span data-ttu-id="57412-105">*Концентратор* является виртуальной сетью в Azure, которая выступает в качестве центральной точки подключения к вашей локальной сети.</span><span class="sxs-lookup"><span data-stu-id="57412-105">The *hub* is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="57412-106">*Периферийные зоны* — это виртуальные сети, которые устанавливают пиринг с концентратором и могут использоваться для изоляции рабочих нагрузок.</span><span class="sxs-lookup"><span data-stu-id="57412-106">The *spokes* are VNets that peer with the hub, and can be used to isolate workloads.</span></span> <span data-ttu-id="57412-107">Трафик передается между локальным центром обработки данных и концентратором через подключение ExpressRoute или VPN-шлюз.</span><span class="sxs-lookup"><span data-stu-id="57412-107">Traffic flows between the on-premises datacenter and the hub through an ExpressRoute or VPN gateway connection.</span></span>  <span data-ttu-id="57412-108">[**Разверните это решение**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="57412-108">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="57412-109">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="57412-109">![[0]][0]</span></span>

<span data-ttu-id="57412-110">*Скачайте [файл Visio][visio-download] этой архитектуры*</span><span class="sxs-lookup"><span data-stu-id="57412-110">*Download a [Visio file][visio-download] of this architecture*</span></span>


<span data-ttu-id="57412-111">Преимущества этой топологии:</span><span class="sxs-lookup"><span data-stu-id="57412-111">The benefits of this toplogy include:</span></span>

* <span data-ttu-id="57412-112">**Сокращение затрат** путем централизации служб, которые могут совместно использоваться несколькими рабочими нагрузками, такими как сетевые виртуальные устройства (NVA) и DNS-серверы в одном расположении.</span><span class="sxs-lookup"><span data-stu-id="57412-112">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
* <span data-ttu-id="57412-113">**Превышение лимитов подписки** путем пиринга виртуальных сетей из разных подписок к центральному концентратору.</span><span class="sxs-lookup"><span data-stu-id="57412-113">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
* <span data-ttu-id="57412-114">**Разделение областей ответственности** между центральным ИТ-отделом (SecOps, InfraOps) и рабочими нагрузками (DevOps).</span><span class="sxs-lookup"><span data-stu-id="57412-114">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="57412-115">Типичные способы использования этой архитектуры:</span><span class="sxs-lookup"><span data-stu-id="57412-115">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="57412-116">Рабочая нагрузка, развернутая в разных средах, например в среде для разработки, тестирования и в рабочей среде, для которых требуются общие службы, например DNS, IDS, NTP или AD DS.</span><span class="sxs-lookup"><span data-stu-id="57412-116">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="57412-117">Общие службы размещаются в виртуальной сети концентратора, в то время как каждая среда развертывается в периферийной зоне для обеспечения изоляции.</span><span class="sxs-lookup"><span data-stu-id="57412-117">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
* <span data-ttu-id="57412-118">Рабочие нагрузки, которые не требуют подключения друг к другу, но требуют доступа к общим службам.</span><span class="sxs-lookup"><span data-stu-id="57412-118">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
* <span data-ttu-id="57412-119">Предприятия, которые требуют централизованного контроля над аспектами безопасности, такими как брандмауэр в концентраторе, таком как сеть периметра, и отдельного управления для рабочих нагрузок в каждой периферийной зоне.</span><span class="sxs-lookup"><span data-stu-id="57412-119">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="57412-120">Архитектура</span><span class="sxs-lookup"><span data-stu-id="57412-120">Architecture</span></span>

<span data-ttu-id="57412-121">Архитектура состоит из следующих компонентов.</span><span class="sxs-lookup"><span data-stu-id="57412-121">The architecture consists of the following components.</span></span>

* <span data-ttu-id="57412-122">**Локальная сеть**.</span><span class="sxs-lookup"><span data-stu-id="57412-122">**On-premises network**.</span></span> <span data-ttu-id="57412-123">Частная локальная сеть, работающая внутри организации.</span><span class="sxs-lookup"><span data-stu-id="57412-123">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="57412-124">**VPN-устройство**.</span><span class="sxs-lookup"><span data-stu-id="57412-124">**VPN device**.</span></span> <span data-ttu-id="57412-125">Устройство или служба, предоставляющая возможность внешнего подключения к локальной сети.</span><span class="sxs-lookup"><span data-stu-id="57412-125">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="57412-126">VPN-устройство может быть аппаратным устройством или программным решением, таким как служба маршрутизации и удаленного доступа (RRAS) в Windows Server 2012.</span><span class="sxs-lookup"><span data-stu-id="57412-126">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="57412-127">Список поддерживаемых VPN-устройств и информацию о настройке выбранных VPN-устройств для подключения к Azure см. в статье [VPN-устройства и параметры IPsec/IKE для подключений типа "сеть — сеть" через VPN-шлюз][vpn-appliance].</span><span class="sxs-lookup"><span data-stu-id="57412-127">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="57412-128">**Сетевой шлюз виртуальной сети VPN или шлюз ExpressRoute**.</span><span class="sxs-lookup"><span data-stu-id="57412-128">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="57412-129">Шлюз виртуальной сети позволяет виртуальной сети подключаться к VPN-устройству или к каналу ExpressRoute, которые используются для подключения к вашей локальной сети.</span><span class="sxs-lookup"><span data-stu-id="57412-129">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="57412-130">Дополнительные сведения см. в статье [Подключение локальной сети к виртуальной сети Microsoft Azure][connect-to-an-Azure-vnet].</span><span class="sxs-lookup"><span data-stu-id="57412-130">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="57412-131">В сценариях развертывания для этой эталонной архитектуры используется VPN-шлюз для подключения, а виртуальная сеть в Azure используется для имитации вашей локальной сети.</span><span class="sxs-lookup"><span data-stu-id="57412-131">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

* <span data-ttu-id="57412-132">**Виртуальная сеть концентратора**.</span><span class="sxs-lookup"><span data-stu-id="57412-132">**Hub VNet**.</span></span> <span data-ttu-id="57412-133">Виртуальная сеть Azure используется в качестве концентратора в звездообразной топологии.</span><span class="sxs-lookup"><span data-stu-id="57412-133">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="57412-134">Концентратор является центральной точкой подключения к вашей локальной сети и местом для размещения служб, которые могут использоваться различными рабочими нагрузками, размещенными в виртуальных сетях.</span><span class="sxs-lookup"><span data-stu-id="57412-134">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

* <span data-ttu-id="57412-135">**Подсеть шлюза**.</span><span class="sxs-lookup"><span data-stu-id="57412-135">**Gateway subnet**.</span></span> <span data-ttu-id="57412-136">Шлюзы виртуальных сетей хранятся в одной подсети.</span><span class="sxs-lookup"><span data-stu-id="57412-136">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="57412-137">**Подсеть общих служб**.</span><span class="sxs-lookup"><span data-stu-id="57412-137">**Shared services subnet**.</span></span> <span data-ttu-id="57412-138">Подсеть в концентраторе виртуальной сети используется для размещения служб, которые могут использоваться совместно во всех периферийных зонах, например DNS или AD DS.</span><span class="sxs-lookup"><span data-stu-id="57412-138">A subnet in the hub VNet used to host services that can be shared among all spokes, such as DNS or AD DS.</span></span>

* <span data-ttu-id="57412-139">**Виртуальные сети периферийных зон**.</span><span class="sxs-lookup"><span data-stu-id="57412-139">**Spoke VNets**.</span></span> <span data-ttu-id="57412-140">Одна или несколько виртуальных сетей Azure, которые используются в качестве периферийных зон в звездообразной топологии.</span><span class="sxs-lookup"><span data-stu-id="57412-140">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="57412-141">Периферийные зоны можно использовать для изоляции рабочих нагрузок в их виртуальных сетях, управляемых отдельно от других периферийных зон.</span><span class="sxs-lookup"><span data-stu-id="57412-141">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="57412-142">Каждая рабочая нагрузка может содержать несколько уровней с несколькими подсетями, подключенными через подсистемы балансировки нагрузки Azure.</span><span class="sxs-lookup"><span data-stu-id="57412-142">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="57412-143">Дополнительные сведения об инфраструктуре приложений см. в статьях [Running Windows VM workloads][windows-vm-ra] (Выполнение рабочих нагрузок виртуальных машин Windows) и [Running Linux VM workloads][linux-vm-ra] (Выполнение рабочих нагрузок виртуальных машин Linux).</span><span class="sxs-lookup"><span data-stu-id="57412-143">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="57412-144">**Пиринговая связь между виртуальными сетями**.</span><span class="sxs-lookup"><span data-stu-id="57412-144">**VNet peering**.</span></span> <span data-ttu-id="57412-145">Две виртуальные сети в одном регионе Azure можно подключить с помощью [пиринга][vnet-peering].</span><span class="sxs-lookup"><span data-stu-id="57412-145">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="57412-146">Пиринговые подключения представляют собой нетранзитивные подключения между виртуальными подсетями с низкой задержкой.</span><span class="sxs-lookup"><span data-stu-id="57412-146">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="57412-147">После установки пирингового подключения виртуальные сети обмениваются трафиком по магистрали Azure без необходимости использования маршрутизатора.</span><span class="sxs-lookup"><span data-stu-id="57412-147">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="57412-148">В звездообразной топологии сети пиринг виртуальной сети используется для подключения концентратора к каждой периферийной зоне.</span><span class="sxs-lookup"><span data-stu-id="57412-148">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="57412-149">В этой статье рассматриваются только [развертывания Resource Manager](/azure/azure-resource-manager/resource-group-overview), но вы также можете подключить классическую виртуальную сеть к виртуальной сети Resource Manager в той же подписке.</span><span class="sxs-lookup"><span data-stu-id="57412-149">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="57412-150">Таким образом, ваши периферийные зоны могут размещать классические развертывания и по-прежнему пользоваться преимуществами общих служб в концентраторе.</span><span class="sxs-lookup"><span data-stu-id="57412-150">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>


## <a name="recommendations"></a><span data-ttu-id="57412-151">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="57412-151">Recommendations</span></span>

<span data-ttu-id="57412-152">Следующие рекомендации применимы для большинства ситуаций.</span><span class="sxs-lookup"><span data-stu-id="57412-152">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="57412-153">Следуйте этим рекомендациям, если они не противоречат особым требованиям для вашего случая.</span><span class="sxs-lookup"><span data-stu-id="57412-153">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="resource-groups"></a><span data-ttu-id="57412-154">Группы ресурсов</span><span class="sxs-lookup"><span data-stu-id="57412-154">Resource groups</span></span>

<span data-ttu-id="57412-155">Концентратор виртуальной сети и каждая периферийная зона виртуальной сети могут быть реализованы в разных группах ресурсов и даже в разных подписках, если они принадлежат одному и тому же клиенту Azure Active Directory (Azure AD) в одном регионе Azure.</span><span class="sxs-lookup"><span data-stu-id="57412-155">The hub VNet, and each spoke VNet, can be implemented in different resource groups, and even different subscriptions, as long as they belong to the same Azure Active Directory (Azure AD) tenant in the same Azure region.</span></span> <span data-ttu-id="57412-156">Это позволяет осуществлять децентрализованное управление каждой рабочей нагрузкой при совместном использовании служб, поддерживаемых в концентраторе виртуальной сети.</span><span class="sxs-lookup"><span data-stu-id="57412-156">This allows for a decentralized management of each workload, while sharing services maintained in the hub VNet.</span></span>

### <a name="vnet-and-gatewaysubnet"></a><span data-ttu-id="57412-157">Виртуальная сеть и подсеть шлюза</span><span class="sxs-lookup"><span data-stu-id="57412-157">VNet and GatewaySubnet</span></span>

<span data-ttu-id="57412-158">Создание подсети с именем *GatewaySubnet* с диапазоном адресов /27.</span><span class="sxs-lookup"><span data-stu-id="57412-158">Create a subnet named *GatewaySubnet*, with an address range of /27.</span></span> <span data-ttu-id="57412-159">Эта подсеть необходима для шлюза виртуальной сети.</span><span class="sxs-lookup"><span data-stu-id="57412-159">This subnet is required by the virtual network gateway.</span></span> <span data-ttu-id="57412-160">Выделение 32 адресов этой подсети поможет предотвратить достижение ограничения размера шлюза в будущем.</span><span class="sxs-lookup"><span data-stu-id="57412-160">Allocating 32 addresses to this subnet will help to prevent reaching gateway size limitations in the future.</span></span>

<span data-ttu-id="57412-161">Дополнительные сведения о настройке шлюза см. в следующих статьях с данными об эталонных архитектурах в зависимости от типа подключения:</span><span class="sxs-lookup"><span data-stu-id="57412-161">For more information about setting up the gateway, see the following reference architectures, depending on your connection type:</span></span>

- <span data-ttu-id="57412-162">[Connect an on-premises network to Azure using ExpressRoute][guidance-expressroute] (Подключение локальной сети к Azure с помощью ExpressRoute).</span><span class="sxs-lookup"><span data-stu-id="57412-162">[Hybrid network using ExpressRoute][guidance-expressroute]</span></span>
- <span data-ttu-id="57412-163">[Connect an on-premises network to Azure using a VPN gateway][guidance-vpn] (Подключение локальной сети к Azure с помощью VPN-шлюза).</span><span class="sxs-lookup"><span data-stu-id="57412-163">[Hybrid network using a VPN gateway][guidance-vpn]</span></span>

<span data-ttu-id="57412-164">Для обеспечения высокой доступности вы можете использовать ExpressRoute вместе с VPN для отработки отказа.</span><span class="sxs-lookup"><span data-stu-id="57412-164">For higher availability, you can use ExpressRoute plus a VPN for failover.</span></span> <span data-ttu-id="57412-165">Ознакомьтесь со статьей [Connect an on-premises network to Azure using ExpressRoute with VPN failover][hybrid-ha] (Подключение локальной сети к Azure с помощью ExpressRoute с отработкой отказа VPN).</span><span class="sxs-lookup"><span data-stu-id="57412-165">See [Connect an on-premises network to Azure using ExpressRoute with VPN failover][hybrid-ha].</span></span>

<span data-ttu-id="57412-166">Звездообразная топология может также использоваться без шлюза, если вам не требуется связь с локальной сетью.</span><span class="sxs-lookup"><span data-stu-id="57412-166">A hub-spoke topology can also be used without a gateway, if you don't need connectivity with your on-premises network.</span></span> 

### <a name="vnet-peering"></a><span data-ttu-id="57412-167">Пиринговая связь между виртуальными сетями</span><span class="sxs-lookup"><span data-stu-id="57412-167">VNet peering</span></span>

<span data-ttu-id="57412-168">Пиринговая связь между виртуальными сетями — это нетранзитивная связь между двумя виртуальными сетями.</span><span class="sxs-lookup"><span data-stu-id="57412-168">VNet peering is a non-transitive relationship between two VNets.</span></span> <span data-ttu-id="57412-169">Если вам требуется, чтобы периферийные зоны соединялись друг с другом, подумайте над добавлением отдельного пирингового подключения между ними.</span><span class="sxs-lookup"><span data-stu-id="57412-169">If you require spokes to connect to each other, consider adding a separate peering connection between those spokes.</span></span>

<span data-ttu-id="57412-170">Однако при наличии нескольких периферийных зон, которые требуется подключить между собой, очень скоро возникнет нехватка пиринговых подключений из-за ограничения [числа пиринговых подключений на каждую виртуальную сеть][vnet-peering-limit].</span><span class="sxs-lookup"><span data-stu-id="57412-170">However, if you have several spokes that need to connect with each other, you will run out of possible peering connections very quickly due to the [limitation on number of VNets peerings per VNet][vnet-peering-limit].</span></span> <span data-ttu-id="57412-171">В этом случае рассмотрите возможность использования пользовательских маршрутов (UDR) для принудительной передачи трафика, предназначенного для периферийной зоны, на виртуальное сетевое устройство (NVA), выступающее в роли маршрутизатора в виртуальной сети концентратора.</span><span class="sxs-lookup"><span data-stu-id="57412-171">In this scenario, consider using user defined routes (UDRs) to force traffic destined to a spoke to be sent to an NVA acting as a router at the hub VNet.</span></span> <span data-ttu-id="57412-172">Это позволит периферийным зонам подключаться друг к другу.</span><span class="sxs-lookup"><span data-stu-id="57412-172">This will allow the spokes to connect to each other.</span></span>

<span data-ttu-id="57412-173">Вы также можете настроить периферийные зоны для использования шлюза виртуальной сети концентратора для подключения к удаленным сетям.</span><span class="sxs-lookup"><span data-stu-id="57412-173">You can also configure spokes to use the hub VNet gateway to communicate with remote networks.</span></span> <span data-ttu-id="57412-174">Чтобы разрешить передачу трафика шлюза из периферийной зоны в концентратор и подключение к удаленным сетям, требуется сделать следующее:</span><span class="sxs-lookup"><span data-stu-id="57412-174">To allow gateway traffic to flow from spoke to hub, and connect to remote networks, you must:</span></span>

  - <span data-ttu-id="57412-175">Настроить пиринговую связь между виртуальными сетями в концентраторе, чтобы **разрешить транзит шлюзов**.</span><span class="sxs-lookup"><span data-stu-id="57412-175">Configure the VNet peering connection in the hub to **allow gateway transit**.</span></span>
  - <span data-ttu-id="57412-176">Настроить пиринговое подключение виртуальной сети к каждой периферийной зоне для **использования удаленных шлюзов**.</span><span class="sxs-lookup"><span data-stu-id="57412-176">Configure the VNet peering connection in each spoke to **use remote gateways**.</span></span>
  - <span data-ttu-id="57412-177">Настроить все пиринговые соединения виртуальных сетей, чтобы **разрешить перенаправление трафика**.</span><span class="sxs-lookup"><span data-stu-id="57412-177">Configure all VNet peering connections to **allow forwarded traffic**.</span></span>

## <a name="considerations"></a><span data-ttu-id="57412-178">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="57412-178">Considerations</span></span>

### <a name="spoke-connectivity"></a><span data-ttu-id="57412-179">Подключение периферийной зоны</span><span class="sxs-lookup"><span data-stu-id="57412-179">Spoke connectivity</span></span>

<span data-ttu-id="57412-180">Если вам требуется подключение между периферийными зонами, подумайте о внедрении виртуального сетевого устройства для маршрутизации в концентраторе и использовании определяемых пользователем маршрутов в периферийных зонах для направления трафика в концентратор.</span><span class="sxs-lookup"><span data-stu-id="57412-180">If you require connectivity between spokes, consider implementing an NVA for routing in the hub, and using UDRs in the spoke to forward traffic to the hub.</span></span>

<span data-ttu-id="57412-181">![[2]][2]</span><span class="sxs-lookup"><span data-stu-id="57412-181">![[2]][2]</span></span>

<span data-ttu-id="57412-182">В этом случае необходимо настроить пиринговые соединения, чтобы **разрешить перенаправленный трафик**.</span><span class="sxs-lookup"><span data-stu-id="57412-182">In this scenario, you must configure the peering connections to **allow forwarded traffic**.</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="57412-183">Преодоление ограничения пиринговых подключений виртуальной сети</span><span class="sxs-lookup"><span data-stu-id="57412-183">Overcoming VNet peering limits</span></span>

<span data-ttu-id="57412-184">Убедитесь, что вы учли [ограничение числа пиринговых подключений виртуальных сетей на одну виртуальную сеть][vnet-peering-limit] в Azure.</span><span class="sxs-lookup"><span data-stu-id="57412-184">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="57412-185">Если вам нужно превысить лимит по периферийным зонам, подумайте о создании вложенной звездообразной топологии, где первый уровень периферийных зон также выступает в качестве концентраторов.</span><span class="sxs-lookup"><span data-stu-id="57412-185">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="57412-186">Этот подход показан на схеме ниже.</span><span class="sxs-lookup"><span data-stu-id="57412-186">The following diagram shows this approach.</span></span>

<span data-ttu-id="57412-187">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="57412-187">![[3]][3]</span></span>

<span data-ttu-id="57412-188">Кроме того, следует учитывать, какие службы используются совместно в концентраторе, чтобы убедиться, что концентратор масштабируется в соответствии с большим количеством периферийных зон.</span><span class="sxs-lookup"><span data-stu-id="57412-188">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="57412-189">Например, если ваш концентратор предоставляет службы брандмауэра, рассмотрите возможность ограничения пропускной способности решения брандмауэра при добавлении нескольких периферийных зон.</span><span class="sxs-lookup"><span data-stu-id="57412-189">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="57412-190">Возможно, вам захочется перенести некоторые из этих общих служб на второй уровень концентраторов.</span><span class="sxs-lookup"><span data-stu-id="57412-190">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="57412-191">Развертывание решения</span><span class="sxs-lookup"><span data-stu-id="57412-191">Deploy the solution</span></span>

<span data-ttu-id="57412-192">Пример развертывания для этой архитектуры можно найти на портале [GitHub][ref-arch-repo].</span><span class="sxs-lookup"><span data-stu-id="57412-192">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="57412-193">В нем используются виртуальные машины Ubuntu в каждой виртуальной сети для проверки возможности подключения.</span><span class="sxs-lookup"><span data-stu-id="57412-193">It uses Ubuntu VMs in each VNet to test connectivity.</span></span> <span data-ttu-id="57412-194">В подсети **shared-services** в **виртуальной сети концентратора** нет настоящих служб.</span><span class="sxs-lookup"><span data-stu-id="57412-194">There are no actual services hosted in the **shared-services** subnet in the **hub VNet**.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="57412-195">Предварительные требования</span><span class="sxs-lookup"><span data-stu-id="57412-195">Prerequisites</span></span>

<span data-ttu-id="57412-196">Перед развертыванием эталонной архитектуры в собственной подписке необходимо выполнить следующие действия.</span><span class="sxs-lookup"><span data-stu-id="57412-196">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="57412-197">Клонируйте и скачайте ZIP-файл [с эталонными архитектурами AzureCAT][ref-arch-repo] в репозитории GitHub, а также выполните его разветвление.</span><span class="sxs-lookup"><span data-stu-id="57412-197">Clone, fork, or download the zip file for the [AzureCAT reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="57412-198">Если вы предпочитаете использовать Azure CLI, убедитесь, что у вас на компьютере установлен Azure CLI 2.0.</span><span class="sxs-lookup"><span data-stu-id="57412-198">If you prefer to use the Azure CLI, make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="57412-199">Чтобы установить CLI, следуйте инструкциям в статье об [установке Azure CLI 2.0][azure-cli-2].</span><span class="sxs-lookup"><span data-stu-id="57412-199">To install the CLI, follow the instructions in [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="57412-200">Если вы предпочитаете использовать PowerShell, убедитесь, что на вашем компьютере установлен последний модуль PowerShell для Azure.</span><span class="sxs-lookup"><span data-stu-id="57412-200">If you prefer to use PowerShell, make sure you have the latest PowerShell module for Azure installed on you computer.</span></span> <span data-ttu-id="57412-201">Чтобы установить последнюю версию модуля Azure PowerShell, следуйте инструкциям в статье об [установке PowerShell для Azure][azure-powershell].</span><span class="sxs-lookup"><span data-stu-id="57412-201">To install the latest Azure PowerShell module, follow the instructions in [Install PowerShell for Azure][azure-powershell].</span></span>

4. <span data-ttu-id="57412-202">В командной строке, строке bash или строке PowerShell войдите в свою учетную запись Azure с помощью одной из приведенных ниже команд и следуйте инструкциям.</span><span class="sxs-lookup"><span data-stu-id="57412-202">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using one of the commands below, and follow the prompts.</span></span>

  ```bash
  az login
  ```

  ```powershell
  Login-AzureRmAccount
  ```

### <a name="deploy-the-simulated-on-premises-datacenter"></a><span data-ttu-id="57412-203">Развертывание имитации локального центра обработки данных</span><span class="sxs-lookup"><span data-stu-id="57412-203">Deploy the simulated on-premises datacenter</span></span>

<span data-ttu-id="57412-204">Чтобы развернуть имитацию локального центра обработки данных в виртуальной сети Azure, сделайте следующее.</span><span class="sxs-lookup"><span data-stu-id="57412-204">To deploy the simulated on-premises datacenter as an Azure VNet, perform the following steps.</span></span>

1. <span data-ttu-id="57412-205">Перейдите в папку `hybrid-networking\hub-spoke\onprem` репозитория, скачанного на предыдущем шаге.</span><span class="sxs-lookup"><span data-stu-id="57412-205">Navigate to the `hybrid-networking\hub-spoke\onprem` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="57412-206">Откройте файл `onprem.vm.parameters.json` и введите имя пользователя и пароль в кавычках в строках 11 и 12, как показано ниже, а затем сохраните файл.</span><span class="sxs-lookup"><span data-stu-id="57412-206">Open the `onprem.vm.parameters.json` file and enter a username and password between the quotes in line 11 and 12, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. <span data-ttu-id="57412-207">Выполните следующую команду PowerShell или bash для развертывания имитации локальной среды в качестве виртуальной сети в Azure.</span><span class="sxs-lookup"><span data-stu-id="57412-207">Run the bash or PowerShell command below to deploy the simulated on-premises environment as a VNet in Azure.</span></span> <span data-ttu-id="57412-208">Замените значения своей подпиской, именем группы ресурсов и регионом Azure.</span><span class="sxs-lookup"><span data-stu-id="57412-208">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./onprem.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-onprem-rg \
    --location westus
  ```

  ```powershell
  ./onprem.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-onprem-rg `
    -Location westus
  ```
  > [!NOTE]
  > <span data-ttu-id="57412-209">Если вы решили использовать другое имя группы ресурсов (отличное от `ra-onprem-rg`), обязательно найдите все файлы параметров, которые используют это имя, и отредактируйте их, чтобы использовать собственное имя группы ресурсов.</span><span class="sxs-lookup"><span data-stu-id="57412-209">If you decide to use a different resource group name (other than `ra-onprem-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

4. <span data-ttu-id="57412-210">Дождитесь завершения развертывания.</span><span class="sxs-lookup"><span data-stu-id="57412-210">Wait for the deployment to finish.</span></span> <span data-ttu-id="57412-211">При развертывании создается виртуальная сеть, виртуальная машина Ubuntu и VPN-шлюз.</span><span class="sxs-lookup"><span data-stu-id="57412-211">This deployment creates a virtual network, a virtual machine running Ubuntu, and a VPN gateway.</span></span> <span data-ttu-id="57412-212">Создание VPN-шлюза может занять более 40 минут.</span><span class="sxs-lookup"><span data-stu-id="57412-212">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="azure-hub-vnet"></a><span data-ttu-id="57412-213">Виртуальная сеть концентратора Azure</span><span class="sxs-lookup"><span data-stu-id="57412-213">Azure hub VNet</span></span>

<span data-ttu-id="57412-214">Чтобы развернуть виртуальную сеть концентратора Azure и подключиться к имитации локальной виртуальной сети, созданной выше, сделайте следующее.</span><span class="sxs-lookup"><span data-stu-id="57412-214">To deploy the hub VNet, and connect to the simulated on-premises VNet created above, perform the following steps.</span></span>

1. <span data-ttu-id="57412-215">Перейдите в папку `hybrid-networking\hub-spoke\hub` репозитория, скачанного на предыдущем шаге.</span><span class="sxs-lookup"><span data-stu-id="57412-215">Navigate to the `hybrid-networking\hub-spoke\hub` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="57412-216">Откройте файл `hub.vm.parameters.json` и введите имя пользователя и пароль в кавычках в строках 11 и 12, как показано ниже, а затем сохраните файл.</span><span class="sxs-lookup"><span data-stu-id="57412-216">Open the `hub.vm.parameters.json` file and enter a username and password between the quotes in line 11 and 12, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. <span data-ttu-id="57412-217">Откройте файл `hub.gateway.parameters.json` и введите общий ключ в кавычках в строке 23, как показано ниже, а затем сохраните файл.</span><span class="sxs-lookup"><span data-stu-id="57412-217">Open the `hub.gateway.parameters.json` file and enter a shared key between the quotes in line 23, as shown below, then save the file.</span></span> <span data-ttu-id="57412-218">Сохраните это значение. Оно понадобится позже при развертывании.</span><span class="sxs-lookup"><span data-stu-id="57412-218">Keep a note of this value, you will need to use it later in the deployment.</span></span>

  ```bash
  "sharedKey": "",
  ```

4. <span data-ttu-id="57412-219">Выполните следующую команду PowerShell или bash для развертывания имитации локальной среды в качестве виртуальной сети в Azure.</span><span class="sxs-lookup"><span data-stu-id="57412-219">Run the bash or PowerShell command below to deploy the simulated on-premises environment as a VNet in Azure.</span></span> <span data-ttu-id="57412-220">Замените значения своей подпиской, именем группы ресурсов и регионом Azure.</span><span class="sxs-lookup"><span data-stu-id="57412-220">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./hub.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus
  ```

  ```powershell
  ./hub.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus
  ```
  > [!NOTE]
  > <span data-ttu-id="57412-221">Если вы решили использовать другое имя группы ресурсов (отличное от `ra-hub-rg`), обязательно найдите все файлы параметров, которые используют это имя, и отредактируйте их, чтобы использовать собственное имя группы ресурсов.</span><span class="sxs-lookup"><span data-stu-id="57412-221">If you decide to use a different resource group name (other than `ra-hub-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="57412-222">Дождитесь завершения развертывания.</span><span class="sxs-lookup"><span data-stu-id="57412-222">Wait for the deployment to finish.</span></span> <span data-ttu-id="57412-223">При этом развертывании создается виртуальная сеть, виртуальная машина Ubuntu, VPN-шлюз и подключение к шлюзу, созданному в предыдущем разделе.</span><span class="sxs-lookup"><span data-stu-id="57412-223">This deployment creates a virtual network, a virtual machine running Ubuntu, a VPN gateway, and a connection to the gateway created in the previous section.</span></span> <span data-ttu-id="57412-224">Создание VPN-шлюза может занять более 40 минут.</span><span class="sxs-lookup"><span data-stu-id="57412-224">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="connection-from-on-premises-to-the-hub"></a><span data-ttu-id="57412-225">Подключение к концентратору из локальной среды</span><span class="sxs-lookup"><span data-stu-id="57412-225">Connection from on-premises to the hub</span></span>

<span data-ttu-id="57412-226">Чтобы подключиться из имитации локального центра обработки данных к виртуальной сети концентратора, сделайте следующее.</span><span class="sxs-lookup"><span data-stu-id="57412-226">To connect from the simulated on-premises datacenter to the hub VNet, perform the following steps.</span></span>

1. <span data-ttu-id="57412-227">Перейдите в папку `hybrid-networking\hub-spoke\onprem` репозитория, скачанного на предыдущем шаге.</span><span class="sxs-lookup"><span data-stu-id="57412-227">Navigate to the `hybrid-networking\hub-spoke\onprem` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="57412-228">Откройте файл `onprem.connection.parameters.json` и введите общий ключ в кавычках в строке 9, как показано ниже, а затем сохраните файл.</span><span class="sxs-lookup"><span data-stu-id="57412-228">Open the `onprem.connection.parameters.json` file and enter a shared key between the quotes in line 9, as shown below, then save the file.</span></span> <span data-ttu-id="57412-229">Это общее значение ключа должно быть таким же, как и в локальном шлюзе, который вы ранее развернули.</span><span class="sxs-lookup"><span data-stu-id="57412-229">This shared key value must be the same used in the on-premises gateway you deployed previously.</span></span>

  ```bash
  "sharedKey": "",
  ```

3. <span data-ttu-id="57412-230">Выполните следующую команду PowerShell или bash для развертывания имитации локальной среды в качестве виртуальной сети в Azure.</span><span class="sxs-lookup"><span data-stu-id="57412-230">Run the bash or PowerShell command below to deploy the simulated on-premises environment as a VNet in Azure.</span></span> <span data-ttu-id="57412-231">Замените значения своей подпиской, именем группы ресурсов и регионом Azure.</span><span class="sxs-lookup"><span data-stu-id="57412-231">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./onprem.connection.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-onprem-rg \
    --location westus
  ```

  ```powershell
  ./onprem.connection.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-onprem-rg `
    -Location westus
  ```
  > [!NOTE]
  > <span data-ttu-id="57412-232">Если вы решили использовать другое имя группы ресурсов (отличное от `ra-onprem-rg`), обязательно найдите все файлы параметров, которые используют это имя, и отредактируйте их, чтобы использовать собственное имя группы ресурсов.</span><span class="sxs-lookup"><span data-stu-id="57412-232">If you decide to use a different resource group name (other than `ra-onprem-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

4. <span data-ttu-id="57412-233">Дождитесь завершения развертывания.</span><span class="sxs-lookup"><span data-stu-id="57412-233">Wait for the deployment to finish.</span></span> <span data-ttu-id="57412-234">При этом развертывании создается подключение между виртуальной сетью, используемой для имитации локального центра обработки данных, и виртуальной сетью концентратора.</span><span class="sxs-lookup"><span data-stu-id="57412-234">This deployment creates a connection between the VNet used to simulate an on-premises datacenter, and the hub VNet.</span></span>

### <a name="azure-spoke-vnets"></a><span data-ttu-id="57412-235">Виртуальные сети периферийных зон Azure</span><span class="sxs-lookup"><span data-stu-id="57412-235">Azure spoke VNets</span></span>

<span data-ttu-id="57412-236">Чтобы развернуть виртуальные сети периферийных зон и подключиться к виртуальной сети концентратора, созданной выше, сделайте следующее.</span><span class="sxs-lookup"><span data-stu-id="57412-236">To deploy the spoke VNets, and connect to the hub VNet created above, perform the following steps.</span></span>

1. <span data-ttu-id="57412-237">Перейдите в папку `hybrid-networking\hub-spoke\spokes` репозитория, скачанного на предыдущем шаге.</span><span class="sxs-lookup"><span data-stu-id="57412-237">Switch to the `hybrid-networking\hub-spoke\spokes` folder for the repository you download in the pre-requisites step above.</span></span>

2. <span data-ttu-id="57412-238">Откройте файл `spoke1.web.parameters.json` и введите имя пользователя и пароль в кавычках в строках 53 и 54, как показано ниже, а затем сохраните файл.</span><span class="sxs-lookup"><span data-stu-id="57412-238">Open the `spoke1.web.parameters.json` file and enter a username and password between the quotes in line 53 and 54, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. <span data-ttu-id="57412-239">Повторите предыдущий шаг для файла `spoke2.web.parameters.json`.</span><span class="sxs-lookup"><span data-stu-id="57412-239">Repeat the previous step for file `spoke2.web.parameters.json`.</span></span>

4. <span data-ttu-id="57412-240">Запустите команду bash или PowerShell, указанную ниже, чтобы развернуть первую периферийную зону и подключить ее к концентратору.</span><span class="sxs-lookup"><span data-stu-id="57412-240">Run the bash or PowerShell command below to deploy the first spoke and connect it to the hub.</span></span> <span data-ttu-id="57412-241">Замените значения своей подпиской, именем группы ресурсов и регионом Azure.</span><span class="sxs-lookup"><span data-stu-id="57412-241">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./spoke.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke1-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./spoke.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke1-rg `
    -Location westus `
    -Spoke 1
  ```
  > [!NOTE]
  > <span data-ttu-id="57412-242">Если вы решили использовать другое имя группы ресурсов (отличное от `ra-spoke1-rg`), обязательно найдите все файлы параметров, которые используют это имя, и отредактируйте их, чтобы использовать собственное имя группы ресурсов.</span><span class="sxs-lookup"><span data-stu-id="57412-242">If you decide to use a different resource group name (other than `ra-spoke1-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="57412-243">Дождитесь завершения развертывания.</span><span class="sxs-lookup"><span data-stu-id="57412-243">Wait for the deployment to finish.</span></span> <span data-ttu-id="57412-244">Это развертывание создает виртуальную сеть, подсистемы балансировки нагрузки с тремя виртуальными машинами, работающими под управлением Ubuntu и Apache, и виртуальное пиринговое подключение к концентратору виртуальной сети, созданной в предыдущем разделе.</span><span class="sxs-lookup"><span data-stu-id="57412-244">This deployment creates a virtual network, a load balancer with three virtual machine running Ubuntu and Apache, and a VNet peering connection to the hub VNet created in the previous section.</span></span> <span data-ttu-id="57412-245">Это развертывание может занять более 20 минут.</span><span class="sxs-lookup"><span data-stu-id="57412-245">This deployment may take over 20 minutes.</span></span>

6. <span data-ttu-id="57412-246">Запустите команду bash или PowerShell, указанную ниже, чтобы развернуть первую периферийную зону и подключить ее к концентратору.</span><span class="sxs-lookup"><span data-stu-id="57412-246">Run the bash or PowerShell command below to deploy the first spoke and connect it to the hub.</span></span> <span data-ttu-id="57412-247">Замените значения своей подпиской, именем группы ресурсов и регионом Azure.</span><span class="sxs-lookup"><span data-stu-id="57412-247">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./spoke.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke2-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./spoke.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke2-rg `
    -Location westus `
    -Spoke 2
  ```
  > [!NOTE]
  > <span data-ttu-id="57412-248">Если вы решили использовать другое имя группы ресурсов (отличное от `ra-spoke2-rg`), обязательно найдите все файлы параметров, которые используют это имя, и отредактируйте их, чтобы использовать собственное имя группы ресурсов.</span><span class="sxs-lookup"><span data-stu-id="57412-248">If you decide to use a different resource group name (other than `ra-spoke2-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="57412-249">Дождитесь завершения развертывания.</span><span class="sxs-lookup"><span data-stu-id="57412-249">Wait for the deployment to finish.</span></span> <span data-ttu-id="57412-250">Это развертывание создает виртуальную сеть, подсистемы балансировки нагрузки с тремя виртуальными машинами, работающими под управлением Ubuntu и Apache, и виртуальное пиринговое подключение к концентратору виртуальной сети, созданной в предыдущем разделе.</span><span class="sxs-lookup"><span data-stu-id="57412-250">This deployment creates a virtual network, a load balancer with three virtual machine running Ubuntu and Apache, and a VNet peering connection to the hub VNet created in the previous section.</span></span> <span data-ttu-id="57412-251">Это развертывание может занять более 20 минут.</span><span class="sxs-lookup"><span data-stu-id="57412-251">This deployment may take over 20 minutes.</span></span>

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a><span data-ttu-id="57412-252">Пиринговое подключение виртуальной сети концентратора Azure к виртуальным сетям периферийных зон</span><span class="sxs-lookup"><span data-stu-id="57412-252">Azure hub VNet peering to spoke VNets</span></span>

<span data-ttu-id="57412-253">Чтобы развернуть пиринговые подключения виртуальной сети для виртуальной сети концентратора, сделайте следующее.</span><span class="sxs-lookup"><span data-stu-id="57412-253">To deploy the VNet peering connections for the hub VNet, perform the following steps.</span></span>

1. <span data-ttu-id="57412-254">Перейдите в папку `hybrid-networking\hub-spoke\hub` репозитория, скачанного на предыдущем шаге.</span><span class="sxs-lookup"><span data-stu-id="57412-254">Switch to the `hybrid-networking\hub-spoke\hub` folder for the repository you download in the pre-requisites step above.</span></span>

2. <span data-ttu-id="57412-255">Запустите команду bash или PowerShell, указанную ниже, чтобы развернуть пиринговое подключение к первой периферийной зоне.</span><span class="sxs-lookup"><span data-stu-id="57412-255">Run the bash or PowerShell command below to deploy the peering connection to the first spoke.</span></span> <span data-ttu-id="57412-256">Замените значения своей подпиской, именем группы ресурсов и регионом Azure.</span><span class="sxs-lookup"><span data-stu-id="57412-256">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./hub.peering.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./hub.peering.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus `
    -Spoke 1
  ```

2. <span data-ttu-id="57412-257">Запустите команду bash или PowerShell, указанную ниже, чтобы развернуть пиринговое подключение ко второй периферийной зоне.</span><span class="sxs-lookup"><span data-stu-id="57412-257">Run the bash or PowerShell command below to deploy the peering connection to the second spoke.</span></span> <span data-ttu-id="57412-258">Замените значения своей подпиской, именем группы ресурсов и регионом Azure.</span><span class="sxs-lookup"><span data-stu-id="57412-258">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./hub.peering.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./hub.peering.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus `
    -Spoke 2
  ```

### <a name="test-connectivity"></a><span data-ttu-id="57412-259">Проверка подключения</span><span class="sxs-lookup"><span data-stu-id="57412-259">Test connectivity</span></span>

<span data-ttu-id="57412-260">Чтобы убедиться, что звездообразная топология, подключенная к локальному развертыванию центра обработки данных, работает, сделайте следующее.</span><span class="sxs-lookup"><span data-stu-id="57412-260">To verify that the hub-spoke topology connected to an on-premises datacenter deployment worked, follow these steps.</span></span>

1. <span data-ttu-id="57412-261">На [портале Azure][портал] подключитесь к своей подписке и перейдите к виртуальной машине `ra-onprem-vm1` в группе ресурсов `ra-onprem-rg`.</span><span class="sxs-lookup"><span data-stu-id="57412-261">From the [Azure portal][portal], connect to your subscription, and navigate to the `ra-onprem-vm1` virtual machine in the `ra-onprem-rg` resource group.</span></span>

2. <span data-ttu-id="57412-262">В колонке `Overview` обратите внимание на значение `Public IP address` виртуальной машины.</span><span class="sxs-lookup"><span data-stu-id="57412-262">In the `Overview` blade, note the `Public IP address` for the VM.</span></span>

3. <span data-ttu-id="57412-263">Используйте SSH-клиент для подключения по указанному выше IP-адресу с использованием имени пользователя и пароля, указанных во время развертывания.</span><span class="sxs-lookup"><span data-stu-id="57412-263">Use an SSH client to connect to the IP address you noted above using the user name and password you specified during deployment.</span></span>

4. <span data-ttu-id="57412-264">В командной строке на виртуальной машине, к которой вы подключились, выполните приведенную ниже команду, чтобы проверить подключение локальной виртуальной сети к виртуальной сети первой периферийной зоны.</span><span class="sxs-lookup"><span data-stu-id="57412-264">From the command prompt on the VM you connected to, run the command below to test connectivity from the on-premises VNet to the Spoke1 VNet.</span></span>

  ```bash
  ping 10.1.1.37
  ```

### <a name="add-connectivity-between-spokes"></a><span data-ttu-id="57412-265">Добавление подключения между периферийными зонами</span><span class="sxs-lookup"><span data-stu-id="57412-265">Add connectivity between spokes</span></span>

<span data-ttu-id="57412-266">Если вы хотите, чтобы периферийные зоны подключались друг к другу, разверните определяемые пользователем маршруты для каждой периферийной зоны, которая перенаправляет трафик, предназначенный для других периферийных зон, в шлюз в виртуальной сети концентратора.</span><span class="sxs-lookup"><span data-stu-id="57412-266">If you want to allow spokes to connect to each other, you must deploy UDRs to each spoke that forward traffic destined to other spokes to the gateway in the hub VNet.</span></span> <span data-ttu-id="57412-267">Выполните следующие шаги, чтобы убедиться, что в настоящий момент вы не можете подключиться из одной периферийной зоны к другой, а затем снова разверните определяемые пользователем маршруты и проверьте подключение.</span><span class="sxs-lookup"><span data-stu-id="57412-267">Perform the following steps to verify that currently you are not able to connect from a spoke to another, then deploy the UDRs and test connectivity again.</span></span>

1. <span data-ttu-id="57412-268">Повторите шаги 1–4 выше, если вы больше не подключены к виртуальной машине jumpbox.</span><span class="sxs-lookup"><span data-stu-id="57412-268">Repeat steps 1 to 4 above, if you are not connected to the jumpbox VM any longer.</span></span>

2. <span data-ttu-id="57412-269">Подключитесь к одному из веб-серверов в первой периферийной зоне.</span><span class="sxs-lookup"><span data-stu-id="57412-269">Connect to one of the web servers in spoke 1.</span></span>

  ```bash
  ssh 10.1.1.37
  ```

3. <span data-ttu-id="57412-270">Проверьте подключение между первой и второй периферийными зонами.</span><span class="sxs-lookup"><span data-stu-id="57412-270">Test the connectivity between spoke 1 and spoke 2.</span></span> <span data-ttu-id="57412-271">Оно должно завершиться ошибкой.</span><span class="sxs-lookup"><span data-stu-id="57412-271">It should fail.</span></span>

  ```bash
  ping 10.1.2.37
  ```

4. <span data-ttu-id="57412-272">Перейдите обратно в командную строку компьютера.</span><span class="sxs-lookup"><span data-stu-id="57412-272">Switch back to your computer's command prompt.</span></span>

5. <span data-ttu-id="57412-273">Перейдите в папку `hybrid-networking\hub-spoke\spokes` репозитория, скачанного на предыдущем шаге.</span><span class="sxs-lookup"><span data-stu-id="57412-273">Switch to the `hybrid-networking\hub-spoke\spokes` folder for the repository you downloaded in the pre-requisites step above.</span></span>

6. <span data-ttu-id="57412-274">Запустите команду bash или PowerShell, указанную ниже, чтобы развернуть определяемый пользователем маршрут в первой периферийной зоне.</span><span class="sxs-lookup"><span data-stu-id="57412-274">Run the bash or PowerShell command below to deploy an UDR to the first spoke.</span></span> <span data-ttu-id="57412-275">Замените значения своей подпиской, именем группы ресурсов и регионом Azure.</span><span class="sxs-lookup"><span data-stu-id="57412-275">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./spoke.udr.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke1-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./spoke.udr.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke1-rg `
    -Location westus `
    -Spoke 1
  ```

7. <span data-ttu-id="57412-276">Запустите команду bash или PowerShell, указанную ниже, чтобы развернуть определяемый пользователем маршрут во второй периферийной зоне.</span><span class="sxs-lookup"><span data-stu-id="57412-276">Run the bash or PowerShell command below to deploy an UDR to the second spoke.</span></span> <span data-ttu-id="57412-277">Замените значения своей подпиской, именем группы ресурсов и регионом Azure.</span><span class="sxs-lookup"><span data-stu-id="57412-277">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./spoke.udr.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke2-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./spoke.udr.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke2-rg `
    -Location westus `
    -Spoke 2
  ```

8. <span data-ttu-id="57412-278">Вернитесь в окно терминала SSH.</span><span class="sxs-lookup"><span data-stu-id="57412-278">Switch back to the ssh terminal.</span></span>

9. <span data-ttu-id="57412-279">Проверьте подключение между первой и второй периферийными зонами.</span><span class="sxs-lookup"><span data-stu-id="57412-279">Test the connectivity between spoke 1 and spoke 2.</span></span> <span data-ttu-id="57412-280">Оно должно выполняться успешно.</span><span class="sxs-lookup"><span data-stu-id="57412-280">It should succeed.</span></span>

  ```bash
  ping 10.1.2.37
  ```

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azure-powershell]: /powershell/azure/install-azure-ps?view=azuresmps-3.7.0
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[best-practices-security]: /azure/best-practices-network-securit
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[linux-vm-ra]: ../virtual-machines-linux/index.md
[hybrid-ha]: ./expressroute-vpn-failover.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[vnet-peering]: /azure/virtual-network/virtual-network-peering-overview
[vnet-peering-limit]: /azure/azure-subscription-service-limits#networking-limits
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.azureedge.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/hub-spoke.png "Звездообразная топология в Azure"
[1]: ./images/hub-spoke-gateway-routing.svg "Звездообразная топология в Azure с транзитивной маршрутизацией"
[2]: ./images/hub-spoke-no-gateway-routing.svg "Звездообразная топология в Azure с транзитивной маршрутизацией с использованием виртуального сетевого устройства (NVA)"
[3]: ./images/hub-spokehub-spoke.svg "Вложенная звездообразная топология в Azure"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
