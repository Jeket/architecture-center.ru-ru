---
title: Подключение локальной сети к Azure с помощью VPN
description: Инструкции по реализации архитектуры защищенной сети типа "сеть — сеть", которая охватывает виртуальную сеть Azure и локальную сеть, подключенные с помощью VPN.
author: RohitSharma-pnp
ms.date: 11/28/2016
pnp.series.title: Connect an on-premises network to Azure
pnp.series.next: expressroute
pnp.series.prev: ./index
cardTitle: VPN
ms.openlocfilehash: dafcee6607d9cc7c56c332f9ed5d9568ff70f0e7
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2018
---
# <a name="connect-an-on-premises-network-to-azure-using-a-vpn-gateway"></a><span data-ttu-id="e76bc-103">Подключение локальной сети к Azure с помощью VPN-шлюза</span><span class="sxs-lookup"><span data-stu-id="e76bc-103">Connect an on-premises network to Azure using a VPN gateway</span></span>

<span data-ttu-id="e76bc-104">На схеме этой эталонной архитектуры представлены сведения о том, как расширить локальную сеть в Azure с помощью подключения VPN "сеть — сеть".</span><span class="sxs-lookup"><span data-stu-id="e76bc-104">This reference architecture shows how to extend an on-premises network to Azure, using a site-to-site virtual private network (VPN).</span></span> <span data-ttu-id="e76bc-105">Трафик проходит между локальной сетью и виртуальной сетью Azure через VPN-туннель IPSec.</span><span class="sxs-lookup"><span data-stu-id="e76bc-105">Traffic flows between the on-premises network and an Azure Virtual Network (VNet) through an IPSec VPN tunnel.</span></span> [<span data-ttu-id="e76bc-106">**Разверните это решение**.</span><span class="sxs-lookup"><span data-stu-id="e76bc-106">**Deploy this solution**.</span></span>](#deploy-the-solution)

<span data-ttu-id="e76bc-107">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="e76bc-107">![[0]][0]</span></span>

<span data-ttu-id="e76bc-108">*Скачайте [файл Visio][visio-download] этой архитектуры.*</span><span class="sxs-lookup"><span data-stu-id="e76bc-108">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="e76bc-109">Архитектура</span><span class="sxs-lookup"><span data-stu-id="e76bc-109">Architecture</span></span> 

<span data-ttu-id="e76bc-110">Архитектура состоит из следующих компонентов:</span><span class="sxs-lookup"><span data-stu-id="e76bc-110">The architecture consists of the following components.</span></span>

* <span data-ttu-id="e76bc-111">**Локальная сеть.**</span><span class="sxs-lookup"><span data-stu-id="e76bc-111">**On-premises network**.</span></span> <span data-ttu-id="e76bc-112">Частная локальная сеть, работающая внутри организации.</span><span class="sxs-lookup"><span data-stu-id="e76bc-112">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="e76bc-113">**VPN-устройство.**</span><span class="sxs-lookup"><span data-stu-id="e76bc-113">**VPN appliance**.</span></span> <span data-ttu-id="e76bc-114">Устройство или служба, предоставляющая возможность внешнего подключения к локальной сети.</span><span class="sxs-lookup"><span data-stu-id="e76bc-114">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="e76bc-115">VPN-устройство может быть аппаратным устройством или программным решением, таким как служба маршрутизации и удаленного доступа (RRAS) в Windows Server 2012.</span><span class="sxs-lookup"><span data-stu-id="e76bc-115">The VPN appliance may be a hardware device, or it can be a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="e76bc-116">Список поддерживаемых VPN-устройств и сведения об их настройке для подключения к VPN-шлюзу Azure см. в инструкциях для выбранного устройства в статье [VPN-устройства и параметры IPsec/IKE для подключений типа "сеть — сеть" через VPN-шлюз][vpn-appliance].</span><span class="sxs-lookup"><span data-stu-id="e76bc-116">For a list of supported VPN appliances and information on configuring them to connect to an Azure VPN gateway, see the instructions for the selected device in the article [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="e76bc-117">**Виртуальная сеть**.</span><span class="sxs-lookup"><span data-stu-id="e76bc-117">**Virtual network (VNet)**.</span></span> <span data-ttu-id="e76bc-118">Облачное приложение и компоненты для VPN-шлюза Azure находятся в одной [виртуальной сети][azure-virtual-network].</span><span class="sxs-lookup"><span data-stu-id="e76bc-118">The cloud application and the components for the Azure VPN gateway reside in the same [VNet][azure-virtual-network].</span></span>

* <span data-ttu-id="e76bc-119">**VPN-шлюз Azure.**</span><span class="sxs-lookup"><span data-stu-id="e76bc-119">**Azure VPN gateway**.</span></span> <span data-ttu-id="e76bc-120">Служба [VPN-шлюза][azure-vpn-gateway] позволяет вам подключать виртуальную сеть к локальной через VPN-устройство.</span><span class="sxs-lookup"><span data-stu-id="e76bc-120">The [VPN gateway][azure-vpn-gateway] service enables you to connect the VNet to the on-premises network through a VPN appliance.</span></span> <span data-ttu-id="e76bc-121">Дополнительные сведения см. в статье [Подключение локальной сети к виртуальной сети Microsoft Azure][connect-to-an-Azure-vnet].</span><span class="sxs-lookup"><span data-stu-id="e76bc-121">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span> <span data-ttu-id="e76bc-122">VPN-шлюз содержит следующие элементы:</span><span class="sxs-lookup"><span data-stu-id="e76bc-122">The VPN gateway includes the following elements:</span></span>
  
  * <span data-ttu-id="e76bc-123">**Шлюз виртуальной сети.**</span><span class="sxs-lookup"><span data-stu-id="e76bc-123">**Virtual network gateway**.</span></span> <span data-ttu-id="e76bc-124">Ресурс, предоставляющий виртуальное VPN-устройство для виртуальной сети.</span><span class="sxs-lookup"><span data-stu-id="e76bc-124">A resource that provides a virtual VPN appliance for the VNet.</span></span> <span data-ttu-id="e76bc-125">Он отвечает за маршрутизацию трафика от локальной сети к виртуальной.</span><span class="sxs-lookup"><span data-stu-id="e76bc-125">It is responsible for routing traffic from the on-premises network to the VNet.</span></span>
  * <span data-ttu-id="e76bc-126">**Шлюз локальной сети.**</span><span class="sxs-lookup"><span data-stu-id="e76bc-126">**Local network gateway**.</span></span> <span data-ttu-id="e76bc-127">Абстракция локального VPN-устройства.</span><span class="sxs-lookup"><span data-stu-id="e76bc-127">An abstraction of the on-premises VPN appliance.</span></span> <span data-ttu-id="e76bc-128">Сетевой трафик из облачного приложения к локальной сети направляется через этот шлюз.</span><span class="sxs-lookup"><span data-stu-id="e76bc-128">Network traffic from the cloud application to the on-premises network is routed through this gateway.</span></span>
  * <span data-ttu-id="e76bc-129">**Подключение.**</span><span class="sxs-lookup"><span data-stu-id="e76bc-129">**Connection**.</span></span> <span data-ttu-id="e76bc-130">У соединения есть свойства, указывающие тип соединения (IPSec) и общий ключ с VPN-устройством в локальной среде для шифрования трафика.</span><span class="sxs-lookup"><span data-stu-id="e76bc-130">The connection has properties that specify the connection type (IPSec) and the key shared with the on-premises VPN appliance to encrypt traffic.</span></span>
  * <span data-ttu-id="e76bc-131">**Подсеть шлюза.**</span><span class="sxs-lookup"><span data-stu-id="e76bc-131">**Gateway subnet**.</span></span> <span data-ttu-id="e76bc-132">Шлюз виртуальной сети находится в собственной подсети с различными требованиями, описанными ниже в разделе рекомендаций.</span><span class="sxs-lookup"><span data-stu-id="e76bc-132">The virtual network gateway is held in its own subnet, which is subject to various requirements, described in the Recommendations section below.</span></span>

* <span data-ttu-id="e76bc-133">**Облачное приложение.**</span><span class="sxs-lookup"><span data-stu-id="e76bc-133">**Cloud application**.</span></span> <span data-ttu-id="e76bc-134">Приложение, размещенное в Azure.</span><span class="sxs-lookup"><span data-stu-id="e76bc-134">The application hosted in Azure.</span></span> <span data-ttu-id="e76bc-135">Оно может содержать несколько уровней с несколькими подсетями, подключенными через подсистемы балансировки нагрузки Azure.</span><span class="sxs-lookup"><span data-stu-id="e76bc-135">It might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="e76bc-136">Дополнительные сведения об инфраструктуре приложений см. в статьях [Запуск рабочих нагрузок на виртуальных машинах Windows][windows-vm-ra] и [Запуск рабочих нагрузок на виртуальной машине Linux][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="e76bc-136">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="e76bc-137">**Внутренний балансировщик нагрузки**.</span><span class="sxs-lookup"><span data-stu-id="e76bc-137">**Internal load balancer**.</span></span> <span data-ttu-id="e76bc-138">Сетевой трафик от VPN-шлюза направляется в облачное приложение через внутреннюю подсистему балансировки нагрузки.</span><span class="sxs-lookup"><span data-stu-id="e76bc-138">Network traffic from the VPN gateway is routed to the cloud application through an internal load balancer.</span></span> <span data-ttu-id="e76bc-139">Подсистема балансировки нагрузки находится в интерфейсной подсети приложения.</span><span class="sxs-lookup"><span data-stu-id="e76bc-139">The load balancer is located in the front-end subnet of the application.</span></span>

## <a name="recommendations"></a><span data-ttu-id="e76bc-140">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="e76bc-140">Recommendations</span></span>

<span data-ttu-id="e76bc-141">Следующие рекомендации применимы для большинства ситуаций.</span><span class="sxs-lookup"><span data-stu-id="e76bc-141">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="e76bc-142">Следуйте этим рекомендациям, если они не противоречат особым требованиям для вашего случая.</span><span class="sxs-lookup"><span data-stu-id="e76bc-142">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="vnet-and-gateway-subnet"></a><span data-ttu-id="e76bc-143">Виртуальная сеть и подсеть шлюза</span><span class="sxs-lookup"><span data-stu-id="e76bc-143">VNet and gateway subnet</span></span>

<span data-ttu-id="e76bc-144">Создайте виртуальную сеть Azure с адресным пространством, способным вместить все необходимые ресурсы.</span><span class="sxs-lookup"><span data-stu-id="e76bc-144">Create an Azure VNet with an address space large enough for all of your required resources.</span></span> <span data-ttu-id="e76bc-145">Убедитесь, что адресное пространство виртуальной сети имеет достаточно места для расширения, которое может потребоваться в будущем для дополнительных виртуальных машин.</span><span class="sxs-lookup"><span data-stu-id="e76bc-145">Ensure that the VNet address space has sufficient room for growth if additional VMs are likely to be needed in the future.</span></span> <span data-ttu-id="e76bc-146">Адресное пространство виртуальной сети не должно перекрываться с локальной сетью.</span><span class="sxs-lookup"><span data-stu-id="e76bc-146">The address space of the VNet must not overlap with the on-premises network.</span></span> <span data-ttu-id="e76bc-147">Например, на схеме выше для виртуальной сети используется адресное пространство 10.20.0.0/16.</span><span class="sxs-lookup"><span data-stu-id="e76bc-147">For example, the diagram above uses the address space 10.20.0.0/16 for the VNet.</span></span>

<span data-ttu-id="e76bc-148">Создайте подсеть с именем *Подсеть шлюза* с диапазоном адресов /27.</span><span class="sxs-lookup"><span data-stu-id="e76bc-148">Create a subnet named *GatewaySubnet*, with an address range of /27.</span></span> <span data-ttu-id="e76bc-149">Эта подсеть необходима для шлюза виртуальной сети.</span><span class="sxs-lookup"><span data-stu-id="e76bc-149">This subnet is required by the virtual network gateway.</span></span> <span data-ttu-id="e76bc-150">Выделение 32 адресов этой подсети поможет предотвратить достижение ограничения размера шлюза в будущем.</span><span class="sxs-lookup"><span data-stu-id="e76bc-150">Allocating 32 addresses to this subnet will help to prevent reaching gateway size limitations in the future.</span></span> <span data-ttu-id="e76bc-151">Избегайте также размещения этой подсети в середине адресного пространства.</span><span class="sxs-lookup"><span data-stu-id="e76bc-151">Also, avoid placing this subnet in the middle of the address space.</span></span> <span data-ttu-id="e76bc-152">Мы рекомендуем задать адресное пространство подсети шлюза в верхней границе адресного пространства виртуальной сети.</span><span class="sxs-lookup"><span data-stu-id="e76bc-152">A good practice is to set the address space for the gateway subnet at the upper end of the VNet address space.</span></span> <span data-ttu-id="e76bc-153">На примере, приведенном на схеме, используется 10.20.255.224/27.</span><span class="sxs-lookup"><span data-stu-id="e76bc-153">The example shown in the diagram uses 10.20.255.224/27.</span></span>  <span data-ttu-id="e76bc-154">Вот краткая процедура вычисления [CIDR]:</span><span class="sxs-lookup"><span data-stu-id="e76bc-154">Here is a quick procedure to calculate the [CIDR]:</span></span>

1. <span data-ttu-id="e76bc-155">Установите для переменных битов в адресном пространстве виртуальной сети значение 1, по количеству битов, используемых подсетью шлюза, а затем задайте для оставшихся битов значение 0.</span><span class="sxs-lookup"><span data-stu-id="e76bc-155">Set the variable bits in the address space of the VNet to 1, up to the bits being used by the gateway subnet, then set the remaining bits to 0.</span></span>
2. <span data-ttu-id="e76bc-156">Преобразуйте получившиеся биты в десятичное число и выразите его в виде адресного пространства с длиной префикса, установленной по размеру подсети шлюза.</span><span class="sxs-lookup"><span data-stu-id="e76bc-156">Convert the resulting bits to decimal and express it as an address space with the prefix length set to the size of the gateway subnet.</span></span>

<span data-ttu-id="e76bc-157">Например, при применении шага 1 диапазон IP-адресов виртуальной сети 10.20.0.0/16 превращается в 10.20.0b11111111.0b11100000.</span><span class="sxs-lookup"><span data-stu-id="e76bc-157">For example, for a VNet with an IP address range of 10.20.0.0/16, applying step #1 above becomes 10.20.0b11111111.0b11100000.</span></span>  <span data-ttu-id="e76bc-158">При преобразовании этого десятичного числа и его выражении в виде адресного пространства получаем такой результат: 10.20.255.224/27.</span><span class="sxs-lookup"><span data-stu-id="e76bc-158">Converting that to decimal and expressing it as an address space yields 10.20.255.224/27.</span></span> 

> [!WARNING]
> <span data-ttu-id="e76bc-159">Не развертывайте виртуальные машины в подсети шлюза.</span><span class="sxs-lookup"><span data-stu-id="e76bc-159">Do not deploy any VMs to the gateway subnet.</span></span> <span data-ttu-id="e76bc-160">Кроме того, не назначайте группу безопасности сети для этой подсети, так как это приведет к прекращению работы шлюза.</span><span class="sxs-lookup"><span data-stu-id="e76bc-160">Also, do not assign an NSG to this subnet, as it will cause the gateway to stop functioning.</span></span>
> 
> 

### <a name="virtual-network-gateway"></a><span data-ttu-id="e76bc-161">Шлюз виртуальной сети</span><span class="sxs-lookup"><span data-stu-id="e76bc-161">Virtual network gateway</span></span>

<span data-ttu-id="e76bc-162">Выделите общедоступный IP-адрес для шлюза виртуальной сети.</span><span class="sxs-lookup"><span data-stu-id="e76bc-162">Allocate a public IP address for the virtual network gateway.</span></span>

<span data-ttu-id="e76bc-163">Создайте шлюз виртуальной сети в подсети шлюза и назначьте ему только что выделенный общедоступный IP-адрес.</span><span class="sxs-lookup"><span data-stu-id="e76bc-163">Create the virtual network gateway in the gateway subnet and assign it the newly allocated public IP address.</span></span> <span data-ttu-id="e76bc-164">Используйте тип шлюза, который больше всего соответствует вашим требованиям и поддерживается вашим VPN-устройством:</span><span class="sxs-lookup"><span data-stu-id="e76bc-164">Use the gateway type that most closely matches your requirements and that is enabled by your VPN appliance:</span></span>

- <span data-ttu-id="e76bc-165">Создайте [шлюз на основе политики][policy-based-routing], если необходимо тщательно управлять маршрутизацией запросов на основе критериев политики, как, например, префиксов адресов.</span><span class="sxs-lookup"><span data-stu-id="e76bc-165">Create a [policy-based gateway][policy-based-routing] if you need to closely control how requests are routed based on policy criteria such as address prefixes.</span></span> <span data-ttu-id="e76bc-166">Шлюзы на основе политики используют статическую маршрутизацию и работают только с подключениями типа "сеть — сеть".</span><span class="sxs-lookup"><span data-stu-id="e76bc-166">Policy-based gateways use static routing, and only work with site-to-site connections.</span></span>

- <span data-ttu-id="e76bc-167">Создайте [шлюз на основе маршрутов][route-based-routing] при подключении к локальной сети с помощью службы RRAS, поддержке многосайтовых или межрегиональных подключений или реализации подключения типа "виртуальная сеть — виртуальная сеть" (включая маршруты через несколько виртуальных сетей).</span><span class="sxs-lookup"><span data-stu-id="e76bc-167">Create a [route-based gateway][route-based-routing] if you connect to the on-premises network using RRAS, support multi-site or cross-region connections, or implement VNet-to-VNet connections (including routes that traverse multiple VNets).</span></span> <span data-ttu-id="e76bc-168">Шлюзы на основе маршрутов используют динамическую маршрутизацию для направления трафика между сетями.</span><span class="sxs-lookup"><span data-stu-id="e76bc-168">Route-based gateways use dynamic routing to direct traffic between networks.</span></span> <span data-ttu-id="e76bc-169">Они надежнее при сбоях в сетевом пути, чем статические маршруты, так как они могут использовать альтернативные маршруты.</span><span class="sxs-lookup"><span data-stu-id="e76bc-169">They can tolerate failures in the network path better than static routes because they can try alternative routes.</span></span> <span data-ttu-id="e76bc-170">Шлюзы на основе маршрутов могут также снижать накладные расходы на управление, так как маршруты не потребуется обновлять вручную при изменении сетевых адресов.</span><span class="sxs-lookup"><span data-stu-id="e76bc-170">Route-based gateways can also reduce the management overhead because routes might not need to be updated manually when network addresses change.</span></span>

<span data-ttu-id="e76bc-171">Список поддерживаемых VPN-устройств см. в статье [VPN-устройства и параметры IPsec/IKE для подключений типа "сеть — сеть" через VPN-шлюз][vpn-appliances].</span><span class="sxs-lookup"><span data-stu-id="e76bc-171">For a list of supported VPN appliances, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliances].</span></span>

> [!NOTE]
> <span data-ttu-id="e76bc-172">После создания шлюза изменить его тип нельзя. Для этого придется удалить шлюз, а затем создать его снова.</span><span class="sxs-lookup"><span data-stu-id="e76bc-172">After the gateway has been created, you cannot change between gateway types without deleting and re-creating the gateway.</span></span>
> 
> 

<span data-ttu-id="e76bc-173">Выберите SKU VPN-шлюза Azure, который больше всего соответствует требованиям к пропускной способности.</span><span class="sxs-lookup"><span data-stu-id="e76bc-173">Select the Azure VPN gateway SKU that most closely matches your throughput requirements.</span></span> <span data-ttu-id="e76bc-174">VPN-шлюз Azure доступен в трех SKU, как показано в следующей таблице.</span><span class="sxs-lookup"><span data-stu-id="e76bc-174">Azure VPN gateway is available in three SKUs shown in the following table.</span></span> 

| <span data-ttu-id="e76bc-175">SKU</span><span class="sxs-lookup"><span data-stu-id="e76bc-175">SKU</span></span> | <span data-ttu-id="e76bc-176">Пропускная способность VPN</span><span class="sxs-lookup"><span data-stu-id="e76bc-176">VPN Throughput</span></span> | <span data-ttu-id="e76bc-177">Макс. количество IPsec-туннелей</span><span class="sxs-lookup"><span data-stu-id="e76bc-177">Max IPSec Tunnels</span></span> |
| --- | --- | --- |
| <span data-ttu-id="e76bc-178">базовая;</span><span class="sxs-lookup"><span data-stu-id="e76bc-178">Basic</span></span> |<span data-ttu-id="e76bc-179">100 Мбит/с</span><span class="sxs-lookup"><span data-stu-id="e76bc-179">100 Mbps</span></span> |<span data-ttu-id="e76bc-180">10</span><span class="sxs-lookup"><span data-stu-id="e76bc-180">10</span></span> |
| <span data-ttu-id="e76bc-181">Стандартная</span><span class="sxs-lookup"><span data-stu-id="e76bc-181">Standard</span></span> |<span data-ttu-id="e76bc-182">100 Мбит/с</span><span class="sxs-lookup"><span data-stu-id="e76bc-182">100 Mbps</span></span> |<span data-ttu-id="e76bc-183">10</span><span class="sxs-lookup"><span data-stu-id="e76bc-183">10</span></span> |
| <span data-ttu-id="e76bc-184">высокопроизводительная</span><span class="sxs-lookup"><span data-stu-id="e76bc-184">High Performance</span></span> |<span data-ttu-id="e76bc-185">200 Мбит/с</span><span class="sxs-lookup"><span data-stu-id="e76bc-185">200 Mbps</span></span> |<span data-ttu-id="e76bc-186">30</span><span class="sxs-lookup"><span data-stu-id="e76bc-186">30</span></span> |

> [!NOTE]
> <span data-ttu-id="e76bc-187">SKU "Базовый" несовместим с Azure ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="e76bc-187">The Basic SKU is not compatible with Azure ExpressRoute.</span></span> <span data-ttu-id="e76bc-188">[SKU можно изменить][changing-SKUs] после создания шлюза.</span><span class="sxs-lookup"><span data-stu-id="e76bc-188">You can [change the SKU][changing-SKUs] after the gateway has been created.</span></span>
> 
> 

<span data-ttu-id="e76bc-189">Плата взимается за каждый час использования предоставленного и доступного шлюза.</span><span class="sxs-lookup"><span data-stu-id="e76bc-189">You are charged based on the amount of time that the gateway is provisioned and available.</span></span> <span data-ttu-id="e76bc-190">Дополнительные сведения см. на странице [цен на VPN-шлюз][azure-gateway-charges].</span><span class="sxs-lookup"><span data-stu-id="e76bc-190">See [VPN Gateway Pricing][azure-gateway-charges].</span></span>

<span data-ttu-id="e76bc-191">Создайте правила маршрутизации для подсети шлюза, направляющей входящий трафик приложения из шлюза к внутренней подсистеме балансировки нагрузки, а не разрешающей передачу запросов непосредственно в виртуальные машины приложения.</span><span class="sxs-lookup"><span data-stu-id="e76bc-191">Create routing rules for the gateway subnet that direct incoming application traffic from the gateway to the internal load balancer, rather than allowing requests to pass directly to the application VMs.</span></span>

### <a name="on-premises-network-connection"></a><span data-ttu-id="e76bc-192">Локальное сетевое подключение</span><span class="sxs-lookup"><span data-stu-id="e76bc-192">On-premises network connection</span></span>

<span data-ttu-id="e76bc-193">Создайте локальный сетевой шлюз.</span><span class="sxs-lookup"><span data-stu-id="e76bc-193">Create a local network gateway.</span></span> <span data-ttu-id="e76bc-194">Укажите общедоступный IP-адрес локального VPN-устройства, а также адресное пространство локальной сети.</span><span class="sxs-lookup"><span data-stu-id="e76bc-194">Specify the public IP address of the on-premises VPN appliance, and the address space of the on-premises network.</span></span> <span data-ttu-id="e76bc-195">Обратите внимание, что локальному VPN-устройству необходим открытый IP-адрес, доступный на шлюзе локальной сети в VPN-шлюзе Azure.</span><span class="sxs-lookup"><span data-stu-id="e76bc-195">Note that the on-premises VPN appliance must have a public IP address that can be accessed by the local network gateway in Azure VPN Gateway.</span></span> <span data-ttu-id="e76bc-196">VPN-устройство не может располагаться вне устройства преобразования сетевых адресов (NAT).</span><span class="sxs-lookup"><span data-stu-id="e76bc-196">The VPN device cannot be located behind a network address translation (NAT) device.</span></span>

<span data-ttu-id="e76bc-197">Создайте подключения типа "сеть — сеть" для шлюза виртуальной и локальной сети.</span><span class="sxs-lookup"><span data-stu-id="e76bc-197">Create a site-to-site connection for the virtual network gateway and the local network gateway.</span></span> <span data-ttu-id="e76bc-198">Выберите подключение типа "сеть — сеть" (IPSec) и укажите общий ключ.</span><span class="sxs-lookup"><span data-stu-id="e76bc-198">Select the site-to-site (IPSec) connection type, and specify the shared key.</span></span> <span data-ttu-id="e76bc-199">Подключение типа "сеть — сеть" с VPN-шлюзом Azure шифруется на основе протокола IPSec с использованием предварительного ключа для проверки подлинности.</span><span class="sxs-lookup"><span data-stu-id="e76bc-199">Site-to-site encryption with the Azure VPN gateway is based on the IPSec protocol, using preshared keys for authentication.</span></span> <span data-ttu-id="e76bc-200">Укажите ключ при создании VPN-шлюза Azure.</span><span class="sxs-lookup"><span data-stu-id="e76bc-200">You specify the key when you create the Azure VPN gateway.</span></span> <span data-ttu-id="e76bc-201">Необходимо настроить VPN-устройство, запущенное локально с тем же ключом.</span><span class="sxs-lookup"><span data-stu-id="e76bc-201">You must configure the VPN appliance running on-premises with the same key.</span></span> <span data-ttu-id="e76bc-202">Сейчас другие механизмы проверки подлинности не поддерживаются.</span><span class="sxs-lookup"><span data-stu-id="e76bc-202">Other authentication mechanisms are not currently supported.</span></span>

<span data-ttu-id="e76bc-203">Убедитесь, что инфраструктура локальной маршрутизации настроена для переадресации запросов по адресам в виртуальной сети Azure VPN-устройству.</span><span class="sxs-lookup"><span data-stu-id="e76bc-203">Ensure that the on-premises routing infrastructure is configured to forward requests intended for addresses in the Azure VNet to the VPN device.</span></span>

<span data-ttu-id="e76bc-204">В локальной сети откройте любые порты, необходимые для облачного приложения.</span><span class="sxs-lookup"><span data-stu-id="e76bc-204">Open any ports required by the cloud application in the on-premises network.</span></span>

<span data-ttu-id="e76bc-205">Проверьте подключение, чтобы убедиться в следующем:</span><span class="sxs-lookup"><span data-stu-id="e76bc-205">Test the connection to verify that:</span></span>

* <span data-ttu-id="e76bc-206">Локальное VPN-устройство правильно направляет трафик в облачное приложение через VPN-шлюз Azure.</span><span class="sxs-lookup"><span data-stu-id="e76bc-206">The on-premises VPN appliance correctly routes traffic to the cloud application through the Azure VPN gateway.</span></span>
* <span data-ttu-id="e76bc-207">Виртуальная сеть правильно направляет трафик обратно в локальную сеть.</span><span class="sxs-lookup"><span data-stu-id="e76bc-207">The VNet correctly routes traffic back to the on-premises network.</span></span>
* <span data-ttu-id="e76bc-208">Запрещенный трафик в обоих направлениях блокируется соответствующим образом.</span><span class="sxs-lookup"><span data-stu-id="e76bc-208">Prohibited traffic in both directions is blocked correctly.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="e76bc-209">Вопросы масштабируемости</span><span class="sxs-lookup"><span data-stu-id="e76bc-209">Scalability considerations</span></span>

<span data-ttu-id="e76bc-210">Вы можете достичь предела вертикального масштабирования, перейдя с SKU "Базовый" или "Стандартный" VPN-шлюза к SKU VPN "Высокопроизводительный".</span><span class="sxs-lookup"><span data-stu-id="e76bc-210">You can achieve limited vertical scalability by moving from the Basic or Standard VPN Gateway SKUs to the High Performance VPN SKU.</span></span>

<span data-ttu-id="e76bc-211">Для виртуальных сетей, в которых ожидается большой объем VPN-трафика, рассмотрите возможность распределения разных рабочих нагрузок по отдельным небольшим виртуальным сетям и настройки VPN-шлюза для каждой из них.</span><span class="sxs-lookup"><span data-stu-id="e76bc-211">For VNets that expect a large volume of VPN traffic, consider distributing the different workloads into separate smaller VNets and configuring a VPN gateway for each of them.</span></span>

<span data-ttu-id="e76bc-212">Виртуальную сеть можно разделить горизонтально или вертикально.</span><span class="sxs-lookup"><span data-stu-id="e76bc-212">You can partition the VNet either horizontally or vertically.</span></span> <span data-ttu-id="e76bc-213">Для разделения по горизонтали переместите некоторые экземпляры виртуальных машин из каждого уровня в подсети новой виртуальной сети.</span><span class="sxs-lookup"><span data-stu-id="e76bc-213">To partition horizontally, move some VM instances from each tier into subnets of the new VNet.</span></span> <span data-ttu-id="e76bc-214">При этом каждая виртуальная сеть будет иметь одинаковую структуру и функциональные возможности.</span><span class="sxs-lookup"><span data-stu-id="e76bc-214">The result is that each VNet has the same structure and functionality.</span></span> <span data-ttu-id="e76bc-215">Для разделения по вертикали переопределите каждый уровень, чтобы разделить функциональные возможности на разные логические области (например, обработка заказов, выставление счетов, управление учетными записями пользователей и т. д.).</span><span class="sxs-lookup"><span data-stu-id="e76bc-215">To partition vertically, redesign each tier to divide the functionality into different logical areas (such as handling orders, invoicing, customer account management, and so on).</span></span> <span data-ttu-id="e76bc-216">Каждую функциональную область можно поместить в собственную виртуальную сеть.</span><span class="sxs-lookup"><span data-stu-id="e76bc-216">Each functional area can then be placed in its own VNet.</span></span>

<span data-ttu-id="e76bc-217">Репликация локального контроллера домена Active Directory в виртуальной сети, а также реализация DNS в виртуальной сети помогает сократить объем трафика, связанного с безопасностью и администрированием, передаваемого из локальной среды в облако.</span><span class="sxs-lookup"><span data-stu-id="e76bc-217">Replicating an on-premises Active Directory domain controller in the VNet, and implementing DNS in the VNet, can help to reduce some of the security-related and administrative traffic flowing from on-premises to the cloud.</span></span> <span data-ttu-id="e76bc-218">Дополнительные сведения см. в статье [Расширение доменных служб Active Directory в Azure][adds-extend-domain].</span><span class="sxs-lookup"><span data-stu-id="e76bc-218">For more information, see [Extending Active Directory Domain Services (AD DS) to Azure][adds-extend-domain].</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="e76bc-219">Вопросы доступности</span><span class="sxs-lookup"><span data-stu-id="e76bc-219">Availability considerations</span></span>

<span data-ttu-id="e76bc-220">Если необходимо убедиться, что локальная сеть остается доступной для VPN-шлюза Azure, реализуйте отказоустойчивый кластер для локального VPN-шлюза.</span><span class="sxs-lookup"><span data-stu-id="e76bc-220">If you need to ensure that the on-premises network remains available to the Azure VPN gateway, implement a failover cluster for the on-premises VPN gateway.</span></span>

<span data-ttu-id="e76bc-221">Если в вашей организации есть несколько локальных сайтов, создайте [многосайтовые подключения][vpn-gateway-multi-site] для одной или нескольких виртуальных сетей Azure.</span><span class="sxs-lookup"><span data-stu-id="e76bc-221">If your organization has multiple on-premises sites, create [multi-site connections][vpn-gateway-multi-site] to one or more Azure VNets.</span></span> <span data-ttu-id="e76bc-222">Этот подход требует динамической (на основе маршрута) маршрутизации, поэтому убедитесь, что локальный VPN-шлюз поддерживает эту функцию.</span><span class="sxs-lookup"><span data-stu-id="e76bc-222">This approach requires dynamic (route-based) routing, so make sure that the on-premises VPN gateway supports this feature.</span></span>

<span data-ttu-id="e76bc-223">Дополнительные сведения о соглашениях об уровне обслуживания для VPN-шлюза см. [здесь][sla-for-vpn-gateway].</span><span class="sxs-lookup"><span data-stu-id="e76bc-223">For details about service level agreements, see [SLA for VPN Gateway][sla-for-vpn-gateway].</span></span> 

## <a name="manageability-considerations"></a><span data-ttu-id="e76bc-224">Вопросы управляемости</span><span class="sxs-lookup"><span data-stu-id="e76bc-224">Manageability considerations</span></span>

<span data-ttu-id="e76bc-225">Наблюдайте за диагностическими сведениями с локального VPN-устройства.</span><span class="sxs-lookup"><span data-stu-id="e76bc-225">Monitor diagnostic information from on-premises VPN appliances.</span></span> <span data-ttu-id="e76bc-226">Этот процесс зависит от функций, предоставляемых VPN-устройством.</span><span class="sxs-lookup"><span data-stu-id="e76bc-226">This process depends on the features provided by the VPN appliance.</span></span> <span data-ttu-id="e76bc-227">Например, если используется служба маршрутизации и удаленного доступа на базе Windows Server 2012 ([ведение журнала для RRAS][rras-logging]).</span><span class="sxs-lookup"><span data-stu-id="e76bc-227">For example, if you are using the Routing and Remote Access Service on Windows Server 2012, [RRAS logging][rras-logging].</span></span>

<span data-ttu-id="e76bc-228">Используйте [диагностику VPN-шлюза Azure][gateway-diagnostic-logs], чтобы получить сведения о проблемах с подключением.</span><span class="sxs-lookup"><span data-stu-id="e76bc-228">Use [Azure VPN gateway diagnostics][gateway-diagnostic-logs] to capture information about connectivity issues.</span></span> <span data-ttu-id="e76bc-229">Эти журналы можно использовать для отслеживания сведений, таких как источник и назначение запросов на соединение, использованный протокол и способ установки подключения (или причина сбоя).</span><span class="sxs-lookup"><span data-stu-id="e76bc-229">These logs can be used to track information such as the source and destinations of connection requests, which protocol was used, and how the connection was established (or why the attempt failed).</span></span>

<span data-ttu-id="e76bc-230">Отслеживайте операционные журналы VPN-шлюза Azure с помощью журналов аудита, доступных на портале Azure.</span><span class="sxs-lookup"><span data-stu-id="e76bc-230">Monitor the operational logs of the Azure VPN gateway using the audit logs available in the Azure portal.</span></span> <span data-ttu-id="e76bc-231">Отдельные журналы доступны для шлюза локальной сети, шлюза сети Azure и подключения.</span><span class="sxs-lookup"><span data-stu-id="e76bc-231">Separate logs are available for the local network gateway, the Azure network gateway, and the connection.</span></span> <span data-ttu-id="e76bc-232">Эти сведения можно использовать для отслеживания любых изменений шлюза. Они пригодятся, если функционирующий шлюз вдруг прекратит свою работу по какой-либо причине.</span><span class="sxs-lookup"><span data-stu-id="e76bc-232">This information can be used to track any changes made to the gateway, and can be useful if a previously functioning gateway stops working for some reason.</span></span>

<span data-ttu-id="e76bc-233">![[2]][2]</span><span class="sxs-lookup"><span data-stu-id="e76bc-233">![[2]][2]</span></span>

<span data-ttu-id="e76bc-234">Наблюдайте за подключениями и отслеживайте соответствующие сбои.</span><span class="sxs-lookup"><span data-stu-id="e76bc-234">Monitor connectivity, and track connectivity failure events.</span></span> <span data-ttu-id="e76bc-235">Для регистрации этих сведений и формирования отчетов на их основе можно воспользоваться пакетом для мониторинга, например [Nagios][nagios].</span><span class="sxs-lookup"><span data-stu-id="e76bc-235">You can use a monitoring package such as [Nagios][nagios] to capture and report this information.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="e76bc-236">Вопросы безопасности</span><span class="sxs-lookup"><span data-stu-id="e76bc-236">Security considerations</span></span>

<span data-ttu-id="e76bc-237">Создайте разные общие ключи для каждого VPN-шлюза.</span><span class="sxs-lookup"><span data-stu-id="e76bc-237">Generate a different shared key for each VPN gateway.</span></span> <span data-ttu-id="e76bc-238">Используйте надежный общий ключ для защиты от атак методом подбора.</span><span class="sxs-lookup"><span data-stu-id="e76bc-238">Use a strong shared key to help resist brute-force attacks.</span></span>

> [!NOTE]
> <span data-ttu-id="e76bc-239">В настоящее время нельзя использовать Azure Key Vault для предварительного предоставления ключей VPN-шлюза Azure.</span><span class="sxs-lookup"><span data-stu-id="e76bc-239">Currently, you cannot use Azure Key Vault to preshare keys for the Azure VPN gateway.</span></span>
> 
> 

<span data-ttu-id="e76bc-240">Убедитесь, что локальное VPN-устройство использует метод шифрования, [совместимый с VPN-шлюзом Azure][vpn-appliance-ipsec].</span><span class="sxs-lookup"><span data-stu-id="e76bc-240">Ensure that the on-premises VPN appliance uses an encryption method that is [compatible with the Azure VPN gateway][vpn-appliance-ipsec].</span></span> <span data-ttu-id="e76bc-241">Для маршрутизации на основе политик VPN-шлюз Azure поддерживает алгоритмы шифрования AES256, AES128 и 3DES.</span><span class="sxs-lookup"><span data-stu-id="e76bc-241">For policy-based routing, the Azure VPN gateway supports the AES256, AES128, and 3DES encryption algorithms.</span></span> <span data-ttu-id="e76bc-242">Шлюзы на основе маршрута поддерживают AES256 и 3DES.</span><span class="sxs-lookup"><span data-stu-id="e76bc-242">Route-based gateways support AES256 and 3DES.</span></span>

<span data-ttu-id="e76bc-243">Если ваше локальное VPN-устройство находится в сети периметра с брандмауэром между ней и Интернетом, то может потребоваться настроить [дополнительные правила брандмауэра][additional-firewall-rules], разрешающие VPN-подключение "сеть — сеть".</span><span class="sxs-lookup"><span data-stu-id="e76bc-243">If your on-premises VPN appliance is on a perimeter network (DMZ) that has a firewall between the perimeter network and the Internet, you might have to configure [additional firewall rules][additional-firewall-rules] to allow the site-to-site VPN connection.</span></span>

<span data-ttu-id="e76bc-244">Если приложение в виртуальной сети отправляет данные в Интернет, следует рассмотреть возможность [реализации принудительного туннелирования][forced-tunneling], чтобы направлять весь интернет-трафик через локальную сеть.</span><span class="sxs-lookup"><span data-stu-id="e76bc-244">If the application in the VNet sends data to the Internet, consider [implementing forced tunneling][forced-tunneling] to route all Internet-bound traffic through the on-premises network.</span></span> <span data-ttu-id="e76bc-245">Такой подход позволяет производить аудит исходящих запросов приложения из локальной инфраструктуры.</span><span class="sxs-lookup"><span data-stu-id="e76bc-245">This approach enables you to audit outgoing requests made by the application from the on-premises infrastructure.</span></span>

> [!NOTE]
> <span data-ttu-id="e76bc-246">Принудительное туннелирование может повлиять на возможность подключения к службам Azure (например, к службе хранилища) и к службе Windows License Manager.</span><span class="sxs-lookup"><span data-stu-id="e76bc-246">Forced tunneling can impact connectivity to Azure services (the Storage Service, for example) and the Windows license manager.</span></span>
> 
> 


## <a name="troubleshooting"></a><span data-ttu-id="e76bc-247">Устранение неполадок</span><span class="sxs-lookup"><span data-stu-id="e76bc-247">Troubleshooting</span></span> 

<span data-ttu-id="e76bc-248">Общие сведения об устранении неполадок, связанных с VPN, см. [здесь][troubleshooting-vpn-errors].</span><span class="sxs-lookup"><span data-stu-id="e76bc-248">For general information on troubleshooting common VPN-related errors, see [Troubleshooting common VPN related errors][troubleshooting-vpn-errors].</span></span>

<span data-ttu-id="e76bc-249">Следующие рекомендации помогут определить, правильно ли работает ваше локальное VPN-устройство.</span><span class="sxs-lookup"><span data-stu-id="e76bc-249">The following recommendations are useful for determining if your on-premises VPN appliance is functioning correctly.</span></span>

- <span data-ttu-id="e76bc-250">**Проверьте все файлы журнала, созданные VPN-устройством, на наличие ошибок или сбоев.**</span><span class="sxs-lookup"><span data-stu-id="e76bc-250">**Check any log files generated by the VPN appliance for errors or failures.**</span></span>

    <span data-ttu-id="e76bc-251">Так вы сможете определить, правильно ли функционирует VPN-устройство.</span><span class="sxs-lookup"><span data-stu-id="e76bc-251">This will help you determine if the VPN appliance is functioning correctly.</span></span> <span data-ttu-id="e76bc-252">Расположение этих сведений будет различаться в зависимости от устройства.</span><span class="sxs-lookup"><span data-stu-id="e76bc-252">The location of this information will vary according to your appliance.</span></span> <span data-ttu-id="e76bc-253">Например, при использовании RRAS на базе Windows Server 2012 для отображения сведений об ошибке события для службы RRAS можно использовать следующую команду PowerShell:</span><span class="sxs-lookup"><span data-stu-id="e76bc-253">For example, if you are using RRAS on Windows Server 2012, you can use the following PowerShell command to display error event information for the RRAS service:</span></span>

    ```PowerShell
    Get-EventLog -LogName System -EntryType Error -Source RemoteAccess | Format-List -Property *
    ```

    <span data-ttu-id="e76bc-254">В свойстве *Message* каждой записи содержится описание ошибки.</span><span class="sxs-lookup"><span data-stu-id="e76bc-254">The *Message* property of each entry provides a description of the error.</span></span> <span data-ttu-id="e76bc-255">Несколько распространенных примеров:</span><span class="sxs-lookup"><span data-stu-id="e76bc-255">Some common examples are:</span></span>

        - Inability to connect, possibly due to an incorrect IP address specified for the Azure VPN gateway in the RRAS VPN network interface configuration.

        ```
        EventID            : 20111
        MachineName        : on-prem-vm
        Data               : {41, 3, 0, 0}
        Index              : 14231
        Category           : (0)
        CategoryNumber     : 0
        EntryType          : Error
        Message            : RoutingDomainID- {00000000-0000-0000-0000-000000000000}: A demand dial connection to the remote
                             interface AzureGateway on port VPN2-4 was successfully initiated but failed to complete
                             successfully because of the  following error: The network connection between your computer and
                             the VPN server could not be established because the remote server is not responding. This could
                             be because one of the network devices (for example, firewalls, NAT, routers, and so on) between your computer
                             and the remote server is not configured to allow VPN connections. Please contact your
                             Administrator or your service provider to determine which device may be causing the problem.
        Source             : RemoteAccess
        ReplacementStrings : {{00000000-0000-0000-0000-000000000000}, AzureGateway, VPN2-4, The network connection between
                             your computer and the VPN server could not be established because the remote server is not
                             responding. This could be because one of the network devices (for example, firewalls, NAT, routers, and so on)
                             between your computer and the remote server is not configured to allow VPN connections. Please
                             contact your Administrator or your service provider to determine which device may be causing the
                             problem.}
        InstanceId         : 20111
        TimeGenerated      : 3/18/2016 1:26:02 PM
        TimeWritten        : 3/18/2016 1:26:02 PM
        UserName           :
        Site               :
        Container          :
        ```

        - The wrong shared key being specified in the RRAS VPN network interface configuration.

        ```
        EventID            : 20111
        MachineName        : on-prem-vm
        Data               : {233, 53, 0, 0}
        Index              : 14245
        Category           : (0)
        CategoryNumber     : 0
        EntryType          : Error
        Message            : RoutingDomainID- {00000000-0000-0000-0000-000000000000}: A demand dial connection to the remote
                             interface AzureGateway on port VPN2-4 was successfully initiated but failed to complete
                             successfully because of the  following error: Internet key exchange (IKE) authentication credentials are unacceptable.

        Source             : RemoteAccess
        ReplacementStrings : {{00000000-0000-0000-0000-000000000000}, AzureGateway, VPN2-4, IKE authentication credentials are
                             unacceptable.
                             }
        InstanceId         : 20111
        TimeGenerated      : 3/18/2016 1:34:22 PM
        TimeWritten        : 3/18/2016 1:34:22 PM
        UserName           :
        Site               :
        Container          :
        ```

    <span data-ttu-id="e76bc-256">Вы также можете получить сведения из журнала событий о попытках установить подключение через службу RRAS с помощью следующей команды PowerShell:</span><span class="sxs-lookup"><span data-stu-id="e76bc-256">You can also obtain event log information about attempts to connect through the RRAS service using the following PowerShell command:</span></span> 

    ```
    Get-EventLog -LogName Application -Source RasClient | Format-List -Property *
    ```

    <span data-ttu-id="e76bc-257">В случае сбоя подключения этот журнал будет содержать ошибки, которые выглядят следующим образом:</span><span class="sxs-lookup"><span data-stu-id="e76bc-257">In the event of a failure to connect, this log will contain errors that look similar to the following:</span></span>

    ```
    EventID            : 20227
    MachineName        : on-prem-vm
    Data               : {}
    Index              : 4203
    Category           : (0)
    CategoryNumber     : 0
    EntryType          : Error
    Message            : CoId={B4000371-A67F-452F-AA4C-3125AA9CFC78}: The user SYSTEM dialed a connection named
                         AzureGateway that has failed. The error code returned on failure is 809.
    Source             : RasClient
    ReplacementStrings : {{B4000371-A67F-452F-AA4C-3125AA9CFC78}, SYSTEM, AzureGateway, 809}
    InstanceId         : 20227
    TimeGenerated      : 3/18/2016 1:29:21 PM
    TimeWritten        : 3/18/2016 1:29:21 PM
    UserName           :
    Site               :
    Container          :
    ```

- <span data-ttu-id="e76bc-258">**Проверьте подключение и маршрутизацию через VPN-шлюз.**</span><span class="sxs-lookup"><span data-stu-id="e76bc-258">**Verify connectivity and routing across the VPN gateway.**</span></span>

    <span data-ttu-id="e76bc-259">VPN-устройство может неправильно осуществлять маршрутизацию трафика через VPN-шлюз Azure.</span><span class="sxs-lookup"><span data-stu-id="e76bc-259">The VPN appliance may not be correctly routing traffic through the Azure VPN Gateway.</span></span> <span data-ttu-id="e76bc-260">Используйте [PsPing][psping] для проверки подключения и маршрутизации через VPN-шлюз.</span><span class="sxs-lookup"><span data-stu-id="e76bc-260">Use a tool such as [PsPing][psping] to verify connectivity and routing across the VPN gateway.</span></span> <span data-ttu-id="e76bc-261">Например, чтобы проверить соединение с локального компьютера на веб-сервер, расположенный в виртуальной сети, выполните следующую команду (заменив `<<web-server-address>>` адресом веб-сервера):</span><span class="sxs-lookup"><span data-stu-id="e76bc-261">For example, to test connectivity from an on-premises machine to a web server located on the VNet, run the following command (replacing `<<web-server-address>>` with the address of the web server):</span></span>

    ```
    PsPing -t <<web-server-address>>:80
    ```

    <span data-ttu-id="e76bc-262">Если локальный компьютер может осуществлять маршрутизацию трафика на веб-сервер, отобразится результат, аналогичный приведенному ниже:</span><span class="sxs-lookup"><span data-stu-id="e76bc-262">If the on-premises machine can route traffic to the web server, you should see output similar to the following:</span></span>

    ```
    D:\PSTools>psping -t 10.20.0.5:80

    PsPing v2.01 - PsPing - ping, latency, bandwidth measurement utility
    Copyright (C) 2012-2014 Mark Russinovich
    Sysinternals - www.sysinternals.com

    TCP connect to 10.20.0.5:80:
    Infinite iterations (warmup 1) connecting test:
    Connecting to 10.20.0.5:80 (warmup): 6.21ms
    Connecting to 10.20.0.5:80: 3.79ms
    Connecting to 10.20.0.5:80: 3.44ms
    Connecting to 10.20.0.5:80: 4.81ms

      Sent = 3, Received = 3, Lost = 0 (0% loss),
      Minimum = 3.44ms, Maximum = 4.81ms, Average = 4.01ms
    ```

    <span data-ttu-id="e76bc-263">Если локальный компьютер не может связаться с указанным адресом назначения, появятся такие сообщения:</span><span class="sxs-lookup"><span data-stu-id="e76bc-263">If the on-premises machine cannot communicate with the specified destination, you will see messages like this:</span></span>

    ```
    D:\PSTools>psping -t 10.20.1.6:80

    PsPing v2.01 - PsPing - ping, latency, bandwidth measurement utility
    Copyright (C) 2012-2014 Mark Russinovich
    Sysinternals - www.sysinternals.com

    TCP connect to 10.20.1.6:80:
    Infinite iterations (warmup 1) connecting test:
    Connecting to 10.20.1.6:80 (warmup): This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80:
      Sent = 3, Received = 0, Lost = 3 (100% loss),
      Minimum = 0.00ms, Maximum = 0.00ms, Average = 0.00ms
    ```

- <span data-ttu-id="e76bc-264">**Убедитесь, что локальный брандмауэр разрешает передачу VPN-трафика и что открыты соответствующие порты.**</span><span class="sxs-lookup"><span data-stu-id="e76bc-264">**Verify that the on-premises firewall allows VPN traffic to pass and that the correct ports are opened.**</span></span>

- <span data-ttu-id="e76bc-265">**Убедитесь, что локальное VPN-устройство использует метод шифрования, [совместимый с VPN-шлюзом Azure][vpn-appliance]**.</span><span class="sxs-lookup"><span data-stu-id="e76bc-265">**Verify that the on-premises VPN appliance uses an encryption method that is [compatible with the Azure VPN gateway][vpn-appliance].**</span></span> <span data-ttu-id="e76bc-266">Для маршрутизации на основе политик VPN-шлюз Azure поддерживает алгоритмы шифрования AES256, AES128 и 3DES.</span><span class="sxs-lookup"><span data-stu-id="e76bc-266">For policy-based routing, the Azure VPN gateway supports the AES256, AES128, and 3DES encryption algorithms.</span></span> <span data-ttu-id="e76bc-267">Шлюзы на основе маршрута поддерживают AES256 и 3DES.</span><span class="sxs-lookup"><span data-stu-id="e76bc-267">Route-based gateways support AES256 and 3DES.</span></span>

<span data-ttu-id="e76bc-268">Для определения проблемы с VPN-шлюзом Azure полезны следующие рекомендации:</span><span class="sxs-lookup"><span data-stu-id="e76bc-268">The following recommendations are useful for determining if there is a problem with the Azure VPN gateway:</span></span>

- <span data-ttu-id="e76bc-269">**Изучите [журналы диагностики VPN-шлюза Azure][gateway-diagnostic-logs] на наличие потенциальных проблем.**</span><span class="sxs-lookup"><span data-stu-id="e76bc-269">**Examine [Azure VPN gateway diagnostic logs][gateway-diagnostic-logs] for potential issues.**</span></span>

- <span data-ttu-id="e76bc-270">**Убедитесь, что VPN-шлюз Azure и локальное VPN-устройство настроены с использованием одного и того же общего ключа проверки подлинности.**</span><span class="sxs-lookup"><span data-stu-id="e76bc-270">**Verify that the Azure VPN gateway and on-premises VPN appliance are configured with the same shared authentication key.**</span></span>

    <span data-ttu-id="e76bc-271">Вы можете просмотреть общий ключ, сохраненный VPN-шлюзом Azure, используя следующую команду Azure CLI:</span><span class="sxs-lookup"><span data-stu-id="e76bc-271">You can view the shared key stored by the Azure VPN gateway using the following Azure CLI command:</span></span>

    ```
    azure network vpn-connection shared-key show <<resource-group>> <<vpn-connection-name>>
    ```

    <span data-ttu-id="e76bc-272">Для просмотра общего ключа, настроенного для этого устройства, используйте команду, подходящую для вашего локального VPN-устройства.</span><span class="sxs-lookup"><span data-stu-id="e76bc-272">Use the command appropriate for your on-premises VPN appliance to show the shared key configured for that appliance.</span></span>

    <span data-ttu-id="e76bc-273">Убедитесь, что подсеть *GatewaySubnet*, в которой размещен VPN-шлюз Azure, не связана с NSG.</span><span class="sxs-lookup"><span data-stu-id="e76bc-273">Verify that the *GatewaySubnet* subnet holding the Azure VPN gateway is not associated with an NSG.</span></span>

    <span data-ttu-id="e76bc-274">Сведения о подсети можно просмотреть, используя следующую команду Azure CLI:</span><span class="sxs-lookup"><span data-stu-id="e76bc-274">You can view the subnet details using the following Azure CLI command:</span></span>

    ```
    azure network vnet subnet show -g <<resource-group>> -e <<vnet-name>> -n GatewaySubnet
    ```

    <span data-ttu-id="e76bc-275">Убедитесь, что нет поля данных с именем *Идентификатор группы безопасности сети*. В следующем примере показаны результаты для экземпляра подсети *GatewaySubnet* с назначенной NSG (*VPN-Gateway-Group*).</span><span class="sxs-lookup"><span data-stu-id="e76bc-275">Ensure there is no data field named *Network Security Group id*. The following example shows the results for an instance of the *GatewaySubnet* that has an assigned NSG (*VPN-Gateway-Group*).</span></span> <span data-ttu-id="e76bc-276">Если для этой NSG определены какие-либо правила, шлюз может неправильно работать.</span><span class="sxs-lookup"><span data-stu-id="e76bc-276">This can prevent the gateway from working correctly if there are any rules defined for this NSG.</span></span>

    ```
    C:\>azure network vnet subnet show -g profx-prod-rg -e profx-vnet -n GatewaySubnet
        info:    Executing command network vnet subnet show
        + Looking up virtual network "profx-vnet"
        + Looking up the subnet "GatewaySubnet"
        data:    Id                              : /subscriptions/########-####-####-####-############/resourceGroups/profx-prod-rg/providers/Microsoft.Network/virtualNetworks/profx-vnet/subnets/GatewaySubnet
        data:    Name                            : GatewaySubnet
        data:    Provisioning state              : Succeeded
        data:    Address prefix                  : 10.20.3.0/27
        data:    Network Security Group id       : /subscriptions/########-####-####-####-############/resourceGroups/profx-prod-rg/providers/Microsoft.Network/networkSecurityGroups/VPN-Gateway-Group
        info:    network vnet subnet show command OK
    ```

- <span data-ttu-id="e76bc-277">**Убедитесь, что виртуальные машины в виртуальной сети Azure настроены для разрешения передачи трафика, поступающего из виртуальной сети.**</span><span class="sxs-lookup"><span data-stu-id="e76bc-277">**Verify that the virtual machines in the Azure VNet are configured to permit traffic coming in from outside the VNet.**</span></span>

    <span data-ttu-id="e76bc-278">Проверьте все правила NSG, связанные с подсетями, содержащими эти виртуальные машины.</span><span class="sxs-lookup"><span data-stu-id="e76bc-278">Check any NSG rules associated with subnets containing these virtual machines.</span></span> <span data-ttu-id="e76bc-279">Все правила NSG можно просмотреть, используя следующую команду Azure CLI:</span><span class="sxs-lookup"><span data-stu-id="e76bc-279">You can view all NSG rules using the following Azure CLI command:</span></span>

    ```
    azure network nsg show -g <<resource-group>> -n <<nsg-name>>
    ```

- <span data-ttu-id="e76bc-280">**Убедитесь, что подключен VPN-шлюз Azure.**</span><span class="sxs-lookup"><span data-stu-id="e76bc-280">**Verify that the Azure VPN gateway is connected.**</span></span>

    <span data-ttu-id="e76bc-281">Чтобы проверить текущее состояние VPN-подключения Azure, можно использовать следующую команду Azure PowerShell.</span><span class="sxs-lookup"><span data-stu-id="e76bc-281">You can use the following Azure PowerShell command to check the current status of the Azure VPN connection.</span></span> <span data-ttu-id="e76bc-282">Параметр `<<connection-name>>` представляет собой имя VPN-подключения Azure, которое связывает шлюз виртуальной сети с локальным.</span><span class="sxs-lookup"><span data-stu-id="e76bc-282">The `<<connection-name>>` parameter is the name of the Azure VPN connection that links the virtual network gateway and the local gateway.</span></span>

    ```
    Get-AzureRmVirtualNetworkGatewayConnection -Name <<connection-name>> - ResourceGroupName <<resource-group>>
    ```

    <span data-ttu-id="e76bc-283">В следующем фрагменте кода показаны результаты для подключенного (первый пример) и отключенного шлюза (второй пример):</span><span class="sxs-lookup"><span data-stu-id="e76bc-283">The following snippets highlight the output generated if the gateway is connected (the first example), and disconnected (the second example):</span></span>

    ```
    PS C:\> Get-AzureRmVirtualNetworkGatewayConnection -Name profx-gateway-connection -ResourceGroupName profx-prod-rg

    AuthorizationKey           :
    VirtualNetworkGateway1     : Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
    VirtualNetworkGateway2     :
    LocalNetworkGateway2       : Microsoft.Azure.Commands.Network.Models.PSLocalNetworkGateway
    Peer                       :
    ConnectionType             : IPsec
    RoutingWeight              : 0
    SharedKey                  : ####################################
    ConnectionStatus           : Connected
    EgressBytesTransferred     : 55254803
    IngressBytesTransferred    : 32227221
    ProvisioningState          : Succeeded
    ...
    ```

    ```
    PS C:\> Get-AzureRmVirtualNetworkGatewayConnection -Name profx-gateway-connection2 -ResourceGroupName profx-prod-rg

    AuthorizationKey           :
    VirtualNetworkGateway1     : Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
    VirtualNetworkGateway2     :
    LocalNetworkGateway2       : Microsoft.Azure.Commands.Network.Models.PSLocalNetworkGateway
    Peer                       :
    ConnectionType             : IPsec
    RoutingWeight              : 0
    SharedKey                  : ####################################
    ConnectionStatus           : NotConnected
    EgressBytesTransferred     : 0
    IngressBytesTransferred    : 0
    ProvisioningState          : Succeeded
    ...
    ```

<span data-ttu-id="e76bc-284">Следующие рекомендации помогут определить, имеются ли проблемы с конфигурацией виртуальной машины узла, использованием пропускной способности сети или производительностью приложения:</span><span class="sxs-lookup"><span data-stu-id="e76bc-284">The following recommendations are useful for determining if there is an issue with Host VM configuration, network bandwidth utilization, or application performance:</span></span>

- <span data-ttu-id="e76bc-285">**Проверьте правильность настройки брандмауэра в гостевой ОС на виртуальных машинах Azure в подсети, чтобы разрешить передачу трафика из локальных диапазонов IP-адресов.**</span><span class="sxs-lookup"><span data-stu-id="e76bc-285">**Verify that the firewall in the guest operating system running on the Azure VMs in the subnet is configured correctly to allow permitted traffic from the on-premises IP ranges.**</span></span>

- <span data-ttu-id="e76bc-286">**Убедитесь, что объем трафика не приближается к лимиту пропускной способности VPN-шлюза Azure.**</span><span class="sxs-lookup"><span data-stu-id="e76bc-286">**Verify that the volume of traffic is not close to the limit of the bandwidth available to the Azure VPN gateway.**</span></span>

    <span data-ttu-id="e76bc-287">Это можно проверить по локальному запущенному VPN-устройству.</span><span class="sxs-lookup"><span data-stu-id="e76bc-287">How to verify this depends on the VPN appliance running on-premises.</span></span> <span data-ttu-id="e76bc-288">Например, при использовании RRAS на базе Windows Server 2012 можно использовать системный монитор для отслеживания объема данных, полученных и переданных по VPN-подключению.</span><span class="sxs-lookup"><span data-stu-id="e76bc-288">For example, if you are using RRAS on Windows Server 2012, you can use Performance Monitor to track the volume of data being received and transmitted over the VPN connection.</span></span> <span data-ttu-id="e76bc-289">С помощью объекта *Всего RAS* выберите счетчики *Получено байт/сек* и *Передано байт/сек*:</span><span class="sxs-lookup"><span data-stu-id="e76bc-289">Using the *RAS Total* object, select the *Bytes Received/Sec* and *Bytes Transmitted/Sec* counters:</span></span>

    <span data-ttu-id="e76bc-290">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="e76bc-290">![[3]][3]</span></span>

    <span data-ttu-id="e76bc-291">Следует сравнить результаты с пропускной способностью VPN-шлюза (100 Мбит/с для SKU "Базовый" или "Стандартный" и 200 Мбит/с для SKU "Высокопроизводительный"):</span><span class="sxs-lookup"><span data-stu-id="e76bc-291">You should compare the results with the bandwidth available to the VPN gateway (100 Mbps for the Basic and Standard SKUs, and 200 Mbps for the High Performance SKU):</span></span>

    <span data-ttu-id="e76bc-292">![[4]][4]</span><span class="sxs-lookup"><span data-stu-id="e76bc-292">![[4]][4]</span></span>

- <span data-ttu-id="e76bc-293">**Убедитесь, что вы развернули правильное количество виртуальных машин нужного размера в соответствии с загруженностью приложения.**</span><span class="sxs-lookup"><span data-stu-id="e76bc-293">**Verify that you have deployed the right number and size of VMs for your application load.**</span></span>

    <span data-ttu-id="e76bc-294">Определите, работает ли какая-либо из виртуальных машин в виртуальной сети Azure медленно.</span><span class="sxs-lookup"><span data-stu-id="e76bc-294">Determine if any of the virtual machines in the Azure VNet are running slowly.</span></span> <span data-ttu-id="e76bc-295">Если это так, то они могут быть перегружены, возможно, их слишком мало для такой загрузки или подсистемы балансировки нагрузки настроены неправильно.</span><span class="sxs-lookup"><span data-stu-id="e76bc-295">If so, they may be overloaded, there may be too few to handle the load, or the load-balancers may not be configured correctly.</span></span> <span data-ttu-id="e76bc-296">Чтобы определить это, [соберите и проанализируйте диагностические сведения][azure-vm-diagnostics].</span><span class="sxs-lookup"><span data-stu-id="e76bc-296">To determine this, [capture and analyze diagnostic information][azure-vm-diagnostics].</span></span> <span data-ttu-id="e76bc-297">Вы можете проверить результаты, используя портал Azure. Кроме того, вы можете воспользоваться различными сторонними средствами, чтобы получить подробное представление о производительности.</span><span class="sxs-lookup"><span data-stu-id="e76bc-297">You can examine the results using the Azure portal, but many third-party tools are also available that can provide detailed insights into the performance data.</span></span>

- <span data-ttu-id="e76bc-298">**Убедитесь, что приложение эффективно использует облачные ресурсы.**</span><span class="sxs-lookup"><span data-stu-id="e76bc-298">**Verify that the application is making efficient use of cloud resources.**</span></span>

    <span data-ttu-id="e76bc-299">Выполните инструментирование кода приложений, выполняемых на каждой виртуальной машине, чтобы определить, используют ли приложения ресурсы максимально эффективно.</span><span class="sxs-lookup"><span data-stu-id="e76bc-299">Instrument application code running on each VM to determine whether applications are making the best use of resources.</span></span> <span data-ttu-id="e76bc-300">Используйте такое средство, как [Application Insights][application-insights].</span><span class="sxs-lookup"><span data-stu-id="e76bc-300">You can use tools such as [Application Insights][application-insights].</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="e76bc-301">Развертывание решения</span><span class="sxs-lookup"><span data-stu-id="e76bc-301">Deploy the solution</span></span>


<span data-ttu-id="e76bc-302">**Предварительные требования.**</span><span class="sxs-lookup"><span data-stu-id="e76bc-302">**Prequisites.**</span></span> <span data-ttu-id="e76bc-303">Вам необходима настроенная локальная инфраструктура с подходящим сетевым устройством.</span><span class="sxs-lookup"><span data-stu-id="e76bc-303">You must have an existing on-premises infrastructure already configured with a suitable network appliance.</span></span>

<span data-ttu-id="e76bc-304">Чтобы развернуть решение, сделайте следующее.</span><span class="sxs-lookup"><span data-stu-id="e76bc-304">To deploy the solution, perform the following steps.</span></span>

1. <span data-ttu-id="e76bc-305">Нажмите кнопку ниже:</span><span class="sxs-lookup"><span data-stu-id="e76bc-305">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fvpn%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. <span data-ttu-id="e76bc-306">Подождите, пока на портале Azure не откроется ссылка, а затем сделайте следующее:</span><span class="sxs-lookup"><span data-stu-id="e76bc-306">Wait for the link to open in the Azure portal, then follow these steps:</span></span> 
   * <span data-ttu-id="e76bc-307">Имя **группы ресурсов** уже определено в файле параметров, поэтому выберите **Создать** и введите `ra-hybrid-vpn-rg` в текстовом поле.</span><span class="sxs-lookup"><span data-stu-id="e76bc-307">The **Resource group** name is already defined in the parameter file, so select **Create New** and enter `ra-hybrid-vpn-rg` in the text box.</span></span>
   * <span data-ttu-id="e76bc-308">В раскрывающемся списке **Расположение** выберите регион.</span><span class="sxs-lookup"><span data-stu-id="e76bc-308">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="e76bc-309">Не изменяйте поля **Template Root Uri** (Корневой URI шаблона) или **Parameter Root Uri** (Корневой URI параметра).</span><span class="sxs-lookup"><span data-stu-id="e76bc-309">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="e76bc-310">Прочтите условия использования и установите флажок **Я принимаю указанные выше условия**.</span><span class="sxs-lookup"><span data-stu-id="e76bc-310">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="e76bc-311">Нажмите кнопку **Приобрести**.</span><span class="sxs-lookup"><span data-stu-id="e76bc-311">Click the **Purchase** button.</span></span>
3. <span data-ttu-id="e76bc-312">Дождитесь завершения развертывания.</span><span class="sxs-lookup"><span data-stu-id="e76bc-312">Wait for the deployment to complete.</span></span>



<!-- links -->

[adds-extend-domain]: ../identity/adds-extend-domain.md
[expressroute]: ../hybrid-networking/expressroute.md
[windows-vm-ra]: ../virtual-machines-windows/index.md
[linux-vm-ra]: ../virtual-machines-linux/index.md

[naming conventions]: /azure/guidance/guidance-naming-conventions

[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[arm-templates]: /azure/resource-group-authoring-templates
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-portal]: /azure/azure-portal/resource-group-portal
[azure-powershell]: /azure/powershell-azure-resource-manager
[azure-virtual-network]: /azure/virtual-network/virtual-networks-overview
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[azure-vpn-gateway]: https://azure.microsoft.com/services/vpn-gateway/
[azure-gateway-charges]: https://azure.microsoft.com/pricing/details/vpn-gateway/
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[vpn-gateway-multi-site]: /azure/vpn-gateway/vpn-gateway-multi-site
[policy-based-routing]: https://en.wikipedia.org/wiki/Policy-based_routing
[route-based-routing]: https://en.wikipedia.org/wiki/Static_routing
[network-security-group]: /azure/virtual-network/virtual-networks-nsg
[sla-for-vpn-gateway]: https://azure.microsoft.com/support/legal/sla/vpn-gateway/v1_2/
[additional-firewall-rules]: https://technet.microsoft.com/library/dn786406.aspx#firewall
[nagios]: https://www.nagios.org/
[azure-vpn-gateway-diagnostics]: http://blogs.technet.com/b/keithmayer/archive/2014/12/18/diagnose-azure-virtual-network-vpn-connectivity-issues-with-powershell.aspx
[ping]: https://technet.microsoft.com/library/ff961503.aspx
[tracert]: https://technet.microsoft.com/library/ff961507.aspx
[psping]: http://technet.microsoft.com/sysinternals/jj729731.aspx
[nmap]: http://nmap.org
[changing-SKUs]: https://azure.microsoft.com/blog/azure-virtual-network-gateway-improvements/
[gateway-diagnostic-logs]: http://blogs.technet.com/b/keithmayer/archive/2015/12/07/step-by-step-capturing-azure-resource-manager-arm-vnet-gateway-diagnostic-logs.aspx
[troubleshooting-vpn-errors]: https://blogs.technet.microsoft.com/rrasblog/2009/08/12/troubleshooting-common-vpn-related-errors/
[rras-logging]: https://www.petri.com/enable-diagnostic-logging-in-windows-server-2012-r2-routing-and-remote-access
[create-on-prem-network]: https://technet.microsoft.com/library/dn786406.aspx#routing
[azure-vm-diagnostics]: https://azure.microsoft.com/blog/windows-azure-virtual-machine-monitoring-with-wad-extension/
[application-insights]: /azure/application-insights/app-insights-overview-usage
[forced-tunneling]: https://azure.microsoft.com/documentation/articles/vpn-gateway-about-forced-tunneling/
[vpn-appliances]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-architectures.vsdx
[vpn-appliance-ipsec]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices#ipsec-parameters
<!--[solution-script]: https://github.com/mspnp/reference-architectures/tree/master/guidance-hybrid-network-vpn/Deploy-ReferenceArchitecture.ps1-->
<!--[solution-script-bash]: https://github.com/mspnp/reference-architectures/tree/master/guidance-hybrid-network-vpn/deploy-reference-architecture.sh-->
<!--[virtualNetworkGateway-parameters]: https://github.com/mspnp/reference-architectures/tree/master/guidance-hybrid-network-vpn/parameters/virtualNetworkGateway.parameters.json-->
[azure-cli]: https://azure.microsoft.com/documentation/articles/xplat-cli-install/
[CIDR]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[0]: ./images/vpn.png "Гибридная сеть, объединяющая локальную среду и инфраструктуру Azure"
[2]: ../_images/guidance-hybrid-network-vpn/audit-logs.png "Журналы аудита на портале Azure"
[3]: ../_images/guidance-hybrid-network-vpn/RRAS-perf-counters.png "Счетчики производительности для мониторинга трафика VPN"
[4]: ../_images/guidance-hybrid-network-vpn/RRAS-perf-graph.png "Пример графика производительности сети VPN"