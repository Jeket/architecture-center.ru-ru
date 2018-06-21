---
title: Условное развертывание ресурсов в шаблоне Azure Resource Manager
description: Расширение возможностей в шаблоне Azure Resource Manager за счет условного развертывания ресурсов в зависимости от значения какого-то параметра
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: e911e7dc41b4f71ebfaf13a00f8cdbb5b4e2578b
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/14/2017
ms.locfileid: "24538398"
---
# <a name="conditionally-deploy-a-resource-in-an-azure-resource-manager-template"></a>Условное развертывание ресурсов в шаблоне Azure Resource Manager

В некоторых ситуациях необходимо создать такой шаблон, который будет развертывать ресурс только при определенных условиях, например при наличии или отсутствии значения конкретного параметра. Например, вы можете создать шаблон для развертывания виртуальной сети, который содержит параметры, описывающие другие виртуальные сети для пиринга. Если параметры для пиринга не указаны, Resource Manager не должен развертывать ресурсы пиринга.

Чтобы реализовать такой вариант, используйте в ресурсе [элемент `condition`][azure-resource-manager-condition], чтобы проверить длину массива параметров. Если эта длина равна нулю, возвращайте значение `false`, чтобы заблокировать развертывание, а при любой длине больше нуля — значение `true`, чтобы разрешить развертывание.

## <a name="example-template"></a>Пример шаблона

Рассмотрим этот вариант на конкретном примере шаблона. В этом шаблоне [ элемент `condition`][azure-resource-manager-condition] управляет развертыванием ресурса `Microsoft.Network/virtualNetworks/virtualNetworkPeerings`. Этот ресурс создает пиринг между двумя виртуальными сетями Azure в одном регионе.

Рассмотрим подробно каждый раздел шаблона.

Элемент `parameters` определяет один параметр с именем `virtualNetworkPeerings`. 

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "virtualNetworkPeerings": {
      "type": "array",
      "defaultValue": []
    }
  },
```
Параметр `virtualNetworkPeerings` является массивом (`array`) со следующей схемой данных.

```json
"virtualNetworkPeerings": [
    {
        "remoteVirtualNetwork": {
            "name": "my-other-virtual-network"
        },
        "allowForwardedTraffic": true,
        "allowGatewayTransit": true,
        "useRemoteGateways": false
    }
]
```

Свойства этого параметра определяют [настройки пиринга виртуальных сетей][vnet-peering-resource-schema]. Чтобы предоставить значения для этих свойств, нужно определить ресурс `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` в разделе `resources`:

```json
"resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2017-05-10",
      "name": "[concat('vnp-', copyIndex())]",
      "condition": "[greater(length(parameters('virtualNetworkPeerings')), 0)]",
      "dependsOn": [
        "virtualNetworks"
      ],
      "copy": {
          "name": "iterator",
          "count": "[length(variables('peerings'))]",
          "mode": "serial"
      },
      "properties": {
        "mode": "Incremental",
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {
          },
          "variables": {
          },
          "resources": [
            {
              "type": "Microsoft.Network/virtualNetworks/virtualNetworkPeerings",
              "apiVersion": "2016-06-01",
              "location": "[resourceGroup().location]",
              "name": "[variables('peerings')[copyIndex()].name]",
              "properties": "[variables('peerings')[copyIndex()].properties]"
            }
          ],
          "outputs": {
          }
        }
      }
    }
]
```
В этой части нашего примера есть несколько элементов. Во-первых, фактически развертываемый ресурс определен во встроенном шаблоне типа `Microsoft.Resources/deployments`, в котором содержится дополнительный шаблон для развертывания `Microsoft.Network/virtualNetworks/virtualNetworkPeerings`.

Чтобы имя (`name`) встроенного шаблона было уникальным, мы вычисляем его значение путем объединения текущего значения `copyIndex()` с префиксом `vnp-`. 

Элемент `condition` указывает, что этот ресурс обрабатывается в том случае, когда функция `greater()` возвращает значение `true`. Здесь мы проверяем, что параметр `virtualNetworkPeerings` для массива имеет значение больше нуля (`greater()`). Если это верно, возвращается значение `true` и наше условие (`condition`) считается выполненным. В противном случае возвращается `false`.

Далее описан цикл `copy`. Это цикл типа `serial`, то есть он выполняется последовательно, и каждый следующий ресурс ожидает, пока завершится развертывание предыдущего ресурса. Свойство `count` указывает, сколько раз будет выполнен этот цикл. Обычно здесь используется значение длины массива `virtualNetworkPeerings`, поскольку именно этот массив содержит объекты параметров для развертывания ресурсов. Но если мы просто подставим это значение, то проверка значения завершится ошибкой в случае пустого массива, ведь Resource Manager зафиксирует попытку обратиться к несуществующим свойствам. Мы можем обойти это ограничение. Для этого нам потребуются следующие переменные.

```json
  "variables": {
    "workaround": {
       "true": "[parameters('virtualNetworkPeerings')]",
       "false": [{
           "name": "workaround",
           "properties": {}
       }]
     },
     "peerings": "[variables('workaround')[string(greater(length(parameters('virtualNetworkPeerings')), 0))]]"
  },
```

Переменная `workaround` включает в себя два свойства с именами `true` и `false`. Свойство `true` указывает на массив параметров `virtualNetworkPeerings`. Свойство `false` содержит пустой объект со всеми именованными свойствами, которые ожидает получить Resource Manager. Обратите внимание, что `false` фактически является массивом, как и параметр `virtualNetworkPeerings` что позволяет успешно пройти проверку. 

Переменная `peerings` использует эту новую переменную `workaround`, чтобы проверить, что длина массива параметров `virtualNetworkPeerings` больше нуля. Если это так, `string` принимает значение `true`, а переменная `workaround` указывает на массив параметров `virtualNetworkPeerings`. В противном случае возвращается `false`, и переменная `workaround` содержит пустой объект в первом элементе массива.

Итак, мы решили проблему с проверкой значений и можем просто указать развертывание ресурса `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` во вложенном шаблоне, передавая ему параметры `name` и `properties` из массива параметров `virtualNetworkPeerings`. Этот механизм реализован в элементе `template`, который вложен в элемент `properties` нашего ресурса.

## <a name="next-steps"></a>Дальнейшие действия

* Эта же методика реализована в [проекте стандартных блоков шаблона](https://github.com/mspnp/template-building-blocks) и [эталонных архитектурах Azure](/azure/architecture/reference-architectures/). Вы можете создать на их основе собственную архитектуру или развернуть готовые примеры архитектуры.

<!-- links -->
[azure-resource-manager-condition]: /azure/azure-resource-manager/resource-group-authoring-templates#resources
[azure-resource-manager-variable]: /azure/azure-resource-manager/resource-group-authoring-templates#variables
[vnet-peering-resource-schema]: /azure/templates/microsoft.network/virtualnetworks/virtualnetworkpeerings