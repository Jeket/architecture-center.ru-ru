---
title: Мониторинг Azure Databricks с помощью Azure Monitor
description: Библиотека Scala для включения мониторинга Azure Databricks в Azure Log Analytics
author: petertaylor9999
ms.date: 03/26/2019
ms.openlocfilehash: 4544d3abc3264ec459a80ac1a61a912e6d30d6b2
ms.sourcegitcommit: 1a3cc91530d56731029ea091db1f15d41ac056af
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/03/2019
ms.locfileid: "58887698"
---
# <a name="monitoring-azure-databricks"></a>Мониторинг Azure Databricks

[Azure Databricks](/azure/azure-databricks/) — это высокопроизводительная и многофункциональная служба аналитики на основе [Apache Spark](https://spark.apache.org/), которая позволяет без труда разрабатывать и развертывать решения аналитики больших данных и искусственного интеллекта (ИИ). Многие пользователи по достоинству оценили простоту записных книжек в решениях Azure Databricks. Если же требуются более надежные вычислительные ресурсы, Azure Databricks поддерживает распределенное выполнение пользовательского кода приложений.

Мониторинг — это важнейшая часть любого решения уровня рабочей среды. Azure Databricks предоставляет надежные функции для мониторинга метрик пользовательских приложений, потоковой передачи событий запроса и отправки сообщений журнала приложений. Данные мониторинга Azure Databricks можно отправлять в различные службы ведения журналов.

В перечисленных ниже статьях объясняется, как отправлять данные мониторинга из Azure Databricks в [Azure Monitor](/azure/azure-monitor/overview), платформу данных мониторинга для Azure. Изучайте эти статьи в указанном порядке.

1. [Настройка Azure Databricks для отправки метрик в Azure Monitor](./configure-cluster.md)
1. [Отправка журналов приложения Azure Databricks в Azure Monitor](./application-logs.md)
1. [Визуализация метрик Azure Databricks с помощью панелей мониторинга](./dashboards.md)

Библиотека кода, сопровождающая эти статьи, расширяет основные возможности мониторинга Azure Databricks для отправки метрик, событий и данных журналов Spark в Azure Monitor.

Эти статьи и сопровождающая их библиотека кода предназначены для разработчиков решений Apache Spark и Azure Databricks. Код нужно встроить в файлы архива Java (JAR), а затем развернуть в кластере Azure Databricks. Код представляет собой сочетание [Scala](https://www.scala-lang.org/) и Java с соответствующим набором файлов объектной модели проекта (POM) [Maven](https://maven.apache.org) для создания выходных JAR-файлов. Перед началом работы рекомендуем изучить общие сведения о Java, Scala и Maven.

## <a name="next-steps"></a>Дополнительная информация

Сначала создайте библиотеку кода и разверните ее в кластере Azure Databricks.

> [!div class="nextstepaction"]
> [Настройка Azure Databricks для отправки метрик в Azure Monitor](./configure-cluster.md)