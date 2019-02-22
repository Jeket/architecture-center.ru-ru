---
title: CAF. Руководства по принятию архитектурных решений
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Сведения о руководствах по архитектурным решениям на платформе внедрения облака.
author: rotycenh
ms.openlocfilehash: b43047b5fc5b636bc84b9f28ec24846730e63672
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55902471"
---
# <a name="architectural-decision-guides"></a>Руководства по принятию архитектурных решений

В руководствах по архитектурным решениям на платформе внедрения облака описаны примеры и модели, которые помогут при создании руководства по проектированию системы управления облаком. В каждом руководстве по принятию решений описывается один основной компонент инфраструктуры облачных развертываний и представлены возможные шаблоны или модели, предназначенные для поддержки определенных сценариев облачных развертываний.

При определении системы управления облаком для вашей организации стратегии действенного управления позволяют получить стратегию базовых показателей. Тем не менее в этих стратегиях предполагаются требования и приоритеты, которые могут не отражать потребности вашей организации.
Эти руководства по принятию решений дополняют примеры стратегий управления, предоставляя альтернативные шаблоны и модели, которые помогают скорректировать варианты проектирования архитектуры, выбранные в примере руководства по проектированию, в соответствии с вашими требованиями.

## <a name="design-guidance-categories"></a>Категории руководств по проектированию

В каждой из следующих категорий представлена основополагающая технология всех облачных развертываний. Изучите варианты в соответствующем разделе ниже, чтобы для каждого значения в примерах стратегий проектирования, которое не соответствуют вашим требованиям, выбрать шаблон или модель, который наиболее полно соответствует вашим потребностям.

[Подписки](./subscriptions/overview.md). Запланируйте разработку подписки и структуру учетной записи облачного развертывания в соответствии с возможностями владения, выставления счетов и управления вашей организации.

[Идентификация](./identity/overview.md). Интегрируйте облачные службы идентификации с имеющимися ресурсами идентификации для поддержки управления доступом в ИТ-среде.

[Принудительное применение политики](./policy-enforcement/overview.md). Определите и примените правила политики организации для ресурсов и рабочих нагрузок, которые можно развернуть в облаке.

[Согласованность ресурсов](./resource-consistency/overview.md). Убедитесь, что развертывание и организация облачных ресурсов выровнены для принудительного применения требований к управлению ресурсами и политикам.

[Добавление тегов к ресурсам](./resource-tagging/overview.md). Упорядочьте облачные ресурсы для поддержки моделей выставления счетов, подходов к учету облака, управления, контроля доступа и эффективности работы. Для добавления тегов к ресурсам требуется согласованное и хорошо организованное именование и схема метаданных.

[Программно-определяемые сети](./software-defined-network/overview.md). Разверните защищенные рабочие нагрузки в облако с помощью быстрого развертывания и модификации возможностей виртуализированной работы с сетью. Программно-определяемые сети (SDN) могут поддерживать гибкие рабочие процессы, изолировать ресурсы и интегрировать облачные системы с имеющейся ИТ-инфраструктурой.

[Шифрование](./encryption/overview.md). Защитите конфиденциальные данные с использованием шифрования — важным аспектом безопасности в рамках облачного развертывания.

[Журналы и отчеты](./log-and-report/overview.md). Отслеживайте данные журнала, создаваемые облачными ресурсами. Анализ данных предоставляет сведения об операциях, обслуживании и состоянии применения политик в рабочих нагрузках.

## <a name="next-steps"></a>Дополнительная информация

Узнайте, каким образом подписки и учетные записи используются в качестве базы для облачного развертывания.

> [!div class="nextstepaction"]
> [Проектирование подписок](subscriptions/overview.md)