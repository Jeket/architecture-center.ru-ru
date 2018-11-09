---
title: Использование объекта в качестве параметра в шаблоне Azure Resource Manager
description: Описывается, как расширить функциональные возможности шаблонов Azure Resource Manager для использования объектов как параметров.
author: petertay
ms.date: 10/30/2018
ms.openlocfilehash: c1955823b3474efa0abea1d9634add5f13d02eda
ms.sourcegitcommit: e9eb2b895037da0633ef3ccebdea2fcce047620f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/30/2018
ms.locfileid: "50251895"
---
# <a name="use-an-object-as-a-parameter-in-an-azure-resource-manager-template"></a>Использование объекта в качестве параметра в шаблоне Azure Resource Manager

Когда вы [создаете шаблоны Azure Resource Manager][azure-resource-manager-create-template], значения свойств ресурсов можно указать непосредственно в шаблоне или определить в виде параметров, значения которых будут предоставлены во время развертывания. В небольших развертываниях можно спокойно применять параметры для всех значений свойств, но с ограничением в 255 параметров на одно развертывание. Когда развертывания станут крупнее и сложнее, вы можете исчерпать доступный запас параметров.

Чтобы решить эту проблему, в качестве параметра можно использовать не значения, а объекты. Для этого просто задайте параметр в шаблоне, а во время развертывания укажите для этого параметра не одно значение, а объект в формате JSON. Тогда вы сможете ссылаться в шаблоне на вложенные свойства этого параметра с помощью [функции `parameter()`][azure-resource-manager-functions] и оператора-точки.

Давайте рассмотрим пример шаблона, который развертывает ресурс виртуальной сети. Прежде всего нужно определить в шаблоне параметр `VNetSettings` и присвоить `type` значение `object`.

```json
...
"parameters": {
    "VNetSettings":{"type":"object"}
},
```
Теперь мы предоставим значения для объекта `VNetSettings`.

> [!NOTE]
> Как предоставить значения параметрам во время развертывания, см. в разделе **Параметры** в статье [Описание структуры и синтаксиса шаблонов Azure Resource Manager][azure-resource-manager-authoring-templates]. 

```json
"parameters":{
    "VNetSettings":{
        "value":{
            "name":"VNet1",
            "addressPrefixes": [
                {
                    "name": "firstPrefix",
                    "addressPrefix": "10.0.0.0/22"
                }
            ],
            "subnets":[
                {
                    "name": "firstSubnet",
                    "addressPrefix": "10.0.0.0/24"
                },
                {
                    "name":"secondSubnet",
                    "addressPrefix":"10.0.1.0/24"
                }
            ]
        }
    }
}
```

Как видите, здесь один параметр фактически задает три вложенных свойства: `name`, `addressPrefixes` и `subnets`. Каждое из вложенных свойств содержит значение или другие вложенные свойства. Таким образом в один параметр можно поместить все значения, необходимые для развертывания виртуальной сети.

Теперь мы перейдем к остальной части шаблона и посмотрим, как используется этот объект `VNetSettings`.

```json
...
"resources": [
    {
        "apiVersion": "2015-06-15",
        "type": "Microsoft.Network/virtualNetworks",
        "name": "[parameters('VNetSettings').name]",
        "location":"[resourceGroup().location]",
        "properties": {
          "addressSpace":{
              "addressPrefixes": [
                "[parameters('VNetSettings').addressPrefixes[0].addressPrefix]"
              ]
          },
          "subnets":[
              {
                  "name":"[parameters('VNetSettings').subnets[0].name]",
                  "properties": {
                      "addressPrefix": "[parameters('VNetSettings').subnets[0].addressPrefix]"
                  }
              },
              {
                  "name":"[parameters('VNetSettings').subnets[1].name]",
                  "properties": {
                      "addressPrefix": "[parameters('VNetSettings').subnets[1].addressPrefix]"
                  }
              }
          ]
        }
    }
  ]
```
Включенные в объект `VNetSettings` значения применяются к свойствам, к которым ресурс виртуальной сети обращается через функцию `parameters()` с использованием индексатора массива `[]` и (или) оператора-точки. Такой подход удобен, когда вам нужно просто применить к ресурсу статические значения объекта параметров. Но если вы хотите динамически назначать значения массиву свойств во время развертывания, используйте [цикл копирования][azure-resource-manager-create-multiple-instances]. Для этого предоставьте массив JSON со значениями свойств ресурсов. Цикл копирования динамически применяет значения к свойствам ресурса. 

При использовании динамического подхода следует учитывать один важный аспект. Потенциальную проблему мы продемонстрируем на примере обычного массива значений свойств. В этом примере значения, которые нужно присвоить свойствам, хранятся в переменной. Обратите внимание, что мы используем два массива с именами `firstProperty` и `secondProperty`. 

```json
"variables": {
    "firstProperty": [
        {
            "name": "A",
            "type": "typeA"
        },
        {
            "name": "B",
            "type": "typeB"
        },
        {
            "name": "C",
            "type": "typeC"
        }
    ],
    "secondProperty": [
        "one","two", "three"
    ]
}
```

Теперь посмотрите, как мы обращаемся к этим свойствам переменной в цикле копирования.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    ...
    "copy": {
        "name": "copyLoop1",
        "count": "[length(variables('firstProperty'))]"
    },
    ...
    "properties": {
        "name": { "value": "[variables('firstProperty')[copyIndex()].name]" },
        "type": { "value": "[variables('firstProperty')[copyIndex()].type]" },
        "number": { "value": "[variables('secondProperty')[copyIndex()]]" }
    }
}
```

Функция `copyIndex()` возвращает текущую итерацию цикла копирования, и мы применяем это значение в качестве индекса, обращаясь одновременно к двум массивам.

Это допустимо, если массивы имеют одинаковую длину. Но если вы ошибетесь при определении массивов, и они будут иметь разную длину, шаблон не пройдет проверку во время развертывания. Эту проблему можно предотвратить, включив все свойства в один объект, ведь так вам будет проще заметить отсутствие значения. Вот еще один пример объекта параметров, в котором каждый элемент массива `propertyObject` представляет объединение массивов `firstProperty` и `secondProperty` из предыдущего примера.

```json
"variables": {
    "propertyObject": [
        {
            "name": "A",
            "type": "typeA",
            "number": "one"
        },
        {
            "name": "B",
            "type": "typeB",
            "number": "two"
        },
        {
            "name": "C",
            "type": "typeC"
        }
    ]
}
```

Обратите внимание на третий элемент в массиве. Здесь отсутствует свойство `number`, но теперь, когда вы создали параметр упомянутым образом, вам намного проще это заметить.

## <a name="using-a-property-object-in-a-copy-loop"></a>Использование объекта свойств в цикле копирования

Такой подход становится еще полезнее в сочетании с [циклом последовательного копирования][azure-resource-manager-create-multiple], особенно для развертывания дочерних ресурсов. 

Чтобы продемонстрировать это, рассмотрим шаблон для развертывания [группы безопасности сети (NSG)][nsg] с двумя правилами безопасности. 

Изучение шаблона мы начнем со списка параметров. В этом шаблоне мы определили один параметр с именем `networkSecurityGroupsSettings`, который включает массив с именем `securityRules`. Этот массив содержит два объекта JSON, которые определяют несколько параметров для правила безопасности.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters":{ 
      "networkSecurityGroupsSettings": {
      "value": {
          "securityRules": [
            {
              "name": "RDPAllow",
              "description": "allow RDP connections",
              "direction": "Inbound",
              "priority": 100,
              "sourceAddressPrefix": "*",
              "destinationAddressPrefix": "10.0.0.0/24",
              "sourcePortRange": "*",
              "destinationPortRange": "3389",
              "access": "Allow",
              "protocol": "Tcp"
            },
            {
              "name": "HTTPAllow",
              "description": "allow HTTP connections",
              "direction": "Inbound",
              "priority": 200,
              "sourceAddressPrefix": "*",
              "destinationAddressPrefix": "10.0.1.0/24",
              "sourcePortRange": "*",
              "destinationPortRange": "80",
              "access": "Allow",
              "protocol": "Tcp"
            }
          ]
        }
      }
    }
  }
```

Теперь перейдем к самому шаблону. Первый ресурс с именем `NSG1` развертывает группу безопасности сети. Второй ресурс с именем `loop-0` выполняет две функции: во-первых, он определяет зависимость (`dependsOn`) от группы безопасности сети, благодаря чему развертывание ресурса не начнется, пока `NSG1` не завершит развертывание; во-вторых, он является первой итерацией последовательного цикла. Третий ресурс здесь — это вложенный шаблон, который развертывает правила безопасности, используя объект для получения значений своих параметров, как показано в предыдущем примере.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
      "networkSecurityGroupsSettings": {"type":"object"}
  },
  "variables": {},
  "resources": [
    {
      "apiVersion": "2015-06-15",
      "type": "Microsoft.Network/networkSecurityGroups",
      "name": "NSG1",
      "location":"[resourceGroup().location]",
      "properties": {
          "securityRules":[]
      }
    },
    {
        "apiVersion": "2015-01-01",
        "type": "Microsoft.Resources/deployments",
        "name": "loop-0",
        "dependsOn": [
            "NSG1"
        ],
        "properties": {
            "mode":"Incremental",
            "parameters":{},
            "template": {
                "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                "contentVersion": "1.0.0.0",
                "parameters": {},
                "variables": {},
                "resources": [],
                "outputs": {}
            }
        }       
    },
    {
        "apiVersion": "2015-01-01",
        "type": "Microsoft.Resources/deployments",
        "name": "[concat('loop-', copyIndex(1))]",
        "dependsOn": [
          "[concat('loop-', copyIndex())]"
        ],
        "copy": {
          "name": "iterator",
          "count": "[length(parameters('networkSecurityGroupsSettings').securityRules)]"
        },
        "properties": {
          "mode": "Incremental",
          "template": {
            "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
            "contentVersion": "1.0.0.0",
           "parameters": {},
            "variables": {},
            "resources": [
                {
                    "name": "[concat('NSG1/' , parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].name)]",
                    "type": "Microsoft.Network/networkSecurityGroups/securityRules",
                    "apiVersion": "2016-09-01",
                    "location":"[resourceGroup().location]",
                    "properties":{
                        "description": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].description]",
                        "priority":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].priority]",
                        "protocol":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].protocol]",
                        "sourcePortRange": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].sourcePortRange]",
                        "destinationPortRange": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].destinationPortRange]",
                        "sourceAddressPrefix": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].sourceAddressPrefix]",
                        "destinationAddressPrefix": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].destinationAddressPrefix]",
                        "access":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].access]",
                        "direction":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].direction]"
                        }
                  }
            ],
            "outputs": {}
          }
        }
    }
  ],          
  "outputs": {}
}
```

Давайте подробнее рассмотрим, как значения свойств указываются в дочернем ресурсе `securityRules`. Мы ссылаемся на свойства с помощью функции `parameter()`, а затем применяем оператор-точку для ссылок на массив `securityRules`, используя в качестве индекса текущее значение итерации. И наконец, еще один оператор-точку мы используем для ссылки на имя объекта. 

## <a name="try-the-template"></a>Пробное использование шаблона

Пример шаблона доступен на [сайте GitHub][github]. Чтобы развернуть этот шаблон с помощью Azure CLI, выполните следующие команды [Azure CLI][cli]:

```bash
git clone https://github.com/mspnp/template-examples.git
cd template-examples/example3-object-param
az group create --location <location> --name <resource-group-name>
az group deployment create -g <resource-group-name> \
    --template-uri https://raw.githubusercontent.com/mspnp/template-examples/master/example3-object-param/deploy.json \
    --parameters deploy.parameters.json
```

## <a name="next-steps"></a>Дополнительная информация

- Узнайте, как создать шаблон, который перебирает массив объектов и преобразует его в схему JSON. См. сведения о [реализации преобразователя и сборщика свойств в шаблоне Azure Resource Manager](./collector.md).


<!-- links -->
[azure-resource-manager-authoring-templates]: /azure/azure-resource-manager/resource-group-authoring-templates
[azure-resource-manager-create-template]: /azure/azure-resource-manager/resource-manager-create-first-template
[azure-resource-manager-create-multiple-instances]: /azure/azure-resource-manager/resource-group-create-multiple
[azure-resource-manager-functions]: /azure/azure-resource-manager/resource-group-template-functions-resource
[nsg]: /azure/virtual-network/virtual-networks-nsg
[cli]: /cli/azure/?view=azure-cli-latest
[github]: https://github.com/mspnp/template-examples
