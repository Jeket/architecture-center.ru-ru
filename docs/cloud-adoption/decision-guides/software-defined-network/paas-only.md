---
title: CAF. Программно-конфигурируемые сети — только PaaS
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Описание модели "Только PaaS" для функции работы в облачной сети
author: rotycenh
ms.openlocfilehash: 2f3f82d781ddb6544721e82e7b7d795222a2f8ff
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58241205"
---
# <a name="software-defined-networks-paas-only"></a><span data-ttu-id="ffb98-103">Программно-определяемые сети. "Только PaaS"</span><span class="sxs-lookup"><span data-stu-id="ffb98-103">Software Defined Networks: PaaS-only</span></span>

<span data-ttu-id="ffb98-104">При реализации ресурса модели "платформа как услуга" (PaaS) в процессе развертывания автоматически создается принятая базовая сеть с ограниченным числом элементов управления, включая балансировку нагрузки, блокировку портов и подключения к другим службам PaaS.</span><span class="sxs-lookup"><span data-stu-id="ffb98-104">When you implement a platform as a service (PaaS) resource, the deployment process automatically creates an assumed underlying network with a limited number of controls over that network, including load balancing, port blocking, and connections to other PaaS services.</span></span>

<span data-ttu-id="ffb98-105">В Azure несколько типов ресурсов PaaS можно [развернуть в](/azure/virtual-network/virtual-network-for-azure-services) виртуальной сети или [подключить к](/azure/virtual-network/virtual-network-service-endpoints-overview) ней, что позволяет этим ресурсам интегрироваться с имеющейся инфраструктурой виртуальной сети.</span><span class="sxs-lookup"><span data-stu-id="ffb98-105">In Azure, several PaaS resource types can be [deployed into](/azure/virtual-network/virtual-network-for-azure-services) or [connected to](/azure/virtual-network/virtual-network-service-endpoints-overview) a virtual network, allowing these resources to integrate with your existing virtual networking infrastructure.</span></span> <span data-ttu-id="ffb98-106">Однако во многих случаях архитектуры сети "Только PaaS", полагающаяся только на сетевые возможности по умолчанию, изначально предоставляемые ресурсами PaaS, достаточно для соответствия требованиям рабочей нагрузки.</span><span class="sxs-lookup"><span data-stu-id="ffb98-106">However, in many cases a PaaS only networking architecture, relying only on these default networking capabilities natively provided by PaaS resources, is sufficient to meet workload requirements.</span></span>

<span data-ttu-id="ffb98-107">Если вы рассматриваете архитектуру сети "Только PaaS", убедитесь, что требуемые предположения соответствуют вашим требованиям.</span><span class="sxs-lookup"><span data-stu-id="ffb98-107">If you are considering a PaaS only networking architecture, be sure you validate that the required assumptions align with your requirements.</span></span>

## <a name="paas-only-assumptions"></a><span data-ttu-id="ffb98-108">Предположения о модели "Только PaaS"</span><span class="sxs-lookup"><span data-stu-id="ffb98-108">PaaS-only assumptions</span></span>

<span data-ttu-id="ffb98-109">Развертывание архитектуры сети "Только PaaS"предполагает следующее:</span><span class="sxs-lookup"><span data-stu-id="ffb98-109">Deploying a PaaS-only networking architecture assumes the following:</span></span>

- <span data-ttu-id="ffb98-110">Развертываемое приложение — это изолированное приложение или зависящее только от других ресурсов PaaS.</span><span class="sxs-lookup"><span data-stu-id="ffb98-110">The application being deployed is a standalone application OR is dependent on only other PaaS resources.</span></span>
- <span data-ttu-id="ffb98-111">Команды ИТ-специалистов могут обновить свои инструменты, улучшить навыки и процессы для поддержки конфигурации и развертывания изолированных приложений PaaS, а также управления ими.</span><span class="sxs-lookup"><span data-stu-id="ffb98-111">Your IT operations teams can update their tools, training, and processes to support management, configuration, and deployment of standalone PaaS applications.</span></span>
- <span data-ttu-id="ffb98-112">Приложение PaaS не является частью более широкого сценария облачной миграции, который будет включать ресурсы IaaS.</span><span class="sxs-lookup"><span data-stu-id="ffb98-112">The PaaS application is not part of a broader cloud migration effort that will include IaaS resources.</span></span>

<span data-ttu-id="ffb98-113">Эти предположения — минимальные квалификаторы, соответствующие развертыванию сети "Только PaaS".</span><span class="sxs-lookup"><span data-stu-id="ffb98-113">These assumptions are minimum qualifiers aligned to deploying a PaaS-only network.</span></span> <span data-ttu-id="ffb98-114">Хотя этот подход может соответствовать требованиям развертывания одного приложения, команде внедрения облака следует изучить эти вопросы, рассчитанные на перспективу:</span><span class="sxs-lookup"><span data-stu-id="ffb98-114">While this approach may align with the requirements of a single application deployment, your Cloud Adoption Team should examine these long-term questions:</span></span>

- <span data-ttu-id="ffb98-115">Расширится ли это развертывание по области или масштабу, чтобы потребовать доступ к другим ресурсам, отличным от ресурсов PaaS?</span><span class="sxs-lookup"><span data-stu-id="ffb98-115">Will this deployment expand in scope or scale to require access to other non-PaaS resources?</span></span>
- <span data-ttu-id="ffb98-116">Планируются ли другие развертывания PaaS за пределами текущего решения?</span><span class="sxs-lookup"><span data-stu-id="ffb98-116">Are other PaaS deployments planned beyond the current solution?</span></span>
- <span data-ttu-id="ffb98-117">Есть ли у организации планы относительно других будущих облачных миграций?</span><span class="sxs-lookup"><span data-stu-id="ffb98-117">Does the organization have plans for other future cloud migrations?</span></span>

<span data-ttu-id="ffb98-118">Ответы на эти вопросы не препятствуют команде выбрать модель "Только PaaS", но следует учитывать этот вариант перед вынесением окончательного решения.</span><span class="sxs-lookup"><span data-stu-id="ffb98-118">The answers to these questions would not preclude a team from choosing a PaaS only option but should be considered before making a final decision.</span></span>
