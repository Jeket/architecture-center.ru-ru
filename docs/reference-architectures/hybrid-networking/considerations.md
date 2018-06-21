---
title: Выбор решения для подключения локальной сети к Azure
description: Сравнение нескольких эталонных архитектур, позволяющих подключить локальную сеть к Azure.
author: telmosampaio
ms.date: 04/06/2017
ms.openlocfilehash: 274b9df1817632a7f3eaafa8bf02e965fdc3feea
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/14/2017
ms.locfileid: "24541054"
---
# <a name="choose-a-solution-for-connecting-an-on-premises-network-to-azure"></a><span data-ttu-id="ecac6-103">Выбор решения для подключения локальной сети к Azure</span><span class="sxs-lookup"><span data-stu-id="ecac6-103">Choose a solution for connecting an on-premises network to Azure</span></span>

<span data-ttu-id="ecac6-104">В этой статье сравниваются варианты подключения локальной сети к виртуальной сети Azure.</span><span class="sxs-lookup"><span data-stu-id="ecac6-104">This article compares options for connecting an on-premises network to an Azure Virtual Network (VNet).</span></span> <span data-ttu-id="ecac6-105">Для каждого варианта мы предоставляем эталонную архитектуру и готовое к развертыванию решение.</span><span class="sxs-lookup"><span data-stu-id="ecac6-105">We provide a reference architecture and a deployable solution for each option.</span></span>

## <a name="vpn-connection"></a><span data-ttu-id="ecac6-106">VPN-подключение</span><span class="sxs-lookup"><span data-stu-id="ecac6-106">VPN connection</span></span>

<span data-ttu-id="ecac6-107">Применение виртуальной частной сети (VPN) для подключения локальной сети к виртуальной сети Azure через туннель IPSec VPN.</span><span class="sxs-lookup"><span data-stu-id="ecac6-107">Use a virtual private network (VPN) to connect your on-premises network with an Azure VNet through an IPSec VPN tunnel.</span></span>

<span data-ttu-id="ecac6-108">Эта архитектура подходит для гибридных приложений, в которых ожидается незначительный трафик между локальным оборудованием и облаком, а также для сценариев, в которых допустимо некоторое увеличение задержки в обмен на дополнительную гибкость и вычислительную мощность облачных систем.</span><span class="sxs-lookup"><span data-stu-id="ecac6-108">This architecture is suitable for hybrid applications where the traffic between on-premises hardware and the cloud is likely to be light, or you are willing to trade slightly extended latency for the flexibility and processing power of the cloud.</span></span>

<span data-ttu-id="ecac6-109">**Преимущества**</span><span class="sxs-lookup"><span data-stu-id="ecac6-109">**Benefits**</span></span>

- <span data-ttu-id="ecac6-110">Простая настройка.</span><span class="sxs-lookup"><span data-stu-id="ecac6-110">Simple to configure.</span></span>

<span data-ttu-id="ecac6-111">**Сложности**</span><span class="sxs-lookup"><span data-stu-id="ecac6-111">**Challenges**</span></span>

- <span data-ttu-id="ecac6-112">Требуется локальное VPN-устройство.</span><span class="sxs-lookup"><span data-stu-id="ecac6-112">Requires an on-premises VPN device.</span></span>
- <span data-ttu-id="ecac6-113">Несмотря на то, что корпорация Майкрософт гарантирует уровень доступности 99,9 % для каждого шлюза VPN, это соглашение об уровне обслуживания относится только к самому шлюзу VPN, но не к сетевому подключению этого шлюза.</span><span class="sxs-lookup"><span data-stu-id="ecac6-113">Although Microsoft guarantees 99.9% availability for each VPN Gateway, this SLA only covers the VPN gateway, and not your network connection to the gateway.</span></span>
- <span data-ttu-id="ecac6-114">В настоящее время VPN-подключения через VPN-шлюз Azure поддерживают пропускную способность не более 200 Мбит/с.</span><span class="sxs-lookup"><span data-stu-id="ecac6-114">A VPN connection over Azure VPN Gateway currently supports a maximum of 200 Mbps bandwidth.</span></span> <span data-ttu-id="ecac6-115">Если вам может потребоваться более высокая пропускная способность, распределите виртуальную сеть Azure на несколько VPN-подключений.</span><span class="sxs-lookup"><span data-stu-id="ecac6-115">You may need to partition your Azure virtual network across multiple VPN connections if you expect to exceed this throughput.</span></span>

<span data-ttu-id="ecac6-116">**[Подробнее...][vpn]**</span><span class="sxs-lookup"><span data-stu-id="ecac6-116">**[Read more...][vpn]**</span></span>

## <a name="azure-expressroute-connection"></a><span data-ttu-id="ecac6-117">Подключение Azure ExpressRoute</span><span class="sxs-lookup"><span data-stu-id="ecac6-117">Azure ExpressRoute connection</span></span>

<span data-ttu-id="ecac6-118">Подключения ExpressRoute создают закрытые выделенные подключения через сети сторонних поставщиков услуг подключения.</span><span class="sxs-lookup"><span data-stu-id="ecac6-118">ExpressRoute connections use a private, dedicated connection through a third-party connectivity provider.</span></span> <span data-ttu-id="ecac6-119">Такое частное подключение расширяет вашу локальную сеть в Azure.</span><span class="sxs-lookup"><span data-stu-id="ecac6-119">The private connection extends your on-premises network into Azure.</span></span> 

<span data-ttu-id="ecac6-120">Эта архитектура подходит для гибридных приложений, на которых выполняются крупномасштабные и критически важные рабочие нагрузки с высокими требованиями к масштабируемости.</span><span class="sxs-lookup"><span data-stu-id="ecac6-120">This architecture is suitable for hybrid applications running large-scale, mission-critical workloads that require a high degree of scalability.</span></span> 

<span data-ttu-id="ecac6-121">**Преимущества**</span><span class="sxs-lookup"><span data-stu-id="ecac6-121">**Benefits**</span></span>

- <span data-ttu-id="ecac6-122">Доступна очень высокая пропускная способность (до 10 Гбит/с в зависимости от поставщика услуг подключения).</span><span class="sxs-lookup"><span data-stu-id="ecac6-122">Much higher bandwidth available; up to 10 Gbps depending on the connectivity provider.</span></span>
- <span data-ttu-id="ecac6-123">Поддерживает динамическое масштабирование пропускной способности, чтобы снизить расходы в период низкого спроса.</span><span class="sxs-lookup"><span data-stu-id="ecac6-123">Supports dynamic scaling of bandwidth to help reduce costs during periods of lower demand.</span></span> <span data-ttu-id="ecac6-124">Эту функциональность поддерживают не все поставщики услуг подключения.</span><span class="sxs-lookup"><span data-stu-id="ecac6-124">However, not all connectivity providers have this option.</span></span>
- <span data-ttu-id="ecac6-125">Может предоставить организации прямой доступ к национальным облакам, если это поддерживается поставщиком услуг подключения.</span><span class="sxs-lookup"><span data-stu-id="ecac6-125">May allow your organization direct access to national clouds, depending on the connectivity provider.</span></span>
- <span data-ttu-id="ecac6-126">Соглашения об уровне обслуживания гарантируют доступность 99,9 % для всего подключения.</span><span class="sxs-lookup"><span data-stu-id="ecac6-126">99.9% availability SLA across the entire connection.</span></span>

<span data-ttu-id="ecac6-127">**Сложности**</span><span class="sxs-lookup"><span data-stu-id="ecac6-127">**Challenges**</span></span>

- <span data-ttu-id="ecac6-128">Может потребоваться сложная настройка.</span><span class="sxs-lookup"><span data-stu-id="ecac6-128">Can be complex to set up.</span></span> <span data-ttu-id="ecac6-129">Для создания подключения ExpressRoute требуется взаимодействие со сторонним поставщиком услуг подключения.</span><span class="sxs-lookup"><span data-stu-id="ecac6-129">Creating an ExpressRoute connection requires working with a third-party connectivity provider.</span></span> <span data-ttu-id="ecac6-130">Этот поставщик отвечает за подготовку сетевого подключения.</span><span class="sxs-lookup"><span data-stu-id="ecac6-130">The provider is responsible for provisioning the network connection.</span></span>
- <span data-ttu-id="ecac6-131">В локальной сети требуются маршрутизаторы с высокой пропускной способностью.</span><span class="sxs-lookup"><span data-stu-id="ecac6-131">Requires high-bandwidth routers on-premises.</span></span>

<span data-ttu-id="ecac6-132">**[Подробнее...][expressroute]**</span><span class="sxs-lookup"><span data-stu-id="ecac6-132">**[Read more...][expressroute]**</span></span>

## <a name="expressroute-with-vpn-failover"></a><span data-ttu-id="ecac6-133">ExpressRoute с отработкой отказа на VPN</span><span class="sxs-lookup"><span data-stu-id="ecac6-133">ExpressRoute with VPN failover</span></span>

<span data-ttu-id="ecac6-134">Этот вариант объединяет два предыдущих. Для обычной работы применяется ExpressRoute, а при возникновении проблем с ним происходит переключение на VPN.</span><span class="sxs-lookup"><span data-stu-id="ecac6-134">This options combines the previous two, using ExpressRoute in normal conditions, but failing over to a VPN connection if there is a loss of connectivity in the ExpressRoute circuit.</span></span>

<span data-ttu-id="ecac6-135">Такая архитектура подходит для гибридных приложений, которым одновременно требуется высокая пропускная способность ExpressRoute и высокий уровень доступности сетевого подключения.</span><span class="sxs-lookup"><span data-stu-id="ecac6-135">This architecture is suitable for hybrid applications that need the higher bandwidth of ExpressRoute, and also require highly available network connectivity.</span></span> 

<span data-ttu-id="ecac6-136">**Преимущества**</span><span class="sxs-lookup"><span data-stu-id="ecac6-136">**Benefits**</span></span>

- <span data-ttu-id="ecac6-137">Высокий уровень доступности. Подключение сохраняется даже при сбое канала ExpressRoute, хотя и со снижением пропускной способности сети.</span><span class="sxs-lookup"><span data-stu-id="ecac6-137">High availability if the ExpressRoute circuit fails, although the fallback connection is on a lower bandwidth network.</span></span>

<span data-ttu-id="ecac6-138">**Сложности**</span><span class="sxs-lookup"><span data-stu-id="ecac6-138">**Challenges**</span></span>

- <span data-ttu-id="ecac6-139">Сложность в настройке.</span><span class="sxs-lookup"><span data-stu-id="ecac6-139">Complex to configure.</span></span> <span data-ttu-id="ecac6-140">Необходимо настроить одновременно VPN-подключение и канал ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="ecac6-140">You need to set up both a VPN connection and an ExpressRoute circuit.</span></span>
- <span data-ttu-id="ecac6-141">Требуется резервное оборудование (устройства VPN) и резервное подключение к шлюзу VPN Azure, что приводит к дополнительным расходам.</span><span class="sxs-lookup"><span data-stu-id="ecac6-141">Requires redundant hardware (VPN appliances), and a redundant Azure VPN Gateway connection for which you pay charges.</span></span>

<span data-ttu-id="ecac6-142">**[Подробнее...][expressroute-vpn-failover]**</span><span class="sxs-lookup"><span data-stu-id="ecac6-142">**[Read more...][expressroute-vpn-failover]**</span></span>

<!-- links -->
[expressroute]: ./expressroute.md
[expressroute-vpn-failover]: ./expressroute-vpn-failover.md
[vpn]: ./vpn.md