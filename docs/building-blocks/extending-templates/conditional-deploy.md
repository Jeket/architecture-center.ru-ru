---
title: "Условное развертывание ресурсов в шаблоне Azure Resource Manager"
description: "Расширение возможностей в шаблоне Azure Resource Manager за счет условного развертывания ресурсов в зависимости от значения какого-то параметра"
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: e911e7dc41b4f71ebfaf13a00f8cdbb5b4e2578b
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/14/2017
---
# <a name="conditionally-deploy-a-resource-in-an-azure-resource-manager-template"></a><span data-ttu-id="70c09-103">Условное развертывание ресурсов в шаблоне Azure Resource Manager</span><span class="sxs-lookup"><span data-stu-id="70c09-103">Conditionally deploy a resource in an Azure Resource Manager template</span></span>

<span data-ttu-id="70c09-104">В некоторых ситуациях необходимо создать такой шаблон, который будет развертывать ресурс только при определенных условиях, например при наличии или отсутствии значения конкретного параметра.</span><span class="sxs-lookup"><span data-stu-id="70c09-104">There are some scenarios in which you need to design your template to deploy a resource based on a condition, such as whether or not a parameter value is present.</span></span> <span data-ttu-id="70c09-105">Например, вы можете создать шаблон для развертывания виртуальной сети, который содержит параметры, описывающие другие виртуальные сети для пиринга.</span><span class="sxs-lookup"><span data-stu-id="70c09-105">For example, your template may deploy a virtual network and include parameters to specify other virtual networks for peering.</span></span> <span data-ttu-id="70c09-106">Если параметры для пиринга не указаны, Resource Manager не должен развертывать ресурсы пиринга.</span><span class="sxs-lookup"><span data-stu-id="70c09-106">If you've not specified any parameter values for peering, you don't want Resource Manager to deploy the peering resource.</span></span>

<span data-ttu-id="70c09-107">Чтобы реализовать такой вариант, используйте в ресурсе [элемент `condition`][azure-resource-manager-condition], чтобы проверить длину массива параметров.</span><span class="sxs-lookup"><span data-stu-id="70c09-107">To accomplish this, use the [`condition` element][azure-resource-manager-condition] in the resource to test the length of your parameter array.</span></span> <span data-ttu-id="70c09-108">Если эта длина равна нулю, возвращайте значение `false`, чтобы заблокировать развертывание, а при любой длине больше нуля — значение `true`, чтобы разрешить развертывание.</span><span class="sxs-lookup"><span data-stu-id="70c09-108">If the length is zero, return `false` to prevent deployment, but for all values greater than zero return `true` to allow deployment.</span></span>

## <a name="example-template"></a><span data-ttu-id="70c09-109">Пример шаблона</span><span class="sxs-lookup"><span data-stu-id="70c09-109">Example template</span></span>

<span data-ttu-id="70c09-110">Рассмотрим этот вариант на конкретном примере шаблона.</span><span class="sxs-lookup"><span data-stu-id="70c09-110">Let's look at an example template that demonstrates this.</span></span> <span data-ttu-id="70c09-111">В этом шаблоне [ элемент `condition`][azure-resource-manager-condition] управляет развертыванием ресурса `Microsoft.Network/virtualNetworks/virtualNetworkPeerings`.</span><span class="sxs-lookup"><span data-stu-id="70c09-111">Our template uses the [`condition` element][azure-resource-manager-condition] to control deployment of the `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` resource.</span></span> <span data-ttu-id="70c09-112">Этот ресурс создает пиринг между двумя виртуальными сетями Azure в одном регионе.</span><span class="sxs-lookup"><span data-stu-id="70c09-112">This resource creates a peering between two Azure Virtual Networks in the same region.</span></span>

<span data-ttu-id="70c09-113">Рассмотрим подробно каждый раздел шаблона.</span><span class="sxs-lookup"><span data-stu-id="70c09-113">Let's take a look at each section of the template.</span></span>

<span data-ttu-id="70c09-114">Элемент `parameters` определяет один параметр с именем `virtualNetworkPeerings`.</span><span class="sxs-lookup"><span data-stu-id="70c09-114">The `parameters` element defines a single parameter named `virtualNetworkPeerings`:</span></span> 

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
<span data-ttu-id="70c09-115">Параметр `virtualNetworkPeerings` является массивом (`array`) со следующей схемой данных.</span><span class="sxs-lookup"><span data-stu-id="70c09-115">Our `virtualNetworkPeerings` parameter is an `array` and has the following schema:</span></span>

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

<span data-ttu-id="70c09-116">Свойства этого параметра определяют [настройки пиринга виртуальных сетей][vnet-peering-resource-schema].</span><span class="sxs-lookup"><span data-stu-id="70c09-116">The properties in our parameter specify the [settings related to peering virtual networks][vnet-peering-resource-schema].</span></span> <span data-ttu-id="70c09-117">Чтобы предоставить значения для этих свойств, нужно определить ресурс `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` в разделе `resources`:</span><span class="sxs-lookup"><span data-stu-id="70c09-117">We'll provide the values for these properties when we specify the `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` resource in the `resources` section:</span></span>

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
<span data-ttu-id="70c09-118">В этой части нашего примера есть несколько элементов.</span><span class="sxs-lookup"><span data-stu-id="70c09-118">There are a couple of things going on in this part of our template.</span></span> <span data-ttu-id="70c09-119">Во-первых, фактически развертываемый ресурс определен во встроенном шаблоне типа `Microsoft.Resources/deployments`, в котором содержится дополнительный шаблон для развертывания `Microsoft.Network/virtualNetworks/virtualNetworkPeerings`.</span><span class="sxs-lookup"><span data-stu-id="70c09-119">First, the actual resource being deployed is an inline template of type `Microsoft.Resources/deployments` that includes its own template that actually deploys the `Microsoft.Network/virtualNetworks/virtualNetworkPeerings`.</span></span>

<span data-ttu-id="70c09-120">Чтобы имя (`name`) встроенного шаблона было уникальным, мы вычисляем его значение путем объединения текущего значения `copyIndex()` с префиксом `vnp-`.</span><span class="sxs-lookup"><span data-stu-id="70c09-120">Our `name` for the inline template is made unique by concatenating the current iteration of the `copyIndex()` with the prefix `vnp-`.</span></span> 

<span data-ttu-id="70c09-121">Элемент `condition` указывает, что этот ресурс обрабатывается в том случае, когда функция `greater()` возвращает значение `true`.</span><span class="sxs-lookup"><span data-stu-id="70c09-121">The `condition` element specifies that our resource should be processed when the `greater()` function evaluates to `true`.</span></span> <span data-ttu-id="70c09-122">Здесь мы проверяем, что параметр `virtualNetworkPeerings` для массива имеет значение больше нуля (`greater()`).</span><span class="sxs-lookup"><span data-stu-id="70c09-122">Here, we're testing if the `virtualNetworkPeerings` parameter array is `greater()` than zero.</span></span> <span data-ttu-id="70c09-123">Если это верно, возвращается значение `true` и наше условие (`condition`) считается выполненным.</span><span class="sxs-lookup"><span data-stu-id="70c09-123">If it is, it evaluates to `true` and the `condition` is satisfied.</span></span> <span data-ttu-id="70c09-124">В противном случае возвращается `false`.</span><span class="sxs-lookup"><span data-stu-id="70c09-124">Otherwise, it's `false`.</span></span>

<span data-ttu-id="70c09-125">Далее описан цикл `copy`.</span><span class="sxs-lookup"><span data-stu-id="70c09-125">Next, we specify our `copy` loop.</span></span> <span data-ttu-id="70c09-126">Это цикл типа `serial`, то есть он выполняется последовательно, и каждый следующий ресурс ожидает, пока завершится развертывание предыдущего ресурса.</span><span class="sxs-lookup"><span data-stu-id="70c09-126">It's a `serial` loop that means the loop is done in sequence, with each resource waiting until the last resource has been deployed.</span></span> <span data-ttu-id="70c09-127">Свойство `count` указывает, сколько раз будет выполнен этот цикл.</span><span class="sxs-lookup"><span data-stu-id="70c09-127">The `count` property specifies the number of times the loop iterates.</span></span> <span data-ttu-id="70c09-128">Обычно здесь используется значение длины массива `virtualNetworkPeerings`, поскольку именно этот массив содержит объекты параметров для развертывания ресурсов.</span><span class="sxs-lookup"><span data-stu-id="70c09-128">Here, normally we'd set it to the length of the `virtualNetworkPeerings` array because it contains the parameter objects specifying the resource we want to deploy.</span></span> <span data-ttu-id="70c09-129">Но если мы просто подставим это значение, то проверка значения завершится ошибкой в случае пустого массива, ведь Resource Manager зафиксирует попытку обратиться к несуществующим свойствам.</span><span class="sxs-lookup"><span data-stu-id="70c09-129">However, if we do that, validation will fail if the array is empty because Resource Manager notices that we are attempting to access properties that do not exist.</span></span> <span data-ttu-id="70c09-130">Мы можем обойти это ограничение.</span><span class="sxs-lookup"><span data-stu-id="70c09-130">We can work around this, however.</span></span> <span data-ttu-id="70c09-131">Для этого нам потребуются следующие переменные.</span><span class="sxs-lookup"><span data-stu-id="70c09-131">Let's take a look at the variables we'll need:</span></span>

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

<span data-ttu-id="70c09-132">Переменная `workaround` включает в себя два свойства с именами `true` и `false`.</span><span class="sxs-lookup"><span data-stu-id="70c09-132">Our `workaround` variable includes two properties, one named `true` and one named `false`.</span></span> <span data-ttu-id="70c09-133">Свойство `true` указывает на массив параметров `virtualNetworkPeerings`.</span><span class="sxs-lookup"><span data-stu-id="70c09-133">The `true` property evaluates to the value of the `virtualNetworkPeerings` parameter array.</span></span> <span data-ttu-id="70c09-134">Свойство `false` содержит пустой объект со всеми именованными свойствами, которые ожидает получить Resource Manager. Обратите внимание, что `false` фактически является массивом, как и параметр `virtualNetworkPeerings` что позволяет успешно пройти проверку.</span><span class="sxs-lookup"><span data-stu-id="70c09-134">The `false` property evaluates to an empty object including the named properties that Resource Manager expects to see&mdash;note that `false` is actually an array, just as our `virtualNetworkPeerings` parameter is, which will satisfy validation.</span></span> 

<span data-ttu-id="70c09-135">Переменная `peerings` использует эту новую переменную `workaround`, чтобы проверить, что длина массива параметров `virtualNetworkPeerings` больше нуля.</span><span class="sxs-lookup"><span data-stu-id="70c09-135">Our `peerings` variable uses our `workaround` variable by once again testing if the length of the `virtualNetworkPeerings` parameter array is greater than zero.</span></span> <span data-ttu-id="70c09-136">Если это так, `string` принимает значение `true`, а переменная `workaround` указывает на массив параметров `virtualNetworkPeerings`.</span><span class="sxs-lookup"><span data-stu-id="70c09-136">If it is, the `string` evaluates to `true` and the `workaround` variable evalutes to the `virtualNetworkPeerings` parameter array.</span></span> <span data-ttu-id="70c09-137">В противном случае возвращается `false`, и переменная `workaround` содержит пустой объект в первом элементе массива.</span><span class="sxs-lookup"><span data-stu-id="70c09-137">Otherwise, it evaluates to `false` and the `workaround` variable evaluates to our empty object in the first element of the array.</span></span>

<span data-ttu-id="70c09-138">Итак, мы решили проблему с проверкой значений и можем просто указать развертывание ресурса `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` во вложенном шаблоне, передавая ему параметры `name` и `properties` из массива параметров `virtualNetworkPeerings`.</span><span class="sxs-lookup"><span data-stu-id="70c09-138">Now that we've worked around the validation issue, we can simply specify the deployment of the `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` resource in the nested template, passing the `name` and `properties` from our `virtualNetworkPeerings` parameter array.</span></span> <span data-ttu-id="70c09-139">Этот механизм реализован в элементе `template`, который вложен в элемент `properties` нашего ресурса.</span><span class="sxs-lookup"><span data-stu-id="70c09-139">You can see this in the `template` element nested in the `properties` element of our resource.</span></span>

## <a name="next-steps"></a><span data-ttu-id="70c09-140">Дальнейшие действия</span><span class="sxs-lookup"><span data-stu-id="70c09-140">Next steps</span></span>

* <span data-ttu-id="70c09-141">Эта же методика реализована в [проекте стандартных блоков шаблона](https://github.com/mspnp/template-building-blocks) и [эталонных архитектурах Azure](/azure/architecture/reference-architectures/).</span><span class="sxs-lookup"><span data-stu-id="70c09-141">This technique is implemented in the [template building blocks project](https://github.com/mspnp/template-building-blocks) and the [Azure reference architectures](/azure/architecture/reference-architectures/).</span></span> <span data-ttu-id="70c09-142">Вы можете создать на их основе собственную архитектуру или развернуть готовые примеры архитектуры.</span><span class="sxs-lookup"><span data-stu-id="70c09-142">You can use these to create your own architecture or deploy one of our reference architectures.</span></span>

<!-- links -->
[azure-resource-manager-condition]: /azure/azure-resource-manager/resource-group-authoring-templates#resources
[azure-resource-manager-variable]: /azure/azure-resource-manager/resource-group-authoring-templates#variables
[vnet-peering-resource-schema]: /azure/templates/microsoft.network/virtualnetworks/virtualnetworkpeerings