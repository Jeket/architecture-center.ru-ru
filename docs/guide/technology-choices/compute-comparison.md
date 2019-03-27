---
title: Критерии для выбора вычислительной службы Azure
titleSuffix: Azure Application Architecture Guide
description: Сравнение вычислительных служб Azure по нескольким критериям.
author: MikeWasson
ms.date: 08/08/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: 2b6b9b941bf7a3c0136b71ecb65bfe4b4a59e07b
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58245605"
---
# <a name="criteria-for-choosing-an-azure-compute-service"></a>Критерии для выбора вычислительной службы Azure

Термин *вычислительная служба* означает модель размещения вычислительных ресурсов, которые используются для выполнения приложений. В приведенных ниже таблицах представлена сравнительная характеристика вычислительных служб Azure по нескольким критериям. Используйте эти таблицы при выборе оптимальной вычислительной службы для приложения.

## <a name="hosting-model"></a>Модель размещения

<!-- markdownlint-disable MD033 -->

| Критерии | Виртуальные машины | Служба приложений | Service Fabric | Функции Azure | Служба Azure Kubernetes | Экземпляры контейнеров | Пакетная служба Azure |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Структура приложения | Не влияет | Приложения, контейнеры | Службы, гостевые исполняемые файлы, контейнеры | Функции Azure | Контейнеры | Контейнеры | Запланированные задания  |
| Плотность | Не влияет | Несколько приложений в каждом экземпляре с использованием планов службы приложений | Несколько служб на каждой виртуальной машине | Бессерверные<a href="#note1"><sup>1</sup></a> | Несколько контейнеров на каждом узле |Нет выделенных экземпляров | Несколько приложений на каждой виртуальной машине |
| Минимальное количество узлов | 1 <a href="#note2"><sup>2</sup></a>  | 1 | 5 <a href="#note3"><sup>3</sup></a> | Бессерверные<a href="#note1"><sup>1</sup></a> | 3 <a href="#note3"><sup>3</sup></a> | Нет выделенных узлов | 1 <a href="#note4"><sup>4</sup></a> |
| Управление данными о состоянии | Без отслеживания или с отслеживанием состояния | Без отслеживания состояния | Без отслеживания или с отслеживанием состояния | Без отслеживания состояния | Без отслеживания или с отслеживанием состояния | Без отслеживания состояния | Без отслеживания состояния |
| Размещение веб-сайта | Не влияет | Встроено | Не влияет | Не применяется | Не влияет | Не влияет | Нет  |
| Можно ли развернуть в выделенной виртуальной сети? | Поддерживаются | Поддерживается<a href="#note5"><sup>5</sup></a> | Поддерживаются | Поддерживается <a href="#note5"><sup>5</sup></a> | [Поддерживаются](/azure/aks/networking-overview) | Не поддерживается | Поддерживаются |
| Гибридное подключение | Поддерживаются | Поддерживается <a href="#note6"><sup>6</sup></a>  | Поддерживаются | Поддерживается <a href="#node7"><sup>7</sup></a> | Поддерживаются | Не поддерживается | Поддерживаются |

Примечания

1. <span id="note1">Если используется план потребления. При использовании плана службы приложений функции выполняются на виртуальных машинах, выделенных для этого плана службы приложений. См. статью [Сравнение планов размещения для решения "Функции Azure"][function-plans].</span>
2. <span id="note2">Более строгие условия соглашения об уровне обслуживания, если используется несколько экземпляров.</span>
3. <span id="note3">Рекомендуется использовать в рабочей среде.</span>
4. <span id="note4">По завершении задания может сводиться к нулю.</span>
5. <span id="note5">Требует наличия среды службы приложений (ASE).</span>
6. <span id="note6">Используйте[ гибридные подключения к службе приложений Azure][app-service-hybrid].</span>
7. <span id="note7">Требуется план службы приложений.</span>

## <a name="devops"></a>DevOps

| Критерии | Виртуальные машины | Служба приложений | Service Fabric | Функции Azure | Служба Azure Kubernetes | Экземпляры контейнеров | Пакетная служба Azure |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Локальная отладка | Не влияет | IIS Express и другие <a href="#note1b"><sup>1</sup></a> | Локальный кластер узлов | Visual Studio или CLI Функций Azure | Minikube и др. | Локальная среда выполнения контейнера | Не поддерживается |
| Модель программирования | Не влияет | Веб-приложения и приложения API, веб-задания для фоновых задач | Гостевой исполняемый файл, модель службы, модель субъекта, контейнеры | Функции с триггерами | Не влияет | Не влияет | Приложение командной строки |
| Обновление приложения | Нет встроенной поддержки | Слоты развертывания | Последовательное обновление (для служб) | Слоты развертывания | Последовательное обновление | Не применяется |

Примечания

1. <span id="note1b">Возможные варианты: IIS Express для ASP.NET или node.js (iisnode); веб-сервер PHP; набор средств Azure для IntelliJ, набор средств Azure для Eclipse и т. д. Также служба приложений поддерживает удаленную отладку развернутого веб-приложения.</span>
2. <span id="note2b">См. статью о [поставщиках диспетчера ресурсов, регионах, версиях API и схемах][resource-manager-supported-services].</span>

## <a name="scalability"></a>Масштабируемость

| Критерии | Виртуальные машины | Служба приложений | Service Fabric | Функции Azure | Служба Azure Kubernetes | Экземпляры контейнеров | Пакетная служба Azure |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Автомасштабирование | Масштабируемые наборы виртуальных машин | Встроенная служба | наборы масштабирования виртуальных машин; | Встроенная служба | Не поддерживается | Не поддерживается | Недоступно |
| Подсистема балансировки нагрузки | Azure Load Balancer | Интегрировано | Azure Load Balancer | Интегрировано | Интегрировано |  Нет встроенной поддержки | Azure Load Balancer |
| Предел масштабирования<a href="#note1c"><sup>1</sup></a> | Образ платформы: 1000 узлов на маштабируемый набор виртуальных машин. Пользовательский образ: 100 узлов на VMSS | 20 экземпляров; 100 экземпляров при использовании среды службы приложений | 100 узлов на VMSS | 200 экземпляров на каждое приложение-функцию | 100 узлов в кластере (ограничение по умолчанию) |20 групп контейнеров на подписку (ограничение по умолчанию) | Ограничение составляет 20 ядер (ограничение по умолчанию) |

Примечания

1. <span id="note1c">См. дополнительные сведения о [подписке Azure, границах, квотах и ограничениях службы](/azure/azure-subscription-service-limits)</span>.

## <a name="availability"></a>Доступность

| Критерии | Виртуальные машины | Служба приложений | Service Fabric | Функции Azure | Служба Azure Kubernetes | Экземпляры контейнеров | Пакетная служба Azure |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| Соглашение об уровне обслуживания | [Соглашение об уровне обслуживания для виртуальных машин][sla-vm] | [Соглашение об уровне обслуживания для службы приложений][sla-app-service] | [Соглашение об уровне обслуживания для Azure Service Fabric][sla-sf] | [Соглашение об уровне обслуживания для решения "Функции Azure"][sla-functions] | [Соглашение об уровне обслуживания для AKS][sla-acs] | [Соглашение об уровне обслуживания для Экземпляры контейнеров](https://azure.microsoft.com/support/legal/sla/container-instances/) | [Соглашение об уровне обслуживания для пакетной службы Azure][sla-batch] |
| Отработка отказа в другой регион | Диспетчер трафика | Диспетчер трафика | Диспетчер трафика, кластер с несколькими регионами | Не поддерживается | Диспетчер трафика | Не поддерживается | Не поддерживается |

## <a name="other"></a>Другие

| Критерии | Виртуальные машины | Служба приложений | Service Fabric | Функции Azure | Служба Azure Kubernetes | Экземпляры контейнеров | Пакетная служба Azure |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| SSL | Настраивается на виртуальной машине | Поддерживаются | Поддерживаются  | Поддерживаются | [Контроллер входящего трафика](/azure/aks/ingress) | Используйте контейнер [расширения](../../patterns/sidecar.md) | Поддерживаются |
| Стоимость | [Windows][cost-windows-vm], [Linux][cost-linux-vm] | [Цены на службу приложений][cost-app-service] | [Цены на Service Fabric][cost-service-fabric] | [Цены на решение "Функции Azure"][cost-functions] | [Цены на AKS][cost-acs] | [Цены на Экземпляры контейнеров](https://azure.microsoft.com/pricing/details/container-instances/) | [Цены на пакетную службу Azure][cost-batch]
| Подходящие стили архитектуры | [n-уровневая][n-tier]; [большие вычисления][big-compute] (HPC) | [Интерфейс — очередь — рабочая роль][w-q-w], [n-уровневая][n-tier] | [Микрослужбы][microservices]; [управляемая событиями архитектура][event-driven] | [Микрослужбы][microservices]; [управляемая событиями архитектура][event-driven] | [Микрослужбы][microservices]; [управляемая событиями архитектура][event-driven] | [Микрослужбами][microservices]; автоматизация задач; пакетные задания  | [Большие вычисления][big-compute] (HPC) |

<!-- markdownlint-enable MD033 -->

[cost-linux-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/linux/
[cost-windows-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/windows/
[cost-app-service]: https://azure.microsoft.com/pricing/details/app-service/
[cost-service-fabric]: https://azure.microsoft.com/pricing/details/service-fabric/
[cost-functions]: https://azure.microsoft.com/pricing/details/functions/
[cost-acs]: https://azure.microsoft.com/pricing/details/kubernetes-service/
[cost-batch]: https://azure.microsoft.com/pricing/details/batch/

[function-plans]: /azure/azure-functions/functions-scale
[sla-acs]: https://azure.microsoft.com/support/legal/sla/kubernetes-service
[sla-app-service]: https://azure.microsoft.com/support/legal/sla/app-service/
[sla-batch]: https://azure.microsoft.com/support/legal/sla/batch/
[sla-functions]: https://azure.microsoft.com/support/legal/sla/functions/
[sla-sf]: https://azure.microsoft.com/support/legal/sla/service-fabric/
[sla-vm]: https://azure.microsoft.com/support/legal/sla/virtual-machines/

[resource-manager-supported-services]: /azure/azure-resource-manager/resource-manager-supported-services
[scale-acs]: /azure/container-service/kubernetes/container-service-scale#scaling-considerations

[n-tier]: ../architecture-styles/n-tier.md
[w-q-w]: ../architecture-styles/web-queue-worker.md
[microservices]: ../architecture-styles/microservices.md
[event-driven]: ../architecture-styles/event-driven.md
[big-date]: ../architecture-styles/big-data.md
[big-compute]: ../architecture-styles/big-compute.md

[app-service-hybrid]: /azure/app-service/app-service-hybrid-connections
