---
title: CAF. Руководство по принятию решений касательно согласованности ресурсов
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Сведения о согласованности ресурсов при планировании миграции в Azure.
author: rotycenh
ms.openlocfilehash: 3159e4b7aeddfdd99261c0f68591998d741f3359
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/17/2019
ms.locfileid: "59640058"
---
# <a name="caf-resource-consistency-decision-guide"></a>CAF. Руководство по принятию решений касательно согласованности ресурсов

Azure [разработка подписки](../subscriptions/overview.md) определяет, как организовать облачных ресурсов по отношению к организации структуры учета рекомендации и требования рабочей нагрузки. Помимо этого уровня структуры адресации политики вашей организации нормативные требования по недвижимости вашего облака требуется возможность согласованно организации, развертывания и управления ресурсами в подписке.

![Схема вариантов обеспечения согласованности ресурсов, отсортированных в порядке увеличения сложности, со ссылками для быстрого перехода.](../../_images/discovery-guides/discovery-guide-resource-consistency.png)

Перейти к разделу: [Простое группирование](#basic-grouping) | [Согласованность развертывания](#deployment-consistency) | [Согласованность политики](#policy-consistency) | [Иерархическая согласованность](#hierarchical-consistency) | [Автоматическая согласованность](#automated-consistency)

Решения по поводу уровнем требований к согласованности ресурсов риэлтерской компании облака в основном обусловлено следующими факторами: размер цифровых недвижимости после миграции, business или требования к среде, которые не удается отнести из существующей подписки подхода к проектированию, или нужно применить управление временем, после развертывания ресурсов. 

Как эти факторы увеличение важности, преимущества обеспечения согласованного развертывания, группирование и управление ресурсами на основе облака становится более важным. Достижение более высокий уровень согласованности ресурсов в соответствии с требованиями увеличение требуются дополнительные усилия, потраченные на автоматизации, средств и внедрения согласованности и это приводит к больше времени, затраченное на управление изменениями и отслеживания.

## <a name="basic-grouping"></a>Простое группирование

В Azure [группы ресурсов](/azure/azure-resource-manager/resource-group-overview#resource-groups) представляют собой базовый механизм организации ресурсов, позволяющий логически группировать ресурсы в подписке.

Это контейнеры для ресурсов с общим жизненным циклом или общими ограничениями управления, такими как требования управления доступом на основе ролей (RBAC) или требования политики. Группы ресурсов не могут быть вложенными, и ресурсы могут принадлежать только одной группе ресурсов. Определенные действия могут применяться для всех ресурсов в группе ресурсов. Например, при удалении группы ресурсов удаляются все входящие в нее ресурсы. Существуют распространенные шаблоны создания групп ресурсов, которые разделены на две общие категории:

- Традиционные рабочие нагрузки ИТ. Чаще всего группируются по элементам в рамках одного жизненного цикла, например приложения. Группирование по приложениям позволяет управлять отдельными приложениями.
- Гибкие рабочие нагрузки ИТ. Сосредотачиваются на облачных приложениях для внешних клиентов. Эти группы ресурсов часто отражают функциональные уровни развертывания (например, уровень Интернета или уровень приложений) и управления.

## <a name="deployment-consistency"></a>Согласованность развертывания

Основываясь на основе базового ресурса, механизм группирования, платформа Azure предоставляет систему использовать шаблоны для развертывания ресурсов в облачной среде. Вы можете использовать шаблоны для создания согласованной структуры и соглашений об именовании при развертывании рабочих нагрузок, применяя эти аспекты развертывания и контроля ресурсов.

[Шаблоны Azure Resource Manager](/azure/azure-resource-manager/resource-group-overview#template-deployment) позволяют многократно развертывать ресурсы в согласованном состоянии с использованием предопределенной структуры группы ресурсов и конфигурации. Эти шаблоны помогают задать набор стандартов в качестве основы для ваших развертываний.

Например может иметь стандартный шаблон для развертывания рабочей нагрузки сервера web, которая содержит две виртуальные машины в качестве веб-серверов, в сочетании с подсистемой балансировки нагрузки для распределения трафика между серверами. Затем можно повторно использовать этот шаблон для создания структурно идентичных виртуальных машин и балансировщик нагрузки, при необходимости этого типа рабочей нагрузки, участвует только изменения, имя развертывания и IP-адреса.

Обратите внимание, что можно также развертывать эти шаблоны программным способом и интегрировать их в свои системы CI/CD.

## <a name="policy-consistency"></a>Согласованность политик

Чтобы обеспечить применение политик управления во время создания ресурсов, как часть проекта группирования ресурсов при развертывании ресурсов необходимо использовать типичную конфигурацию.

Объединяя группы ресурсов и стандартизированные шаблоны Resource Manager, можно применять стандарты для параметров, необходимых в развертывании, и правил [Политики Azure](/azure/governance/policy/overview), применяемых к каждой группе ресурсов или ресурсу.

Например, может существовать требование подключения всех виртуальных машин, развернутых в рамках вашей подписки, к общей подсети, управляемой центральным ИТ-отделом. Вы можете создать стандартный шаблон для развертывания виртуальных машин рабочей нагрузки, который будет создавать отдельную группу ресурсов для рабочей нагрузки и развертывать в ней виртуальные машины. У этой группы ресурсов будет правило политики, которое разрешает присоединение к общей подсети только тех сетевых интерфейсов, которые находятся внутри группы ресурсов.

Более подробное рассмотрение принудительного применения решений политики в рамках облачного развертывания см. в [этой статье](../policy-enforcement/overview.md).

## <a name="hierarchical-consistency"></a>Иерархическая согласованность

Группы ресурсов позволяет поддерживать дополнительных уровней иерархии в вашей организации в рамках подписки, применение правил политики Azure и доступа к элементам управления на уровне группы ресурсов. Тем не менее по мере роста размера вашей облачной недвижимости, может потребоваться поддержка сложнее между подписками нормативные требования к чем может поддерживаться с помощью иерархии соглашения Enterprise Azure Enterprise, отдел или учетную запись и подписки. 

[Группы управления Azure](../subscriptions/overview.md#management-groups) позволяет организации подписок в более сложные структуры организации путем группирования подписок в иерархии альтернативные для, установленного с структура соглашения enterprise. Этот альтернативный иерархия позволяет применить механизмы принудительного применения элемента управления и политики доступа в нескольких подписках и ресурсы, которые они содержат. Иерархии группы управления могут использоваться для сопоставления риэлтерской компании облачных подписок с помощью операции или нормативные требования бизнеса. 

## <a name="automated-consistency"></a>Автоматическая согласованность

Для больших облачных развертываний глобальное управление становится более важным и более сложным. Очень важно автоматически принудительно применять нормативные требования при развертывании ресурсов, а также выполнять обновленные требования для существующих развертываний.

Служба [Azure Blueprints](/azure/governance/blueprints/overview) позволяет организациям поддерживать глобальное управление большими облачными инфраструктурами в Azure. Схемы выходят за рамки возможностей, предоставляемых стандартными шаблонами Azure Resource Manager. Они позволяют создавать полные оркестрации развертываний, поддерживающие правила развертывания ресурсов и применение правил политики. Схемы поддерживают управление версиями, возможность применения обновлений для всех подписок, где использовалась схема, и возможность блокировки развернутых подписок во избежание несанкционированного создания и изменения ресурсов.

Эти пакеты развертывания позволяют ИТ-специалистам и командам разработчиков быстро развертывать новые рабочие нагрузки и сетевые ресурсы, которые соответствуют меняющимся требованиям политики организации. Схемы также можно интегрировать в конвейеры CI/CD для применения пересмотренных стандартов управления к развертываниям по мере их обновления.

## <a name="next-steps"></a>Дальнейшие действия

Ознакомьтесь со сведениями об именовании ресурсов и добавлении к ним тегов для дальнейшей организации облачных ресурсов и управления ими.

> [!div class="nextstepaction"]
> [Именование ресурсов и добавление к ним тегов](../resource-tagging/overview.md)
