---
title: Шаблоны безопасности
titleSuffix: Cloud Design Patterns
description: Безопасность — это способность системы предотвращать вредоносные или случайные действия, выходящие за рамки планируемого использования, а также предупреждать разглашение или потерю информации. Облачные приложения доступны в Интернете за пределами доверенных локальных границ, зачастую общедоступны и могут применяться ненадежными пользователями. Приложения следует разрабатывать и развертывать таким образом, чтобы защитить их от вредоносных атак, предоставить доступ только утвержденным пользователям и обеспечить безопасность конфиденциальных данных.
keywords: Конструктивный шаблон
author: dragon119
ms.date: 06/23/2017
ms.custom: seodec18
ms.openlocfilehash: 259d4bd5178ca8ab85df0cd03e5268c5a7845a3a
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/04/2019
ms.locfileid: "54009446"
---
# <a name="security-patterns"></a><span data-ttu-id="0cfdb-106">Шаблоны безопасности</span><span class="sxs-lookup"><span data-stu-id="0cfdb-106">Security patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="0cfdb-107">Безопасность — это способность системы предотвращать вредоносные или случайные действия, выходящие за рамки планируемого использования, а также предупреждать разглашение или потерю информации.</span><span class="sxs-lookup"><span data-stu-id="0cfdb-107">Security is the capability of a system to prevent malicious or accidental actions outside of the designed usage, and to prevent disclosure or loss of information.</span></span> <span data-ttu-id="0cfdb-108">Облачные приложения доступны в Интернете за пределами доверенных локальных границ, зачастую общедоступны и могут применяться ненадежными пользователями.</span><span class="sxs-lookup"><span data-stu-id="0cfdb-108">Cloud applications are exposed on the Internet outside trusted on-premises boundaries, are often open to the public, and may serve untrusted users.</span></span> <span data-ttu-id="0cfdb-109">Приложения следует разрабатывать и развертывать таким образом, чтобы защитить их от вредоносных атак, предоставить доступ только утвержденным пользователям и обеспечить безопасность конфиденциальных данных.</span><span class="sxs-lookup"><span data-stu-id="0cfdb-109">Applications must be designed and deployed in a way that protects them from malicious attacks, restricts access to only approved users, and protects sensitive data.</span></span>

|                    <span data-ttu-id="0cfdb-110">Модель</span><span class="sxs-lookup"><span data-stu-id="0cfdb-110">Pattern</span></span>                     |                                                                                                         <span data-ttu-id="0cfdb-111">Сводка</span><span class="sxs-lookup"><span data-stu-id="0cfdb-111">Summary</span></span>                                                                                                         |
|------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [<span data-ttu-id="0cfdb-112">Федеративная идентификация</span><span class="sxs-lookup"><span data-stu-id="0cfdb-112">Federated Identity</span></span>](../federated-identity.md) |                                                                                <span data-ttu-id="0cfdb-113">Делегирование проверки подлинности внешнему поставщику удостоверений.</span><span class="sxs-lookup"><span data-stu-id="0cfdb-113">Delegate authentication to an external identity provider.</span></span>                                                                                |
|         [<span data-ttu-id="0cfdb-114">Привратник</span><span class="sxs-lookup"><span data-stu-id="0cfdb-114">Gatekeeper</span></span>](../gatekeeper.md)         | <span data-ttu-id="0cfdb-115">Защита приложений и служб с помощью выделенного экземпляра узла, который выполняет роль брокера между клиентами и приложением или службой, проверяет и очищает запросы, а также передает запросы и данные между ними.</span><span class="sxs-lookup"><span data-stu-id="0cfdb-115">Protect applications and services by using a dedicated host instance that acts as a broker between clients and the application or service, validates and sanitizes requests, and passes requests and data between them.</span></span> |
|          [<span data-ttu-id="0cfdb-116">Ключ камердинера</span><span class="sxs-lookup"><span data-stu-id="0cfdb-116">Valet Key</span></span>](../valet-key.md)          |                                                        <span data-ttu-id="0cfdb-117">Использование токена или ключа, который предоставляет клиентам ограниченный прямой доступ к определенному ресурсу или службе.</span><span class="sxs-lookup"><span data-stu-id="0cfdb-117">Use a token or key that provides clients with restricted direct access to a specific resource or service.</span></span>                                                        |
