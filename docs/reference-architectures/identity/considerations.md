---
title: "Выберите решение для интеграции локальной среды Active Directory с Azure."
description: "Здесь сравниваются эталонные архитектуры для интеграции локальной среды Active Directory с Azure."
ms.date: 04/06/2017
ms.openlocfilehash: 413a5463d90547197c4b6834d353b4ecf61483ee
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/14/2017
---
# <a name="choose-a-solution-for-integrating-on-premises-active-directory-with-azure"></a><span data-ttu-id="9c741-103">Выбор решения для интеграции локальной среды Active Directory с Azure</span><span class="sxs-lookup"><span data-stu-id="9c741-103">Choose a solution for integrating on-premises Active Directory with Azure</span></span>

<span data-ttu-id="9c741-104">В данной статье сравниваются варианты интеграции локальной среды Active Directory (AD) с сетью Azure.</span><span class="sxs-lookup"><span data-stu-id="9c741-104">This article compares options for integrating your on-premises Active Directory (AD) environment with an Azure network.</span></span> <span data-ttu-id="9c741-105">Для каждого варианта мы предоставляем эталонную архитектуру и развертываемое решение.</span><span class="sxs-lookup"><span data-stu-id="9c741-105">We provide a reference architecture and a deployable solution for each option.</span></span>

<span data-ttu-id="9c741-106">Во множестве организаций [доменные службы Active Directory (AD DS)][active-directory-domain-services] используются для аутентификации удостоверений, связанных с пользователями, компьютерами, приложениями или другими ресурсами, включенными в периметр безопасности.</span><span class="sxs-lookup"><span data-stu-id="9c741-106">Many organizations use [Active Directory Domain Services (AD DS)][active-directory-domain-services] to authenticate identities associated with users, computers, applications, or other resources that are included in a security boundary.</span></span> <span data-ttu-id="9c741-107">Как правило, службы каталогов и удостоверений размещаются в локальной среде, но если приложение размещено частично локально и частично в Azure, при отправке запросов на аутентификацию от Azure в локальную среду может возникнуть задержка.</span><span class="sxs-lookup"><span data-stu-id="9c741-107">Directory and identity services are typically hosted on-premises, but if your application is hosted partly on-premises and partly in Azure, there may be latency sending authentication requests from Azure back to on-premises.</span></span> <span data-ttu-id="9c741-108">Внедрение службы каталогов и удостоверений в Azure может сократить ее.</span><span class="sxs-lookup"><span data-stu-id="9c741-108">Implementing directory and identity services in Azure can reduce this latency.</span></span>

<span data-ttu-id="9c741-109">В Azure предусмотрено два решения для внедрения служб каталогов и удостоверений:</span><span class="sxs-lookup"><span data-stu-id="9c741-109">Azure provides two solutions for implementing directory and identity services in Azure:</span></span> 

* <span data-ttu-id="9c741-110">[Azure AD][azure-active-directory] для создания домена Active Directory в облаке c последующим его подключением к домену Active Directory в локальной среде.</span><span class="sxs-lookup"><span data-stu-id="9c741-110">Use [Azure AD][azure-active-directory] to create an Active Directory domain in the cloud and connect it to your on-premises Active Directory domain.</span></span> <span data-ttu-id="9c741-111">[Azure AD Connect][azure-ad-connect] для интеграции локальных каталогов с Azure AD.</span><span class="sxs-lookup"><span data-stu-id="9c741-111">[Azure AD Connect][azure-ad-connect] integrates your on-premises directories with Azure AD.</span></span>

* <span data-ttu-id="9c741-112">Расширьте имеющуюся инфраструктуру локальной среды Active Directory в Azure, развернув виртуальную машину, на которой выполняются AD DS, как контроллер домена.</span><span class="sxs-lookup"><span data-stu-id="9c741-112">Extend your existing on-premises Active Directory infrastructure to Azure, by deploying a VM in Azure that runs AD DS as a domain controller.</span></span> <span data-ttu-id="9c741-113">Эта архитектура чаще используется при подключении к локальной и виртуальной сети Azure по VPN или ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="9c741-113">This architecture is more common when the on-premises network and the Azure virtual network (VNet) are connected by a VPN or ExpressRoute connection.</span></span> <span data-ttu-id="9c741-114">Существует несколько вариантов этой архитектуры:</span><span class="sxs-lookup"><span data-stu-id="9c741-114">Several variations of this architecture are possible:</span></span> 

    - <span data-ttu-id="9c741-115">Создайте домен в Azure и присоедините его к своему локальному лесу AD.</span><span class="sxs-lookup"><span data-stu-id="9c741-115">Create a domain in Azure and join it to your on-premises AD forest.</span></span>
    - <span data-ttu-id="9c741-116">Создайте в локальном лесу отдельный лес в Azure, который является доверенным для доменов.</span><span class="sxs-lookup"><span data-stu-id="9c741-116">Create a separate forest in Azure that is trusted by domains in your on-premises forest.</span></span>
    - <span data-ttu-id="9c741-117">Реплицируйте развернутые службы федерации Active Directory (AD FS) в Azure.</span><span class="sxs-lookup"><span data-stu-id="9c741-117">Replicate an Active Directory Federation Services (AD FS) deployment to Azure.</span></span> 

<span data-ttu-id="9c741-118">В следующих разделах более подробно описан каждый из этих вариантов.</span><span class="sxs-lookup"><span data-stu-id="9c741-118">The next sections describe each of these options in more detail.</span></span>

## <a name="integrate-your-on-premises-domains-with-azure-ad"></a><span data-ttu-id="9c741-119">Интеграция локальных доменов с Azure AD</span><span class="sxs-lookup"><span data-stu-id="9c741-119">Integrate your on-premises domains with Azure AD</span></span>

<span data-ttu-id="9c741-120">Используйте Azure Active Directory (Azure AD) для создания домена в Azure и его привязки к локальному домену AD.</span><span class="sxs-lookup"><span data-stu-id="9c741-120">Use Azure Active Directory (Azure AD) to create a domain in Azure and link it to an on-premises AD domain.</span></span> 

<span data-ttu-id="9c741-121">Каталог Azure AD не является расширением локального каталога.</span><span class="sxs-lookup"><span data-stu-id="9c741-121">The Azure AD directory is not an extension of an on-premises directory.</span></span> <span data-ttu-id="9c741-122">Скорее, это копия с теми же объектами и удостоверениями.</span><span class="sxs-lookup"><span data-stu-id="9c741-122">Rather, it's a copy that contains the same objects and identities.</span></span> <span data-ttu-id="9c741-123">Изменения, внесенные в эти элементы в локальной среде, копируются в Azure AD, но изменения, внесенные в Azure AD, не реплицируются обратно в локальный домен.</span><span class="sxs-lookup"><span data-stu-id="9c741-123">Changes made to these items on-premises are copied to Azure AD, but changes made in Azure AD are not replicated back to the on-premises domain.</span></span>

<span data-ttu-id="9c741-124">Кроме того, можно использовать Azure AD, не используя локальный каталог.</span><span class="sxs-lookup"><span data-stu-id="9c741-124">You can also use Azure AD without using an on-premises directory.</span></span> <span data-ttu-id="9c741-125">При этом Azure AD служит основным источником всех данных идентификации и не содержит данные, полученные при репликации из локального каталога.</span><span class="sxs-lookup"><span data-stu-id="9c741-125">In this case, Azure AD acts as the primary source of all identity information, rather than containing data replicated from an on-premises directory.</span></span>


<span data-ttu-id="9c741-126">**Преимущества**</span><span class="sxs-lookup"><span data-stu-id="9c741-126">**Benefits**</span></span>

* <span data-ttu-id="9c741-127">Отсутствие необходимости в поддержке инфраструктуры AD в облаке.</span><span class="sxs-lookup"><span data-stu-id="9c741-127">You don't need to maintain an AD infrastructure in the cloud.</span></span> <span data-ttu-id="9c741-128">Полная поддержка и управление Azure AD корпорацией Майкрософт.</span><span class="sxs-lookup"><span data-stu-id="9c741-128">Azure AD is entirely managed and maintained by Microsoft.</span></span>
* <span data-ttu-id="9c741-129">Предоставление в Azure AD тех же сведений об идентификаторах, что и доступны локально.</span><span class="sxs-lookup"><span data-stu-id="9c741-129">Azure AD provides the same identity information that is available on-premises.</span></span>
* <span data-ttu-id="9c741-130">Аутентификация выполняется в Azure. При этом внешним приложениям и пользователям не нужно использовать локальный домен.</span><span class="sxs-lookup"><span data-stu-id="9c741-130">Authentication can happen in Azure, reducing the need for external applications and users to contact the on-premises domain.</span></span>

<span data-ttu-id="9c741-131">**Сложности**</span><span class="sxs-lookup"><span data-stu-id="9c741-131">**Challenges**</span></span>

* <span data-ttu-id="9c741-132">Ограничение служб удостоверений пользователями и группами.</span><span class="sxs-lookup"><span data-stu-id="9c741-132">Identity services are limited to users and groups.</span></span> <span data-ttu-id="9c741-133">Отсутствие возможности аутентификации учетных записей служб и компьютеров.</span><span class="sxs-lookup"><span data-stu-id="9c741-133">There is no ability to authenticate service and computer accounts.</span></span>
* <span data-ttu-id="9c741-134">Необходимость настройки соединения с локальным доменом для синхронизации каталога Azure AD.</span><span class="sxs-lookup"><span data-stu-id="9c741-134">You must configure connectivity with your on-premises domain to keep the Azure AD directory synchronized.</span></span> 
* <span data-ttu-id="9c741-135">Необходимость перезаписи приложения с помощью Azure AD для включения аутентификации.</span><span class="sxs-lookup"><span data-stu-id="9c741-135">Applications may need to be rewritten to enable authentication through Azure AD.</span></span>

<span data-ttu-id="9c741-136">**[Подробнее][aad]**</span><span class="sxs-lookup"><span data-stu-id="9c741-136">**[Read more...][aad]**</span></span>

## <a name="ad-ds-in-azure-joined-to-an-on-premises-forest"></a><span data-ttu-id="9c741-137">Присоединение AD DS в Azure к локальному лесу</span><span class="sxs-lookup"><span data-stu-id="9c741-137">AD DS in Azure joined to an on-premises forest</span></span>

<span data-ttu-id="9c741-138">Разверните серверы доменных служб AD (AD DS) в Azure.</span><span class="sxs-lookup"><span data-stu-id="9c741-138">Deploy AD Domain Services (AD DS) servers to Azure.</span></span> <span data-ttu-id="9c741-139">Создайте домен в Azure и присоедините его к своему локальному лесу AD.</span><span class="sxs-lookup"><span data-stu-id="9c741-139">Create a domain in Azure and join it to your on-premises AD forest.</span></span> 

<span data-ttu-id="9c741-140">При использовании функций AD DS, которые сейчас не реализованы в Azure AD, мы рекомендуем использовать этот вариант.</span><span class="sxs-lookup"><span data-stu-id="9c741-140">Consider this option if you need to use AD DS features that are not currently implemented by Azure AD.</span></span> 

<span data-ttu-id="9c741-141">**Преимущества**</span><span class="sxs-lookup"><span data-stu-id="9c741-141">**Benefits**</span></span>

* <span data-ttu-id="9c741-142">Предоставление тех же сведений об идентификаторах, что и доступны локально.</span><span class="sxs-lookup"><span data-stu-id="9c741-142">Provides access to the same identity information that is available on-premises.</span></span>
* <span data-ttu-id="9c741-143">Аутентификация учетных записей пользователя, службы и компьютеров в Azure.</span><span class="sxs-lookup"><span data-stu-id="9c741-143">You can authenticate user, service, and computer accounts on-premises and in Azure.</span></span>
* <span data-ttu-id="9c741-144">Отсутствие необходимости в управлении отдельным лесом AD.</span><span class="sxs-lookup"><span data-stu-id="9c741-144">You don't need to manage a separate AD forest.</span></span> <span data-ttu-id="9c741-145">Принадлежность домена в Azure к локальному лесу.</span><span class="sxs-lookup"><span data-stu-id="9c741-145">The domain in Azure can belong to the on-premises forest.</span></span>
* <span data-ttu-id="9c741-146">Применение групповой политики, определяемой локальными объектами групповой политики, к домену в Azure.</span><span class="sxs-lookup"><span data-stu-id="9c741-146">You can apply group policy defined by on-premises Group Policy Objects to the domain in Azure.</span></span>

<span data-ttu-id="9c741-147">**Сложности**</span><span class="sxs-lookup"><span data-stu-id="9c741-147">**Challenges**</span></span>

* <span data-ttu-id="9c741-148">Необходимость в самостоятельном развертывании серверов и домена AD DS в облаке и управлении ими.</span><span class="sxs-lookup"><span data-stu-id="9c741-148">You must deploy and manage your own AD DS servers and domain in the cloud.</span></span>
* <span data-ttu-id="9c741-149">Задержка синхронизации между серверами домена в облаке и локальными серверами.</span><span class="sxs-lookup"><span data-stu-id="9c741-149">There may be some synchronization latency between the domain servers in the cloud and the servers running on-premises.</span></span>

<span data-ttu-id="9c741-150">**[Подробнее][ad-ds]**</span><span class="sxs-lookup"><span data-stu-id="9c741-150">**[Read more...][ad-ds]**</span></span>

## <a name="ad-ds-in-azure-with-a-separate-forest"></a><span data-ttu-id="9c741-151">AD DS в Azure с отдельным лесом</span><span class="sxs-lookup"><span data-stu-id="9c741-151">AD DS in Azure with a separate forest</span></span>

<span data-ttu-id="9c741-152">Создайте [лес][ad-forest-defn] Active Directory, отделенный от локального, при развертывании серверов доменных служб AD (AD DS) в Azure.</span><span class="sxs-lookup"><span data-stu-id="9c741-152">Deploy AD Domain Services (AD DS) servers to Azure, but create a separate Active Directory [forest][ad-forest-defn] that is separate from the on-premises forest.</span></span> <span data-ttu-id="9c741-153">Этот лес является доверенным для доменов в локальном лесу.</span><span class="sxs-lookup"><span data-stu-id="9c741-153">This forest is trusted by domains in your on-premises forest.</span></span>

<span data-ttu-id="9c741-154">Типичные способы применения этой архитектуры включают поддержку разделения по безопасности для облачных объектов и удостоверений, а также перенос отдельных доменов из локальной среды в облако.</span><span class="sxs-lookup"><span data-stu-id="9c741-154">Typical uses for this architecture include maintaining security separation for objects and identities held in the cloud, and migrating individual domains from on-premises to the cloud.</span></span>

<span data-ttu-id="9c741-155">**Преимущества**</span><span class="sxs-lookup"><span data-stu-id="9c741-155">**Benefits**</span></span>

* <span data-ttu-id="9c741-156">Реализация локальных удостоверений и отдельных удостоверений, предназначенных только для Azure.</span><span class="sxs-lookup"><span data-stu-id="9c741-156">You can implement on-premises identities and separate Azure-only identities.</span></span>
* <span data-ttu-id="9c741-157">Отсутствие необходимости в репликации из локального леса AD в Azure.</span><span class="sxs-lookup"><span data-stu-id="9c741-157">You don't need to replicate from the on-premises AD forest to Azure.</span></span>

<span data-ttu-id="9c741-158">**Сложности**</span><span class="sxs-lookup"><span data-stu-id="9c741-158">**Challenges**</span></span>

* <span data-ttu-id="9c741-159">Необходимость дополнительных сетевых переходов для локальных серверов AD при аутентификации в Azure для локальных удостоверений.</span><span class="sxs-lookup"><span data-stu-id="9c741-159">Authentication within Azure for on-premises identities requires extra network hops to the on-premises AD servers.</span></span>
* <span data-ttu-id="9c741-160">Необходимость в развертывании собственных серверов и лесов AD DS в облаке, а также в настройке соответствующих отношений доверия между лесами.</span><span class="sxs-lookup"><span data-stu-id="9c741-160">You must deploy your own AD DS servers and forest in the cloud, and establish the appropriate trust relationships between forests.</span></span>

<span data-ttu-id="9c741-161">**[Подробнее][ad-ds-forest]**</span><span class="sxs-lookup"><span data-stu-id="9c741-161">**[Read more...][ad-ds-forest]**</span></span>

## <a name="extend-ad-fs-to-azure"></a><span data-ttu-id="9c741-162">Расширение AD FS в Azure</span><span class="sxs-lookup"><span data-stu-id="9c741-162">Extend AD FS to Azure</span></span>

<span data-ttu-id="9c741-163">Реплицируйте развернутые службы федерации Active Directory (AD FS) в Azure для выполнения федеративной аутентификации и авторизации для компонентов, работающих в Azure.</span><span class="sxs-lookup"><span data-stu-id="9c741-163">Replicate an Active Directory Federation Services (AD FS) deployment to Azure, to perform federated authentication and authorization for components running in Azure.</span></span> 

<span data-ttu-id="9c741-164">Типичные способы использования этой архитектуры:</span><span class="sxs-lookup"><span data-stu-id="9c741-164">Typical uses for this architecture:</span></span>

* <span data-ttu-id="9c741-165">Аутентификация и авторизация пользователей из партнерских организаций.</span><span class="sxs-lookup"><span data-stu-id="9c741-165">Authenticate and authorize users from partner organizations.</span></span>
* <span data-ttu-id="9c741-166">Предоставление пользователям возможности прохождения аутентификации в веб-браузерах за пределами брандмауэра организации.</span><span class="sxs-lookup"><span data-stu-id="9c741-166">Allow users to authenticate from web browsers running outside of the organizational firewall.</span></span>
* <span data-ttu-id="9c741-167">Предоставление пользователям возможности подключения с помощью авторизованных внешних устройств, таких как мобильные устройства.</span><span class="sxs-lookup"><span data-stu-id="9c741-167">Allow users to connect from authorized external devices such as mobile devices.</span></span> 

<span data-ttu-id="9c741-168">**Преимущества**</span><span class="sxs-lookup"><span data-stu-id="9c741-168">**Benefits**</span></span>

* <span data-ttu-id="9c741-169">Использование приложений с поддержкой утверждений.</span><span class="sxs-lookup"><span data-stu-id="9c741-169">You can leverage claims-aware applications.</span></span>
* <span data-ttu-id="9c741-170">Настройка отношений доверия с внешними партнерами при аутентификации.</span><span class="sxs-lookup"><span data-stu-id="9c741-170">Provides the ability to trust external partners for authentication.</span></span>
* <span data-ttu-id="9c741-171">Совместимость со множеством протоколов аутентификации.</span><span class="sxs-lookup"><span data-stu-id="9c741-171">Compatibility with large set of authentication protocols.</span></span>

<span data-ttu-id="9c741-172">**Сложности**</span><span class="sxs-lookup"><span data-stu-id="9c741-172">**Challenges**</span></span>

* <span data-ttu-id="9c741-173">Необходимость в развертывании собственных прокси-серверов веб-приложения AD DS и AD FS в Azure.</span><span class="sxs-lookup"><span data-stu-id="9c741-173">You must deploy your own AD DS, AD FS, and AD FS Web Application Proxy servers in Azure.</span></span>
* <span data-ttu-id="9c741-174">Эту архитектуру сложно настроить.</span><span class="sxs-lookup"><span data-stu-id="9c741-174">This architecture can be complex to configure.</span></span>

<span data-ttu-id="9c741-175">**[Подробнее][adfs]**</span><span class="sxs-lookup"><span data-stu-id="9c741-175">**[Read more...][adfs]**</span></span>

<!-- links -->

[aad]: ./azure-ad.md
[ad-ds]: ./adds-extend-domain.md
[ad-ds-forest]: ./adds-forest.md
[ad-forest-defn]: https://msdn.microsoft.com/library/ms676906.aspx
[adfs]: ./adfs.md

[active-directory-domain-services]: https://technet.microsoft.com/library/dd448614.aspx
[azure-active-directory]: /azure/active-directory-domain-services/active-directory-ds-overview
[azure-ad-connect]: /azure/active-directory/active-directory-aadconnect
