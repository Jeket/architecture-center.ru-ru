---
title: CAF. Пять аспектов управления облаком
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Общие сведения о системе управления содержимым в CAF
author: BrianBlanchard
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: ddd7f5e4b7e79ebe2ddf87be7421b6c4691aced1
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/17/2019
ms.locfileid: "59639875"
---
# <a name="the-five-disciplines-of-cloud-governance"></a>Пять аспектов управления облаком

<!-- markdownlint-disable MD033 -->

<ul class="panelContent cardsI">
<li style="display: flex; flex-direction: column;">
    <div class="cardSize">
        <div class="cardPadding" style="padding-bottom:10px;">
            <div class="card" style="padding-bottom:10px;">
                <div class="cardText" style="padding-left:0px;">
Любое изменение бизнес-процессов или технологических платформ создает риск. Перед командами управления облаком, членов которых иногда называют хранителями облачных сред, стоит задача снижения этих рисков с минимальным вмешательством в усилия по реализации или внедрению инноваций.<br/><br/>Модель управления КАФЕ проведет эти решения (независимо от выбранной облачной платформы), сосредоточившись на разработку корпоративной политики и <a href="#disciplines-of-cloud-governance">дисциплин облачная система управления</a>. <a href="./journeys/overview.md">Практические путей взаимодействия</a> демонстрации этой модели, с помощью служб Azure. Эта статья служит целевой страницей для пяти дисциплин модели управления CAF.
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
<i>Рис. 1. Схема корпоративной политики и пять дисциплины облако руководства</i>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

<!-- markdownlint-enable MD033 -->

## <a name="disciplines-of-cloud-governance"></a>Дисциплины управления облаком

Для каждого поставщика облачных служб существуют Общие руководства дисциплин, может служить руководством для информирования политик и выровнять цепочек инструментов. Эти дисциплины определяют решения относительно надлежащего уровня автоматизации и обеспечения соблюдения корпоративной политики в поставщиках облачных служб.

<!-- markdownlint-disable MD033 -->

<ul class="panelContent cardsA">
<li style="display: flex; flex-direction: column;">
    <a href="./cost-management/overview.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/governance/cost-management.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Управление затратами</h3>
                        <p>Стоимость является основной проблемой пользователей облака. Разработайте политику контроля затрат для всех облачных платформ.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./security-baseline/overview.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/governance/security-baseline.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Основные способы защиты</h3>
                        <p>Безопасность — это сложная и личная тема, уникальная для каждой компании. После установления требований безопасности политики и процессы применения управления облаком применяют эти требования ко всем конфигурациям сети, данных и ресурсов.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./identity-baseline/overview.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/governance/identity-baseline.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Основные способы идентификации</h3>
                        <p>Несоответствия в применении требований к удостоверению могут увеличить риск нарушения. Дисциплина "Основные способы идентификации" посвящена способам обеспечения того, чтобы идентификация последовательно применялась в рамках усилий по внедрению облака.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./resource-consistency/overview.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/governance/resource-consistency.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Согласованность ресурсов</h3>
                        <p>Облачные операции зависят от согласованности конфигурации ресурсов. За счет инструментария управления ресурсы можно последовательно настраивать для снижения рисков, связанных с подключением, смещением, обнаруживаемостью и восстановлением.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./deployment-acceleration/overview.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/governance/deployment-acceleration.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Ускорение развертывания</h3>
                        <p>Централизация, стандартизация и согласованность в подходах к развертыванию и настройке улучшают методы управления. Благодаря предоставлению доступа с помощью облачных средств управления они создают облачный фактор, который может ускорить действия развертывания.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

<!-- markdownlint-enable MD033 -->
