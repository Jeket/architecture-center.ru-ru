---
title: Реализация звездообразной топологии сети в Azure
description: Как реализовывать звездообразную топологию сети в Azure.
author: telmosampaio
ms.date: 04/09/2018
pnp.series.title: Implement a hub-spoke network topology in Azure
pnp.series.prev: expressroute
ms.openlocfilehash: fcdbb7ca8d02745d4d9ab82f0bce79ab378d843c
ms.sourcegitcommit: f6be2825bf2d37dfe25cfab92b9e3973a6b51e16
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/08/2018
ms.locfileid: "48858203"
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a><span data-ttu-id="4e087-103">Реализация звездообразной топологии сети в Azure</span><span class="sxs-lookup"><span data-stu-id="4e087-103">Implement a hub-spoke network topology in Azure</span></span>

<span data-ttu-id="4e087-104">На схеме эталонной архитектуры представлены сведения о том, как реализовать звездообразную топологию в Azure.</span><span class="sxs-lookup"><span data-stu-id="4e087-104">This reference architecture shows how to implement a hub-spoke topology in Azure.</span></span> <span data-ttu-id="4e087-105">*Концентратор* является виртуальной сетью в Azure, которая выступает в качестве центральной точки подключения к вашей локальной сети.</span><span class="sxs-lookup"><span data-stu-id="4e087-105">The *hub* is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="4e087-106">*Периферийные зоны* — это виртуальные сети, которые устанавливают пиринг с концентратором и могут использоваться для изоляции рабочих нагрузок.</span><span class="sxs-lookup"><span data-stu-id="4e087-106">The *spokes* are VNets that peer with the hub, and can be used to isolate workloads.</span></span> <span data-ttu-id="4e087-107">Трафик передается между локальным центром обработки данных и концентратором через подключение ExpressRoute или VPN-шлюз.</span><span class="sxs-lookup"><span data-stu-id="4e087-107">Traffic flows between the on-premises datacenter and the hub through an ExpressRoute or VPN gateway connection.</span></span>  <span data-ttu-id="4e087-108">[**Разверните это решение**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="4e087-108">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="4e087-109">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="4e087-109">![[0]][0]</span></span>

<span data-ttu-id="4e087-110">*Скачайте [файл Visio][visio-download] этой архитектуры*</span><span class="sxs-lookup"><span data-stu-id="4e087-110">*Download a [Visio file][visio-download] of this architecture*</span></span>


<span data-ttu-id="4e087-111">Преимущества этой топологии:</span><span class="sxs-lookup"><span data-stu-id="4e087-111">The benefits of this toplogy include:</span></span>

* <span data-ttu-id="4e087-112">**Сокращение затрат** путем централизации служб, которые могут совместно использоваться несколькими рабочими нагрузками, такими как сетевые виртуальные устройства (NVA) и DNS-серверы в одном расположении.</span><span class="sxs-lookup"><span data-stu-id="4e087-112">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
* <span data-ttu-id="4e087-113">**Превышение лимитов подписки** путем пиринга виртуальных сетей из разных подписок к центральному концентратору.</span><span class="sxs-lookup"><span data-stu-id="4e087-113">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
* <span data-ttu-id="4e087-114">**Разделение областей ответственности** между центральным ИТ-отделом (SecOps, InfraOps) и рабочими нагрузками (DevOps).</span><span class="sxs-lookup"><span data-stu-id="4e087-114">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="4e087-115">Типичные способы использования этой архитектуры:</span><span class="sxs-lookup"><span data-stu-id="4e087-115">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="4e087-116">Рабочая нагрузка, развернутая в разных средах, например в среде для разработки, тестирования и в рабочей среде, для которых требуются общие службы, например DNS, IDS, NTP или AD DS.</span><span class="sxs-lookup"><span data-stu-id="4e087-116">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="4e087-117">Общие службы размещаются в виртуальной сети концентратора, в то время как каждая среда развертывается в периферийной зоне для обеспечения изоляции.</span><span class="sxs-lookup"><span data-stu-id="4e087-117">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
* <span data-ttu-id="4e087-118">Рабочие нагрузки, которые не требуют подключения друг к другу, но требуют доступа к общим службам.</span><span class="sxs-lookup"><span data-stu-id="4e087-118">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
* <span data-ttu-id="4e087-119">Предприятия, которые требуют централизованного контроля над аспектами безопасности, такими как брандмауэр в концентраторе, таком как сеть периметра, и отдельного управления для рабочих нагрузок в каждой периферийной зоне.</span><span class="sxs-lookup"><span data-stu-id="4e087-119">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="4e087-120">Архитектура</span><span class="sxs-lookup"><span data-stu-id="4e087-120">Architecture</span></span>

<span data-ttu-id="4e087-121">Архитектура состоит из следующих компонентов:</span><span class="sxs-lookup"><span data-stu-id="4e087-121">The architecture consists of the following components.</span></span>

* <span data-ttu-id="4e087-122">**Локальная сеть.**</span><span class="sxs-lookup"><span data-stu-id="4e087-122">**On-premises network**.</span></span> <span data-ttu-id="4e087-123">Частная локальная сеть, работающая внутри организации.</span><span class="sxs-lookup"><span data-stu-id="4e087-123">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="4e087-124">**VPN-устройство**.</span><span class="sxs-lookup"><span data-stu-id="4e087-124">**VPN device**.</span></span> <span data-ttu-id="4e087-125">Устройство или служба, предоставляющая возможность внешнего подключения к локальной сети.</span><span class="sxs-lookup"><span data-stu-id="4e087-125">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="4e087-126">VPN-устройство может быть аппаратным устройством или программным решением, таким как служба маршрутизации и удаленного доступа (RRAS) в Windows Server 2012.</span><span class="sxs-lookup"><span data-stu-id="4e087-126">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="4e087-127">Список поддерживаемых VPN-устройств и информацию о настройке выбранных VPN-устройств для подключения к Azure см. в статье [VPN-устройства и параметры IPsec/IKE для подключений типа "сеть — сеть" через VPN-шлюз][vpn-appliance].</span><span class="sxs-lookup"><span data-stu-id="4e087-127">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="4e087-128">**Сетевой шлюз виртуальной сети VPN или шлюз ExpressRoute**.</span><span class="sxs-lookup"><span data-stu-id="4e087-128">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="4e087-129">Шлюз виртуальной сети позволяет виртуальной сети подключаться к VPN-устройству или к каналу ExpressRoute, которые используются для подключения к вашей локальной сети.</span><span class="sxs-lookup"><span data-stu-id="4e087-129">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="4e087-130">Дополнительные сведения см. в статье [Подключение локальной сети к виртуальной сети Microsoft Azure][connect-to-an-Azure-vnet].</span><span class="sxs-lookup"><span data-stu-id="4e087-130">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="4e087-131">В сценариях развертывания для этой эталонной архитектуры используется VPN-шлюз для подключения, а виртуальная сеть в Azure используется для имитации вашей локальной сети.</span><span class="sxs-lookup"><span data-stu-id="4e087-131">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

* <span data-ttu-id="4e087-132">**Виртуальная сеть концентратора**.</span><span class="sxs-lookup"><span data-stu-id="4e087-132">**Hub VNet**.</span></span> <span data-ttu-id="4e087-133">Виртуальная сеть Azure используется в качестве концентратора в звездообразной топологии.</span><span class="sxs-lookup"><span data-stu-id="4e087-133">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="4e087-134">Концентратор является центральной точкой подключения к вашей локальной сети и местом для размещения служб, которые могут использоваться различными рабочими нагрузками, размещенными в виртуальных сетях.</span><span class="sxs-lookup"><span data-stu-id="4e087-134">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

* <span data-ttu-id="4e087-135">**Подсеть шлюза**.</span><span class="sxs-lookup"><span data-stu-id="4e087-135">**Gateway subnet**.</span></span> <span data-ttu-id="4e087-136">Шлюзы виртуальных сетей хранятся в одной подсети.</span><span class="sxs-lookup"><span data-stu-id="4e087-136">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="4e087-137">**Виртуальные сети периферийных зон**.</span><span class="sxs-lookup"><span data-stu-id="4e087-137">**Spoke VNets**.</span></span> <span data-ttu-id="4e087-138">Одна или несколько виртуальных сетей Azure, которые используются в качестве периферийных зон в звездообразной топологии.</span><span class="sxs-lookup"><span data-stu-id="4e087-138">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="4e087-139">Периферийные зоны можно использовать для изоляции рабочих нагрузок в их виртуальных сетях, управляемых отдельно от других периферийных зон.</span><span class="sxs-lookup"><span data-stu-id="4e087-139">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="4e087-140">Каждая рабочая нагрузка может содержать несколько уровней с несколькими подсетями, подключенными через подсистемы балансировки нагрузки Azure.</span><span class="sxs-lookup"><span data-stu-id="4e087-140">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="4e087-141">Дополнительные сведения об инфраструктуре приложений см. в статьях [Запуск рабочих нагрузок на виртуальных машинах Windows][windows-vm-ra] и [Запуск рабочих нагрузок на виртуальной машине Linux][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="4e087-141">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="4e087-142">**Пиринговая связь между виртуальными сетями**.</span><span class="sxs-lookup"><span data-stu-id="4e087-142">**VNet peering**.</span></span> <span data-ttu-id="4e087-143">Две виртуальные сети можно подключить между собой с помощью [пирингового подключения][vnet-peering].</span><span class="sxs-lookup"><span data-stu-id="4e087-143">Two VNets can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="4e087-144">Пиринговые подключения представляют собой нетранзитивные подключения между виртуальными подсетями с низкой задержкой.</span><span class="sxs-lookup"><span data-stu-id="4e087-144">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="4e087-145">После установки пирингового подключения виртуальные сети обмениваются трафиком по магистрали Azure без необходимости использования маршрутизатора.</span><span class="sxs-lookup"><span data-stu-id="4e087-145">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="4e087-146">В звездообразной топологии сети пиринг виртуальной сети используется для подключения концентратора к каждой периферийной зоне.</span><span class="sxs-lookup"><span data-stu-id="4e087-146">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span> <span data-ttu-id="4e087-147">Пиринг можно создать между виртуальными сетями в одном или в разных регионах.</span><span class="sxs-lookup"><span data-stu-id="4e087-147">You can peer virtual networks in the same region, or different regions.</span></span> <span data-ttu-id="4e087-148">Дополнительные сведения см. в разделе [Требования и ограничения][vnet-peering-requirements].</span><span class="sxs-lookup"><span data-stu-id="4e087-148">For more information, see [Requirements and constraints][vnet-peering-requirements].</span></span>

> [!NOTE]
> <span data-ttu-id="4e087-149">В этой статье рассматриваются только [развертывания Resource Manager](/azure/azure-resource-manager/resource-group-overview), но вы также можете подключить классическую виртуальную сеть к виртуальной сети Resource Manager в той же подписке.</span><span class="sxs-lookup"><span data-stu-id="4e087-149">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="4e087-150">Таким образом, ваши периферийные зоны могут размещать классические развертывания и по-прежнему пользоваться преимуществами общих служб в концентраторе.</span><span class="sxs-lookup"><span data-stu-id="4e087-150">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>

## <a name="recommendations"></a><span data-ttu-id="4e087-151">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="4e087-151">Recommendations</span></span>

<span data-ttu-id="4e087-152">Следующие рекомендации применимы для большинства ситуаций.</span><span class="sxs-lookup"><span data-stu-id="4e087-152">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="4e087-153">Следуйте этим рекомендациям, если они не противоречат особым требованиям для вашего случая.</span><span class="sxs-lookup"><span data-stu-id="4e087-153">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="resource-groups"></a><span data-ttu-id="4e087-154">Группы ресурсов</span><span class="sxs-lookup"><span data-stu-id="4e087-154">Resource groups</span></span>

<span data-ttu-id="4e087-155">Центральную виртуальную сеть и каждую периферийную виртуальную сеть, можно реализовать в разных группах ресурсов и даже в разных подписках.</span><span class="sxs-lookup"><span data-stu-id="4e087-155">The hub VNet, and each spoke VNet, can be implemented in different resource groups, and even different subscriptions.</span></span> <span data-ttu-id="4e087-156">Если вы устанавливаете пиринг виртуальных сетей в разных подписках, обе подписки могут быть связаны с одним и тем же или с разными клиентами Azure Active Directory.</span><span class="sxs-lookup"><span data-stu-id="4e087-156">When you peer virtual networks in different subscriptions, both subscriptions can be associated to the same or different Azure Active Directory tenant.</span></span> <span data-ttu-id="4e087-157">Это позволяет осуществлять децентрализованное управление каждой рабочей нагрузкой при совместном использовании служб, поддерживаемых в концентраторе виртуальной сети.</span><span class="sxs-lookup"><span data-stu-id="4e087-157">This allows for a decentralized management of each workload, while sharing services maintained in the hub VNet.</span></span> 

### <a name="vnet-and-gatewaysubnet"></a><span data-ttu-id="4e087-158">Виртуальная сеть и подсеть шлюза</span><span class="sxs-lookup"><span data-stu-id="4e087-158">VNet and GatewaySubnet</span></span>

<span data-ttu-id="4e087-159">Создание подсети с именем *GatewaySubnet* с диапазоном адресов /27.</span><span class="sxs-lookup"><span data-stu-id="4e087-159">Create a subnet named *GatewaySubnet*, with an address range of /27.</span></span> <span data-ttu-id="4e087-160">Эта подсеть необходима для шлюза виртуальной сети.</span><span class="sxs-lookup"><span data-stu-id="4e087-160">This subnet is required by the virtual network gateway.</span></span> <span data-ttu-id="4e087-161">Выделение 32 адресов этой подсети поможет предотвратить достижение ограничения размера шлюза в будущем.</span><span class="sxs-lookup"><span data-stu-id="4e087-161">Allocating 32 addresses to this subnet will help to prevent reaching gateway size limitations in the future.</span></span>

<span data-ttu-id="4e087-162">Дополнительные сведения о настройке шлюза см. в следующих статьях с данными об эталонных архитектурах в зависимости от типа подключения:</span><span class="sxs-lookup"><span data-stu-id="4e087-162">For more information about setting up the gateway, see the following reference architectures, depending on your connection type:</span></span>

- <span data-ttu-id="4e087-163">[Connect an on-premises network to Azure using ExpressRoute][guidance-expressroute] (Подключение локальной сети к Azure с помощью ExpressRoute).</span><span class="sxs-lookup"><span data-stu-id="4e087-163">[Hybrid network using ExpressRoute][guidance-expressroute]</span></span>
- <span data-ttu-id="4e087-164">[Connect an on-premises network to Azure using a VPN gateway][guidance-vpn] (Подключение локальной сети к Azure с помощью VPN-шлюза).</span><span class="sxs-lookup"><span data-stu-id="4e087-164">[Hybrid network using a VPN gateway][guidance-vpn]</span></span>

<span data-ttu-id="4e087-165">Для обеспечения высокой доступности вы можете использовать ExpressRoute вместе с VPN для отработки отказа.</span><span class="sxs-lookup"><span data-stu-id="4e087-165">For higher availability, you can use ExpressRoute plus a VPN for failover.</span></span> <span data-ttu-id="4e087-166">Ознакомьтесь со статьей [Connect an on-premises network to Azure using ExpressRoute with VPN failover][hybrid-ha] (Подключение локальной сети к Azure с помощью ExpressRoute с отработкой отказа VPN).</span><span class="sxs-lookup"><span data-stu-id="4e087-166">See [Connect an on-premises network to Azure using ExpressRoute with VPN failover][hybrid-ha].</span></span>

<span data-ttu-id="4e087-167">Звездообразная топология может также использоваться без шлюза, если вам не требуется связь с локальной сетью.</span><span class="sxs-lookup"><span data-stu-id="4e087-167">A hub-spoke topology can also be used without a gateway, if you don't need connectivity with your on-premises network.</span></span> 

### <a name="vnet-peering"></a><span data-ttu-id="4e087-168">Пиринговая связь между виртуальными сетями</span><span class="sxs-lookup"><span data-stu-id="4e087-168">VNet peering</span></span>

<span data-ttu-id="4e087-169">Пиринговая связь между виртуальными сетями — это нетранзитивная связь между двумя виртуальными сетями.</span><span class="sxs-lookup"><span data-stu-id="4e087-169">VNet peering is a non-transitive relationship between two VNets.</span></span> <span data-ttu-id="4e087-170">Если вам требуется, чтобы периферийные зоны соединялись друг с другом, подумайте над добавлением отдельного пирингового подключения между ними.</span><span class="sxs-lookup"><span data-stu-id="4e087-170">If you require spokes to connect to each other, consider adding a separate peering connection between those spokes.</span></span>

<span data-ttu-id="4e087-171">Однако при наличии нескольких периферийных зон, которые требуется подключить между собой, очень скоро возникнет нехватка пиринговых подключений из-за ограничения [числа пиринговых подключений на каждую виртуальную сеть][vnet-peering-limit].</span><span class="sxs-lookup"><span data-stu-id="4e087-171">However, if you have several spokes that need to connect with each other, you will run out of possible peering connections very quickly due to the [limitation on number of VNets peerings per VNet][vnet-peering-limit].</span></span> <span data-ttu-id="4e087-172">В этом случае рассмотрите возможность использования пользовательских маршрутов (UDR) для принудительной передачи трафика, предназначенного для периферийной зоны, на виртуальное сетевое устройство (NVA), выступающее в роли маршрутизатора в виртуальной сети концентратора.</span><span class="sxs-lookup"><span data-stu-id="4e087-172">In this scenario, consider using user defined routes (UDRs) to force traffic destined to a spoke to be sent to an NVA acting as a router at the hub VNet.</span></span> <span data-ttu-id="4e087-173">Это позволит периферийным зонам подключаться друг к другу.</span><span class="sxs-lookup"><span data-stu-id="4e087-173">This will allow the spokes to connect to each other.</span></span>

<span data-ttu-id="4e087-174">Вы также можете настроить периферийные зоны для использования шлюза виртуальной сети концентратора для подключения к удаленным сетям.</span><span class="sxs-lookup"><span data-stu-id="4e087-174">You can also configure spokes to use the hub VNet gateway to communicate with remote networks.</span></span> <span data-ttu-id="4e087-175">Чтобы разрешить передачу трафика шлюза из периферийной зоны в концентратор и подключение к удаленным сетям, требуется сделать следующее:</span><span class="sxs-lookup"><span data-stu-id="4e087-175">To allow gateway traffic to flow from spoke to hub, and connect to remote networks, you must:</span></span>

  - <span data-ttu-id="4e087-176">Настроить пиринговую связь между виртуальными сетями в концентраторе, чтобы **разрешить транзит шлюзов**.</span><span class="sxs-lookup"><span data-stu-id="4e087-176">Configure the VNet peering connection in the hub to **allow gateway transit**.</span></span>
  - <span data-ttu-id="4e087-177">Настроить пиринговое подключение виртуальной сети к каждой периферийной зоне для **использования удаленных шлюзов**.</span><span class="sxs-lookup"><span data-stu-id="4e087-177">Configure the VNet peering connection in each spoke to **use remote gateways**.</span></span>
  - <span data-ttu-id="4e087-178">Настроить все пиринговые соединения виртуальных сетей, чтобы **разрешить перенаправление трафика**.</span><span class="sxs-lookup"><span data-stu-id="4e087-178">Configure all VNet peering connections to **allow forwarded traffic**.</span></span>

## <a name="considerations"></a><span data-ttu-id="4e087-179">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="4e087-179">Considerations</span></span>

### <a name="spoke-connectivity"></a><span data-ttu-id="4e087-180">Подключение периферийной зоны</span><span class="sxs-lookup"><span data-stu-id="4e087-180">Spoke connectivity</span></span>

<span data-ttu-id="4e087-181">Если вам требуется подключение между периферийными зонами, подумайте о внедрении виртуального сетевого устройства для маршрутизации в концентраторе и использовании определяемых пользователем маршрутов в периферийных зонах для направления трафика в концентратор.</span><span class="sxs-lookup"><span data-stu-id="4e087-181">If you require connectivity between spokes, consider implementing an NVA for routing in the hub, and using UDRs in the spoke to forward traffic to the hub.</span></span>

<span data-ttu-id="4e087-182">![[2]][2]</span><span class="sxs-lookup"><span data-stu-id="4e087-182">![[2]][2]</span></span>

<span data-ttu-id="4e087-183">В этом случае необходимо настроить пиринговые соединения, чтобы **разрешить перенаправленный трафик**.</span><span class="sxs-lookup"><span data-stu-id="4e087-183">In this scenario, you must configure the peering connections to **allow forwarded traffic**.</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="4e087-184">Преодоление ограничения пиринговых подключений виртуальной сети</span><span class="sxs-lookup"><span data-stu-id="4e087-184">Overcoming VNet peering limits</span></span>

<span data-ttu-id="4e087-185">Убедитесь, что вы учли [ограничение числа пиринговых подключений виртуальных сетей на одну виртуальную сеть][vnet-peering-limit] в Azure.</span><span class="sxs-lookup"><span data-stu-id="4e087-185">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="4e087-186">Если вам нужно превысить лимит по периферийным зонам, подумайте о создании вложенной звездообразной топологии, где первый уровень периферийных зон также выступает в качестве концентраторов.</span><span class="sxs-lookup"><span data-stu-id="4e087-186">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="4e087-187">Этот подход показан на схеме ниже.</span><span class="sxs-lookup"><span data-stu-id="4e087-187">The following diagram shows this approach.</span></span>

<span data-ttu-id="4e087-188">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="4e087-188">![[3]][3]</span></span>

<span data-ttu-id="4e087-189">Кроме того, следует учитывать, какие службы используются совместно в концентраторе, чтобы убедиться, что концентратор масштабируется в соответствии с большим количеством периферийных зон.</span><span class="sxs-lookup"><span data-stu-id="4e087-189">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="4e087-190">Например, если ваш концентратор предоставляет службы брандмауэра, рассмотрите возможность ограничения пропускной способности решения брандмауэра при добавлении нескольких периферийных зон.</span><span class="sxs-lookup"><span data-stu-id="4e087-190">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="4e087-191">Возможно, вам захочется перенести некоторые из этих общих служб на второй уровень концентраторов.</span><span class="sxs-lookup"><span data-stu-id="4e087-191">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="4e087-192">Развертывание решения</span><span class="sxs-lookup"><span data-stu-id="4e087-192">Deploy the solution</span></span>

<span data-ttu-id="4e087-193">Пример развертывания для этой архитектуры можно найти на портале [GitHub][ref-arch-repo].</span><span class="sxs-lookup"><span data-stu-id="4e087-193">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="4e087-194">В нем используются виртуальные машины в каждой виртуальной сети для проверки возможности подключения.</span><span class="sxs-lookup"><span data-stu-id="4e087-194">It uses VMs in each VNet to test connectivity.</span></span> <span data-ttu-id="4e087-195">В подсети **shared-services** в **виртуальной сети концентратора** нет настоящих служб.</span><span class="sxs-lookup"><span data-stu-id="4e087-195">There are no actual services hosted in the **shared-services** subnet in the **hub VNet**.</span></span>

<span data-ttu-id="4e087-196">Развертывание создает в вашей подписке следующие группы ресурсов:</span><span class="sxs-lookup"><span data-stu-id="4e087-196">The deployment creates the following resource groups in your subscription:</span></span>

- <span data-ttu-id="4e087-197">hub-nva-rg</span><span class="sxs-lookup"><span data-stu-id="4e087-197">hub-nva-rg</span></span>
- <span data-ttu-id="4e087-198">hub-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="4e087-198">hub-vnet-rg</span></span>
- <span data-ttu-id="4e087-199">onprem-jb-rg</span><span class="sxs-lookup"><span data-stu-id="4e087-199">onprem-jb-rg</span></span>
- <span data-ttu-id="4e087-200">onprem-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="4e087-200">onprem-vnet-rg</span></span>
- <span data-ttu-id="4e087-201">spoke1-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="4e087-201">spoke1-vnet-rg</span></span>
- <span data-ttu-id="4e087-202">spoke2-vent-rg</span><span class="sxs-lookup"><span data-stu-id="4e087-202">spoke2-vent-rg</span></span>

<span data-ttu-id="4e087-203">Файлы параметров шаблона ссылаются на эти имена, поэтому, если вы их изменяете, соответствующим образом обновите файлы параметров.</span><span class="sxs-lookup"><span data-stu-id="4e087-203">The template parameter files refer to these names, so if you change them, update the parameter files to match.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="4e087-204">Предварительные требования</span><span class="sxs-lookup"><span data-stu-id="4e087-204">Prerequisites</span></span>

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-simulated-on-premises-datacenter"></a><span data-ttu-id="4e087-205">Развертывание имитации локального центра обработки данных</span><span class="sxs-lookup"><span data-stu-id="4e087-205">Deploy the simulated on-premises datacenter</span></span>

<span data-ttu-id="4e087-206">Чтобы развернуть имитацию локального центра обработки данных в виртуальной сети Azure, следуйте этим инструкциям:</span><span class="sxs-lookup"><span data-stu-id="4e087-206">To deploy the simulated on-premises datacenter as an Azure VNet, follow these steps:</span></span>

1. <span data-ttu-id="4e087-207">Перейдите в папку `hybrid-networking/hub-spoke` в репозитории эталонных архитектур.</span><span class="sxs-lookup"><span data-stu-id="4e087-207">Navigate to the `hybrid-networking/hub-spoke` folder of the reference architectures repository.</span></span>

2. <span data-ttu-id="4e087-208">Откройте файл `onprem.json` .</span><span class="sxs-lookup"><span data-stu-id="4e087-208">Open the `onprem.json` file.</span></span> <span data-ttu-id="4e087-209">Замените значения для `adminUsername` и `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="4e087-209">Replace the values for `adminUsername` and `adminPassword`.</span></span>

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

3. <span data-ttu-id="4e087-210">Для развертывания Linux установите `osType` как `Linux` (необязательно).</span><span class="sxs-lookup"><span data-stu-id="4e087-210">(Optional) For a Linux deployment, set `osType` to `Linux`.</span></span>

4. <span data-ttu-id="4e087-211">Выполните следующую команду:</span><span class="sxs-lookup"><span data-stu-id="4e087-211">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g onprem-vnet-rg -l <location> -p onprem.json --deploy
    ```

5. <span data-ttu-id="4e087-212">Дождитесь завершения развертывания.</span><span class="sxs-lookup"><span data-stu-id="4e087-212">Wait for the deployment to finish.</span></span> <span data-ttu-id="4e087-213">При развертывании создается виртуальная сеть, виртуальная машина и VPN-шлюз.</span><span class="sxs-lookup"><span data-stu-id="4e087-213">This deployment creates a virtual network, a virtual machine, and a VPN gateway.</span></span> <span data-ttu-id="4e087-214">Для создания VPN-шлюза может потребоваться около 40 минут.</span><span class="sxs-lookup"><span data-stu-id="4e087-214">It can take about 40 minutes to create the VPN gateway.</span></span>

### <a name="deploy-the-hub-vnet"></a><span data-ttu-id="4e087-215">Развертывание концентратора виртуальной сети</span><span class="sxs-lookup"><span data-stu-id="4e087-215">Deploy the hub VNet</span></span>

<span data-ttu-id="4e087-216">Чтобы развернуть концентратор виртуальной сети, выполните следующие действия.</span><span class="sxs-lookup"><span data-stu-id="4e087-216">To deploy the hub VNet, perform the following steps.</span></span>

1. <span data-ttu-id="4e087-217">Откройте файл `hub-vnet.json` .</span><span class="sxs-lookup"><span data-stu-id="4e087-217">Open the `hub-vnet.json` file.</span></span> <span data-ttu-id="4e087-218">Замените значения для `adminUsername` и `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="4e087-218">Replace the values for `adminUsername` and `adminPassword`.</span></span>

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. <span data-ttu-id="4e087-219">Для развертывания Linux установите `osType` как `Linux` (необязательно).</span><span class="sxs-lookup"><span data-stu-id="4e087-219">(Optional) For a Linux deployment, set `osType` to `Linux`.</span></span>

3. <span data-ttu-id="4e087-220">Найдите оба экземпляра `sharedKey` и введите общий ключ для подключения VPN.</span><span class="sxs-lookup"><span data-stu-id="4e087-220">Find both instances of `sharedKey` and enter a shared key for the VPN connection.</span></span> <span data-ttu-id="4e087-221">Значения должны совпадать.</span><span class="sxs-lookup"><span data-stu-id="4e087-221">The values must match.</span></span>

    ```bash
    "sharedKey": "",
    ```

4. <span data-ttu-id="4e087-222">Выполните следующую команду:</span><span class="sxs-lookup"><span data-stu-id="4e087-222">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet.json --deploy
    ```

5. <span data-ttu-id="4e087-223">Дождитесь завершения развертывания.</span><span class="sxs-lookup"><span data-stu-id="4e087-223">Wait for the deployment to finish.</span></span> <span data-ttu-id="4e087-224">В ходе этого развертывания создается виртуальная сеть, виртуальная машина, VPN-шлюз и подключение к шлюзу.</span><span class="sxs-lookup"><span data-stu-id="4e087-224">This deployment creates a virtual network, a virtual machine, a VPN gateway, and a connection to the gateway.</span></span>  <span data-ttu-id="4e087-225">Для создания VPN-шлюза может потребоваться около 40 минут.</span><span class="sxs-lookup"><span data-stu-id="4e087-225">It can take about 40 minutes to create the VPN gateway.</span></span>

### <a name="test-connectivity-with-the-hub"></a><span data-ttu-id="4e087-226">Проверка подключения к концентратору</span><span class="sxs-lookup"><span data-stu-id="4e087-226">Test connectivity with the hub</span></span>

<span data-ttu-id="4e087-227">Протестируйте подключение от моделируемой локальной среды к концентратору виртуальной сети.</span><span class="sxs-lookup"><span data-stu-id="4e087-227">Test conectivity from the simulated on-premises environment to the hub VNet.</span></span>

<span data-ttu-id="4e087-228">**Развертывание ОС Windows**</span><span class="sxs-lookup"><span data-stu-id="4e087-228">**Windows deployment**</span></span>

1. <span data-ttu-id="4e087-229">Используйте портал Azure для поиска в группе ресурсов `onprem-jb-rg` виртуальной машины с именем `jb-vm1`.</span><span class="sxs-lookup"><span data-stu-id="4e087-229">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="4e087-230">Нажмите `Connect`, чтобы открыть сеанс удаленного рабочего стола для виртуальной машины.</span><span class="sxs-lookup"><span data-stu-id="4e087-230">Click `Connect` to open a remote desktop session to the VM.</span></span> <span data-ttu-id="4e087-231">Используйте пароль, указанный в файле параметров `onprem.json`.</span><span class="sxs-lookup"><span data-stu-id="4e087-231">Use the password that you specified in the `onprem.json` parameter file.</span></span>

3. <span data-ttu-id="4e087-232">Откройте консоль PowerShell в виртуальной машине и используйте командлет `Test-NetConnection`, чтобы убедиться, что вы можете подключиться к виртуальной машине Jumpbox в виртуальной сети концентратора.</span><span class="sxs-lookup"><span data-stu-id="4e087-232">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the jumpbox VM in the hub VNet.</span></span>

   ```powershell
   Test-NetConnection 10.0.0.68 -CommonTCPPort RDP
   ```
<span data-ttu-id="4e087-233">Результат должен выглядеть следующим образом.</span><span class="sxs-lookup"><span data-stu-id="4e087-233">The output should look similar to the following:</span></span>

```powershell
ComputerName     : 10.0.0.68
RemoteAddress    : 10.0.0.68
RemotePort       : 3389
InterfaceAlias   : Ethernet 2
SourceAddress    : 192.168.1.000
TcpTestSucceeded : True
```

> [!NOTE]
> <span data-ttu-id="4e087-234">По умолчанию виртуальные машины Windows Server не разрешают трафик ICMP в Azure.</span><span class="sxs-lookup"><span data-stu-id="4e087-234">By default, Windows Server VMs do not allow ICMP responses in Azure.</span></span> <span data-ttu-id="4e087-235">Если вы хотите использовать команду `ping` для проверки возможности подключения, включите трафик ICMP в расширенном брандмауэре Windows для каждой виртуальной машины.</span><span class="sxs-lookup"><span data-stu-id="4e087-235">If you want to use `ping` to test connectivity, you need to enable ICMP traffic in the Windows Advanced Firewall for each VM.</span></span>

<span data-ttu-id="4e087-236">**Развертывание ОС Linux**</span><span class="sxs-lookup"><span data-stu-id="4e087-236">**Linux deployment**</span></span>

1. <span data-ttu-id="4e087-237">Используйте портал Azure для поиска в группе ресурсов `onprem-jb-rg` виртуальной машины с именем `jb-vm1`.</span><span class="sxs-lookup"><span data-stu-id="4e087-237">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="4e087-238">Нажмите `Connect` и скопируйте команду `ssh`, показанную на портале.</span><span class="sxs-lookup"><span data-stu-id="4e087-238">Click `Connect` and copy the `ssh` command shown in the portal.</span></span> 

3. <span data-ttu-id="4e087-239">В командной строке Linux запустите `ssh`, чтобы подключиться к моделируемой локальной среде.</span><span class="sxs-lookup"><span data-stu-id="4e087-239">From a Linux prompt, run `ssh` to connect to the simulated on-premises environment.</span></span> <span data-ttu-id="4e087-240">Используйте пароль, указанный в файле параметров `onprem.json`.</span><span class="sxs-lookup"><span data-stu-id="4e087-240">Use the password that you specified in the `onprem.json` parameter file.</span></span>

4. <span data-ttu-id="4e087-241">Используйте команду `ping` для проверки подключения к виртуальной машине Jumpbox в виртуальной сети концентратора:</span><span class="sxs-lookup"><span data-stu-id="4e087-241">Use the `ping` command to test connectivity to the jumpbox VM in the hub VNet:</span></span>

   ```bash
   ping 10.0.0.68
   ```

### <a name="deploy-the-spoke-vnets"></a><span data-ttu-id="4e087-242">Развертывание виртуальных сетей периферийных зон</span><span class="sxs-lookup"><span data-stu-id="4e087-242">Deploy the spoke VNets</span></span>

<span data-ttu-id="4e087-243">Чтобы развернуть виртуальные сети периферийных зон, сделайте следующее.</span><span class="sxs-lookup"><span data-stu-id="4e087-243">To deploy the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="4e087-244">Откройте файл `spoke1.json` .</span><span class="sxs-lookup"><span data-stu-id="4e087-244">Open the `spoke1.json` file.</span></span> <span data-ttu-id="4e087-245">Замените значения для `adminUsername` и `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="4e087-245">Replace the values for `adminUsername` and `adminPassword`.</span></span>

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. <span data-ttu-id="4e087-246">Для развертывания Linux установите `osType` как `Linux` (необязательно).</span><span class="sxs-lookup"><span data-stu-id="4e087-246">(Optional) For a Linux deployment, set `osType` to `Linux`.</span></span>

3. <span data-ttu-id="4e087-247">Выполните следующую команду:</span><span class="sxs-lookup"><span data-stu-id="4e087-247">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg -l <location> -p spoke1.json --deploy
   ```
  
4. <span data-ttu-id="4e087-248">Повторите шаги 1–2 для файла `spoke2.json`.</span><span class="sxs-lookup"><span data-stu-id="4e087-248">Repeat steps 1-2 for the `spoke2.json` file.</span></span>

5. <span data-ttu-id="4e087-249">Выполните следующую команду:</span><span class="sxs-lookup"><span data-stu-id="4e087-249">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg -l <location> -p spoke2.json --deploy
   ```

6. <span data-ttu-id="4e087-250">Выполните следующую команду:</span><span class="sxs-lookup"><span data-stu-id="4e087-250">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet-peering.json --deploy
   ```

### <a name="test-connectivity"></a><span data-ttu-id="4e087-251">Проверка подключения</span><span class="sxs-lookup"><span data-stu-id="4e087-251">Test connectivity</span></span>

<span data-ttu-id="4e087-252">Протестируйте подключение от моделируемой локальной среды к виртуальным сетям периферийных зон.</span><span class="sxs-lookup"><span data-stu-id="4e087-252">Test conectivity from the simulated on-premises environment to the spoke VNets.</span></span>

<span data-ttu-id="4e087-253">**Развертывание ОС Windows**</span><span class="sxs-lookup"><span data-stu-id="4e087-253">**Windows deployment**</span></span>

1. <span data-ttu-id="4e087-254">Используйте портал Azure для поиска в группе ресурсов `onprem-jb-rg` виртуальной машины с именем `jb-vm1`.</span><span class="sxs-lookup"><span data-stu-id="4e087-254">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="4e087-255">Нажмите `Connect`, чтобы открыть сеанс удаленного рабочего стола для виртуальной машины.</span><span class="sxs-lookup"><span data-stu-id="4e087-255">Click `Connect` to open a remote desktop session to the VM.</span></span> <span data-ttu-id="4e087-256">Используйте пароль, указанный в файле параметров `onprem.json`.</span><span class="sxs-lookup"><span data-stu-id="4e087-256">Use the password that you specified in the `onprem.json` parameter file.</span></span>

3. <span data-ttu-id="4e087-257">Откройте консоль PowerShell в виртуальной машине и используйте командлет `Test-NetConnection`, чтобы проверить возможность подключения к виртуальным машинам Jumpbox в виртуальных сетях периферийных зон.</span><span class="sxs-lookup"><span data-stu-id="4e087-257">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the jumpbox VMs in the spoke VNets.</span></span>

   ```powershell
   Test-NetConnection 10.1.0.68 -CommonTCPPort RDP
   Test-NetConnection 10.2.0.68 -CommonTCPPort RDP
   ```

<span data-ttu-id="4e087-258">**Развертывание ОС Linux**</span><span class="sxs-lookup"><span data-stu-id="4e087-258">**Linux deployment**</span></span>

<span data-ttu-id="4e087-259">Чтобы проверить возможность подключения из имитированной локальной среды к виртуальной сети периферийных зон с помощью виртуальных машин Linux, сделайте следующее:</span><span class="sxs-lookup"><span data-stu-id="4e087-259">To test conectivity from the simulated on-premises environment to the spoke VNets using Linux VMs, perform the following steps:</span></span>

1. <span data-ttu-id="4e087-260">Используйте портал Azure для поиска в группе ресурсов `onprem-jb-rg` виртуальной машины с именем `jb-vm1`.</span><span class="sxs-lookup"><span data-stu-id="4e087-260">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="4e087-261">Нажмите `Connect` и скопируйте команду `ssh`, показанную на портале.</span><span class="sxs-lookup"><span data-stu-id="4e087-261">Click `Connect` and copy the `ssh` command shown in the portal.</span></span> 

3. <span data-ttu-id="4e087-262">В командной строке Linux запустите `ssh`, чтобы подключиться к моделируемой локальной среде.</span><span class="sxs-lookup"><span data-stu-id="4e087-262">From a Linux prompt, run `ssh` to connect to the simulated on-premises environment.</span></span> <span data-ttu-id="4e087-263">Используйте пароль, указанный в файле параметров `onprem.json`.</span><span class="sxs-lookup"><span data-stu-id="4e087-263">Use the password that you specified in the `onprem.json` parameter file.</span></span>

5. <span data-ttu-id="4e087-264">Используйте команду `ping`, чтобы проверить возможность подключения к виртуальным машинам jumpbox в каждой периферийной зоне.</span><span class="sxs-lookup"><span data-stu-id="4e087-264">Use the `ping` command to test connectivity to the jumpbox VMs in each spoke:</span></span>

   ```bash
   ping 10.1.0.68
   ping 10.2.0.68
   ```

### <a name="add-connectivity-between-spokes"></a><span data-ttu-id="4e087-265">Добавление подключения между периферийными зонами</span><span class="sxs-lookup"><span data-stu-id="4e087-265">Add connectivity between spokes</span></span>

<span data-ttu-id="4e087-266">Этот шаг не является обязательным.</span><span class="sxs-lookup"><span data-stu-id="4e087-266">This step is optional.</span></span> <span data-ttu-id="4e087-267">Чтобы включить возможность подключения между периферийными зонами, используйте виртуальный сетевой модуль в качестве маршрутизатора виртуальной сети концентратора и принудительно отправляйте трафик из периферийных зон на маршрутизатор при попытке подключения к другой периферийной зоне.</span><span class="sxs-lookup"><span data-stu-id="4e087-267">If you want to allow spokes to connect to each other, you must use a network virtual appliance (NVA) as a router in the hub VNet, and force traffic from spokes to the router when trying to connect to another spoke.</span></span> <span data-ttu-id="4e087-268">Чтобы развернуть базовый пример виртуального сетевого модуля в качестве одной виртуальной машины и разрешить определяемым пользователем маршрутам соединять две виртуальные машины периферийных зон, сделайте следующее:</span><span class="sxs-lookup"><span data-stu-id="4e087-268">To deploy a basic sample NVA as a single VM, along with user-defined routes (UDRs) to allow the two spoke VNets to connect, perform the following steps:</span></span>

1. <span data-ttu-id="4e087-269">Откройте файл `hub-nva.json` .</span><span class="sxs-lookup"><span data-stu-id="4e087-269">Open the `hub-nva.json` file.</span></span> <span data-ttu-id="4e087-270">Замените значения для `adminUsername` и `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="4e087-270">Replace the values for `adminUsername` and `adminPassword`.</span></span>

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. <span data-ttu-id="4e087-271">Выполните следующую команду:</span><span class="sxs-lookup"><span data-stu-id="4e087-271">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-nva-rg -l <location> -p hub-nva.json --deploy
   ```

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
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
[vnet-peering-requirements]: /azure/virtual-network/virtual-network-manage-peering#requirements-and-constraints
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/hub-spoke.png "Звездообразная топология в Azure"
[1]: ./images/hub-spoke-gateway-routing.svg "Звездообразная топология в Azure с транзитивной маршрутизацией"
[2]: ./images/hub-spoke-no-gateway-routing.svg "Звездообразная топология в Azure с транзитивной маршрутизацией с использованием виртуального сетевого устройства (NVA)"
[3]: ./images/hub-spokehub-spoke.svg "Вложенная звездообразная топология в Azure"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
