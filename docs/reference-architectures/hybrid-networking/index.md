---
title: "Подключение локальной сети к Azure"
description: "Рекомендуемые архитектуры для обеспечения безопасных и надежных сетевых подключений между локальными сетями и Azure."
layout: LandingPage
ms.openlocfilehash: 0707d17295e338af0176bd0cea806615ef05f9ad
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/14/2017
---
# <a name="connect-an-on-premises-network-to-azure"></a>Подключение локальной сети к Azure

В этих эталонных архитектурах демонстрируются проверенные методы для создания надежного сетевого соединения между локальной сетью и Azure. [Какой вариант следует использовать?](./considerations.md)

<ul class="panelContent">
    <li>
        <a href="./vpn.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                            <img src="./images/vpn.svg">
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
    <li>
        <a href="./expressroute.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                            <img src="./images/expressroute.svg">
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>ExpressRoute</h3>
                            <p>Расширение локальной сети в Azure с помощью Azure ExpressRoute</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <li>
        <a href="./expressroute-vpn-failover.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                            <img src="./images/expressroute-vpn-failover.svg">
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>ExpressRoute с отработкой отказа VPN</h3>
                            <p>Расширение локальной сети в Azure с помощью Azure ExpressRoute с VPN в качестве отказоустойчивого соединения.</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <li>
        <a href="./hub-spoke.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                            <img src="./images/hub-spoke.svg">
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>Звездообразная топология</h3>
                            <p>Концентратор представляет собой центральную точку подключения к локальной сети. Периферийные зоны — это виртуальные сети, которые устанавливают пиринг с концентратором и могут использоваться для изоляции рабочих нагрузок. </p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
</ul>

