---
title: Сведения о приложении Tailspin Surveys
description: Обзор приложения Tailspin Surveys
author: MikeWasson
ms.date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: index
pnp.series.next: authenticate
ms.openlocfilehash: a1c357bd1b5306d1255c66aaea96d86be55e7b77
ms.sourcegitcommit: e7e0e0282fa93f0063da3b57128ade395a9c1ef9
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/05/2018
ms.locfileid: "52902074"
---
# <a name="the-tailspin-scenario"></a><span data-ttu-id="709d4-103">Сценарий Tailspin</span><span class="sxs-lookup"><span data-stu-id="709d4-103">The Tailspin scenario</span></span>

<span data-ttu-id="709d4-104">[![GitHub](../_images/github.png) Пример кода][sample application]</span><span class="sxs-lookup"><span data-stu-id="709d4-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="709d4-105">Tailspin — это вымышленная компания, которая разрабатывает приложение SaaS с именем Surveys.</span><span class="sxs-lookup"><span data-stu-id="709d4-105">Tailspin is a fictitious company that is developing a SaaS application named Surveys.</span></span> <span data-ttu-id="709d4-106">Это приложение позволяет организациям создавать и публиковать интерактивные опросы.</span><span class="sxs-lookup"><span data-stu-id="709d4-106">This application enables organizations to create and publish online surveys.</span></span>

* <span data-ttu-id="709d4-107">Организация может зарегистрироваться для использования приложения.</span><span class="sxs-lookup"><span data-stu-id="709d4-107">An organization can sign up for the application.</span></span>
* <span data-ttu-id="709d4-108">После регистрации пользователи могут входить в приложение с помощью учетных данных организации.</span><span class="sxs-lookup"><span data-stu-id="709d4-108">After the organization is signed up, users can sign into the application with their organizational credentials.</span></span>
* <span data-ttu-id="709d4-109">Они могут создавать, изменять и публиковать опросы.</span><span class="sxs-lookup"><span data-stu-id="709d4-109">Users can create, edit, and publish surveys.</span></span>

> [!NOTE]
> <span data-ttu-id="709d4-110">Чтобы приступить к работе с приложением, см. статью [Запуск приложения Surveys].</span><span class="sxs-lookup"><span data-stu-id="709d4-110">To get started with the application, see [Run the Surveys application].</span></span>
> 
> 

## <a name="users-can-create-edit-and-view-surveys"></a><span data-ttu-id="709d4-111">Пользователи могут создавать, изменять и просматривать опросы</span><span class="sxs-lookup"><span data-stu-id="709d4-111">Users can create, edit, and view surveys</span></span>
<span data-ttu-id="709d4-112">Пользователь, прошедший проверку подлинности, может просматривать все опросы, которые он создал или для которых ему предоставлены права участника, а также создавать новые опросы.</span><span class="sxs-lookup"><span data-stu-id="709d4-112">An authenticated user can view all the surveys that he or she has created or has contributor rights to, and create new surveys.</span></span> <span data-ttu-id="709d4-113">Обратите внимание, что пользователь выполнил вход с помощью своего удостоверения организации — `bob@contoso.com`.</span><span class="sxs-lookup"><span data-stu-id="709d4-113">Notice that the user is signed in with his organizational identity, `bob@contoso.com`.</span></span>

![Приложение Surveys](./images/surveys-screenshot.png)

<span data-ttu-id="709d4-115">На этом снимке экрана показана страница редактирования опроса:</span><span class="sxs-lookup"><span data-stu-id="709d4-115">This screenshot shows the Edit Survey page:</span></span>

![Изменение опроса](./images/edit-survey.png)

<span data-ttu-id="709d4-117">Пользователи также могут просматривать любые опросы, созданные другими пользователями, в рамках одного клиента.</span><span class="sxs-lookup"><span data-stu-id="709d4-117">Users can also view any surveys created by other users within the same tenant.</span></span>

![Опросы клиентов](./images/tenant-surveys.png)

## <a name="survey-owners-can-invite-contributors"></a><span data-ttu-id="709d4-119">Владельцы опросов могут приглашать участников</span><span class="sxs-lookup"><span data-stu-id="709d4-119">Survey owners can invite contributors</span></span>
<span data-ttu-id="709d4-120">Когда пользователь создает опрос, он может приглашать других пользователей в качестве участников опроса.</span><span class="sxs-lookup"><span data-stu-id="709d4-120">When a user creates a survey, he or she can invite other people to be contributors on the survey.</span></span> <span data-ttu-id="709d4-121">Участники могут изменять опрос, но не могут удалять или публиковать его.</span><span class="sxs-lookup"><span data-stu-id="709d4-121">Contributors can edit the survey, but cannot delete or publish it.</span></span>  

![Добавление участника](./images/add-contributor.png)

<span data-ttu-id="709d4-123">Пользователь может добавлять участников из других клиентов, что позволяет совместно использовать ресурсы между клиентами.</span><span class="sxs-lookup"><span data-stu-id="709d4-123">A user can add contributors from other tenants, which enables cross-tenant sharing of resources.</span></span> <span data-ttu-id="709d4-124">На этом снимке экрана Боб (`bob@contoso.com`) добавляет в свой опрос Элис (`alice@fabrikam.com`) в качестве участника.</span><span class="sxs-lookup"><span data-stu-id="709d4-124">In this screenshot, Bob (`bob@contoso.com`) is adding Alice (`alice@fabrikam.com`) as a contributor to a survey that Bob created.</span></span>

<span data-ttu-id="709d4-125">При входе в систему Элис видит опрос в разделе "Опросы, участником которых я могу стать".</span><span class="sxs-lookup"><span data-stu-id="709d4-125">When Alice logs in, she sees the survey listed under "Surveys I can contribute to".</span></span>

![Участник опроса](./images/contributor.png)

<span data-ttu-id="709d4-127">Обратите внимание, что Элис входит в собственный клиент не как гость клиента Contoso.</span><span class="sxs-lookup"><span data-stu-id="709d4-127">Note that Alice signs into her own tenant, not as a guest of the Contoso tenant.</span></span> <span data-ttu-id="709d4-128">У Элис есть разрешения участника только для этого опроса &mdash; она не может просматривать другие опросы в клиенте Contoso.</span><span class="sxs-lookup"><span data-stu-id="709d4-128">Alice has contributor permissions only for that survey &mdash; she cannot view other surveys from the Contoso tenant.</span></span>

## <a name="architecture"></a><span data-ttu-id="709d4-129">Архитектура</span><span class="sxs-lookup"><span data-stu-id="709d4-129">Architecture</span></span>
<span data-ttu-id="709d4-130">Приложение Surveys состоит из веб-интерфейса и серверной части веб-API.</span><span class="sxs-lookup"><span data-stu-id="709d4-130">The Surveys application consists of a web front end and a web API backend.</span></span> <span data-ttu-id="709d4-131">Оба компонента реализованы с помощью [ASP.NET Core].</span><span class="sxs-lookup"><span data-stu-id="709d4-131">Both are implemented using [ASP.NET Core].</span></span>

<span data-ttu-id="709d4-132">Для проверки подлинности веб-приложение использует Azure Active Directory (Azure AD).</span><span class="sxs-lookup"><span data-stu-id="709d4-132">The web application uses Azure Active Directory (Azure AD) to authenticate users.</span></span> <span data-ttu-id="709d4-133">Веб-приложение также вызывает Azure AD для получения маркеров доступа OAuth 2 для веб-API.</span><span class="sxs-lookup"><span data-stu-id="709d4-133">The web application also calls Azure AD to get OAuth 2 access tokens for the Web API.</span></span> <span data-ttu-id="709d4-134">Маркеры доступа кэшируются в кэше Redis для Azure.</span><span class="sxs-lookup"><span data-stu-id="709d4-134">Access tokens are cached in Azure Redis Cache.</span></span> <span data-ttu-id="709d4-135">Кэш позволяет нескольким экземплярам совместно использовать один кэш маркеров (например, в ферме серверов).</span><span class="sxs-lookup"><span data-stu-id="709d4-135">The cache enables multiple instances to share the same token cache (e.g., in a server farm).</span></span>

![Архитектура](./images/architecture.png)

<span data-ttu-id="709d4-137">[**Далее**][authentication]</span><span class="sxs-lookup"><span data-stu-id="709d4-137">[**Next**][authentication]</span></span>

<!-- Links -->

[authentication]: authenticate.md

[Запуск приложения Surveys]: ./run-the-app.md
[Run the Surveys application]: ./run-the-app.md
[ASP.NET Core]: /aspnet/core
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
