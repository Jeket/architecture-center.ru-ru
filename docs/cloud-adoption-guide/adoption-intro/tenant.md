---
title: Руководство. Разработка клиента Azure AD
description: Руководство по разработке клиента Azure в рамках базовой стратегии внедрения облака
author: telmosampaio
ms.openlocfilehash: 9ac52e9fd44bd8b9c777625002d5960f4f269be2
ms.sourcegitcommit: 29fbcb1eec44802d2c01b6d3bcf7d7bd0bae65fc
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/27/2018
ms.locfileid: "29563461"
---
# <a name="guidance-azure-ad-tenant-design"></a><span data-ttu-id="3fff4-103">Руководство. Разработка клиента Azure AD</span><span class="sxs-lookup"><span data-stu-id="3fff4-103">Guidance: Azure AD tenant design</span></span>

<span data-ttu-id="3fff4-104">Клиент Azure AD предоставляет службы идентификации по цифровым удостоверениям и пространства имен, используемые для одного или нескольких [подписок Azure](subscription-explainer.md).</span><span class="sxs-lookup"><span data-stu-id="3fff4-104">An Azure AD tenant provides digital identity services and namespaces used for one or more [Azure subscriptions](subscription-explainer.md).</span></span> <span data-ttu-id="3fff4-105">Если вы придерживаетесь плана базового внедрения, то уже знаете, [как получить клиент Azure AD][how-to-get-aad-tenant].</span><span class="sxs-lookup"><span data-stu-id="3fff4-105">If you are following the foundational adoption outline, you have already learned [how to get an Azure AD tenant][how-to-get-aad-tenant].</span></span> 

## <a name="design-considerations"></a><span data-ttu-id="3fff4-106">Рекомендации по проектированию</span><span class="sxs-lookup"><span data-stu-id="3fff4-106">Design considerations</span></span>

- <span data-ttu-id="3fff4-107">На базовом этапе внедрения вы можете начать с одного клиента Azure AD.</span><span class="sxs-lookup"><span data-stu-id="3fff4-107">At the foundational adoption stage, you can begin with a single Azure AD tenant.</span></span> <span data-ttu-id="3fff4-108">Если в организации есть подписка Office 365 или подписка Azure, то у вас уже есть клиент Azure AD, который можно использовать.</span><span class="sxs-lookup"><span data-stu-id="3fff4-108">If your organization has an existing Office 365 subscription or an Azure subscription, you already have an Azure AD tenant that you can use.</span></span> <span data-ttu-id="3fff4-109">Если таких подписок нет, вы можете узнать больше о том, [как получить клиент Azure AD][how-to-get-aad-tenant].</span><span class="sxs-lookup"><span data-stu-id="3fff4-109">If you do not have either or of these, you can learn more about [how to get an Azure AD tenant][how-to-get-aad-tenant].</span></span> 
- <span data-ttu-id="3fff4-110">На этапах промежуточного и расширенного внедрения вы узнаете, как синхронизировать или объединять в федерацию локальные каталоги и Azure AD.</span><span class="sxs-lookup"><span data-stu-id="3fff4-110">In the intermediate and advanced adoption stages, you will learn how to synchronize or federate on-premises directories with Azure AD.</span></span> <span data-ttu-id="3fff4-111">Это позволит применять локальные цифровые удостоверения в Azure AD.</span><span class="sxs-lookup"><span data-stu-id="3fff4-111">This will allow you to use on-premises digital identity in Azure AD.</span></span> <span data-ttu-id="3fff4-112">Но на базовом этапе вы добавите новых пользователей, удостоверения которых будут проверяться только на одном клиенте Azure AD.</span><span class="sxs-lookup"><span data-stu-id="3fff4-112">However, at the foundational stage, you will be adding new users that only have identity your single Azure AD tenant.</span></span> <span data-ttu-id="3fff4-113">Вы будете отвечать за управление этими удостоверениями.</span><span class="sxs-lookup"><span data-stu-id="3fff4-113">You will be responsible for managing those identities.</span></span> <span data-ttu-id="3fff4-114">Например, нужно зарегистрировать новых пользователей в Azure AD и удалить тех, которым больше не требуется доступ к ресурсам Azure, и внести другие изменения в разрешения пользователей.</span><span class="sxs-lookup"><span data-stu-id="3fff4-114">For example, you will have to on-board new Azure AD users, off-board Azure AD users that you no longer wish to have access to Azure resources, and other changes to user permissions.</span></span>

## <a name="next-steps"></a><span data-ttu-id="3fff4-115">Дальнейшие действия</span><span class="sxs-lookup"><span data-stu-id="3fff4-115">Next Steps</span></span>

* <span data-ttu-id="3fff4-116">Теперь, когда у вас есть клиент Azure AD, узнайте о том, как [добавить пользователя][azure-ad-add-user].</span><span class="sxs-lookup"><span data-stu-id="3fff4-116">Now that you have an Azure AD tenant, learn [how to add a user][azure-ad-add-user].</span></span> <span data-ttu-id="3fff4-117">Когда добавите одного или нескольких новых пользователей в клиент Azure AD, ознакомьтесь со статьей о [подписках Azure](subscription-explainer.md).</span><span class="sxs-lookup"><span data-stu-id="3fff4-117">After you have added one or more new users to your Azure AD tenant, your next step is learning about [Azure subscriptions](subscription-explainer.md).</span></span>

<!-- Links -->

[azure-ad-add-user]: /azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-manage-azure-ad]: /azure/active-directory/active-directory-administer?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-associate-subscription]: /azure/active-directory/active-directory-how-subscriptions-associated-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[how-to-get-aad-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json
