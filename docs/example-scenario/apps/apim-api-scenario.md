---
title: Перенос веб-приложений в архитектуру на основе API
titleSuffix: Azure Example Scenarios
description: Использование службы "Управление API Azure" для модернизации устаревшего веб-приложения.
author: begim
ms.date: 09/13/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: fasttrack
social_image_url: /azure/architecture/example-scenario/apps/media/architecture-apim-api-scenario.png
ms.openlocfilehash: cf3d4b7ed7fce04e6688f68e382caeec78abd100
ms.sourcegitcommit: 3b15d65e7c35a19506e562c444343f8467b6a073
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/25/2019
ms.locfileid: "54907968"
---
# <a name="migrating-a-legacy-web-application-to-an-api-based-architecture-on-azure"></a><span data-ttu-id="068a8-103">Перенос устаревших веб-приложений в архитектуру на основе API в Azure</span><span class="sxs-lookup"><span data-stu-id="068a8-103">Migrating a legacy web application to an API-based architecture on Azure</span></span>

<span data-ttu-id="068a8-104">Компания электронной коммерции в сфере туризма модернизирует свой старый стек программного обеспечения на основе браузера.</span><span class="sxs-lookup"><span data-stu-id="068a8-104">An e-commerce company in the travel industry is modernizing their legacy browser-based software stack.</span></span> <span data-ttu-id="068a8-105">Хотя существующий стек в основном монолитный, в недавнем проекте существуют некоторые [службы HTTP на основе SOAP][soap].</span><span class="sxs-lookup"><span data-stu-id="068a8-105">While their existing stack is mostly monolithic, some [SOAP-based HTTP services][soap] exist from a recent project.</span></span> <span data-ttu-id="068a8-106">Они рассматривают возможность создания дополнительных потоков дохода, чтобы монетизировать часть внутренней интеллектуальной собственности, которая была разработана.</span><span class="sxs-lookup"><span data-stu-id="068a8-106">They are considering the creation of additional revenue streams to monetize some of the internal intellectual property that's been developed.</span></span>

<span data-ttu-id="068a8-107">Цели проекта включают в себя устранение технического долга, улучшение текущего обслуживания и ускорение разработки функций с меньшим количеством ошибок регрессии.</span><span class="sxs-lookup"><span data-stu-id="068a8-107">Goals for the project include addressing technical debt, improving ongoing maintenance, and accelerating feature development with fewer regression bugs.</span></span> <span data-ttu-id="068a8-108">Во избежание риска проект будет использовать итеративный процесс, а некоторые шаги будут выполняться параллельно:</span><span class="sxs-lookup"><span data-stu-id="068a8-108">The project will use an iterative process to avoid risk, with some steps performed in parallel:</span></span>

- <span data-ttu-id="068a8-109">Команда разработчиков будет модернизировать серверную часть приложения, которая состоит из реляционных баз данных, размещенных на виртуальных машинах.</span><span class="sxs-lookup"><span data-stu-id="068a8-109">The development team will modernize the application back end, which is composed of relational databases hosted on VMs.</span></span>
- <span data-ttu-id="068a8-110">Внутренняя команда разработчиков создаст бизнес-функцию, которая будет предоставляться через новые API по протоколу HTTP.</span><span class="sxs-lookup"><span data-stu-id="068a8-110">The in-house development team will write new business functionality that will be exposed over new HTTP APIs.</span></span>
- <span data-ttu-id="068a8-111">Команда по разработке контрактов создаст новый пользовательский интерфейс на основе браузера, который будет размещен в Azure.</span><span class="sxs-lookup"><span data-stu-id="068a8-111">A contract development team will build a new browser-based UI, which will be hosted in Azure.</span></span>

<span data-ttu-id="068a8-112">Новые возможности приложения будут доставлены поэтапно.</span><span class="sxs-lookup"><span data-stu-id="068a8-112">New application features will be delivered in stages.</span></span> <span data-ttu-id="068a8-113">Они будут постепенно заменять существующие функции пользовательского интерфейса в рамках клиент-серверной архитектуры на основе браузера (локальное размещение), которые компания, занимающаяся электронной коммерцией, использует сегодня.</span><span class="sxs-lookup"><span data-stu-id="068a8-113">These features will gradually replace the existing browser-based client-server UI functionality (hosted on-premises) that powers their e-commerce business today.</span></span>

<span data-ttu-id="068a8-114">Команда управления не хочет модернизировать что-то без необходимости.</span><span class="sxs-lookup"><span data-stu-id="068a8-114">The management team does not want to modernize unnecessarily.</span></span> <span data-ttu-id="068a8-115">Они также хотят сохранить контроль над областью и затратами.</span><span class="sxs-lookup"><span data-stu-id="068a8-115">They also want to maintain control of scope and costs.</span></span> <span data-ttu-id="068a8-116">Для этого они решили сохранить существующие службы HTTP на основе SOAP.</span><span class="sxs-lookup"><span data-stu-id="068a8-116">To do this, they have decided to preserve their existing SOAP HTTP services.</span></span> <span data-ttu-id="068a8-117">Они также намерены свести к минимуму изменения в существующем пользовательском интерфейсе.</span><span class="sxs-lookup"><span data-stu-id="068a8-117">They also intend to minimize changes to the existing UI.</span></span> <span data-ttu-id="068a8-118">[Управление Azure API (APIM)][apim] может использоваться для решения многих требований и ограничений проекта.</span><span class="sxs-lookup"><span data-stu-id="068a8-118">[Azure API Management (APIM)][apim] can be utilized to address many of the project's requirements and constraints.</span></span>

## <a name="architecture"></a><span data-ttu-id="068a8-119">Архитектура</span><span class="sxs-lookup"><span data-stu-id="068a8-119">Architecture</span></span>

![Схема архитектуры][architecture]

<span data-ttu-id="068a8-121">Новый пользовательский интерфейс будет размещен как приложение Azure "платформа как служба" (PaaS) и будет зависеть как от существующих, так и от новых API-интерфейсов HTTP.</span><span class="sxs-lookup"><span data-stu-id="068a8-121">The new UI will be hosted as a platform as a service (PaaS) application on Azure, and will depend on both existing and new HTTP APIs.</span></span> <span data-ttu-id="068a8-122">Эти API-интерфейсы будут поставляться с более совершенным набором интерфейсов, обеспечивающим лучшую производительность, упрощенную интеграцию и будущую расширяемость.</span><span class="sxs-lookup"><span data-stu-id="068a8-122">These APIs will ship with a better-designed set of interfaces enabling better performance, easier integration, and future extensibility.</span></span>

### <a name="components-and-security"></a><span data-ttu-id="068a8-123">Компоненты и безопасность</span><span class="sxs-lookup"><span data-stu-id="068a8-123">Components and Security</span></span>

1. <span data-ttu-id="068a8-124">Существующее локальное веб-приложение будет по-прежнему напрямую использовать существующие локальные веб-службы.</span><span class="sxs-lookup"><span data-stu-id="068a8-124">The existing on-premises web application will continue to directly consume the existing on-premises web services.</span></span>
2. <span data-ttu-id="068a8-125">Вызовы из существующего веб-приложения к существующим службам HTTP останутся без изменений.</span><span class="sxs-lookup"><span data-stu-id="068a8-125">Calls from the existing web app to the existing HTTP services will remain unchanged.</span></span> <span data-ttu-id="068a8-126">Для корпоративной сети эти вызовы являются внутренними.</span><span class="sxs-lookup"><span data-stu-id="068a8-126">These calls are internal to the corporate network.</span></span>
3. <span data-ttu-id="068a8-127">Входящие вызовы осуществляются от Azure к существующим внутренним службам:</span><span class="sxs-lookup"><span data-stu-id="068a8-127">Inbound calls are made from Azure to the existing internal services:</span></span>
    - <span data-ttu-id="068a8-128">Группа безопасности позволяет трафику из экземпляра APIM проходить через корпоративный брандмауэр к существующим локальным службам, [используя безопасную транспортировку (HTTPS/SSL)][apim-ssl].</span><span class="sxs-lookup"><span data-stu-id="068a8-128">The security team allows traffic from the APIM instance to pass through the corporate firewall to the existing on-premises services [using secure transport (HTTPS/SSL)][apim-ssl].</span></span>
    - <span data-ttu-id="068a8-129">Группа пользователей будет разрешать входящие вызовы к службам только из экземпляра APIM.</span><span class="sxs-lookup"><span data-stu-id="068a8-129">The operations team will allow inbound calls to the services only from the APIM instance.</span></span> <span data-ttu-id="068a8-130">Это требование удовлетворяется с помощью [ведения белых списков IP-адресов экземпляра APIM ][apim-whitelist-ip] по периметру корпоративной сети.</span><span class="sxs-lookup"><span data-stu-id="068a8-130">This requirement is met by [white-listing the IP address of the APIM instance][apim-whitelist-ip] within the corporate network perimeter.</span></span>
    - <span data-ttu-id="068a8-131">В локальном конвейере запросов на обслуживание HTTP настраивается новый модуль (для выполнения **только** тех подключений, исходящих извне), который будет проверять [сертификат, который предоставит APIM][apim-mutualcert-auth].</span><span class="sxs-lookup"><span data-stu-id="068a8-131">A new module is configured into the on-premises HTTP services request pipeline (to act upon **only** those connections originating externally), which will validate [a certificate which APIM will provide][apim-mutualcert-auth].</span></span>
4. <span data-ttu-id="068a8-132">Новый API:</span><span class="sxs-lookup"><span data-stu-id="068a8-132">The new API:</span></span>
    - <span data-ttu-id="068a8-133">Открывается только через экземпляр APIM, который предоставит API-интерфейс.</span><span class="sxs-lookup"><span data-stu-id="068a8-133">Is surfaced only through the APIM instance, which will provide the API facade.</span></span> <span data-ttu-id="068a8-134">Новый API не будет доступен напрямую.</span><span class="sxs-lookup"><span data-stu-id="068a8-134">The new API won't be accessed directly.</span></span>
    - <span data-ttu-id="068a8-135">Разработано и опубликовано как [Веб-API приложения PaaS Azure][azure-api-apps].</span><span class="sxs-lookup"><span data-stu-id="068a8-135">Is developed and published as an [Azure PaaS Web API App][azure-api-apps].</span></span>
    - <span data-ttu-id="068a8-136">Добавлен в белый список (через [параметры веб-приложения][azure-appservice-ip-restrict]) для приема только [виртуального IP-адреса APIM][apim-faq-vip].</span><span class="sxs-lookup"><span data-stu-id="068a8-136">Is white-listed (via [Web App settings][azure-appservice-ip-restrict]) to accept only the [APIM VIP][apim-faq-vip].</span></span>
    - <span data-ttu-id="068a8-137">Размещен в веб-приложениях Azure со включенной безопасной транспортировкой/SSL.</span><span class="sxs-lookup"><span data-stu-id="068a8-137">Is hosted in Azure Web Apps with Secure Transport/SSL turned on.</span></span>
    - <span data-ttu-id="068a8-138">Имеет включенную авторизацию, [предоставленную службой приложений Azure][azure-appservice-auth] с использованием Azure Active Directory и OAuth2.</span><span class="sxs-lookup"><span data-stu-id="068a8-138">Has authorization enabled, [provided by the Azure App Service][azure-appservice-auth] using Azure Active Directory and OAuth2.</span></span>
5. <span data-ttu-id="068a8-139">Новое веб-приложение на основе браузера будет зависеть от экземпляра управления API Azure для **обоих** существующего API протокола HTTP и нового API.</span><span class="sxs-lookup"><span data-stu-id="068a8-139">The new browser-based web application will depend on the Azure API Management instance for **both** the existing HTTP API and the new API.</span></span>

<span data-ttu-id="068a8-140">Экземпляр APIM будет настроен для сопоставления устаревших служб HTTP с новым контрактом API.</span><span class="sxs-lookup"><span data-stu-id="068a8-140">The APIM instance will be configured to map the legacy HTTP services to a new API contract.</span></span> <span data-ttu-id="068a8-141">Таким образом, новый веб-интерфейс не знает об интеграции с набором устаревших служб/API-интерфейсов и новых API-интерфейсов.</span><span class="sxs-lookup"><span data-stu-id="068a8-141">By doing this, the new Web UI is unaware of the integration with a set of legacy services/APIs and new APIs.</span></span> <span data-ttu-id="068a8-142">В будущем команда проекта постепенно добавит новым API-интерфейсам функциональность и удалит исходные службы.</span><span class="sxs-lookup"><span data-stu-id="068a8-142">In the future, the project team will gradually port functionality to the new APIs and retire the original services.</span></span> <span data-ttu-id="068a8-143">Эти изменения будут обрабатываться конфигурацией APIM, оставляя пользовательский интерфейс переднего плана нетронутым и избегая работы по перепланировке.</span><span class="sxs-lookup"><span data-stu-id="068a8-143">These changes will be handled within APIM configuration, leaving the front-end UI unaffected and avoiding redevelopment work.</span></span>

### <a name="alternatives"></a><span data-ttu-id="068a8-144">Альтернативные варианты</span><span class="sxs-lookup"><span data-stu-id="068a8-144">Alternatives</span></span>

- <span data-ttu-id="068a8-145">Если организация планировала полностью переместить свою инфраструктуру в Azure, включая виртуальные машины, на которых размещены приложения прежних версий, то APIM все равно будет отличным вариантом, поскольку он может выступать в качестве интерфейса для любой доступной конечной точки HTTP.</span><span class="sxs-lookup"><span data-stu-id="068a8-145">If the organization was planning to move their infrastructure entirely to Azure, including the VMs hosting the legacy applications, then APIM would still be a great option since it can act as a facade for any addressable HTTP endpoint.</span></span>
- <span data-ttu-id="068a8-146">Если клиент решил, что существующие конечные точки должны быть скрыты и не должны быть опубликованы публично, то их экземпляр управления API может быть связан с [виртуальной сетью Azure (VNet)] [ azure-vnet]:</span><span class="sxs-lookup"><span data-stu-id="068a8-146">If the customer had decided to keep the existing endpoints private and not expose them publicly, their API Management instance could be linked to an [Azure Virtual Network (VNet)][azure-vnet]:</span></span>
  - <span data-ttu-id="068a8-147">В [сценарии Azure lift and shift (перемещение)] [ azure-vm-lift-shift], связанном с развернутой виртуальной сетью Microsoft Azure, клиент может напрямую обращаться к серверной службе через частные IP-адреса.</span><span class="sxs-lookup"><span data-stu-id="068a8-147">In an [Azure lift and shift scenario][azure-vm-lift-shift] linked to their deployed Azure Virtual Network, the customer could directly address the back-end service through private IP addresses.</span></span>
  - <span data-ttu-id="068a8-148">В локальном сценарии экземпляр управления API может вернуться к внутренней службе в частном порядке через [VPN шлюз Azure и межсетевое VPN соединение IPSec][azure-vpn] или [ExpressRoute][azure-er], что делает его [гибридным использованием Azure и локальным сценарием ][azure-hybrid].</span><span class="sxs-lookup"><span data-stu-id="068a8-148">In the on-premises scenario, the API Management instance could reach back to the internal service privately via an [Azure VPN gateway and site-to-site IPSec VPN connection][azure-vpn] or [ExpressRoute][azure-er] making this a [hybrid Azure and on-premises scenario][azure-hybrid].</span></span>
- <span data-ttu-id="068a8-149">Экземпляр управления API можно сохранить конфиденциальным, развернув его во внутреннем режиме.</span><span class="sxs-lookup"><span data-stu-id="068a8-149">The API Management instance can be kept private by deploying the API Management instance in Internal mode.</span></span> <span data-ttu-id="068a8-150">Затем развертывание можно использовать со [Шлюзом приложения Azure ][azure-appgw], чтобы разрешить публичный доступ для некоторых API, в то время как другие остаются внутренними.</span><span class="sxs-lookup"><span data-stu-id="068a8-150">The deployment could then be used with an [Azure Application Gateway][azure-appgw] to enable public access for some APIs while others remain internal.</span></span> <span data-ttu-id="068a8-151">Дополнительные сведения см. в статье [Интеграция службы управления API во внутреннюю сеть со Щлюзом приложений][apim-vnet-internal].</span><span class="sxs-lookup"><span data-stu-id="068a8-151">For more information, see [Connecting APIM in internal mode to a VNET][apim-vnet-internal].</span></span>

> [!NOTE]
> <span data-ttu-id="068a8-152">Общие сведения о подключении управления API к виртуальной сети [см.здесь][apim-vnet].</span><span class="sxs-lookup"><span data-stu-id="068a8-152">For general information on connecting API Management to a VNET, [see here][apim-vnet].</span></span>

### <a name="availability-and-scalability"></a><span data-ttu-id="068a8-153">Доступность и масштабируемость</span><span class="sxs-lookup"><span data-stu-id="068a8-153">Availability and scalability</span></span>

- <span data-ttu-id="068a8-154">Службу управления API Azure можно [масштабировать][apim-scaleout], выбрав ценовую категорию и затем добавив единицы.</span><span class="sxs-lookup"><span data-stu-id="068a8-154">Azure API Management can be [scaled out][apim-scaleout] by choosing a pricing tier and then adding units.</span></span>
- <span data-ttu-id="068a8-155">Масштабирование также происходит [автоматически с помощью автоматического масштабирования][apim-autoscale].</span><span class="sxs-lookup"><span data-stu-id="068a8-155">Scaling also happen [automatically with auto scaling][apim-autoscale].</span></span>
- <span data-ttu-id="068a8-156">[Развертывание в несколько регионов] [apim-multi-regions] позволит включить опции по выполнению отработки отказа и может быть выполнено с [уровнем "Премиум"][apim-pricing].</span><span class="sxs-lookup"><span data-stu-id="068a8-156">[Deploying across multiple regions][apim-multi-regions] will enable fail over options and can be done in the [Premium tier][apim-pricing].</span></span>
- <span data-ttu-id="068a8-157">Рассмотрим [интеграцию управления API Azure в Azure Application Insights][azure-apim-ai], которая также отображает метрики через [Azure Monitor][azure-mon].</span><span class="sxs-lookup"><span data-stu-id="068a8-157">Consider [Integrating with Azure Application Insights][azure-apim-ai], which also surfaces metrics through [Azure Monitor][azure-mon] for monitoring.</span></span>

## <a name="deploy-the-scenario"></a><span data-ttu-id="068a8-158">Развертывание сценария</span><span class="sxs-lookup"><span data-stu-id="068a8-158">Deploy the scenario</span></span>

<span data-ttu-id="068a8-159">Чтобы начать работу, [создайте в портале экземпляр управления API Azure.][apim-create]</span><span class="sxs-lookup"><span data-stu-id="068a8-159">To get started, [create an Azure API Management instance in the portal.][apim-create]</span></span>

<span data-ttu-id="068a8-160">Кроме того, в существующем Azure Resource Manager вы можете выбрать [шаблон быстрого запуска][azure-quickstart-templates-apim], который будет соответствовать вашему конкретному варианту использования.</span><span class="sxs-lookup"><span data-stu-id="068a8-160">Alternatively, you can choose from an existing Azure Resource Manager [quickstart template][azure-quickstart-templates-apim] that aligns to your specific use case.</span></span>

## <a name="pricing"></a><span data-ttu-id="068a8-161">Цены</span><span class="sxs-lookup"><span data-stu-id="068a8-161">Pricing</span></span>

<span data-ttu-id="068a8-162">Для службы управления API предлагаются четыре уровня: "Разработка", "Базовый", Стандартный" и "Премиум".</span><span class="sxs-lookup"><span data-stu-id="068a8-162">API Management is offered in four tiers: developer, basic, standard, and premium.</span></span> <span data-ttu-id="068a8-163">Подробное руководство по различиям этих уровней можно найти на странице [Цены на управление API.][apim-pricing]</span><span class="sxs-lookup"><span data-stu-id="068a8-163">You can find detailed guidance on the difference in these tiers at the [Azure API Management pricing guidance here.][apim-pricing]</span></span>

<span data-ttu-id="068a8-164">Клиенты могут масштабировать управление API путем добавления и удаления единиц.</span><span class="sxs-lookup"><span data-stu-id="068a8-164">Customers can scale API Management by adding and removing units.</span></span> <span data-ttu-id="068a8-165">Мощность единиц зависит от их уровня.</span><span class="sxs-lookup"><span data-stu-id="068a8-165">Each unit has capacity that depends on its tier.</span></span>

> [!NOTE]
> <span data-ttu-id="068a8-166">Уровень "Разработка" можно использовать для оценки компонентов управления API.</span><span class="sxs-lookup"><span data-stu-id="068a8-166">The Developer tier can be used for evaluation of the API Management features.</span></span> <span data-ttu-id="068a8-167">Уровень "Разработка" нельзя использовать для рабочей среды.</span><span class="sxs-lookup"><span data-stu-id="068a8-167">The Developer tier should not be used for production.</span></span>

<span data-ttu-id="068a8-168">Чтобы просмотреть прогнозируемые затраты и настроить потребности развертывания, вы можете изменить количество единиц масштабирования и экземпляров службы приложений в [Калькуляторе цен Azure][pricing-calculator].</span><span class="sxs-lookup"><span data-stu-id="068a8-168">To view projected costs and customize to your deployment needs, you can modify the number of scale units and App Service instances in the [Azure Pricing Calculator][pricing-calculator].</span></span>

## <a name="related-resources"></a><span data-ttu-id="068a8-169">Связанные ресурсы</span><span class="sxs-lookup"><span data-stu-id="068a8-169">Related resources</span></span>

<span data-ttu-id="068a8-170">См. обширную документацию по [управлению API Azure][apim]</span><span class="sxs-lookup"><span data-stu-id="068a8-170">Review the extensive Azure API Management [documentation and reference articles][apim].</span></span>

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
