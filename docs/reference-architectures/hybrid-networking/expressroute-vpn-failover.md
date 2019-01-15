---
title: Подключение локальной сети к Azure
titleSuffix: Azure Reference Architectures
description: Реализуйте архитектуру высокодоступной защищенной сети типа "сеть — сеть", которая охватывает виртуальную сеть Azure и локальную сеть, подключенные с помощью ExpressRoute с отработкой отказа VPN-шлюза.
author: telmosampaio
ms.date: 10/22/2017
ms.custom: seodec18
ms.openlocfilehash: d32e4dfa81cf74a4ca74746120c15f1ddc066c3e
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/04/2019
ms.locfileid: "54011112"
---
# <a name="connect-an-on-premises-network-to-azure-using-expressroute-with-vpn-failover"></a><span data-ttu-id="8ded2-103">Подключение локальной сети к Azure с помощью ExpressRoute с отработкой отказа VPN</span><span class="sxs-lookup"><span data-stu-id="8ded2-103">Connect an on-premises network to Azure using ExpressRoute with VPN failover</span></span>

<span data-ttu-id="8ded2-104">В этой эталонной архитектуре показан способ подключения локальной сети к виртуальной сети Azure с помощью ExpressRoute, в котором частная сеть с подключением "сеть — сеть" (VPN) выступает в качестве подключения для отработки отказа.</span><span class="sxs-lookup"><span data-stu-id="8ded2-104">This reference architecture shows how to connect an on-premises network to an Azure virtual network (VNet) using ExpressRoute, with a site-to-site virtual private network (VPN) as a failover connection.</span></span> <span data-ttu-id="8ded2-105">Трафик проходит между локальной сетью и виртуальной сетью Azure через подключение ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="8ded2-105">Traffic flows between the on-premises network and the Azure VNet through an ExpressRoute connection.</span></span> <span data-ttu-id="8ded2-106">Если возникает потеря подключения к каналу ExpressRoute, трафик направляется через туннель VPN IPSec.</span><span class="sxs-lookup"><span data-stu-id="8ded2-106">If there is a loss of connectivity in the ExpressRoute circuit, traffic is routed through an IPSec VPN tunnel.</span></span> <span data-ttu-id="8ded2-107">[**Разверните это решение**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="8ded2-107">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="8ded2-108">Обратите внимание, что в случае недоступности канала ExpressRoute VPN-маршрут обрабатывает только частные пиринговые подключения.</span><span class="sxs-lookup"><span data-stu-id="8ded2-108">Note that if the ExpressRoute circuit is unavailable, the VPN route will only handle private peering connections.</span></span> <span data-ttu-id="8ded2-109">Открытые пиринговые подключения и пиринговые подключения Майкрософт проходят через Интернет.</span><span class="sxs-lookup"><span data-stu-id="8ded2-109">Public peering and Microsoft peering connections will pass over the Internet.</span></span>

![Эталонная архитектура высокодоступной гибридной сети с использованием ExpressRoute и VPN-шлюза](./images/expressroute-vpn-failover.png)

<span data-ttu-id="8ded2-111">*Скачайте [файл Visio][visio-download] этой архитектуры.*</span><span class="sxs-lookup"><span data-stu-id="8ded2-111">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="8ded2-112">Архитектура</span><span class="sxs-lookup"><span data-stu-id="8ded2-112">Architecture</span></span>

<span data-ttu-id="8ded2-113">Архитектура состоит из следующих компонентов:</span><span class="sxs-lookup"><span data-stu-id="8ded2-113">The architecture consists of the following components.</span></span>

- <span data-ttu-id="8ded2-114">**Локальная сеть.**</span><span class="sxs-lookup"><span data-stu-id="8ded2-114">**On-premises network**.</span></span> <span data-ttu-id="8ded2-115">Частная локальная сеть, работающая внутри организации.</span><span class="sxs-lookup"><span data-stu-id="8ded2-115">A private local-area network running within an organization.</span></span>

- <span data-ttu-id="8ded2-116">**VPN-устройство.**</span><span class="sxs-lookup"><span data-stu-id="8ded2-116">**VPN appliance**.</span></span> <span data-ttu-id="8ded2-117">Устройство или служба, предоставляющая возможность внешнего подключения к локальной сети.</span><span class="sxs-lookup"><span data-stu-id="8ded2-117">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="8ded2-118">VPN-устройство может быть аппаратным устройством или программным решением, таким как служба маршрутизации и удаленного доступа (RRAS) в Windows Server 2012.</span><span class="sxs-lookup"><span data-stu-id="8ded2-118">The VPN appliance may be a hardware device, or it can be a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="8ded2-119">Список поддерживаемых VPN-устройств и информацию о настройке выбранных VPN-устройств для подключения к Azure см. в статье [VPN-устройства и параметры IPsec/IKE для подключений типа "сеть — сеть" через VPN-шлюз][vpn-appliance].</span><span class="sxs-lookup"><span data-stu-id="8ded2-119">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

- <span data-ttu-id="8ded2-120">**Канал ExpressRoute.**</span><span class="sxs-lookup"><span data-stu-id="8ded2-120">**ExpressRoute circuit**.</span></span> <span data-ttu-id="8ded2-121">Это канал уровня 2 или 3, который предоставляет поставщик подключения. Он позволяет подключиться к локальной сети с Azure через пограничные маршрутизаторы.</span><span class="sxs-lookup"><span data-stu-id="8ded2-121">A layer 2 or layer 3 circuit supplied by the connectivity provider that joins the on-premises network with Azure through the edge routers.</span></span> <span data-ttu-id="8ded2-122">Канал использует инфраструктуру оборудования, настроенную поставщиком подключения.</span><span class="sxs-lookup"><span data-stu-id="8ded2-122">The circuit uses the hardware infrastructure managed by the connectivity provider.</span></span>

- <span data-ttu-id="8ded2-123">**Шлюз виртуальной сети ExpressRoute.**</span><span class="sxs-lookup"><span data-stu-id="8ded2-123">**ExpressRoute virtual network gateway**.</span></span> <span data-ttu-id="8ded2-124">Шлюз виртуальной сети ExpressRoute позволяет виртуальной сети подключаться к каналу ExpressRoute, который используется для подключения к вашей локальной сети.</span><span class="sxs-lookup"><span data-stu-id="8ded2-124">The ExpressRoute virtual network gateway enables the VNet to connect to the ExpressRoute circuit used for connectivity with your on-premises network.</span></span>

- <span data-ttu-id="8ded2-125">**Шлюз виртуальной сети VPN.**</span><span class="sxs-lookup"><span data-stu-id="8ded2-125">**VPN virtual network gateway**.</span></span> <span data-ttu-id="8ded2-126">Шлюз виртуальной сети VPN позволяет виртуальной сети подключаться к VPN-устройству в локальной сети.</span><span class="sxs-lookup"><span data-stu-id="8ded2-126">The VPN virtual network gateway enables the VNet to connect to the VPN appliance in the on-premises network.</span></span> <span data-ttu-id="8ded2-127">Шлюз виртуальной сети VPN настроен на прием запросов от локальной сети только с помощью VPN-устройства.</span><span class="sxs-lookup"><span data-stu-id="8ded2-127">The VPN virtual network gateway is configured to accept requests from the on-premises network only through the VPN appliance.</span></span> <span data-ttu-id="8ded2-128">Дополнительные сведения см. в статье [Подключение локальной сети к виртуальной сети Microsoft Azure][connect-to-an-Azure-vnet].</span><span class="sxs-lookup"><span data-stu-id="8ded2-128">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

- <span data-ttu-id="8ded2-129">**VPN-подключение.**</span><span class="sxs-lookup"><span data-stu-id="8ded2-129">**VPN connection**.</span></span> <span data-ttu-id="8ded2-130">У соединения есть свойства, указывающие тип соединения (IPSec) и общий ключ с VPN-устройством в локальной среде для шифрования трафика.</span><span class="sxs-lookup"><span data-stu-id="8ded2-130">The connection has properties that specify the connection type (IPSec) and the key shared with the on-premises VPN appliance to encrypt traffic.</span></span>

- <span data-ttu-id="8ded2-131">**Виртуальная сеть Azure.**</span><span class="sxs-lookup"><span data-stu-id="8ded2-131">**Azure Virtual Network (VNet)**.</span></span> <span data-ttu-id="8ded2-132">Каждая виртуальная сеть находится в одном регионе Azure и может содержать несколько уровней приложения.</span><span class="sxs-lookup"><span data-stu-id="8ded2-132">Each VNet resides in a single Azure region, and can host multiple application tiers.</span></span> <span data-ttu-id="8ded2-133">Уровни приложения могут быть разделены на сегменты с помощью подсетей в каждой виртуальной сети.</span><span class="sxs-lookup"><span data-stu-id="8ded2-133">Application tiers can be segmented using subnets in each VNet.</span></span>

- <span data-ttu-id="8ded2-134">**Подсеть шлюза.**</span><span class="sxs-lookup"><span data-stu-id="8ded2-134">**Gateway subnet**.</span></span> <span data-ttu-id="8ded2-135">Шлюзы виртуальных сетей хранятся в одной подсети.</span><span class="sxs-lookup"><span data-stu-id="8ded2-135">The virtual network gateways are held in the same subnet.</span></span>

- <span data-ttu-id="8ded2-136">**Облачное приложение.**</span><span class="sxs-lookup"><span data-stu-id="8ded2-136">**Cloud application**.</span></span> <span data-ttu-id="8ded2-137">Приложение, размещенное в Azure.</span><span class="sxs-lookup"><span data-stu-id="8ded2-137">The application hosted in Azure.</span></span> <span data-ttu-id="8ded2-138">Оно может содержать несколько уровней с несколькими подсетями, подключенными через подсистемы балансировки нагрузки Azure.</span><span class="sxs-lookup"><span data-stu-id="8ded2-138">It might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="8ded2-139">Дополнительные сведения об инфраструктуре приложений см. в статьях [Запуск рабочих нагрузок на виртуальных машинах Windows][windows-vm-ra] и [Запуск рабочих нагрузок на виртуальной машине Linux][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="8ded2-139">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

## <a name="recommendations"></a><span data-ttu-id="8ded2-140">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="8ded2-140">Recommendations</span></span>

<span data-ttu-id="8ded2-141">Следующие рекомендации применимы для большинства ситуаций.</span><span class="sxs-lookup"><span data-stu-id="8ded2-141">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="8ded2-142">Следуйте этим рекомендациям, если они не противоречат особым требованиям для вашего случая.</span><span class="sxs-lookup"><span data-stu-id="8ded2-142">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="vnet-and-gatewaysubnet"></a><span data-ttu-id="8ded2-143">Виртуальная сеть и подсеть шлюза</span><span class="sxs-lookup"><span data-stu-id="8ded2-143">VNet and GatewaySubnet</span></span>

<span data-ttu-id="8ded2-144">Создайте шлюз виртуальной сети ExpressRoute и VPN-шлюз виртуальной сети в одной виртуальной сети.</span><span class="sxs-lookup"><span data-stu-id="8ded2-144">Create the ExpressRoute virtual network gateway and the VPN virtual network gateway in the same VNet.</span></span> <span data-ttu-id="8ded2-145">Это означает, что шлюзы будут использовать одну подсеть под названием *GatewaySubnet*.</span><span class="sxs-lookup"><span data-stu-id="8ded2-145">This means that they should share the same subnet named *GatewaySubnet*.</span></span>

<span data-ttu-id="8ded2-146">Если виртуальная сеть уже содержит подсеть *GatewaySubnet*, убедитесь, что у нее есть адресное пространство с блоком адресов /27 или больше.</span><span class="sxs-lookup"><span data-stu-id="8ded2-146">If the VNet already includes a subnet named *GatewaySubnet*, ensure that it has a /27 or larger address space.</span></span> <span data-ttu-id="8ded2-147">Если имеющаяся подсеть слишком мала, удалите подсеть с помощью следующей команды PowerShell:</span><span class="sxs-lookup"><span data-stu-id="8ded2-147">If the existing subnet is too small, use the following PowerShell command to remove the subnet:</span></span>

```powershell
$vnet = Get-AzureRmVirtualNetworkGateway -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
Remove-AzureRmVirtualNetworkSubnetConfig -Name GatewaySubnet -VirtualNetwork $vnet
```

<span data-ttu-id="8ded2-148">Если в виртуальной сети нет подсети **GatewaySubnet**, создайте ее, выполнив следующую команду Powershell:</span><span class="sxs-lookup"><span data-stu-id="8ded2-148">If the VNet does not contain a subnet named **GatewaySubnet**, create a new one using the following Powershell command:</span></span>

```powershell
$vnet = Get-AzureRmVirtualNetworkGateway -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
Add-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet -AddressPrefix "10.200.255.224/27"
$vnet = Set-AzureRmVirtualNetwork -VirtualNetwork $vnet
```

### <a name="vpn-and-expressroute-gateways"></a><span data-ttu-id="8ded2-149">Шлюз ExpressRoute и VPN-шлюз</span><span class="sxs-lookup"><span data-stu-id="8ded2-149">VPN and ExpressRoute gateways</span></span>

<span data-ttu-id="8ded2-150">Чтобы подключиться к Azure, убедитесь, что ваша организация соответствует [обязательным требованиям ExpressRoute][expressroute-prereq].</span><span class="sxs-lookup"><span data-stu-id="8ded2-150">Verify that your organization meets the [ExpressRoute prerequisite requirements][expressroute-prereq] for connecting to Azure.</span></span>

<span data-ttu-id="8ded2-151">Если у вас уже есть VPN-шлюз виртуальной сети в виртуальной сети Azure, удалите его с помощью следующей команды Powershell:</span><span class="sxs-lookup"><span data-stu-id="8ded2-151">If you already have a VPN virtual network gateway in your Azure VNet, use the following Powershell command to remove it:</span></span>

```powershell
Remove-AzureRmVirtualNetworkGateway -Name <yourgatewayname> -ResourceGroupName <yourresourcegroup>
```

<span data-ttu-id="8ded2-152">Чтобы установить подключение ExpressRoute, следуйте инструкциям в статье [Подключение локальной сети к Azure с помощью ExpressRoute][implementing-expressroute].</span><span class="sxs-lookup"><span data-stu-id="8ded2-152">Follow the instructions in [Implementing a hybrid network architecture with Azure ExpressRoute][implementing-expressroute] to establish your ExpressRoute connection.</span></span>

<span data-ttu-id="8ded2-153">Чтобы установить подключение VPN-шлюза, следуйте инструкциям в статье [Connect an on-premises network to Azure using a VPN gateway][implementing-vpn] (Подключение локальной сети к Azure с помощью VPN-шлюза).</span><span class="sxs-lookup"><span data-stu-id="8ded2-153">Follow the instructions in [Implementing a hybrid network architecture with Azure and On-premises VPN][implementing-vpn] to establish your VPN virtual network gateway connection.</span></span>

<span data-ttu-id="8ded2-154">Подключившись к шлюзу виртуальной сети, протестируйте среду следующим образом:</span><span class="sxs-lookup"><span data-stu-id="8ded2-154">After you have established the virtual network gateway connections, test the environment as follows:</span></span>

1. <span data-ttu-id="8ded2-155">Убедитесь, что вы можете подключиться к сети Azure из локальной сети.</span><span class="sxs-lookup"><span data-stu-id="8ded2-155">Make sure you can connect from your on-premises network to your Azure VNet.</span></span>
2. <span data-ttu-id="8ded2-156">Обратитесь к поставщику, чтобы удалить подключение ExpressRoute для тестирования.</span><span class="sxs-lookup"><span data-stu-id="8ded2-156">Contact your provider to stop ExpressRoute connectivity for testing.</span></span>
3. <span data-ttu-id="8ded2-157">Убедитесь, что по-прежнему можете подключиться из локальной сети к виртуальной сети Azure с помощью VPN-шлюза виртуальной сети.</span><span class="sxs-lookup"><span data-stu-id="8ded2-157">Verify that you can still connect from your on-premises network to your Azure VNet using the VPN virtual network gateway connection.</span></span>
4. <span data-ttu-id="8ded2-158">Обратитесь к поставщику, чтобы повторно установить подключение ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="8ded2-158">Contact your provider to reestablish ExpressRoute connectivity.</span></span>

## <a name="considerations"></a><span data-ttu-id="8ded2-159">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="8ded2-159">Considerations</span></span>

<span data-ttu-id="8ded2-160">Рекомендации по подключению ExpressRoute см. в руководстве [Connect an on-premises network to Azure using ExpressRoute][guidance-expressroute] (Подключение локальной сети к Azure с помощью ExpressRoute).</span><span class="sxs-lookup"><span data-stu-id="8ded2-160">For ExpressRoute considerations, see the [Implementing a Hybrid Network Architecture with Azure ExpressRoute][guidance-expressroute] guidance.</span></span>

<span data-ttu-id="8ded2-161">Рекомендации по подключению VPN типа "сеть — сеть" см. в руководстве [Connect an on-premises network to Azure using a VPN gateway][guidance-vpn] (Подключение локальной сети к Azure с помощью VPN-шлюза).</span><span class="sxs-lookup"><span data-stu-id="8ded2-161">For site-to-site VPN considerations, see the [Implementing a Hybrid Network Architecture with Azure and On-premises VPN][guidance-vpn] guidance.</span></span>

<span data-ttu-id="8ded2-162">Рекомендации по обеспечению безопасности Azure см. в статье [Облачные службы Microsoft Cloud и сетевая безопасность][best-practices-security].</span><span class="sxs-lookup"><span data-stu-id="8ded2-162">For general Azure security considerations, see [Microsoft cloud services and network security][best-practices-security].</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="8ded2-163">Развертывание решения</span><span class="sxs-lookup"><span data-stu-id="8ded2-163">Deploy the solution</span></span>

<span data-ttu-id="8ded2-164">**Предварительные требования**.</span><span class="sxs-lookup"><span data-stu-id="8ded2-164">**Prerequisites**.</span></span> <span data-ttu-id="8ded2-165">Вам необходима настроенная локальная инфраструктура с подходящим сетевым устройством.</span><span class="sxs-lookup"><span data-stu-id="8ded2-165">You must have an existing on-premises infrastructure already configured with a suitable network appliance.</span></span>

<span data-ttu-id="8ded2-166">Чтобы развернуть решение, сделайте следующее.</span><span class="sxs-lookup"><span data-stu-id="8ded2-166">To deploy the solution, perform the following steps.</span></span>

<!-- markdownlint-disable MD033 -->

1. <span data-ttu-id="8ded2-167">Нажмите кнопку ниже:</span><span class="sxs-lookup"><span data-stu-id="8ded2-167">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute-vpn-failover%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>

2. <span data-ttu-id="8ded2-168">Подождите, пока на портале Azure не откроется ссылка, а затем сделайте следующее:</span><span class="sxs-lookup"><span data-stu-id="8ded2-168">Wait for the link to open in the Azure portal, then follow these steps:</span></span>
   - <span data-ttu-id="8ded2-169">Имя **группы ресурсов** уже определено в файле параметров, поэтому выберите **Создать** и введите `ra-hybrid-vpn-er-rg` в текстовом поле.</span><span class="sxs-lookup"><span data-stu-id="8ded2-169">The **Resource group** name is already defined in the parameter file, so select **Create New** and enter `ra-hybrid-vpn-er-rg` in the text box.</span></span>
   - <span data-ttu-id="8ded2-170">В раскрывающемся списке **Расположение** выберите регион.</span><span class="sxs-lookup"><span data-stu-id="8ded2-170">Select the region from the **Location** drop down box.</span></span>
   - <span data-ttu-id="8ded2-171">Не изменяйте поля **Template Root Uri** (Корневой URI шаблона) или **Parameter Root Uri** (Корневой URI параметра).</span><span class="sxs-lookup"><span data-stu-id="8ded2-171">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   - <span data-ttu-id="8ded2-172">Прочтите условия использования и установите флажок **Я принимаю указанные выше условия**.</span><span class="sxs-lookup"><span data-stu-id="8ded2-172">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   - <span data-ttu-id="8ded2-173">Нажмите кнопку **Приобрести**.</span><span class="sxs-lookup"><span data-stu-id="8ded2-173">Click the **Purchase** button.</span></span>

3. <span data-ttu-id="8ded2-174">Дождитесь завершения развертывания.</span><span class="sxs-lookup"><span data-stu-id="8ded2-174">Wait for the deployment to complete.</span></span>

4. <span data-ttu-id="8ded2-175">Нажмите кнопку ниже:</span><span class="sxs-lookup"><span data-stu-id="8ded2-175">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute-vpn-failover%2Fazuredeploy-expressRouteCircuit.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>

5. <span data-ttu-id="8ded2-176">Подождите, пока на портале Azure не откроется ссылка, а затем сделайте следующее:</span><span class="sxs-lookup"><span data-stu-id="8ded2-176">Wait for the link to open in the Azure portal, then enter then follow these steps:</span></span>
   - <span data-ttu-id="8ded2-177">В разделе **Группа ресурсов** выберите **Использовать имеющийся**, а затем в текстовом поле введите `ra-hybrid-vpn-er-rg`.</span><span class="sxs-lookup"><span data-stu-id="8ded2-177">Select **Use existing** in the **Resource group** section and enter `ra-hybrid-vpn-er-rg` in the text box.</span></span>
   - <span data-ttu-id="8ded2-178">В раскрывающемся списке **Расположение** выберите регион.</span><span class="sxs-lookup"><span data-stu-id="8ded2-178">Select the region from the **Location** drop down box.</span></span>
   - <span data-ttu-id="8ded2-179">Не изменяйте поля **Template Root Uri** (Корневой URI шаблона) или **Parameter Root Uri** (Корневой URI параметра).</span><span class="sxs-lookup"><span data-stu-id="8ded2-179">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   - <span data-ttu-id="8ded2-180">Прочтите условия использования и установите флажок **Я принимаю указанные выше условия**.</span><span class="sxs-lookup"><span data-stu-id="8ded2-180">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   - <span data-ttu-id="8ded2-181">Нажмите кнопку **Приобрести**.</span><span class="sxs-lookup"><span data-stu-id="8ded2-181">Click the **Purchase** button.</span></span>

<!-- markdownlint-enable MD033 -->

<!-- links -->

[windows-vm-ra]: ../virtual-machines-windows/index.md
[linux-vm-ra]: ../virtual-machines-linux/index.md
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[expressroute-prereq]: /azure/expressroute/expressroute-prerequisites
[implementing-expressroute]: ./expressroute.md
[implementing-vpn]: ./vpn.md
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[best-practices-security]: /azure/best-practices-network-security
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-architectures.vsdx
