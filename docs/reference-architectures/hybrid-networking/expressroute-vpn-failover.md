---
title: Подключение локальной сети к Azure
titleSuffix: Azure Reference Architectures
description: Реализуйте архитектуру высокодоступной защищенной сети типа "сеть — сеть", которая охватывает виртуальную сеть Azure и локальную сеть, подключенные с помощью ExpressRoute с отработкой отказа VPN-шлюза.
author: telmosampaio
ms.date: 10/22/2017
ms.custom: seodec18
ms.openlocfilehash: d44c046f2351d6103a01108574e0295302f0ba11
ms.sourcegitcommit: 88a68c7e9b6b772172b7faa4b9fd9c061a9f7e9d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/08/2018
ms.locfileid: "53119954"
---
# <a name="connect-an-on-premises-network-to-azure-using-expressroute-with-vpn-failover"></a>Подключение локальной сети к Azure с помощью ExpressRoute с отработкой отказа VPN

В этой эталонной архитектуре показан способ подключения локальной сети к виртуальной сети Azure с помощью ExpressRoute, в котором частная сеть с подключением "сеть — сеть" (VPN) выступает в качестве подключения для отработки отказа. Трафик проходит между локальной сетью и виртуальной сетью Azure через подключение ExpressRoute. Если возникает потеря подключения к каналу ExpressRoute, трафик направляется через туннель VPN IPSec. [**Разверните это решение**](#deploy-the-solution).

Обратите внимание, что в случае недоступности канала ExpressRoute VPN-маршрут обрабатывает только частные пиринговые подключения. Открытые пиринговые подключения и пиринговые подключения Майкрософт проходят через Интернет.

![Эталонная архитектура высокодоступной гибридной сети с использованием ExpressRoute и VPN-шлюза](./images/expressroute-vpn-failover.png)

*Скачайте [файл Visio][visio-download] этой архитектуры.*

## <a name="architecture"></a>Архитектура

Архитектура состоит из следующих компонентов:

- **Локальная сеть.** Частная локальная сеть, работающая внутри организации.

- **VPN-устройство.** Устройство или служба, предоставляющая возможность внешнего подключения к локальной сети. VPN-устройство может быть аппаратным устройством или программным решением, таким как служба маршрутизации и удаленного доступа (RRAS) в Windows Server 2012. Список поддерживаемых VPN-устройств и информацию о настройке выбранных VPN-устройств для подключения к Azure см. в статье [VPN-устройства и параметры IPsec/IKE для подключений типа "сеть — сеть" через VPN-шлюз][vpn-appliance].

- **Канал ExpressRoute.** Это канал уровня 2 или 3, который предоставляет поставщик подключения. Он позволяет подключиться к локальной сети с Azure через пограничные маршрутизаторы. Канал использует инфраструктуру оборудования, настроенную поставщиком подключения.

- **Шлюз виртуальной сети ExpressRoute.** Шлюз виртуальной сети ExpressRoute позволяет виртуальной сети подключаться к каналу ExpressRoute, который используется для подключения к вашей локальной сети.

- **Шлюз виртуальной сети VPN.** Шлюз виртуальной сети VPN позволяет виртуальной сети подключаться к VPN-устройству в локальной сети. Шлюз виртуальной сети VPN настроен на прием запросов от локальной сети только с помощью VPN-устройства. Дополнительные сведения см. в статье [Подключение локальной сети к виртуальной сети Microsoft Azure][connect-to-an-Azure-vnet].

- **VPN-подключение.** У соединения есть свойства, указывающие тип соединения (IPSec) и общий ключ с VPN-устройством в локальной среде для шифрования трафика.

- **Виртуальная сеть Azure.** Каждая виртуальная сеть находится в одном регионе Azure и может содержать несколько уровней приложения. Уровни приложения могут быть разделены на сегменты с помощью подсетей в каждой виртуальной сети.

- **Подсеть шлюза.** Шлюзы виртуальных сетей хранятся в одной подсети.

- **Облачное приложение.** Приложение, размещенное в Azure. Оно может содержать несколько уровней с несколькими подсетями, подключенными через подсистемы балансировки нагрузки Azure. Дополнительные сведения об инфраструктуре приложений см. в статьях [Запуск рабочих нагрузок на виртуальных машинах Windows][windows-vm-ra] и [Запуск рабочих нагрузок на виртуальной машине Linux][linux-vm-ra].

## <a name="recommendations"></a>Рекомендации

Следующие рекомендации применимы для большинства ситуаций. Следуйте этим рекомендациям, если они не противоречат особым требованиям для вашего случая.

### <a name="vnet-and-gatewaysubnet"></a>Виртуальная сеть и подсеть шлюза

Создайте шлюз виртуальной сети ExpressRoute и VPN-шлюз виртуальной сети в одной виртуальной сети. Это означает, что шлюзы будут использовать одну подсеть под названием *GatewaySubnet*.

Если виртуальная сеть уже содержит подсеть *GatewaySubnet*, убедитесь, что у нее есть адресное пространство с блоком адресов /27 или больше. Если имеющаяся подсеть слишком мала, удалите подсеть с помощью следующей команды PowerShell:

```powershell
$vnet = Get-AzureRmVirtualNetworkGateway -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
Remove-AzureRmVirtualNetworkSubnetConfig -Name GatewaySubnet -VirtualNetwork $vnet
```

Если в виртуальной сети нет подсети **GatewaySubnet**, создайте ее, выполнив следующую команду Powershell:

```powershell
$vnet = Get-AzureRmVirtualNetworkGateway -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
Add-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet -AddressPrefix "10.200.255.224/27"
$vnet = Set-AzureRmVirtualNetwork -VirtualNetwork $vnet
```

### <a name="vpn-and-expressroute-gateways"></a>Шлюз ExpressRoute и VPN-шлюз

Чтобы подключиться к Azure, убедитесь, что ваша организация соответствует [обязательным требованиям ExpressRoute][expressroute-prereq].

Если у вас уже есть VPN-шлюз виртуальной сети в виртуальной сети Azure, удалите его с помощью следующей команды Powershell:

```powershell
Remove-AzureRmVirtualNetworkGateway -Name <yourgatewayname> -ResourceGroupName <yourresourcegroup>
```

Чтобы установить подключение ExpressRoute, следуйте инструкциям в статье [Подключение локальной сети к Azure с помощью ExpressRoute][implementing-expressroute].

Чтобы установить подключение VPN-шлюза, следуйте инструкциям в статье [Connect an on-premises network to Azure using a VPN gateway][implementing-vpn] (Подключение локальной сети к Azure с помощью VPN-шлюза).

Подключившись к шлюзу виртуальной сети, протестируйте среду следующим образом:

1. Убедитесь, что вы можете подключиться к сети Azure из локальной сети.
2. Обратитесь к поставщику, чтобы удалить подключение ExpressRoute для тестирования.
3. Убедитесь, что по-прежнему можете подключиться из локальной сети к виртуальной сети Azure с помощью VPN-шлюза виртуальной сети.
4. Обратитесь к поставщику, чтобы повторно установить подключение ExpressRoute.

## <a name="considerations"></a>Рекомендации

Рекомендации по подключению ExpressRoute см. в руководстве [Connect an on-premises network to Azure using ExpressRoute][guidance-expressroute] (Подключение локальной сети к Azure с помощью ExpressRoute).

Рекомендации по подключению VPN типа "сеть — сеть" см. в руководстве [Connect an on-premises network to Azure using a VPN gateway][guidance-vpn] (Подключение локальной сети к Azure с помощью VPN-шлюза).

Рекомендации по обеспечению безопасности Azure см. в статье [Облачные службы Microsoft Cloud и сетевая безопасность][best-practices-security].

## <a name="deploy-the-solution"></a>Развертывание решения

**Предварительные требования**. Вам необходима настроенная локальная инфраструктура с подходящим сетевым устройством.

Чтобы развернуть решение, сделайте следующее.

<!-- markdownlint-disable MD033 -->

1. Нажмите кнопку ниже:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute-vpn-failover%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>

2. Подождите, пока на портале Azure не откроется ссылка, а затем сделайте следующее:
   - Имя **группы ресурсов** уже определено в файле параметров, поэтому выберите **Создать** и введите `ra-hybrid-vpn-er-rg` в текстовом поле.
   - В раскрывающемся списке **Расположение** выберите регион.
   - Не изменяйте поля **Template Root Uri** (Корневой URI шаблона) или **Parameter Root Uri** (Корневой URI параметра).
   - Прочтите условия использования и установите флажок **Я принимаю указанные выше условия**.
   - Нажмите кнопку **Приобрести**.

3. Дождитесь завершения развертывания.

4. Нажмите кнопку ниже:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute-vpn-failover%2Fazuredeploy-expressRouteCircuit.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>

5. Подождите, пока на портале Azure не откроется ссылка, а затем сделайте следующее:
   - В разделе **Группа ресурсов** выберите **Использовать имеющийся**, а затем в текстовом поле введите `ra-hybrid-vpn-er-rg`.
   - В раскрывающемся списке **Расположение** выберите регион.
   - Не изменяйте поля **Template Root Uri** (Корневой URI шаблона) или **Parameter Root Uri** (Корневой URI параметра).
   - Прочтите условия использования и установите флажок **Я принимаю указанные выше условия**.
   - Нажмите кнопку **Приобрести**.

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
