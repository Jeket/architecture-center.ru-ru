---
title: "Руководство. Разработка клиента Azure AD"
description: "Руководство по разработке клиента Azure в рамках базовой стратегии внедрения облака"
author: telmosampaio
ms.openlocfilehash: 5bf710f74bde04e041f2896b4a638c3513e4aa44
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/09/2018
---
# <a name="guidance-azure-ad-tenant-design"></a>Руководство. Разработка клиента Azure AD

Клиент Azure AD предоставляет службы идентификации по цифровым удостоверениям и пространства имен, используемые для одного или нескольких [подписок Azure](subscription-explainer.md). Если вы придерживаетесь плана базового внедрения, то уже знаете, [как получить клиент Azure AD][how-to-get-aad-tenant]. 

## <a name="design-considerations"></a>Рекомендации по проектированию

- На базовом этапе внедрения вы можете начать с одного клиента Azure AD. Если в организации есть подписка Office 365 или подписка Azure, то у вас уже есть клиент Azure AD, который можно использовать. Если таких подписок нет, вы можете узнать больше о том, [как получить клиент Azure AD][how-to-get-aad-tenant]. 
- На этапах промежуточного и расширенного внедрения вы узнаете, как синхронизировать или объединять в федерацию локальные каталоги и Azure AD. Это позволит применять локальные цифровые удостоверения в Azure AD. Но на базовом этапе вы добавите новых пользователей, удостоверения которых будут проверяться только на одном клиенте Azure AD. Вы будете отвечать за управление этими удостоверениями. Например, нужно зарегистрировать новых пользователей в Azure AD и удалить тех, которым больше не требуется доступ к ресурсам Azure, и внести другие изменения в разрешения пользователей.

## <a name="next-steps"></a>Дальнейшие действия

* Теперь, когда у вас есть клиент Azure AD, узнайте о том, как [добавить пользователя][azure-ad-add-user]. Когда добавите одного или нескольких новых пользователей в клиент Azure AD, ознакомьтесь со статьей о [подписках Azure](subscription-explainer.md).

<!-- Links -->

[azure-ad-add-user]: /azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-manage-azure-ad]: /azure/active-directory/active-directory-administer?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-associate-subscription]: /azure/active-directory/active-directory-how-subscriptions-associated-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[how-to-get-aad-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json