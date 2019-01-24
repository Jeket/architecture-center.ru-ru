---
title: Обновление ресурсов в шаблоне Azure Resource Manager
description: В статье описано, как расширить возможности шаблонов Azure Resource Manager для обновления ресурса.
author: petertay
ms.date: 10/31/2018
ms.topic: article
ms.service: architecture-center
ms.subservice: reference-architecture
ms.openlocfilehash: de76e69e94917bbbe94c0f87fda2cdbe415181dc
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54487157"
---
# <a name="update-a-resource-in-an-azure-resource-manager-template"></a><span data-ttu-id="5831c-103">Обновление ресурсов в шаблоне Azure Resource Manager</span><span class="sxs-lookup"><span data-stu-id="5831c-103">Update a resource in an Azure Resource Manager template</span></span>

<span data-ttu-id="5831c-104">Существует несколько сценариев, в которых во время развертывания необходимо обновить ресурс.</span><span class="sxs-lookup"><span data-stu-id="5831c-104">There are some scenarios in which you need to update a resource during a deployment.</span></span> <span data-ttu-id="5831c-105">Например, у вас нет возможности указать все свойства ресурса, пока не созданы другие (зависимые) ресурсы.</span><span class="sxs-lookup"><span data-stu-id="5831c-105">You might encounter this scenario when you cannot specify all the properties for a resource until other, dependent resources are created.</span></span> <span data-ttu-id="5831c-106">Например, при создании внутреннего пула для подсистемы балансировки нагрузки на виртуальных машинах можно обновить сетевые интерфейсы (NIC), чтобы включить их во внутренний пул.</span><span class="sxs-lookup"><span data-stu-id="5831c-106">For example, if you create a backend pool for a load balancer, you might update the network interfaces (NICs) on your virtual machines (VMs) to include them in the backend pool.</span></span> <span data-ttu-id="5831c-107">Resource Manager поддерживает обновление ресурсов во время развертывания. Но для этого нужно правильно разработать шаблон, чтобы избежать ошибок и гарантировать, что развертывание будет обработано как обновление.</span><span class="sxs-lookup"><span data-stu-id="5831c-107">And while Resource Manager supports updating resources during deployment, you must design your template correctly to avoid errors and to ensure the deployment is handled as an update.</span></span>

<span data-ttu-id="5831c-108">Сначала необходимо добавить ссылку на ресурс в шаблоне, чтобы создать его, а затем еще раз добавить ссылку на этот ресурс, используя то же самое имя, чтобы его обновить.</span><span class="sxs-lookup"><span data-stu-id="5831c-108">First, you must reference the resource once in the template to create it and then reference the resource by the same name to update it later.</span></span> <span data-ttu-id="5831c-109">Необходимо учитывать, что если в шаблоне два ресурса имеют одинаковые имена, то Resource Manager порождает исключение.</span><span class="sxs-lookup"><span data-stu-id="5831c-109">However, if two resources have the same name in a template, Resource Manager throws an exception.</span></span> <span data-ttu-id="5831c-110">Во избежание этой ошибки укажите обновленный ресурс во втором шаблоне, который связан с первым или включен как вложенный шаблон с помощью типа ресурса `Microsoft.Resources/deployments`.</span><span class="sxs-lookup"><span data-stu-id="5831c-110">To avoid this error, specify the updated resource in a second template that's either linked or included as a subtemplate using the `Microsoft.Resources/deployments` resource type.</span></span>

<span data-ttu-id="5831c-111">Затем во вложенном шаблоне необходимо указать имя существующего свойства для изменения или новое имя для добавления свойства.</span><span class="sxs-lookup"><span data-stu-id="5831c-111">Second, you must either specify the name of the existing property to change or a new name for a property to add in the nested template.</span></span> <span data-ttu-id="5831c-112">Также нужно указать исходные свойства и их исходные значения.</span><span class="sxs-lookup"><span data-stu-id="5831c-112">You must also specify the original properties and their original values.</span></span> <span data-ttu-id="5831c-113">Если вы не укажете исходные свойства и значения, в Resource Manager определяется, что будет создан новый ресурс, и удаляется исходный ресурс.</span><span class="sxs-lookup"><span data-stu-id="5831c-113">If you fail to provide the original properties and values, Resource Manager assumes you want to create a new resource and deletes the original resource.</span></span>

## <a name="example-template"></a><span data-ttu-id="5831c-114">Пример шаблона</span><span class="sxs-lookup"><span data-stu-id="5831c-114">Example template</span></span>

<span data-ttu-id="5831c-115">Рассмотрим этот вариант на конкретном примере шаблона.</span><span class="sxs-lookup"><span data-stu-id="5831c-115">Let's look at an example template that demonstrates this.</span></span> <span data-ttu-id="5831c-116">Этот шаблон развертывает виртуальную сеть с именем `firstVNet`, которая содержит одну подсеть с именем `firstSubnet`.</span><span class="sxs-lookup"><span data-stu-id="5831c-116">Our template deploys a virtual network named `firstVNet` that has one subnet named `firstSubnet`.</span></span> <span data-ttu-id="5831c-117">Затем развертывается виртуальный сетевой интерфейс (NIC) с именем `nic1`, который связывается с подсетью.</span><span class="sxs-lookup"><span data-stu-id="5831c-117">It then deploys a virtual network interface (NIC) named `nic1` and associates it with our subnet.</span></span> <span data-ttu-id="5831c-118">В ресурс развертывания с именем `updateVNet` помещается вложенный шаблон, при помощи которого в ресурс `firstVNet` добавляется вторая подсеть с именем `secondSubnet`.</span><span class="sxs-lookup"><span data-stu-id="5831c-118">Then, a deployment resource named `updateVNet` includes a nested template that updates our `firstVNet` resource by adding a second subnet named `secondSubnet`.</span></span>

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

<span data-ttu-id="5831c-119">Сначала давайте рассмотрим объект ресурса для `firstVNet`.</span><span class="sxs-lookup"><span data-stu-id="5831c-119">Let's take a look at the resource object for our `firstVNet` resource first.</span></span> <span data-ttu-id="5831c-120">Обратите внимание, что во вложенном шаблоне повторно перечислены параметры `firstVNet`. Это связано с тем, что Resource Manager не допускает наличия одинаковых имен развертывания в одном шаблоне, а вложенный шаблон считается уже другим шаблоном.</span><span class="sxs-lookup"><span data-stu-id="5831c-120">Notice that we respecify the settings for our `firstVNet` in a nested template&mdash;this is because Resource Manager doesn't allow the same deployment name within the same template and nested templates are considered to be a different template.</span></span> <span data-ttu-id="5831c-121">Повторно перечисляя значения, указанные для ресурса `firstSubnet`, мы указываем Resource Manager, что нужно обновить существующий ресурс, а не удалять его и развертывать новый.</span><span class="sxs-lookup"><span data-stu-id="5831c-121">By respecifying our values for our `firstSubnet` resource, we are telling Resource Manager to update the existing resource instead of deleting it and redeploying it.</span></span> <span data-ttu-id="5831c-122">И, наконец, в этом развертывании устанавливаются новые параметры для `secondSubnet`.</span><span class="sxs-lookup"><span data-stu-id="5831c-122">Finally, our new settings for `secondSubnet` are picked up during this update.</span></span>

## <a name="try-the-template"></a><span data-ttu-id="5831c-123">Пробное использование шаблона</span><span class="sxs-lookup"><span data-stu-id="5831c-123">Try the template</span></span>

<span data-ttu-id="5831c-124">Пример шаблона доступен на [сайте GitHub][github].</span><span class="sxs-lookup"><span data-stu-id="5831c-124">An example template is available on [GitHub][github].</span></span> <span data-ttu-id="5831c-125">Чтобы развернуть этот шаблон, выполните следующие команды [Azure CLI][cli]:</span><span class="sxs-lookup"><span data-stu-id="5831c-125">To deploy the template, run the following [Azure CLI][cli] commands:</span></span>

```bash
az group create --location <location> --name <resource-group-name>
az group deployment create -g <resource-group-name> \
    --template-uri https://raw.githubusercontent.com/mspnp/template-examples/master/example1-update/deploy.json
```

<span data-ttu-id="5831c-126">По завершении развертывания откройте группу ресурсов, которую вы указали на портале.</span><span class="sxs-lookup"><span data-stu-id="5831c-126">Once deployment has finished, open the resource group you specified in the portal.</span></span> <span data-ttu-id="5831c-127">Отобразится виртуальная сеть с именем `firstVNet` и сетевой адаптер с именем `nic1`.</span><span class="sxs-lookup"><span data-stu-id="5831c-127">You see a virtual network named `firstVNet` and a NIC named `nic1`.</span></span> <span data-ttu-id="5831c-128">Щелкните `firstVNet`, а затем — `subnets`.</span><span class="sxs-lookup"><span data-stu-id="5831c-128">Click `firstVNet`, then click `subnets`.</span></span> <span data-ttu-id="5831c-129">Отобразится подсеть `firstSubnet`, созданная изначально, а также подсеть `secondSubnet`, добавленная в ресурсе `updateVNet`.</span><span class="sxs-lookup"><span data-stu-id="5831c-129">You see the `firstSubnet` that was originally created, and you see the `secondSubnet` that was added in the `updateVNet` resource.</span></span>

![Исходная подсеть и обновленная подсеть](../_images/firstVNet-subnets.png)

<span data-ttu-id="5831c-131">Затем вернитесь в группу ресурсов и щелкните `nic1`, а затем — `IP configurations`.</span><span class="sxs-lookup"><span data-stu-id="5831c-131">Then, go back to the resource group and click `nic1` then click `IP configurations`.</span></span> <span data-ttu-id="5831c-132">В разделе `IP configurations` для `subnet` задано значение `firstSubnet (10.0.0.0/24)`.</span><span class="sxs-lookup"><span data-stu-id="5831c-132">In the `IP configurations` section, the `subnet` is set to `firstSubnet (10.0.0.0/24)`.</span></span>

![Конфигурации IP виртуального сетевого интерфейса nic1](../_images/nic1-ipconfigurations.png)

<span data-ttu-id="5831c-134">Исходная виртуальная сеть `firstVNet` была обновлена, а не создана повторно.</span><span class="sxs-lookup"><span data-stu-id="5831c-134">The original `firstVNet` has been updated instead of recreated.</span></span> <span data-ttu-id="5831c-135">Если бы `firstVNet` была создана повторно, то виртуальный сетевой интерфейс `nic1` не был бы связан с `firstVNet`.</span><span class="sxs-lookup"><span data-stu-id="5831c-135">If `firstVNet` had been recreated, `nic1` would not be associated with `firstVNet`.</span></span>

## <a name="next-steps"></a><span data-ttu-id="5831c-136">Дополнительная информация</span><span class="sxs-lookup"><span data-stu-id="5831c-136">Next steps</span></span>

* <span data-ttu-id="5831c-137">Узнайте, как развертывать ресурс только при определенных условиях, например при наличии или отсутствии значения конкретного параметра.</span><span class="sxs-lookup"><span data-stu-id="5831c-137">Learn how deploy a resource based on a condition, such as whether a parameter value is present.</span></span> <span data-ttu-id="5831c-138">См. сведения об [условном развертывании ресурсов в шаблоне Azure Resource Manager](./conditional-deploy.md).</span><span class="sxs-lookup"><span data-stu-id="5831c-138">See [Conditionally deploy a resource in an Azure Resource Manager template](./conditional-deploy.md).</span></span>

[cli]: /cli/azure/?view=azure-cli-latest
[github]: https://github.com/mspnp/template-examples
