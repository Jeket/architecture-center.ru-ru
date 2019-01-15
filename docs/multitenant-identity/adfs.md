---
title: Федерация с клиентской службой AD FS
description: Как создать федерацию с AD FS клиента в мультитенантном приложении.
author: MikeWasson
ms.date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: token-cache
pnp.series.next: client-assertion
ms.openlocfilehash: 27fad1aab8d359346353cc031a2e8d8746294818
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54113558"
---
# <a name="federate-with-a-customers-ad-fs"></a><span data-ttu-id="89551-103">Федерация с клиентской службой AD FS</span><span class="sxs-lookup"><span data-stu-id="89551-103">Federate with a customer's AD FS</span></span>

<span data-ttu-id="89551-104">В этой статье описывается, каким образом мультитенантное приложение SaaS поддерживает проверку подлинности через службы федерации Active Directory (AD FS) для создания федерации со службой AD FS клиента.</span><span class="sxs-lookup"><span data-stu-id="89551-104">This article describes how a multi-tenant SaaS application can support authentication via Active Directory Federation Services (AD FS), in order to federate with a customer's AD FS.</span></span>

## <a name="overview"></a><span data-ttu-id="89551-105">Обзор</span><span class="sxs-lookup"><span data-stu-id="89551-105">Overview</span></span>

<span data-ttu-id="89551-106">Azure Active Directory (Azure AD) упрощает процесс входа пользователей из клиентов Azure AD, включая клиентов Office 365 и Dynamics CRM Online.</span><span class="sxs-lookup"><span data-stu-id="89551-106">Azure Active Directory (Azure AD) makes it easy to sign in users from Azure AD tenants, including Office365 and Dynamics CRM Online customers.</span></span> <span data-ttu-id="89551-107">Но как быть с пользователями, использующими локальную службу Active Directory в корпоративной интрасети?</span><span class="sxs-lookup"><span data-stu-id="89551-107">But what about customers who use on-premise Active Directory on a corporate intranet?</span></span>

<span data-ttu-id="89551-108">Одним из вариантов для таких пользователей является синхронизация локальной службы AD с Azure AD с помощью [Azure AD Connect].</span><span class="sxs-lookup"><span data-stu-id="89551-108">One option is for these customers to sync their on-premise AD with Azure AD, using [Azure AD Connect].</span></span> <span data-ttu-id="89551-109">Но, возможно, некоторым клиентам не удастся реализовать этот подход ввиду особенностей корпоративной ИТ-политики или по другим причинам.</span><span class="sxs-lookup"><span data-stu-id="89551-109">However, some customers may be unable to use this approach, due to corporate IT policy or other reasons.</span></span> <span data-ttu-id="89551-110">В этом случае следует рассмотреть другой вариант — создание федерации с помощью служб федерации Active Directory (AD FS).</span><span class="sxs-lookup"><span data-stu-id="89551-110">In that case, another option is to federate through Active Directory Federation Services (AD FS).</span></span>

<span data-ttu-id="89551-111">Чтобы реализовать этот сценарий, необходимо выполнить приведенные ниже условия.</span><span class="sxs-lookup"><span data-stu-id="89551-111">To enable this scenario:</span></span>

* <span data-ttu-id="89551-112">У клиента должна быть доступная через Интернет ферма AD FS.</span><span class="sxs-lookup"><span data-stu-id="89551-112">The customer must have an Internet-facing AD FS farm.</span></span>
* <span data-ttu-id="89551-113">Поставщик SaaS развертывает собственную ферму AD FS.</span><span class="sxs-lookup"><span data-stu-id="89551-113">The SaaS provider deploys their own AD FS farm.</span></span>
* <span data-ttu-id="89551-114">Клиент и поставщик SaaS должны установить [доверие федерации].</span><span class="sxs-lookup"><span data-stu-id="89551-114">The customer and the SaaS provider must set up [federation trust].</span></span> <span data-ttu-id="89551-115">Этот процесс выполняется в ручную.</span><span class="sxs-lookup"><span data-stu-id="89551-115">This is a manual process.</span></span>

<span data-ttu-id="89551-116">В отношении доверия существуют три основные роли.</span><span class="sxs-lookup"><span data-stu-id="89551-116">There are three main roles in the trust relation:</span></span>

* <span data-ttu-id="89551-117">AD FS клиента является [партнером по учетным записям], отвечающим за проверку подлинности пользователей в службе AD клиента и за создание маркеров безопасности с утверждениями пользователей.</span><span class="sxs-lookup"><span data-stu-id="89551-117">The customer's AD FS is the [account partner], responsible for authenticating users from the customer's AD, and creating security tokens with user claims.</span></span>
* <span data-ttu-id="89551-118">AD FS поставщика SaaS является [партнером по учетным запися], который доверяет партнеру по учетным записям и получает утверждения пользователей.</span><span class="sxs-lookup"><span data-stu-id="89551-118">The SaaS provider's AD FS is the [resource partner], which trusts the account partner and receives the user claims.</span></span>
* <span data-ttu-id="89551-119">Приложение настроено в качестве проверяющей стороны (RP) в службе AD FS поставщика SaaS.</span><span class="sxs-lookup"><span data-stu-id="89551-119">The application is configured as a relying party (RP) in the SaaS provider's AD FS.</span></span>
  
  ![доверие федерации](./images/federation-trust.png)

> [!NOTE]
> <span data-ttu-id="89551-121">В этой статье предполагается, что в качестве протокола проверки подлинности приложение использует OpenID Connect.</span><span class="sxs-lookup"><span data-stu-id="89551-121">In this article, we assume the application uses OpenID connect as the authentication protocol.</span></span> <span data-ttu-id="89551-122">Другим вариантом является использование WS-Federation.</span><span class="sxs-lookup"><span data-stu-id="89551-122">Another option is to use WS-Federation.</span></span>
>
> <span data-ttu-id="89551-123">Чтобы использовать OpenID Connect, поставщик SaaS должен работать с AD FS 2016 под управлением Windows Server 2016.</span><span class="sxs-lookup"><span data-stu-id="89551-123">For OpenID Connect, the SaaS provider must use AD FS 2016, running in Windows Server 2016.</span></span> <span data-ttu-id="89551-124">AD FS 3.0 не поддерживает OpenID Connect.</span><span class="sxs-lookup"><span data-stu-id="89551-124">AD FS 3.0 does not support OpenID Connect.</span></span>
>
> <span data-ttu-id="89551-125">В ASP.NET Core не предусмотрена встроенная поддержка WS-Federation.</span><span class="sxs-lookup"><span data-stu-id="89551-125">ASP.NET Core does not include out-of-the-box support for WS-Federation.</span></span>
>
>

<span data-ttu-id="89551-126">Пример использования WS-Federation для ASP.NET 4 вы найдете на странице GitHub [active-directory-dotnet-webapp-wsfederation][active-directory-dotnet-webapp-wsfederation].</span><span class="sxs-lookup"><span data-stu-id="89551-126">For an example of using WS-Federation with ASP.NET 4, see the [active-directory-dotnet-webapp-wsfederation sample][active-directory-dotnet-webapp-wsfederation].</span></span>

## <a name="authentication-flow"></a><span data-ttu-id="89551-127">Поток проверки подлинности</span><span class="sxs-lookup"><span data-stu-id="89551-127">Authentication flow</span></span>

1. <span data-ttu-id="89551-128">Когда пользователь нажимает кнопку входа, приложение выполняет перенаправление в конечную точку OpenID Connect в службе AD FS поставщика SaaS.</span><span class="sxs-lookup"><span data-stu-id="89551-128">When the user clicks "sign in", the application redirects to an OpenID Connect endpoint on the SaaS provider's AD FS.</span></span>
2. <span data-ttu-id="89551-129">Пользователь вводит имя пользователя организации ("`alice@corp.contoso.com`").</span><span class="sxs-lookup"><span data-stu-id="89551-129">The user enters his or her organizational user name ("`alice@corp.contoso.com`").</span></span> <span data-ttu-id="89551-130">AD FS использует обнаружение домашней области для перенаправления в службу AD FS клиента, где пользователь вводит свои учетные данные.</span><span class="sxs-lookup"><span data-stu-id="89551-130">AD FS uses home realm discovery to redirect to the customer's AD FS, where the user enters their credentials.</span></span>
3. <span data-ttu-id="89551-131">AD FS клиента отправляет утверждения пользователя в AD FS поставщика SaaS с помощью WF-Federation (или SAML).</span><span class="sxs-lookup"><span data-stu-id="89551-131">The customer's AD FS sends user claims to the SaaS provider's AD FS, using WF-Federation (or SAML).</span></span>
4. <span data-ttu-id="89551-132">Утверждения отправляются из AD FS в приложение с помощью OpenID Connect.</span><span class="sxs-lookup"><span data-stu-id="89551-132">Claims flow from AD FS to the app, using OpenID Connect.</span></span> <span data-ttu-id="89551-133">Для этого необходимо сменить протокол WS-Federation.</span><span class="sxs-lookup"><span data-stu-id="89551-133">This requires a protocol transition from WS-Federation.</span></span>

## <a name="limitations"></a><span data-ttu-id="89551-134">Ограничения</span><span class="sxs-lookup"><span data-stu-id="89551-134">Limitations</span></span>

<span data-ttu-id="89551-135">По умолчанию приложение проверяющей стороны получает через id_token только фиксированный набор утверждений, которые перечислены в приведенной ниже таблице.</span><span class="sxs-lookup"><span data-stu-id="89551-135">By default, the relying party application receives only a fixed set of claims available in the id_token, shown in the following table.</span></span> <span data-ttu-id="89551-136">В AD FS 2016 вы можете настроить id_token для сценариев OpenID Connect.</span><span class="sxs-lookup"><span data-stu-id="89551-136">With AD FS 2016, you can customize the id_token in OpenID Connect scenarios.</span></span> <span data-ttu-id="89551-137">Дополнительные сведения см. в разделе [Custom ID Tokens in AD FS](/windows-server/identity/ad-fs/development/customize-id-token-ad-fs-2016) (Пользовательские маркеры идентификаторов в Azure AD FS).</span><span class="sxs-lookup"><span data-stu-id="89551-137">For more information, see [Custom ID Tokens in AD FS](/windows-server/identity/ad-fs/development/customize-id-token-ad-fs-2016).</span></span>

| <span data-ttu-id="89551-138">Утверждение</span><span class="sxs-lookup"><span data-stu-id="89551-138">Claim</span></span> | <span data-ttu-id="89551-139">ОПИСАНИЕ</span><span class="sxs-lookup"><span data-stu-id="89551-139">Description</span></span> |
| --- | --- |
| <span data-ttu-id="89551-140">aud</span><span class="sxs-lookup"><span data-stu-id="89551-140">aud</span></span> |<span data-ttu-id="89551-141">Аудитория.</span><span class="sxs-lookup"><span data-stu-id="89551-141">Audience.</span></span> <span data-ttu-id="89551-142">Это приложение, для которого выданы утверждения.</span><span class="sxs-lookup"><span data-stu-id="89551-142">The application for which the claims were issued.</span></span> |
| <span data-ttu-id="89551-143">authenticationinstant</span><span class="sxs-lookup"><span data-stu-id="89551-143">authenticationinstant</span></span> |<span data-ttu-id="89551-144">[Время выполнения проверки подлинности]</span><span class="sxs-lookup"><span data-stu-id="89551-144">[Authentication instant].</span></span> <span data-ttu-id="89551-145">Время, когда происходила аутентификация.</span><span class="sxs-lookup"><span data-stu-id="89551-145">The time at which authentication occurred.</span></span> |
| <span data-ttu-id="89551-146">c_hash</span><span class="sxs-lookup"><span data-stu-id="89551-146">c_hash</span></span> |<span data-ttu-id="89551-147">Значение хэш-кода.</span><span class="sxs-lookup"><span data-stu-id="89551-147">Code hash value.</span></span> <span data-ttu-id="89551-148">Это хэш содержимого маркера.</span><span class="sxs-lookup"><span data-stu-id="89551-148">This is a hash of the token contents.</span></span> |
| <span data-ttu-id="89551-149">exp</span><span class="sxs-lookup"><span data-stu-id="89551-149">exp</span></span> |<span data-ttu-id="89551-150">[Время окончания срока действия]</span><span class="sxs-lookup"><span data-stu-id="89551-150">[Expiration time].</span></span> <span data-ttu-id="89551-151">После наступления этого момента маркер больше не будет приниматься.</span><span class="sxs-lookup"><span data-stu-id="89551-151">The time after which the token will no longer be accepted.</span></span> |
| <span data-ttu-id="89551-152">iat</span><span class="sxs-lookup"><span data-stu-id="89551-152">iat</span></span> |<span data-ttu-id="89551-153">Время выдачи.</span><span class="sxs-lookup"><span data-stu-id="89551-153">Issued at.</span></span> <span data-ttu-id="89551-154">Это время, когда был выдан маркер.</span><span class="sxs-lookup"><span data-stu-id="89551-154">The time when the token was issued.</span></span> |
| <span data-ttu-id="89551-155">iss</span><span class="sxs-lookup"><span data-stu-id="89551-155">iss</span></span> |<span data-ttu-id="89551-156">Издатель.</span><span class="sxs-lookup"><span data-stu-id="89551-156">Issuer.</span></span> <span data-ttu-id="89551-157">Значение этого утверждения всегда указывает на AD FS партнера по ресурсам.</span><span class="sxs-lookup"><span data-stu-id="89551-157">The value of this claim is always the resource partner's AD FS.</span></span> |
| <span data-ttu-id="89551-158">name</span><span class="sxs-lookup"><span data-stu-id="89551-158">name</span></span> |<span data-ttu-id="89551-159">Имя пользователя.</span><span class="sxs-lookup"><span data-stu-id="89551-159">User name.</span></span> <span data-ttu-id="89551-160">Пример: `john@corp.fabrikam.com`</span><span class="sxs-lookup"><span data-stu-id="89551-160">Example: `john@corp.fabrikam.com`</span></span> |
| <span data-ttu-id="89551-161">nameidentifier</span><span class="sxs-lookup"><span data-stu-id="89551-161">nameidentifier</span></span> |<span data-ttu-id="89551-162">[Идентификатор имени]</span><span class="sxs-lookup"><span data-stu-id="89551-162">[Name identifier].</span></span> <span data-ttu-id="89551-163">Идентификатор имени сущности, для которой был выдан маркер.</span><span class="sxs-lookup"><span data-stu-id="89551-163">The identifier for the name of the entity for which the token was issued.</span></span> |
| <span data-ttu-id="89551-164">nonce</span><span class="sxs-lookup"><span data-stu-id="89551-164">nonce</span></span> |<span data-ttu-id="89551-165">Специальное утверждение сеанса.</span><span class="sxs-lookup"><span data-stu-id="89551-165">Session nonce.</span></span> <span data-ttu-id="89551-166">Уникальное значение, которое AD FS создает для защиты от атак с повторением пакетов.</span><span class="sxs-lookup"><span data-stu-id="89551-166">A unique value generated by AD FS to help prevent replay attacks.</span></span> |
| <span data-ttu-id="89551-167">upn</span><span class="sxs-lookup"><span data-stu-id="89551-167">upn</span></span> |<span data-ttu-id="89551-168">Имя участника-пользователя (UPN).</span><span class="sxs-lookup"><span data-stu-id="89551-168">User principal name (UPN).</span></span> <span data-ttu-id="89551-169">Пример: `john@corp.fabrikam.com`</span><span class="sxs-lookup"><span data-stu-id="89551-169">Example: `john@corp.fabrikam.com`</span></span> |
| <span data-ttu-id="89551-170">pwd_exp</span><span class="sxs-lookup"><span data-stu-id="89551-170">pwd_exp</span></span> |<span data-ttu-id="89551-171">Период до истечения срока действия пароля.</span><span class="sxs-lookup"><span data-stu-id="89551-171">Password expiration period.</span></span> <span data-ttu-id="89551-172">Время в секундах, оставшееся до окончания срока действия для пароля пользователя или аналогичного секрета аутентификации,</span><span class="sxs-lookup"><span data-stu-id="89551-172">The number of seconds until the user's password or a similar authentication secret, such as a PIN.</span></span> <span data-ttu-id="89551-173">например ПИН-кода.</span><span class="sxs-lookup"><span data-stu-id="89551-173">expires.</span></span> |

> [!NOTE]
> <span data-ttu-id="89551-174">Утверждение "iss" указывает на AD FS партнера (как правило, это утверждение определяет поставщика SaaS у этого издателя).</span><span class="sxs-lookup"><span data-stu-id="89551-174">The "iss" claim contains the AD FS of the partner (typically, this claim will identify the SaaS provider as the issuer).</span></span> <span data-ttu-id="89551-175">Оно не имеет отношения к AD FS клиента.</span><span class="sxs-lookup"><span data-stu-id="89551-175">It does not identify the customer's AD FS.</span></span> <span data-ttu-id="89551-176">Домен клиента вы можете извлечь из имени участника-пользователя.</span><span class="sxs-lookup"><span data-stu-id="89551-176">You can find the customer's domain as part of the UPN.</span></span>

<span data-ttu-id="89551-177">Далее в этой статье описывается процедура установки отношения доверия между RP (приложением) и партнером по учетным записям (клиентом).</span><span class="sxs-lookup"><span data-stu-id="89551-177">The rest of this article describes how to set up the trust relationship between the RP (the app) and the account partner (the customer).</span></span>

## <a name="ad-fs-deployment"></a><span data-ttu-id="89551-178">Развертывание AD FS</span><span class="sxs-lookup"><span data-stu-id="89551-178">AD FS deployment</span></span>

<span data-ttu-id="89551-179">Поставщик SaaS может развернуть AD FS локально или на виртуальных машинах Azure.</span><span class="sxs-lookup"><span data-stu-id="89551-179">The SaaS provider can deploy AD FS either on-premise or on Azure VMs.</span></span> <span data-ttu-id="89551-180">В целях обеспечения безопасности и доступности важно учесть приведенные далее рекомендации.</span><span class="sxs-lookup"><span data-stu-id="89551-180">For security and availability, the following guidelines are important:</span></span>

* <span data-ttu-id="89551-181">Для обеспечения максимальной доступности службы AD FS следует развернуть по меньшей мере два сервера AD FS и два прокси-сервера AD FS.</span><span class="sxs-lookup"><span data-stu-id="89551-181">Deploy at least two AD FS servers and two AD FS proxy servers to achieve the best availability of the AD FS service.</span></span>
* <span data-ttu-id="89551-182">Контроллеры домена и серверы AD FS нельзя предоставлять для прямого доступа из Интернета. Их следует разместить в виртуальной сети с прямым доступом к ним.</span><span class="sxs-lookup"><span data-stu-id="89551-182">Domain controllers and AD FS servers should never be exposed directly to the Internet and should be in a virtual network with direct access to them.</span></span>
* <span data-ttu-id="89551-183">Для публикации серверов AD FS в Интернете необходимо использовать прокси-службы веб-приложений (ранее — прокси AD FS).</span><span class="sxs-lookup"><span data-stu-id="89551-183">Web application proxies (previously AD FS proxies) must be used to publish AD FS servers to the Internet.</span></span>

<span data-ttu-id="89551-184">Чтобы настроить такую топологию в Azure, нужно применить виртуальные сети, группы безопасности сети, виртуальные машины Azure и группы доступности.</span><span class="sxs-lookup"><span data-stu-id="89551-184">To set up a similar topology in Azure requires the use of Virtual networks, NSG’s, azure VM’s and availability sets.</span></span> <span data-ttu-id="89551-185">См. [рекомендации по развертыванию Windows Server Active Directory на виртуальных машинах Azure][active-directory-on-azure].</span><span class="sxs-lookup"><span data-stu-id="89551-185">For more details, see [Guidelines for Deploying Windows Server Active Directory on Azure Virtual Machines][active-directory-on-azure].</span></span>

## <a name="configure-openid-connect-authentication-with-ad-fs"></a><span data-ttu-id="89551-186">Настройка аутентификации OpenID Connect в AD FS</span><span class="sxs-lookup"><span data-stu-id="89551-186">Configure OpenID Connect authentication with AD FS</span></span>

<span data-ttu-id="89551-187">Поставщик SaaS должен включить OpenID Connect между приложением и AD FS.</span><span class="sxs-lookup"><span data-stu-id="89551-187">The SaaS provider must enable OpenID Connect between the application and AD FS.</span></span> <span data-ttu-id="89551-188">Для этого нужно добавить группу приложений в AD FS.</span><span class="sxs-lookup"><span data-stu-id="89551-188">To do so, add an application group in AD FS.</span></span>  <span data-ttu-id="89551-189">Подробные инструкции можно найти в этой [записи блога]в разделе "Настройка веб-приложения для входа OpenId Connect в AD FS".</span><span class="sxs-lookup"><span data-stu-id="89551-189">You can find detailed instructions in this [blog post], under " Setting up a Web App for OpenId Connect sign in AD FS."</span></span>

<span data-ttu-id="89551-190">Теперь настройте ПО промежуточного слоя OpenID Connect.</span><span class="sxs-lookup"><span data-stu-id="89551-190">Next, configure the OpenID Connect middleware.</span></span> <span data-ttu-id="89551-191">Конечной точкой метаданных является `https://domain/adfs/.well-known/openid-configuration`, где домен представляет собой домен AD FS поставщика SaaS.</span><span class="sxs-lookup"><span data-stu-id="89551-191">The metadata endpoint is `https://domain/adfs/.well-known/openid-configuration`, where domain is the SaaS provider's AD FS domain.</span></span>

<span data-ttu-id="89551-192">Как правило, эту точку можно объединять с другими конечными точками OpenID Connect (например AAD).</span><span class="sxs-lookup"><span data-stu-id="89551-192">Typically you might combine this with other OpenID Connect endpoints (such as AAD).</span></span> <span data-ttu-id="89551-193">Чтобы различать эти точки и отправлять пользователя в нужную конечную точку проверки подлинности, потребуется две разные кнопки входа или какой-либо другой способ определения соответствующей точки.</span><span class="sxs-lookup"><span data-stu-id="89551-193">You'll need two different sign-in buttons or some other way to distinguish them, so that the user is sent to the correct authentication endpoint.</span></span>

## <a name="configure-the-ad-fs-resource-partner"></a><span data-ttu-id="89551-194">Настройка партнера по ресурсам AD FS</span><span class="sxs-lookup"><span data-stu-id="89551-194">Configure the AD FS Resource Partner</span></span>

<span data-ttu-id="89551-195">Поставщик SaaS должен выполнить приведенные далее действия для каждого клиента, который хочет подключаться через ADFS.</span><span class="sxs-lookup"><span data-stu-id="89551-195">The SaaS provider must do the following for each customer that wants to connect via ADFS:</span></span>

1. <span data-ttu-id="89551-196">Добавление отношения доверия поставщика утверждений.</span><span class="sxs-lookup"><span data-stu-id="89551-196">Add a claims provider trust.</span></span>
2. <span data-ttu-id="89551-197">Добавление правил утверждений.</span><span class="sxs-lookup"><span data-stu-id="89551-197">Add claims rules.</span></span>
3. <span data-ttu-id="89551-198">Включение обнаружения домашней области.</span><span class="sxs-lookup"><span data-stu-id="89551-198">Enable home-realm discovery.</span></span>

<span data-ttu-id="89551-199">Ниже приведено более подробное описание этих шагов.</span><span class="sxs-lookup"><span data-stu-id="89551-199">Here are the steps in more detail.</span></span>

### <a name="add-the-claims-provider-trust"></a><span data-ttu-id="89551-200">Добавление отношения доверия поставщика утверждений</span><span class="sxs-lookup"><span data-stu-id="89551-200">Add the claims provider trust</span></span>

1. <span data-ttu-id="89551-201">В диспетчере сервера щелкните **Средства** и выберите **Управление AD FS**.</span><span class="sxs-lookup"><span data-stu-id="89551-201">In Server Manager, click **Tools**, and then select **AD FS Management**.</span></span>
2. <span data-ttu-id="89551-202">В дереве консоли в области **AD FS** щелкните правой кнопкой мыши **Отношения доверия поставщиков утверждений**.</span><span class="sxs-lookup"><span data-stu-id="89551-202">In the console tree, under **AD FS**, right click **Claims Provider Trusts**.</span></span> <span data-ttu-id="89551-203">Выберите **Добавить отношение доверия поставщика утверждений**.</span><span class="sxs-lookup"><span data-stu-id="89551-203">Select **Add Claims Provider Trust**.</span></span>
3. <span data-ttu-id="89551-204">Щелкните **Запустить** , чтобы запустить мастер.</span><span class="sxs-lookup"><span data-stu-id="89551-204">Click **Start** to start the wizard.</span></span>
4. <span data-ttu-id="89551-205">Выберите параметр "Импорт данных о поставщике утверждений, опубликованных в Интернете или локальной сети".</span><span class="sxs-lookup"><span data-stu-id="89551-205">Select the option "Import data about the claims provider published online or on a local network".</span></span> <span data-ttu-id="89551-206">Введите URI конечной точки метаданных федерации, предоставленной поставщиком SaaS.</span><span class="sxs-lookup"><span data-stu-id="89551-206">Enter the URI of the customer's federation metadata endpoint.</span></span> <span data-ttu-id="89551-207">(Например: `https://contoso.com/FederationMetadata/2007-06/FederationMetadata.xml`.) Эти данные следует получить у клиента.</span><span class="sxs-lookup"><span data-stu-id="89551-207">(Example: `https://contoso.com/FederationMetadata/2007-06/FederationMetadata.xml`.) You will need to get this from the customer.</span></span>
5. <span data-ttu-id="89551-208">Завершите работу мастера, оставив значения по умолчанию.</span><span class="sxs-lookup"><span data-stu-id="89551-208">Complete the wizard using the default options.</span></span>

### <a name="edit-claims-rules"></a><span data-ttu-id="89551-209">Изменение правил утверждений</span><span class="sxs-lookup"><span data-stu-id="89551-209">Edit claims rules</span></span>

1. <span data-ttu-id="89551-210">Щелкните правой кнопкой мыши только что добавленное отношение доверия поставщика утверждений и выберите команду **Изменить правила утверждений**.</span><span class="sxs-lookup"><span data-stu-id="89551-210">Right-click the newly added claims provider trust, and select **Edit Claims Rules**.</span></span>
2. <span data-ttu-id="89551-211">Щелкните **Добавить правило**.</span><span class="sxs-lookup"><span data-stu-id="89551-211">Click **Add Rule**.</span></span>
3. <span data-ttu-id="89551-212">Выберите "Проход через входящее утверждение или его фильтрация" и щелкните **Далее**.</span><span class="sxs-lookup"><span data-stu-id="89551-212">Select "Pass Through or Filter an Incoming Claim" and click **Next**.</span></span>
   <span data-ttu-id="89551-213">![Мастер добавления правила преобразования утверждений](./images/edit-claims-rule.png)</span><span class="sxs-lookup"><span data-stu-id="89551-213">![Add Transform Claim Rule Wizard](./images/edit-claims-rule.png)</span></span>
4. <span data-ttu-id="89551-214">Введите имя правила.</span><span class="sxs-lookup"><span data-stu-id="89551-214">Enter a name for the rule.</span></span>
5. <span data-ttu-id="89551-215">В поле "Тип входящего утверждения" выберите **UPN**.</span><span class="sxs-lookup"><span data-stu-id="89551-215">Under "Incoming claim type", select **UPN**.</span></span>
6. <span data-ttu-id="89551-216">Выберите "Пропускать все значения утверждений".</span><span class="sxs-lookup"><span data-stu-id="89551-216">Select "Pass through all claim values".</span></span>
   <span data-ttu-id="89551-217">![Мастер добавления правила преобразования утверждений](./images/edit-claims-rule2.png)</span><span class="sxs-lookup"><span data-stu-id="89551-217">![Add Transform Claim Rule Wizard](./images/edit-claims-rule2.png)</span></span>
7. <span data-ttu-id="89551-218">Нажмите кнопку **Готово**</span><span class="sxs-lookup"><span data-stu-id="89551-218">Click **Finish**.</span></span>
8. <span data-ttu-id="89551-219">Повторите шаги 2–7 и укажите **Тип утверждения привязки** для типа входящего утверждения.</span><span class="sxs-lookup"><span data-stu-id="89551-219">Repeat steps 2 - 7, and specify **Anchor Claim Type** for the incoming claim type.</span></span>
9. <span data-ttu-id="89551-220">Нажмите кнопку **ОК** , чтобы завершить работу мастера.</span><span class="sxs-lookup"><span data-stu-id="89551-220">Click **OK** to complete the wizard.</span></span>

### <a name="enable-home-realm-discovery"></a><span data-ttu-id="89551-221">Включение обнаружения домашней области</span><span class="sxs-lookup"><span data-stu-id="89551-221">Enable home-realm discovery</span></span>

<span data-ttu-id="89551-222">Выполните следующий сценарий PowerShell:</span><span class="sxs-lookup"><span data-stu-id="89551-222">Run the following PowerShell script:</span></span>

```powershell
Set-ADFSClaimsProviderTrust -TargetName "name" -OrganizationalAccountSuffix @("suffix")
```

<span data-ttu-id="89551-223">где "name" — понятное имя доверия поставщика утверждений, а "suffix" — суффикс UPN для AD клиента (например "corp.fabrikam.com").</span><span class="sxs-lookup"><span data-stu-id="89551-223">where "name" is the friendly name of the claims provider trust, and "suffix" is the UPN suffix for the customer's AD (example, "corp.fabrikam.com").</span></span>

<span data-ttu-id="89551-224">Эта конфигурация позволяет пользователям вводить данные рабочей учетной записи, для которой AD FS автоматически выберет соответствующий поставщик утверждений.</span><span class="sxs-lookup"><span data-stu-id="89551-224">With this configuration, end users can type in their organizational account, and AD FS automatically selects the corresponding claims provider.</span></span> <span data-ttu-id="89551-225">См. статью о [Настройка страниц входа AD FS] и раздел "Настройка поставщика удостоверений для использования определенных суффиксов электронной почты".</span><span class="sxs-lookup"><span data-stu-id="89551-225">See [Customizing the AD FS Sign-in Pages], under the section "Configure Identity Provider to use certain email suffixes".</span></span>

## <a name="configure-the-ad-fs-account-partner"></a><span data-ttu-id="89551-226">Настройка партнера по учетным записям AD FS</span><span class="sxs-lookup"><span data-stu-id="89551-226">Configure the AD FS Account Partner</span></span>

<span data-ttu-id="89551-227">Клиент должен выполнить приведенные ниже действия.</span><span class="sxs-lookup"><span data-stu-id="89551-227">The customer must do the following:</span></span>

1. <span data-ttu-id="89551-228">Добавление отношение доверия с проверяющей стороной (RP).</span><span class="sxs-lookup"><span data-stu-id="89551-228">Add a relying party (RP) trust.</span></span>
2. <span data-ttu-id="89551-229">Добавление правил утверждений.</span><span class="sxs-lookup"><span data-stu-id="89551-229">Adds claims rules.</span></span>

### <a name="add-the-rp-trust"></a><span data-ttu-id="89551-230">Добавление отношения доверия с проверяющей стороной</span><span class="sxs-lookup"><span data-stu-id="89551-230">Add the RP trust</span></span>

1. <span data-ttu-id="89551-231">В диспетчере сервера щелкните **Средства** и выберите **Управление AD FS**.</span><span class="sxs-lookup"><span data-stu-id="89551-231">In Server Manager, click **Tools**, and then select **AD FS Management**.</span></span>
2. <span data-ttu-id="89551-232">В дереве консоли в области **AD FS** щелкните правой кнопкой мыши **Отношения доверия проверяющей стороны**.</span><span class="sxs-lookup"><span data-stu-id="89551-232">In the console tree, under **AD FS**, right click **Relying Party Trusts**.</span></span> <span data-ttu-id="89551-233">Выберите **Добавить отношение доверия с проверяющей стороной**.</span><span class="sxs-lookup"><span data-stu-id="89551-233">Select **Add Relying Party Trust**.</span></span>
3. <span data-ttu-id="89551-234">Выберите **Поддерживающие утверждения** и щелкните **Запустить**.</span><span class="sxs-lookup"><span data-stu-id="89551-234">Select **Claims Aware** and click **Start**.</span></span>
4. <span data-ttu-id="89551-235">На странице **Выбор источника данных** выберите параметр "Импорт данных о поставщике утверждений, опубликованных в Интернете или локальной сети".</span><span class="sxs-lookup"><span data-stu-id="89551-235">On the **Select Data Source** page, select the option "Import data about the claims provider published online or on a local network".</span></span> <span data-ttu-id="89551-236">Введите URI конечной точки метаданных федерации поставщика SaaS.</span><span class="sxs-lookup"><span data-stu-id="89551-236">Enter the URI of the SaaS provider's federation metadata endpoint.</span></span>
   <span data-ttu-id="89551-237">![Мастер добавления отношений доверия с проверяющей стороной](./images/add-rp-trust.png)</span><span class="sxs-lookup"><span data-stu-id="89551-237">![Add Relying Party Trust Wizard](./images/add-rp-trust.png)</span></span>
5. <span data-ttu-id="89551-238">На странице **Указание отображаемого имени** введите любое имя.</span><span class="sxs-lookup"><span data-stu-id="89551-238">On the **Specify Display Name** page, enter any name.</span></span>
6. <span data-ttu-id="89551-239">На странице **Выбор политики управления доступом** выберите политику.</span><span class="sxs-lookup"><span data-stu-id="89551-239">On the **Choose Access Control Policy** page, choose a policy.</span></span> <span data-ttu-id="89551-240">Можно выбрать всех пользователей в организации или определенную группу безопасности.</span><span class="sxs-lookup"><span data-stu-id="89551-240">You could permit everyone in the organization, or choose a specific security group.</span></span>
   <span data-ttu-id="89551-241">![Мастер добавления отношений доверия с проверяющей стороной](./images/add-rp-trust2.png)</span><span class="sxs-lookup"><span data-stu-id="89551-241">![Add Relying Party Trust Wizard](./images/add-rp-trust2.png)</span></span>
7. <span data-ttu-id="89551-242">Укажите все необходимые параметры в поле **Политика**.</span><span class="sxs-lookup"><span data-stu-id="89551-242">Enter any parameters required in the **Policy** box.</span></span>
8. <span data-ttu-id="89551-243">Чтобы завершить работу мастера, нажмите кнопку **Далее** .</span><span class="sxs-lookup"><span data-stu-id="89551-243">Click **Next** to complete the wizard.</span></span>

### <a name="add-claims-rules"></a><span data-ttu-id="89551-244">Добавление правил утверждений</span><span class="sxs-lookup"><span data-stu-id="89551-244">Add claims rules</span></span>

1. <span data-ttu-id="89551-245">Щелкните правой кнопкой мыши добавленное отношение доверия с проверяющей стороной, а затем выберите команду **Изменить политику выдачи утверждений**.</span><span class="sxs-lookup"><span data-stu-id="89551-245">Right-click the newly added relying party trust, and select **Edit Claim Issuance Policy**.</span></span>
2. <span data-ttu-id="89551-246">Щелкните **Добавить правило**.</span><span class="sxs-lookup"><span data-stu-id="89551-246">Click **Add Rule**.</span></span>
3. <span data-ttu-id="89551-247">Выберите "Отправка атрибутов LDAP в виде утверждений" и нажмите кнопку **Далее**.</span><span class="sxs-lookup"><span data-stu-id="89551-247">Select "Send LDAP Attributes as Claims" and click **Next**.</span></span>
4. <span data-ttu-id="89551-248">Введите имя правила, например "UPN".</span><span class="sxs-lookup"><span data-stu-id="89551-248">Enter a name for the rule, such as "UPN".</span></span>
5. <span data-ttu-id="89551-249">Для параметра **Хранилище атрибутов** выберите значение **Active Directory**.</span><span class="sxs-lookup"><span data-stu-id="89551-249">Under **Attribute store**, select **Active Directory**.</span></span>
   <span data-ttu-id="89551-250">![Мастер добавления правила преобразования утверждений](./images/add-claims-rules.png)</span><span class="sxs-lookup"><span data-stu-id="89551-250">![Add Transform Claim Rule Wizard](./images/add-claims-rules.png)</span></span>
6. <span data-ttu-id="89551-251">В разделе **Сопоставление атрибутов LDAP** :</span><span class="sxs-lookup"><span data-stu-id="89551-251">In the **Mapping of LDAP attributes** section:</span></span>
   * <span data-ttu-id="89551-252">Для параметра **Атрибут LDAP** выберите значение **Имя участника-пользователя**.</span><span class="sxs-lookup"><span data-stu-id="89551-252">Under **LDAP Attribute**, select **User-Principal-Name**.</span></span>
   * <span data-ttu-id="89551-253">В поле **Тип исходящего утверждения** выберите **UPN**.</span><span class="sxs-lookup"><span data-stu-id="89551-253">Under **Outgoing Claim Type**, select **UPN**.</span></span>
     <span data-ttu-id="89551-254">![Мастер добавления правила преобразования утверждений](./images/add-claims-rules2.png)</span><span class="sxs-lookup"><span data-stu-id="89551-254">![Add Transform Claim Rule Wizard](./images/add-claims-rules2.png)</span></span>
7. <span data-ttu-id="89551-255">Нажмите кнопку **Готово**</span><span class="sxs-lookup"><span data-stu-id="89551-255">Click **Finish**.</span></span>
8. <span data-ttu-id="89551-256">Щелкните **Добавить правило** еще раз.</span><span class="sxs-lookup"><span data-stu-id="89551-256">Click **Add Rule** again.</span></span>
9. <span data-ttu-id="89551-257">Выберите "Отправлять утверждения с помощью настраиваемого правила" и нажмите кнопку **Далее**.</span><span class="sxs-lookup"><span data-stu-id="89551-257">Select "Send Claims Using a Custom Rule" and click **Next**.</span></span>
10. <span data-ttu-id="89551-258">Введите имя этого правила, например "Anchor Claim Type" (Тип утверждения привязки).</span><span class="sxs-lookup"><span data-stu-id="89551-258">Enter a name for the rule, such as "Anchor Claim Type".</span></span>
11. <span data-ttu-id="89551-259">В разделе **Настраиваемое правило**введите приведенный ниже код.</span><span class="sxs-lookup"><span data-stu-id="89551-259">Under **Custom rule**, enter the following:</span></span>

    ```console
    EXISTS([Type == "http://schemas.microsoft.com/ws/2014/01/identity/claims/anchorclaimtype"])=>
    issue (Type = "http://schemas.microsoft.com/ws/2014/01/identity/claims/anchorclaimtype",
          Value = "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn");
    ```

    <span data-ttu-id="89551-260">Это правило выдает утверждение типа `anchorclaimtype`.</span><span class="sxs-lookup"><span data-stu-id="89551-260">This rule issues a claim of type `anchorclaimtype`.</span></span> <span data-ttu-id="89551-261">Такое утверждение указывает проверяющей стороне, что в качестве неизменяемого идентификатора пользователя нужно использовать имя участника-пользователя.</span><span class="sxs-lookup"><span data-stu-id="89551-261">The claim tells the relying party to use UPN as the user's immutable ID.</span></span>
12. <span data-ttu-id="89551-262">Нажмите кнопку **Готово**</span><span class="sxs-lookup"><span data-stu-id="89551-262">Click **Finish**.</span></span>
13. <span data-ttu-id="89551-263">Нажмите кнопку **ОК** , чтобы завершить работу мастера.</span><span class="sxs-lookup"><span data-stu-id="89551-263">Click **OK** to complete the wizard.</span></span>

<!-- links -->

[Azure AD Connect]: /azure/active-directory/hybrid/whatis-hybrid-identity
[доверие федерации]: https://technet.microsoft.com/library/cc770993(v=ws.11).aspx
[federation trust]: https://technet.microsoft.com/library/cc770993(v=ws.11).aspx
[партнером по учетным записям]: https://technet.microsoft.com/library/cc731141(v=ws.11).aspx
[account partner]: https://technet.microsoft.com/library/cc731141(v=ws.11).aspx
[партнером по учетным запися]: https://technet.microsoft.com/library/cc731141(v=ws.11).aspx
[resource partner]: https://technet.microsoft.com/library/cc731141(v=ws.11).aspx
[Время выполнения проверки подлинности]: https://msdn.microsoft.com/library/system.security.claims.claimtypes.authenticationinstant%28v=vs.110%29.aspx
[Authentication instant]: https://msdn.microsoft.com/library/system.security.claims.claimtypes.authenticationinstant%28v=vs.110%29.aspx
[Время окончания срока действия]: https://tools.ietf.org/html/draft-ietf-oauth-json-web-token-25#section-4.1.
[Expiration time]: https://tools.ietf.org/html/draft-ietf-oauth-json-web-token-25#section-4.1.
[Идентификатор имени]: https://msdn.microsoft.com/library/system.security.claims.claimtypes.nameidentifier(v=vs.110).aspx
[Name identifier]: https://msdn.microsoft.com/library/system.security.claims.claimtypes.nameidentifier(v=vs.110).aspx
[active-directory-on-azure]: https://msdn.microsoft.com/library/azure/jj156090.aspx
[записи блога]: https://www.cloudidentity.com/blog/2015/08/21/OPENID-CONNECT-WEB-SIGN-ON-WITH-ADFS-IN-WINDOWS-SERVER-2016-TP3/
[blog post]: https://www.cloudidentity.com/blog/2015/08/21/OPENID-CONNECT-WEB-SIGN-ON-WITH-ADFS-IN-WINDOWS-SERVER-2016-TP3/
[Настройка страниц входа AD FS]: https://technet.microsoft.com/library/dn280950.aspx
[Customizing the AD FS Sign-in Pages]: https://technet.microsoft.com/library/dn280950.aspx
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[client assertion]: client-assertion.md
[active-directory-dotnet-webapp-wsfederation]: https://github.com/Azure-Samples/active-directory-dotnet-webapp-wsfederation
