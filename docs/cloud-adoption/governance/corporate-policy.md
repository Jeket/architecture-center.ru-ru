---
title: CAF. Реализация стратегии управления облаком
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
description: Узнайте, как использовать Microsoft Cloud Adoption Framework для Azure (CAF) для реализации стратегии управления облаком.
author: BrianBlanchard
ms.date: 1/3/2019
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: 88aeb92157a37baacc34884de67024ec87b20980
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55902399"
---
# <a name="implement-a-cloud-governance-strategy"></a>Реализация стратегии управления облаком

<!-- markdownlint-disable MD033 -->

<ul class="panelContent cardsI">
<li style="display: flex; flex-direction: column;">
    <div class="cardSize">
        <div class="cardPadding" style="padding-bottom:10px;">
            <div class="card" style="padding-bottom:10px;">
                <div class="cardText" style="padding-left:0px;">
Любое изменение бизнес-процессов или технологических платформ создает риск для бизнеса. Перед командами управления облаком, членов которых иногда называют хранителями облачных сред, стоит задача снижения этих рисков с минимальным вмешательством в усилия по внедрению или внедрению инноваций.<br/><br/>Тем не менее управление облаком требует не только технической реализации. Незначительные изменения в корпоративной концепции или корпоративной политике могут существенно повлиять на внедрение. Перед внедрением важно не ограничиваться ИТ-ресурсами, а определять корпоративную политику.<br/><br/>
                </div>
            </div>
        </div>
    </div>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="../_images/operational-transformation-govern-highres.png" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize">
            <div class="cardPadding" style="padding-bottom:10px;">
                <div class="card" style="padding-bottom:10px;">
                    <div class="cardText" style="padding-left:0px;">
<img src="../_images/operational-transformation-govern-highres.png" alt="Diagram of the CAF governance model: Corporate policy and governance disciplines">
<br>
<i>Рис. 1. Схема корпоративной политики и пяти дисциплин управления облаком</i>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

<!-- markdownlint-enable MD033 -->

## <a name="define-corporate-policy"></a>Определение корпоративной политики

Определение корпоративной политики направлено на выявление и снижение бизнес-рисков, независимо от облачной платформы. Работоспособная стратегия управления облаком начинается с обоснованной корпоративной политики. Следующий трехэтапный процесс направляет последовательную разработку таких политик.

<!-- markdownlint-disable MD033 -->

<ul  class="panelContent cardsF">
<li style="display: flex; flex-direction: column;">
    <a href="./policy-compliance/understanding-business-risk.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/governance/business-risk.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Бизнес-риск</h3>
                        <p>Изучите текущие планы внедрения облачных вычислений и классификации данных, чтобы определить риски для бизнеса. Работайте с бизнесом, чтобы сбалансировать риски и снизить затраты.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./policy-compliance/define-policy.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/governance/corporate-policy.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Политика и соответствие требованиям</h3>
                        <p>Оцените допустимость риска, чтобы информировать о минимально агрессивных политиках, которые управляют внедрением облака и снижают риски. В некоторых отраслях соблюдение требований сторонних производителей влияет на создание исходной политики.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./policy-compliance/processes.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/governance/enforcement.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Процессы</h3>
                        <p>Темп внедрения и инновационная деятельность естественным образом приведет к нарушениям политики. Выполнение соответствующих процессов поможет в мониторинге и обеспечении соблюдения политики.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

<!-- markdownlint-enable MD033 -->

## <a name="next-steps"></a>Дополнительная информация

Надежная стратегия управления облаком начинается с понимания бизнес-рисков.

> [!div class="nextstepaction"]
> [How does business risk change in the cloud?](./policy-compliance/understanding-business-risk.md) (Изменения бизнес-риска в облаке)
