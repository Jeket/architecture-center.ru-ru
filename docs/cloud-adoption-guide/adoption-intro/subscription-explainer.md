---
title: Пояснения. Знакомство с подпиской Azure
description: Пояснения о подписках, учетных записях и предложениях Azure
author: alexbuckgit
ms.openlocfilehash: 1650d90d6f78b46b7fe4128d2dab6a80bd6cca78
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/23/2018
---
# <a name="explainer-what-is-an-azure-subscription"></a><span data-ttu-id="0fb47-103">Пояснения. Знакомство с подпиской Azure</span><span class="sxs-lookup"><span data-stu-id="0fb47-103">Explainer: What is an Azure subscription?</span></span>

<span data-ttu-id="0fb47-104">В статье с пояснениями [Знакомство с клиентом Azure Active Directory](tenant-explainer.md) вы узнали, что цифровой идентификатор для организации хранится в клиенте Azure Active Directory.</span><span class="sxs-lookup"><span data-stu-id="0fb47-104">In the [what is an Azure Active Directory tenant?](tenant-explainer.md) explainer article, you learned that digital identity for your organization is stored in an Azure Active Directory tenant.</span></span> <span data-ttu-id="0fb47-105">Вы также узнали, что Azure доверяет Azure Active Directory аутентификацию запросов пользователей на создание, чтение, обновление или удаление ресурса.</span><span class="sxs-lookup"><span data-stu-id="0fb47-105">You also learned that Azure trusts Azure Active Directory to authenticate user requests to create, read, update, or delete a resource.</span></span> 

<span data-ttu-id="0fb47-106">Мы отлично понимаем, почему важно ограничивать доступ к этим операциям с ресурсами — чтобы пользователи, не прошедшие аутентификацию и авторизацию, не могли получить доступ к ресурсам.</span><span class="sxs-lookup"><span data-stu-id="0fb47-106">We fundamentally understand why it's necessary to restrict access to these operations on a resource - to prevent unauthenticated and unauthorized users from accessing our resources.</span></span> <span data-ttu-id="0fb47-107">Но эти операции с ресурсами имеют другие свойства, которые организация должна контролировать. Например, количество ресурсов, которые пользователь или группа пользователей могут создать, и стоимость, связанная с запуском этих ресурсов.</span><span class="sxs-lookup"><span data-stu-id="0fb47-107">However, these resource operations have other properties that an organization would like to control, such as the number of resources a user or group of users is allowed to create, and, the cost to run those resources.</span></span> 

<span data-ttu-id="0fb47-108">Azure реализует такой элемент управления, и он называется **подпиской**.</span><span class="sxs-lookup"><span data-stu-id="0fb47-108">Azure implements this control, and it is named a **subscription**.</span></span> <span data-ttu-id="0fb47-109">Подписка позволяет объединять в группы пользователей и ресурсы, которые были созданы этими пользователям.</span><span class="sxs-lookup"><span data-stu-id="0fb47-109">A subscription groups together users and the resources that have been created by those users.</span></span> <span data-ttu-id="0fb47-110">Каждый из этих ресурсов выделяет средства в соответствии с [общим пределом][subscription-service-limits] для этого ресурса.</span><span class="sxs-lookup"><span data-stu-id="0fb47-110">Each of those resources contributes to an [overall limit][subscription-service-limits] on that particular resource.</span></span>

<span data-ttu-id="0fb47-111">Организации могут использовать подписки для управления затратами и созданием ресурсов пользователями, командами, проектными группами или с использованием других стратегий.</span><span class="sxs-lookup"><span data-stu-id="0fb47-111">Organizations can use subscriptions to manage costs and creation of resource by users, teams, projects, or using many other strategies.</span></span> <span data-ttu-id="0fb47-112">Эти стратегии рассматриваются в статьях на этапах промежуточного и расширенного внедрения.</span><span class="sxs-lookup"><span data-stu-id="0fb47-112">These strategies will be discussed in the intermediate and advanced adoption stage articles.</span></span> 

## <a name="next-steps"></a><span data-ttu-id="0fb47-113">Дополнительная информация</span><span class="sxs-lookup"><span data-stu-id="0fb47-113">Next steps</span></span>

* <span data-ttu-id="0fb47-114">Теперь, когда вы узнали о подписках Azure, ознакомьтесь со статьей о [создании подписки](subscription.md), прежде чем создать первые ресурсы Azure.</span><span class="sxs-lookup"><span data-stu-id="0fb47-114">Now that you have learned about Azure subscriptions, learn more about [creating a subscription](subscription.md) before you create your first Azure resources..</span></span>

<!-- Links -->
[azure-get-started]: https://azure.microsoft.com/get-started/
[azure-offers]: https://azure.microsoft.com/support/legal/offer-details/
[azure-free-trial]: https://azure.microsoft.com/offers/ms-azr-0044p/
[azure-change-subscription-offer]: /azure/billing/billing-how-to-switch-azure-offer
[microsoft-account]: https://account.microsoft.com/account
[subscription-service-limits]: /azure/azure-subscription-service-limits
[docs-organizational-account]: https://docs.microsoft.com/azure/active-directory/sign-up-organization
