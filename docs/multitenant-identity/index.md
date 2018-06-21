---
title: Управление удостоверениями для мультитенантных приложений
description: Рекомендации по аутентификации, авторизации и управлению удостоверениями в мультитенантных приложениях.
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.next: tailspin
ms.openlocfilehash: c363ac01e798b522fa95f39586e28fe3af5fae4a
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/14/2017
ms.locfileid: "24541678"
---
# <a name="manage-identity-in-multitenant-applications"></a><span data-ttu-id="a71b3-103">Управление удостоверением в мультитенантных приложениях</span><span class="sxs-lookup"><span data-stu-id="a71b3-103">Manage Identity in Multitenant Applications</span></span>

<span data-ttu-id="a71b3-104">В этом цикле статей описываются рекомендации по архитектуре обслуживания одним экземпляром приложения нескольких развертываний, когда Azure AD используется для аутентификации и управления удостоверениями.</span><span class="sxs-lookup"><span data-stu-id="a71b3-104">This series of articles describes best practices for multitenancy, when using Azure AD for authentication and identity management.</span></span>

<span data-ttu-id="a71b3-105">[![GitHub](../_images/github.png) Пример кода][sample application]</span><span class="sxs-lookup"><span data-stu-id="a71b3-105">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="a71b3-106">При создании мультитенантного приложения одной из первейших задач является управление удостоверениями пользователей, так как теперь каждый пользователь относится к клиенту.</span><span class="sxs-lookup"><span data-stu-id="a71b3-106">When you're building a multitenant application, one of the first challenges is managing user identities, because now every user belongs to a tenant.</span></span> <span data-ttu-id="a71b3-107">Например:</span><span class="sxs-lookup"><span data-stu-id="a71b3-107">For example:</span></span>

* <span data-ttu-id="a71b3-108">Пользователи входят с использованием учетных данных организации.</span><span class="sxs-lookup"><span data-stu-id="a71b3-108">Users sign in with their organizational credentials.</span></span>
* <span data-ttu-id="a71b3-109">У пользователей должен быть доступ к их данным в организации, но не к данным, относящимся к другим клиентам.</span><span class="sxs-lookup"><span data-stu-id="a71b3-109">Users should have access to their organization's data, but not data that belongs to other tenants.</span></span>
* <span data-ttu-id="a71b3-110">Организация может зарегистрироваться для использования приложения, а затем назначить роли приложения своим участникам.</span><span class="sxs-lookup"><span data-stu-id="a71b3-110">An organization can sign up for the application, and then assign application roles to its members.</span></span>

<span data-ttu-id="a71b3-111">В Azure Active Directory (Azure AD) имеется несколько удобных функций, которые делают возможными все эти сценарии.</span><span class="sxs-lookup"><span data-stu-id="a71b3-111">Azure Active Directory (Azure AD) has some great features that support all of these scenarios.</span></span>

<span data-ttu-id="a71b3-112">Для этого цикла статей мы создали комплексную [сквозную реализацию][sample application] мультитенантного приложения.</span><span class="sxs-lookup"><span data-stu-id="a71b3-112">To accompany this series of articles, we created a complete [end-to-end implementation][sample application] of a multitenant application.</span></span> <span data-ttu-id="a71b3-113">Статьи отражают то, что мы узнали в процессе создания приложения.</span><span class="sxs-lookup"><span data-stu-id="a71b3-113">The articles reflect what we learned in the process of building the application.</span></span> <span data-ttu-id="a71b3-114">Чтобы приступить к работе с приложением, см. статью [Запуск приложения Surveys][running-the-app].</span><span class="sxs-lookup"><span data-stu-id="a71b3-114">To get started with the application, see [Run the Surveys application][running-the-app].</span></span>

## <a name="introduction"></a><span data-ttu-id="a71b3-115">Введение</span><span class="sxs-lookup"><span data-stu-id="a71b3-115">Introduction</span></span>

<span data-ttu-id="a71b3-116">Предположим, что вы разрабатываете корпоративное приложение SaaS, которое будет размещено в облаке.</span><span class="sxs-lookup"><span data-stu-id="a71b3-116">Let's say you're writing an enterprise SaaS application to be hosted in the cloud.</span></span> <span data-ttu-id="a71b3-117">Конечно, в приложении будут существовать пользователи.</span><span class="sxs-lookup"><span data-stu-id="a71b3-117">Of course, the application will have users:</span></span>

![Пользователи](./images/users.png)

<span data-ttu-id="a71b3-119">Эти пользователи принадлежат организациям.</span><span class="sxs-lookup"><span data-stu-id="a71b3-119">But those users belong to organizations:</span></span>

![Пользователи организации](./images/org-users.png)

<span data-ttu-id="a71b3-121">Пример: Tailspin продает подписки на свое приложение SaaS.</span><span class="sxs-lookup"><span data-stu-id="a71b3-121">Example: Tailspin sells subscriptions to its SaaS application.</span></span> <span data-ttu-id="a71b3-122">Contoso и Fabrikam регистрируются для использования приложения.</span><span class="sxs-lookup"><span data-stu-id="a71b3-122">Contoso and Fabrikam sign up for the app.</span></span> <span data-ttu-id="a71b3-123">Когда Элис (`alice@contoso`) выполняет вход, приложение должно знать, что она является пользователем Contoso.</span><span class="sxs-lookup"><span data-stu-id="a71b3-123">When Alice (`alice@contoso`) signs in, the application should know that Alice is part of Contoso.</span></span>

* <span data-ttu-id="a71b3-124">Элис *должна* иметь доступ к данным компании Contoso.</span><span class="sxs-lookup"><span data-stu-id="a71b3-124">Alice *should* have access to Contoso data.</span></span>
* <span data-ttu-id="a71b3-125">У Элис *не должно* быть доступа к данным компании Fabrikam.</span><span class="sxs-lookup"><span data-stu-id="a71b3-125">Alice *should not* have access to Fabrikam data.</span></span>

<span data-ttu-id="a71b3-126">При помощи этого руководства будет показано, как управлять удостоверениями пользователей в мультитенантном приложении, используя [Azure Active Directory][AzureAD] (Azure AD) для обработки операций входа и аутентификации.</span><span class="sxs-lookup"><span data-stu-id="a71b3-126">This guidance will show you how to manage user identities in a multitenant application, using [Azure Active Directory][AzureAD] (Azure AD) to handle sign-in and authentication.</span></span>

## <a name="what-is-multitenancy"></a><span data-ttu-id="a71b3-127">Что такое мультитенантность?</span><span class="sxs-lookup"><span data-stu-id="a71b3-127">What is multitenancy?</span></span>
<span data-ttu-id="a71b3-128">*Клиент* — это группа пользователей.</span><span class="sxs-lookup"><span data-stu-id="a71b3-128">A *tenant* is a group of users.</span></span> <span data-ttu-id="a71b3-129">В приложении SaaS клиентом является подписчик или заказчик приложения.</span><span class="sxs-lookup"><span data-stu-id="a71b3-129">In a SaaS application, the tenant is a subscriber or customer of the application.</span></span> <span data-ttu-id="a71b3-130">*Мультитенантность* представляет собой архитектуру использования несколькими клиентами одного физического экземпляра приложения.</span><span class="sxs-lookup"><span data-stu-id="a71b3-130">*Multitenancy* is an architecture where multiple tenants share the same physical instance of the app.</span></span> <span data-ttu-id="a71b3-131">Несмотря на то, что клиенты совместно используют физические ресурсы (например виртуальные машины и хранилище данных), каждый клиент получает собственный логический экземпляр приложения.</span><span class="sxs-lookup"><span data-stu-id="a71b3-131">Although tenants share physical resources (such as VMs or storage), each tenant gets its own logical instance of the app.</span></span>

<span data-ttu-id="a71b3-132">Как правило, данные приложения являются общими для пользователей в пределах конкретного клиента, но не в других клиентах.</span><span class="sxs-lookup"><span data-stu-id="a71b3-132">Typically, application data is shared among the users within a tenant, but not with other tenants.</span></span>

![Мультитенантное приложение](./images/multitenant.png)

<span data-ttu-id="a71b3-134">Сравним эту архитектуру с однотенантной архитектурой, где клиент имеет выделенный физический экземпляр.</span><span class="sxs-lookup"><span data-stu-id="a71b3-134">Compare this architecture with a single-tenant architecture, where each tenant has a dedicated physical instance.</span></span> <span data-ttu-id="a71b3-135">В однотенантной архитектуре добавление клиентов осуществляется путем развертывания новых экземпляров приложения.</span><span class="sxs-lookup"><span data-stu-id="a71b3-135">In a single-tenant architecture, you add tenants by spinning up new instances of the app.</span></span>

![Однотенантное приложение](./images/single-tenant.png)

### <a name="multitenancy-and-horizontal-scaling"></a><span data-ttu-id="a71b3-137">Мультитенантность и горизонтальное масштабирование</span><span class="sxs-lookup"><span data-stu-id="a71b3-137">Multitenancy and horizontal scaling</span></span>
<span data-ttu-id="a71b3-138">Масштабирование в облаке чаще всего достигается за счет добавления дополнительных физических экземпляров.</span><span class="sxs-lookup"><span data-stu-id="a71b3-138">To achieve scale in the cloud, it’s common to add more physical instances.</span></span> <span data-ttu-id="a71b3-139">Это известно как *горизонтальное масштабирование* или *развертывание*. Рассмотрим веб-приложение.</span><span class="sxs-lookup"><span data-stu-id="a71b3-139">This is known as *horizontal scaling* or *scaling out*. Consider a web app.</span></span> <span data-ttu-id="a71b3-140">Для обработки большего объема трафика можно добавить дополнительные серверные виртуальные машины и разместить их за балансировщиком нагрузки.</span><span class="sxs-lookup"><span data-stu-id="a71b3-140">To handle more traffic, you can add more server VMs and put them behind a load balancer.</span></span> <span data-ttu-id="a71b3-141">На каждой виртуальной машине запущен отдельный физический экземпляр веб-приложения.</span><span class="sxs-lookup"><span data-stu-id="a71b3-141">Each VM runs a separate physical instance of the web app.</span></span>

![Балансировка нагрузки веб-сайта](./images/load-balancing.png)

<span data-ttu-id="a71b3-143">Запросы могут направляться в любой экземпляр.</span><span class="sxs-lookup"><span data-stu-id="a71b3-143">Any request can be routed to any instance.</span></span> <span data-ttu-id="a71b3-144">Вся система функционирует как единый логический экземпляр.</span><span class="sxs-lookup"><span data-stu-id="a71b3-144">Together, the system functions as a single logical instance.</span></span> <span data-ttu-id="a71b3-145">Процессы удаления ВМ или запуска новой ВМ не нарушают работу пользователей.</span><span class="sxs-lookup"><span data-stu-id="a71b3-145">You can tear down a VM or spin up a new VM, without affecting users.</span></span> <span data-ttu-id="a71b3-146">В этой архитектуре каждый физический экземпляр является мультитенантным, а масштабирование осуществляется путем добавления дополнительных экземпляров.</span><span class="sxs-lookup"><span data-stu-id="a71b3-146">In this architecture, each physical instance is multi-tenant, and you scale by adding more instances.</span></span> <span data-ttu-id="a71b3-147">Если какой-либо экземпляр выходит из строя, он не влияет на клиенты.</span><span class="sxs-lookup"><span data-stu-id="a71b3-147">If one instance goes down, it should not affect any tenant.</span></span>

## <a name="identity-in-a-multitenant-app"></a><span data-ttu-id="a71b3-148">Идентификация в мультитенантном приложении</span><span class="sxs-lookup"><span data-stu-id="a71b3-148">Identity in a multitenant app</span></span>
<span data-ttu-id="a71b3-149">В мультитенантном приложении пользователей необходимо рассматривать в контексте клиентов.</span><span class="sxs-lookup"><span data-stu-id="a71b3-149">In a multitenant app, you must consider users in the context of tenants.</span></span>

<span data-ttu-id="a71b3-150">**Аутентификация**</span><span class="sxs-lookup"><span data-stu-id="a71b3-150">**Authentication**</span></span>

* <span data-ttu-id="a71b3-151">Пользователи входят в приложение с использованием учетных данных организации.</span><span class="sxs-lookup"><span data-stu-id="a71b3-151">Users sign into the app with their organization credentials.</span></span> <span data-ttu-id="a71b3-152">Им не нужно создавать профили пользователей для приложения.</span><span class="sxs-lookup"><span data-stu-id="a71b3-152">They don't have to create new user profiles for the app.</span></span>
* <span data-ttu-id="a71b3-153">Пользователи одной организации являются частью одного и того же клиента.</span><span class="sxs-lookup"><span data-stu-id="a71b3-153">Users within the same organization are part of the same tenant.</span></span>
* <span data-ttu-id="a71b3-154">Когда пользователь входит в систему, приложение знает, какому клиенту он принадлежит.</span><span class="sxs-lookup"><span data-stu-id="a71b3-154">When a user signs in, the application knows which tenant the user belongs to.</span></span>

<span data-ttu-id="a71b3-155">**Авторизация**</span><span class="sxs-lookup"><span data-stu-id="a71b3-155">**Authorization**</span></span>

* <span data-ttu-id="a71b3-156">При авторизации действий пользователя (например просмотра ресурса) приложение должно учитывать клиент пользователя.</span><span class="sxs-lookup"><span data-stu-id="a71b3-156">When authorizing a user's actions (say, viewing a resource), the app must take into account the user's tenant.</span></span>
* <span data-ttu-id="a71b3-157">Пользователям могут быть назначены роли в приложении, такие как "Администратор" или "Обычный пользователь".</span><span class="sxs-lookup"><span data-stu-id="a71b3-157">Users might be assigned roles within the application, such as "Admin" or "Standard User".</span></span> <span data-ttu-id="a71b3-158">Назначениями ролей должен управлять клиент, а не поставщик SaaS.</span><span class="sxs-lookup"><span data-stu-id="a71b3-158">Role assignments should be managed by the customer, not by the SaaS provider.</span></span>

<span data-ttu-id="a71b3-159">**Пример.**</span><span class="sxs-lookup"><span data-stu-id="a71b3-159">**Example.**</span></span> <span data-ttu-id="a71b3-160">Элис, сотрудник компании Contoso, переходит к приложению в браузере и нажимает кнопку "Вход".</span><span class="sxs-lookup"><span data-stu-id="a71b3-160">Alice, an employee at Contoso, navigates to the application in her browser and clicks the “Log in” button.</span></span> <span data-ttu-id="a71b3-161">Она перенаправляется на экран входа, где вводит свои корпоративные учетные данные (имя пользователя и пароль).</span><span class="sxs-lookup"><span data-stu-id="a71b3-161">She is redirected to a login screen where she enters her corporate credentials (username and password).</span></span> <span data-ttu-id="a71b3-162">Она выполнила вход в приложение как `alice@contoso.com`.</span><span class="sxs-lookup"><span data-stu-id="a71b3-162">At this point, she is logged into the app as `alice@contoso.com`.</span></span> <span data-ttu-id="a71b3-163">Приложению также известно, что Элис является администратором этого приложения.</span><span class="sxs-lookup"><span data-stu-id="a71b3-163">The application also knows that Alice is an admin user for this application.</span></span> <span data-ttu-id="a71b3-164">Так как она имеет права администратора, она может просмотреть список всех ресурсов, принадлежащих компании Contoso.</span><span class="sxs-lookup"><span data-stu-id="a71b3-164">Because she is an admin, she can see a list of all the resources that belong to Contoso.</span></span> <span data-ttu-id="a71b3-165">Однако она не может просмотреть ресурсы компании Fabrikam, так как является администратором только в рамках своего клиента.</span><span class="sxs-lookup"><span data-stu-id="a71b3-165">However, she cannot view Fabrikam's resources, because she is an admin only within her tenant.</span></span>

<span data-ttu-id="a71b3-166">В этом руководстве мы отдельно рассмотрим использование Azure AD для управления удостоверениями.</span><span class="sxs-lookup"><span data-stu-id="a71b3-166">In this guidance, we'll look specifically at using Azure AD for identity management.</span></span>

* <span data-ttu-id="a71b3-167">Предполагается, что клиент хранит профили пользователей в Azure AD (включая клиенты Office 365 и Dynamics CRM).</span><span class="sxs-lookup"><span data-stu-id="a71b3-167">We assume the customer stores their user profiles in Azure AD (including Office365 and Dynamics CRM tenants)</span></span>
* <span data-ttu-id="a71b3-168">[Azure AD Connect][ADConnect] могут использовать клиенты с локальной Active Directory (AD) для синхронизации их локальной службы AD с Azure AD.</span><span class="sxs-lookup"><span data-stu-id="a71b3-168">Customers with on-premise Active Directory (AD) can use [Azure AD Connect][ADConnect] to sync their on-premise AD with Azure AD.</span></span>

<span data-ttu-id="a71b3-169">Если клиент с локальной службой AD не может использовать Azure AD Connect (ввиду особенностей корпоративной ИТ-политики или по другим причинам), поставщик SaaS может создать федерацию с AD с помощью служб федерации Active Directory (AD FS).</span><span class="sxs-lookup"><span data-stu-id="a71b3-169">If a customer with on-premise AD cannot use Azure AD Connect (due to corporate IT policy or other reasons), the SaaS provider can federate with the customer's AD through Active Directory Federation Services (AD FS).</span></span> <span data-ttu-id="a71b3-170">Эта возможность описана в статье [Федерация со службой AD FS клиента].</span><span class="sxs-lookup"><span data-stu-id="a71b3-170">This option is described in [Federating with a customer's AD FS].</span></span>

<span data-ttu-id="a71b3-171">В этом руководстве не учитываются другие аспекты мультитенантности, такие как секционирование данных, конфигурация для отдельных клиентов и т. д.</span><span class="sxs-lookup"><span data-stu-id="a71b3-171">This guidance does not consider other aspects of multitenancy such as data partitioning, per-tenant configuration, and so forth.</span></span>

<span data-ttu-id="a71b3-172">[**Далее**][tailpin]</span><span class="sxs-lookup"><span data-stu-id="a71b3-172">[**Next**][tailpin]</span></span>



<!-- Links -->
[ADConnect]: /azure/active-directory/active-directory-aadconnect
[AzureAD]: /azure/active-directory

[Федерация со службой AD FS клиента]: adfs.md
[Federating with a customer's AD FS]: adfs.md
[tailpin]: tailspin.md

[running-the-app]: ./run-the-app.md
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
