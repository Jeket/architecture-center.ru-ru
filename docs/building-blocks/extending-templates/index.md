---
title: "Расширение функциональности шаблонов Azure Resource Manager"
description: "Здесь описываются советы и рекомендации по расширению функциональных возможностей шаблонов Azure Resource Manager"
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: d9cad6cfbfe8d42bcb21ef9e77504b80aef2d694
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/14/2017
---
# <a name="extend-azure-resource-manager-template-functionality"></a>Расширение функциональности шаблонов Azure Resource Manager

В 2016 году команда разработчиков шаблонов и рекомендаций AzureCAT создала набор [стандартных блоков шаблонов](https://github.com/mspnp/template-building-blocks/wiki) Azure Resource Manager для упрощения развертывания ресурсов. Каждый стандартный блок состоит из набора готовых шаблонов, которые развертывают наборы ресурсов, заданные отдельными файлами параметров.

Шаблоны стандартных блоков объединяются для создания более крупных и более сложных вариантов развертывания. Например, для развертывания виртуальной машины в Azure требуется виртуальная сеть, учетные записи хранения и другие ресурсы. [Шаблон стандартных блоков виртуальной сети](https://github.com/mspnp/template-building-blocks/wiki/VNet-(v1)) развертывает виртуальную сеть и подсети. [Шаблон стандартных блоков виртуальной машины](https://github.com/mspnp/template-building-blocks/wiki/Windows-and-Linux-VMs-(v1)) развертывает учетные записи хранения, сетевые интерфейсы и фактические виртуальные машины. Для развертывания полной архитектуры с одной операцией создайте скрипт или шаблон для вызова каждого из шаблонов стандартных блоков с соответствующими файлами параметров.

В процессе разработки шаблонов стандартных блоков команда разработала несколько основных концепций для расширения функциональности шаблонов Azure Resource Manager. В этой серии статей описаны некоторые из этих концепций, которые можно использовать в собственных шаблонах.

> [!NOTE]
> В этих статьях предполагается, что вы уже хорошо знакомы с шаблонами Azure Resource Manager.