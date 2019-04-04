---
title: Проектирование микрослужб в Azure
description: Проектирование, создание и использование архитектуры микрослужб в Azure
ms.date: 03/07/2019
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: e74d6f6098eb68c8bbd737cf3a047ce5de04327d
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58346150"
---
# <a name="building-microservices-on-azure"></a>Проектирование микрослужб в Azure

<!-- markdownlint-disable MD033 -->

<img src="../_images/microservices.svg" style="float:left; margin-top:8px; margin-right:8px; max-width: 80px; max-height: 80px;"/>

Архитектура микрослужб широко используется для создания устойчивых, высокомасштабируемых, независимо развертываемых и быстро растущих приложений. Но для этого требуется другой подход к проектированию приложений.

<ul  class="panelContent cardsZ">
<li style="display: flex; flex-direction: column;">
    <a href="./introduction.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Что такое микрослужбы?</h3>
                        <p>Чем отличается архитектура микрослужб и когда ее следует использовать?</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="../guide/architecture-styles/microservices.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Стиль архитектуры микрослужб</h3>
                        <p>Общие сведения об архитектуре микрослужб</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="examples-of-microservices-architectures"></a>Примеры архитектуры микрослужб

<ul  class="panelContent cardsZ">
<li style="display: flex; flex-direction: column;">
    <a href="../example-scenario/infrastructure/service-fabric-microservices.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Использование Service Fabric для декомпозиции монолитных приложений</h3>
                        <p>Итеративный подход к декомпозиции веб-сайта ASP.NET на микрослужбы.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="../example-scenario/data/ecommerce-order-processing.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Масштабируемая обработка заказов в Azure</h3>
                        <p>Обработка заказов с использованием модели функционального программирования, реализованной с помощью микрослужб.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="build-a-microservices-application"></a>Создание приложений для микрослужб

<ul  class="panelContent cardsZ">
<li style="display: flex; flex-direction: column;">
    <a href="./model/domain-analysis.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Анализ предметной области для моделирования микрослужб</h3>
                        <p>Чтобы избежать некоторых распространенных ошибок, используйте анализ предметной области для определения границ микрослужб.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="../reference-architectures/microservices/aks.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Эталонная архитектура для Службы Azure Kubernetes (AKS)</h3>
                        <p>Эта эталонная архитектура представляет базовую конфигурацию AKS, которая может быть отправной точкой для большинства развертываний.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="../reference-architectures/microservices/service-fabric.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Эталонная архитектура для Azure Service Fabric</h3>
                        <p>Эта эталонная архитектура представляет рекомендуемую конфигурацию, которая может быть отправной точкой для большинства развертываний.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./design/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Проектирование архитектуры микрослужб</h3>
                        <p>В этих статьях подробно описаны способы создания приложений микрослужб на основе эталонной реализации с использованием Службы Azure Kubernetes (AKS).</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./design/patterns.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Шаблоны проектирования</h3>
                        <p>Набор полезных конструктивных шаблонов для микрослужб.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="operate-microservices-in-production"></a>Использование микрослужб в рабочей среде

<ul  class="panelContent cardsZ">
<li style="display: flex; flex-direction: column;">
    <a href="./logging-monitoring.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Ведение журналов и мониторинг</h3>
                        <p>Распределенный характер архитектуры микрослужб предполагает внимательное отношение к ведению журналов и мониторингу.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./ci-cd.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Непрерывная интеграция и развертывание</h3>
                        <p>Использование непрерывной интеграции и непрерывной поставки (CI/CD) является основным требованием для успешной работы с микрослужбами.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>
