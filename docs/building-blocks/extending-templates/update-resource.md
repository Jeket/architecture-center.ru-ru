---
title: Обновление ресурсов в шаблоне Azure Resource Manager
description: Описывается, как расширить функциональные возможности шаблонов Azure Resource Manager для обновления ресурса.
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: f235f0b4d54d65ccc2fa67876916e922d75f6d07
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/28/2018
ms.locfileid: "47429047"
---
# <a name="update-a-resource-in-an-azure-resource-manager-template"></a>Обновление ресурсов в шаблоне Azure Resource Manager

Существует несколько сценариев, в которых во время развертывания необходимо обновить ресурс. Например, у вас нет возможности указать все свойства ресурса, пока не созданы другие (зависимые) ресурсы. Например, при создании внутреннего пула для подсистемы балансировки нагрузки на виртуальных машинах можно обновить сетевые интерфейсы (NIC), чтобы включить их во внутренний пул. Resource Manager поддерживает обновление ресурсов во время развертывания. Но для этого нужно правильно разработать шаблон, чтобы избежать ошибок и гарантировать, что развертывание будет обработано как обновление.

Сначала необходимо добавить ссылку на ресурс в шаблоне, чтобы создать его, а затем еще раз добавить ссылку на этот ресурс, используя то же самое имя, чтобы его обновить. Необходимо учитывать, что если в шаблоне два ресурса имеют одинаковые имена, то Resource Manager порождает исключение. Во избежание этой ошибки укажите обновленный ресурс во втором шаблоне, который связан с первым или включен как вложенный шаблон с помощью типа ресурса `Microsoft.Resources/deployments`.

Затем во вложенном шаблоне необходимо указать имя существующего свойства для изменения или новое имя для добавления свойства. Также нужно указать исходные свойства и их исходные значения. Если вы не укажете исходные свойства и значения, в Resource Manager определяется, что будет создан новый ресурс, и удаляется исходный ресурс.

## <a name="example-template"></a>Пример шаблона

Рассмотрим этот вариант на конкретном примере шаблона. При помощи этого шаблона развертывается виртуальная сеть с именем `firstVNet`, которая содержит одну подсеть с именем `firstSubnet`. Затем развертывается виртуальный сетевой интерфейс (NIC) с именем `nic1`, который связывается с подсетью. В ресурс развертывания с именем `updateVNet` помещается вложенный шаблон, при помощи которого в ресурс `firstVNet` добавляется вторая подсеть с именем `secondSubnet`. 

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "resources": [
      {
      "apiVersion": "2016-03-30",
      "name": "firstVNet",
      "location":"[resourceGroup().location]",
      "type": "Microsoft.Network/virtualNetworks",
      "properties": {
          "addressSpace":{"addressPrefixes": [
              "10.0.0.0/22"
          ]},
          "subnets":[              
              {
                  "name":"firstSubnet",
                  "properties":{
                    "addressPrefix":"10.0.0.0/24"
                  }
              }
            ]
      }
    },
    {
        "apiVersion": "2015-06-15",
        "type":"Microsoft.Network/networkInterfaces",
        "name":"nic1",
        "location":"[resourceGroup().location]",
        "dependsOn": [
            "firstVNet"
        ],
        "properties": {
            "ipConfigurations":[
                {
                    "name":"ipconfig1",
                    "properties": {
                        "privateIPAllocationMethod":"Dynamic",
                        "subnet": {
                            "id": "[concat(resourceId('Microsoft.Network/virtualNetworks','firstVNet'),'/subnets/firstSubnet')]"
                        }
                    }
                }
            ]
        }
    },
    {
      "apiVersion": "2015-01-01",
      "type": "Microsoft.Resources/deployments",
      "name": "updateVNet",
      "dependsOn": [
          "nic1"
      ],
      "properties": {
        "mode": "Incremental",
        "parameters": {},
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {},
          "variables": {},
          "resources": [
              {
                  "apiVersion": "2016-03-30",
                  "name": "firstVNet",
                  "location":"[resourceGroup().location]",
                  "type": "Microsoft.Network/virtualNetworks",
                  "properties": {
                      "addressSpace": "[reference('firstVNet').addressSpace]",
                      "subnets":[
                          {
                              "name":"[reference('firstVNet').subnets[0].name]",
                              "properties":{
                                  "addressPrefix":"[reference('firstVNet').subnets[0].properties.addressPrefix]"
                                  }
                          },
                          {
                              "name":"secondSubnet",
                              "properties":{
                                  "addressPrefix":"10.0.1.0/24"
                                  }
                          }
                     ]
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

Сначала давайте рассмотрим объект ресурса для `firstVNet`. Обратите внимание, что во вложенном шаблоне повторно перечислены параметры `firstVNet`. Это связано с тем, что Resource Manager не допускает наличия одинаковых имен развертывания в одном шаблоне, а вложенный шаблон считается уже другим шаблоном. Повторно перечисляя значения, указанные для ресурса `firstSubnet`, мы указываем Resource Manager, что нужно обновить существующий ресурс, а не удалять его и развертывать новый. И, наконец, в этом развертывании устанавливаются новые параметры для `secondSubnet`.

## <a name="try-the-template"></a>Пробное использование шаблона

Если вы хотите поэкспериментировать с этим шаблоном, выполните следующее.

1.  Перейдите на портал Azure, щелкните значок **+**, выполните поиск по типу ресурса **Развертывание шаблона** и выберите этот ресурс.
2.  На странице **Развертывание шаблона** нажмите кнопку **Создать**. Откроется колонка **Настраиваемое развертывание**.
3.  Щелкните значок **Изменить**.
4.  Удалите пустой шаблон.
5.  Скопируйте этот пример шаблона и вставьте его в область справа.
6.  Нажмите кнопку **Сохранить**.
7.  Вы вернетесь в область **Настраиваемое развертывание**, но на этот раз в ней будет несколько раскрывающихся списков. Выберите свою подписку, создайте новую или используйте существующую группу ресурсов и выберите расположение. Ознакомьтесь с условиями и нажмите кнопку **Принимаю**.
8.  Нажмите кнопку **Приобрести**.

По завершении развертывания откройте группу ресурсов, которую вы указали на портале. Отобразится виртуальная сеть с именем `firstVNet` и сетевой адаптер с именем `nic1`. Щелкните `firstVNet`, а затем — `subnets`. Отобразится подсеть `firstSubnet`, созданная изначально, а также подсеть `secondSubnet`, добавленная в ресурсе `updateVNet`. 

![Исходная подсеть и обновленная подсеть](../_images/firstVNet-subnets.png)

Затем вернитесь в группу ресурсов и щелкните `nic1`, а затем — `IP configurations`. В разделе `IP configurations` для `subnet` задано значение `firstSubnet (10.0.0.0/24)`. 

![Конфигурации IP виртуального сетевого интерфейса nic1](../_images/nic1-ipconfigurations.png)

Исходная виртуальная сеть `firstVNet` была обновлена, а не создана повторно. Если бы `firstVNet` была создана повторно, то виртуальный сетевой интерфейс `nic1` не был бы связан с `firstVNet`.

## <a name="next-steps"></a>Дополнительная информация

* Эта же методика реализована в [проекте стандартных блоков шаблона](https://github.com/mspnp/template-building-blocks) и [эталонных архитектурах Azure](/azure/architecture/reference-architectures/). Вы можете создать на их основе собственную архитектуру или развернуть готовые примеры архитектуры.
