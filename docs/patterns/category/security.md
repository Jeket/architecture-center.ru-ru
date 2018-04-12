---
title: Шаблоны безопасности
description: Безопасность — это способность системы предотвращать вредоносные или случайные действия, выходящие за рамки планируемого использования, а также предупреждать разглашение или потерю информации. Облачные приложения доступны в Интернете за пределами доверенных локальных границ, зачастую общедоступны и могут применяться ненадежными пользователями. Приложения следует разрабатывать и развертывать таким образом, чтобы защитить их от вредоносных атак, предоставить доступ только утвержденным пользователям и обеспечить безопасность конфиденциальных данных.
keywords: Конструктивный шаблон
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: 8437b8dfef751226580437a1b5678ca0e0e71f18
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2018
---
# <a name="security-patterns"></a><span data-ttu-id="0d533-106">Шаблоны безопасности</span><span class="sxs-lookup"><span data-stu-id="0d533-106">Security patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="0d533-107">Безопасность — это способность системы предотвращать вредоносные или случайные действия, выходящие за рамки планируемого использования, а также предупреждать разглашение или потерю информации.</span><span class="sxs-lookup"><span data-stu-id="0d533-107">Security is the capability of a system to prevent malicious or accidental actions outside of the designed usage, and to prevent disclosure or loss of information.</span></span> <span data-ttu-id="0d533-108">Облачные приложения доступны в Интернете за пределами доверенных локальных границ, зачастую общедоступны и могут применяться ненадежными пользователями.</span><span class="sxs-lookup"><span data-stu-id="0d533-108">Cloud applications are exposed on the Internet outside trusted on-premises boundaries, are often open to the public, and may serve untrusted users.</span></span> <span data-ttu-id="0d533-109">Приложения следует разрабатывать и развертывать таким образом, чтобы защитить их от вредоносных атак, предоставить доступ только утвержденным пользователям и обеспечить безопасность конфиденциальных данных.</span><span class="sxs-lookup"><span data-stu-id="0d533-109">Applications must be designed and deployed in a way that protects them from malicious attacks, restricts access to only approved users, and protects sensitive data.</span></span>


|                    <span data-ttu-id="0d533-110">Модель</span><span class="sxs-lookup"><span data-stu-id="0d533-110">Pattern</span></span>                     |                                                                                                         <span data-ttu-id="0d533-111">Сводка</span><span class="sxs-lookup"><span data-stu-id="0d533-111">Summary</span></span>                                                                                                         |
|------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [<span data-ttu-id="0d533-112">Федеративная идентификация</span><span class="sxs-lookup"><span data-stu-id="0d533-112">Federated Identity</span></span>](../federated-identity.md) |                                                                                <span data-ttu-id="0d533-113">Делегирование проверки подлинности внешнему поставщику удостоверений.</span><span class="sxs-lookup"><span data-stu-id="0d533-113">Delegate authentication to an external identity provider.</span></span>                                                                                |
|         [<span data-ttu-id="0d533-114">Привратник</span><span class="sxs-lookup"><span data-stu-id="0d533-114">Gatekeeper</span></span>](../gatekeeper.md)         | <span data-ttu-id="0d533-115">Защита приложений и служб с помощью выделенного экземпляра узла, который выполняет роль брокера между клиентами и приложением или службой, проверяет и очищает запросы, а также передает запросы и данные между ними.</span><span class="sxs-lookup"><span data-stu-id="0d533-115">Protect applications and services by using a dedicated host instance that acts as a broker between clients and the application or service, validates and sanitizes requests, and passes requests and data between them.</span></span> |
|          [<span data-ttu-id="0d533-116">Ключ камердинера</span><span class="sxs-lookup"><span data-stu-id="0d533-116">Valet Key</span></span>](../valet-key.md)          |                                                        <span data-ttu-id="0d533-117">Использование токена или ключа, который предоставляет клиентам ограниченный прямой доступ к определенному ресурсу или службе.</span><span class="sxs-lookup"><span data-stu-id="0d533-117">Use a token or key that provides clients with restricted direct access to a specific resource or service.</span></span>                                                        |

