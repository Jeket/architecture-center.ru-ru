---
title: Использование панелей мониторинга для визуализации метрик Azure Databricks
description: Развертывание панели мониторинга Grafana для наблюдения за производительностью в Azure Databricks.
author: petertaylor9999
ms.date: 03/26/2019
ms.openlocfilehash: 36fcd93f6ca757e8e750d0fcbbdf0311c08560b0
ms.sourcegitcommit: 1a3cc91530d56731029ea091db1f15d41ac056af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/03/2019
ms.locfileid: "58887834"
---
# <a name="use-dashboards-to-visualize-azure-databricks-metrics"></a><span data-ttu-id="cdcc7-103">Использование панелей мониторинга для визуализации метрик Azure Databricks</span><span class="sxs-lookup"><span data-stu-id="cdcc7-103">Use dashboards to visualize Azure Databricks metrics</span></span>

<span data-ttu-id="cdcc7-104">В этой статье показано, как настроить панели мониторинга Grafana для мониторинга задания Azure Databricks для проблем с производительностью.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-104">This article shows how to set up a Grafana dashboard to monitor Azure Databricks jobs for performance issues.</span></span>

<span data-ttu-id="cdcc7-105">[Azure Databricks](/azure/azure-databricks/) — это Быстрая, эффективные и совместной работы [Apache Spark](https://spark.apache.org/)— на основе Служба аналитики, которая позволяет легко разрабатывать и развертывать аналитику больших данных и решений искусственного интеллекта (ИИ).</span><span class="sxs-lookup"><span data-stu-id="cdcc7-105">[Azure Databricks](/azure/azure-databricks/) is a fast, powerful, and collaborative [Apache Spark](https://spark.apache.org/)–based analytics service that makes it easy to rapidly develop and deploy big data analytics and artificial intelligence (AI) solutions.</span></span> <span data-ttu-id="cdcc7-106">Мониторинг — это важная составляющая операционных рабочих нагрузок Azure Databricks в рабочей среде.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-106">Monitoring is a critical component of operating Azure Databricks workloads in production.</span></span> <span data-ttu-id="cdcc7-107">Первым делом для сбора метрик в рабочую область для анализа.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-107">The first step is to gather metrics into a workspace for analysis.</span></span> <span data-ttu-id="cdcc7-108">В Azure является лучшим решением для управления данными журнала [Azure Monitor](/azure/azure-monitor/).</span><span class="sxs-lookup"><span data-stu-id="cdcc7-108">In Azure, the best solution for managing log data is [Azure Monitor](/azure/azure-monitor/).</span></span> <span data-ttu-id="cdcc7-109">Azure Databricks не встроена поддержка отправке данных журнала в Azure monitor, но [библиотеки для использования этой функции](https://github.com/mspnp/spark-monitoring) доступен в [Github](https://github.com).</span><span class="sxs-lookup"><span data-stu-id="cdcc7-109">Azure Databricks does not natively support sending log data to Azure monitor, but a [library for this functionality](https://github.com/mspnp/spark-monitoring) is available in [Github](https://github.com).</span></span>

<span data-ttu-id="cdcc7-110">Эта библиотека позволяет ведение журнала метрик службы Azure Databricks, а также структуру Apache Spark, потоковая передача событий метрики запросов.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-110">This library enables logging of Azure Databricks service metrics as well as Apache Spark structure streaming query event metrics.</span></span> <span data-ttu-id="cdcc7-111">После успешного развертывания этой библиотеки в кластер Azure Databricks, Дополнительно можно развернуть набор [Grafana](https://granfana.com) панелей мониторинга, которые можно развернуть как часть рабочей среды.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-111">Once you've successfully deployed this library to an Azure Databricks cluster, you can further deploy a set of [Grafana](https://granfana.com) dashboards that you can deploy as part of your production environment.</span></span>

![Снимок экрана панели мониторинга](./_images/dashboard-screenshot.png)

## <a name="prequisites"></a><span data-ttu-id="cdcc7-113">Предварительные требования</span><span class="sxs-lookup"><span data-stu-id="cdcc7-113">Prequisites</span></span>

<span data-ttu-id="cdcc7-114">Клон [репозиторий Github](https://github.com/mspnp/spark-monitoring) и [следовать инструкциям по развертыванию](./configure-cluster.md) для создания и настройки ведения журнала Azure Monitor для библиотеки Azure Databricks для отправки журналов в рабочей области Azure Log Analytics.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-114">Clone the [Github repository](https://github.com/mspnp/spark-monitoring) and [follow the deployment instructions](./configure-cluster.md) to build and configure the Azure Monitor logging for Azure Databricks library to send logs to your Azure Log Analytics workspace.</span></span>

## <a name="deploy-the-azure-log-analytics-workspace"></a><span data-ttu-id="cdcc7-115">Развертывание рабочей области Azure Log Analytics</span><span class="sxs-lookup"><span data-stu-id="cdcc7-115">Deploy the Azure Log Analytics workspace</span></span>

<span data-ttu-id="cdcc7-116">Чтобы развернуть рабочую область Azure Log Analytics, выполните следующие действия.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-116">To deploy the Azure Log Analytics workspace, follow these steps:</span></span>

1. <span data-ttu-id="cdcc7-117">Перейдите к `/perftools/deployment/loganalytics` каталога.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-117">Navigate to the `/perftools/deployment/loganalytics` directory.</span></span>
1. <span data-ttu-id="cdcc7-118">Развертывание **logAnalyticsDeploy.json** шаблона Azure Resource Manager.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-118">Deploy the **logAnalyticsDeploy.json** Azure Resource Manager template.</span></span> <span data-ttu-id="cdcc7-119">Дополнительные сведения о развертывании шаблонов Resource Manager, см. в разделе [развертывание ресурсов с использованием шаблонов Resource Manager и Azure CLI][rm-cli].</span><span class="sxs-lookup"><span data-stu-id="cdcc7-119">For more information about deploying Resource Manager templates, see [Deploy resources with Resource Manager templates and Azure CLI][rm-cli].</span></span> <span data-ttu-id="cdcc7-120">Шаблон имеет следующие параметры:</span><span class="sxs-lookup"><span data-stu-id="cdcc7-120">The template has the following parameters:</span></span>

    * <span data-ttu-id="cdcc7-121">**location**: Регион, где развернуты рабочей области Log Analytics и панели мониторинга.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-121">**location**: The region where the Log Analytics workspace and dashboards are deployed.</span></span>
    * <span data-ttu-id="cdcc7-122">**serviceTier**: Заливки Ценовая категория рабочей области.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-122">**serviceTier**: Rhe workspace pricing tier.</span></span> <span data-ttu-id="cdcc7-123">См. в разделе [здесь] [ sku] список допустимых значений.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-123">See [here][sku] for a list of valid values.</span></span>
    * <span data-ttu-id="cdcc7-124">**dataRetention** (необязательно): Количество дней данные журнала сохраняются в рабочей области Log Analytics.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-124">**dataRetention** (optional): The number of days log data is retained in the Log Analytics workspace.</span></span> <span data-ttu-id="cdcc7-125">Значение по умолчанию — 30 дней.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-125">The default value is 30 days.</span></span> <span data-ttu-id="cdcc7-126">Если ценовая `Free`, срок хранения данных должен быть 7 дней.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-126">If the pricing tier is `Free`, the data retention must be 7 days.</span></span>
    * <span data-ttu-id="cdcc7-127">**WorkspaceName** (необязательно): Имя рабочей области.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-127">**workspaceName** (optional): A name for the workspace.</span></span> <span data-ttu-id="cdcc7-128">Если не указан, шаблон создает имя.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-128">If not specified, the template generates a name.</span></span>

    ```bash
    az group deployment create --resource-group <resource-group-name> --template-file logAnalyticsDeploy.json --parameters location='East US' serviceTier='Standalone'
    ```

<span data-ttu-id="cdcc7-129">Этот шаблон создает рабочую область, а также создает набор предопределенных запросов, используемые панелью мониторинга.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-129">This template creates the workspace and also creates a set of predefined queries that are used by by dashboard.</span></span>

## <a name="deploy-grafana-in-a-virtual-machine"></a><span data-ttu-id="cdcc7-130">Развертывание на виртуальной машине Grafana</span><span class="sxs-lookup"><span data-stu-id="cdcc7-130">Deploy Grafana in a virtual machine</span></span>

<span data-ttu-id="cdcc7-131">Grafana — это проект с открытым исходным кодом, которые можно развернуть для визуализации метрик ряда времени, хранящиеся в рабочей области Azure Log Analytics, с помощью Grafana подключаемый модуль для Azure Monitor.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-131">Grafana is an open source project you can deploy to visualize the time series metrics stored in your Azure Log Analytics workspace using the Grafana plugin for Azure Monitor.</span></span> <span data-ttu-id="cdcc7-132">Grafana выполняется на виртуальной машине (VM) и требует учетной записи хранения, виртуальная сеть и другие ресурсы.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-132">Grafana executes on a virtual machine (VM) and requires a storage account, virtual network, and other resources.</span></span> <span data-ttu-id="cdcc7-133">Для развертывания виртуальной машины с помощью bitnami сертифицированных Grafana изображения и связанные с ней ресурсы, выполните следующие действия:</span><span class="sxs-lookup"><span data-stu-id="cdcc7-133">To deploy a virtual machine with the bitnami certified Grafana image and associated resources, follow these steps:</span></span>

1. <span data-ttu-id="cdcc7-134">Используйте Azure CLI, чтобы принять условия образа Azure Marketplace для Grafana.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-134">Use the Azure CLI to accept the Azure Marketplace image terms for Grafana.</span></span>

    ```bash
    az vm image accept-terms --publisher bitnami --offer grafana --plan default
    ```

1. <span data-ttu-id="cdcc7-135">Перейдите к `/spark-monitoring/perftools/deployment/grafana` каталог в локальной копии репозитория GitHub.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-135">Navigate to the `/spark-monitoring/perftools/deployment/grafana` directory in your local copy of the GitHub repo.</span></span>
1. <span data-ttu-id="cdcc7-136">Развертывание **grafanaDeploy.json** шаблона Resource Manager следующим образом:</span><span class="sxs-lookup"><span data-stu-id="cdcc7-136">Deploy the **grafanaDeploy.json** Resource Manager template as follows:</span></span>

    ```bash
    export DATA_SOURCE="https://raw.githubusercontent.com/mspnp/spark-monitoring/master/perftools/deployment/grafana/AzureDataSource.sh"
    az group deployment create \
        --resource-group <resource-group-name> \
        --template-file grafanaDeploy.json \
        --parameters adminPass='<vm password>' dataSource=$DATA_SOURCE
    ```

<span data-ttu-id="cdcc7-137">После завершения развертывания образа bitnami Grafana устанавливается на виртуальной машине.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-137">Once the deployment is complete, the bitnami image of Grafana is installed on the virtual machine.</span></span>

## <a name="update-the-grafana-password"></a><span data-ttu-id="cdcc7-138">Обновить пароль Grafana</span><span class="sxs-lookup"><span data-stu-id="cdcc7-138">Update the Grafana password</span></span>

<span data-ttu-id="cdcc7-139">Как часть процесса установки, сценарий установки Grafana выводит временный пароль для **администратора** пользователя.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-139">As part of the setup process, the Grafana installation script outputs a temporary password for the **admin** user.</span></span> <span data-ttu-id="cdcc7-140">Требуется этот временный пароль для входа.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-140">You need this temporary password to sign in.</span></span> <span data-ttu-id="cdcc7-141">Чтобы получить временный пароль, выполните следующие действия.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-141">To obtain the temporary password, follow these steps:</span></span>  

1. <span data-ttu-id="cdcc7-142">Войдите на портал Azure.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-142">Log in to the Azure portal.</span></span>  
1. <span data-ttu-id="cdcc7-143">Выберите группу ресурсов, где были развернуты ресурсы.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-143">Select the resource group where the resources were deployed.</span></span>
1. <span data-ttu-id="cdcc7-144">Выберите виртуальную Машину, в котором была установлена Grafana.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-144">Select the VM where Grafana was installed.</span></span> <span data-ttu-id="cdcc7-145">Если вы использовали имя параметра по умолчанию в шаблоне развертывания, имя виртуальной Машины предшествовало **sparkmonitoring-vm-grafana**.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-145">If you used the default parameter name in the deployment template, the VM name is prefaced with **sparkmonitoring-vm-grafana**.</span></span>
1. <span data-ttu-id="cdcc7-146">В **поддержка и устранение неполадок** щелкните **диагностику загрузки** чтобы открыть страницу диагностики загрузки.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-146">In the **Support + troubleshooting** section, click **Boot diagnostics** to open the boot diagnostics page.</span></span>
1. <span data-ttu-id="cdcc7-147">Нажмите кнопку **журнал последовательного вывода** на странице диагностики загрузки.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-147">Click **Serial log** on the boot diagnostics page.</span></span>
1. <span data-ttu-id="cdcc7-148">Найдите следующую строку: «Задать пароль приложений Bitnami».</span><span class="sxs-lookup"><span data-stu-id="cdcc7-148">Search for the following string: "Setting Bitnami application password to".</span></span>
1. <span data-ttu-id="cdcc7-149">Скопируйте пароль в безопасном месте.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-149">Copy the password to a safe location.</span></span>

<span data-ttu-id="cdcc7-150">Затем измените пароль администратора Grafana, выполнив следующие действия:</span><span class="sxs-lookup"><span data-stu-id="cdcc7-150">Next, change the Grafana administrator password by following these steps:</span></span>

1. <span data-ttu-id="cdcc7-151">На портале Azure выберите виртуальную Машину и нажмите кнопку **Обзор**.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-151">In the Azure portal, select the VM and click **Overview**.</span></span>
1. <span data-ttu-id="cdcc7-152">Скопируйте общедоступный IP-адрес.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-152">Copy the public IP address.</span></span>
1. <span data-ttu-id="cdcc7-153">Откройте веб-браузер и перейдите к следующему URL-АДРЕСУ: `http://<IP addresss>:3000`.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-153">Open a web browser and navigate to the following URL: `http://<IP addresss>:3000`.</span></span>
1. <span data-ttu-id="cdcc7-154">На экране входа Grafana, введите **администратора** для имени пользователя и используйте пароль Grafana из предыдущих шагов.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-154">At the Grafana log in screen, enter **admin** for the user name, and use the Grafana password from the previous steps.</span></span>
1. <span data-ttu-id="cdcc7-155">После входа в систему, выберите **конфигурации** (значок шестеренки).</span><span class="sxs-lookup"><span data-stu-id="cdcc7-155">Once logged in, select **Configuration** (the gear icon).</span></span>
1. <span data-ttu-id="cdcc7-156">Выберите **администратора сервера**.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-156">Select **Server Admin**.</span></span>
1. <span data-ttu-id="cdcc7-157">На **пользователей** выберите **администратора** имени входа.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-157">On the **Users** tab, select the **admin** login.</span></span>
1. <span data-ttu-id="cdcc7-158">Обновите пароль.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-158">Update the password.</span></span>

## <a name="create-an-azure-monitor-data-source"></a><span data-ttu-id="cdcc7-159">Создание источника данных Azure Monitor</span><span class="sxs-lookup"><span data-stu-id="cdcc7-159">Create an Azure Monitor data source</span></span>

1. <span data-ttu-id="cdcc7-160">Создание службы субъекта, который позволяет Grafana для управления доступом к рабочей области Log Analytics.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-160">Create a service principal that allows Grafana to manage access to your Log Analytics workspace.</span></span> <span data-ttu-id="cdcc7-161">Дополнительные сведения см. в разделе [Создание субъекта-службы Azure с помощью Azure CLI](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest)</span><span class="sxs-lookup"><span data-stu-id="cdcc7-161">For more information, see [Create an Azure service principal with Azure CLI](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest)</span></span>

    ```bash
    az ad sp create-for-rbac --name http://<service principal name> --role "Log Analytics Reader"
    ```

1. <span data-ttu-id="cdcc7-162">Обратите внимание на значения appId, password и tenant в выходных данных этой команды:</span><span class="sxs-lookup"><span data-stu-id="cdcc7-162">Note the values for appId, password, and tenant in the output from this command:</span></span>

    ```json
    {
        "appId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "displayName": "azure-cli-2019-03-27-00-33-39",
        "name": "http://<service principal name>",
        "password": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "tenant": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    }
    ```

1. <span data-ttu-id="cdcc7-163">Войдите на Grafana, описанных ранее.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-163">Log into Grafana as described earlier.</span></span> <span data-ttu-id="cdcc7-164">Выберите **конфигурации** (значок шестеренки) и затем **источников данных**.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-164">Select **Configuration** (the gear icon) and then **Data Sources**.</span></span>
1. <span data-ttu-id="cdcc7-165">В **источников данных** щелкните **добавить источник данных**.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-165">In the **Data Sources** tab, click **Add data source**.</span></span>
1. <span data-ttu-id="cdcc7-166">Выберите **Azure Monitor** в качестве типа источника данных.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-166">Select **Azure Monitor** as the data source type.</span></span>
1. <span data-ttu-id="cdcc7-167">В **параметры** введите имя источника данных в **имя** текстового поля.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-167">In the **Settings** section, enter a name for the data source in the **Name** textbox.</span></span>
1. <span data-ttu-id="cdcc7-168">В **сведения об интерфейсе API Azure Monitor** введите следующие сведения:</span><span class="sxs-lookup"><span data-stu-id="cdcc7-168">In the **Azure Monitor API Details** section, enter the following information:</span></span>

    * <span data-ttu-id="cdcc7-169">Идентификатор подписки: идентификатор подписки Azure;</span><span class="sxs-lookup"><span data-stu-id="cdcc7-169">Subscription Id: Your Azure subscription ID.</span></span>
    * <span data-ttu-id="cdcc7-170">Идентификатор клиента: Идентификатор клиента из ранее.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-170">Tenant Id: The tenant ID from earlier.</span></span>
    * <span data-ttu-id="cdcc7-171">Идентификатор клиента: Значение «appId» выше.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-171">Client Id: The value of "appId" from earlier.</span></span>
    * <span data-ttu-id="cdcc7-172">Секрет клиента: Значение «password» выше.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-172">Client Secret: The value of "password" from earlier.</span></span>

1. <span data-ttu-id="cdcc7-173">В **сведения об интерфейсе API Azure Log Analytics** установите флажок **и те же сведения, как Azure Monitor API** флажок.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-173">In the **Azure Log Analytics API Details** section, check the **Same Details as Azure Monitor API** checkbox.</span></span>
1. <span data-ttu-id="cdcc7-174">Нажмите кнопку **сохранить и протестировать**.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-174">Click **Save & Test**.</span></span> <span data-ttu-id="cdcc7-175">Если источник данных Log Analytics настроена правильно, отображается сообщение об успешном выполнении.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-175">If the Log Analytics data source is correctly configured, a success message is displayed.</span></span>

## <a name="create-the-dashboard"></a><span data-ttu-id="cdcc7-176">Создание панели мониторинга</span><span class="sxs-lookup"><span data-stu-id="cdcc7-176">Create the dashboard</span></span>

<span data-ttu-id="cdcc7-177">Создание панелей мониторинга в Grafana, выполнив следующие действия:</span><span class="sxs-lookup"><span data-stu-id="cdcc7-177">Create the dashboards in Grafana by following these steps:</span></span>

1. <span data-ttu-id="cdcc7-178">Перейдите к `/perftools/dashboards/grafana` каталог в локальной копии репозитория GitHub.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-178">Navigate to the `/perftools/dashboards/grafana` directory in your local copy of the GitHub repo.</span></span>
1. <span data-ttu-id="cdcc7-179">Выполните следующий скрипт:</span><span class="sxs-lookup"><span data-stu-id="cdcc7-179">Run the following script:</span></span>

    ```bash
    export WORKSPACE=<your Azure Log Analytics workspace ID>
    export LOGTYPE=SparkListenerEvent_CL

    sh DashGen.sh
    ```

    <span data-ttu-id="cdcc7-180">Выходные данные этого сценария — это файл с именем **SparkMonitoringDash.json**.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-180">The output from the script is a file named **SparkMonitoringDash.json**.</span></span>

1. <span data-ttu-id="cdcc7-181">Вернитесь к панели мониторинга Grafana и выберите **создать** (значок плюса).</span><span class="sxs-lookup"><span data-stu-id="cdcc7-181">Return to the Grafana dashboard and select **Create** (the plus icon).</span></span>
1. <span data-ttu-id="cdcc7-182">Выберите **Импортировать**.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-182">Select **Import**.</span></span>
1. <span data-ttu-id="cdcc7-183">Нажмите кнопку **отправить JSON-файл**.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-183">Click **Upload .json File**.</span></span>
1. <span data-ttu-id="cdcc7-184">Выберите **SparkMonitoringDash.json** файл, созданный на шаге 2.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-184">Select the **SparkMonitoringDash.json** file created in step 2.</span></span>
1. <span data-ttu-id="cdcc7-185">В **параметры** раздела **АЛА**, выберите источник данных монитора Azure, созданной ранее.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-185">In the **Options** section, under **ALA**, select the Azure Monitor data source created earlier.</span></span>
1. <span data-ttu-id="cdcc7-186">Щелкните **Импорт**.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-186">Click **Import**.</span></span>

## <a name="visualizations-in-the-dashboards"></a><span data-ttu-id="cdcc7-187">Визуализации на панелях мониторинга</span><span class="sxs-lookup"><span data-stu-id="cdcc7-187">Visualizations in the dashboards</span></span>

<span data-ttu-id="cdcc7-188">Панели мониторинга Grafana и Azure Log Analytics содержат набор визуализации временных рядов.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-188">Both the Azure Log Analytics and Grafana dashboards include a set of time-series visualizations.</span></span> <span data-ttu-id="cdcc7-189">Каждый граф — график временных рядов данных метрики, связанные с Apache Spark [задания](https://spark.apache.org/docs/latest/job-scheduling.html), этапы задания и задачи, составляющие каждый этап.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-189">Each graph is time-series plot of metric data related to an Apache Spark [job](https://spark.apache.org/docs/latest/job-scheduling.html), stages of the job, and tasks that make up each stage.</span></span>

<span data-ttu-id="cdcc7-190">Визуализация — следующим образом:</span><span class="sxs-lookup"><span data-stu-id="cdcc7-190">The visualizations are as follows:</span></span>

### <a name="job-latency"></a><span data-ttu-id="cdcc7-191">Задание задержки</span><span class="sxs-lookup"><span data-stu-id="cdcc7-191">Job latency</span></span>

<span data-ttu-id="cdcc7-192">Эта визуализация показывает задержка выполнения для задания, которые грубое представление на общую производительность задания.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-192">This visualization shows execution latency for a job, which is a coarse view on the overall peformance of a job.</span></span> <span data-ttu-id="cdcc7-193">Отображает время выполнения задания от начала до конца.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-193">Displays the job execution duration from start to completion.</span></span> <span data-ttu-id="cdcc7-194">Обратите внимание, что время запуска задания не является таким же, как время отправки задания.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-194">Note that the job start time is not the same as the job submission time.</span></span> <span data-ttu-id="cdcc7-195">Задержка представляется в виде процентилей (10%, 30%, 50%, 90%) выполнения задания, индексированных по идентификатор кластера и идентификатор приложения.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-195">Latency is represented as percentiles (10%, 30%, 50%, 90%) of job execution indexed by cluster ID and application ID.</span></span>

### <a name="stage-latency"></a><span data-ttu-id="cdcc7-196">Задержка рабочей области</span><span class="sxs-lookup"><span data-stu-id="cdcc7-196">Stage latency</span></span>

<span data-ttu-id="cdcc7-197">Визуализация показывает задержку каждого этапа, на кластер, каждое приложение и каждого отдельного этапа.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-197">The visualization shows the latency of each stage per cluster, per application, and per individual stage.</span></span> <span data-ttu-id="cdcc7-198">Эта визуализация используется для идентификации определенного этапа, на котором выполняется медленно.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-198">This visualization is useful for identifying a particular stage that is running slowly.</span></span>

### <a name="task-latency"></a><span data-ttu-id="cdcc7-199">Задержка задачи</span><span class="sxs-lookup"><span data-stu-id="cdcc7-199">Task latency</span></span>

<span data-ttu-id="cdcc7-200">Эта визуализация показывает задержка выполнения задачи.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-200">This visualization shows task execution latency.</span></span> <span data-ttu-id="cdcc7-201">Задержка представляется как процент выполнения задачи для каждого кластера, имя рабочей области и приложения.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-201">Latency is represented as a percentile of task execution per cluster, stage name, and application.</span></span>

### <a name="sum-task-execution-per-host"></a><span data-ttu-id="cdcc7-202">Выполнение задач сумма для каждого узла</span><span class="sxs-lookup"><span data-stu-id="cdcc7-202">Sum Task Execution per host</span></span>

<span data-ttu-id="cdcc7-203">Эта визуализация показывает сумму задержка выполнения задачи для каждого узла в кластере.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-203">This visualization shows the sum of task execution latency per host running on a cluster.</span></span> <span data-ttu-id="cdcc7-204">Просмотр задержка выполнения задачи каждого узла определяет узлы, которые имеют гораздо более длительных задержек задач общую, чем другие узлы.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-204">Viewing task execution latency per host identifies hosts that have much higher overall task latency than other hosts.</span></span> <span data-ttu-id="cdcc7-205">Это может означать, что задачи были неэффективно или неравномерно распространены на узлы.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-205">This may mean that tasks have been inefficiently or unevenly distributed to hosts.</span></span>

### <a name="task-metrics"></a><span data-ttu-id="cdcc7-206">Метрики задач</span><span class="sxs-lookup"><span data-stu-id="cdcc7-206">Task metrics</span></span>

<span data-ttu-id="cdcc7-207">Эта визуализация отображается набор метрик выполнения для выполнения данной задачи.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-207">This visualization shows a set of the execution metrics for a given task's execution.</span></span> <span data-ttu-id="cdcc7-208">Это может быть размер и продолжительность смешения данных, продолжительность операций сериализации и десериализации и другие.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-208">These metrics include the size and duration of a data shuffle, duration of serialization and deserialization operations, and others.</span></span> <span data-ttu-id="cdcc7-209">Для полного набора метрик просмотрите запрос Log Analytics для панели.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-209">For the full set of metrics, view the Log Analytics query for the panel.</span></span> <span data-ttu-id="cdcc7-210">Эта визуализация полезно для понимания операции, которые составляют задачи и идентифицирующий потребление ресурсов каждой операции.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-210">This visualization is useful for understanding the operations that make up a task and identifying resource consumption of each operation.</span></span> <span data-ttu-id="cdcc7-211">Пики на графике представляют затратных операций, которые необходимо изучить.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-211">Spikes in the graph represent costly operations that should be investigated.</span></span>

### <a name="cluster-throughput"></a><span data-ttu-id="cdcc7-212">Пропускная способность кластера</span><span class="sxs-lookup"><span data-stu-id="cdcc7-212">Cluster throughput</span></span>

<span data-ttu-id="cdcc7-213">Эта визуализация — это общая схема рабочих элементов, индексированных по кластера и приложения для представления объем работы каждого кластера и приложения.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-213">This visualization is a high level view of work items indexed by cluster and application to represent the amount of work done per cluster and application.</span></span> <span data-ttu-id="cdcc7-214">Он показывает количество заданий, задач и этапов, выполнить для каждого кластера, приложения и рабочей области с шагом в одну минуту.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-214">It shows the number of jobs, tasks, and stages completed per cluster, application, and stage in one minute increments.</span></span> 

### <a name="streaming-throughputlatency"></a><span data-ttu-id="cdcc7-215">Потоковой передачи пропускной способности и задержки</span><span class="sxs-lookup"><span data-stu-id="cdcc7-215">Streaming Throughput/Latency</span></span>

<span data-ttu-id="cdcc7-216">Этот visualzation связана с метрики, связанные с запросом структурированный потоковой передачи.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-216">This visualzation is related to the metrics associated with a structured streaming query.</span></span> <span data-ttu-id="cdcc7-217">На графах представлены количество входных строк в секунду и количество строк, обрабатываемых за секунду.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-217">The graphs shows the number of input rows per second and the number of rows processed per second.</span></span> <span data-ttu-id="cdcc7-218">Метрики потоковой передачи также представлены каждого приложения.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-218">The streaming metrics are also represented per application.</span></span> <span data-ttu-id="cdcc7-219">Эти метрики отправляются при возникновении события OnQueryProgress структурированной потоковой передачи запрос обрабатывается, и представляет визуализацию в миллисекундах, потоковая передача задержка, как время, затраченное на выполнение в пакетном запросе.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-219">These metrics are sent when the OnQueryProgress event is generated as the structured streaming query is processed and the visualization represents streaming latency as the amount of time, in milliseconds, taken to execute a query batch.</span></span>

### <a name="resource-consumption-per-executor"></a><span data-ttu-id="cdcc7-220">Потребление ресурсов на каждый исполнитель</span><span class="sxs-lookup"><span data-stu-id="cdcc7-220">Resource consumption per executor</span></span>

<span data-ttu-id="cdcc7-221">Далее — это набор визуализации для отображения панели мониторинга определенного типа ресурса и его потребления на исполнителя для каждого кластера.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-221">Next is a set of visualizations for the dashboard show the particular type of resource and how it is consumed per executor on each cluster.</span></span> <span data-ttu-id="cdcc7-222">Эти представления помогают идентифицировать выбросы потребления ресурсов на каждый исполнитель.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-222">These visualizations help identify outliers in resource consumption per executor.</span></span> <span data-ttu-id="cdcc7-223">Например если распределение работы для конкретного исполнителя неравномерно, потребление ресурсов будут повышены относительно других исполнителей, запущенных в кластере.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-223">For example, if the work allocation for a particular executor is skewed, resource consumption will be elevated in relation to other executors running on the cluster.</span></span> <span data-ttu-id="cdcc7-224">Это можно определить, пики потребления ресурсов для исполнителя.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-224">This can be identified by spikes in the resource consumption for an executor.</span></span>

### <a name="executor-compute-time-metrics"></a><span data-ttu-id="cdcc7-225">Метрики времени вычислений исполнителя</span><span class="sxs-lookup"><span data-stu-id="cdcc7-225">Executor compute time metrics</span></span>

<span data-ttu-id="cdcc7-226">Далее — это набор визуализаций для панели мониторинга, Показать отношение исполнителя сериализации время десериализации время, время ЦП и общую исполнителя вычислительное время время виртуальной машины Java.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-226">Next is a set of visualizations for the dashboard that show the ratio of executor serialize time, deserialize time, CPU time, and Java virtual machine time to overall executor compute time.</span></span> <span data-ttu-id="cdcc7-227">Этот пример демонстрирует визуально каждого из этих четырех показателей вносят вклад в общую выполнения обработки.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-227">This demonstrates visually how much each of these four metrics are contributing to overall executor processing.</span></span>

### <a name="shuffle-metrics"></a><span data-ttu-id="cdcc7-228">Метрики в случайном порядке</span><span class="sxs-lookup"><span data-stu-id="cdcc7-228">Shuffle metrics</span></span>

<span data-ttu-id="cdcc7-229">Окончательный набор данных метрик, связанных с помощью структурированной потоковой передачи запроса между всеми исполнителями в случайном порядке Показать визуализации.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-229">The final set of visualizations show the data shuffle metrics associated with a structured streaming query across all executors.</span></span> <span data-ttu-id="cdcc7-230">К ним относятся случайном порядке байтов, чтение, байтов, записанных в случайном порядке, в случайном порядке памяти и использования диска в запросах использования файловой системы.</span><span class="sxs-lookup"><span data-stu-id="cdcc7-230">These include shuffle bytes read, shuffle bytes written, shuffle memory, and disk usage in queries where the file system is used.</span></span>

## <a name="next-steps"></a><span data-ttu-id="cdcc7-231">Дальнейшие действия</span><span class="sxs-lookup"><span data-stu-id="cdcc7-231">Next steps</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="cdcc7-232">Устранение узких мест производительности</span><span class="sxs-lookup"><span data-stu-id="cdcc7-232">Troubleshoot performance bottlenecks</span></span>](./performance-troubleshooting.md)

<!-- links -->

[rm-cli]: /azure/azure-resource-manager/resource-group-template-deploy-cli
[sku]: /azure/templates/Microsoft.OperationalInsights/2015-11-01-preview/workspaces#sku-object