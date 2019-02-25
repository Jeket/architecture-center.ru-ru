---
title: CAF. Общие сведения о дисциплине базовой системы идентификации
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Описание базовый системы идентификации в отношении системы управления облаком
author: BrianBlanchard
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: 14569b8db68434ee30fa7bff7fba2da5fa537049
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55902511"
---
# <a name="caf-identity-baseline-discipline-overview"></a>CAF. Общие сведения о дисциплине базовой системы идентификации

Базовая система идентификации является одной из [пяти дисциплин системы управления облаком](../governance-disciplines.md) в [модели управления CAF](../overview.md). Идентификатор все чаще рассматривается в качестве основного периметра безопасности в облаке, что является переходом от традиционного подхода к безопасности сети. Службы идентификации предоставляют основные механизмы, поддерживающие контроль доступа и организацию ресурсов в информационной среде, а дисциплина "Базовая система идентификации" дополняет [дисциплину "Базовая система безопасности"](../security-baseline/overview.md), последовательно применяя требования проверки подлинности и авторизации в рамках усилий по принятию облака.

> [!NOTE]
> Система управления базовой идентификацией не заменяет существующие ИТ-подразделения, процессы и процедуры, которые позволяют вашей организации управлять службами идентификации и защищать их. Основная цель этой дисциплины — выявить потенциальные бизнес-риски, связанные с идентификацией, и предоставить рекомендации по их устранению для ИТ-специалистов, отвечающих за внедрение, обслуживание и эксплуатацию инфраструктуры управления идентификацией. При разработке политик и процессов управления не забудьте привлечь соответствующих ИТ-специалистов к процессам планирования и проверки.

Этот раздел руководства CAF описывает подход к развитию дисциплины "Базовая система идентификации" как части стратегии системы управления облаком. Основная аудитория этого руководства — облачные архитекторы вашей организации и другие участники вашей команды системы управления облаком. Тем не менее решения, политики и процессы, вытекающие из этой дисциплины, должны включать привлечение соответствующих участников ИТ-команды, отвечающих за внедрение решениями по управлению идентификацией в вашей организации и управление ими, а также обсуждение с ними связанных с этим вопросов.

Если вашей организации не хватает внутреннего опыта в области базовой системы идентификации и безопасности, рассмотрите возможность привлечения внешних консультантов в рамках этой дисциплины. Кроме того, рассмотрите возможность привлечения [служб консультирования Майкрософт](https://www.microsoft.com/enterprise/services), службы [Microsoft FastTrack](https://azure.microsoft.com/programs/azure-fasttrack) по внедрению облака или других внешних экспертов по внедрению облачных технологий для обсуждения проблем, связанных с этой дисциплиной.

## <a name="policy-statements"></a>Правила политики

Действующие правила политики и соответствующие требования к архитектуре служат основой дисциплины "Базовая система идентификации". Примеры правил политики см. в [этой статье](./policy-statements.md). Эти примеры можно использовать в качестве отправной точки для политик системы управления организации.

> [!CAUTION]
> Примеры политик взяты из общего опыта клиентов. Чтобы лучше согласовать эти политики с конкретными потребностями системы управления облаком, выполните следующие шаги для создания правил политики, соответствующих уникальным потребностям вашей компании.

## <a name="developing-identity-baseline-governance-policy-statements"></a>Разработка правил политики системы управления базовой идентификацией

В следующих шести пунктах показаны примеры и потенциальные варианты, которые следует учитывать при разработке системы управления базовых способов идентификации. Используйте каждый пункт в качестве начальной точки для обсуждений в вашей команде управления облачными решениями и с действующей компанией, а также с ИТ-подразделениями в вашей организации для определения политик и процессов, необходимых для снижения рисков, связанных с идентификацией.

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
                        <h3>Шаблон базовой системы идентификации</h3>
                        <p class="x-hidden-focus">Скачайте шаблон для документирования дисциплины "Базовая система идентификации".</p>
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
                        <p class="x-hidden-focus">Вы должны распознать мотивы и риски, связанные с дисциплиной "Базовая система идентификации".</p>
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
                        <p class="x-hidden-focus">Показатели, которые помогут понять, настало ли время для инвестиций в дисциплину "Базовая система идентификации".</p>
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
                        <p class="x-hidden-focus">Здесь предложены процессы для поддержки соблюдения политики в дисциплине "Базовая система идентификации".</p>
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
                        <p class="x-hidden-focus">Службы Azure, которые можно использовать для поддержки дисциплины "Базовая система идентификации".</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

<!-- markdownlint-enable MD033 -->

## <a name="next-steps"></a>Дополнительная информация

Начните с оценки [бизнес-рисков](./business-risks.md) в конкретной среде.

> [!div class="nextstepaction"]
> [Описание бизнес-рисков](./business-risks.md)
