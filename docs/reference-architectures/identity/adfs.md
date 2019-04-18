---
title: Развертывание локальных служб федерации Active Directory в Azure
titleSuffix: Azure Reference Architectures
description: Реализуйте защищенную гибридную сетевую архитектуру с помощью авторизации службы федерации Active Directory в Azure.
author: telmosampaio
ms.date: 12/18.2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18, identity
ms.openlocfilehash: 873b6a86da14e00d0a537f910d10922444cc1ded
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/17/2019
ms.locfileid: "59640742"
---
# <a name="extend-active-directory-federation-services-ad-fs-to-azure"></a><span data-ttu-id="282a7-103">Расширение служб федерации Active Directory (AD FS) в Azure</span><span class="sxs-lookup"><span data-stu-id="282a7-103">Extend Active Directory Federation Services (AD FS) to Azure</span></span>

<span data-ttu-id="282a7-104">Эта эталонная архитектура реализует безопасную гибридную сеть, которая расширяет вашу локальную сеть в Azure и использует [службы федерации Active Directory (AD FS)][active-directory-federation-services] для выполнения федеративной проверки подлинности и авторизации компонентов, работающих в Azure.</span><span class="sxs-lookup"><span data-stu-id="282a7-104">This reference architecture implements a secure hybrid network that extends your on-premises network to Azure and uses [Active Directory Federation Services (AD FS)][active-directory-federation-services] to perform federated authentication and authorization for components running in Azure.</span></span> <span data-ttu-id="282a7-105">[**Разверните это решение**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="282a7-105">[**Deploy this solution**](#deploy-the-solution).</span></span>

![Защищенная гибридная сетевая архитектура с Active Directory](./images/adfs.png)

<span data-ttu-id="282a7-107">*Скачайте [файл Visio][visio-download] этой архитектуры.*</span><span class="sxs-lookup"><span data-stu-id="282a7-107">*Download a [Visio file][visio-download] of this architecture.*</span></span>

<span data-ttu-id="282a7-108">AD FS может размещаться в локальной среде, но если ваше приложение является гибридным, в котором некоторые части реализованы в Azure, эффективнее реплицировать AD FS в облако.</span><span class="sxs-lookup"><span data-stu-id="282a7-108">AD FS can be hosted on-premises, but if your application is a hybrid in which some parts are implemented in Azure, it may be more efficient to replicate AD FS in the cloud.</span></span>

<span data-ttu-id="282a7-109">На схеме показаны следующие сценарии:</span><span class="sxs-lookup"><span data-stu-id="282a7-109">The diagram shows the following scenarios:</span></span>

- <span data-ttu-id="282a7-110">Код приложения организации-партнера обращается к веб-приложению, размещенному внутри виртуальной сети Azure.</span><span class="sxs-lookup"><span data-stu-id="282a7-110">Application code from a partner organization accesses a web application hosted inside your Azure VNet.</span></span>
- <span data-ttu-id="282a7-111">Внешний зарегистрированный пользователь с учетными данными, хранящимися в доменных службах Active Directory (DS), обращается к веб-приложению, размещенному внутри виртуальной сети Azure.</span><span class="sxs-lookup"><span data-stu-id="282a7-111">An external, registered user with credentials stored inside Active Directory Domain Services (DS) accesses a web application hosted inside your Azure VNet.</span></span>
- <span data-ttu-id="282a7-112">Пользователь, подключенный к виртуальной сети с помощью авторизованного устройства, запускает веб-приложение, размещенное внутри виртуальной сети Azure.</span><span class="sxs-lookup"><span data-stu-id="282a7-112">A user connected to your VNet using an authorized device executes a web application hosted inside your Azure VNet.</span></span>

<span data-ttu-id="282a7-113">Типичные способы использования этой архитектуры:</span><span class="sxs-lookup"><span data-stu-id="282a7-113">Typical uses for this architecture include:</span></span>

- <span data-ttu-id="282a7-114">Гибридные приложения, в которых рабочие нагрузки выполняются частично локально и частично в Azure.</span><span class="sxs-lookup"><span data-stu-id="282a7-114">Hybrid applications where workloads run partly on-premises and partly in Azure.</span></span>
- <span data-ttu-id="282a7-115">Решения, использующие федеративную авторизацию для публикации веб-приложений в партнерских организациях.</span><span class="sxs-lookup"><span data-stu-id="282a7-115">Solutions that use federated authorization to expose web applications to partner organizations.</span></span>
- <span data-ttu-id="282a7-116">Системы, которые поддерживают доступ из веб-браузеров за пределами брандмауэра организации.</span><span class="sxs-lookup"><span data-stu-id="282a7-116">Systems that support access from web browsers running outside of the organizational firewall.</span></span>
- <span data-ttu-id="282a7-117">Системы, которые позволяют пользователям получать доступ к веб-приложениям, подключаясь к авторизованным внешним устройствам, таким как удаленные компьютеры, ноутбуки и другие мобильные устройства.</span><span class="sxs-lookup"><span data-stu-id="282a7-117">Systems that enable users to access to web applications by connecting from authorized external devices such as remote computers, notebooks, and other mobile devices.</span></span>

<span data-ttu-id="282a7-118">В этой эталонной архитектуре основное внимание уделяется *пассивной федерации*, в которой серверы федерации определяют, как и когда выполнять проверку подлинности пользователя.</span><span class="sxs-lookup"><span data-stu-id="282a7-118">This reference architecture focuses on *passive federation*, in which the federation servers decide how and when to authenticate a user.</span></span> <span data-ttu-id="282a7-119">Пользователь предоставляет сведения для входа при запуске приложения.</span><span class="sxs-lookup"><span data-stu-id="282a7-119">The user provides sign in information when the application is started.</span></span> <span data-ttu-id="282a7-120">Этот механизм чаще всего используется веб-браузерами и включает в себя протокол, который перенаправляет браузер на сайт, на котором пользователь проходит проверку подлинности.</span><span class="sxs-lookup"><span data-stu-id="282a7-120">This mechanism is most commonly used by web browsers and involves a protocol that redirects the browser to a site where the user authenticates.</span></span> <span data-ttu-id="282a7-121">Службы федерации Active Directory поддерживают также *активную федерацию*, где приложение отвечает за предоставление учетных данных без дальнейшего участия пользователя, но этот сценарий выходит за рамки этой архитектуры.</span><span class="sxs-lookup"><span data-stu-id="282a7-121">AD FS also supports *active federation*, where an application takes on responsibility for supplying credentials without further user interaction, but that scenario is outside the scope of this architecture.</span></span>

<span data-ttu-id="282a7-122">Дополнительные рекомендации см. в статье [Выбор решения для интеграции локальной среды Active Directory с Azure][considerations].</span><span class="sxs-lookup"><span data-stu-id="282a7-122">For additional considerations, see [Choose a solution for integrating on-premises Active Directory with Azure][considerations].</span></span>

## <a name="architecture"></a><span data-ttu-id="282a7-123">Архитектура</span><span class="sxs-lookup"><span data-stu-id="282a7-123">Architecture</span></span>

<span data-ttu-id="282a7-124">Эта архитектура расширяет реализацию, описанную в статье [Расширение доменных служб Active Directory в Azure][extending-ad-to-azure].</span><span class="sxs-lookup"><span data-stu-id="282a7-124">This architecture extends the implementation described in [Extending AD DS to Azure][extending-ad-to-azure].</span></span> <span data-ttu-id="282a7-125">Она содержит следующие компоненты:</span><span class="sxs-lookup"><span data-stu-id="282a7-125">It contains the followign components.</span></span>

- <span data-ttu-id="282a7-126">**Подсеть AD DS.**</span><span class="sxs-lookup"><span data-stu-id="282a7-126">**AD DS subnet**.</span></span> <span data-ttu-id="282a7-127">Серверы AD DS содержатся в собственной подсети с правилами группы сетевой безопасности (NSG), действующими в качестве брандмауэра.</span><span class="sxs-lookup"><span data-stu-id="282a7-127">The AD DS servers are contained in their own subnet with network security group (NSG) rules acting as a firewall.</span></span>

- <span data-ttu-id="282a7-128">**Серверы AD DS.**</span><span class="sxs-lookup"><span data-stu-id="282a7-128">**AD DS servers**.</span></span> <span data-ttu-id="282a7-129">Контроллеры домена, выполняющие функции виртуальных машин в Azure.</span><span class="sxs-lookup"><span data-stu-id="282a7-129">Domain controllers running as VMs in Azure.</span></span> <span data-ttu-id="282a7-130">Эти серверы предоставляют проверку подлинности локальных удостоверений в домене.</span><span class="sxs-lookup"><span data-stu-id="282a7-130">These servers provide authentication of local identities within the domain.</span></span>

- <span data-ttu-id="282a7-131">**Подсеть AD FS.**</span><span class="sxs-lookup"><span data-stu-id="282a7-131">**AD FS subnet**.</span></span> <span data-ttu-id="282a7-132">Серверы AD FS расположены в своей собственной подсети с правилами NSG, действующими в качестве брандмауэра.</span><span class="sxs-lookup"><span data-stu-id="282a7-132">The AD FS servers are located within their own subnet with NSG rules acting as a firewall.</span></span>

- <span data-ttu-id="282a7-133">**Серверы AD FS.**</span><span class="sxs-lookup"><span data-stu-id="282a7-133">**AD FS servers**.</span></span> <span data-ttu-id="282a7-134">Серверы AD FS предоставляют федеративную авторизацию и проверку подлинности.</span><span class="sxs-lookup"><span data-stu-id="282a7-134">The AD FS servers provide federated authorization and authentication.</span></span> <span data-ttu-id="282a7-135">В этой архитектуре они выполняют следующие задачи:</span><span class="sxs-lookup"><span data-stu-id="282a7-135">In this architecture, they perform the following tasks:</span></span>

  - <span data-ttu-id="282a7-136">Получение маркеров безопасности, содержащих утверждения, сделанные сервером федерации партнера от имени пользователя партнера.</span><span class="sxs-lookup"><span data-stu-id="282a7-136">Receiving security tokens containing claims made by a partner federation server on behalf of a partner user.</span></span> <span data-ttu-id="282a7-137">Службы федерации Active Directory проверяют действительность токенов, прежде чем передавать утверждения в веб-приложение, запущенное в Azure, для авторизации запросов.</span><span class="sxs-lookup"><span data-stu-id="282a7-137">AD FS verifies that the tokens are valid before passing the claims to the web application running in Azure to authorize requests.</span></span>

    <span data-ttu-id="282a7-138">Приложение, выполняемое в Azure, является *проверяющей стороной*.</span><span class="sxs-lookup"><span data-stu-id="282a7-138">The application running in Azure is the *relying party*.</span></span> <span data-ttu-id="282a7-139">Сервер федерации партнера должен выдавать утверждения, которые распознаются веб-приложением.</span><span class="sxs-lookup"><span data-stu-id="282a7-139">The partner federation server must issue claims that are understood by the web application.</span></span> <span data-ttu-id="282a7-140">Серверы федерации партнера называются *партнерами по учетным записям*, так как они отправляют запросы на доступ от имени прошедших проверку подлинности учетных записей в организации партнера.</span><span class="sxs-lookup"><span data-stu-id="282a7-140">The partner federation servers are referred to as *account partners*, because they submit access requests on behalf of authenticated accounts in the partner organization.</span></span> <span data-ttu-id="282a7-141">Серверы AD FS называются *партнерами по ресурсам*, так как они обеспечивают доступ к ресурсам (веб-приложение).</span><span class="sxs-lookup"><span data-stu-id="282a7-141">The AD FS servers are called *resource partners* because they provide access to resources (the web application).</span></span>

  - <span data-ttu-id="282a7-142">Проверка подлинности и авторизация входящих запросов от внешних пользователей, работающих с веб-браузером или устройством, которым необходим доступ к веб-приложениям, с использованием доменных служб Active Directory и [службы регистрации устройств Active Directory][ADDRS].</span><span class="sxs-lookup"><span data-stu-id="282a7-142">Authenticating and authorizing incoming requests from external users running a web browser or device that needs access to web applications, by using AD DS and the [Active Directory Device Registration Service][ADDRS].</span></span>

  <span data-ttu-id="282a7-143">Серверы AD FS настроены как ферма, доступ к которой осуществляется через подсистему балансировки нагрузки Azure.</span><span class="sxs-lookup"><span data-stu-id="282a7-143">The AD FS servers are configured as a farm accessed through an Azure load balancer.</span></span> <span data-ttu-id="282a7-144">Эта реализация позволяет повысить доступность и масштабируемость.</span><span class="sxs-lookup"><span data-stu-id="282a7-144">This implementation improves availability and scalability.</span></span> <span data-ttu-id="282a7-145">Серверы AD FS недоступны напрямую в Интернете.</span><span class="sxs-lookup"><span data-stu-id="282a7-145">The AD FS servers are not exposed directly to the Internet.</span></span> <span data-ttu-id="282a7-146">Весь интернет-трафик фильтруется через прокси-серверы веб-приложений AD FS и промежуточную сеть (также называемую сетью периметра).</span><span class="sxs-lookup"><span data-stu-id="282a7-146">All Internet traffic is filtered through AD FS web application proxy servers and a DMZ (also referred to as a perimeter network).</span></span>

  <span data-ttu-id="282a7-147">Дополнительные сведения о работе служб федерации Active Directory см. в статье [Active Directory Federation Services Overview][active-directory-federation-services-overview] (Службы федерации Active Directory).</span><span class="sxs-lookup"><span data-stu-id="282a7-147">For more information about how AD FS works, see [Active Directory Federation Services Overview][active-directory-federation-services-overview].</span></span> <span data-ttu-id="282a7-148">Подробные пошаговые инструкции по реализации см. в статье [Развертывание служб федерации Active Directory в Azure][adfs-intro].</span><span class="sxs-lookup"><span data-stu-id="282a7-148">Also, the article [AD FS deployment in Azure][adfs-intro] contains a detailed step-by-step introduction to implementation.</span></span>

- <span data-ttu-id="282a7-149">**Подсеть прокси-сервера AD FS.**</span><span class="sxs-lookup"><span data-stu-id="282a7-149">**AD FS proxy subnet**.</span></span> <span data-ttu-id="282a7-150">Прокси-серверы AD FS могут содержаться в собственной подсети с правилами NSG, обеспечивающими защиту.</span><span class="sxs-lookup"><span data-stu-id="282a7-150">The AD FS proxy servers can be contained within their own subnet, with NSG rules providing protection.</span></span> <span data-ttu-id="282a7-151">Серверы этой подсети доступны в Интернете через набор сетевых виртуальных устройств, которые обеспечивают брандмауэр между вашей виртуальной сетью Azure и Интернетом.</span><span class="sxs-lookup"><span data-stu-id="282a7-151">The servers in this subnet are exposed to the Internet through a set of network virtual appliances that provide a firewall between your Azure virtual network and the Internet.</span></span>

- <span data-ttu-id="282a7-152">**Прокси-серверы веб-приложения (WAP) AD FS.**</span><span class="sxs-lookup"><span data-stu-id="282a7-152">**AD FS web application proxy (WAP) servers**.</span></span> <span data-ttu-id="282a7-153">Эти виртуальные машины действуют как серверы AD FS для входящих запросов от партнерских организаций и внешних устройств.</span><span class="sxs-lookup"><span data-stu-id="282a7-153">These VMs act as AD FS servers for incoming requests from partner organizations and external devices.</span></span> <span data-ttu-id="282a7-154">WAP-серверы действуют как фильтр, защищая серверы AD FS от прямого доступа из Интернета.</span><span class="sxs-lookup"><span data-stu-id="282a7-154">The WAP servers act as a filter, shielding the AD FS servers from direct access from the Internet.</span></span> <span data-ttu-id="282a7-155">Как и для серверов AD FS, развертывание WAP-серверов в ферме с подсистемой балансировки нагрузки обеспечивает большую доступность и масштабируемость, чем развертывание коллекции автономных серверов.</span><span class="sxs-lookup"><span data-stu-id="282a7-155">As with the AD FS servers, deploying the WAP servers in a farm with load balancing gives you greater availability and scalability than deploying a collection of stand-alone servers.</span></span>

  > [!NOTE]
  > <span data-ttu-id="282a7-156">Подробные сведения об установке WAP-серверов см. в статье [Установка и настройка прокси-сервера веб-приложений][install_and_configure_the_web_application_proxy_server].</span><span class="sxs-lookup"><span data-stu-id="282a7-156">For detailed information about installing WAP servers, see [Install and Configure the Web Application Proxy Server][install_and_configure_the_web_application_proxy_server]</span></span>
  >

- <span data-ttu-id="282a7-157">**Партнерская организация.**</span><span class="sxs-lookup"><span data-stu-id="282a7-157">**Partner organization**.</span></span> <span data-ttu-id="282a7-158">Партнерская организация, запускающая веб-приложение, которое запрашивает доступ к веб-приложению, запущенному в Azure.</span><span class="sxs-lookup"><span data-stu-id="282a7-158">A partner organization running a web application that requests access to a web application running in Azure.</span></span> <span data-ttu-id="282a7-159">Сервер федерации в партнерской организации проверяет подлинность запросов локально и отправляет маркеры безопасности, содержащие утверждения, в AD FS, работающие в Azure.</span><span class="sxs-lookup"><span data-stu-id="282a7-159">The federation server at the partner organization authenticates requests locally, and submits security tokens containing claims to AD FS running in Azure.</span></span> <span data-ttu-id="282a7-160">Службы AD FS в Azure проверяют маркеры безопасности, и если они действительны, передают утверждения в веб-приложение, запущенное в Azure, для их авторизации.</span><span class="sxs-lookup"><span data-stu-id="282a7-160">AD FS in Azure validates the security tokens, and if valid can pass the claims to the web application running in Azure to authorize them.</span></span>

  > [!NOTE]
  > <span data-ttu-id="282a7-161">Вы также можете настроить VPN-туннель с помощью шлюза Azure для предоставления прямого доступа к AD FS доверенным партнерам.</span><span class="sxs-lookup"><span data-stu-id="282a7-161">You can also configure a VPN tunnel using Azure gateway to provide direct access to AD FS for trusted partners.</span></span> <span data-ttu-id="282a7-162">Запросы, полученные от этих партнеров, не проходят через WAP-серверы.</span><span class="sxs-lookup"><span data-stu-id="282a7-162">Requests received from these partners do not pass through the WAP servers.</span></span>
  >

## <a name="recommendations"></a><span data-ttu-id="282a7-163">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="282a7-163">Recommendations</span></span>

<span data-ttu-id="282a7-164">Следующие рекомендации применимы для большинства ситуаций.</span><span class="sxs-lookup"><span data-stu-id="282a7-164">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="282a7-165">Следуйте этим рекомендациям, если они не противоречат особым требованиям для вашего случая.</span><span class="sxs-lookup"><span data-stu-id="282a7-165">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="networking-recommendations"></a><span data-ttu-id="282a7-166">Рекомендации по сети</span><span class="sxs-lookup"><span data-stu-id="282a7-166">Networking recommendations</span></span>

<span data-ttu-id="282a7-167">Настройте сетевой интерфейс для каждой виртуальной машины, на которой размещены AD FS и WAP-серверы со статическими частными IP-адресами.</span><span class="sxs-lookup"><span data-stu-id="282a7-167">Configure the network interface for each of the VMs hosting AD FS and WAP servers with static private IP addresses.</span></span>

<span data-ttu-id="282a7-168">Не предоставляйте общедоступные IP-адреса виртуальным машинам AD FS.</span><span class="sxs-lookup"><span data-stu-id="282a7-168">Do not give the AD FS VMs public IP addresses.</span></span> <span data-ttu-id="282a7-169">Дополнительные сведения см. в разделе [Вопросы безопасности](#security-considerations).</span><span class="sxs-lookup"><span data-stu-id="282a7-169">For more information, see the [Security considerations](#security-considerations) section.</span></span>

<span data-ttu-id="282a7-170">Задайте IP-адрес основных и вторичных серверов служб доменных имен (DNS) для сетевых интерфейсов каждой виртуальной машины AD FS и WAP, чтобы указать виртуальные машины AD DS.</span><span class="sxs-lookup"><span data-stu-id="282a7-170">Set the IP address of the preferred and secondary domain name service (DNS) servers for the network interfaces for each AD FS and WAP VM to reference the Active Directory DS VMs.</span></span> <span data-ttu-id="282a7-171">На виртуальных машинах AD DS должен быть запущен DNS.</span><span class="sxs-lookup"><span data-stu-id="282a7-171">The Active Directory DS VMs should be running DNS.</span></span> <span data-ttu-id="282a7-172">Этот шаг является обязательным для подключения каждой виртуальной машины к домену.</span><span class="sxs-lookup"><span data-stu-id="282a7-172">This step is necessary to enable each VM to join the domain.</span></span>

### <a name="ad-fs-installation"></a><span data-ttu-id="282a7-173">Установка AD FS</span><span class="sxs-lookup"><span data-stu-id="282a7-173">AD FS installation</span></span>

<span data-ttu-id="282a7-174">В статье [Развертывание служб федерации Active Directory в Azure][Deploying_a_federation_server_farm] приведены подробные инструкции по установке и настройке служб федерации Active Directory.</span><span class="sxs-lookup"><span data-stu-id="282a7-174">The article [Deploying a Federation Server Farm][Deploying_a_federation_server_farm] provides detailed instructions for installing and configuring AD FS.</span></span> <span data-ttu-id="282a7-175">Выполните следующие задачи перед началом настройки первого сервера AD FS в ферме:</span><span class="sxs-lookup"><span data-stu-id="282a7-175">Perform the following tasks before configuring the first AD FS server in the farm:</span></span>

1. <span data-ttu-id="282a7-176">Получите общедоступный доверенный сертификат для проверки подлинности сервера.</span><span class="sxs-lookup"><span data-stu-id="282a7-176">Obtain a publicly trusted certificate for performing server authentication.</span></span> <span data-ttu-id="282a7-177">*Имя субъекта* должно содержать имя, используемое клиентами для доступа к службе федерации.</span><span class="sxs-lookup"><span data-stu-id="282a7-177">The *subject name* must contain the name clients use to access the federation service.</span></span> <span data-ttu-id="282a7-178">Это может быть DNS-имя, зарегистрированное для подсистемы балансировки нагрузки, например *adfs.contoso.com* (из соображений безопасности не используйте подстановочные имена, такие как \**.contoso.com* ).</span><span class="sxs-lookup"><span data-stu-id="282a7-178">This can be the DNS name registered for the load balancer, for example, *adfs.contoso.com* (avoid using wildcard names such as \**.contoso.com*, for security reasons).</span></span> <span data-ttu-id="282a7-179">Используйте один сертификат на всех виртуальных машинах сервера AD FS.</span><span class="sxs-lookup"><span data-stu-id="282a7-179">Use the same certificate on all AD FS server VMs.</span></span> <span data-ttu-id="282a7-180">Вы можете приобрести сертификат в доверенном центре сертификации, но если ваша организация использует службы сертификатов Active Directory, вы можете создать собственный.</span><span class="sxs-lookup"><span data-stu-id="282a7-180">You can purchase a certificate from a trusted certification authority, but if your organization uses Active Directory Certificate Services you can create your own.</span></span>

    <span data-ttu-id="282a7-181">*Альтернативное имя субъекта* используется службой регистрации устройств (DRS) для разрешения доступа с внешних устройств.</span><span class="sxs-lookup"><span data-stu-id="282a7-181">The *subject alternative name* is used by the device registration service (DRS) to enable access from external devices.</span></span> <span data-ttu-id="282a7-182">Оно должно указываться в формате *enterpriseregistration.contoso.com*.</span><span class="sxs-lookup"><span data-stu-id="282a7-182">This should be of the form *enterpriseregistration.contoso.com*.</span></span>

    <span data-ttu-id="282a7-183">Дополнительные сведения см. в статье [Managing SSL Certificates in AD FS and WAP in Windows Server 2016][adfs_certificates] (Управление SSL-сертификатами в AD FS и WAP в Windows Server 2016).</span><span class="sxs-lookup"><span data-stu-id="282a7-183">For more information, see [Obtain and Configure a Secure Sockets Layer (SSL) Certificate for AD FS][adfs_certificates].</span></span>

2. <span data-ttu-id="282a7-184">На контроллере домена создайте корневой ключ для службы распределения ключей.</span><span class="sxs-lookup"><span data-stu-id="282a7-184">On the domain controller, generate a new root key for the Key Distribution Service.</span></span> <span data-ttu-id="282a7-185">Установите срок действия на текущее время минус 10 часов (эта конфигурация уменьшает задержку, которая может возникать при распространении и синхронизации ключей по всему домену).</span><span class="sxs-lookup"><span data-stu-id="282a7-185">Set the effective time to the current time minus 10 hours (this configuration reduces the delay that can occur in distributing and synchronizing keys across the domain).</span></span> <span data-ttu-id="282a7-186">Этот шаг необходим для поддержки создания групповой учетной записи службы, используемой для запуска службы AD FS.</span><span class="sxs-lookup"><span data-stu-id="282a7-186">This step is necessary to support creating the group service account that is used to run the AD FS service.</span></span> <span data-ttu-id="282a7-187">В следующей команде PowerShell показан пример того, как это сделать:</span><span class="sxs-lookup"><span data-stu-id="282a7-187">The following PowerShell command shows an example of how to do this:</span></span>

    ```powershell
    Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
    ```

3. <span data-ttu-id="282a7-188">Добавьте все виртуальные машины сервера AD FS в домен.</span><span class="sxs-lookup"><span data-stu-id="282a7-188">Add each AD FS server VM to the domain.</span></span>

> [!NOTE]
> <span data-ttu-id="282a7-189">Чтобы установить AD FS, контроллер домена, на котором выполняется роль FSMO эмулятора основного контроллера домена, должен быть запущен и доступен из виртуальных машин AD FS.</span><span class="sxs-lookup"><span data-stu-id="282a7-189">To install AD FS, the domain controller running the primary domain controller (PDC) emulator flexible single master operation (FSMO) role for the domain must be running and accessible from the AD FS VMs.</span></span> <span data-ttu-id="282a7-190"><<RBC: можно ли сократить повторяемость?>></span><span class="sxs-lookup"><span data-stu-id="282a7-190"><<RBC: Is there a way to make this less repetitive?>></span></span>
>

### <a name="ad-fs-trust"></a><span data-ttu-id="282a7-191">Доверие AD FS</span><span class="sxs-lookup"><span data-stu-id="282a7-191">AD FS trust</span></span>

<span data-ttu-id="282a7-192">Установите доверие федерации между установкой AD FS и серверами федерации любых партнерских организаций.</span><span class="sxs-lookup"><span data-stu-id="282a7-192">Establish federation trust between your AD FS installation, and the federation servers of any partner organizations.</span></span> <span data-ttu-id="282a7-193">Настройте требуемую фильтрацию и сопоставление утверждений.</span><span class="sxs-lookup"><span data-stu-id="282a7-193">Configure any claims filtering and mapping required.</span></span>

- <span data-ttu-id="282a7-194">Сотрудники отдела DevOps в каждой партнерской организации должны добавить доверие проверяющей стороны для веб-приложений, доступных через ваши серверы AD FS.</span><span class="sxs-lookup"><span data-stu-id="282a7-194">DevOps staff at each partner organization must add a relying party trust for the web applications accessible through your AD FS servers.</span></span>
- <span data-ttu-id="282a7-195">Сотрудники отдела DevOps в вашей организации должны настроить доверие поставщика утверждений, чтобы ваши серверы AD FS могли доверять утверждениям, предоставляемым партнерскими организациями,</span><span class="sxs-lookup"><span data-stu-id="282a7-195">DevOps staff in your organization must configure claims-provider trust to enable your AD FS servers to trust the claims that partner organizations provide.</span></span>
- <span data-ttu-id="282a7-196">а также настроить в AD FS передачу утверждений в веб-приложения вашей организации.</span><span class="sxs-lookup"><span data-stu-id="282a7-196">DevOps staff in your organization must also configure AD FS to pass claims on to your organization's web applications.</span></span>

<span data-ttu-id="282a7-197">Дополнительные сведения см. в статье [Establishing Federation Trust][establishing-federation-trust] (Настройка доверия федерации).</span><span class="sxs-lookup"><span data-stu-id="282a7-197">For more information, see [Establishing Federation Trust][establishing-federation-trust].</span></span>

<span data-ttu-id="282a7-198">Публикуйте веб-приложения своей организации и предоставляйте их внешним партнерам, используя предварительную проверку подлинности через WAP-серверы.</span><span class="sxs-lookup"><span data-stu-id="282a7-198">Publish your organization's web applications and make them available to external partners by using preauthentication through the WAP servers.</span></span> <span data-ttu-id="282a7-199">Дополнительные сведения см. в статье [Публикация приложений с использованием предварительной проверки подлинности AD FS][publish_applications_using_AD_FS_preauthentication].</span><span class="sxs-lookup"><span data-stu-id="282a7-199">For more information, see [Publish Applications using AD FS Preauthentication][publish_applications_using_AD_FS_preauthentication]</span></span>

<span data-ttu-id="282a7-200">Службы федерации Active Directory поддерживают преобразования токенов и приращение.</span><span class="sxs-lookup"><span data-stu-id="282a7-200">AD FS supports token transformation and augmentation.</span></span> <span data-ttu-id="282a7-201">Azure Active Directory не поддерживает эту функцию.</span><span class="sxs-lookup"><span data-stu-id="282a7-201">Azure Active Directory does not provide this feature.</span></span> <span data-ttu-id="282a7-202">В AD FS при настройке отношения доверия вы можете:</span><span class="sxs-lookup"><span data-stu-id="282a7-202">With AD FS, when you set up the trust relationships, you can:</span></span>

- <span data-ttu-id="282a7-203">Настраивать преобразования утверждений для правил авторизации.</span><span class="sxs-lookup"><span data-stu-id="282a7-203">Configure claim transformations for authorization rules.</span></span> <span data-ttu-id="282a7-204">Например можно сопоставить безопасность группы из представления, используемого сторонней партнерской организацией, в то, что Active Directory DS может авторизовать в вашей организации.</span><span class="sxs-lookup"><span data-stu-id="282a7-204">For example, you can map group security from a representation used by a non-Microsoft partner organization to something that Active Directory DS can authorize in your organization.</span></span>
- <span data-ttu-id="282a7-205">Преобразовывать утверждения из одного формата в другой.</span><span class="sxs-lookup"><span data-stu-id="282a7-205">Transform claims from one format to another.</span></span> <span data-ttu-id="282a7-206">Например, вы можете сопоставить SAML 2.0 с SAML 1.1, если ваше приложение поддерживает только утверждения SAML 1.1.</span><span class="sxs-lookup"><span data-stu-id="282a7-206">For example, you can map from SAML 2.0 to SAML 1.1 if your application only supports SAML 1.1 claims.</span></span>

### <a name="ad-fs-monitoring"></a><span data-ttu-id="282a7-207">Мониторинг AD FS</span><span class="sxs-lookup"><span data-stu-id="282a7-207">AD FS monitoring</span></span>

<span data-ttu-id="282a7-208">[Пакет управления Microsoft System Center для AD FS 2012 R2][oms-adfs-pack] предоставляет средства активного и реактивного мониторинга развертывания сервера федерации AD FS.</span><span class="sxs-lookup"><span data-stu-id="282a7-208">The [Microsoft System Center Management Pack for Active Directory Federation Services 2012 R2][oms-adfs-pack] provides both proactive and reactive monitoring of your AD FS deployment for the federation server.</span></span> <span data-ttu-id="282a7-209">Этот пакет управления отслеживает:</span><span class="sxs-lookup"><span data-stu-id="282a7-209">This management pack monitors:</span></span>

- <span data-ttu-id="282a7-210">События, которые служба AD FS записывает в свои журналы событий.</span><span class="sxs-lookup"><span data-stu-id="282a7-210">Events that the AD FS service records in its event logs.</span></span>
- <span data-ttu-id="282a7-211">Данные о производительности, которые собирают счетчики производительности AD FS.</span><span class="sxs-lookup"><span data-stu-id="282a7-211">The performance data that the AD FS performance counters collect.</span></span>
- <span data-ttu-id="282a7-212">Общую работоспособность системы AD FS и веб-приложений (проверяющих сторон), а также оповещения о критических проблемах и предупреждения.</span><span class="sxs-lookup"><span data-stu-id="282a7-212">The overall health of the AD FS system and web applications (relying parties), and provides alerts for critical issues and warnings.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="282a7-213">Вопросы масштабируемости</span><span class="sxs-lookup"><span data-stu-id="282a7-213">Scalability considerations</span></span>

<span data-ttu-id="282a7-214">Следующие аспекты, изложенные в статье [Планирование развертывания служб федерации Active Directory][plan-your-adfs-deployment], позволяют приступить к изменению размера ферм AD FS:</span><span class="sxs-lookup"><span data-stu-id="282a7-214">The following considerations, summarized from the article [Plan your AD FS deployment][plan-your-adfs-deployment], give a starting point for sizing AD FS farms:</span></span>

- <span data-ttu-id="282a7-215">Если у вас менее 1000 пользователей, не создавайте выделенные серверы, а вместо этого установите AD FS на каждом из серверов Active Directory DS в облаке.</span><span class="sxs-lookup"><span data-stu-id="282a7-215">If you have fewer than 1000 users, do not create dedicated servers, but instead install AD FS on each of the Active Directory DS servers in the cloud.</span></span> <span data-ttu-id="282a7-216">Убедитесь, что у вас есть как минимум два сервера AD DS для обеспечения доступности.</span><span class="sxs-lookup"><span data-stu-id="282a7-216">Make sure that you have at least two Active Directory DS servers to maintain availability.</span></span> <span data-ttu-id="282a7-217">Создайте один сервер WAP.</span><span class="sxs-lookup"><span data-stu-id="282a7-217">Create a single WAP server.</span></span>
- <span data-ttu-id="282a7-218">Если у вас от 1000 до 15 000 пользователей, создайте два выделенных сервера AD FS и два выделенных сервера WAP.</span><span class="sxs-lookup"><span data-stu-id="282a7-218">If you have between 1000 and 15000 users, create two dedicated AD FS servers and two dedicated WAP servers.</span></span>
- <span data-ttu-id="282a7-219">Если у вас от 15 000 до 60 000 пользователей, создайте от трех до пяти выделенных серверов AD FS и не менее двух выделенных WAP-серверов.</span><span class="sxs-lookup"><span data-stu-id="282a7-219">If you have between 15000 and 60000 users, create between three and five dedicated AD FS servers and at least two dedicated WAP servers.</span></span>

<span data-ttu-id="282a7-220">Предполагается, что вы используете две четырехъядерные виртуальные машины (Standard D4_v2 или лучше) в Azure.</span><span class="sxs-lookup"><span data-stu-id="282a7-220">These considerations assume that you are using dual quad-core VM (Standard D4_v2, or better) sizes in Azure.</span></span>

<span data-ttu-id="282a7-221">Если вы используете внутреннюю базу данных Windows для хранения данных конфигурации AD FS, вы ограничены восемью серверами AD FS в ферме.</span><span class="sxs-lookup"><span data-stu-id="282a7-221">If you are using the Windows Internal Database to store AD FS configuration data, you are limited to eight AD FS servers in the farm.</span></span> <span data-ttu-id="282a7-222">Если предполагается, что в будущем вам понадобится больше, используйте SQL Server.</span><span class="sxs-lookup"><span data-stu-id="282a7-222">If you anticipate that you will need more in the future, use SQL Server.</span></span> <span data-ttu-id="282a7-223">Дополнительные сведения см. в статье [The Role of the AD FS Configuration Database][adfs-configuration-database] (Роль базы данных конфигурации AD FS).</span><span class="sxs-lookup"><span data-stu-id="282a7-223">For more information, see [The Role of the AD FS Configuration Database][adfs-configuration-database].</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="282a7-224">Вопросы доступности</span><span class="sxs-lookup"><span data-stu-id="282a7-224">Availability considerations</span></span>

<span data-ttu-id="282a7-225">Создайте ферму AD FS с как минимум двумя серверами, чтобы увеличить доступность службы.</span><span class="sxs-lookup"><span data-stu-id="282a7-225">Create an AD FS farm with at least two servers to increase availability of the service.</span></span> <span data-ttu-id="282a7-226">Используйте разные учетные записи для каждой виртуальной машины AD FS в ферме.</span><span class="sxs-lookup"><span data-stu-id="282a7-226">Use different storage accounts for each AD FS VM in the farm.</span></span> <span data-ttu-id="282a7-227">Такой подход помогает гарантировать, что сбой в одной учетной записи хранения не сделает недоступной всю ферму.</span><span class="sxs-lookup"><span data-stu-id="282a7-227">This approach helps to ensure that a failure in a single storage account does not make the entire farm inaccessible.</span></span>

<span data-ttu-id="282a7-228">Создайте отдельные группы доступности Azure для виртуальных машин AD FS и WAP.</span><span class="sxs-lookup"><span data-stu-id="282a7-228">Create separate Azure availability sets for the AD FS and WAP VMs.</span></span> <span data-ttu-id="282a7-229">Убедитесь, что в каждой группе есть по крайней мере две виртуальные машины.</span><span class="sxs-lookup"><span data-stu-id="282a7-229">Ensure that there are at least two VMs in each set.</span></span> <span data-ttu-id="282a7-230">Для каждой группы доступности нужно настроить по крайней мере два домена обновления и два домена сбоя.</span><span class="sxs-lookup"><span data-stu-id="282a7-230">Each availability set must have at least two update domains and two fault domains.</span></span>

<span data-ttu-id="282a7-231">Настройте подсистемы балансировки нагрузки для виртуальных машин AD FS и WAP следующим образом:</span><span class="sxs-lookup"><span data-stu-id="282a7-231">Configure the load balancers for the AD FS VMs and WAP VMs as follows:</span></span>

- <span data-ttu-id="282a7-232">Используйте подсистему балансировки нагрузки Azure для обеспечения внешнего доступа к виртуальным машинам WAP и внутреннюю систему балансировки нагрузки для распределения нагрузки между серверами AD FS в ферме.</span><span class="sxs-lookup"><span data-stu-id="282a7-232">Use an Azure load balancer to provide external access to the WAP VMs, and an internal load balancer to distribute the load across the AD FS servers in the farm.</span></span>
- <span data-ttu-id="282a7-233">Передавайте на серверы AD FS и WAP только трафик с порта 443 (HTTPS).</span><span class="sxs-lookup"><span data-stu-id="282a7-233">Only pass traffic appearing on port 443 (HTTPS) to the AD FS/WAP servers.</span></span>
- <span data-ttu-id="282a7-234">Предоставьте статический IP-адрес подсистеме балансировки нагрузки.</span><span class="sxs-lookup"><span data-stu-id="282a7-234">Give the load balancer a static IP address.</span></span>
- <span data-ttu-id="282a7-235">Создайте проверку работоспособности для `/adfs/probe` с помощью HTTP.</span><span class="sxs-lookup"><span data-stu-id="282a7-235">Create a health probe using HTTP against `/adfs/probe`.</span></span> <span data-ttu-id="282a7-236">Дополнительные сведения см. в статье [Hardware Load Balancer Health Checks and Web Application Proxy / AD FS 2012 R2](https://blogs.technet.microsoft.com/applicationproxyblog/2014/10/17/hardware-load-balancer-health-checks-and-web-application-proxy-ad-fs-2012-r2/) (Проверки работоспособности аппаратной подсистемы балансировки нагрузки и прокси-службы веб-приложения или AD FS 2012 R2).</span><span class="sxs-lookup"><span data-stu-id="282a7-236">For more information, see [Hardware Load Balancer Health Checks and Web Application Proxy / AD FS 2012 R2](https://blogs.technet.microsoft.com/applicationproxyblog/2014/10/17/hardware-load-balancer-health-checks-and-web-application-proxy-ad-fs-2012-r2/).</span></span>

  > [!NOTE]
  > <span data-ttu-id="282a7-237">Серверы AD FS используют протокол указания имени сервера (SNI), поэтому попытка проверки с использованием конечной точки HTTPS из подсистемы балансировки нагрузки не выполняется.</span><span class="sxs-lookup"><span data-stu-id="282a7-237">AD FS servers use the Server Name Indication (SNI) protocol, so attempting to probe using an HTTPS endpoint from the load balancer fails.</span></span>
  >

- <span data-ttu-id="282a7-238">Добавьте запись *A* DNS в домен для подсистемы балансировки нагрузки AD FS.</span><span class="sxs-lookup"><span data-stu-id="282a7-238">Add a DNS *A* record to the domain for the AD FS load balancer.</span></span> <span data-ttu-id="282a7-239">Укажите IP-адрес подсистемы балансировки нагрузки и присвойте ей имя домена (например, adfs.contoso.com).</span><span class="sxs-lookup"><span data-stu-id="282a7-239">Specify the IP address of the load balancer, and give it a name in the domain (such as adfs.contoso.com).</span></span> <span data-ttu-id="282a7-240">Это имя, которое клиенты и серверы WAP используют для доступа к ферме серверов AD FS.</span><span class="sxs-lookup"><span data-stu-id="282a7-240">This is the name clients and the WAP servers use to access the AD FS server farm.</span></span>

<span data-ttu-id="282a7-241">Вы можете использовать SQL Server или внутреннюю базу данных Windows для хранения данных о конфигурации AD FS.</span><span class="sxs-lookup"><span data-stu-id="282a7-241">You can use either SQL Server or the Windows Internal Database to hold AD FS configuration information.</span></span> <span data-ttu-id="282a7-242">Внутренняя база данных Windows обеспечивает основную избыточность.</span><span class="sxs-lookup"><span data-stu-id="282a7-242">The Windows Internal Database provides basic redundancy.</span></span> <span data-ttu-id="282a7-243">Изменения записываются непосредственно только в одну из баз данных AD FS в кластере AD FS, в то время как другие серверы используют репликацию по запросу, чтобы обновлять свои базы данных.</span><span class="sxs-lookup"><span data-stu-id="282a7-243">Changes are written directly to only one of the AD FS databases in the AD FS cluster, while the other servers use pull replication to keep their databases up to date.</span></span> <span data-ttu-id="282a7-244">Использование SQL Server обеспечивает избыточность и высокую доступность всей базы данных за счет отказоустойчивой кластеризации или зеркального отображения.</span><span class="sxs-lookup"><span data-stu-id="282a7-244">Using SQL Server can provide full database redundancy and high availability using failover clustering or mirroring.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="282a7-245">Вопросы управляемости</span><span class="sxs-lookup"><span data-stu-id="282a7-245">Manageability considerations</span></span>

<span data-ttu-id="282a7-246">Сотрудники DevOps должны быть готовы выполнить следующие задачи:</span><span class="sxs-lookup"><span data-stu-id="282a7-246">DevOps staff should be prepared to perform the following tasks:</span></span>

- <span data-ttu-id="282a7-247">Управление серверами федерации, включая управление фермой AD FS, управление политикой доверия на серверах федерации и управление сертификатами, используемыми службами федерации.</span><span class="sxs-lookup"><span data-stu-id="282a7-247">Managing the federation servers, including managing the AD FS farm, managing trust policy on the federation servers, and managing the certificates used by the federation services.</span></span>
- <span data-ttu-id="282a7-248">Управление WAP-серверами, включая управление фермой и сертификатами WAP.</span><span class="sxs-lookup"><span data-stu-id="282a7-248">Managing the WAP servers including managing the WAP farm and certificates.</span></span>
- <span data-ttu-id="282a7-249">Управление веб-приложениями, включая настройку проверяющих сторон, методы проверки подлинности и сопоставления утверждений.</span><span class="sxs-lookup"><span data-stu-id="282a7-249">Managing web applications including configuring relying parties, authentication methods, and claims mappings.</span></span>
- <span data-ttu-id="282a7-250">Резервное копирование компонентов AD FS.</span><span class="sxs-lookup"><span data-stu-id="282a7-250">Backing up AD FS components.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="282a7-251">Вопросы безопасности</span><span class="sxs-lookup"><span data-stu-id="282a7-251">Security considerations</span></span>

<span data-ttu-id="282a7-252">AD FS используют протокол HTTPS, поэтому убедитесь, что правила NSG для подсети, содержащей виртуальные машины веб-уровня, разрешают HTTPS-запросы.</span><span class="sxs-lookup"><span data-stu-id="282a7-252">AD FS uses HTTPS, so make sure that the NSG rules for the subnet containing the web tier VMs permit HTTPS requests.</span></span> <span data-ttu-id="282a7-253">Эти запросы могут быть получены из локальной сети, подсетей, содержащих веб-уровень, бизнес-уровень, уровень данных, частную и общедоступную сеть периметра, а также подсеть, содержащую серверы AD FS.</span><span class="sxs-lookup"><span data-stu-id="282a7-253">These requests can originate from the on-premises network, the subnets containing the web tier, business tier, data tier, private DMZ, public DMZ, and the subnet containing the AD FS servers.</span></span>

<span data-ttu-id="282a7-254">Предотвратите прямой доступ серверов AD FS к Интернету.</span><span class="sxs-lookup"><span data-stu-id="282a7-254">Prevent direct exposure of the AD FS servers to the Internet.</span></span> <span data-ttu-id="282a7-255">Серверы AD FS — это компьютеры, присоединенные к доменам, которые имеют полную авторизацию для предоставления маркеров безопасности.</span><span class="sxs-lookup"><span data-stu-id="282a7-255">AD FS servers are domain-joined computers that have full authorization to grant security tokens.</span></span> <span data-ttu-id="282a7-256">Если сервер взломан, злоумышленник может выдавать маркеры доступа всем веб-приложениям и всем серверам федерации, которые защищены AD FS.</span><span class="sxs-lookup"><span data-stu-id="282a7-256">If a server is compromised, a malicious user can issue full access tokens to all web applications and to all federation servers that are protected by AD FS.</span></span> <span data-ttu-id="282a7-257">Если ваша система должна обрабатывать запросы от внешних пользователей, не подключающихся с сайтов доверенных партнеров, используйте WAP-серверы для обработки этих запросов.</span><span class="sxs-lookup"><span data-stu-id="282a7-257">If your system must handle requests from external users not connecting from trusted partner sites, use WAP servers to handle these requests.</span></span> <span data-ttu-id="282a7-258">Дополнительные сведения см. в статье [Where to Place a Federation Server Proxy][where-to-place-an-fs-proxy] (Место размещения прокси-сервера федерации).</span><span class="sxs-lookup"><span data-stu-id="282a7-258">For more information, see [Where to Place a Federation Server Proxy][where-to-place-an-fs-proxy].</span></span>

<span data-ttu-id="282a7-259">Поместите серверы AD FS и WAP-серверы в отдельные подсети со своими брандмауэрами.</span><span class="sxs-lookup"><span data-stu-id="282a7-259">Place AD FS servers and WAP servers in separate subnets with their own firewalls.</span></span> <span data-ttu-id="282a7-260">Для определения правил брандмауэра можно использовать правила NSG.</span><span class="sxs-lookup"><span data-stu-id="282a7-260">You can use NSG rules to define firewall rules.</span></span> <span data-ttu-id="282a7-261">Все брандмауэры должны пропускать трафик через порт 443 (HTTPS).</span><span class="sxs-lookup"><span data-stu-id="282a7-261">All firewalls should allow traffic on port 443 (HTTPS).</span></span>

<span data-ttu-id="282a7-262">Ограничьте прямой вход на серверы AD FS и WAP.</span><span class="sxs-lookup"><span data-stu-id="282a7-262">Restrict direct sign in access to the AD FS and WAP servers.</span></span> <span data-ttu-id="282a7-263">Только сотрудники DevOps должны иметь возможность подключения.</span><span class="sxs-lookup"><span data-stu-id="282a7-263">Only DevOps staff should be able to connect.</span></span> <span data-ttu-id="282a7-264">Не присоединяйте серверы WAP к домену.</span><span class="sxs-lookup"><span data-stu-id="282a7-264">Do not join the WAP servers to the domain.</span></span>

<span data-ttu-id="282a7-265">Для выполнения аудита рассмотрите возможность использования набора сетевых виртуальных устройств, которые регистрируют подробные сведения о трафике, пересекающем край вашей виртуальной сети.</span><span class="sxs-lookup"><span data-stu-id="282a7-265">Consider using a set of network virtual appliances that logs detailed information on traffic traversing the edge of your virtual network for auditing purposes.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="282a7-266">Развертывание решения</span><span class="sxs-lookup"><span data-stu-id="282a7-266">Deploy the solution</span></span>

<span data-ttu-id="282a7-267">Пример развертывания для этой архитектуры можно найти на портале [GitHub][github].</span><span class="sxs-lookup"><span data-stu-id="282a7-267">A deployment for this architecture is available on [GitHub][github].</span></span> <span data-ttu-id="282a7-268">Обратите внимание, что для полного развертывания может потребоваться до двух часов, включая создание VPN-шлюза и запуск скриптов, которые настраивают доменные службы Active Directory и AD FS.</span><span class="sxs-lookup"><span data-stu-id="282a7-268">Note that the entire deployment can take up to two hours, which includes creating the VPN gateway and running the scripts that configure Active Directory and AD FS.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="282a7-269">Технические условия</span><span class="sxs-lookup"><span data-stu-id="282a7-269">Prerequisites</span></span>

1. <span data-ttu-id="282a7-270">Клонируйте или скачайте ZIP-файл в [репозитории GitHub](https://github.com/mspnp/identity-reference-architectures) либо создайте для него вилку.</span><span class="sxs-lookup"><span data-stu-id="282a7-270">Clone, fork, or download the zip file for the [GitHub repository](https://github.com/mspnp/identity-reference-architectures).</span></span>

1. <span data-ttu-id="282a7-271">Установите [Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest).</span><span class="sxs-lookup"><span data-stu-id="282a7-271">Install [Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest).</span></span>

1. <span data-ttu-id="282a7-272">Установите пакет npm [стандартных блоков Azure](https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks).</span><span class="sxs-lookup"><span data-stu-id="282a7-272">Install the [Azure building blocks](https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks) npm package.</span></span>

   ```bash
   npm install -g @mspnp/azure-building-blocks
   ```

1. <span data-ttu-id="282a7-273">Из командной строки, строки bash или строки PowerShell войдите в свою учетную запись Azure, как показано ниже:</span><span class="sxs-lookup"><span data-stu-id="282a7-273">From a command prompt, bash prompt, or PowerShell prompt, sign into your Azure account as follows:</span></span>

   ```bash
   az login
   ```

### <a name="deploy-the-simulated-on-premises-datacenter"></a><span data-ttu-id="282a7-274">Развертывание имитации локального центра обработки данных</span><span class="sxs-lookup"><span data-stu-id="282a7-274">Deploy the simulated on-premises datacenter</span></span>

1. <span data-ttu-id="282a7-275">Перейдите в папку `adfs` в репозитории GitHub.</span><span class="sxs-lookup"><span data-stu-id="282a7-275">Navigate to the `adfs` folder of the GitHub repository.</span></span>

1. <span data-ttu-id="282a7-276">Откройте файл `onprem.json` .</span><span class="sxs-lookup"><span data-stu-id="282a7-276">Open the `onprem.json` file.</span></span> <span data-ttu-id="282a7-277">Найдите экземпляры `adminPassword`, `Password` и `SafeModeAdminPassword` и обновите пароли.</span><span class="sxs-lookup"><span data-stu-id="282a7-277">Search for instances of `adminPassword`, `Password`, and `SafeModeAdminPassword` and update the passwords.</span></span>

1. <span data-ttu-id="282a7-278">Выполните следующую команду и дождитесь завершения развертывания:</span><span class="sxs-lookup"><span data-stu-id="282a7-278">Run the following command and wait for the deployment to finish:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p onprem.json --deploy
    ```

### <a name="deploy-the-azure-infrastructure"></a><span data-ttu-id="282a7-279">Развертывание инфраструктуры Azure</span><span class="sxs-lookup"><span data-stu-id="282a7-279">Deploy the Azure infrastructure</span></span>

1. <span data-ttu-id="282a7-280">Откройте файл `azure.json` .</span><span class="sxs-lookup"><span data-stu-id="282a7-280">Open the `azure.json` file.</span></span>  <span data-ttu-id="282a7-281">Найдите экземпляры `adminPassword` и `Password` и добавьте значения для паролей.</span><span class="sxs-lookup"><span data-stu-id="282a7-281">Search for instances of `adminPassword` and `Password` and add values for the passwords.</span></span>

1. <span data-ttu-id="282a7-282">Выполните следующую команду и дождитесь завершения развертывания:</span><span class="sxs-lookup"><span data-stu-id="282a7-282">Run the following command and wait for the deployment to finish:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p azure.json --deploy
    ```

### <a name="set-up-the-ad-fs-farm"></a><span data-ttu-id="282a7-283">Настройка фермы AD FS</span><span class="sxs-lookup"><span data-stu-id="282a7-283">Set up the AD FS farm</span></span>

1. <span data-ttu-id="282a7-284">Откройте файл `adfs-farm-first.json` .</span><span class="sxs-lookup"><span data-stu-id="282a7-284">Open the `adfs-farm-first.json` file.</span></span>  <span data-ttu-id="282a7-285">Найдите `AdminPassword` и замените пароль по умолчанию.</span><span class="sxs-lookup"><span data-stu-id="282a7-285">Search for `AdminPassword` and replace the default password.</span></span>

1. <span data-ttu-id="282a7-286">Выполните следующую команду:</span><span class="sxs-lookup"><span data-stu-id="282a7-286">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p adfs-farm-first.json --deploy
    ```

1. <span data-ttu-id="282a7-287">Откройте файл `adfs-farm-rest.json` .</span><span class="sxs-lookup"><span data-stu-id="282a7-287">Open the `adfs-farm-rest.json` file.</span></span>  <span data-ttu-id="282a7-288">Найдите `AdminPassword` и замените пароль по умолчанию.</span><span class="sxs-lookup"><span data-stu-id="282a7-288">Search for `AdminPassword` and replace the default password.</span></span>

1. <span data-ttu-id="282a7-289">Выполните следующую команду и дождитесь завершения развертывания:</span><span class="sxs-lookup"><span data-stu-id="282a7-289">Run the following command and wait for the deployment to finish:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p adfs-farm-rest.json --deploy
    ```

### <a name="configure-ad-fs-part-1"></a><span data-ttu-id="282a7-290">Настройка AD FS (часть 1)</span><span class="sxs-lookup"><span data-stu-id="282a7-290">Configure AD FS (part 1)</span></span>

1. <span data-ttu-id="282a7-291">Откройте сеанс удаленного входа в систему к виртуальной машине `ra-adfs-jb-vm1`, который является виртуальной машиной окна перехода.</span><span class="sxs-lookup"><span data-stu-id="282a7-291">Open a remote desktop session to the VM named `ra-adfs-jb-vm1`, which is the jumpbox VM.</span></span> <span data-ttu-id="282a7-292">Имя пользователя — `testuser`.</span><span class="sxs-lookup"><span data-stu-id="282a7-292">The user name is `testuser`.</span></span>

1. <span data-ttu-id="282a7-293">В окне перехода откройте сеанс удаленного входа в систему для виртуальной машины `ra-adfs-proxy-vm1`.</span><span class="sxs-lookup"><span data-stu-id="282a7-293">From the jumpbox, open a remote desktop session to the VM named `ra-adfs-proxy-vm1`.</span></span> <span data-ttu-id="282a7-294">Частный IP-адрес – 10.0.6.4.</span><span class="sxs-lookup"><span data-stu-id="282a7-294">The private IP address is 10.0.6.4.</span></span>

1. <span data-ttu-id="282a7-295">Из этого сеанса удаленного входа в систему запустите [Интегрированную среду сценариев PowerShell](/powershell/scripting/components/ise/windows-powershell-integrated-scripting-environment--ise-).</span><span class="sxs-lookup"><span data-stu-id="282a7-295">From this remote desktop session, run the [PowerShell ISE](/powershell/scripting/components/ise/windows-powershell-integrated-scripting-environment--ise-).</span></span>

1. <span data-ttu-id="282a7-296">В PowerShell перейдите в следующий каталог.</span><span class="sxs-lookup"><span data-stu-id="282a7-296">In PowerShell, navigate to the following directory:</span></span>

    ```powershell
    C:\Packages\Plugins\Microsoft.Powershell.DSC\2.77.0.0\DSCWork\adfs-v2.0
    ```

1. <span data-ttu-id="282a7-297">Вставьте следующий код в область сценариев и запустите его.</span><span class="sxs-lookup"><span data-stu-id="282a7-297">Paste the following code into a script pane and run it:</span></span>

    ```powershell
    . .\adfs-webproxy.ps1
    $cd = @{
        AllNodes = @(
            @{
                NodeName = 'localhost'
                PSDscAllowPlainTextPassword = $true
                PSDscAllowDomainUser = $true
            }
        )
    }

    $c1 = Get-Credential -UserName testuser -Message "Enter password"
    InstallWebProxyApp -DomainName contoso.com -FederationName adfs.contoso.com -WebApplicationProxyName "Contoso App" -AdminCreds $c1 -ConfigurationData $cd
    Start-DscConfiguration .\InstallWebProxyApp
    ```

    <span data-ttu-id="282a7-298">В строке `Get-Credential` введите пароль, указанный в файле параметров развертывания.</span><span class="sxs-lookup"><span data-stu-id="282a7-298">At the `Get-Credential` prompt, enter the password that you specified in the deployment parameter file.</span></span>

1. <span data-ttu-id="282a7-299">Выполните следующую команду, чтобы отслеживать ход выполнения конфигурации [DSC](/powershell/dsc/overview/overview).</span><span class="sxs-lookup"><span data-stu-id="282a7-299">Run the following command to monitor the progress of the [DSC](/powershell/dsc/overview/overview) configuration:</span></span>

    ```powershell
    Get-DscConfigurationStatus
    ```

    <span data-ttu-id="282a7-300">Необходимо несколько минут, чтобы достичь согласованности.</span><span class="sxs-lookup"><span data-stu-id="282a7-300">It can take several minutes to reach consistency.</span></span> <span data-ttu-id="282a7-301">В течение этого времени из команды могут возникнуть ошибки.</span><span class="sxs-lookup"><span data-stu-id="282a7-301">During this time, you may see errors from the command.</span></span> <span data-ttu-id="282a7-302">После успешного завершения конфигурации результат должен выглядеть так, как показано ниже.</span><span class="sxs-lookup"><span data-stu-id="282a7-302">When the configuration succeeds, the output should look similar to the following:</span></span>

    ```powershell
    PS C:\Packages\Plugins\Microsoft.Powershell.DSC\2.77.0.0\DSCWork\adfs-v2.0> Get-DscConfigurationStatus

    Status     StartDate                 Type            Mode  RebootRequested      NumberOfResources
    ------     ---------                 ----            ----  ---------------      -----------------
    Success    12/17/2018 8:21:09 PM     Consistency     PUSH  True                 4
    ```

### <a name="configure-ad-fs-part-2"></a><span data-ttu-id="282a7-303">Настройка AD FS (часть 2)</span><span class="sxs-lookup"><span data-stu-id="282a7-303">Configure AD FS (part 2)</span></span>

1. <span data-ttu-id="282a7-304">В окне перехода откройте сеанс удаленного входа в систему для виртуальной машины `ra-adfs-proxy-vm2`.</span><span class="sxs-lookup"><span data-stu-id="282a7-304">From the jumpbox, open a remote desktop session to the VM named `ra-adfs-proxy-vm2`.</span></span> <span data-ttu-id="282a7-305">Частный IP-адрес – 10.0.6.5.</span><span class="sxs-lookup"><span data-stu-id="282a7-305">The private IP address is 10.0.6.5.</span></span>

1. <span data-ttu-id="282a7-306">Из этого сеанса удаленного входа в систему запустите [Интегрированную среду сценариев PowerShell](/powershell/scripting/components/ise/windows-powershell-integrated-scripting-environment--ise-).</span><span class="sxs-lookup"><span data-stu-id="282a7-306">From this remote desktop session, run the [PowerShell ISE](/powershell/scripting/components/ise/windows-powershell-integrated-scripting-environment--ise-).</span></span>

1. <span data-ttu-id="282a7-307">Перейдите в следующую папку:</span><span class="sxs-lookup"><span data-stu-id="282a7-307">Navigate to the following directory:</span></span>

    ```powershell
    C:\Packages\Plugins\Microsoft.Powershell.DSC\2.77.0.0\DSCWork\adfs-v2.0
    ```

1. <span data-ttu-id="282a7-308">Вставьте следующее в область сценариев и выполните этот сценарий:</span><span class="sxs-lookup"><span data-stu-id="282a7-308">Past the following in a script pane and run the script:</span></span>

    ```powershell
    . .\adfs-webproxy-rest.ps1
    $cd = @{
        AllNodes = @(
            @{
                NodeName = 'localhost'
                PSDscAllowPlainTextPassword = $true
                PSDscAllowDomainUser = $true
            }
        )
    }

    $c1 = Get-Credential -UserName testuser -Message "Enter password"
    InstallWebProxy -DomainName contoso.com -FederationName adfs.contoso.com -WebApplicationProxyName "Contoso App" -AdminCreds $c1 -ConfigurationData $cd
    Start-DscConfiguration .\InstallWebProxy
    ```

    <span data-ttu-id="282a7-309">В строке `Get-Credential` введите пароль, указанный в файле параметров развертывания.</span><span class="sxs-lookup"><span data-stu-id="282a7-309">At the `Get-Credential` prompt, enter the password that you specified in the deployment parameter file.</span></span>

1. <span data-ttu-id="282a7-310">Выполните следующую команду, чтобы отслеживать ход выполнения конфигурации.</span><span class="sxs-lookup"><span data-stu-id="282a7-310">Run the following command to monitor the progress of the DSC configuration:</span></span>

    ```powershell
    Get-DscConfigurationStatus
    ```

    <span data-ttu-id="282a7-311">Необходимо несколько минут, чтобы достичь согласованности.</span><span class="sxs-lookup"><span data-stu-id="282a7-311">It can take several minutes to reach consistency.</span></span> <span data-ttu-id="282a7-312">В течение этого времени из команды могут возникнуть ошибки.</span><span class="sxs-lookup"><span data-stu-id="282a7-312">During this time, you may see errors from the command.</span></span> <span data-ttu-id="282a7-313">После успешного завершения конфигурации результат должен выглядеть так, как показано ниже.</span><span class="sxs-lookup"><span data-stu-id="282a7-313">When the configuration succeeds, the output should look similar to the following:</span></span>

    ```powershell
    PS C:\Packages\Plugins\Microsoft.Powershell.DSC\2.77.0.0\DSCWork\adfs-v2.0> Get-DscConfigurationStatus

    Status     StartDate                 Type            Mode  RebootRequested      NumberOfResources
    ------     ---------                 ----            ----  ---------------      -----------------
    Success    12/17/2018 8:21:09 PM     Consistency     PUSH  True                 4
    ```

    <span data-ttu-id="282a7-314">Иногда происходит сбой DSC.</span><span class="sxs-lookup"><span data-stu-id="282a7-314">Sometimes this DSC fails.</span></span> <span data-ttu-id="282a7-315">Если проверка состояния возвращает `Status=Failure` и `Type=Consistency`, попробуйте перезапустить шаг 4.</span><span class="sxs-lookup"><span data-stu-id="282a7-315">If the status check shows `Status=Failure` and `Type=Consistency`, try re-running step 4.</span></span>

### <a name="sign-into-ad-fs"></a><span data-ttu-id="282a7-316">Вход в AD FS</span><span class="sxs-lookup"><span data-stu-id="282a7-316">Sign into AD FS</span></span>

1. <span data-ttu-id="282a7-317">В окне перехода откройте сеанс удаленного входа в систему для виртуальной машины `ra-adfs-adfs-vm1`.</span><span class="sxs-lookup"><span data-stu-id="282a7-317">From the jumpbox, open a remote desktop session to the VM named `ra-adfs-adfs-vm1`.</span></span> <span data-ttu-id="282a7-318">Частный IP-адрес – 10.0.5.4.</span><span class="sxs-lookup"><span data-stu-id="282a7-318">The private IP address is 10.0.5.4.</span></span>

1. <span data-ttu-id="282a7-319">Выполните действия, описанные в разделе [Enable the Idp-Intiated Sign on page](/windows-server/identity/ad-fs/troubleshooting/ad-fs-tshoot-initiatedsignon#enable-the-idp-intiated-sign-on-page) (Включение страницы входа для поставщика удостоверений), чтобы включить страницу входа.</span><span class="sxs-lookup"><span data-stu-id="282a7-319">Follow the steps in [Enable the Idp-Intiated Sign on page](/windows-server/identity/ad-fs/troubleshooting/ad-fs-tshoot-initiatedsignon#enable-the-idp-intiated-sign-on-page) to enable the sign-on page.</span></span>

1. <span data-ttu-id="282a7-320">В окне перехода просмотрите `https://adfs.contoso.com/adfs/ls/idpinitiatedsignon.htm`.</span><span class="sxs-lookup"><span data-stu-id="282a7-320">From the jump box, browse to `https://adfs.contoso.com/adfs/ls/idpinitiatedsignon.htm`.</span></span> <span data-ttu-id="282a7-321">Здесь может отобразиться предупреждение сертификата, которое в этом примере можно проигнорировать.</span><span class="sxs-lookup"><span data-stu-id="282a7-321">You may receive a certificate warning that you can ignore for this test.</span></span>

1. <span data-ttu-id="282a7-322">Убедитесь, что отображается страница входа в корпорацию Contoso.</span><span class="sxs-lookup"><span data-stu-id="282a7-322">Verify that the Contoso Corporation sign-in page appears.</span></span> <span data-ttu-id="282a7-323">Выполните вход с именем пользователя **contoso\testuser**.</span><span class="sxs-lookup"><span data-stu-id="282a7-323">Sign in as **contoso\testuser**.</span></span>

<!-- links -->
[extending-ad-to-azure]: adds-extend-domain.md

[vm-recommendations]: ../virtual-machines-windows/single-vm.md
[implementing-a-secure-hybrid-network-architecture]: ../dmz/secure-vnet-hybrid.md
[implementing-a-secure-hybrid-network-architecture-with-internet-access]: ../dmz/secure-vnet-dmz.md
[hybrid-azure-on-prem-vpn]: ../hybrid-networking/vpn.md

[azure-cli]: /azure/azure-resource-manager/xplat-cli-azure-resource-manager
[DRS]: https://technet.microsoft.com/library/dn280945.aspx
[where-to-place-an-fs-proxy]: https://technet.microsoft.com/library/dd807048.aspx
[ADDRS]: https://technet.microsoft.com/library/dn486831.aspx
[plan-your-adfs-deployment]: https://msdn.microsoft.com/library/azure/dn151324.aspx
[ad_network_recommendations]: #network_configuration_recommendations_for_AD_DS_VMs
[adfs_certificates]: https://technet.microsoft.com/library/dn781428(v=ws.11).aspx
[create_service_account_for_adfs_farm]: https://technet.microsoft.com/library/dd807078.aspx
[adfs-configuration-database]: https://technet.microsoft.com/library/ee913581(v=ws.11).aspx
[active-directory-federation-services]: /windows-server/identity/active-directory-federation-services
[security-considerations]: #security-considerations
[recommendations]: #recommendations
[active-directory-federation-services-overview]: https://technet.microsoft.com/library/hh831502(v=ws.11).aspx
[establishing-federation-trust]: https://blogs.msdn.microsoft.com/alextch/2011/06/27/establishing-federation-trust/
[Deploying_a_federation_server_farm]:  /windows-server/identity/ad-fs/deployment/deploying-a-federation-server-farm
[install_and_configure_the_web_application_proxy_server]: https://technet.microsoft.com/library/dn383662.aspx
[publish_applications_using_AD_FS_preauthentication]: https://technet.microsoft.com/library/dn383640.aspx
[managing-adfs-components]: https://technet.microsoft.com/library/cc759026.aspx
[oms-adfs-pack]: https://www.microsoft.com/download/details.aspx?id=41184
[azure-powershell-download]: /powershell/azure/overview
[aad]: /azure/active-directory/
[aadb2c]: /azure/active-directory-b2c/
[adfs-intro]: /azure/active-directory/hybrid/whatis-hybrid-identity
[github]: https://github.com/mspnp/identity-reference-architectures/tree/master/adfs
[adfs_certificates]: https://technet.microsoft.com/library/dn781428(v=ws.11).aspx
[considerations]: ./considerations.md
[visio-download]: https://archcenter.blob.core.windows.net/cdn/identity-architectures.vsdx
