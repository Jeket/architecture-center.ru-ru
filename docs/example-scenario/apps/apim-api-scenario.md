---
title: Перенос устаревших веб-приложений в архитектуру на основе API в Azure
description: Использование службы "Управление API Azure" для модернизации устаревшего веб-приложения.
author: begim
ms.date: 09/13/2018
ms.openlocfilehash: f468b3c6dc1c58e03555613b152882316ae2a017
ms.sourcegitcommit: 0a31fad9b68d54e2858314ca5fe6cba6c6b95ae4
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/13/2018
ms.locfileid: "51610589"
---
# <a name="migrating-a-legacy-web-application-to-an-api-based-architecture-on-azure"></a>Перенос устаревших веб-приложений в архитектуру на основе API в Azure

Компания электронной коммерции в сфере туризма модернизирует свой старый стек программного обеспечения на основе браузера. Хотя существующий стек в основном монолитный, в недавнем проекте существуют некоторые [службы HTTP на основе SOAP][soap]. Они рассматривают возможность создания дополнительных потоков дохода, чтобы монетизировать часть внутренней интеллектуальной собственности, которая была разработана.

Цели проекта включают в себя устранение технического долга, улучшение текущего обслуживания и ускорение разработки функций с меньшим количеством ошибок регрессии. Во избежание риска проект будет использовать итеративный процесс, а некоторые шаги будут выполняться параллельно:

* Команда разработчиков будет модернизировать серверную часть приложения, которая состоит из реляционных баз данных, размещенных на виртуальных машинах.
* Внутренняя команда разработчиков создаст бизнес-функцию, которая будет предоставляться через новые API по протоколу HTTP.
* Команда по разработке контрактов создаст новый пользовательский интерфейс на основе браузера, который будет размещен в Azure.

Новые возможности приложения будут доставлены поэтапно. Они будут постепенно заменять существующие функции пользовательского интерфейса в рамках клиент-серверной архитектуры на основе браузера (локальное размещение), которые компания, занимающаяся электронной коммерцией, использует сегодня.

Команда управления не хочет модернизировать что-то без необходимости. Они также хотят сохранить контроль над областью и затратами. Для этого они решили сохранить существующие службы HTTP на основе SOAP. Они также намерены свести к минимуму изменения в существующем пользовательском интерфейсе. [Управление Azure API (APIM)][apim] может использоваться для решения многих требований и ограничений проекта.

## <a name="architecture"></a>Архитектура

![Схема архитектуры][architecture]

Новый пользовательский интерфейс будет размещен как приложение Azure "платформа как служба" (PaaS) и будет зависеть как от существующих, так и от новых API-интерфейсов HTTP. Эти API-интерфейсы будут поставляться с более совершенным набором интерфейсов, обеспечивающим лучшую производительность, упрощенную интеграцию и будущую расширяемость.

### <a name="components-and-security"></a>Компоненты и безопасность

1. Существующее локальное веб-приложение будет по-прежнему напрямую использовать существующие локальные веб-службы.
2. Вызовы из существующего веб-приложения к существующим службам HTTP останутся без изменений. Для корпоративной сети эти вызовы являются внутренними.
3. Входящие вызовы осуществляются от Azure к существующим внутренним службам:
    * Группа безопасности позволяет трафику из экземпляра APIM проходить через корпоративный брандмауэр к существующим локальным службам, [используя безопасную транспортировку (HTTPS/SSL)][apim-ssl].
    * Группа пользователей будет разрешать входящие вызовы к службам только из экземпляра APIM. Это требование удовлетворяется с помощью [ведения белых списков IP-адресов экземпляра APIM ][apim-whitelist-ip] по периметру корпоративной сети.
    * В локальном конвейере запросов на обслуживание HTTP настраивается новый модуль (для выполнения **только** тех подключений, исходящих извне), который будет проверять [сертификат, который предоставит APIM][apim-mutualcert-auth].
1. Новый API:
    * Открывается только через экземпляр APIM, который предоставит API-интерфейс. Новый API не будет доступен напрямую.
    * Разработано и опубликовано как [Веб-API приложения PaaS Azure][azure-api-apps].
    * Добавлен в белый список (через [параметры веб-приложения][azure-appservice-ip-restrict]) для приема только [виртуального IP-адреса APIM][apim-faq-vip].
    * Размещен в веб-приложениях Azure со включенной безопасной транспортировкой/SSL.
    * Имеет включенную авторизацию, [предоставленную службой приложений Azure][azure-appservice-auth] с использованием Azure Active Directory и OAuth2.
2. Новое веб-приложение на основе браузера будет зависеть от экземпляра управления API Azure для **обоих** существующего API протокола HTTP и нового API.

Экземпляр APIM будет настроен для сопоставления устаревших служб HTTP с новым контрактом API. Таким образом, новый веб-интерфейс не знает об интеграции с набором устаревших служб/API-интерфейсов и новых API-интерфейсов. В будущем команда проекта постепенно добавит новым API-интерфейсам функциональность и удалит исходные службы. Эти изменения будут обрабатываться конфигурацией APIM, оставляя пользовательский интерфейс переднего плана нетронутым и избегая работы по перепланировке.

### <a name="alternatives"></a>Альтернативные варианты

* Если организация планировала полностью переместить свою инфраструктуру в Azure, включая виртуальные машины, на которых размещены приложения прежних версий, то APIM все равно будет отличным вариантом, поскольку он может выступать в качестве интерфейса для любой доступной конечной точки HTTP.
* Если клиент решил, что существующие конечные точки должны быть скрыты и не должны быть опубликованы публично, то их экземпляр управления API может быть связан с [виртуальной сетью Azure (VNet)] [ azure-vnet]:
  * В [сценарии Azure lift and shift (перемещение)] [ azure-vm-lift-shift], связанном с развернутой виртуальной сетью Microsoft Azure, клиент может напрямую обращаться к серверной службе через частные IP-адреса.
  * В локальном сценарии экземпляр управления API может вернуться к внутренней службе в частном порядке через [VPN шлюз Azure и межсетевое VPN соединение IPSec][azure-vpn] или [ExpressRoute][azure-er], что делает его [гибридным использованием Azure и локальным сценарием ][azure-hybrid].
* Экземпляр управления API можно сохранить конфиденциальным, развернув его во внутреннем режиме. Затем развертывание можно использовать со [Шлюзом приложения Azure ][azure-appgw], чтобы разрешить публичный доступ для некоторых API, в то время как другие остаются внутренними. Дополнительные сведения см. в статье [Интеграция службы управления API во внутреннюю сеть со Щлюзом приложений][apim-vnet-internal].

> [!NOTE]
> Общие сведения о подключении управления API к виртуальной сети [см.здесь][apim-vnet].

### <a name="availability-and-scalability"></a>Доступность и масштабируемость

* Службу управления API Azure можно [масштабировать][apim-scaleout], выбрав ценовую категорию и затем добавив единицы.
* Масштабирование также происходит [автоматически с помощью автоматического масштабирования][apim-autoscale].
* [Развертывание в несколько регионов] [apim-multi-regions] позволит включить опции по выполнению отработки отказа и может быть выполнено с [уровнем "Премиум"][apim-pricing].
* Рассмотрим [интеграцию управления API Azure в Azure Application Insights][azure-apim-ai], которая также отображает метрики через [Azure Monitor][azure-mon].

## <a name="deployment"></a>Развертывание

Чтобы начать работу, [создайте в портале экземпляр управления API Azure.][apim-create]

Кроме того, в существующем Azure Resource Manager вы можете выбрать [шаблон быстрого запуска][azure-quickstart-templates-apim], который будет соответствовать вашему конкретному варианту использования.

## <a name="pricing"></a>Цены

Для службы управления API предлагаются четыре уровня: "Разработка", "Базовый", Стандартный" и "Премиум". Подробное руководство по различиям этих уровней можно найти на странице [Цены на управление API.][apim-pricing]

Клиенты могут масштабировать управление API путем добавления и удаления единиц. Мощность единиц зависит от их уровня.

> [!NOTE]
> Уровень "Разработка" можно использовать для оценки компонентов управления API. Уровень "Разработка" нельзя использовать для рабочей среды.

Чтобы просмотреть прогнозируемые затраты и настроить потребности развертывания, вы можете изменить количество единиц масштабирования и экземпляров службы приложений в [Калькуляторе цен Azure][pricing-calculator].

## <a name="related-resources"></a>Связанные ресурсы

См. обширную документацию по [управлению API Azure][apim]


<!-- links -->
[architecture]: ./media/architecture-apim-api-scenario.png
[apim-create]: /azure/api-management/get-started-create-service-instance
[apim-git]: /azure/api-management/api-management-configuration-repository-git
[apim-multi-regions]: /azure/api-management/api-management-howto-deploy-multi-region
[apim-autoscale]: /azure/api-management/api-management-howto-autoscale
[apim-scaleout]: /azure/api-management/upgrade-and-scale
[azure-apim-ai]: /azure/api-management/api-management-howto-app-insights
[azure-ai]: /azure/application-insights/
[azure-mon]: /azure/monitoring-and-diagnostics/monitoring-overview
[azure-appgw]: /azure/application-gateway/application-gateway-introduction
[apim-vnet-internal]: /azure/api-management/api-management-howto-integrate-internal-vnet-appgateway
[apim-vnet]: /azure/api-management/api-management-using-with-vnet
[azure-hybrid]: /azure/architecture/reference-architectures/hybrid-networking/
[azure-er]: /azure/expressroute/expressroute-introduction
[azure-vpn]: /azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal
[azure-vnet]: /azure/virtual-network/virtual-networks-overview
[azure-appservice-auth]: /azure/app-service/app-service-authentication-overview#identity-providers
[apim-faq-vip]: /azure/api-management/api-management-faq#is-the-api-management-gateway-ip-address-constant-can-i-use-it-in-firewall-rules
[azure-appservice-ip-restrict]: /azure/app-service/app-service-ip-restrictions
[azure-api-apps]: /azure/app-service/
[apim-ssl]: /azure/api-management/api-management-howto-manage-protocols-ciphers
[apim-mutualcert-auth]: /azure/api-management/api-management-howto-mutual-certificates
[apim-whitelist-ip]: /azure/api-management/api-management-faq#is-the-api-management-gateway-ip-address-constant-can-i-use-it-in-firewall-rules
[anti-corruption-layer-pattern]: /azure/architecture/patterns/anti-corruption-layer
[apim]: /azure/api-management/api-management-key-concepts
[apim-api-design-guidance]: /azure/architecture/best-practices/api-design
[visualstudio-youtube-solid-design]: https://youtu.be/agkWYPUcLpg
[azure-vm-lift-shift]: https://azure.microsoft.com/resources/azure-virtual-datacenter-lift-and-shift-guide/
[standard-pricing-calc]: https://azure.com/e/
[premium-pricing-calc]: https://azure.com/e/
[apim-pricing]: https://azure.microsoft.com/pricing/details/api-management/
[azure-quickstart-templates-apim]: https://azure.microsoft.com/resources/templates/?term=API+Management&pageNumber=1
[soap]: https://en.wikipedia.org/wiki/SOAP
[pricing-calculator]: https://azure.com/e/0e916a861fac464db61342d378cc0bd6
