---
title: CAF. Общие сведения о дисциплине "Согласованность ресурсов"
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Общие сведения о дисциплине "Согласованность ресурсов"
author: BrianBlanchard
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: ce29efab1502d59513528ab4b640173346aee516
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58241985"
---
# <a name="caf-resource-consistency-discipline-overview"></a>CAF. Общие сведения о дисциплине "Согласованность ресурсов"

Согласованность ресурсов является одной из [пяти дисциплин системы управления облаком](../governance-disciplines.md) в [модели управления CAF](../overview.md). Эта дисциплина направлена на создание политик, связанных с оперативным управлением рабочей средой, приложением или рабочей нагрузкой. Команды ИТ-специалистов часто выполняют мониторинг приложений, рабочей нагрузки и производительности ресурсов. Они также обычно выполняют задачи, которые поддерживают потребность в масштабировании, устраняют нарушения Соглашений об уровне обслуживания по аспектам производительности и предотвращают их, применяя автоматизированные исправления. В рамках пяти дисциплин системы управления облаком согласованность ресурсов — это дисциплина, которая обеспечивает согласованную настройку ресурсов таким образом, чтобы их можно было обнаружить с помощью ИТ-операций, добавление их в решения восстановления и повторяемые рабочие процессы.

> [!NOTE]
> Управление согласованностью ресурсов не заменит имеющихся ИТ-специалистов, процессов и процедур, которые позволят вашей организации эффективно управлять облачными ресурсами. Основная цель этой дисциплины — выявить потенциальные бизнес-риски и предоставить рекомендации по их снижению для ИТ-персонала, отвечающего за управление вашими ресурсами в облаке. При разработке политик и процессов управления не забудьте привлечь соответствующих ИТ-специалистов к процессам планирования и проверки.

В этом разделе CAF описывается, как разработать дисциплину согласованности ресурсов как часть стратегии управления облаком. Основная аудитория этого руководства — облачные архитекторы вашей организации и другие члены вашей команды системы управления облаком. Тем не менее решения, политики и процессы, вытекающие из этой дисциплины, должны включать привлечение соответствующих членов ИТ-команды, отвечающих за внедрение решениями по согласованности ресурсов в вашей организации и управление ими, а также обсуждение с ними связанных с этим вопросов.

Если вашей организации не хватает собственных специалистов в области стратегий согласованности ресурсов, рассмотрите возможность привлечения внешних консультантов как части этой дисциплины. Кроме того, рассмотрите возможность внедрения службы [Microsoft Consulting Services](https://www.microsoft.com/enterprise/services) либо [Microsoft FastTrack](https://azure.microsoft.com/programs/azure-fasttrack) для перехода на облачную службу, или других внешних экспертов по внедрению облачных технологий для обсуждения лучшего способа организации, отслеживания и оптимизации облачных ресурсов.

## <a name="policy-statements"></a>Правила политики

Действующие правила политики и соответствующие требования к архитектуре служат основой дисциплины согласованности ресурсов. Примеры правил политики см. в статье [Resource Consistency sample policy statements](./policy-statements.md) (Положения примера политики согласованности ресурсов). Эти примеры можно использовать в качестве отправной точки для политик системы управления организации.

> [!CAUTION]
> Примеры политик взяты из общего опыта клиентов. Чтобы лучше согласовать эти политики с конкретными потребностями системы управления облаком, выполните следующие шаги для создания политик, соответствующих уникальным потребностям вашей компании.

## <a name="developing-resource-consistency-governance-policy-statements"></a>Разработка правил политики управления согласованностью ресурсов

В следующих шести пунктах показаны примеры и потенциальные варианты, которые следует учитывать при разработке системы управления согласованностью ресурсов. Используйте каждый пункт в качестве начальной точки для обсуждений в вашей команде управления облаком и с действующей компанией, а также с ИТ-подразделениями в вашей организации для определения политик и процессов, необходимых для снижения рисков, связанных с согласованностью ресурсов.

<!-- markdownlint-disable MD033 -->

<ul class="panelContent cardsE">
<li style="display: flex; flex-direction: column;">
    <a href="./template.md">
        <div class="cardSize">
            <div class="cardPadding" >
                <div class="card" >
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../../_images/governance/process-template.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText" style="padding-left:0px;">
                        <h3>Шаблон согласованности ресурсов</h3>
                        <p class="x-hidden-focus">Скачайте шаблон для документирования дисциплины "Согласованность ресурсов"</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li><li style="display: flex; flex-direction: column;">
    <a href="./business-risks.md">
        <div class="cardSize">
            <div class="cardPadding" >
                <div class="card" >
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../../_images/governance/process-risks.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText" style="padding-left:0px;">
                        <h3>Бизнес-риски</h3>
                        <p class="x-hidden-focus">Вы должны распознать мотивы и риски, связанные с дисциплиной "Согласованность ресурсов".</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./metrics-tolerance.md">
        <div class="cardSize">
            <div class="cardPadding" >
                <div class="card" >
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../../_images/governance/process-metrics.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText" style="padding-left:0px;">
                        <h3>Индикаторы и метрики</h3>
                        <p class="x-hidden-focus">Показатели, которые помогут понять, настало ли время для инвестиций в дисциплину "Согласованность ресурсов".</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./compliance-processes.md">
        <div class="cardSize">
            <div class="cardPadding" >
                <div class="card" >
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../../_images/governance/process-enforce.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText" style="padding-left:0px;">
                        <h3>Процессы, обеспечивающие соблюдение политики</h3>
                        <p class="x-hidden-focus">Здесь предложены процессы для поддержки соблюдения политики в дисциплине "Согласованность ресурсов".</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./discipline-improvement.md">
        <div class="cardSize">
            <div class="cardPadding" >
                <div class="card" >
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../../_images/governance/process-maturity.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText" style="padding-left:0px;">
                        <h3>Зрелость</h3>
                        <p class="x-hidden-focus">Сопоставление развития системы управления облаком с этапами внедрения облачных технологий.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./toolchain.md">
        <div class="cardSize">
            <div class="cardPadding" >
                <div class="card" >
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../../_images/governance/process-toolchain.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText" style="padding-left:0px;">
                        <h3>Цепочка инструментов</h3>
                        <p class="x-hidden-focus">Службы Azure, которые можно использовать для поддержки дисциплины "Согласованность ресурсов".</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="next-steps"></a>Дополнительная информация

Начните с оценки [бизнес-рисков](./business-risks.md) в конкретной среде.

> [!div class="nextstepaction"]
> [Описание бизнес-рисков](./business-risks.md)
