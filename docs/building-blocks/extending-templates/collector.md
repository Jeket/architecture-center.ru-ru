---
title: "Создание преобразователя и сборщика свойств в шаблоне Azure Resource Manager"
description: "В статье описано, как создать преобразователь и сборщик свойств в шаблоне Azure Resource Manager"
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: 893779e652b845b3d936d11936dc767ef632fa43
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/14/2017
---
# <a name="implement-a-property-transformer-and-collector-in-an-azure-resource-manager-template"></a><span data-ttu-id="9fc05-103">Создание преобразователя и сборщика свойств в шаблоне Azure Resource Manager</span><span class="sxs-lookup"><span data-stu-id="9fc05-103">Implement a property transformer and collector in an Azure Resource Manager template</span></span>

<span data-ttu-id="9fc05-104">Из статьи [Использование объекта в качестве параметра в шаблоне Azure Resource Manager][objects-as-parameters] вы узнаете, как сохранять в объекте значения свойств ресурсов и применять их к ресурсу во время развертывания.</span><span class="sxs-lookup"><span data-stu-id="9fc05-104">In [use an object as a parameter in an Azure Resource Manager template][objects-as-parameters], you learned how to store resource property values in an object and apply them to a resource during deployment.</span></span> <span data-ttu-id="9fc05-105">Этот метод очень удобен для управления параметрами, но для работы с ним необходимо сопоставлять свойства объекта со свойствами ресурсов при каждом их использовании в шаблоне.</span><span class="sxs-lookup"><span data-stu-id="9fc05-105">While this is a very useful way to manage your parameters, it still requires you to map the object's properties to resource properties each time you use it in your template.</span></span>

<span data-ttu-id="9fc05-106">Чтобы не делать это, вы можете реализовать шаблон с преобразователем и сборщиком свойств, который выполняет итерацию массива объекта и преобразует его в схему JSON, ожидаемую ресурсом.</span><span class="sxs-lookup"><span data-stu-id="9fc05-106">To work around this, you can implement a property transform and collector template that iterates your object array and transforms it into the JSON schema expected by the resource.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="9fc05-107">Такой подход требует глубокого понимания функций и шаблонов Resource Manager.</span><span class="sxs-lookup"><span data-stu-id="9fc05-107">This approach requires that you have a deep understanding of Resource Manager templates and functions.</span></span>

<span data-ttu-id="9fc05-108">Далее мы объясним, как создать сборщик и преобразователь свойств на конкретном примере шаблона, который развертывает [группу безопасности сети (NSG)][nsg].</span><span class="sxs-lookup"><span data-stu-id="9fc05-108">Let's take a look at how we can implement a property collector and transformer with an example that deploys a [network security group (NSG)][nsg].</span></span> <span data-ttu-id="9fc05-109">На схеме ниже показана связь между используемыми шаблонами и ресурсами в этих шаблонах.</span><span class="sxs-lookup"><span data-stu-id="9fc05-109">The diagram below shows the relationship between our templates and our resources within those templates:</span></span>

![Архитектура сборщика и преобразователя свойств](../_images/collector-transformer.png)

<span data-ttu-id="9fc05-111">У нас есть **вызывающий шаблон** с двумя ресурсами:</span><span class="sxs-lookup"><span data-stu-id="9fc05-111">Our **calling template** includes two resources:</span></span>
* <span data-ttu-id="9fc05-112">ссылка на шаблон, которая вызывает **шаблон сборщика**;</span><span class="sxs-lookup"><span data-stu-id="9fc05-112">a template link that invokes our **collector template**.</span></span>
* <span data-ttu-id="9fc05-113">ресурс группы безопасности сети, который мы собираемся развертывать.</span><span class="sxs-lookup"><span data-stu-id="9fc05-113">the NSG resource to deploy.</span></span>

<span data-ttu-id="9fc05-114">**Шаблон сборщика** также содержит два ресурса:</span><span class="sxs-lookup"><span data-stu-id="9fc05-114">Our **collector template** includes two resources:</span></span>
* <span data-ttu-id="9fc05-115">ресурс **привязки**;</span><span class="sxs-lookup"><span data-stu-id="9fc05-115">an **anchor** resource.</span></span>
* <span data-ttu-id="9fc05-116">ссылка на шаблон, которая вызывает шаблон преобразования в цикле копирования.</span><span class="sxs-lookup"><span data-stu-id="9fc05-116">a template link that invokes the transform template in a copy loop.</span></span>

<span data-ttu-id="9fc05-117">Этот **шаблон преобразования** включает один ресурс: пустой шаблон с одной переменной, которая преобразует объект `source` в схему JSON, ожидаемую ресурсом NSG в **основном шаблоне**.</span><span class="sxs-lookup"><span data-stu-id="9fc05-117">Our **transform template** includes a single resource: an empty template with a variable that transforms our `source` JSON to the JSON schema expected by our NSG resource in the **main template**.</span></span>

## <a name="parameter-object"></a><span data-ttu-id="9fc05-118">Объект параметров</span><span class="sxs-lookup"><span data-stu-id="9fc05-118">Parameter object</span></span>

<span data-ttu-id="9fc05-119">В этом примере мы применим тот же объект параметров `securityRules`, который мы создали в руководстве по [использованию объектов в качестве параметров][objects-as-parameters].</span><span class="sxs-lookup"><span data-stu-id="9fc05-119">We'll be using our `securityRules` parameter object from [objects as parameters][objects-as-parameters].</span></span> <span data-ttu-id="9fc05-120">**Шаблон преобразования** преобразует каждый объект из массива `securityRules` в схему JSON, которую ожидает ресурс NSG в **вызывающем шаблоне**.</span><span class="sxs-lookup"><span data-stu-id="9fc05-120">Our **transform template** will transform each object in the `securityRules` array into the JSON schema expected by the NSG resource in our **calling template**.</span></span>

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

<span data-ttu-id="9fc05-121">Изучение шаблонов мы начнем с **шаблона преобразования**.</span><span class="sxs-lookup"><span data-stu-id="9fc05-121">Let's look at our **transform template** first.</span></span>

## <a name="transform-template"></a><span data-ttu-id="9fc05-122">Шаблон преобразования</span><span class="sxs-lookup"><span data-stu-id="9fc05-122">Transform template</span></span>

<span data-ttu-id="9fc05-123">**Шаблон преобразования** содержит два параметра, которые передаются от **шаблона сборщика**:</span><span class="sxs-lookup"><span data-stu-id="9fc05-123">Our **transform template** includes two parameters that are passed from the **collector template**:</span></span> 
* <span data-ttu-id="9fc05-124">объект `source`, который принимает значение одного из объектов со значениям свойств, полученных из массива свойств.</span><span class="sxs-lookup"><span data-stu-id="9fc05-124">`source` is an object that receives one of the property value objects from the property array.</span></span> <span data-ttu-id="9fc05-125">В нашем примере в него поочередно будут передаваться объекты из массива `"securityRules"`;</span><span class="sxs-lookup"><span data-stu-id="9fc05-125">In our example, each object from the `"securityRules"` array will be passed in one at a time.</span></span>
* <span data-ttu-id="9fc05-126">массив `state`, в который передаются сцепленные результаты всех предыдущих преобразований.</span><span class="sxs-lookup"><span data-stu-id="9fc05-126">`state` is an array that receives the concatenated results of all the previous transforms.</span></span> <span data-ttu-id="9fc05-127">Это и есть сборщик преобразованных данных в формате JSON.</span><span class="sxs-lookup"><span data-stu-id="9fc05-127">This is the collection of transformed JSON.</span></span>

<span data-ttu-id="9fc05-128">Мы используем следующий набор параметров:</span><span class="sxs-lookup"><span data-stu-id="9fc05-128">Our parameters look like this:</span></span>

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "source": { "type": "object" },
    "state": {
      "type": "array",
      "defaultValue": [ ]
    }
  },
```

<span data-ttu-id="9fc05-129">Также этот шаблон определяет переменную с именем `instance`.</span><span class="sxs-lookup"><span data-stu-id="9fc05-129">Our template also defines a variable named `instance`.</span></span> <span data-ttu-id="9fc05-130">Он выполняет фактическое преобразование объекта `source` в требуемую схему JSON:</span><span class="sxs-lookup"><span data-stu-id="9fc05-130">It performs the actual tranform of our `source` object into the required JSON schema:</span></span>

```json
  "variables": {
    "instance": [
      {
        "name": "[parameters('source').name]",
        "properties":{
            "description": "[parameters('source').description]",
            "protocol": "[parameters('source').protocol]",
            "sourcePortRange": "[parameters('source').sourcePortRange]",
            "destinationPortRange": "[parameters('source').destinationPortRange]",
            "sourceAddressPrefix": "[parameters('source').sourceAddressPrefix]",
            "destinationAddressPrefix": "[parameters('source').destinationAddressPrefix]",
            "access": "[parameters('source').access]",
            "priority": "[parameters('source').priority]",
            "direction": "[parameters('source').direction]"            
        }
      }
    ]

  },
```

<span data-ttu-id="9fc05-131">И в завершение шаблон сцепляет в переменной `output` все собранные преобразования параметра `state` и выполняет текущее преобразование в переменной `instance`:</span><span class="sxs-lookup"><span data-stu-id="9fc05-131">Finally, the `output` of our template concatenates the collected transforms of our `state` parameter with the current transform performed by our `instance` variable:</span></span>

```json
  "outputs": {
    "collection": {
      "type": "array",
      "value": "[concat(parameters('state'), variables('instance'))]"
    }
```

<span data-ttu-id="9fc05-132">Теперь мы перейдем к **шаблону сборщика** и посмотрим, как он передает значения параметров.</span><span class="sxs-lookup"><span data-stu-id="9fc05-132">Next, let's take a look at our **collector template** to see how it passes in our parameter values.</span></span>

## <a name="collector-template"></a><span data-ttu-id="9fc05-133">Шаблон сборщика</span><span class="sxs-lookup"><span data-stu-id="9fc05-133">Collector template</span></span>

<span data-ttu-id="9fc05-134">**Шаблон сборщика** содержит три параметра:</span><span class="sxs-lookup"><span data-stu-id="9fc05-134">Our **collector template** includes three parameters:</span></span>
* <span data-ttu-id="9fc05-135">`source` — это полный массив объекта параметров.</span><span class="sxs-lookup"><span data-stu-id="9fc05-135">`source` is our complete parameter object array.</span></span> <span data-ttu-id="9fc05-136">Он передается из **вызывающего шаблона**.</span><span class="sxs-lookup"><span data-stu-id="9fc05-136">It's passed in by the **calling template**.</span></span> <span data-ttu-id="9fc05-137">Он имеет такое же имя, как и параметр `source` в **шаблоне преобразования**, но есть и одно важное различие: хотя это полный массив, в один момент времени в **шаблон преобразования** передается только один элемент этого массива.</span><span class="sxs-lookup"><span data-stu-id="9fc05-137">This has the same name as the `source` parameter in our **transform template** but there is one key difference that you may have already noticed: this is the complete array, but we only pass one element of this array to the **transform template** at a time.</span></span>
* <span data-ttu-id="9fc05-138">`transformTemplateUri` содержит URI **шаблона преобразования**.</span><span class="sxs-lookup"><span data-stu-id="9fc05-138">`transformTemplateUri` is the URI of our **transform template**.</span></span> <span data-ttu-id="9fc05-139">Здесь мы определяем его как параметр, чтобы шаблон можно было использовать в других развертываниях.</span><span class="sxs-lookup"><span data-stu-id="9fc05-139">We're defining it as a parameter here for template reusability.</span></span>
* <span data-ttu-id="9fc05-140">`state` изначально является пустым массивом, который передается в **шаблон преобразования**.</span><span class="sxs-lookup"><span data-stu-id="9fc05-140">`state` is an initially empty array that we pass to our **transform template**.</span></span> <span data-ttu-id="9fc05-141">Когда цикл копирования завершится, в этом массиве будет храниться коллекция преобразованных объектов параметров.</span><span class="sxs-lookup"><span data-stu-id="9fc05-141">It stores the collection of transformed parameter objects when the copy loop is complete.</span></span>

<span data-ttu-id="9fc05-142">Мы используем следующий набор параметров:</span><span class="sxs-lookup"><span data-stu-id="9fc05-142">Our parameters look like this:</span></span>

```json
  "parameters": {
    "source": { "type": "array" },
    "transformTemplateUri": { "type": "string" },
    "state": {
      "type": "array",
      "defaultValue": [ ]
    }
``` 

<span data-ttu-id="9fc05-143">Затем определяется переменная с именем `count`.</span><span class="sxs-lookup"><span data-stu-id="9fc05-143">Next, we define a variable named `count`.</span></span> <span data-ttu-id="9fc05-144">В качестве значения ей присваивается длина массива объектов параметров `source`:</span><span class="sxs-lookup"><span data-stu-id="9fc05-144">Its value is the length of the `source` parameter object array:</span></span>

```json
  "variables": {
    "count": "[length(parameters('source'))]"
  },
```

<span data-ttu-id="9fc05-145">Как и следует ожидать, мы используем его для определения числа итераций в цикле копирования.</span><span class="sxs-lookup"><span data-stu-id="9fc05-145">As you might suspect, we use it for the number of iterations in our copy loop.</span></span>

<span data-ttu-id="9fc05-146">Теперь давайте посмотрим на ресурсы.</span><span class="sxs-lookup"><span data-stu-id="9fc05-146">Now let's take a look at our resources.</span></span> <span data-ttu-id="9fc05-147">Мы определим два ресурса:</span><span class="sxs-lookup"><span data-stu-id="9fc05-147">We define two resources:</span></span>
* <span data-ttu-id="9fc05-148">`loop-0` имеет нулевое начальное значение и используется для цикла копирования;</span><span class="sxs-lookup"><span data-stu-id="9fc05-148">`loop-0` is the zero-based resource for our copy loop.</span></span>
* <span data-ttu-id="9fc05-149">`loop-` объединяется с результатом функции `copyIndex(1)` для получения уникального имени ресурса для каждой итерации, начиная с `1`.</span><span class="sxs-lookup"><span data-stu-id="9fc05-149">`loop-` is concatenated with the result of the `copyIndex(1)` function to generate a unique iteration-based name for our resource, starting with `1`.</span></span>

<span data-ttu-id="9fc05-150">Описание ресурсов выглядит следующим образом:</span><span class="sxs-lookup"><span data-stu-id="9fc05-150">Our resources look like this:</span></span>

```json
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2015-01-01",
      "name": "loop-0",
      "properties": {
        "mode": "Incremental",
        "parameters": { },
        "template": {
          "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": { },
          "variables": { },
          "resources": [ ],
          "outputs": {
            "collection": {
              "type": "array",
              "value": "[parameters('state')]"
            }
          }
        }
      }
    },
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2015-01-01",
      "name": "[concat('loop-', copyindex(1))]",
      "copy": {
        "name": "iterator",
        "count": "[variables('count')]",
        "mode": "serial"
      },
      "dependsOn": [
        "loop-0"
      ],
      "properties": {
        "mode": "Incremental",
        "templateLink": { "uri": "[parameters('transformTemplateUri')]" },
        "parameters": {
          "source": { "value": "[parameters('source')[copyindex()]]" },
          "state": { "value": "[reference(concat('loop-', copyindex())).outputs.collection.value]" }
        }
      }
    }
  ],
```

<span data-ttu-id="9fc05-151">Давайте более подробно рассмотрим параметры, которые передаются в **шаблон преобразования** из вложенного шаблона.</span><span class="sxs-lookup"><span data-stu-id="9fc05-151">Let's take a closer look at the parameters we're passing to our **transform template** in the nested template.</span></span> <span data-ttu-id="9fc05-152">Как вы помните, параметр `source` здесь передает текущий объект в массив объектов параметров `source`.</span><span class="sxs-lookup"><span data-stu-id="9fc05-152">Recall from earlier that our `source` parameter passes the current object in the `source` parameter object array.</span></span> <span data-ttu-id="9fc05-153">Параметр `state` позволяет собирать результаты, поочередно принимая выходные данные предыдущей итерации цикла копирования. Обратите внимание, что функция `reference()` вызывает функцию `copyIndex()` без параметров, чтобы указать имя (`name`) ранее связанного объекта шаблона и передать его в текущую итерацию.</span><span class="sxs-lookup"><span data-stu-id="9fc05-153">The `state` parameter is where the collection happens, because it takes the output of the previous iteration of our copy loop&mdash;notice that the `reference()` function uses the `copyIndex()` function with no parameter to reference the `name` of our previous linked template object&mdash;and passes it to the current iteration.</span></span>

<span data-ttu-id="9fc05-154">И наконец, объект `output` в нашем шаблоне возвращает `output` выходные данные последней итерации в **шаблоне преобразования**:</span><span class="sxs-lookup"><span data-stu-id="9fc05-154">Finally, the `output` of our template returns the `output` of the last iteration of our **transform template**:</span></span>

```json
  "outputs": {
    "result": {
      "type": "array",
      "value": "[reference(concat('loop-', variables('count'))).outputs.collection.value]"
    }
  }
```
<span data-ttu-id="9fc05-155">Может показаться нелогичным, что мы возвращаем `output` из последней итерации **шаблона преобразования** в **вызывающий шаблон**, ведь мы уже сохранили это значение в параметре `source`.</span><span class="sxs-lookup"><span data-stu-id="9fc05-155">It may seem counterintuitive to return the `output` of the last iteration of our **transform template** to our **calling template** because it appeared we were storing it in our `source` parameter.</span></span> <span data-ttu-id="9fc05-156">Но не забывайте, что в последней итерации **шаблона преобразования** у нас собран полный массив преобразованных объектов свойств, и именно его нам нужно вернуть.</span><span class="sxs-lookup"><span data-stu-id="9fc05-156">However, remember that it's the last iteration of our **transform template** that holds the complete array of transformed property objects, and that's what we want to return.</span></span>

<span data-ttu-id="9fc05-157">В конце этой статьи мы рассмотрим способ вызова **шаблона сборщика** из **вызывающего шаблона**.</span><span class="sxs-lookup"><span data-stu-id="9fc05-157">Finally, let's take a look at how to call the **collector template** from our **calling template**.</span></span>

## <a name="calling-template"></a><span data-ttu-id="9fc05-158">Вызывающий шаблон</span><span class="sxs-lookup"><span data-stu-id="9fc05-158">Calling template</span></span>

<span data-ttu-id="9fc05-159">**Вызывающий шаблон** определяет один параметр с именем `networkSecurityGroupsSettings`:</span><span class="sxs-lookup"><span data-stu-id="9fc05-159">Our **calling template** defines a single parameter named `networkSecurityGroupsSettings`:</span></span>

```json
...
"parameters": {
    "networkSecurityGroupsSettings": {
        "type": "object"
    }
```

<span data-ttu-id="9fc05-160">Затем этот шаблон определяет одну переменную с именем `collectorTemplateUri`:</span><span class="sxs-lookup"><span data-stu-id="9fc05-160">Next, our template defines a single variable named `collectorTemplateUri`:</span></span>

```json
"variables": {
    "collectorTemplateUri": "[uri(deployment().properties.templateLink.uri, 'collector.template.json')]"
  }
```

<span data-ttu-id="9fc05-161">Как и следовало ожидать, в ней хранится URI **шаблона сборщика**, который будет использоваться в ресурсе связанного шаблона:</span><span class="sxs-lookup"><span data-stu-id="9fc05-161">As you would expect, this is the URI for the **collector template** that will be used by our linked template resource:</span></span>

```json
{
    "apiVersion": "2015-01-01",
    "name": "collector",
    "type": "Microsoft.Resources/deployments",
    "properties": {
        "mode": "Incremental",
        "templateLink": {
            "uri": "[variables('linkedTemplateUri')]",
            "contentVersion": "1.0.0.0"
        },
        "parameters": {
            "source" : {"value": "[parameters('networkSecurityGroupsSettings').securityRules]"},
            "transformTemplateUri": { "value": "[uri(deployment().properties.templateLink.uri, 'transform.json')]"}
        }
    }
}
```

<span data-ttu-id="9fc05-162">Затем мы передаем два параметра в **шаблон сборщика**:</span><span class="sxs-lookup"><span data-stu-id="9fc05-162">We pass two parameters to the **collector template**:</span></span>
* <span data-ttu-id="9fc05-163">`source` — это массив объектов свойств.</span><span class="sxs-lookup"><span data-stu-id="9fc05-163">`source` is our property object array.</span></span> <span data-ttu-id="9fc05-164">В нашем примере это параметр `networkSecurityGroupsSettings`.</span><span class="sxs-lookup"><span data-stu-id="9fc05-164">In our example, it's our `networkSecurityGroupsSettings` parameter.</span></span>
* <span data-ttu-id="9fc05-165">Переменную `transformTemplateUri` мы только что определили, присвоив ей значение URI **шаблона сборщика**.</span><span class="sxs-lookup"><span data-stu-id="9fc05-165">`transformTemplateUri` is the variable we just defined with the URI of our **collector template**.</span></span>

<span data-ttu-id="9fc05-166">И наконец, ресурс `Microsoft.Network/networkSecurityGroups` напрямую связывает выходные данные (`output`) связанного шаблона `collector` с его свойством `securityRules`:</span><span class="sxs-lookup"><span data-stu-id="9fc05-166">Finally, our `Microsoft.Network/networkSecurityGroups` resource directly assigns the `output` of the `collector` linked template resource to its `securityRules` property:</span></span>

```json
    {
      "apiVersion": "2015-06-15",
      "type": "Microsoft.Network/networkSecurityGroups",
      "name": "networkSecurityGroup1",
      "location": "[resourceGroup().location]",
      "properties": {
        "securityRules": "[reference('firstResource').outputs.result.value]"
      }
    }
  ],
  "outputs": {
      "instance":{
          "type": "array",
          "value": "[reference('firstResource').outputs.result.value]"
      }

  }
```

## <a name="next-steps"></a><span data-ttu-id="9fc05-167">Дальнейшие действия</span><span class="sxs-lookup"><span data-stu-id="9fc05-167">Next steps</span></span>

* <span data-ttu-id="9fc05-168">Эта же методика реализована в [проекте стандартных блоков шаблона](https://github.com/mspnp/template-building-blocks) и [эталонных архитектурах Azure](/azure/architecture/reference-architectures/).</span><span class="sxs-lookup"><span data-stu-id="9fc05-168">This technique is implemented in the [template building blocks project](https://github.com/mspnp/template-building-blocks) and the [Azure reference architectures](/azure/architecture/reference-architectures/).</span></span> <span data-ttu-id="9fc05-169">Вы можете создать на их основе собственную архитектуру или развернуть готовые примеры архитектуры.</span><span class="sxs-lookup"><span data-stu-id="9fc05-169">You can use these to create your own architecture or deploy one of our reference architectures.</span></span>

<!-- links -->
[objects-as-parameters]: ./objects-as-parameters.md
[resource-manager-linked-template]: /azure/azure-resource-manager/resource-group-linked-templates
[resource-manager-variables]: /azure/azure-resource-manager/resource-group-template-functions-deployment
[nsg]: /azure/virtual-network/virtual-networks-nsg
