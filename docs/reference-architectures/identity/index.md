---
title: Интеграция локальной службы Active Directory с Azure
titleSuffix: Azure Reference Architectures
description: Сравнение эталонных архитектур для интеграции локальной службы Active Directory с Azure.
ms.date: 07/02/2018
ms.custom: seodec18
ms.openlocfilehash: 99a64f0a5fbe5624aa8ad05bd3565ab2aef618b3
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/04/2019
ms.locfileid: "54011792"
---
# <a name="choose-a-solution-for-integrating-on-premises-active-directory-with-azure"></a><span data-ttu-id="da99e-103">Выбор решения для интеграции локальной среды Active Directory с Azure</span><span class="sxs-lookup"><span data-stu-id="da99e-103">Choose a solution for integrating on-premises Active Directory with Azure</span></span>

<span data-ttu-id="da99e-104">В данной статье сравниваются варианты интеграции локальной среды Active Directory (AD) с сетью Azure.</span><span class="sxs-lookup"><span data-stu-id="da99e-104">This article compares options for integrating your on-premises Active Directory (AD) environment with an Azure network.</span></span> <span data-ttu-id="da99e-105">Для каждого варианта доступна подробная эталонная архитектура.</span><span class="sxs-lookup"><span data-stu-id="da99e-105">For each option, a more detailed reference architecture is available.</span></span>

<span data-ttu-id="da99e-106">Во многих организациях доменные службы Active Directory (AD DS) используются для проверки подлинности удостоверений, связанных с пользователями, компьютерами, приложениями или другими ресурсами, включенными в периметр безопасности.</span><span class="sxs-lookup"><span data-stu-id="da99e-106">Many organizations use Active Directory Domain Services (AD DS) to authenticate identities associated with users, computers, applications, or other resources that are included in a security boundary.</span></span> <span data-ttu-id="da99e-107">Как правило, службы каталогов и удостоверений размещаются в локальной среде, но если приложение размещено частично локально и частично в Azure, при отправке запросов на аутентификацию от Azure в локальную среду может возникнуть задержка.</span><span class="sxs-lookup"><span data-stu-id="da99e-107">Directory and identity services are typically hosted on-premises, but if your application is hosted partly on-premises and partly in Azure, there may be latency sending authentication requests from Azure back to on-premises.</span></span> <span data-ttu-id="da99e-108">Внедрение службы каталогов и удостоверений в Azure может сократить ее.</span><span class="sxs-lookup"><span data-stu-id="da99e-108">Implementing directory and identity services in Azure can reduce this latency.</span></span>

<span data-ttu-id="da99e-109">В Azure предусмотрено два решения для внедрения служб каталогов и удостоверений:</span><span class="sxs-lookup"><span data-stu-id="da99e-109">Azure provides two solutions for implementing directory and identity services in Azure:</span></span>

- <span data-ttu-id="da99e-110">[Azure AD][azure-active-directory] для создания домена Active Directory в облаке c последующим его подключением к домену Active Directory в локальной среде.</span><span class="sxs-lookup"><span data-stu-id="da99e-110">Use [Azure AD][azure-active-directory] to create an Active Directory domain in the cloud and connect it to your on-premises Active Directory domain.</span></span> <span data-ttu-id="da99e-111">[Azure AD Connect][azure-ad-connect] для интеграции локальных каталогов с Azure AD.</span><span class="sxs-lookup"><span data-stu-id="da99e-111">[Azure AD Connect][azure-ad-connect] integrates your on-premises directories with Azure AD.</span></span>

- <span data-ttu-id="da99e-112">Расширьте имеющуюся инфраструктуру локальной среды Active Directory в Azure, развернув виртуальную машину, на которой выполняются AD DS, как контроллер домена.</span><span class="sxs-lookup"><span data-stu-id="da99e-112">Extend your existing on-premises Active Directory infrastructure to Azure, by deploying a VM in Azure that runs AD DS as a domain controller.</span></span> <span data-ttu-id="da99e-113">Эта архитектура чаще используется при подключении к локальной и виртуальной сети Azure по VPN или ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="da99e-113">This architecture is more common when the on-premises network and the Azure virtual network (VNet) are connected by a VPN or ExpressRoute connection.</span></span> <span data-ttu-id="da99e-114">Существует несколько вариантов этой архитектуры:</span><span class="sxs-lookup"><span data-stu-id="da99e-114">Several variations of this architecture are possible:</span></span>

  - <span data-ttu-id="da99e-115">Создайте домен в Azure и присоедините его к своему локальному лесу AD.</span><span class="sxs-lookup"><span data-stu-id="da99e-115">Create a domain in Azure and join it to your on-premises AD forest.</span></span>
  - <span data-ttu-id="da99e-116">Создайте в локальном лесу отдельный лес в Azure, который является доверенным для доменов.</span><span class="sxs-lookup"><span data-stu-id="da99e-116">Create a separate forest in Azure that is trusted by domains in your on-premises forest.</span></span>
  - <span data-ttu-id="da99e-117">Реплицируйте развернутые службы федерации Active Directory (AD FS) в Azure.</span><span class="sxs-lookup"><span data-stu-id="da99e-117">Replicate an Active Directory Federation Services (AD FS) deployment to Azure.</span></span>

<span data-ttu-id="da99e-118">В следующих разделах более подробно описан каждый из этих вариантов.</span><span class="sxs-lookup"><span data-stu-id="da99e-118">The next sections describe each of these options in more detail.</span></span>

## <a name="integrate-your-on-premises-domains-with-azure-ad"></a><span data-ttu-id="da99e-119">Интеграция локальных доменов с Azure AD</span><span class="sxs-lookup"><span data-stu-id="da99e-119">Integrate your on-premises domains with Azure AD</span></span>

<span data-ttu-id="da99e-120">Используйте Azure Active Directory (Azure AD) для создания домена в Azure и его привязки к локальному домену AD.</span><span class="sxs-lookup"><span data-stu-id="da99e-120">Use Azure Active Directory (Azure AD) to create a domain in Azure and link it to an on-premises AD domain.</span></span>

<span data-ttu-id="da99e-121">Каталог Azure AD не является расширением локального каталога.</span><span class="sxs-lookup"><span data-stu-id="da99e-121">The Azure AD directory is not an extension of an on-premises directory.</span></span> <span data-ttu-id="da99e-122">Скорее, это копия с теми же объектами и удостоверениями.</span><span class="sxs-lookup"><span data-stu-id="da99e-122">Rather, it's a copy that contains the same objects and identities.</span></span> <span data-ttu-id="da99e-123">Изменения, внесенные в эти элементы в локальной среде, копируются в Azure AD, но изменения, внесенные в Azure AD, не реплицируются обратно в локальный домен.</span><span class="sxs-lookup"><span data-stu-id="da99e-123">Changes made to these items on-premises are copied to Azure AD, but changes made in Azure AD are not replicated back to the on-premises domain.</span></span>

<span data-ttu-id="da99e-124">Кроме того, можно использовать Azure AD, не используя локальный каталог.</span><span class="sxs-lookup"><span data-stu-id="da99e-124">You can also use Azure AD without using an on-premises directory.</span></span> <span data-ttu-id="da99e-125">При этом Azure AD служит основным источником всех данных идентификации и не содержит данные, полученные при репликации из локального каталога.</span><span class="sxs-lookup"><span data-stu-id="da99e-125">In this case, Azure AD acts as the primary source of all identity information, rather than containing data replicated from an on-premises directory.</span></span>

<span data-ttu-id="da99e-126">**Преимущества**</span><span class="sxs-lookup"><span data-stu-id="da99e-126">**Benefits**</span></span>

- <span data-ttu-id="da99e-127">Отсутствие необходимости в поддержке инфраструктуры AD в облаке.</span><span class="sxs-lookup"><span data-stu-id="da99e-127">You don't need to maintain an AD infrastructure in the cloud.</span></span> <span data-ttu-id="da99e-128">Полная поддержка и управление Azure AD корпорацией Майкрософт.</span><span class="sxs-lookup"><span data-stu-id="da99e-128">Azure AD is entirely managed and maintained by Microsoft.</span></span>
- <span data-ttu-id="da99e-129">Предоставление в Azure AD тех же сведений об идентификаторах, что и доступны локально.</span><span class="sxs-lookup"><span data-stu-id="da99e-129">Azure AD provides the same identity information that is available on-premises.</span></span>
- <span data-ttu-id="da99e-130">Аутентификация выполняется в Azure. При этом внешним приложениям и пользователям не нужно использовать локальный домен.</span><span class="sxs-lookup"><span data-stu-id="da99e-130">Authentication can happen in Azure, reducing the need for external applications and users to contact the on-premises domain.</span></span>

<span data-ttu-id="da99e-131">**Сложности**</span><span class="sxs-lookup"><span data-stu-id="da99e-131">**Challenges**</span></span>

- <span data-ttu-id="da99e-132">Ограничение служб удостоверений пользователями и группами.</span><span class="sxs-lookup"><span data-stu-id="da99e-132">Identity services are limited to users and groups.</span></span> <span data-ttu-id="da99e-133">Отсутствие возможности аутентификации учетных записей служб и компьютеров.</span><span class="sxs-lookup"><span data-stu-id="da99e-133">There is no ability to authenticate service and computer accounts.</span></span>
- <span data-ttu-id="da99e-134">Необходимость настройки соединения с локальным доменом для синхронизации каталога Azure AD.</span><span class="sxs-lookup"><span data-stu-id="da99e-134">You must configure connectivity with your on-premises domain to keep the Azure AD directory synchronized.</span></span>
- <span data-ttu-id="da99e-135">Необходимость перезаписи приложения с помощью Azure AD для включения аутентификации.</span><span class="sxs-lookup"><span data-stu-id="da99e-135">Applications may need to be rewritten to enable authentication through Azure AD.</span></span>

<span data-ttu-id="da99e-136">**Эталонная архитектура**</span><span class="sxs-lookup"><span data-stu-id="da99e-136">**Reference architecture**</span></span>

- <span data-ttu-id="da99e-137">[Интеграция локальных доменов Active Directory с Azure Active Directory][aad]</span><span class="sxs-lookup"><span data-stu-id="da99e-137">[Integrate on-premises Active Directory domains with Azure Active Directory][aad]</span></span>

## <a name="ad-ds-in-azure-joined-to-an-on-premises-forest"></a><span data-ttu-id="da99e-138">Присоединение AD DS в Azure к локальному лесу</span><span class="sxs-lookup"><span data-stu-id="da99e-138">AD DS in Azure joined to an on-premises forest</span></span>

<span data-ttu-id="da99e-139">Разверните серверы доменных служб AD (AD DS) в Azure.</span><span class="sxs-lookup"><span data-stu-id="da99e-139">Deploy AD Domain Services (AD DS) servers to Azure.</span></span> <span data-ttu-id="da99e-140">Создайте домен в Azure и присоедините его к своему локальному лесу AD.</span><span class="sxs-lookup"><span data-stu-id="da99e-140">Create a domain in Azure and join it to your on-premises AD forest.</span></span>

<span data-ttu-id="da99e-141">При использовании функций AD DS, которые сейчас не реализованы в Azure AD, мы рекомендуем использовать этот вариант.</span><span class="sxs-lookup"><span data-stu-id="da99e-141">Consider this option if you need to use AD DS features that are not currently implemented by Azure AD.</span></span>

<span data-ttu-id="da99e-142">**Преимущества**</span><span class="sxs-lookup"><span data-stu-id="da99e-142">**Benefits**</span></span>

- <span data-ttu-id="da99e-143">Предоставление тех же сведений об идентификаторах, что и доступны локально.</span><span class="sxs-lookup"><span data-stu-id="da99e-143">Provides access to the same identity information that is available on-premises.</span></span>
- <span data-ttu-id="da99e-144">Аутентификация учетных записей пользователя, службы и компьютеров в Azure.</span><span class="sxs-lookup"><span data-stu-id="da99e-144">You can authenticate user, service, and computer accounts on-premises and in Azure.</span></span>
- <span data-ttu-id="da99e-145">Отсутствие необходимости в управлении отдельным лесом AD.</span><span class="sxs-lookup"><span data-stu-id="da99e-145">You don't need to manage a separate AD forest.</span></span> <span data-ttu-id="da99e-146">Принадлежность домена в Azure к локальному лесу.</span><span class="sxs-lookup"><span data-stu-id="da99e-146">The domain in Azure can belong to the on-premises forest.</span></span>
- <span data-ttu-id="da99e-147">Применение групповой политики, определяемой локальными объектами групповой политики, к домену в Azure.</span><span class="sxs-lookup"><span data-stu-id="da99e-147">You can apply group policy defined by on-premises Group Policy Objects to the domain in Azure.</span></span>

<span data-ttu-id="da99e-148">**Сложности**</span><span class="sxs-lookup"><span data-stu-id="da99e-148">**Challenges**</span></span>

- <span data-ttu-id="da99e-149">Необходимость в самостоятельном развертывании серверов и домена AD DS в облаке и управлении ими.</span><span class="sxs-lookup"><span data-stu-id="da99e-149">You must deploy and manage your own AD DS servers and domain in the cloud.</span></span>
- <span data-ttu-id="da99e-150">Задержка синхронизации между серверами домена в облаке и локальными серверами.</span><span class="sxs-lookup"><span data-stu-id="da99e-150">There may be some synchronization latency between the domain servers in the cloud and the servers running on-premises.</span></span>

<span data-ttu-id="da99e-151">**Эталонная архитектура**</span><span class="sxs-lookup"><span data-stu-id="da99e-151">**Reference architecture**</span></span>

- <span data-ttu-id="da99e-152">[Расширение доменных служб Active Directory (AD DS) в Azure][ad-ds]</span><span class="sxs-lookup"><span data-stu-id="da99e-152">[Extend Active Directory Domain Services (AD DS) to Azure][ad-ds]</span></span>

## <a name="ad-ds-in-azure-with-a-separate-forest"></a><span data-ttu-id="da99e-153">AD DS в Azure с отдельным лесом</span><span class="sxs-lookup"><span data-stu-id="da99e-153">AD DS in Azure with a separate forest</span></span>

<span data-ttu-id="da99e-154">Создайте [лес][ad-forest-defn] Active Directory, отделенный от локального, при развертывании серверов доменных служб AD (AD DS) в Azure.</span><span class="sxs-lookup"><span data-stu-id="da99e-154">Deploy AD Domain Services (AD DS) servers to Azure, but create a separate Active Directory [forest][ad-forest-defn] that is separate from the on-premises forest.</span></span> <span data-ttu-id="da99e-155">Этот лес является доверенным для доменов в локальном лесу.</span><span class="sxs-lookup"><span data-stu-id="da99e-155">This forest is trusted by domains in your on-premises forest.</span></span>

<span data-ttu-id="da99e-156">Типичные способы применения этой архитектуры включают поддержку разделения по безопасности для облачных объектов и удостоверений, а также перенос отдельных доменов из локальной среды в облако.</span><span class="sxs-lookup"><span data-stu-id="da99e-156">Typical uses for this architecture include maintaining security separation for objects and identities held in the cloud, and migrating individual domains from on-premises to the cloud.</span></span>

<span data-ttu-id="da99e-157">**Преимущества**</span><span class="sxs-lookup"><span data-stu-id="da99e-157">**Benefits**</span></span>

- <span data-ttu-id="da99e-158">Реализация локальных удостоверений и отдельных удостоверений, предназначенных только для Azure.</span><span class="sxs-lookup"><span data-stu-id="da99e-158">You can implement on-premises identities and separate Azure-only identities.</span></span>
- <span data-ttu-id="da99e-159">Отсутствие необходимости в репликации из локального леса AD в Azure.</span><span class="sxs-lookup"><span data-stu-id="da99e-159">You don't need to replicate from the on-premises AD forest to Azure.</span></span>

<span data-ttu-id="da99e-160">**Сложности**</span><span class="sxs-lookup"><span data-stu-id="da99e-160">**Challenges**</span></span>

- <span data-ttu-id="da99e-161">Необходимость дополнительных сетевых переходов для локальных серверов AD при аутентификации в Azure для локальных удостоверений.</span><span class="sxs-lookup"><span data-stu-id="da99e-161">Authentication within Azure for on-premises identities requires extra network hops to the on-premises AD servers.</span></span>
- <span data-ttu-id="da99e-162">Необходимость в развертывании собственных серверов и лесов AD DS в облаке, а также в настройке соответствующих отношений доверия между лесами.</span><span class="sxs-lookup"><span data-stu-id="da99e-162">You must deploy your own AD DS servers and forest in the cloud, and establish the appropriate trust relationships between forests.</span></span>

<span data-ttu-id="da99e-163">**Эталонная архитектура**</span><span class="sxs-lookup"><span data-stu-id="da99e-163">**Reference architecture**</span></span>

- <span data-ttu-id="da99e-164">[Создание леса ресурсов доменных служб Active Directory (AD DS) в Azure][ad-ds-forest]</span><span class="sxs-lookup"><span data-stu-id="da99e-164">[Create an Active Directory Domain Services (AD DS) resource forest in Azure][ad-ds-forest]</span></span>

## <a name="extend-ad-fs-to-azure"></a><span data-ttu-id="da99e-165">Расширение AD FS в Azure</span><span class="sxs-lookup"><span data-stu-id="da99e-165">Extend AD FS to Azure</span></span>

<span data-ttu-id="da99e-166">Реплицируйте развернутые службы федерации Active Directory (AD FS) в Azure для выполнения федеративной аутентификации и авторизации для компонентов, работающих в Azure.</span><span class="sxs-lookup"><span data-stu-id="da99e-166">Replicate an Active Directory Federation Services (AD FS) deployment to Azure, to perform federated authentication and authorization for components running in Azure.</span></span>

<span data-ttu-id="da99e-167">Типичные способы использования этой архитектуры:</span><span class="sxs-lookup"><span data-stu-id="da99e-167">Typical uses for this architecture:</span></span>

- <span data-ttu-id="da99e-168">Аутентификация и авторизация пользователей из партнерских организаций.</span><span class="sxs-lookup"><span data-stu-id="da99e-168">Authenticate and authorize users from partner organizations.</span></span>
- <span data-ttu-id="da99e-169">Предоставление пользователям возможности прохождения аутентификации в веб-браузерах за пределами брандмауэра организации.</span><span class="sxs-lookup"><span data-stu-id="da99e-169">Allow users to authenticate from web browsers running outside of the organizational firewall.</span></span>
- <span data-ttu-id="da99e-170">Предоставление пользователям возможности подключения с помощью авторизованных внешних устройств, таких как мобильные устройства.</span><span class="sxs-lookup"><span data-stu-id="da99e-170">Allow users to connect from authorized external devices such as mobile devices.</span></span>

<span data-ttu-id="da99e-171">**Преимущества**</span><span class="sxs-lookup"><span data-stu-id="da99e-171">**Benefits**</span></span>

- <span data-ttu-id="da99e-172">Использование приложений с поддержкой утверждений.</span><span class="sxs-lookup"><span data-stu-id="da99e-172">You can leverage claims-aware applications.</span></span>
- <span data-ttu-id="da99e-173">Настройка отношений доверия с внешними партнерами при аутентификации.</span><span class="sxs-lookup"><span data-stu-id="da99e-173">Provides the ability to trust external partners for authentication.</span></span>
- <span data-ttu-id="da99e-174">Совместимость со множеством протоколов аутентификации.</span><span class="sxs-lookup"><span data-stu-id="da99e-174">Compatibility with large set of authentication protocols.</span></span>

<span data-ttu-id="da99e-175">**Сложности**</span><span class="sxs-lookup"><span data-stu-id="da99e-175">**Challenges**</span></span>

- <span data-ttu-id="da99e-176">Необходимость в развертывании собственных прокси-серверов веб-приложения AD DS и AD FS в Azure.</span><span class="sxs-lookup"><span data-stu-id="da99e-176">You must deploy your own AD DS, AD FS, and AD FS Web Application Proxy servers in Azure.</span></span>
- <span data-ttu-id="da99e-177">Эту архитектуру сложно настроить.</span><span class="sxs-lookup"><span data-stu-id="da99e-177">This architecture can be complex to configure.</span></span>

<span data-ttu-id="da99e-178">**Эталонная архитектура**</span><span class="sxs-lookup"><span data-stu-id="da99e-178">**Reference architecture**</span></span>

- <span data-ttu-id="da99e-179">[Расширение служб федерации Active Directory (AD FS) в Azure][adfs]</span><span class="sxs-lookup"><span data-stu-id="da99e-179">[Extend Active Directory Federation Services (AD FS) to Azure][adfs]</span></span>

<!-- links -->

[aad]: ./azure-ad.md
[ad-ds]: ./adds-extend-domain.md
[ad-ds-forest]: ./adds-forest.md
[ad-forest-defn]: /windows/desktop/AD/forests
[adfs]: ./adfs.md

[azure-active-directory]: /azure/active-directory-domain-services/active-directory-ds-overview
[azure-ad-connect]: /azure/active-directory/hybrid/whatis-hybrid-identity
