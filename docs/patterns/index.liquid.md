---
title: Конструктивные шаблоны облачных решений
description: Конструктивные шаблоны облачных решений для Microsoft Azure
keywords: Таблицы Azure
ms.openlocfilehash: 4747c896fc6fc5866be782d76c5290d6b49ad451
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2018
---
# <a name="cloud-design-patterns"></a>Конструктивные шаблоны облачных решений

[!INCLUDE [header](../../_includes/header.md)]

Эти конструктивные шаблоны полезны для создания надежных, масштабируемых, безопасных приложений в облаке.

В каждом из шаблонов описывается проблема, которую он устраняет, приведены рекомендации для применения шаблона и пример, основанный на Microsoft Azure. Большинство шаблонов содержат примеры или фрагменты кода, в которых показано, как реализовать шаблон в Azure. Однако многие шаблоны относятся к любой распределенной системе, размещенной в Azure или на других облачных платформах.

## <a name="problem-areas-in-the-cloud"></a>Проблемные области в облаке

<ul id="categories" class="panel">
{%- for category in categories %}
    <li>
    {% include 'pattern-category-card' %}
    </li>
{%- endfor %}
</ul>

## <a name="catalog-of-patterns"></a>Каталог шаблонов

| Модель | Сводка |
|---------|---------|
|         |         |

{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}
