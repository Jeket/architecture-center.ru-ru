---
title: "Подключение локальной сети к Azure"
description: "Рекомендуемые архитектуры для обеспечения безопасных и надежных сетевых подключений между локальными сетями и Azure."
layout: LandingPage
ms.openlocfilehash: 372efb8ecf69245a5895c51e3da156a348bd665e
ms.sourcegitcommit: 2123c25b1a0b5501ff1887f98030787191cf6994
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/08/2018
---
<!-- This file is generated! -->
<!-- See the templates in ./build/reference-architectures  -->
<!-- See data in index.json -->

# <a name="connect-an-on-premises-network-to-azure"></a>Подключение локальной сети к Azure

В этих эталонных архитектурах демонстрируются проверенные методы для создания надежного сетевого соединения между локальной сетью и Azure. [Какой вариант следует использовать?](./considerations.md)

<section class="series">
    <ul class="panelContent">
    <!-- VPN -->
<li style="display: flex; flex-direction: column;">
    <a href="./vpn.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/vpn.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Виртуальная частная сеть</h3>
                        <p>Расширение локальной сети в Azure с помощью подключения VPN "сеть — сеть".</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- ExpressRoute -->
<li style="display: flex; flex-direction: column;">
    <a href="./expressroute.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/expressroute.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>ExpressRoute</h3>
                        <p>Расширение локальной сети в Azure с помощью Microsoft Azure ExpressRoute.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- ExpressRoute with VPN failover -->
<li style="display: flex; flex-direction: column;">
    <a href="./expressroute-vpn-failover.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/expressroute-vpn-failover.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>ExpressRoute с отработкой отказа на VPN</h3>
                        <p>Расширение локальной сети в Azure с помощью Azure ExpressRoute с VPN в качестве отказоустойчивого соединения.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Hub-spoke topology -->
<li style="display: flex; flex-direction: column;">
    <a href="./hub-spoke.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/hub-spoke.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Звездообразная топология</h3>
                        <p>Концентратор представляет собой центральную точку подключения к локальной сети. Периферийные зоны — это виртуальные сети, которые устанавливают пиринг с концентратором и могут использоваться для изоляции рабочих нагрузок.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Hub-spoke topology with shared services -->
<li style="display: flex; flex-direction: column;">
    <a href="./shared-services.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/shared-services.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Звездообразная топология с общими службами</h3>
                        <p>Развертывание звездообразной топологии, которая содержит общие службы, в том числе службы Active Directory и виртуальный сетевой модуль (NVA). Общие службы могут использоваться всеми периферийными зонами.</p>
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