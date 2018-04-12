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
# <a name="connect-an-on-premises-network-to-azure-using-a-vpn-gateway"></a>Подключение локальной сети к Azure с помощью VPN-шлюза

На схеме этой эталонной архитектуры представлены сведения о том, как расширить локальную сеть в Azure с помощью подключения VPN "сеть — сеть". Трафик проходит между локальной сетью и виртуальной сетью Azure через VPN-туннель IPSec. [**Разверните это решение**.](#deploy-the-solution)

![[0]][0]

*Скачайте [файл Visio][visio-download] этой архитектуры.*

## <a name="architecture"></a>Архитектура 

Архитектура состоит из следующих компонентов:

* **Локальная сеть.** Частная локальная сеть, работающая внутри организации.

* **VPN-устройство.** Устройство или служба, предоставляющая возможность внешнего подключения к локальной сети. VPN-устройство может быть аппаратным устройством или программным решением, таким как служба маршрутизации и удаленного доступа (RRAS) в Windows Server 2012. Список поддерживаемых VPN-устройств и сведения об их настройке для подключения к VPN-шлюзу Azure см. в инструкциях для выбранного устройства в статье [VPN-устройства и параметры IPsec/IKE для подключений типа "сеть — сеть" через VPN-шлюз][vpn-appliance].

* **Виртуальная сеть**. Облачное приложение и компоненты для VPN-шлюза Azure находятся в одной [виртуальной сети][azure-virtual-network].

* **VPN-шлюз Azure.** Служба [VPN-шлюза][azure-vpn-gateway] позволяет вам подключать виртуальную сеть к локальной через VPN-устройство. Дополнительные сведения см. в статье [Подключение локальной сети к виртуальной сети Microsoft Azure][connect-to-an-Azure-vnet]. VPN-шлюз содержит следующие элементы:
  
  * **Шлюз виртуальной сети.** Ресурс, предоставляющий виртуальное VPN-устройство для виртуальной сети. Он отвечает за маршрутизацию трафика от локальной сети к виртуальной.
  * **Шлюз локальной сети.** Абстракция локального VPN-устройства. Сетевой трафик из облачного приложения к локальной сети направляется через этот шлюз.
  * **Подключение.** У соединения есть свойства, указывающие тип соединения (IPSec) и общий ключ с VPN-устройством в локальной среде для шифрования трафика.
  * **Подсеть шлюза.** Шлюз виртуальной сети находится в собственной подсети с различными требованиями, описанными ниже в разделе рекомендаций.

* **Облачное приложение.** Приложение, размещенное в Azure. Оно может содержать несколько уровней с несколькими подсетями, подключенными через подсистемы балансировки нагрузки Azure. Дополнительные сведения об инфраструктуре приложений см. в статьях [Запуск рабочих нагрузок на виртуальных машинах Windows][windows-vm-ra] и [Запуск рабочих нагрузок на виртуальной машине Linux][linux-vm-ra].

* **Внутренний балансировщик нагрузки**. Сетевой трафик от VPN-шлюза направляется в облачное приложение через внутреннюю подсистему балансировки нагрузки. Подсистема балансировки нагрузки находится в интерфейсной подсети приложения.

## <a name="recommendations"></a>Рекомендации

Следующие рекомендации применимы для большинства ситуаций. Следуйте этим рекомендациям, если они не противоречат особым требованиям для вашего случая.

### <a name="vnet-and-gateway-subnet"></a>Виртуальная сеть и подсеть шлюза

Создайте виртуальную сеть Azure с адресным пространством, способным вместить все необходимые ресурсы. Убедитесь, что адресное пространство виртуальной сети имеет достаточно места для расширения, которое может потребоваться в будущем для дополнительных виртуальных машин. Адресное пространство виртуальной сети не должно перекрываться с локальной сетью. Например, на схеме выше для виртуальной сети используется адресное пространство 10.20.0.0/16.

Создайте подсеть с именем *Подсеть шлюза* с диапазоном адресов /27. Эта подсеть необходима для шлюза виртуальной сети. Выделение 32 адресов этой подсети поможет предотвратить достижение ограничения размера шлюза в будущем. Избегайте также размещения этой подсети в середине адресного пространства. Мы рекомендуем задать адресное пространство подсети шлюза в верхней границе адресного пространства виртуальной сети. На примере, приведенном на схеме, используется 10.20.255.224/27.  Вот краткая процедура вычисления [CIDR]:

1. Установите для переменных битов в адресном пространстве виртуальной сети значение 1, по количеству битов, используемых подсетью шлюза, а затем задайте для оставшихся битов значение 0.
2. Преобразуйте получившиеся биты в десятичное число и выразите его в виде адресного пространства с длиной префикса, установленной по размеру подсети шлюза.

Например, при применении шага 1 диапазон IP-адресов виртуальной сети 10.20.0.0/16 превращается в 10.20.0b11111111.0b11100000.  При преобразовании этого десятичного числа и его выражении в виде адресного пространства получаем такой результат: 10.20.255.224/27. 

> [!WARNING]
> Не развертывайте виртуальные машины в подсети шлюза. Кроме того, не назначайте группу безопасности сети для этой подсети, так как это приведет к прекращению работы шлюза.
> 
> 

### <a name="virtual-network-gateway"></a>Шлюз виртуальной сети

Выделите общедоступный IP-адрес для шлюза виртуальной сети.

Создайте шлюз виртуальной сети в подсети шлюза и назначьте ему только что выделенный общедоступный IP-адрес. Используйте тип шлюза, который больше всего соответствует вашим требованиям и поддерживается вашим VPN-устройством:

- Создайте [шлюз на основе политики][policy-based-routing], если необходимо тщательно управлять маршрутизацией запросов на основе критериев политики, как, например, префиксов адресов. Шлюзы на основе политики используют статическую маршрутизацию и работают только с подключениями типа "сеть — сеть".

- Создайте [шлюз на основе маршрутов][route-based-routing] при подключении к локальной сети с помощью службы RRAS, поддержке многосайтовых или межрегиональных подключений или реализации подключения типа "виртуальная сеть — виртуальная сеть" (включая маршруты через несколько виртуальных сетей). Шлюзы на основе маршрутов используют динамическую маршрутизацию для направления трафика между сетями. Они надежнее при сбоях в сетевом пути, чем статические маршруты, так как они могут использовать альтернативные маршруты. Шлюзы на основе маршрутов могут также снижать накладные расходы на управление, так как маршруты не потребуется обновлять вручную при изменении сетевых адресов.

Список поддерживаемых VPN-устройств см. в статье [VPN-устройства и параметры IPsec/IKE для подключений типа "сеть — сеть" через VPN-шлюз][vpn-appliances].

> [!NOTE]
> После создания шлюза изменить его тип нельзя. Для этого придется удалить шлюз, а затем создать его снова.
> 
> 

Выберите SKU VPN-шлюза Azure, который больше всего соответствует требованиям к пропускной способности. VPN-шлюз Azure доступен в трех SKU, как показано в следующей таблице. 

| SKU | Пропускная способность VPN | Макс. количество IPsec-туннелей |
| --- | --- | --- |
| базовая; |100 Мбит/с |10 |
| Стандартная |100 Мбит/с |10 |
| высокопроизводительная |200 Мбит/с |30 |

> [!NOTE]
> SKU "Базовый" несовместим с Azure ExpressRoute. [SKU можно изменить][changing-SKUs] после создания шлюза.
> 
> 

Плата взимается за каждый час использования предоставленного и доступного шлюза. Дополнительные сведения см. на странице [цен на VPN-шлюз][azure-gateway-charges].

Создайте правила маршрутизации для подсети шлюза, направляющей входящий трафик приложения из шлюза к внутренней подсистеме балансировки нагрузки, а не разрешающей передачу запросов непосредственно в виртуальные машины приложения.

### <a name="on-premises-network-connection"></a>Локальное сетевое подключение

Создайте локальный сетевой шлюз. Укажите общедоступный IP-адрес локального VPN-устройства, а также адресное пространство локальной сети. Обратите внимание, что локальному VPN-устройству необходим открытый IP-адрес, доступный на шлюзе локальной сети в VPN-шлюзе Azure. VPN-устройство не может располагаться вне устройства преобразования сетевых адресов (NAT).

Создайте подключения типа "сеть — сеть" для шлюза виртуальной и локальной сети. Выберите подключение типа "сеть — сеть" (IPSec) и укажите общий ключ. Подключение типа "сеть — сеть" с VPN-шлюзом Azure шифруется на основе протокола IPSec с использованием предварительного ключа для проверки подлинности. Укажите ключ при создании VPN-шлюза Azure. Необходимо настроить VPN-устройство, запущенное локально с тем же ключом. Сейчас другие механизмы проверки подлинности не поддерживаются.

Убедитесь, что инфраструктура локальной маршрутизации настроена для переадресации запросов по адресам в виртуальной сети Azure VPN-устройству.

В локальной сети откройте любые порты, необходимые для облачного приложения.

Проверьте подключение, чтобы убедиться в следующем:

* Локальное VPN-устройство правильно направляет трафик в облачное приложение через VPN-шлюз Azure.
* Виртуальная сеть правильно направляет трафик обратно в локальную сеть.
* Запрещенный трафик в обоих направлениях блокируется соответствующим образом.

## <a name="scalability-considerations"></a>Вопросы масштабируемости

Вы можете достичь предела вертикального масштабирования, перейдя с SKU "Базовый" или "Стандартный" VPN-шлюза к SKU VPN "Высокопроизводительный".

Для виртуальных сетей, в которых ожидается большой объем VPN-трафика, рассмотрите возможность распределения разных рабочих нагрузок по отдельным небольшим виртуальным сетям и настройки VPN-шлюза для каждой из них.

Виртуальную сеть можно разделить горизонтально или вертикально. Для разделения по горизонтали переместите некоторые экземпляры виртуальных машин из каждого уровня в подсети новой виртуальной сети. При этом каждая виртуальная сеть будет иметь одинаковую структуру и функциональные возможности. Для разделения по вертикали переопределите каждый уровень, чтобы разделить функциональные возможности на разные логические области (например, обработка заказов, выставление счетов, управление учетными записями пользователей и т. д.). Каждую функциональную область можно поместить в собственную виртуальную сеть.

Репликация локального контроллера домена Active Directory в виртуальной сети, а также реализация DNS в виртуальной сети помогает сократить объем трафика, связанного с безопасностью и администрированием, передаваемого из локальной среды в облако. Дополнительные сведения см. в статье [Расширение доменных служб Active Directory в Azure][adds-extend-domain].

## <a name="availability-considerations"></a>Вопросы доступности

Если необходимо убедиться, что локальная сеть остается доступной для VPN-шлюза Azure, реализуйте отказоустойчивый кластер для локального VPN-шлюза.

Если в вашей организации есть несколько локальных сайтов, создайте [многосайтовые подключения][vpn-gateway-multi-site] для одной или нескольких виртуальных сетей Azure. Этот подход требует динамической (на основе маршрута) маршрутизации, поэтому убедитесь, что локальный VPN-шлюз поддерживает эту функцию.

Дополнительные сведения о соглашениях об уровне обслуживания для VPN-шлюза см. [здесь][sla-for-vpn-gateway]. 

## <a name="manageability-considerations"></a>Вопросы управляемости

Наблюдайте за диагностическими сведениями с локального VPN-устройства. Этот процесс зависит от функций, предоставляемых VPN-устройством. Например, если используется служба маршрутизации и удаленного доступа на базе Windows Server 2012 ([ведение журнала для RRAS][rras-logging]).

Используйте [диагностику VPN-шлюза Azure][gateway-diagnostic-logs], чтобы получить сведения о проблемах с подключением. Эти журналы можно использовать для отслеживания сведений, таких как источник и назначение запросов на соединение, использованный протокол и способ установки подключения (или причина сбоя).

Отслеживайте операционные журналы VPN-шлюза Azure с помощью журналов аудита, доступных на портале Azure. Отдельные журналы доступны для шлюза локальной сети, шлюза сети Azure и подключения. Эти сведения можно использовать для отслеживания любых изменений шлюза. Они пригодятся, если функционирующий шлюз вдруг прекратит свою работу по какой-либо причине.

![[2]][2]

Наблюдайте за подключениями и отслеживайте соответствующие сбои. Для регистрации этих сведений и формирования отчетов на их основе можно воспользоваться пакетом для мониторинга, например [Nagios][nagios].

## <a name="security-considerations"></a>Вопросы безопасности

Создайте разные общие ключи для каждого VPN-шлюза. Используйте надежный общий ключ для защиты от атак методом подбора.

> [!NOTE]
> В настоящее время нельзя использовать Azure Key Vault для предварительного предоставления ключей VPN-шлюза Azure.
> 
> 

Убедитесь, что локальное VPN-устройство использует метод шифрования, [совместимый с VPN-шлюзом Azure][vpn-appliance-ipsec]. Для маршрутизации на основе политик VPN-шлюз Azure поддерживает алгоритмы шифрования AES256, AES128 и 3DES. Шлюзы на основе маршрута поддерживают AES256 и 3DES.

Если ваше локальное VPN-устройство находится в сети периметра с брандмауэром между ней и Интернетом, то может потребоваться настроить [дополнительные правила брандмауэра][additional-firewall-rules], разрешающие VPN-подключение "сеть — сеть".

Если приложение в виртуальной сети отправляет данные в Интернет, следует рассмотреть возможность [реализации принудительного туннелирования][forced-tunneling], чтобы направлять весь интернет-трафик через локальную сеть. Такой подход позволяет производить аудит исходящих запросов приложения из локальной инфраструктуры.

> [!NOTE]
> Принудительное туннелирование может повлиять на возможность подключения к службам Azure (например, к службе хранилища) и к службе Windows License Manager.
> 
> 


## <a name="troubleshooting"></a>Устранение неполадок 

Общие сведения об устранении неполадок, связанных с VPN, см. [здесь][troubleshooting-vpn-errors].

Следующие рекомендации помогут определить, правильно ли работает ваше локальное VPN-устройство.

- **Проверьте все файлы журнала, созданные VPN-устройством, на наличие ошибок или сбоев.**

    Так вы сможете определить, правильно ли функционирует VPN-устройство. Расположение этих сведений будет различаться в зависимости от устройства. Например, при использовании RRAS на базе Windows Server 2012 для отображения сведений об ошибке события для службы RRAS можно использовать следующую команду PowerShell:

    ```PowerShell
    Get-EventLog -LogName System -EntryType Error -Source RemoteAccess | Format-List -Property *
    ```

    В свойстве *Message* каждой записи содержится описание ошибки. Несколько распространенных примеров:

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

    Вы также можете получить сведения из журнала событий о попытках установить подключение через службу RRAS с помощью следующей команды PowerShell: 

    ```
    Get-EventLog -LogName Application -Source RasClient | Format-List -Property *
    ```

    В случае сбоя подключения этот журнал будет содержать ошибки, которые выглядят следующим образом:

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

- **Проверьте подключение и маршрутизацию через VPN-шлюз.**

    VPN-устройство может неправильно осуществлять маршрутизацию трафика через VPN-шлюз Azure. Используйте [PsPing][psping] для проверки подключения и маршрутизации через VPN-шлюз. Например, чтобы проверить соединение с локального компьютера на веб-сервер, расположенный в виртуальной сети, выполните следующую команду (заменив `<<web-server-address>>` адресом веб-сервера):

    ```
    PsPing -t <<web-server-address>>:80
    ```

    Если локальный компьютер может осуществлять маршрутизацию трафика на веб-сервер, отобразится результат, аналогичный приведенному ниже:

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

    Если локальный компьютер не может связаться с указанным адресом назначения, появятся такие сообщения:

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

- **Убедитесь, что локальный брандмауэр разрешает передачу VPN-трафика и что открыты соответствующие порты.**

- **Убедитесь, что локальное VPN-устройство использует метод шифрования, [совместимый с VPN-шлюзом Azure][vpn-appliance]**. Для маршрутизации на основе политик VPN-шлюз Azure поддерживает алгоритмы шифрования AES256, AES128 и 3DES. Шлюзы на основе маршрута поддерживают AES256 и 3DES.

Для определения проблемы с VPN-шлюзом Azure полезны следующие рекомендации:

- **Изучите [журналы диагностики VPN-шлюза Azure][gateway-diagnostic-logs] на наличие потенциальных проблем.**

- **Убедитесь, что VPN-шлюз Azure и локальное VPN-устройство настроены с использованием одного и того же общего ключа проверки подлинности.**

    Вы можете просмотреть общий ключ, сохраненный VPN-шлюзом Azure, используя следующую команду Azure CLI:

    ```
    azure network vpn-connection shared-key show <<resource-group>> <<vpn-connection-name>>
    ```

    Для просмотра общего ключа, настроенного для этого устройства, используйте команду, подходящую для вашего локального VPN-устройства.

    Убедитесь, что подсеть *GatewaySubnet*, в которой размещен VPN-шлюз Azure, не связана с NSG.

    Сведения о подсети можно просмотреть, используя следующую команду Azure CLI:

    ```
    azure network vnet subnet show -g <<resource-group>> -e <<vnet-name>> -n GatewaySubnet
    ```

    Убедитесь, что нет поля данных с именем *Идентификатор группы безопасности сети*. В следующем примере показаны результаты для экземпляра подсети *GatewaySubnet* с назначенной NSG (*VPN-Gateway-Group*). Если для этой NSG определены какие-либо правила, шлюз может неправильно работать.

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

- **Убедитесь, что виртуальные машины в виртуальной сети Azure настроены для разрешения передачи трафика, поступающего из виртуальной сети.**

    Проверьте все правила NSG, связанные с подсетями, содержащими эти виртуальные машины. Все правила NSG можно просмотреть, используя следующую команду Azure CLI:

    ```
    azure network nsg show -g <<resource-group>> -n <<nsg-name>>
    ```

- **Убедитесь, что подключен VPN-шлюз Azure.**

    Чтобы проверить текущее состояние VPN-подключения Azure, можно использовать следующую команду Azure PowerShell. Параметр `<<connection-name>>` представляет собой имя VPN-подключения Azure, которое связывает шлюз виртуальной сети с локальным.

    ```
    Get-AzureRmVirtualNetworkGatewayConnection -Name <<connection-name>> - ResourceGroupName <<resource-group>>
    ```

    В следующем фрагменте кода показаны результаты для подключенного (первый пример) и отключенного шлюза (второй пример):

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

Следующие рекомендации помогут определить, имеются ли проблемы с конфигурацией виртуальной машины узла, использованием пропускной способности сети или производительностью приложения:

- **Проверьте правильность настройки брандмауэра в гостевой ОС на виртуальных машинах Azure в подсети, чтобы разрешить передачу трафика из локальных диапазонов IP-адресов.**

- **Убедитесь, что объем трафика не приближается к лимиту пропускной способности VPN-шлюза Azure.**

    Это можно проверить по локальному запущенному VPN-устройству. Например, при использовании RRAS на базе Windows Server 2012 можно использовать системный монитор для отслеживания объема данных, полученных и переданных по VPN-подключению. С помощью объекта *Всего RAS* выберите счетчики *Получено байт/сек* и *Передано байт/сек*:

    ![[3]][3]

    Следует сравнить результаты с пропускной способностью VPN-шлюза (100 Мбит/с для SKU "Базовый" или "Стандартный" и 200 Мбит/с для SKU "Высокопроизводительный"):

    ![[4]][4]

- **Убедитесь, что вы развернули правильное количество виртуальных машин нужного размера в соответствии с загруженностью приложения.**

    Определите, работает ли какая-либо из виртуальных машин в виртуальной сети Azure медленно. Если это так, то они могут быть перегружены, возможно, их слишком мало для такой загрузки или подсистемы балансировки нагрузки настроены неправильно. Чтобы определить это, [соберите и проанализируйте диагностические сведения][azure-vm-diagnostics]. Вы можете проверить результаты, используя портал Azure. Кроме того, вы можете воспользоваться различными сторонними средствами, чтобы получить подробное представление о производительности.

- **Убедитесь, что приложение эффективно использует облачные ресурсы.**

    Выполните инструментирование кода приложений, выполняемых на каждой виртуальной машине, чтобы определить, используют ли приложения ресурсы максимально эффективно. Используйте такое средство, как [Application Insights][application-insights].

## <a name="deploy-the-solution"></a>Развертывание решения


**Предварительные требования.** Вам необходима настроенная локальная инфраструктура с подходящим сетевым устройством.

Чтобы развернуть решение, сделайте следующее.

1. Нажмите кнопку ниже:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fvpn%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Подождите, пока на портале Azure не откроется ссылка, а затем сделайте следующее: 
   * Имя **группы ресурсов** уже определено в файле параметров, поэтому выберите **Создать** и введите `ra-hybrid-vpn-rg` в текстовом поле.
   * В раскрывающемся списке **Расположение** выберите регион.
   * Не изменяйте поля **Template Root Uri** (Корневой URI шаблона) или **Parameter Root Uri** (Корневой URI параметра).
   * Прочтите условия использования и установите флажок **Я принимаю указанные выше условия**.
   * Нажмите кнопку **Приобрести**.
3. Дождитесь завершения развертывания.



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