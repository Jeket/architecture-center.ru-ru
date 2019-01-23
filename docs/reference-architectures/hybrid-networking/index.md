---
title: Подключение локальной сети к Azure
titleSuffix: Azure Reference Architectures
description: Сравнение нескольких эталонных архитектур, позволяющих подключить локальную сеть к Azure.
author: telmosampaio
ms.date: 07/02/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: networking
ms.openlocfilehash: 6172866b08197b0ca1cd3aabb3c14c01b4f06f9c
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54486863"
---
# <a name="choose-a-solution-for-connecting-an-on-premises-network-to-azure"></a><span data-ttu-id="11b21-103">Выбор решения для подключения локальной сети к Azure</span><span class="sxs-lookup"><span data-stu-id="11b21-103">Choose a solution for connecting an on-premises network to Azure</span></span>

<span data-ttu-id="11b21-104">В этой статье сравниваются варианты подключения локальной сети к виртуальной сети Azure.</span><span class="sxs-lookup"><span data-stu-id="11b21-104">This article compares options for connecting an on-premises network to an Azure Virtual Network (VNet).</span></span> <span data-ttu-id="11b21-105">Для каждого варианта доступна подробная эталонная архитектура.</span><span class="sxs-lookup"><span data-stu-id="11b21-105">For each option, a more detailed reference architecture is available.</span></span>

## <a name="vpn-connection"></a><span data-ttu-id="11b21-106">VPN-подключение</span><span class="sxs-lookup"><span data-stu-id="11b21-106">VPN connection</span></span>

<span data-ttu-id="11b21-107">[VPN-шлюз](/azure/vpn-gateway/vpn-gateway-about-vpngateways) — это разновидность шлюза виртуальной сети, который передает зашифрованный трафик между виртуальной сетью Azure и локальным расположением.</span><span class="sxs-lookup"><span data-stu-id="11b21-107">A [VPN gateway](/azure/vpn-gateway/vpn-gateway-about-vpngateways) is a type of virtual network gateway that sends encrypted traffic between an Azure virtual network and an on-premises location.</span></span> <span data-ttu-id="11b21-108">Зашифрованный трафик проходит через общедоступный Интернет.</span><span class="sxs-lookup"><span data-stu-id="11b21-108">The encrypted traffic goes over the public Internet.</span></span>

<span data-ttu-id="11b21-109">Эта архитектура подходит для гибридных приложений, в которых ожидается незначительный трафик между локальным оборудованием и облаком, а также для сценариев, в которых допустимо некоторое увеличение задержки в обмен на дополнительную гибкость и вычислительную мощность облачных систем.</span><span class="sxs-lookup"><span data-stu-id="11b21-109">This architecture is suitable for hybrid applications where the traffic between on-premises hardware and the cloud is likely to be light, or you are willing to trade slightly extended latency for the flexibility and processing power of the cloud.</span></span>

### <a name="benefits"></a><span data-ttu-id="11b21-110">Преимущества</span><span class="sxs-lookup"><span data-stu-id="11b21-110">Benefits</span></span>

- <span data-ttu-id="11b21-111">Простая настройка.</span><span class="sxs-lookup"><span data-stu-id="11b21-111">Simple to configure.</span></span>

### <a name="challenges"></a><span data-ttu-id="11b21-112">Сложности</span><span class="sxs-lookup"><span data-stu-id="11b21-112">Challenges</span></span>

- <span data-ttu-id="11b21-113">Требуется локальное VPN-устройство.</span><span class="sxs-lookup"><span data-stu-id="11b21-113">Requires an on-premises VPN device.</span></span>
- <span data-ttu-id="11b21-114">Несмотря на то что корпорация Майкрософт гарантирует уровень доступности 99,9 % для каждого VPN-шлюза, это [соглашение об уровне обслуживания](https://azure.microsoft.com/support/legal/sla/vpn-gateway/) касается только самого VPN-шлюза, но не сетевого подключения к этому шлюзу.</span><span class="sxs-lookup"><span data-stu-id="11b21-114">Although Microsoft guarantees 99.9% availability for each VPN Gateway, this [SLA](https://azure.microsoft.com/support/legal/sla/vpn-gateway/) only covers the VPN gateway, and not your network connection to the gateway.</span></span>
- <span data-ttu-id="11b21-115">Сейчас VPN-подключения через VPN-шлюз Azure поддерживают пропускную способность не более 1,25 Мбит/с.</span><span class="sxs-lookup"><span data-stu-id="11b21-115">A VPN connection over Azure VPN Gateway currently supports a maximum of 1.25 Gbps bandwidth.</span></span> <span data-ttu-id="11b21-116">Если вам может потребоваться более высокая пропускная способность, распределите виртуальную сеть Azure на несколько VPN-подключений.</span><span class="sxs-lookup"><span data-stu-id="11b21-116">You may need to partition your Azure virtual network across multiple VPN connections if you expect to exceed this throughput.</span></span>

### <a name="reference-architecture"></a><span data-ttu-id="11b21-117">Эталонная архитектура</span><span class="sxs-lookup"><span data-stu-id="11b21-117">Reference architecture</span></span>

- [<span data-ttu-id="11b21-118">Гибридная сеть с VPN-шлюзом</span><span class="sxs-lookup"><span data-stu-id="11b21-118">Hybrid network with VPN gateway</span></span>](./vpn.md)

<!-- markdownlint-disable MD024 -->

## <a name="azure-expressroute-connection"></a><span data-ttu-id="11b21-119">Подключение Azure ExpressRoute</span><span class="sxs-lookup"><span data-stu-id="11b21-119">Azure ExpressRoute connection</span></span>

<span data-ttu-id="11b21-120">В подключениях [ExpressRoute](/azure/expressroute/) используются частные выделенные подключения через сети сторонних поставщиков услуг подключения.</span><span class="sxs-lookup"><span data-stu-id="11b21-120">[ExpressRoute](/azure/expressroute/) connections use a private, dedicated connection through a third-party connectivity provider.</span></span> <span data-ttu-id="11b21-121">Такое частное подключение расширяет вашу локальную сеть в Azure.</span><span class="sxs-lookup"><span data-stu-id="11b21-121">The private connection extends your on-premises network into Azure.</span></span>

<span data-ttu-id="11b21-122">Эта архитектура подходит для гибридных приложений, на которых выполняются крупномасштабные и критически важные рабочие нагрузки с высокими требованиями к масштабируемости.</span><span class="sxs-lookup"><span data-stu-id="11b21-122">This architecture is suitable for hybrid applications running large-scale, mission-critical workloads that require a high degree of scalability.</span></span>

### <a name="benefits"></a><span data-ttu-id="11b21-123">Преимущества</span><span class="sxs-lookup"><span data-stu-id="11b21-123">Benefits</span></span>

- <span data-ttu-id="11b21-124">Доступна очень высокая пропускная способность (до 10 Гбит/с в зависимости от поставщика услуг подключения).</span><span class="sxs-lookup"><span data-stu-id="11b21-124">Much higher bandwidth available; up to 10 Gbps depending on the connectivity provider.</span></span>
- <span data-ttu-id="11b21-125">Поддерживает динамическое масштабирование пропускной способности, чтобы снизить расходы в период низкого спроса.</span><span class="sxs-lookup"><span data-stu-id="11b21-125">Supports dynamic scaling of bandwidth to help reduce costs during periods of lower demand.</span></span> <span data-ttu-id="11b21-126">Эту функциональность поддерживают не все поставщики услуг подключения.</span><span class="sxs-lookup"><span data-stu-id="11b21-126">However, not all connectivity providers have this option.</span></span>
- <span data-ttu-id="11b21-127">Может предоставить организации прямой доступ к национальным облакам, если это поддерживается поставщиком услуг подключения.</span><span class="sxs-lookup"><span data-stu-id="11b21-127">May allow your organization direct access to national clouds, depending on the connectivity provider.</span></span>
- <span data-ttu-id="11b21-128">Соглашения об уровне обслуживания гарантируют доступность 99,9 % для всего подключения.</span><span class="sxs-lookup"><span data-stu-id="11b21-128">99.9% availability SLA across the entire connection.</span></span>

### <a name="challenges"></a><span data-ttu-id="11b21-129">Сложности</span><span class="sxs-lookup"><span data-stu-id="11b21-129">Challenges</span></span>

- <span data-ttu-id="11b21-130">Может потребоваться сложная настройка.</span><span class="sxs-lookup"><span data-stu-id="11b21-130">Can be complex to set up.</span></span> <span data-ttu-id="11b21-131">Для создания подключения ExpressRoute требуется взаимодействие со сторонним поставщиком услуг подключения.</span><span class="sxs-lookup"><span data-stu-id="11b21-131">Creating an ExpressRoute connection requires working with a third-party connectivity provider.</span></span> <span data-ttu-id="11b21-132">Этот поставщик отвечает за подготовку сетевого подключения.</span><span class="sxs-lookup"><span data-stu-id="11b21-132">The provider is responsible for provisioning the network connection.</span></span>
- <span data-ttu-id="11b21-133">В локальной сети требуются маршрутизаторы с высокой пропускной способностью.</span><span class="sxs-lookup"><span data-stu-id="11b21-133">Requires high-bandwidth routers on-premises.</span></span>

### <a name="reference-architecture"></a><span data-ttu-id="11b21-134">Эталонная архитектура</span><span class="sxs-lookup"><span data-stu-id="11b21-134">Reference architecture</span></span>

- [<span data-ttu-id="11b21-135">Гибридная сеть с использованием ExpressRoute</span><span class="sxs-lookup"><span data-stu-id="11b21-135">Hybrid network with ExpressRoute</span></span>](./expressroute.md)

## <a name="expressroute-with-vpn-failover"></a><span data-ttu-id="11b21-136">ExpressRoute с отработкой отказа на VPN</span><span class="sxs-lookup"><span data-stu-id="11b21-136">ExpressRoute with VPN failover</span></span>

<span data-ttu-id="11b21-137">Этот вариант объединяет два предыдущих. Для обычной работы применяется ExpressRoute, а при возникновении проблем с ним происходит переключение на VPN.</span><span class="sxs-lookup"><span data-stu-id="11b21-137">This options combines the previous two, using ExpressRoute in normal conditions, but failing over to a VPN connection if there is a loss of connectivity in the ExpressRoute circuit.</span></span>

<span data-ttu-id="11b21-138">Такая архитектура подходит для гибридных приложений, которым одновременно требуется высокая пропускная способность ExpressRoute и высокий уровень доступности сетевого подключения.</span><span class="sxs-lookup"><span data-stu-id="11b21-138">This architecture is suitable for hybrid applications that need the higher bandwidth of ExpressRoute, and also require highly available network connectivity.</span></span>

### <a name="benefits"></a><span data-ttu-id="11b21-139">Преимущества</span><span class="sxs-lookup"><span data-stu-id="11b21-139">Benefits</span></span>

- <span data-ttu-id="11b21-140">Высокий уровень доступности. Подключение сохраняется даже при сбое канала ExpressRoute, хотя и со снижением пропускной способности сети.</span><span class="sxs-lookup"><span data-stu-id="11b21-140">High availability if the ExpressRoute circuit fails, although the fallback connection is on a lower bandwidth network.</span></span>

### <a name="challenges"></a><span data-ttu-id="11b21-141">Сложности</span><span class="sxs-lookup"><span data-stu-id="11b21-141">Challenges</span></span>

- <span data-ttu-id="11b21-142">Сложность в настройке.</span><span class="sxs-lookup"><span data-stu-id="11b21-142">Complex to configure.</span></span> <span data-ttu-id="11b21-143">Необходимо настроить одновременно VPN-подключение и канал ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="11b21-143">You need to set up both a VPN connection and an ExpressRoute circuit.</span></span>
- <span data-ttu-id="11b21-144">Требуется резервное оборудование (устройства VPN) и резервное подключение к шлюзу VPN Azure, что приводит к дополнительным расходам.</span><span class="sxs-lookup"><span data-stu-id="11b21-144">Requires redundant hardware (VPN appliances), and a redundant Azure VPN Gateway connection for which you pay charges.</span></span>

### <a name="reference-architecture"></a><span data-ttu-id="11b21-145">Эталонная архитектура</span><span class="sxs-lookup"><span data-stu-id="11b21-145">Reference architecture</span></span>

- [<span data-ttu-id="11b21-146">Гибридная сеть с использованием отработки отказа в ExpressRoute с VPN</span><span class="sxs-lookup"><span data-stu-id="11b21-146">Hybrid network with ExpressRoute and VPN failover</span></span>](./expressroute-vpn-failover.md)

<!-- markdownlint-disable MD024 -->

## <a name="hub-spoke-network-topology"></a><span data-ttu-id="11b21-147">Звездообразная топология сети</span><span class="sxs-lookup"><span data-stu-id="11b21-147">Hub-spoke network topology</span></span>

<span data-ttu-id="11b21-148">Звездообразная топология сети — это способ изоляции рабочих нагрузок при совместном использовании служб, таких как службы идентификации и безопасности.</span><span class="sxs-lookup"><span data-stu-id="11b21-148">A hub-spoke network topology is a way to isolate workloads while sharing services such as identity and security.</span></span> <span data-ttu-id="11b21-149">Концентратор (центр топологии) — это виртуальная сеть в Azure, которая выступает в качестве центральной точки подключения к локальной сети.</span><span class="sxs-lookup"><span data-stu-id="11b21-149">The hub is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="11b21-150">Периферийные зоны — это виртуальные сети, которые устанавливают пиринг с концентратором.</span><span class="sxs-lookup"><span data-stu-id="11b21-150">The spokes are VNets that peer with the hub.</span></span> <span data-ttu-id="11b21-151">Общие службы развертываются в концентраторе, а отдельные рабочие нагрузки — в периферийных зонах.</span><span class="sxs-lookup"><span data-stu-id="11b21-151">Shared services are deployed in the hub, while individual workloads are deployed as spokes.</span></span>

### <a name="reference-architectures"></a><span data-ttu-id="11b21-152">Эталонные архитектуры</span><span class="sxs-lookup"><span data-stu-id="11b21-152">Reference architectures</span></span>

- [<span data-ttu-id="11b21-153">Звездообразная топология</span><span class="sxs-lookup"><span data-stu-id="11b21-153">Hub-spoke topology</span></span>](./hub-spoke.md)
- [<span data-ttu-id="11b21-154">Звездообразная топология с общими службами</span><span class="sxs-lookup"><span data-stu-id="11b21-154">Hub-spoke with shared services</span></span>](./shared-services.md)
