---
title: "Рабочие нагрузки виртуальных машин Linux"
description: "Описание некоторых общих архитектур для развертывания виртуальных машин, в которых размещены приложения корпоративного масштаба в Azure."
layout: LandingPage
ms.openlocfilehash: ef06fb93355f4676f44954930bcfeb2c2d012d43
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/23/2018
---
<!-- This file is generated! -->
<!-- See the templates in ./build/reference-architectures  -->
<!-- See data in index.json -->

# <a name="linux-vm-workloads"></a>Рабочие нагрузки виртуальных машин Linux

В этих эталонных архитектурах демонстрируются проверенные методы для запуска виртуальных машин Linux в Azure.

<section class="series">
    <ul class="panelContent">
    <!-- Single VM -->
<li style="display: flex; flex-direction: column;">
    <a href="./single-vm.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/single-vm.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Одиночная виртуальная машина</h3>
                        <p>Базовые рекомендации для запуска любой виртуальной машины Linux в Azure.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Load balanced VMs -->
<li style="display: flex; flex-direction: column;">
    <a href="./multi-vm.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/multi-vm.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Виртуальные машины с балансировкой нагрузки</h3>
                        <p>Несколько виртуальных машин, размещенных за подсистемой балансировки нагрузки для обеспечения масштабируемости и доступности.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- N-tier application -->
<li style="display: flex; flex-direction: column;">
    <a href="./n-tier.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/n-tier.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>n-уровневневое приложение</h3>
                        <p>Виртуальные машины настраиваются для n-уровневого приложения с помощью Apache Cassandra.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Multi-region application -->
<li style="display: flex; flex-direction: column;">
    <a href="./multi-region-application.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/multi-region-application.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Приложение для нескольких регионов</h3>
                        <p>n-уровневое приложение, развернутое в двух регионах для обеспечения высокой доступности.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    </ul>
</section>

<ul class="panelContent cardsI">
</ul>