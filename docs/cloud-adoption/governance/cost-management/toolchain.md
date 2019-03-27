---
title: CAF. Средства управления затратами в Azure
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Средства управления затратами в Azure
author: BrianBlanchard
ms.openlocfilehash: 58dfa604863f704fd9b9fbb8d0693447cecdaf84
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58246605"
---
# <a name="cost-management-tools-in-azure"></a>Средства управления затратами в Azure

[Управление затратами](overview.md) — это одна из [пяти дисциплин управления облаком](../governance-disciplines.md). Эта дисциплина сосредоточена на способах составления планов облачных расходов, распределения облачных бюджетов, мониторинга и реализации облачных бюджетов, выявления дорогостоящих аномалий и корректировки плана управления облаком, когда фактические расходы не совпадают.

Ниже приведен список собственных инструментов Azure, которые могут помочь в совершенствовании политик и процессов, поддерживающих эту дисциплину управления.

|  | [портал Azure](https://azure.microsoft.com/features/azure-portal/)  | [Управление затратами Azure](/azure/cost-management/overview-cost-mgt)  | [Пакет содержимого Azure EA](/power-bi/service-connect-to-azure-enterprise)  | [Политика Azure](/azure/governance/policy/overview) |
|---------|---------|---------|---------|---------|
|Требуется ли соглашение Enterprise?     | Нет          | Да (не обязательно для [Cloudyn](/azure/cost-management/overview))         | Yes         | Нет          |
|Управление бюджетом     | Нет          | Yes         | Нет          | Yes         |
|Отслеживание затрат на отдельном ресурсе    | Yes         | Да         | Да         | Нет          |
|Отслеживание затрат на нескольких ресурсах    | Нет          | Yes        | Да         | Нет          |
|Контроль затрат на отдельном ресурсе     | Да (изменение размера вручную)         | Yes         | Нет          | Yes         |
|Применение затрат для нескольких ресурсов    | Нет          | Yes         | Нет          | Yes         |
|Отслеживание и выявление тенденций     | Да (ограничено)         | Yes        | Да         | Нет          |
|Обнаружение аномалий в затратах     | Нет          | Yes        | Да         | Нет         |
|Социализация отклонений     | Нет         | Yes        | Да        | Нет         |
