---
title: Использование объекта в качестве параметра в шаблоне Azure Resource Manager
description: В статье описывается, как расширить возможности шаблонов Azure Resource Manager для использования объектов как параметров.
author: petertay
ms.date: 10/30/2018
ms.openlocfilehash: f0826d8ed1ce446d295ebdacc66d8b0bef0b0dec
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54111212"
---
# <a name="use-an-object-as-a-parameter-in-an-azure-resource-manager-template"></a><span data-ttu-id="ea5b4-103">Использование объекта в качестве параметра в шаблоне Azure Resource Manager</span><span class="sxs-lookup"><span data-stu-id="ea5b4-103">Use an object as a parameter in an Azure Resource Manager template</span></span>

<span data-ttu-id="ea5b4-104">Когда вы [создаете шаблоны Azure Resource Manager][azure-resource-manager-create-template], значения свойств ресурсов можно указать непосредственно в шаблоне или определить в виде параметров, значения которых будут предоставлены во время развертывания.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-104">When you [author Azure Resource Manager templates][azure-resource-manager-create-template], you can either specify resource property values directly in the template or define a parameter and provide values during deployment.</span></span> <span data-ttu-id="ea5b4-105">В небольших развертываниях можно спокойно применять параметры для всех значений свойств, но с ограничением в 255 параметров на одно развертывание.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-105">It's fine to use a parameter for each property value for small deployments, but there is a limit of 255 parameters per deployment.</span></span> <span data-ttu-id="ea5b4-106">Когда развертывания станут крупнее и сложнее, вы можете исчерпать доступный запас параметров.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-106">Once you get to larger and more complex deployments you may run out of parameters.</span></span>

<span data-ttu-id="ea5b4-107">Чтобы решить эту проблему, в качестве параметра можно использовать не значения, а объекты.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-107">One way to solve this problem is to use an object as a parameter instead of a value.</span></span> <span data-ttu-id="ea5b4-108">Для этого просто задайте параметр в шаблоне, а во время развертывания укажите для этого параметра не одно значение, а объект в формате JSON.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-108">To do this, define the parameter in your template and specify a JSON object instead of a single value during deployment.</span></span> <span data-ttu-id="ea5b4-109">Тогда вы сможете ссылаться в шаблоне на вложенные свойства этого параметра с помощью [функции `parameter()`][azure-resource-manager-functions] и оператора-точки.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-109">Then, reference the subproperties of the parameter using the [`parameter()` function][azure-resource-manager-functions] and dot operator in your template.</span></span>

<span data-ttu-id="ea5b4-110">Давайте рассмотрим пример шаблона, который развертывает ресурс виртуальной сети.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-110">Let's take a look at an example that deploys a virtual network resource.</span></span> <span data-ttu-id="ea5b4-111">Прежде всего нужно определить в шаблоне параметр `VNetSettings` и присвоить `type` значение `object`.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-111">First, let's specify a `VNetSettings` parameter in our template and set the `type` to `object`:</span></span>

```json
...
"parameters": {
    "VNetSettings":{"type":"object"}
},
```

<span data-ttu-id="ea5b4-112">Теперь мы предоставим значения для объекта `VNetSettings`.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-112">Next, let's provide values for the `VNetSettings` object:</span></span>

> [!NOTE]
> <span data-ttu-id="ea5b4-113">Как предоставить значения параметрам во время развертывания, см. в разделе **Параметры** в статье [Описание структуры и синтаксиса шаблонов Azure Resource Manager][azure-resource-manager-authoring-templates].</span><span class="sxs-lookup"><span data-stu-id="ea5b4-113">To learn how to provide parameter values during deployment, see the **parameters** section of [understand the structure and syntax of Azure Resource Manager templates][azure-resource-manager-authoring-templates].</span></span>

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

<span data-ttu-id="ea5b4-114">Как видите, здесь один параметр фактически задает три вложенных свойства: `name`, `addressPrefixes` и `subnets`.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-114">As you can see, our single parameter actually specifies three subproperties: `name`, `addressPrefixes`, and `subnets`.</span></span> <span data-ttu-id="ea5b4-115">Каждое из вложенных свойств содержит значение или другие вложенные свойства.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-115">Each of these subproperties either specifies a value or other subproperties.</span></span> <span data-ttu-id="ea5b4-116">Таким образом в один параметр можно поместить все значения, необходимые для развертывания виртуальной сети.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-116">The result is that our single parameter specifies all the values necessary to deploy our virtual network.</span></span>

<span data-ttu-id="ea5b4-117">Теперь мы перейдем к остальной части шаблона и посмотрим, как используется этот объект `VNetSettings`.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-117">Now let's have a look at the rest of our template to see how the `VNetSettings` object is used:</span></span>

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

<span data-ttu-id="ea5b4-118">Включенные в объект `VNetSettings` значения применяются к свойствам, к которым ресурс виртуальной сети обращается через функцию `parameters()` с использованием индексатора массива `[]` и (или) оператора-точки.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-118">The values of our `VNetSettings` object are applied to the properties required by our virtual network resource using the `parameters()` function with both the `[]` array indexer and the dot operator.</span></span> <span data-ttu-id="ea5b4-119">Такой подход удобен, когда вам нужно просто применить к ресурсу статические значения объекта параметров.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-119">This approach works if you just want to statically apply the values of the parameter object to the resource.</span></span> <span data-ttu-id="ea5b4-120">Но если вы хотите динамически назначать значения массиву свойств во время развертывания, используйте [цикл копирования][azure-resource-manager-create-multiple-instances].</span><span class="sxs-lookup"><span data-stu-id="ea5b4-120">However, if you want to dynamically assign an array of property values during deployment you can use a [copy loop][azure-resource-manager-create-multiple-instances].</span></span> <span data-ttu-id="ea5b4-121">Для этого предоставьте массив JSON со значениями свойств ресурсов. Цикл копирования динамически применяет значения к свойствам ресурса.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-121">To use a copy loop, you provide a JSON array of resource property values and the copy loop dynamically applies the values to the resource's properties.</span></span>

<span data-ttu-id="ea5b4-122">При использовании динамического подхода следует учитывать один важный аспект.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-122">There is one issue to be aware of if you use the dynamic approach.</span></span> <span data-ttu-id="ea5b4-123">Потенциальную проблему мы продемонстрируем на примере обычного массива значений свойств.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-123">To demonstrate the issue, let's take a look at a typical array of property values.</span></span> <span data-ttu-id="ea5b4-124">В этом примере значения, которые нужно присвоить свойствам, хранятся в переменной.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-124">In this example the values for our properties are stored in a variable.</span></span> <span data-ttu-id="ea5b4-125">Обратите внимание, что мы используем два массива с именами `firstProperty` и `secondProperty`.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-125">Notice we have two arrays here&mdash;one named `firstProperty` and one named `secondProperty`.</span></span>

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

<span data-ttu-id="ea5b4-126">Теперь посмотрите, как мы обращаемся к этим свойствам переменной в цикле копирования.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-126">Now let's take a look at the way we access the properties in the variable using a copy loop.</span></span>

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

<span data-ttu-id="ea5b4-127">Функция `copyIndex()` возвращает текущую итерацию цикла копирования, и мы применяем это значение в качестве индекса, обращаясь одновременно к двум массивам.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-127">The `copyIndex()` function returns the current iteration of the copy loop, and we use that as an index into each of the two arrays simultaneously.</span></span>

<span data-ttu-id="ea5b4-128">Это допустимо, если массивы имеют одинаковую длину.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-128">This works fine when the two arrays are the same length.</span></span> <span data-ttu-id="ea5b4-129">Но если вы ошибетесь при определении массивов, и они будут иметь разную длину, шаблон не пройдет проверку во время развертывания.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-129">The issue arises if you've made a mistake and the two arrays are different lengths&mdash;in this case your template will fail validation during deployment.</span></span> <span data-ttu-id="ea5b4-130">Эту проблему можно предотвратить, включив все свойства в один объект, ведь так вам будет проще заметить отсутствие значения.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-130">You can avoid this issue by including all your properties in a single object, because it is much easier to see when a value is missing.</span></span> <span data-ttu-id="ea5b4-131">Вот еще один пример объекта параметров, в котором каждый элемент массива `propertyObject` представляет объединение массивов `firstProperty` и `secondProperty` из предыдущего примера.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-131">For example, let's take a look another parameter object in which each element of the `propertyObject` array is the union of the `firstProperty` and `secondProperty` arrays from earlier.</span></span>

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

<span data-ttu-id="ea5b4-132">Обратите внимание на третий элемент в массиве.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-132">Notice the third element in the array?</span></span> <span data-ttu-id="ea5b4-133">Здесь отсутствует свойство `number`, но теперь, когда вы создали параметр упомянутым образом, вам намного проще это заметить.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-133">It's missing the `number` property, but it's much easier to notice that you've missed it when you're authoring the parameter values this way.</span></span>

## <a name="using-a-property-object-in-a-copy-loop"></a><span data-ttu-id="ea5b4-134">Использование объекта свойств в цикле копирования</span><span class="sxs-lookup"><span data-stu-id="ea5b4-134">Using a property object in a copy loop</span></span>

<span data-ttu-id="ea5b4-135">Такой подход становится еще полезнее в сочетании с [циклом последовательного копирования][azure-resource-manager-create-multiple], особенно для развертывания дочерних ресурсов.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-135">This approach becomes even more useful when combined with the [serial copy loop][azure-resource-manager-create-multiple], particularly for deploying child resources.</span></span>

<span data-ttu-id="ea5b4-136">Чтобы продемонстрировать это, рассмотрим шаблон для развертывания [группы безопасности сети (NSG)][nsg] с двумя правилами безопасности.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-136">To demonstrate this, let's look at a template that deploys a [network security group (NSG)][nsg] with two security rules.</span></span>

<span data-ttu-id="ea5b4-137">Изучение шаблона мы начнем со списка параметров.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-137">First, let's take a look at our parameters.</span></span> <span data-ttu-id="ea5b4-138">В этом шаблоне мы определили один параметр с именем `networkSecurityGroupsSettings`, который включает массив с именем `securityRules`.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-138">When we look at our template we'll see that we've defined one parameter named `networkSecurityGroupsSettings` that includes an array named `securityRules`.</span></span> <span data-ttu-id="ea5b4-139">Этот массив содержит два объекта JSON, которые определяют несколько параметров для правила безопасности.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-139">This array contains two JSON objects that specify a number of settings for a security rule.</span></span>

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

<span data-ttu-id="ea5b4-140">Теперь перейдем к самому шаблону.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-140">Now let's take a look at our template.</span></span> <span data-ttu-id="ea5b4-141">Первый ресурс с именем `NSG1` развертывает группу безопасности сети.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-141">Our first resource named `NSG1` deploys the NSG.</span></span> <span data-ttu-id="ea5b4-142">Второй ресурс с именем `loop-0` выполняет две функции: во-первых, он определяет зависимость (`dependsOn`) от группы безопасности сети, благодаря чему развертывание ресурса не начнется, пока `NSG1` не завершит развертывание; во-вторых, он является первой итерацией последовательного цикла.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-142">Our second resource named `loop-0` performs two functions: first, it `dependsOn` the NSG so its deployment doesn't begin until `NSG1` is completed, and it is the first iteration of the sequential loop.</span></span> <span data-ttu-id="ea5b4-143">Третий ресурс здесь — это вложенный шаблон, который развертывает правила безопасности, используя объект для получения значений своих параметров, как показано в предыдущем примере.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-143">Our third resource is a nested template that deploys our security rules using an object for its parameter values as in the last example.</span></span>

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

<span data-ttu-id="ea5b4-144">Давайте подробнее рассмотрим, как значения свойств указываются в дочернем ресурсе `securityRules`.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-144">Let's take a closer look at how we specify our property values in the `securityRules` child resource.</span></span> <span data-ttu-id="ea5b4-145">Мы ссылаемся на свойства с помощью функции `parameter()`, а затем применяем оператор-точку для ссылок на массив `securityRules`, используя в качестве индекса текущее значение итерации.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-145">All of our properties are referenced using the `parameter()` function, and then we use the dot operator to reference our `securityRules` array, indexed by the current value of the iteration.</span></span> <span data-ttu-id="ea5b4-146">И наконец, еще один оператор-точку мы используем для ссылки на имя объекта.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-146">Finally, we use another dot operator to reference the name of the object.</span></span>

## <a name="try-the-template"></a><span data-ttu-id="ea5b4-147">Пробное использование шаблона</span><span class="sxs-lookup"><span data-stu-id="ea5b4-147">Try the template</span></span>

<span data-ttu-id="ea5b4-148">Пример шаблона доступен на [сайте GitHub][github].</span><span class="sxs-lookup"><span data-stu-id="ea5b4-148">An example template is available on [GitHub][github].</span></span> <span data-ttu-id="ea5b4-149">Чтобы развернуть этот шаблон с помощью Azure CLI, выполните следующие команды [Azure CLI][cli]:</span><span class="sxs-lookup"><span data-stu-id="ea5b4-149">To deploy the template, clone the repo and run the following [Azure CLI][cli] commands:</span></span>

```bash
git clone https://github.com/mspnp/template-examples.git
cd template-examples/example3-object-param
az group create --location <location> --name <resource-group-name>
az group deployment create -g <resource-group-name> \
    --template-uri https://raw.githubusercontent.com/mspnp/template-examples/master/example3-object-param/deploy.json \
    --parameters deploy.parameters.json
```

## <a name="next-steps"></a><span data-ttu-id="ea5b4-150">Дополнительная информация</span><span class="sxs-lookup"><span data-stu-id="ea5b4-150">Next steps</span></span>

- <span data-ttu-id="ea5b4-151">Узнайте, как создать шаблон, который перебирает массив объектов и преобразует его в схему JSON.</span><span class="sxs-lookup"><span data-stu-id="ea5b4-151">Learn how to create a template that iterates through an object array and transforms it into a JSON schema.</span></span> <span data-ttu-id="ea5b4-152">См. сведения о [реализации преобразователя и сборщика свойств в шаблоне Azure Resource Manager](./collector.md).</span><span class="sxs-lookup"><span data-stu-id="ea5b4-152">See [Implement a property transformer and collector in an Azure Resource Manager template](./collector.md)</span></span>

<!-- links -->

[azure-resource-manager-authoring-templates]: /azure/azure-resource-manager/resource-group-authoring-templates
[azure-resource-manager-create-template]: /azure/azure-resource-manager/resource-manager-create-first-template
[azure-resource-manager-create-multiple-instances]: /azure/azure-resource-manager/resource-group-create-multiple
[azure-resource-manager-functions]: /azure/azure-resource-manager/resource-group-template-functions-resource
[nsg]: /azure/virtual-network/virtual-networks-nsg
[cli]: /cli/azure/?view=azure-cli-latest
[github]: https://github.com/mspnp/template-examples
