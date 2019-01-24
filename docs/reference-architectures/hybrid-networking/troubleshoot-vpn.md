---
title: Устранение неполадок при гибридном VPN-подключении
titleSuffix: Azure Reference Architectures
description: Устранение неполадок при подключении между локальной сетью и Azure с использованием VPN-шлюза.
author: telmosampaio
ms.date: 10/08/2018
ms.topic: article
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: networking
ms.openlocfilehash: 49dfcf444665ef526e76cac0f4ad13104a142840
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54485935"
---
# <a name="troubleshoot-a-hybrid-vpn-connection"></a>Устранение неполадок при гибридном VPN-подключении

В этой статье приводится ряд советов по устранению неполадок при подключение между локальной сетью и Azure с использованием VPN-шлюза. Общие сведения об устранении неполадок, связанных с VPN, см. [здесь][troubleshooting-vpn-errors].

## <a name="verify-the-vpn-appliance-is-functioning-correctly"></a>Проверка работы VPN-устройства

Следующие рекомендации помогут определить, правильно ли работает ваше локальное VPN-устройство.

**Проверьте все файлы журнала, созданные VPN-устройством, на наличие ошибок или сбоев.** Так вы сможете определить, правильно ли функционирует VPN-устройство. Расположение этих сведений будет различаться в зависимости от устройства. Например, при использовании RRAS на базе Windows Server 2012 для отображения сведений об ошибке события для службы RRAS можно использовать следующую команду PowerShell:

```PowerShell
Get-EventLog -LogName System -EntryType Error -Source RemoteAccess | Format-List -Property *
```

В свойстве *Message* каждой записи содержится описание ошибки. Несколько распространенных примеров:

- Отсутствие подключения, возможной причиной которого является неправильный IP-адрес для шлюза Azure VPN в конфигурации сетевого интерфейса VPN RRAS.

  ```console
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

- Неправильный общий ключ, заданный в конфигурации сетевого интерфейса VPN RRAS.

  ```console
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

Вы также можете получить сведения из журнала событий о попытках установить подключение через службу RRAS с помощью следующей команды PowerShell:

```powershell
Get-EventLog -LogName Application -Source RasClient | Format-List -Property *
```

В случае сбоя подключения этот журнал будет содержать ошибки, которые выглядят следующим образом:

```console
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

## <a name="verify-connectivity"></a>Проверка подключения

**Проверьте подключение и маршрутизацию через VPN-шлюз.** VPN-устройство может неправильно осуществлять маршрутизацию трафика через VPN-шлюз Azure. Используйте [PsPing][psping] для проверки подключения и маршрутизации через VPN-шлюз. Например, чтобы проверить соединение с локального компьютера на веб-сервер, расположенный в виртуальной сети, выполните следующую команду (заменив `<<web-server-address>>` адресом веб-сервера):

```console
PsPing -t <<web-server-address>>:80
```

Если локальный компьютер может осуществлять маршрутизацию трафика на веб-сервер, отобразится результат, аналогичный приведенному ниже:

```console
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

Если локальный компьютер не может связаться с указанным адресом назначения, появятся такие сообщения:

```console
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

**Убедитесь, что локальный брандмауэр разрешает передачу VPN-трафика и что открыты соответствующие порты.**

**Убедитесь, что локальное VPN-устройство использует метод шифрования, совместимый с VPN-шлюзом Azure.** Для маршрутизации на основе политик VPN-шлюз Azure поддерживает алгоритмы шифрования AES256, AES128 и 3DES. Шлюзы на основе маршрута поддерживают AES256 и 3DES. См. дополнительные сведения о [VPN-устройствах и параметрах IPsec/IKE для подключений "сеть — сеть" через VPN-шлюз][vpn-appliance].

## <a name="check-for-problems-with-the-azure-vpn-gateway"></a>Проверка на наличие проблем с VPN-шлюзом Azure

Для определения проблемы с VPN-шлюзом Azure полезны следующие рекомендации:

**Изучите журналы диагностики VPN-шлюза Azure на предмет потенциальных проблем.** См. [пошаговое руководство по сбору журналов диагностики шлюза виртуальной сети Azure Resource Manager][gateway-diagnostic-logs].

**Убедитесь, что VPN-шлюз Azure и локальное VPN-устройство настроены с использованием одного и того же общего ключа проверки подлинности.** Вы можете просмотреть общий ключ, сохраненный VPN-шлюзом Azure, используя следующую команду Azure CLI:

```azurecli
azure network vpn-connection shared-key show <<resource-group>> <<vpn-connection-name>>
```

Для просмотра общего ключа, настроенного для этого устройства, используйте команду, подходящую для вашего локального VPN-устройства.

Убедитесь, что подсеть *GatewaySubnet*, в которой размещен VPN-шлюз Azure, не связана с NSG.

Сведения о подсети можно просмотреть, используя следующую команду Azure CLI:

```azurecli
azure network vnet subnet show -g <<resource-group>> -e <<vnet-name>> -n GatewaySubnet
```

Убедитесь в отсутствии поля данных с именем *Идентификатор группы безопасности сети*. В следующем примере показаны результаты для экземпляра подсети *GatewaySubnet* с назначенной NSG (*VPN-Gateway-Group*). Если для этой NSG определены какие-либо правила, шлюз может неправильно работать.

```console
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

**Убедитесь, что виртуальные машины в виртуальной сети Azure настроены для разрешения передачи трафика, поступающего из виртуальной сети.** Проверьте все правила NSG, связанные с подсетями, содержащими эти виртуальные машины. Все правила NSG можно просмотреть, используя следующую команду Azure CLI:

```azurecli
azure network nsg show -g <<resource-group>> -n <<nsg-name>>
```

**Убедитесь, что подключен VPN-шлюз Azure.** Чтобы проверить текущее состояние VPN-подключения Azure, можно использовать следующую команду Azure PowerShell. Параметр `<<connection-name>>` представляет собой имя VPN-подключения Azure, которое связывает шлюз виртуальной сети с локальным.

```powershell
Get-AzureRmVirtualNetworkGatewayConnection -Name <<connection-name>> - ResourceGroupName <<resource-group>>
```

В следующем фрагменте кода показаны результаты для подключенного (первый пример) и отключенного шлюза (второй пример):

```powershell
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

```powershell
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

## <a name="miscellaneous-issues"></a>Прочие проблемы

Следующие рекомендации помогут определить, имеются ли проблемы с конфигурацией виртуальной машины узла, использованием пропускной способности сети или производительностью приложения:

**Проверьте конфигурацию брандмауэра.** Проверьте правильность настройки брандмауэра в гостевой ОС на виртуальных машинах Azure в подсети, чтобы разрешить передачу трафика из локальных диапазонов IP-адресов.

**Убедитесь, что объем трафика не приближается к лимиту пропускной способности VPN-шлюза Azure.** Это можно проверить по локальному запущенному VPN-устройству. Например, при использовании RRAS на базе Windows Server 2012 можно использовать системный монитор для отслеживания объема данных, полученных и переданных по VPN-подключению. С помощью объекта *Всего RAS* выберите счетчики *Получено байт/сек* и *Передано байт/сек*:

![Счетчики производительности для мониторинга трафика VPN](../_images/guidance-hybrid-network-vpn/RRAS-perf-counters.png)

Следует сравнить результаты с пропускной способностью VPN-шлюза (от 100 Мбит/с для SKU "Базовый" или "Стандартный" до 1,25 Гбит/с для SKU VpnGw3):

![Пример графика производительности сети VPN](../_images/guidance-hybrid-network-vpn/RRAS-perf-graph.png)

**Убедитесь, что вы развернули правильное количество виртуальных машин нужного размера в соответствии с загруженностью приложения.** Определите, работает ли какая-либо из виртуальных машин в виртуальной сети Azure медленно. Если это так, то они могут быть перегружены, возможно, их слишком мало для такой загрузки или подсистемы балансировки нагрузки настроены неправильно. Чтобы определить это, [соберите и проанализируйте диагностические сведения][azure-vm-diagnostics]. Вы можете проверить результаты, используя портал Azure. Кроме того, вы можете воспользоваться различными сторонними средствами, чтобы получить подробное представление о производительности.

**Убедитесь, что приложение эффективно использует облачные ресурсы.** Выполните инструментирование кода приложений, выполняемых на каждой виртуальной машине, чтобы определить, используют ли приложения ресурсы максимально эффективно. Используйте такое средство, как [Application Insights][application-insights].

<!-- links -->

[application-insights]: /azure/application-insights/app-insights-overview-usage
[azure-vm-diagnostics]: https://azure.microsoft.com/blog/windows-azure-virtual-machine-monitoring-with-wad-extension/
[gateway-diagnostic-logs]: https://blogs.technet.microsoft.com/keithmayer/2016/10/12/step-by-step-capturing-azure-resource-manager-arm-vnet-gateway-diagnostic-logs/
[psping]: https://technet.microsoft.com/sysinternals/jj729731.aspx
[troubleshooting-vpn-errors]: https://blogs.technet.microsoft.com/rrasblog/2009/08/12/troubleshooting-common-vpn-related-errors/
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
