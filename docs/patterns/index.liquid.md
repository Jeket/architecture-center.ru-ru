---
title: Конструктивные шаблоны облачных решений
titleSuffix: Azure Architecture Center
description: Конструктивные шаблоны облачных решений для Microsoft Azure
keywords: Таблицы Azure
author: dragon119
ms.date: 12/10/2018
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
---

# <a name="cloud-design-patterns"></a>Конструктивные шаблоны облачных решений

[!INCLUDE [header](../../_includes/header.md)]

Эти конструктивные шаблоны полезны для создания надежных, масштабируемых, безопасных приложений в облаке.

В каждом из шаблонов описывается проблема, которую он устраняет, приведены рекомендации для применения шаблона и пример, основанный на Microsoft Azure. Большинство шаблонов содержат примеры или фрагменты кода, в которых показано, как реализовать шаблон в Azure. Однако многие шаблоны относятся к любой распределенной системе, размещенной в Azure или на других облачных платформах.

## <a name="problem-areas-in-the-cloud"></a>Проблемные области в облаке

<!-- markdownlint-disable MD033 -->

<ul id="categories" class="panel">
{%- for category in categories %}
    <li>
    {% include 'pattern-category-card' %}
    </li>
{%- endfor %}
</ul>

<!-- markdownlint-enable MD033 -->

## <a name="catalog-of-patterns"></a>Каталог шаблонов

| Модель | Сводка |
|---------|---------|
|         |         |

{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}
