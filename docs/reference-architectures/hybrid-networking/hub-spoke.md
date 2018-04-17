---
title: Реализация звездообразной топологии сети в Azure
description: Как реализовывать звездообразную топологию сети в Azure.
author: telmosampaio
ms.date: 02/23/2018
pnp.series.title: Implement a hub-spoke network topology in Azure
pnp.series.prev: expressroute
ms.openlocfilehash: 243ad026c7c9703d9659cbef6815131fcdaa8a11
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2018
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a><span data-ttu-id="0f715-103">Реализация звездообразной топологии сети в Azure</span><span class="sxs-lookup"><span data-stu-id="0f715-103">Implement a hub-spoke network topology in Azure</span></span>

<span data-ttu-id="0f715-104">На схеме эталонной архитектуры представлены сведения о том, как реализовать звездообразную топологию в Azure.</span><span class="sxs-lookup"><span data-stu-id="0f715-104">This reference architecture shows how to implement a hub-spoke topology in Azure.</span></span> <span data-ttu-id="0f715-105">*Концентратор* является виртуальной сетью в Azure, которая выступает в качестве центральной точки подключения к вашей локальной сети.</span><span class="sxs-lookup"><span data-stu-id="0f715-105">The *hub* is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="0f715-106">*Периферийные зоны* — это виртуальные сети, которые устанавливают пиринг с концентратором и могут использоваться для изоляции рабочих нагрузок.</span><span class="sxs-lookup"><span data-stu-id="0f715-106">The *spokes* are VNets that peer with the hub, and can be used to isolate workloads.</span></span> <span data-ttu-id="0f715-107">Трафик передается между локальным центром обработки данных и концентратором через подключение ExpressRoute или VPN-шлюз.</span><span class="sxs-lookup"><span data-stu-id="0f715-107">Traffic flows between the on-premises datacenter and the hub through an ExpressRoute or VPN gateway connection.</span></span>  <span data-ttu-id="0f715-108">[**Разверните это решение**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="0f715-108">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="0f715-109">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="0f715-109">![[0]][0]</span></span>

<span data-ttu-id="0f715-110">*Скачайте [файл Visio][visio-download] этой архитектуры*</span><span class="sxs-lookup"><span data-stu-id="0f715-110">*Download a [Visio file][visio-download] of this architecture*</span></span>


<span data-ttu-id="0f715-111">Преимущества этой топологии:</span><span class="sxs-lookup"><span data-stu-id="0f715-111">The benefits of this toplogy include:</span></span>

* <span data-ttu-id="0f715-112">**Сокращение затрат** путем централизации служб, которые могут совместно использоваться несколькими рабочими нагрузками, такими как сетевые виртуальные устройства (NVA) и DNS-серверы в одном расположении.</span><span class="sxs-lookup"><span data-stu-id="0f715-112">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
* <span data-ttu-id="0f715-113">**Превышение лимитов подписки** путем пиринга виртуальных сетей из разных подписок к центральному концентратору.</span><span class="sxs-lookup"><span data-stu-id="0f715-113">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
* <span data-ttu-id="0f715-114">**Разделение областей ответственности** между центральным ИТ-отделом (SecOps, InfraOps) и рабочими нагрузками (DevOps).</span><span class="sxs-lookup"><span data-stu-id="0f715-114">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="0f715-115">Типичные способы использования этой архитектуры:</span><span class="sxs-lookup"><span data-stu-id="0f715-115">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="0f715-116">Рабочая нагрузка, развернутая в разных средах, например в среде для разработки, тестирования и в рабочей среде, для которых требуются общие службы, например DNS, IDS, NTP или AD DS.</span><span class="sxs-lookup"><span data-stu-id="0f715-116">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="0f715-117">Общие службы размещаются в виртуальной сети концентратора, в то время как каждая среда развертывается в периферийной зоне для обеспечения изоляции.</span><span class="sxs-lookup"><span data-stu-id="0f715-117">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
* <span data-ttu-id="0f715-118">Рабочие нагрузки, которые не требуют подключения друг к другу, но требуют доступа к общим службам.</span><span class="sxs-lookup"><span data-stu-id="0f715-118">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
* <span data-ttu-id="0f715-119">Предприятия, которые требуют централизованного контроля над аспектами безопасности, такими как брандмауэр в концентраторе, таком как сеть периметра, и отдельного управления для рабочих нагрузок в каждой периферийной зоне.</span><span class="sxs-lookup"><span data-stu-id="0f715-119">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="0f715-120">Архитектура</span><span class="sxs-lookup"><span data-stu-id="0f715-120">Architecture</span></span>

<span data-ttu-id="0f715-121">Архитектура состоит из следующих компонентов:</span><span class="sxs-lookup"><span data-stu-id="0f715-121">The architecture consists of the following components.</span></span>

* <span data-ttu-id="0f715-122">**Локальная сеть.**</span><span class="sxs-lookup"><span data-stu-id="0f715-122">**On-premises network**.</span></span> <span data-ttu-id="0f715-123">Частная локальная сеть, работающая внутри организации.</span><span class="sxs-lookup"><span data-stu-id="0f715-123">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="0f715-124">**VPN-устройство**.</span><span class="sxs-lookup"><span data-stu-id="0f715-124">**VPN device**.</span></span> <span data-ttu-id="0f715-125">Устройство или служба, предоставляющая возможность внешнего подключения к локальной сети.</span><span class="sxs-lookup"><span data-stu-id="0f715-125">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="0f715-126">VPN-устройство может быть аппаратным устройством или программным решением, таким как служба маршрутизации и удаленного доступа (RRAS) в Windows Server 2012.</span><span class="sxs-lookup"><span data-stu-id="0f715-126">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="0f715-127">Список поддерживаемых VPN-устройств и информацию о настройке выбранных VPN-устройств для подключения к Azure см. в статье [VPN-устройства и параметры IPsec/IKE для подключений типа "сеть — сеть" через VPN-шлюз][vpn-appliance].</span><span class="sxs-lookup"><span data-stu-id="0f715-127">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="0f715-128">**Сетевой шлюз виртуальной сети VPN или шлюз ExpressRoute**.</span><span class="sxs-lookup"><span data-stu-id="0f715-128">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="0f715-129">Шлюз виртуальной сети позволяет виртуальной сети подключаться к VPN-устройству или к каналу ExpressRoute, которые используются для подключения к вашей локальной сети.</span><span class="sxs-lookup"><span data-stu-id="0f715-129">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="0f715-130">Дополнительные сведения см. в статье [Подключение локальной сети к виртуальной сети Microsoft Azure][connect-to-an-Azure-vnet].</span><span class="sxs-lookup"><span data-stu-id="0f715-130">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="0f715-131">В сценариях развертывания для этой эталонной архитектуры используется VPN-шлюз для подключения, а виртуальная сеть в Azure используется для имитации вашей локальной сети.</span><span class="sxs-lookup"><span data-stu-id="0f715-131">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

* <span data-ttu-id="0f715-132">**Виртуальная сеть концентратора**.</span><span class="sxs-lookup"><span data-stu-id="0f715-132">**Hub VNet**.</span></span> <span data-ttu-id="0f715-133">Виртуальная сеть Azure используется в качестве концентратора в звездообразной топологии.</span><span class="sxs-lookup"><span data-stu-id="0f715-133">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="0f715-134">Концентратор является центральной точкой подключения к вашей локальной сети и местом для размещения служб, которые могут использоваться различными рабочими нагрузками, размещенными в виртуальных сетях.</span><span class="sxs-lookup"><span data-stu-id="0f715-134">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

* <span data-ttu-id="0f715-135">**Подсеть шлюза**.</span><span class="sxs-lookup"><span data-stu-id="0f715-135">**Gateway subnet**.</span></span> <span data-ttu-id="0f715-136">Шлюзы виртуальных сетей хранятся в одной подсети.</span><span class="sxs-lookup"><span data-stu-id="0f715-136">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="0f715-137">**Виртуальные сети периферийных зон**.</span><span class="sxs-lookup"><span data-stu-id="0f715-137">**Spoke VNets**.</span></span> <span data-ttu-id="0f715-138">Одна или несколько виртуальных сетей Azure, которые используются в качестве периферийных зон в звездообразной топологии.</span><span class="sxs-lookup"><span data-stu-id="0f715-138">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="0f715-139">Периферийные зоны можно использовать для изоляции рабочих нагрузок в их виртуальных сетях, управляемых отдельно от других периферийных зон.</span><span class="sxs-lookup"><span data-stu-id="0f715-139">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="0f715-140">Каждая рабочая нагрузка может содержать несколько уровней с несколькими подсетями, подключенными через подсистемы балансировки нагрузки Azure.</span><span class="sxs-lookup"><span data-stu-id="0f715-140">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="0f715-141">Дополнительные сведения об инфраструктуре приложений см. в статьях [Запуск рабочих нагрузок на виртуальных машинах Windows][windows-vm-ra] и [Запуск рабочих нагрузок на виртуальной машине Linux][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="0f715-141">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="0f715-142">**Пиринговая связь между виртуальными сетями**.</span><span class="sxs-lookup"><span data-stu-id="0f715-142">**VNet peering**.</span></span> <span data-ttu-id="0f715-143">Две виртуальные сети в одном регионе Azure можно подключить с помощью [пиринга][vnet-peering].</span><span class="sxs-lookup"><span data-stu-id="0f715-143">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="0f715-144">Пиринговые подключения представляют собой нетранзитивные подключения между виртуальными подсетями с низкой задержкой.</span><span class="sxs-lookup"><span data-stu-id="0f715-144">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="0f715-145">После установки пирингового подключения виртуальные сети обмениваются трафиком по магистрали Azure без необходимости использования маршрутизатора.</span><span class="sxs-lookup"><span data-stu-id="0f715-145">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="0f715-146">В звездообразной топологии сети пиринг виртуальной сети используется для подключения концентратора к каждой периферийной зоне.</span><span class="sxs-lookup"><span data-stu-id="0f715-146">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="0f715-147">В этой статье рассматриваются только [развертывания Resource Manager](/azure/azure-resource-manager/resource-group-overview), но вы также можете подключить классическую виртуальную сеть к виртуальной сети Resource Manager в той же подписке.</span><span class="sxs-lookup"><span data-stu-id="0f715-147">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="0f715-148">Таким образом, ваши периферийные зоны могут размещать классические развертывания и по-прежнему пользоваться преимуществами общих служб в концентраторе.</span><span class="sxs-lookup"><span data-stu-id="0f715-148">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>

## <a name="recommendations"></a><span data-ttu-id="0f715-149">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="0f715-149">Recommendations</span></span>

<span data-ttu-id="0f715-150">Следующие рекомендации применимы для большинства ситуаций.</span><span class="sxs-lookup"><span data-stu-id="0f715-150">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="0f715-151">Следуйте этим рекомендациям, если они не противоречат особым требованиям для вашего случая.</span><span class="sxs-lookup"><span data-stu-id="0f715-151">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="resource-groups"></a><span data-ttu-id="0f715-152">Группы ресурсов</span><span class="sxs-lookup"><span data-stu-id="0f715-152">Resource groups</span></span>

<span data-ttu-id="0f715-153">Концентратор виртуальной сети и каждая периферийная зона виртуальной сети могут быть реализованы в разных группах ресурсов и даже в разных подписках, если они принадлежат одному и тому же клиенту Azure Active Directory (Azure AD) в одном регионе Azure.</span><span class="sxs-lookup"><span data-stu-id="0f715-153">The hub VNet, and each spoke VNet, can be implemented in different resource groups, and even different subscriptions, as long as they belong to the same Azure Active Directory (Azure AD) tenant in the same Azure region.</span></span> <span data-ttu-id="0f715-154">Это позволяет осуществлять децентрализованное управление каждой рабочей нагрузкой при совместном использовании служб, поддерживаемых в концентраторе виртуальной сети.</span><span class="sxs-lookup"><span data-stu-id="0f715-154">This allows for a decentralized management of each workload, while sharing services maintained in the hub VNet.</span></span>

### <a name="vnet-and-gatewaysubnet"></a><span data-ttu-id="0f715-155">Виртуальная сеть и подсеть шлюза</span><span class="sxs-lookup"><span data-stu-id="0f715-155">VNet and GatewaySubnet</span></span>

<span data-ttu-id="0f715-156">Создание подсети с именем *GatewaySubnet* с диапазоном адресов /27.</span><span class="sxs-lookup"><span data-stu-id="0f715-156">Create a subnet named *GatewaySubnet*, with an address range of /27.</span></span> <span data-ttu-id="0f715-157">Эта подсеть необходима для шлюза виртуальной сети.</span><span class="sxs-lookup"><span data-stu-id="0f715-157">This subnet is required by the virtual network gateway.</span></span> <span data-ttu-id="0f715-158">Выделение 32 адресов этой подсети поможет предотвратить достижение ограничения размера шлюза в будущем.</span><span class="sxs-lookup"><span data-stu-id="0f715-158">Allocating 32 addresses to this subnet will help to prevent reaching gateway size limitations in the future.</span></span>

<span data-ttu-id="0f715-159">Дополнительные сведения о настройке шлюза см. в следующих статьях с данными об эталонных архитектурах в зависимости от типа подключения:</span><span class="sxs-lookup"><span data-stu-id="0f715-159">For more information about setting up the gateway, see the following reference architectures, depending on your connection type:</span></span>

- <span data-ttu-id="0f715-160">[Connect an on-premises network to Azure using ExpressRoute][guidance-expressroute] (Подключение локальной сети к Azure с помощью ExpressRoute).</span><span class="sxs-lookup"><span data-stu-id="0f715-160">[Hybrid network using ExpressRoute][guidance-expressroute]</span></span>
- <span data-ttu-id="0f715-161">[Connect an on-premises network to Azure using a VPN gateway][guidance-vpn] (Подключение локальной сети к Azure с помощью VPN-шлюза).</span><span class="sxs-lookup"><span data-stu-id="0f715-161">[Hybrid network using a VPN gateway][guidance-vpn]</span></span>

<span data-ttu-id="0f715-162">Для обеспечения высокой доступности вы можете использовать ExpressRoute вместе с VPN для отработки отказа.</span><span class="sxs-lookup"><span data-stu-id="0f715-162">For higher availability, you can use ExpressRoute plus a VPN for failover.</span></span> <span data-ttu-id="0f715-163">Ознакомьтесь со статьей [Connect an on-premises network to Azure using ExpressRoute with VPN failover][hybrid-ha] (Подключение локальной сети к Azure с помощью ExpressRoute с отработкой отказа VPN).</span><span class="sxs-lookup"><span data-stu-id="0f715-163">See [Connect an on-premises network to Azure using ExpressRoute with VPN failover][hybrid-ha].</span></span>

<span data-ttu-id="0f715-164">Звездообразная топология может также использоваться без шлюза, если вам не требуется связь с локальной сетью.</span><span class="sxs-lookup"><span data-stu-id="0f715-164">A hub-spoke topology can also be used without a gateway, if you don't need connectivity with your on-premises network.</span></span> 

### <a name="vnet-peering"></a><span data-ttu-id="0f715-165">Пиринговая связь между виртуальными сетями</span><span class="sxs-lookup"><span data-stu-id="0f715-165">VNet peering</span></span>

<span data-ttu-id="0f715-166">Пиринговая связь между виртуальными сетями — это нетранзитивная связь между двумя виртуальными сетями.</span><span class="sxs-lookup"><span data-stu-id="0f715-166">VNet peering is a non-transitive relationship between two VNets.</span></span> <span data-ttu-id="0f715-167">Если вам требуется, чтобы периферийные зоны соединялись друг с другом, подумайте над добавлением отдельного пирингового подключения между ними.</span><span class="sxs-lookup"><span data-stu-id="0f715-167">If you require spokes to connect to each other, consider adding a separate peering connection between those spokes.</span></span>

<span data-ttu-id="0f715-168">Однако при наличии нескольких периферийных зон, которые требуется подключить между собой, очень скоро возникнет нехватка пиринговых подключений из-за ограничения [числа пиринговых подключений на каждую виртуальную сеть][vnet-peering-limit].</span><span class="sxs-lookup"><span data-stu-id="0f715-168">However, if you have several spokes that need to connect with each other, you will run out of possible peering connections very quickly due to the [limitation on number of VNets peerings per VNet][vnet-peering-limit].</span></span> <span data-ttu-id="0f715-169">В этом случае рассмотрите возможность использования пользовательских маршрутов (UDR) для принудительной передачи трафика, предназначенного для периферийной зоны, на виртуальное сетевое устройство (NVA), выступающее в роли маршрутизатора в виртуальной сети концентратора.</span><span class="sxs-lookup"><span data-stu-id="0f715-169">In this scenario, consider using user defined routes (UDRs) to force traffic destined to a spoke to be sent to an NVA acting as a router at the hub VNet.</span></span> <span data-ttu-id="0f715-170">Это позволит периферийным зонам подключаться друг к другу.</span><span class="sxs-lookup"><span data-stu-id="0f715-170">This will allow the spokes to connect to each other.</span></span>

<span data-ttu-id="0f715-171">Вы также можете настроить периферийные зоны для использования шлюза виртуальной сети концентратора для подключения к удаленным сетям.</span><span class="sxs-lookup"><span data-stu-id="0f715-171">You can also configure spokes to use the hub VNet gateway to communicate with remote networks.</span></span> <span data-ttu-id="0f715-172">Чтобы разрешить передачу трафика шлюза из периферийной зоны в концентратор и подключение к удаленным сетям, требуется сделать следующее:</span><span class="sxs-lookup"><span data-stu-id="0f715-172">To allow gateway traffic to flow from spoke to hub, and connect to remote networks, you must:</span></span>

  - <span data-ttu-id="0f715-173">Настроить пиринговую связь между виртуальными сетями в концентраторе, чтобы **разрешить транзит шлюзов**.</span><span class="sxs-lookup"><span data-stu-id="0f715-173">Configure the VNet peering connection in the hub to **allow gateway transit**.</span></span>
  - <span data-ttu-id="0f715-174">Настроить пиринговое подключение виртуальной сети к каждой периферийной зоне для **использования удаленных шлюзов**.</span><span class="sxs-lookup"><span data-stu-id="0f715-174">Configure the VNet peering connection in each spoke to **use remote gateways**.</span></span>
  - <span data-ttu-id="0f715-175">Настроить все пиринговые соединения виртуальных сетей, чтобы **разрешить перенаправление трафика**.</span><span class="sxs-lookup"><span data-stu-id="0f715-175">Configure all VNet peering connections to **allow forwarded traffic**.</span></span>

## <a name="considerations"></a><span data-ttu-id="0f715-176">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="0f715-176">Considerations</span></span>

### <a name="spoke-connectivity"></a><span data-ttu-id="0f715-177">Подключение периферийной зоны</span><span class="sxs-lookup"><span data-stu-id="0f715-177">Spoke connectivity</span></span>

<span data-ttu-id="0f715-178">Если вам требуется подключение между периферийными зонами, подумайте о внедрении виртуального сетевого устройства для маршрутизации в концентраторе и использовании определяемых пользователем маршрутов в периферийных зонах для направления трафика в концентратор.</span><span class="sxs-lookup"><span data-stu-id="0f715-178">If you require connectivity between spokes, consider implementing an NVA for routing in the hub, and using UDRs in the spoke to forward traffic to the hub.</span></span>

<span data-ttu-id="0f715-179">![[2]][2]</span><span class="sxs-lookup"><span data-stu-id="0f715-179">![[2]][2]</span></span>

<span data-ttu-id="0f715-180">В этом случае необходимо настроить пиринговые соединения, чтобы **разрешить перенаправленный трафик**.</span><span class="sxs-lookup"><span data-stu-id="0f715-180">In this scenario, you must configure the peering connections to **allow forwarded traffic**.</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="0f715-181">Преодоление ограничения пиринговых подключений виртуальной сети</span><span class="sxs-lookup"><span data-stu-id="0f715-181">Overcoming VNet peering limits</span></span>

<span data-ttu-id="0f715-182">Убедитесь, что вы учли [ограничение числа пиринговых подключений виртуальных сетей на одну виртуальную сеть][vnet-peering-limit] в Azure.</span><span class="sxs-lookup"><span data-stu-id="0f715-182">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="0f715-183">Если вам нужно превысить лимит по периферийным зонам, подумайте о создании вложенной звездообразной топологии, где первый уровень периферийных зон также выступает в качестве концентраторов.</span><span class="sxs-lookup"><span data-stu-id="0f715-183">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="0f715-184">Этот подход показан на схеме ниже.</span><span class="sxs-lookup"><span data-stu-id="0f715-184">The following diagram shows this approach.</span></span>

<span data-ttu-id="0f715-185">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="0f715-185">![[3]][3]</span></span>

<span data-ttu-id="0f715-186">Кроме того, следует учитывать, какие службы используются совместно в концентраторе, чтобы убедиться, что концентратор масштабируется в соответствии с большим количеством периферийных зон.</span><span class="sxs-lookup"><span data-stu-id="0f715-186">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="0f715-187">Например, если ваш концентратор предоставляет службы брандмауэра, рассмотрите возможность ограничения пропускной способности решения брандмауэра при добавлении нескольких периферийных зон.</span><span class="sxs-lookup"><span data-stu-id="0f715-187">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="0f715-188">Возможно, вам захочется перенести некоторые из этих общих служб на второй уровень концентраторов.</span><span class="sxs-lookup"><span data-stu-id="0f715-188">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="0f715-189">Развертывание решения</span><span class="sxs-lookup"><span data-stu-id="0f715-189">Deploy the solution</span></span>

<span data-ttu-id="0f715-190">Пример развертывания для этой архитектуры можно найти на портале [GitHub][ref-arch-repo].</span><span class="sxs-lookup"><span data-stu-id="0f715-190">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="0f715-191">В нем используются виртуальные машины Ubuntu в каждой виртуальной сети для проверки возможности подключения.</span><span class="sxs-lookup"><span data-stu-id="0f715-191">It uses Ubuntu VMs in each VNet to test connectivity.</span></span> <span data-ttu-id="0f715-192">В подсети **shared-services** в **виртуальной сети концентратора** нет настоящих служб.</span><span class="sxs-lookup"><span data-stu-id="0f715-192">There are no actual services hosted in the **shared-services** subnet in the **hub VNet**.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="0f715-193">предварительным требованиям</span><span class="sxs-lookup"><span data-stu-id="0f715-193">Prerequisites</span></span>

<span data-ttu-id="0f715-194">Перед развертыванием эталонной архитектуры в собственной подписке необходимо выполнить следующие действия.</span><span class="sxs-lookup"><span data-stu-id="0f715-194">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="0f715-195">Клонируйте или скачайте ZIP-файл [с эталонными архитектурами][ref-arch-repo] в репозитории GitHub либо создайте для него вилку.</span><span class="sxs-lookup"><span data-stu-id="0f715-195">Clone, fork, or download the zip file for the [reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="0f715-196">Убедитесь, что Azure CLI 2.0 установлен на компьютере.</span><span class="sxs-lookup"><span data-stu-id="0f715-196">Make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="0f715-197">См. инструкции по [установке Azure CLI 2.0][azure-cli-2].</span><span class="sxs-lookup"><span data-stu-id="0f715-197">For CLI installation instructions, see [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="0f715-198">Установите пакет npm [стандартных блоков Azure][azbb].</span><span class="sxs-lookup"><span data-stu-id="0f715-198">Install the [Azure buulding blocks][azbb] npm package.</span></span>

4. <span data-ttu-id="0f715-199">Из командной строки, строки bash или PowerShell войдите в свою учетную запись Azure с помощью приведенной ниже команды и следуйте инструкциям.</span><span class="sxs-lookup"><span data-stu-id="0f715-199">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using the command below, and follow the prompts.</span></span>

   ```bash
   az login
   ```

### <a name="deploy-the-simulated-on-premises-datacenter-using-azbb"></a><span data-ttu-id="0f715-200">Развертывание имитации локального центра обработки данных с помощью azbb</span><span class="sxs-lookup"><span data-stu-id="0f715-200">Deploy the simulated on-premises datacenter using azbb</span></span>

<span data-ttu-id="0f715-201">Чтобы развернуть имитацию локального центра обработки данных в виртуальной сети Azure, следуйте этим инструкциям:</span><span class="sxs-lookup"><span data-stu-id="0f715-201">To deploy the simulated on-premises datacenter as an Azure VNet, follow these steps:</span></span>

1. <span data-ttu-id="0f715-202">Перейдите в папку `hybrid-networking\hub-spoke\` репозитория, скачанного на предыдущем шаге.</span><span class="sxs-lookup"><span data-stu-id="0f715-202">Navigate to the `hybrid-networking\hub-spoke\` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="0f715-203">Откройте файл `onprem.json` и введите имя пользователя и пароль в кавычках в строки 36 и 37, как показано ниже, а затем сохраните файл.</span><span class="sxs-lookup"><span data-stu-id="0f715-203">Open the `onprem.json` file and enter a username and password between the quotes in line 36 and 37, as shown below, then save the file.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```

3. <span data-ttu-id="0f715-204">В строку 38 для параметра `osType` введите `Windows` или `Linux`, чтобы установить Windows Server 2016 Datacenter или Ubuntu 16.04 в качестве операционной системы для jumpbox.</span><span class="sxs-lookup"><span data-stu-id="0f715-204">On line 38, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

4. <span data-ttu-id="0f715-205">Выполните команду `azbb`, чтобы развернуть имитированную локальную среду, как показано ниже.</span><span class="sxs-lookup"><span data-stu-id="0f715-205">Run `azbb` to deploy the simulated onprem environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g onprem-vnet-rg - l <location> -p onoprem.json --deploy
   ```
   > [!NOTE]
   > <span data-ttu-id="0f715-206">Если вы решили использовать другое имя группы ресурсов (отличное от `onprem-vnet-rg`), обязательно найдите все файлы параметров, которые используют это имя, и отредактируйте их, чтобы использовать собственное имя группы ресурсов.</span><span class="sxs-lookup"><span data-stu-id="0f715-206">If you decide to use a different resource group name (other than `onprem-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="0f715-207">Дождитесь завершения развертывания.</span><span class="sxs-lookup"><span data-stu-id="0f715-207">Wait for the deployment to finish.</span></span> <span data-ttu-id="0f715-208">При развертывании создается виртуальная сеть, виртуальная машина и VPN-шлюз.</span><span class="sxs-lookup"><span data-stu-id="0f715-208">This deployment creates a virtual network, a virtual machine, and a VPN gateway.</span></span> <span data-ttu-id="0f715-209">Создание VPN-шлюза может занять более 40 минут.</span><span class="sxs-lookup"><span data-stu-id="0f715-209">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="azure-hub-vnet"></a><span data-ttu-id="0f715-210">Виртуальная сеть концентратора Azure</span><span class="sxs-lookup"><span data-stu-id="0f715-210">Azure hub VNet</span></span>

<span data-ttu-id="0f715-211">Чтобы развернуть виртуальную сеть концентратора Azure и подключиться к имитации локальной виртуальной сети, созданной выше, сделайте следующее.</span><span class="sxs-lookup"><span data-stu-id="0f715-211">To deploy the hub VNet, and connect to the simulated on-premises VNet created above, perform the following steps.</span></span>

1. <span data-ttu-id="0f715-212">Откройте файл `hub-vnet.json` и введите имя пользователя и пароль в кавычках в строки 39 и 40, как показано ниже.</span><span class="sxs-lookup"><span data-stu-id="0f715-212">Open the `hub-vnet.json` file and enter a username and password between the quotes in line 39 and 40, as shown below.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```

2. <span data-ttu-id="0f715-213">В строку 41 для параметра `osType` введите `Windows` или `Linux`, чтобы установить Windows Server 2016 Datacenter или Ubuntu 16.04 в качестве операционной системой для jumpbox.</span><span class="sxs-lookup"><span data-stu-id="0f715-213">On line 41, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

3. <span data-ttu-id="0f715-214">Введите общий ключ в кавычках в строку 72, как показано ниже, а затем сохраните файл.</span><span class="sxs-lookup"><span data-stu-id="0f715-214">Enter a shared key between the quotes in line 72, as shown below, then save the file.</span></span>

   ```bash
   "sharedKey": "",
   ```

4. <span data-ttu-id="0f715-215">Выполните команду `azbb`, чтобы развернуть имитированную локальную среду, как показано ниже.</span><span class="sxs-lookup"><span data-stu-id="0f715-215">Run `azbb` to deploy the simulated onprem environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet.json --deploy
   ```
   > [!NOTE]
   > <span data-ttu-id="0f715-216">Если вы решили использовать другое имя группы ресурсов (отличное от `hub-vnet-rg`), обязательно найдите все файлы параметров, которые используют это имя, и отредактируйте их, чтобы использовать собственное имя группы ресурсов.</span><span class="sxs-lookup"><span data-stu-id="0f715-216">If you decide to use a different resource group name (other than `hub-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="0f715-217">Дождитесь завершения развертывания.</span><span class="sxs-lookup"><span data-stu-id="0f715-217">Wait for the deployment to finish.</span></span> <span data-ttu-id="0f715-218">В ходе этого развертывания создается виртуальная сеть, виртуальная машина, VPN-шлюз и подключение к шлюзу, созданному в предыдущем разделе.</span><span class="sxs-lookup"><span data-stu-id="0f715-218">This deployment creates a virtual network, a virtual machine, a VPN gateway, and a connection to the gateway created in the previous section.</span></span> <span data-ttu-id="0f715-219">Создание VPN-шлюза может занять более 40 минут.</span><span class="sxs-lookup"><span data-stu-id="0f715-219">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="optional-test-connectivity-from-onprem-to-hub"></a><span data-ttu-id="0f715-220">(Необязательно.) Проверка возможности подключения из локальной среды к концентратору</span><span class="sxs-lookup"><span data-stu-id="0f715-220">(Optional) Test connectivity from onprem to hub</span></span>

<span data-ttu-id="0f715-221">Чтобы проверить возможность подключения из имитированной локальной среды к виртуальной сети концентратора с помощью виртуальных машин Windows, сделайте следующее.</span><span class="sxs-lookup"><span data-stu-id="0f715-221">To test conectivity from the simulated on-premises environment to the hub VNet using Windows VMs, perform the following steps.</span></span>

1. <span data-ttu-id="0f715-222">На портале Azure перейдите к группе ресурсов `onprem-jb-rg`, а затем выберите ресурс виртуальной машины `jb-vm1`.</span><span class="sxs-lookup"><span data-stu-id="0f715-222">From the Azure portal, navigate to the `onprem-jb-rg` resource group, then click on the `jb-vm1` virtual machine resource.</span></span>

2. <span data-ttu-id="0f715-223">В левом верхнем углу колонки виртуальной машины на портале выберите `Connect` и следуйте инструкциям на экране, чтобы подключиться к виртуальной машине с помощью удаленного рабочего стола.</span><span class="sxs-lookup"><span data-stu-id="0f715-223">On the top left hand corner of your VM blade in the portal, click `Connect`, and follow the prompts to use remote desktop to connect to the VM.</span></span> <span data-ttu-id="0f715-224">Используйте имя пользователя и пароль, указанные в строках 36 и 37 файла `onprem.json`.</span><span class="sxs-lookup"><span data-stu-id="0f715-224">Make sure to use the username and password you specified in lines 36 and 37 in the `onprem.json` file.</span></span>

3. <span data-ttu-id="0f715-225">Откройте консоль PowerShell на виртуальной машине и используйте командлет `Test-NetConnection`, чтобы проверить возможность подключения к виртуальной машине jumpbox концентратора, как показано ниже.</span><span class="sxs-lookup"><span data-stu-id="0f715-225">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the hub jumpbox VM as shown below.</span></span>

   ```powershell
   Test-NetConnection 10.0.0.68 -CommonTCPPort RDP
   ```
   > [!NOTE]
   > <span data-ttu-id="0f715-226">По умолчанию виртуальные машины Windows Server не разрешают трафик ICMP в Azure.</span><span class="sxs-lookup"><span data-stu-id="0f715-226">By default, Windows Server VMs do not allow ICMP responses in Azure.</span></span> <span data-ttu-id="0f715-227">Если вы хотите использовать команду `ping` для проверки возможности подключения, включите трафик ICMP в расширенном брандмауэре Windows для каждой виртуальной машины.</span><span class="sxs-lookup"><span data-stu-id="0f715-227">If you want to use `ping` to test connectivity, you need to enable ICMP traffic in the Windows Advanced Firewall for each VM.</span></span>

<span data-ttu-id="0f715-228">Чтобы проверить возможность подключения из имитированной локальной среды к виртуальной сети концентратора с помощью виртуальных машин Linux, сделайте следующее:</span><span class="sxs-lookup"><span data-stu-id="0f715-228">To test conectivity from the simulated on-premises environment to the hub VNet using Linux VMs, perform the following steps:</span></span>

1. <span data-ttu-id="0f715-229">На портале Azure перейдите к группе ресурсов `onprem-jb-rg`, а затем выберите ресурс виртуальной машины `jb-vm1`.</span><span class="sxs-lookup"><span data-stu-id="0f715-229">From the Azure portal, navigate to the `onprem-jb-rg` resource group, then click on the `jb-vm1` virtual machine resource.</span></span>

2. <span data-ttu-id="0f715-230">В левом верхнем углу колонки виртуальной машины на портале выберите `Connect`, а затем скопируйте команду `ssh`, которая отображается на портале.</span><span class="sxs-lookup"><span data-stu-id="0f715-230">On the top left hand corner of your VM blade in the portal, click `Connect`, and then copy the `ssh` command shown on the portal.</span></span> 

3. <span data-ttu-id="0f715-231">Из командной строки Linux запустите `ssh`, чтобы подключиться к jumpbox имитированной локальной среды с помощью сведений, скопированных на шаге 2 выше, как показано ниже.</span><span class="sxs-lookup"><span data-stu-id="0f715-231">From a Linux prompt, run `ssh` to connect to the simulated on-premises environment jumpbox witht the information you copied in step 2 above, as shown below.</span></span>

   ```bash
   ssh <your_user>@<public_ip_address>
   ```

4. <span data-ttu-id="0f715-232">Используйте пароль, указанный в строке 37 файла `onprem.json`, чтобы подключиться к виртуальной машине.</span><span class="sxs-lookup"><span data-stu-id="0f715-232">Use the password you specified in line 37 in the `onprem.json` file to the connect to the VM.</span></span>

5. <span data-ttu-id="0f715-233">Используйте команду `ping`, чтобы проверить возможность подключения к jumpbox концентратора, как показано ниже.</span><span class="sxs-lookup"><span data-stu-id="0f715-233">Use the `ping` command to test connectivity to the hub jumpbox, as shown below.</span></span>

   ```bash
   ping 10.0.0.68
   ```

### <a name="azure-spoke-vnets"></a><span data-ttu-id="0f715-234">Виртуальные сети периферийных зон Azure</span><span class="sxs-lookup"><span data-stu-id="0f715-234">Azure spoke VNets</span></span>

<span data-ttu-id="0f715-235">Чтобы развернуть виртуальные сети периферийных зон, сделайте следующее.</span><span class="sxs-lookup"><span data-stu-id="0f715-235">To deploy the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="0f715-236">Откройте файл `spoke1.json` и введите имя пользователя и пароль в кавычках в строки 47 и 48, как показано ниже, а затем сохраните файл.</span><span class="sxs-lookup"><span data-stu-id="0f715-236">Open the `spoke1.json` file and enter a username and password between the quotes in lines 47 and 48, as shown below, then save the file.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```

2. <span data-ttu-id="0f715-237">В строку 49 для параметра `osType` введите `Windows` или `Linux`, чтобы установить Windows Server 2016 Datacenter или Ubuntu 16.04 в качестве операционной системы для jumpbox.</span><span class="sxs-lookup"><span data-stu-id="0f715-237">On line 49, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

3. <span data-ttu-id="0f715-238">Выполните команду `azbb`, чтобы развернуть первую среду виртуальной сети периферийных зон, как показано ниже.</span><span class="sxs-lookup"><span data-stu-id="0f715-238">Run `azbb` to deploy the first spoke VNet environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg - l <location> -p spoke1.json --deploy
   ```
  
   > [!NOTE]
   > <span data-ttu-id="0f715-239">Если вы решили использовать другое имя группы ресурсов (отличное от `spoke1-vnet-rg`), обязательно найдите все файлы параметров, которые используют это имя, и отредактируйте их, чтобы использовать собственное имя группы ресурсов.</span><span class="sxs-lookup"><span data-stu-id="0f715-239">If you decide to use a different resource group name (other than `spoke1-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

4. <span data-ttu-id="0f715-240">Повторите шаг 1 для файла `spoke2.json`.</span><span class="sxs-lookup"><span data-stu-id="0f715-240">Repeat step 1 above for file `spoke2.json`.</span></span>

5. <span data-ttu-id="0f715-241">Выполните команду `azbb`, чтобы развернуть вторую среду виртуальной сети периферийных зон, как показано ниже.</span><span class="sxs-lookup"><span data-stu-id="0f715-241">Run `azbb` to deploy the second spoke VNet environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg - l <location> -p spoke2.json --deploy
   ```
   > [!NOTE]
   > <span data-ttu-id="0f715-242">Если вы решили использовать другое имя группы ресурсов (отличное от `spoke2-vnet-rg`), обязательно найдите все файлы параметров, которые используют это имя, и отредактируйте их, чтобы использовать собственное имя группы ресурсов.</span><span class="sxs-lookup"><span data-stu-id="0f715-242">If you decide to use a different resource group name (other than `spoke2-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a><span data-ttu-id="0f715-243">Пиринговое подключение виртуальной сети концентратора Azure к виртуальным сетям периферийных зон</span><span class="sxs-lookup"><span data-stu-id="0f715-243">Azure hub VNet peering to spoke VNets</span></span>

<span data-ttu-id="0f715-244">Чтобы создать пиринговую связь между виртуальной сетью концентратора и виртуальными сетями периферийных зон, сделайте следующее.</span><span class="sxs-lookup"><span data-stu-id="0f715-244">To create a peering connection from the hub VNet to the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="0f715-245">Откройте файл `hub-vnet-peering.json`, чтобы проверить правильность имени группы ресурсов и имени виртуальной сети для каждого пиринга виртуальных сетей, начиная с 29 строки.</span><span class="sxs-lookup"><span data-stu-id="0f715-245">Open the `hub-vnet-peering.json` file and verify that the resource group name, and virtual network name for each of the virtual network peerings starting in line 29 are correct.</span></span>

2. <span data-ttu-id="0f715-246">Выполните команду `azbb`, чтобы развернуть первую среду виртуальной сети периферийных зон, как показано ниже.</span><span class="sxs-lookup"><span data-stu-id="0f715-246">Run `azbb` to deploy the first spoke VNet environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet-peering.json --deploy
   ```

   > [!NOTE]
   > <span data-ttu-id="0f715-247">Если вы решили использовать другое имя группы ресурсов (отличное от `hub-vnet-rg`), обязательно найдите все файлы параметров, которые используют это имя, и отредактируйте их, чтобы использовать собственное имя группы ресурсов.</span><span class="sxs-lookup"><span data-stu-id="0f715-247">If you decide to use a different resource group name (other than `hub-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

### <a name="test-connectivity"></a><span data-ttu-id="0f715-248">Проверка подключения</span><span class="sxs-lookup"><span data-stu-id="0f715-248">Test connectivity</span></span>

<span data-ttu-id="0f715-249">Чтобы проверить возможность подключения из имитированной локальной среды к виртуальной сети периферийных зон с помощью виртуальных машин Windows, сделайте следующее.</span><span class="sxs-lookup"><span data-stu-id="0f715-249">To test conectivity from the simulated on-premises environment to the spoke VNets using Windows VMs, perform the following steps.</span></span>

1. <span data-ttu-id="0f715-250">На портале Azure перейдите к группе ресурсов `onprem-jb-rg`, а затем выберите ресурс виртуальной машины `jb-vm1`.</span><span class="sxs-lookup"><span data-stu-id="0f715-250">From the Azure portal, navigate to the `onprem-jb-rg` resource group, then click on the `jb-vm1` virtual machine resource.</span></span>

2. <span data-ttu-id="0f715-251">В левом верхнем углу колонки виртуальной машины на портале выберите `Connect` и следуйте инструкциям на экране, чтобы подключиться к виртуальной машине с помощью удаленного рабочего стола.</span><span class="sxs-lookup"><span data-stu-id="0f715-251">On the top left hand corner of your VM blade in the portal, click `Connect`, and follow the prompts to use remote desktop to connect to the VM.</span></span> <span data-ttu-id="0f715-252">Используйте имя пользователя и пароль, указанные в строках 36 и 37 файла `onprem.json`.</span><span class="sxs-lookup"><span data-stu-id="0f715-252">Make sure to use the username and password you specified in lines 36 and 37 in the `onprem.json` file.</span></span>

3. <span data-ttu-id="0f715-253">Откройте консоль PowerShell на виртуальной машине и используйте командлет `Test-NetConnection`, чтобы проверить возможность подключения к виртуальной машине jumpbox концентратора, как показано ниже.</span><span class="sxs-lookup"><span data-stu-id="0f715-253">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the hub jumpbox VM as shown below.</span></span>

   ```powershell
   Test-NetConnection 10.1.0.68 -CommonTCPPort RDP
   Test-NetConnection 10.2.0.68 -CommonTCPPort RDP
   ```

<span data-ttu-id="0f715-254">Чтобы проверить возможность подключения из имитированной локальной среды к виртуальной сети периферийных зон с помощью виртуальных машин Linux, сделайте следующее:</span><span class="sxs-lookup"><span data-stu-id="0f715-254">To test conectivity from the simulated on-premises environment to the spoke VNets using Linux VMs, perform the following steps:</span></span>

1. <span data-ttu-id="0f715-255">На портале Azure перейдите к группе ресурсов `onprem-jb-rg`, а затем выберите ресурс виртуальной машины `jb-vm1`.</span><span class="sxs-lookup"><span data-stu-id="0f715-255">From the Azure portal, navigate to the `onprem-jb-rg` resource group, then click on the `jb-vm1` virtual machine resource.</span></span>

2. <span data-ttu-id="0f715-256">В левом верхнем углу колонки виртуальной машины на портале выберите `Connect`, а затем скопируйте команду `ssh`, которая отображается на портале.</span><span class="sxs-lookup"><span data-stu-id="0f715-256">On the top left hand corner of your VM blade in the portal, click `Connect`, and then copy the `ssh` command shown on the portal.</span></span> 

3. <span data-ttu-id="0f715-257">Из командной строки Linux запустите `ssh`, чтобы подключиться к jumpbox имитированной локальной среды с помощью сведений, скопированных на шаге 2 выше, как показано ниже.</span><span class="sxs-lookup"><span data-stu-id="0f715-257">From a Linux prompt, run `ssh` to connect to the simulated on-premises environment jumpbox witht the information you copied in step 2 above, as shown below.</span></span>

   ```bash
   ssh <your_user>@<public_ip_address>
   ```

4. <span data-ttu-id="0f715-258">Используйте пароль, указанный в строке 37 файла `onprem.json`, чтобы подключиться к виртуальной машине.</span><span class="sxs-lookup"><span data-stu-id="0f715-258">Use the password you specified in line 37 in the `onprem.json` file to the connect to the VM.</span></span>

5. <span data-ttu-id="0f715-259">Используйте команду `ping`, чтобы проверить возможность подключения к виртуальным машинам jumpbox в каждой периферийной зоне, как показано ниже.</span><span class="sxs-lookup"><span data-stu-id="0f715-259">Use the `ping` command to test connectivity to the jumpbox VMs in each spoke, as shown below.</span></span>

   ```bash
   ping 10.1.0.68
   ping 10.2.0.68
   ```

### <a name="add-connectivity-between-spokes"></a><span data-ttu-id="0f715-260">Добавление подключения между периферийными зонами</span><span class="sxs-lookup"><span data-stu-id="0f715-260">Add connectivity between spokes</span></span>

<span data-ttu-id="0f715-261">Чтобы включить возможность подключения между периферийными зонами, используйте виртуальный сетевой модуль в качестве маршрутизатора виртуальной сети концентратора и принудительно отправляйте трафик из периферийных зон на маршрутизатор при попытке подключения к другой периферийной зоне.</span><span class="sxs-lookup"><span data-stu-id="0f715-261">If you want to allow spokes to connect to each other, you need to use a newtwork virtual appliance (NVA) as a router in the hub virtual netowrk, and force traffic from spokes to the router when trying to connect to another spoke.</span></span> <span data-ttu-id="0f715-262">Чтобы развернуть базовый пример виртуального сетевого модуля в качестве одной виртуальной машины и разрешить определяемым пользователем маршрутам соединять две виртуальные машины периферийных зон, сделайте следующее:</span><span class="sxs-lookup"><span data-stu-id="0f715-262">To deploy a basic sample NVA as a single VM, and the necessary uder defined routes to allow the two spoke VNets to connect, perform the following steps:</span></span>

1. <span data-ttu-id="0f715-263">Откройте файл `hub-nva.json` и введите имя пользователя и пароль в кавычках в строках 13 и 14, как показано ниже, а затем сохраните файл.</span><span class="sxs-lookup"><span data-stu-id="0f715-263">Open the `hub-nva.json` file and enter a username and password between the quotes in lines 13 and 14, as shown below, then save the file.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```
2. <span data-ttu-id="0f715-264">Выполните команду `azbb`, чтобы развернуть виртуальную машину NVA и определяемые пользователем маршруты.</span><span class="sxs-lookup"><span data-stu-id="0f715-264">Run `azbb` to deploy the NVA VM and user defined routes.</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-nva-rg - l <location> -p hub-nva.json --deploy
   ```
   > [!NOTE]
   > <span data-ttu-id="0f715-265">Если вы решили использовать другое имя группы ресурсов (отличное от `hub-nva-rg`), обязательно найдите все файлы параметров, которые используют это имя, и отредактируйте их, чтобы использовать собственное имя группы ресурсов.</span><span class="sxs-lookup"><span data-stu-id="0f715-265">If you decide to use a different resource group name (other than `hub-nva-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

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
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/hub-spoke.png "Звездообразная топология в Azure"
[1]: ./images/hub-spoke-gateway-routing.svg "Звездообразная топология в Azure с транзитивной маршрутизацией"
[2]: ./images/hub-spoke-no-gateway-routing.svg "Звездообразная топология в Azure с транзитивной маршрутизацией с использованием виртуального сетевого устройства (NVA)"
[3]: ./images/hub-spokehub-spoke.svg "Вложенная звездообразная топология в Azure"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
