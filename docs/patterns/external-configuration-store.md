---
title: Шаблон внешнего хранилища конфигураций
titleSuffix: Cloud Design Patterns
description: Переместите сведения о конфигурации из пакета развертывания приложения в централизованное расположение.
keywords: Конструктивный шаблон
author: dragon119
ms.date: 06/23/2017
ms.custom: seodec18
ms.openlocfilehash: 7e37e5bc052a9d8e8747a3a4ac3d79a311185ea4
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/04/2019
ms.locfileid: "54011316"
---
# <a name="external-configuration-store-pattern"></a><span data-ttu-id="3aa69-104">Шаблон внешнего хранилища конфигураций</span><span class="sxs-lookup"><span data-stu-id="3aa69-104">External Configuration Store pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="3aa69-105">Переместите сведения о конфигурации из пакета развертывания приложения в централизованное расположение.</span><span class="sxs-lookup"><span data-stu-id="3aa69-105">Move configuration information out of the application deployment package to a centralized location.</span></span> <span data-ttu-id="3aa69-106">Это может помочь упростить управление данными конфигурации, а также обмен данными конфигурации между приложениями и экземплярами приложений.</span><span class="sxs-lookup"><span data-stu-id="3aa69-106">This can provide opportunities for easier management and control of configuration data, and for sharing configuration data across applications and application instances.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="3aa69-107">Контекст и проблема</span><span class="sxs-lookup"><span data-stu-id="3aa69-107">Context and problem</span></span>

<span data-ttu-id="3aa69-108">В большинстве сред выполнения приложения содержатся сведения о конфигурации, которые хранятся в файлах, развернутых с этим приложением.</span><span class="sxs-lookup"><span data-stu-id="3aa69-108">The majority of application runtime environments include configuration information that's held in files deployed with the application.</span></span> <span data-ttu-id="3aa69-109">В некоторых случаях можно изменить эти файлы, чтобы повлиять на поведение приложения после его развертывания.</span><span class="sxs-lookup"><span data-stu-id="3aa69-109">In some cases, it's possible to edit these files to change the application behavior after it's been deployed.</span></span> <span data-ttu-id="3aa69-110">Однако изменения в конфигурации требуют повторного развертывания приложения, что часто приводит к неприемлемому простою и другим административным издержкам.</span><span class="sxs-lookup"><span data-stu-id="3aa69-110">However, changes to the configuration require the application be redeployed, often resulting in unacceptable downtime and other administrative overhead.</span></span>

<span data-ttu-id="3aa69-111">Локальные файлы конфигурации также ограничивают конфигурацию для одного приложения, но иногда могут быть полезными для совместного использования параметров конфигурации несколькими приложениями.</span><span class="sxs-lookup"><span data-stu-id="3aa69-111">Local configuration files also limit the configuration to a single application, but sometimes it would be useful to share configuration settings across multiple applications.</span></span> <span data-ttu-id="3aa69-112">В примерах содержатся строки подключения к базе данных, сведения о теме пользовательского интерфейса, а также URL-адреса очередей и хранилища, используемые связанным набором приложений.</span><span class="sxs-lookup"><span data-stu-id="3aa69-112">Examples include database connection strings, UI theme information, or the URLs of queues and storage used by a related set of applications.</span></span>

<span data-ttu-id="3aa69-113">Иногда сложно управлять изменениями для локальных конфигураций на нескольких запущенных экземплярах приложения, особенно при использовании сценария размещения в облаке.</span><span class="sxs-lookup"><span data-stu-id="3aa69-113">It's challenging to manage changes to local configurations across multiple running instances of the application, especially in a cloud-hosted scenario.</span></span> <span data-ttu-id="3aa69-114">Это может привести к тому, что экземпляры используют другие параметры конфигурации во время развертывания обновления.</span><span class="sxs-lookup"><span data-stu-id="3aa69-114">It can result in instances using different configuration settings while the update is being deployed.</span></span>

<span data-ttu-id="3aa69-115">Кроме того, для обновления приложений и компонентов может потребоваться внести изменения в схемы конфигурации.</span><span class="sxs-lookup"><span data-stu-id="3aa69-115">In addition, updates to applications and components might require changes to configuration schemas.</span></span> <span data-ttu-id="3aa69-116">Во многих системах конфигурации не поддерживаются разные версии сведений о конфигурации.</span><span class="sxs-lookup"><span data-stu-id="3aa69-116">Many configuration systems don't support different versions of configuration information.</span></span>

## <a name="solution"></a><span data-ttu-id="3aa69-117">Решение</span><span class="sxs-lookup"><span data-stu-id="3aa69-117">Solution</span></span>

<span data-ttu-id="3aa69-118">Храните сведения о конфигурации во внешнем хранилище и предоставьте интерфейс, который можно использовать для быстрого и эффективного чтения и обновления параметров конфигурации.</span><span class="sxs-lookup"><span data-stu-id="3aa69-118">Store the configuration information in external storage, and provide an interface that can be used to quickly and efficiently read and update configuration settings.</span></span> <span data-ttu-id="3aa69-119">Тип внешнего хранилища зависит от среды выполнения и размещения приложения.</span><span class="sxs-lookup"><span data-stu-id="3aa69-119">The type of external store depends on the hosting and runtime environment of the application.</span></span> <span data-ttu-id="3aa69-120">В сценарии размещения в облаке обычно используется служба облачного хранения, но может также применяться размещенная база данных или другая система.</span><span class="sxs-lookup"><span data-stu-id="3aa69-120">In a cloud-hosted scenario it's typically a cloud-based storage service, but could be a hosted database or other system.</span></span>

<span data-ttu-id="3aa69-121">В выбранном для информации о конфигурации резервном хранилище должен быть простой в использовании и последовательный интерфейс.</span><span class="sxs-lookup"><span data-stu-id="3aa69-121">The backing store you choose for configuration information should have an interface that provides consistent and easy-to-use access.</span></span> <span data-ttu-id="3aa69-122">Он должен предоставлять сведения в правильно типизированном и структурированном формате.</span><span class="sxs-lookup"><span data-stu-id="3aa69-122">It should expose the information in a correctly typed and structured format.</span></span> <span data-ttu-id="3aa69-123">Для реализации может также потребоваться разрешить доступ пользователям, чтобы защитить данные конфигурации и обеспечить достаточную гибкость, чтобы использовать хранилище с несколькими версиями конфигурации (например, среда разработки, промежуточная среда или рабочая среда, а также несколько версий выпуска каждой из них).</span><span class="sxs-lookup"><span data-stu-id="3aa69-123">The implementation might also need to authorize users’ access in order to protect configuration data, and be flexible enough to allow storage of multiple versions of the configuration (such as development, staging, or production, including multiple release versions of each one).</span></span>

> <span data-ttu-id="3aa69-124">Многие встроенные системы конфигурации считывают данные при запуске приложения и выполняют кэширование данных в память для быстрого доступа к ним и минимизации влияния на производительность приложения.</span><span class="sxs-lookup"><span data-stu-id="3aa69-124">Many built-in configuration systems read the data when the application starts up, and cache the data in memory to provide fast access and minimize the impact on application performance.</span></span> <span data-ttu-id="3aa69-125">В зависимости от типа используемого резервного хранилища и его задержки может быть полезно реализовать механизм кэширования во внешнем хранилище конфигураций.</span><span class="sxs-lookup"><span data-stu-id="3aa69-125">Depending on the type of backing store used, and the latency of this store, it might be helpful to implement a caching mechanism within the external configuration store.</span></span> <span data-ttu-id="3aa69-126">Дополнительные сведения см. в [руководстве по кэшированию](https://msdn.microsoft.com/library/dn589802.aspx).</span><span class="sxs-lookup"><span data-stu-id="3aa69-126">For more information, see the [Caching Guidance](https://msdn.microsoft.com/library/dn589802.aspx).</span></span> <span data-ttu-id="3aa69-127">На этом рисунке показаны общие сведения о шаблоне внешнего хранилища конфигураций с необязательным локальным кэшем.</span><span class="sxs-lookup"><span data-stu-id="3aa69-127">The figure illustrates an overview of the External Configuration Store pattern with optional local cache.</span></span>

![Общие сведения о шаблоне внешнего хранилища конфигураций с необязательным локальным кэшем](./_images/external-configuration-store-overview.png)

## <a name="issues-and-considerations"></a><span data-ttu-id="3aa69-129">Проблемы и рекомендации</span><span class="sxs-lookup"><span data-stu-id="3aa69-129">Issues and considerations</span></span>

<span data-ttu-id="3aa69-130">При принятии решения о реализации этого шаблона необходимо учитывать следующие моменты.</span><span class="sxs-lookup"><span data-stu-id="3aa69-130">Consider the following points when deciding how to implement this pattern:</span></span>

<span data-ttu-id="3aa69-131">Выберите резервное хранилище, обеспечивающее приемлемую производительность, высокий уровень доступности, надежность и для которого можно создать резервную копию при обслуживании и администрировании приложения.</span><span class="sxs-lookup"><span data-stu-id="3aa69-131">Choose a backing store that offers acceptable performance, high availability, robustness, and can be backed up as part of the application maintenance and administration process.</span></span> <span data-ttu-id="3aa69-132">Для удовлетворения этих требований в приложении, размещенном в облаке, обычно применяется механизм облачного хранилища.</span><span class="sxs-lookup"><span data-stu-id="3aa69-132">In a cloud-hosted application, using a cloud storage mechanism is usually a good choice to meet these requirements.</span></span>

<span data-ttu-id="3aa69-133">Разработайте схему резервного хранилища, чтобы обеспечивать гибкость типов хранимых данных.</span><span class="sxs-lookup"><span data-stu-id="3aa69-133">Design the schema of the backing store to allow flexibility in the types of information it can hold.</span></span> <span data-ttu-id="3aa69-134">Убедитесь, что предусмотрены все требования конфигурации (например, введенные данные, коллекции настроек, несколько версий параметров и любые другие компоненты, требуемые для использования приложения).</span><span class="sxs-lookup"><span data-stu-id="3aa69-134">Ensure that it provides for all configuration requirements such as typed data, collections of settings, multiple versions of settings, and any other features that the applications using it require.</span></span> <span data-ttu-id="3aa69-135">Схема должна легко расширяться для поддержки дополнительных параметров по мере изменения требований.</span><span class="sxs-lookup"><span data-stu-id="3aa69-135">The schema should be easy to extend to support additional settings as requirements change.</span></span>

<span data-ttu-id="3aa69-136">Учитывайте физические возможности резервного хранилища, как оно связано со способом хранения сведений о конфигурации, а также его влияние на производительность.</span><span class="sxs-lookup"><span data-stu-id="3aa69-136">Consider the physical capabilities of the backing store, how it relates to the way configuration information is stored, and the effects on performance.</span></span> <span data-ttu-id="3aa69-137">Например, для хранения XML-документа, в котором содержатся сведения о конфигурации, потребуется интерфейс конфигурации или приложение для синтаксического анализа документа, чтобы выполнить чтение отдельных параметров.</span><span class="sxs-lookup"><span data-stu-id="3aa69-137">For example, storing an XML document containing configuration information will require either the configuration interface or the application to parse the document in order to read individual settings.</span></span> <span data-ttu-id="3aa69-138">Это обновление параметра слегка усложнится, хотя кэширование параметров может помочь компенсировать медленную скорость чтения.</span><span class="sxs-lookup"><span data-stu-id="3aa69-138">It'll make updating a setting more complicated, though caching the settings can help to offset slower read performance.</span></span>

<span data-ttu-id="3aa69-139">Учтите то, каким образом интерфейс конфигурации будет разрешать управлять областью и наследованием настроек конфигурации.</span><span class="sxs-lookup"><span data-stu-id="3aa69-139">Consider how the configuration interface will permit control of the scope and inheritance of configuration settings.</span></span> <span data-ttu-id="3aa69-140">Например, требования к настройкам конфигурации области на уровне организации, приложения и компьютера.</span><span class="sxs-lookup"><span data-stu-id="3aa69-140">For example, it might be a requirement to scope configuration settings at the organization, application, and the machine level.</span></span> <span data-ttu-id="3aa69-141">Эти требования могут понадобиться, чтобы поддержать делегирование контроля над доступом к разным областям, а также чтобы запретить или разрешить отдельным приложениям переопределять параметры.</span><span class="sxs-lookup"><span data-stu-id="3aa69-141">It might need to support delegation of control over access to different scopes, and to prevent or allow individual applications to override settings.</span></span>

<span data-ttu-id="3aa69-142">Убедитесь, что интерфейс конфигурации может предоставлять данные конфигурации в требуемых форматах (например, типизированные значения, коллекции, пары "ключ — значение" или контейнеры свойств).</span><span class="sxs-lookup"><span data-stu-id="3aa69-142">Ensure that the configuration interface can expose the configuration data in the required formats such as typed values, collections, key/value pairs, or property bags.</span></span>

<span data-ttu-id="3aa69-143">Посмотрите, как будет работать интерфейс хранилища конфигурации, когда настройки содержат ошибки или не существуют в резервном хранилище.</span><span class="sxs-lookup"><span data-stu-id="3aa69-143">Consider how the configuration store interface will behave when settings contain errors, or don't exist in the backing store.</span></span> <span data-ttu-id="3aa69-144">Возможно, потребуется восстановить параметры по умолчанию и зарегистрировать ошибки.</span><span class="sxs-lookup"><span data-stu-id="3aa69-144">It might be appropriate to return default settings and log errors.</span></span> <span data-ttu-id="3aa69-145">Кроме того, обратите внимание на учет регистра в ключах или именах параметров конфигурации, хранение и обработку двоичных данных и способы обработки пустых или нулевых значений.</span><span class="sxs-lookup"><span data-stu-id="3aa69-145">Also consider aspects such as the case sensitivity of configuration setting keys or names, the storage and handling of binary data, and the ways that null or empty values are handled.</span></span>

<span data-ttu-id="3aa69-146">Рассмотрите возможность защиты данных конфигурации, чтобы предоставить доступ только соответствующим пользователям и приложениям.</span><span class="sxs-lookup"><span data-stu-id="3aa69-146">Consider how to protect the configuration data to allow access to only the appropriate users and applications.</span></span> <span data-ttu-id="3aa69-147">Это, скорее всего, возможность интерфейса хранилища конфигурации, но необходимо убедиться, что к данным в резервном хранилище невозможно получить доступ напрямую без соответствующего разрешения.</span><span class="sxs-lookup"><span data-stu-id="3aa69-147">This is likely a feature of the configuration store interface, but it's also necessary to ensure that the data in the backing store can't be accessed directly without the appropriate permission.</span></span> <span data-ttu-id="3aa69-148">Обеспечьте строгое разделение между разрешениями, необходимыми для чтения и записи данных конфигурации.</span><span class="sxs-lookup"><span data-stu-id="3aa69-148">Ensure strict separation between the permissions required to read and to write configuration data.</span></span> <span data-ttu-id="3aa69-149">Учтите также необходимость шифрования некоторых или всех параметров конфигурации и способ их реализации в интерфейсе хранилища конфигурации.</span><span class="sxs-lookup"><span data-stu-id="3aa69-149">Also consider whether you need to encrypt some or all of the configuration settings, and how this'll be implemented in the configuration store interface.</span></span>

<span data-ttu-id="3aa69-150">Централизованно хранимые конфигурации, которые влияют на поведение приложения при выполнении, являются критически важными, и их необходимо развернуть, обновить и управлять ими с помощью механизмов, используемых для развертывания кода приложения.</span><span class="sxs-lookup"><span data-stu-id="3aa69-150">Centrally stored configurations, which change application behavior during runtime, are critically important and should be deployed, updated, and managed using the same mechanisms as deploying application code.</span></span> <span data-ttu-id="3aa69-151">Например, изменения, которые могут повлиять на несколько приложений, необходимо выполнять с использованием подхода полного тестирования и промежуточного развертывания. Это позволит убедиться, что изменение подходит для всех приложений, использующих эту конфигурацию.</span><span class="sxs-lookup"><span data-stu-id="3aa69-151">For example, changes that can affect more than one application must be carried out using a full test and staged deployment approach to ensure that the change is appropriate for all applications that use this configuration.</span></span> <span data-ttu-id="3aa69-152">Если администратор изменяет параметр, чтобы обновить одно приложение, это может отрицательно повлиять на другие приложения, использующие тот же параметр.</span><span class="sxs-lookup"><span data-stu-id="3aa69-152">If an administrator edits a setting to update one application, it could adversely impact other applications that use the same setting.</span></span>

<span data-ttu-id="3aa69-153">Если приложение кэширует сведения о конфигурации, ему необходимо получать предупреждения об изменениях конфигурации.</span><span class="sxs-lookup"><span data-stu-id="3aa69-153">If an application caches configuration information, the application needs to be alerted if the configuration changes.</span></span> <span data-ttu-id="3aa69-154">Чтобы время от времени эти сведения автоматически обновлялись и были учтены все изменения (и обрабатывались), для кэшированных данных конфигурации можно реализовать политику срока действия.</span><span class="sxs-lookup"><span data-stu-id="3aa69-154">It might be possible to implement an expiration policy over cached configuration data so that this information is automatically refreshed periodically and any changes picked up (and acted on).</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="3aa69-155">Когда следует использовать этот шаблон</span><span class="sxs-lookup"><span data-stu-id="3aa69-155">When to use this pattern</span></span>

<span data-ttu-id="3aa69-156">Этот шаблон можно использовать для следующих целей:</span><span class="sxs-lookup"><span data-stu-id="3aa69-156">This pattern is useful for:</span></span>

- <span data-ttu-id="3aa69-157">если параметры конфигурации являются общими для нескольких приложений и экземпляров приложения, или если для нескольких приложений и экземпляров приложений применяется стандартная конфигурация;</span><span class="sxs-lookup"><span data-stu-id="3aa69-157">Configuration settings that are shared between multiple applications and application instances, or where a standard configuration must be enforced across multiple applications and application instances.</span></span>

- <span data-ttu-id="3aa69-158">если стандартная система конфигураций не поддерживает все необходимые параметры конфигурации (например, сохранение образов или сложных типов данных);</span><span class="sxs-lookup"><span data-stu-id="3aa69-158">A standard configuration system that doesn't support all of the required configuration settings, such as storing images or complex data types.</span></span>

- <span data-ttu-id="3aa69-159">если дополнительное хранилище для некоторых параметров приложений позволяет приложениям переопределять некоторые или все централизованно хранимые параметры;</span><span class="sxs-lookup"><span data-stu-id="3aa69-159">As a complementary store for some of the settings for applications, perhaps allowing applications to override some or all of the centrally-stored settings.</span></span>

- <span data-ttu-id="3aa69-160">если нужно упростить администрирование нескольких приложений и при необходимости проводить мониторинг использования параметров конфигурации путем ведения журнала для некоторых или всех типов доступа к хранилищу конфигурации.</span><span class="sxs-lookup"><span data-stu-id="3aa69-160">As a way to simplify administration of multiple applications, and optionally for monitoring use of configuration settings by logging some or all types of access to the configuration store.</span></span>

## <a name="example"></a><span data-ttu-id="3aa69-161">Пример</span><span class="sxs-lookup"><span data-stu-id="3aa69-161">Example</span></span>

<span data-ttu-id="3aa69-162">Для внешнего хранения информации размещаемого приложения Microsoft Azure обычно используется служба хранилища Azure.</span><span class="sxs-lookup"><span data-stu-id="3aa69-162">In a Microsoft Azure hosted application, a typical choice for storing configuration information externally is to use Azure Storage.</span></span> <span data-ttu-id="3aa69-163">Эта служба устойчива, обеспечивает высокую производительность, предоставляет высокий уровень доступности и реплицируется три раза с помощью автоматической отработки отказа.</span><span class="sxs-lookup"><span data-stu-id="3aa69-163">This is resilient, offers high performance, and is replicated three times with automatic failover to offer high availability.</span></span> <span data-ttu-id="3aa69-164">Хранилище таблиц Azure предоставляет хранилище ключей и значений с возможностью использования гибкой схемы для значений.</span><span class="sxs-lookup"><span data-stu-id="3aa69-164">Azure Table storage provides a key/value store with the ability to use a flexible schema for the values.</span></span> <span data-ttu-id="3aa69-165">Хранилище BLOB-объектов Azure предоставляет иерархическое, основанное на контейнере хранилище, которое может содержать любой тип данных в отдельно именованных больших двоичных объектах.</span><span class="sxs-lookup"><span data-stu-id="3aa69-165">Azure Blob storage provides a hierarchical, container-based store that can hold any type of data in individually named blobs.</span></span>

<span data-ttu-id="3aa69-166">В следующем примере показано, как хранилище конфигурации можно реализовать в хранилище BLOB-объектов для хранения и предоставления сведений о конфигурации.</span><span class="sxs-lookup"><span data-stu-id="3aa69-166">The following example shows how a configuration store can be implemented over Blob storage to store and expose configuration information.</span></span> <span data-ttu-id="3aa69-167">Класс `BlobSettingsStore` абстрагирует хранилище BLOB-объектов для хранения сведений о конфигурации, а также реализует интерфейс `ISettingsStore`, показанный в следующем коде.</span><span class="sxs-lookup"><span data-stu-id="3aa69-167">The `BlobSettingsStore` class abstracts Blob storage for holding configuration information, and implements the `ISettingsStore` interface shown in the following code.</span></span>

> <span data-ttu-id="3aa69-168">Этот код предоставляется в проекте _ExternalConfigurationStore.Cloud_ в решении _ExternalConfigurationStore_, доступном на портале [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store).</span><span class="sxs-lookup"><span data-stu-id="3aa69-168">This code is provided in the _ExternalConfigurationStore.Cloud_ project in the _ExternalConfigurationStore_ solution, available from [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store).</span></span>

```csharp
public interface ISettingsStore
{
    Task<string> GetVersionAsync();

    Task<Dictionary<string, string>> FindAllAsync();
}
```

<span data-ttu-id="3aa69-169">Этот интерфейс определяет методы для получения и обновления параметров конфигурации, которые содержатся в хранилище конфигураций, и включает в себя номер версии, который можно использовать, чтобы определить, изменялись ли в последнее время параметры конфигурации.</span><span class="sxs-lookup"><span data-stu-id="3aa69-169">This interface defines methods for retrieving and updating configuration settings held in the configuration store, and includes a version number that can be used to detect whether any configuration settings have been modified recently.</span></span> <span data-ttu-id="3aa69-170">Класс `BlobSettingsStore` использует свойство `ETag` большого двоичного объекта для реализации управления версиями.</span><span class="sxs-lookup"><span data-stu-id="3aa69-170">The `BlobSettingsStore` class uses the `ETag` property of the blob to implement versioning.</span></span> <span data-ttu-id="3aa69-171">Свойство `ETag` обновляется автоматически при каждой записи большого двоичного объекта.</span><span class="sxs-lookup"><span data-stu-id="3aa69-171">The `ETag` property is updated automatically each time the blob is written.</span></span>

> <span data-ttu-id="3aa69-172">Исходя из структуры, это простое решение предоставляет все параметры конфигурации в виде строковых, а не типизированных значений.</span><span class="sxs-lookup"><span data-stu-id="3aa69-172">By design, this simple solution exposes all configuration settings as string values rather than typed values.</span></span>

<span data-ttu-id="3aa69-173">Класс `ExternalConfigurationManager` предоставляет программу-оболочку для объекта `BlobSettingsStore`.</span><span class="sxs-lookup"><span data-stu-id="3aa69-173">The `ExternalConfigurationManager` class provides a wrapper around a `BlobSettingsStore` object.</span></span> <span data-ttu-id="3aa69-174">Приложение может использовать этот класс для хранения и извлечения сведений о конфигурации.</span><span class="sxs-lookup"><span data-stu-id="3aa69-174">An application can use this class to store and retrieve configuration information.</span></span> <span data-ttu-id="3aa69-175">Этот класс использует библиотеку [реактивных расширений](https://msdn.microsoft.com/library/hh242985.aspx) Майкрософт, чтобы предоставить любые изменения, внесенные в конфигурацию, через реализацию интерфейса `IObservable`.</span><span class="sxs-lookup"><span data-stu-id="3aa69-175">This class uses the Microsoft [Reactive Extensions](https://msdn.microsoft.com/library/hh242985.aspx) library to expose any changes made to the configuration through an implementation of the `IObservable` interface.</span></span> <span data-ttu-id="3aa69-176">Если параметр изменяется путем вызова метода `SetAppSetting`, об этом будут уведомлены событие `Changed` и все подписчики этого события.</span><span class="sxs-lookup"><span data-stu-id="3aa69-176">If a setting is modified by calling the `SetAppSetting` method, the `Changed` event is raised and all subscribers to this event will be notified.</span></span>

<span data-ttu-id="3aa69-177">Обратите внимание, что для быстрого получения доступа все параметры также кэшируются в объекте `Dictionary` в классе `ExternalConfigurationManager`.</span><span class="sxs-lookup"><span data-stu-id="3aa69-177">Note that all settings are also cached in a `Dictionary` object inside the `ExternalConfigurationManager` class for fast access.</span></span> <span data-ttu-id="3aa69-178">Метод `GetSetting`, используемый для получения параметров конфигурации, считывает данные из кэша.</span><span class="sxs-lookup"><span data-stu-id="3aa69-178">The `GetSetting` method used to retrieve a configuration setting reads the data from the cache.</span></span> <span data-ttu-id="3aa69-179">Если параметр не найден в кэше, он извлекается из объекта `BlobSettingsStore`.</span><span class="sxs-lookup"><span data-stu-id="3aa69-179">If the setting isn't found in the cache, it's retrieved from the `BlobSettingsStore` object instead.</span></span>

<span data-ttu-id="3aa69-180">Метод `GetSettings` вызывает метод `CheckForConfigurationChanges`, чтобы узнать, изменились ли сведения о конфигурации в хранилище BLOB-объектов.</span><span class="sxs-lookup"><span data-stu-id="3aa69-180">The `GetSettings` method invokes the `CheckForConfigurationChanges` method to detect whether the configuration information in blob storage has changed.</span></span> <span data-ttu-id="3aa69-181">Это делается путем проверки и сравнения номера версии с текущим номером версии, хранимым в объекте `ExternalConfigurationManager`.</span><span class="sxs-lookup"><span data-stu-id="3aa69-181">It does this by examining the version number and comparing it with the current version number held by the `ExternalConfigurationManager` object.</span></span> <span data-ttu-id="3aa69-182">Если имеется одно или несколько изменений, возникает событие `Changed` и обновляются параметры конфигурации, кэшированные в объекте `Dictionary`.</span><span class="sxs-lookup"><span data-stu-id="3aa69-182">If one or more changes have occurred, the `Changed` event is raised and the configuration settings cached in the `Dictionary` object are refreshed.</span></span> <span data-ttu-id="3aa69-183">Это приложение шаблона [Кэш на стороне](./cache-aside.md).</span><span class="sxs-lookup"><span data-stu-id="3aa69-183">This is an application of the [Cache-Aside pattern](./cache-aside.md).</span></span>

<span data-ttu-id="3aa69-184">В следующем примере кода показано, как реализуются событие `Changed`, метод `GetSettings` и `CheckForConfigurationChanges`.</span><span class="sxs-lookup"><span data-stu-id="3aa69-184">The following code sample shows how the `Changed` event, the `GetSettings` method, and the `CheckForConfigurationChanges` method are implemented:</span></span>

```csharp
public class ExternalConfigurationManager : IDisposable
{
  // An abstraction of the configuration store.
  private readonly ISettingsStore settings;
  private readonly ISubject<KeyValuePair<string, string>> changed;
  ...
  private readonly ReaderWriterLockSlim settingsCacheLock = new ReaderWriterLockSlim();
  private readonly SemaphoreSlim syncCacheSemaphore = new SemaphoreSlim(1);  
  ...
  private Dictionary<string, string> settingsCache;
  private string currentVersion;
  ...
  public ExternalConfigurationManager(ISettingsStore settings, ...)
  {
    this.settings = settings;
    ...
  }
  ...
  public IObservable<KeyValuePair<string, string>> Changed => this.changed.AsObservable();
  ...

  public string GetAppSetting(string key)
  {
    ...
    // Try to get the value from the settings cache.
    // If there's a cache miss, get the setting from the settings store and refresh the settings cache.

    string value;
    try
    {
        this.settingsCacheLock.EnterReadLock();

        this.settingsCache.TryGetValue(key, out value);
    }
    finally
    {
        this.settingsCacheLock.ExitReadLock();
    }

    return value;
  }
  ...
  private void CheckForConfigurationChanges()
  {
    try
    {
        // It is assumed that updates are infrequent.
        // To avoid race conditions in refreshing the cache, synchronize access to the in-memory cache.
        await this.syncCacheSemaphore.WaitAsync();

        var latestVersion = await this.settings.GetVersionAsync();

        // If the versions are the same, nothing has changed in the configuration.
        if (this.currentVersion == latestVersion) return;

        // Get the latest settings from the settings store and publish changes.
        var latestSettings = await this.settings.FindAllAsync();

        // Refresh the settings cache.
        try
        {
            this.settingsCacheLock.EnterWriteLock();

            if (this.settingsCache != null)
            {
                //Notify settings changed
                latestSettings.Except(this.settingsCache).ToList().ForEach(kv => this.changed.OnNext(kv));
            }
            this.settingsCache = latestSettings;
        }
        finally
        {
            this.settingsCacheLock.ExitWriteLock();
        }

        // Update the current version.
        this.currentVersion = latestVersion;
    }
    catch (Exception ex)
    {
        this.changed.OnError(ex);
    }
    finally
    {
        this.syncCacheSemaphore.Release();
    }
  }
}
```

> <span data-ttu-id="3aa69-185">Класс `ExternalConfigurationManager` также предоставляет свойство с именем `Environment`.</span><span class="sxs-lookup"><span data-stu-id="3aa69-185">The `ExternalConfigurationManager` class also provides a property named `Environment`.</span></span> <span data-ttu-id="3aa69-186">Это свойство поддерживает различные конфигурации для приложения, работающего в разных средах (например, в промежуточной и рабочей).</span><span class="sxs-lookup"><span data-stu-id="3aa69-186">This property supports varying configurations for an application running in different environments, such as staging and production.</span></span>

<span data-ttu-id="3aa69-187">Объект `ExternalConfigurationManager` может также периодически выполнять запрос по изменениям объекта `BlobSettingsStore`.</span><span class="sxs-lookup"><span data-stu-id="3aa69-187">An `ExternalConfigurationManager` object can also query the `BlobSettingsStore` object periodically for any changes.</span></span> <span data-ttu-id="3aa69-188">В следующем коде метод `StartMonitor` вызывает `CheckForConfigurationChanges` с интервалом, чтобы обнаружить все изменения, и инициирует событие `Changed`, как описано выше.</span><span class="sxs-lookup"><span data-stu-id="3aa69-188">In the following code, the `StartMonitor` method calls `CheckForConfigurationChanges` at an interval to detect any changes and raise the `Changed` event, as described earlier.</span></span>

```csharp
public class ExternalConfigurationManager : IDisposable
{
  ...
  private readonly ISubject<KeyValuePair<string, string>> changed;
  private Dictionary<string, string> settingsCache;
  private readonly CancellationTokenSource cts = new CancellationTokenSource();
  private Task monitoringTask;
  private readonly TimeSpan interval;

  private readonly SemaphoreSlim timerSemaphore = new SemaphoreSlim(1);
  ...
  public ExternalConfigurationManager(string environment) : this(new BlobSettingsStore(environment), TimeSpan.FromSeconds(15), environment)
  {
  }
  
  public ExternalConfigurationManager(ISettingsStore settings, TimeSpan interval, string environment)
  {
      this.settings = settings;
      this.interval = interval;
      this.CheckForConfigurationChangesAsync().Wait();
      this.changed = new Subject<KeyValuePair<string, string>>();
      this.Environment = environment;
  }
  ...
  /// <summary>
  /// Check to see if the current instance is monitoring for changes
  /// </summary>
  public bool IsMonitoring => this.monitoringTask != null && !this.monitoringTask.IsCompleted;

  /// <summary>
  /// Start the background monitoring for configuration changes in the central store
  /// </summary>
  public void StartMonitor()
  {
      if (this.IsMonitoring)
          return;

      try
      {
          this.timerSemaphore.Wait();

          // Check again to make sure we are not already running.
          if (this.IsMonitoring)
              return;

          // Start running our task loop.
          this.monitoringTask = ConfigChangeMonitor();
      }
      finally
      {
          this.timerSemaphore.Release();
      }
  }

  /// <summary>
  /// Loop that monitors for configuration changes
  /// </summary>
  /// <returns></returns>
  public async Task ConfigChangeMonitor()
  {
      while (!cts.Token.IsCancellationRequested)
      {
          await this.CheckForConfigurationChangesAsync();
          await Task.Delay(this.interval, cts.Token);
      }
  }

  /// <summary>
  /// Stop monitoring for configuration changes
  /// </summary>
  public void StopMonitor()
  {
      try
      {
          this.timerSemaphore.Wait();

          // Signal the task to stop.
          this.cts.Cancel();

          // Wait for the loop to stop.
          this.monitoringTask.Wait();

          this.monitoringTask = null;
      }
      finally
      {
          this.timerSemaphore.Release();
      }
  }

  public void Dispose()
  {
      this.cts.Cancel();
  }
  ...
}
```

<span data-ttu-id="3aa69-189">Класс `ExternalConfigurationManager` создан в качестве одноэлементного экземпляра с помощью класса `ExternalConfiguration`, показанного ниже.</span><span class="sxs-lookup"><span data-stu-id="3aa69-189">The `ExternalConfigurationManager` class is instantiated as a singleton instance by the `ExternalConfiguration` class shown below.</span></span>

```csharp
public static class ExternalConfiguration
{
    private static readonly Lazy<ExternalConfigurationManager> configuredInstance = new Lazy<ExternalConfigurationManager>(
        () =>
        {
            var environment = CloudConfigurationManager.GetSetting("environment");
            return new ExternalConfigurationManager(environment);
        });

    public static ExternalConfigurationManager Instance => configuredInstance.Value;
}
```

<span data-ttu-id="3aa69-190">Приведенный ниже код взят из класса `WorkerRole` в проекте _ExternalConfigurationStore.Cloud_.</span><span class="sxs-lookup"><span data-stu-id="3aa69-190">The following code is taken from the `WorkerRole` class in the _ExternalConfigurationStore.Cloud_ project.</span></span> <span data-ttu-id="3aa69-191">В нем показано, как приложение использует класс `ExternalConfiguration` для чтения параметра.</span><span class="sxs-lookup"><span data-stu-id="3aa69-191">It shows how the application uses the `ExternalConfiguration` class to read a setting.</span></span>

```csharp
public override void Run()
{
  // Start monitoring configuration changes.
  ExternalConfiguration.Instance.StartMonitor();

  // Get a setting.
  var setting = ExternalConfiguration.Instance.GetAppSetting("setting1");
  Trace.TraceInformation("Worker Role: Get setting1, value: " + setting);

  this.completeEvent.WaitOne();
}
```

<span data-ttu-id="3aa69-192">Следующий код также взят из класса `WorkerRole`, и в нем показывается, как приложение подписывается на события конфигурации.</span><span class="sxs-lookup"><span data-stu-id="3aa69-192">The following code, also from the `WorkerRole` class, shows how the application subscribes to configuration events.</span></span>

```csharp
public override bool OnStart()
{
  ...
  // Subscribe to the event.
  ExternalConfiguration.Instance.Changed.Subscribe(
     m => Trace.TraceInformation("Configuration has changed. Key:{0} Value:{1}",
          m.Key, m.Value),
     ex => Trace.TraceError("Error detected: " + ex.Message));
  ...
}
```

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="3aa69-193">Связанные шаблоны и рекомендации</span><span class="sxs-lookup"><span data-stu-id="3aa69-193">Related patterns and guidance</span></span>

- <span data-ttu-id="3aa69-194">Пример, демонстрирующий этот шаблон, можно найти на сайте [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store).</span><span class="sxs-lookup"><span data-stu-id="3aa69-194">A sample that demonstrates this pattern is available on [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store).</span></span>
