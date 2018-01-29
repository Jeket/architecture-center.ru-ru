---
title: "Эталонная архитектура Azure"
description: "Эталонные архитектуры, проекты и рекомендуемые руководства по реализации для общих нагрузок в Azure."
layout: LandingPage
ms.openlocfilehash: 0cc03f87dae39517e1a72a65d4767dcc21879d8f
ms.sourcegitcommit: 9998334bebccb86be0f715ac7dffc0c3175aea68
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/26/2018
---
# <a name="azure-reference-architectures"></a>Эталонная архитектура Azure

Эталонные архитектуры упорядочены по сценарию, а связанные архитектуры сгруппированы. Каждая архитектура содержит предлагаемые методики, а также рекомендации по масштабируемости, доступности, управляемости и безопасности. В большинство из них также включено развертываемое решение.

<section class="series">
    <ul class="panelContent">
    <!--Windows VM -->
    <li>
        <a href="./virtual-machines-windows/index.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                                <img src="./virtual-machines-windows/images/n-tier.svg" height="140px" />
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>Рабочие нагрузки виртуальных машин Windows</h3>
                            <p>Эта статьи начинаются с рекомендаций по запуску одной виртуальной машины Windows, а затем нескольких виртуальных машин с балансировкой нагрузки и, наконец, запуском n-уровневого приложения в нескольких регионах.</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <!-- Linux VM -->
    <li>
        <a href="./virtual-machines-linux/index.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                                <img src="./virtual-machines-linux/images/n-tier.svg" height="140px" />
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>Рабочие нагрузки виртуальных машин Linux</h3>
                            <p>Эта статьи начинаются с рекомендации по запуску одной виртуальной машины Linux, а затем нескольких виртуальных машин с балансировкой нагрузки и, наконец, запуском n-уровневого приложения в нескольких регионах.</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <!-- Hybrid network -->
    <li>
        <a href="./hybrid-networking/index.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                                <img src="./hybrid-networking/images/vpn.svg" height="140px" />
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>Гибридная сеть</h3>
                            <p>В этих статьях показаны варианты создания сетевого соединения между локальной сетью и Azure.</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <!-- DMZ -->
    <li>
        <a href="./dmz/index.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                                <img src="./dmz/images/secure-vnet-dmz.svg" height="140px" />
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>Сеть периметра</h3>
                            <p>В этих статьях показано, как создать сеть периметра для защиты границы между виртуальной сетью Azure и локальной сетью или Интернетом.</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <!-- Identity -->
    <li>
        <a href="./identity/index.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                                <img src="./identity/images/adds-extend-domain.svg" height="140px" >
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>Управление удостоверениями</h3>
                            <p>В этих статьях представлены варианты интеграции локальной среды Active Directory (AD) с сетью Azure.</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <!-- Managed web app -->
    <li>
        <a href="./app-service-web-app/index.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                                <img src="./app-service-web-app/images/scalable-web-app.svg" height="140px" />
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>Веб-приложение службы приложений</h3>
                            <p>В этих статьях показаны лучшие методики для веб-приложений, использующих службу приложений Azure.</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    </ul>

<ul class="panelContent cardsI">
<li>
    <a href="./jenkins/index.md">
    <div class="cardSize">
        <div class="cardPadding">
            <div class="card">
                <div class="cardImageOuter">
                    <div class="cardImage">
                        <img src="./jenkins/images/logo.svg" alt="Jenkins" height="100%" />
                    </div>
                </div>
                <div class="cardText">
                    <h3>Сервер сборки Jenkins</h3>
                    <p>Развертывание и использование масштабируемого корпоративного сервера Jenkins в Azure.</p>
                </div>
            </div>
        </div>
    </div>
    </a>
</li>

<li>
    <a href="./sharepoint/index.md">
    <div class="cardSize">
        <div class="cardPadding">
            <div class="card">
                <div class="cardImageOuter">
                    <div class="cardImage">
                        <img src="./sharepoint/images/sharepoint.svg" alt="SharePoint Server 2016" height="100%" />
                    </div>
                </div>
                <div class="cardText">
                    <h3>Ферма SharePoint Server 2016</h3>
                    <p>Развертывание и запуск высокодоступных ферм SharePoint Server 2016 с использованием групп доступности AlwaysOn для SQL Server в Azure.</p>
                </div>
            </div>
        </div>
    </div>
    </a>
</li>

<li>
    <a href="./sap/index.md">
    <div class="cardSize">
        <div class="cardPadding">
            <div class="card">
                <div class="cardImageOuter">
                    <div class="cardImage">
                        <img src="./sap/images/sap.svg" alt="SAP NetWeaver and SAP HANA" width="100%" />
                    </div>
                </div>
                <div class="cardText">
                    <h3>SAP NetWeaver и SAP HANA</h3>
                    <p>Развертывание и запуск SAP NetWeaver и SAP HANA в среде с высокой доступностью Azure.</p>
                </div>
            </div>
        </div>
    </div>
    </a>
</li>
</ul>


</section>

