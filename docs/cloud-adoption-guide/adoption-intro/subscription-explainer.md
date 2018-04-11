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
# <a name="explainer-what-is-an-azure-subscription"></a>Пояснения. Знакомство с подпиской Azure

В статье с пояснениями [Знакомство с клиентом Azure Active Directory](tenant-explainer.md) вы узнали, что цифровой идентификатор для организации хранится в клиенте Azure Active Directory. Вы также узнали, что Azure доверяет Azure Active Directory аутентификацию запросов пользователей на создание, чтение, обновление или удаление ресурса. 

Мы отлично понимаем, почему важно ограничивать доступ к этим операциям с ресурсами — чтобы пользователи, не прошедшие аутентификацию и авторизацию, не могли получить доступ к ресурсам. Но эти операции с ресурсами имеют другие свойства, которые организация должна контролировать. Например, количество ресурсов, которые пользователь или группа пользователей могут создать, и стоимость, связанная с запуском этих ресурсов. 

Azure реализует такой элемент управления, и он называется **подпиской**. Подписка позволяет объединять в группы пользователей и ресурсы, которые были созданы этими пользователям. Каждый из этих ресурсов выделяет средства в соответствии с [общим пределом][subscription-service-limits] для этого ресурса.

Организации могут использовать подписки для управления затратами и созданием ресурсов пользователями, командами, проектными группами или с использованием других стратегий. Эти стратегии рассматриваются в статьях на этапах промежуточного и расширенного внедрения. 

## <a name="next-steps"></a>Дополнительная информация

* Теперь, когда вы узнали о подписках Azure, ознакомьтесь со статьей о [создании подписки](subscription.md), прежде чем создать первые ресурсы Azure.

<!-- Links -->
[azure-get-started]: https://azure.microsoft.com/get-started/
[azure-offers]: https://azure.microsoft.com/support/legal/offer-details/
[azure-free-trial]: https://azure.microsoft.com/offers/ms-azr-0044p/
[azure-change-subscription-offer]: /azure/billing/billing-how-to-switch-azure-offer
[microsoft-account]: https://account.microsoft.com/account
[subscription-service-limits]: /azure/azure-subscription-service-limits
[docs-organizational-account]: https://docs.microsoft.com/azure/active-directory/sign-up-organization
