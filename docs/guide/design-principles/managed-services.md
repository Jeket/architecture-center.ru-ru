---
title: Использование управляемых служб
titleSuffix: Azure Application Architecture Guide
description: При возможности выбирайте службы PaaS (платформа как услуга) вместо служб IaaS (инфраструктура как услуга).
author: MikeWasson
ms.date: 08/30/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: f08914e1eacc4f02fdb16093fe590f46249e3da3
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58249539"
---
# <a name="use-managed-services"></a>Использование управляемых служб

## <a name="when-possible-use-platform-as-a-service-paas-rather-than-infrastructure-as-a-service-iaas"></a>При возможности выбирайте службы PaaS (платформа как услуга), а не IaaS (инфраструктура как услуга).

IaaS предоставляет лишь набор компонентов. Вы можете собрать из них что угодно, но придется делать это самостоятельно. Управляемые службы намного удобнее в настройке и администрировании. Вам не придется подготавливать виртуальные машины, настраивать виртуальные сети, управлять исправлениями и обновлениями, а также беспокоиться о множестве других проблем, связанных с работой программного обеспечения на виртуальной машине.

Предположим, что в вашем приложении нужна очередь сообщений. Чтобы настроить собственную службу обмена сообщениями на виртуальной машине,вам потребуется установить, например, RabbitMQ. А ведь служебная шина Azure уже предоставляет надежную службу для обмена сообщениями, и ее гораздо проще настроить. Достаточно создать пространство имен служебной шины (например, прямо в скрипте развертывания) и обратиться к ней через клиентский пакет SDK.

Конечно, у вашего приложения могут быть особые требования, для которых модель IaaS подходит лучше. Но даже для приложений на основе службы IaaS можно найти возможность естественного применения управляемых служб. Например, служб кэширования, управления очередями и хранилища данных.

| Вместо службы... | рекомендуем использовать... |
|-----------------------|-------------|
| Active Directory | Доменные службы Azure Active Directory |
| Elasticsearch | Поиск Azure |
| Hadoop | HDInsight |
| IIS | Служба приложений |
| MongoDB | База данных Cosmos |
| Redis | кэш Azure Redis |
| SQL Server; | Базы данных SQL Azure |
