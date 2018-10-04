---
title: Размещение статического содержимого
description: Разверните статическое содержимое в облачной службе хранения для предоставления непосредственно клиенту.
keywords: Конструктивный шаблон
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- design-implementation
- performance-scalability
ms.openlocfilehash: 450d0c4c08098c1ba48e4c0dac3d058a46e3709b
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/28/2018
ms.locfileid: "47428217"
---
# <a name="static-content-hosting-pattern"></a><span data-ttu-id="91310-104">Шаблон размещения статического содержимого</span><span class="sxs-lookup"><span data-stu-id="91310-104">Static Content Hosting pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="91310-105">Развертывайте статическое содержимое в облачной службе хранения, чтобы предоставлять его непосредственно клиенту.</span><span class="sxs-lookup"><span data-stu-id="91310-105">Deploy static content to a cloud-based storage service that can deliver them directly to the client.</span></span> <span data-ttu-id="91310-106">Это позволяет снизить потребность в использовании дорогостоящих вычислительных экземпляров.</span><span class="sxs-lookup"><span data-stu-id="91310-106">This can reduce the need for potentially expensive compute instances.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="91310-107">Контекст и проблема</span><span class="sxs-lookup"><span data-stu-id="91310-107">Context and problem</span></span>

<span data-ttu-id="91310-108">Как правило, веб-приложения содержат элементы статического содержимого.</span><span class="sxs-lookup"><span data-stu-id="91310-108">Web applications typically include some elements of static content.</span></span> <span data-ttu-id="91310-109">Это статическое содержимое может включать HTML-страницы и другие ресурсы, такие как изображения и документы, доступные для клиента либо в составе HTML-страниц (например, встроенные изображения, таблицы стилей и клиентские файлы JavaScript), либо как отдельно загружаемые страницы (например, PDF-документы).</span><span class="sxs-lookup"><span data-stu-id="91310-109">This static content might include HTML pages and other resources such as images and documents that are available to the client, either as part of an HTML page (such as inline images, style sheets, and client-side JavaScript files) or as separate downloads (such as PDF documents).</span></span>

<span data-ttu-id="91310-110">Хотя веб-серверы отлично настроены для оптимизации запросов путем эффективного динамического выполнения кода страницы и кэширования выводимых данных, они по-прежнему должны обрабатывать запросы на скачивание статического содержимого.</span><span class="sxs-lookup"><span data-stu-id="91310-110">Although web servers are well tuned to optimize requests through efficient dynamic page code execution and output caching, they still have to handle requests to download static content.</span></span> <span data-ttu-id="91310-111">Для этого задействуются циклы обработки, которые могли бы использоваться для других задач.</span><span class="sxs-lookup"><span data-stu-id="91310-111">This consumes processing cycles that could often be put to better use.</span></span>

## <a name="solution"></a><span data-ttu-id="91310-112">Решение</span><span class="sxs-lookup"><span data-stu-id="91310-112">Solution</span></span>

<span data-ttu-id="91310-113">В большинстве сред для облачного размещения можно уменьшить потребность в вычислительных экземплярах (например, использовать экземпляры меньшего размера или меньшее число экземпляров), разместив некоторые ресурсы и статические страницы приложения в службе хранения.</span><span class="sxs-lookup"><span data-stu-id="91310-113">In most cloud hosting environments it's possible to minimize the need for compute instances (for example, use a smaller instance or fewer instances), by locating some of an application’s resources and static pages in a storage service.</span></span> <span data-ttu-id="91310-114">Как правило, облачные хранилища намного дешевле, чем вычислительные экземпляры.</span><span class="sxs-lookup"><span data-stu-id="91310-114">The cost for cloud-hosted storage is typically much less than for compute instances.</span></span>

<span data-ttu-id="91310-115">Основные проблемы при размещении некоторых частей приложения в службе хранения связаны c развертыванием приложения и защитой ресурсов, которые не должны быть доступны для анонимных пользователей.</span><span class="sxs-lookup"><span data-stu-id="91310-115">When hosting some parts of an application in a storage service, the main considerations are related to deployment of the application and to securing resources that aren't intended to be available to anonymous users.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="91310-116">Проблемы и рекомендации</span><span class="sxs-lookup"><span data-stu-id="91310-116">Issues and considerations</span></span>

<span data-ttu-id="91310-117">При принятии решения о реализации этого шаблона необходимо учитывать следующие моменты.</span><span class="sxs-lookup"><span data-stu-id="91310-117">Consider the following points when deciding how to implement this pattern:</span></span>

- <span data-ttu-id="91310-118">Размещенная служба хранения должна предоставлять конечную точку HTTP, к которой пользователи смогут получать доступ для скачивания статических ресурсов.</span><span class="sxs-lookup"><span data-stu-id="91310-118">The hosted storage service must expose an HTTP endpoint that users can access to download the static resources.</span></span> <span data-ttu-id="91310-119">Некоторые службы хранения также поддерживают протокол HTTPS, что позволяет размещать в службах хранения ресурсы, для которых требуется протокол SSL.</span><span class="sxs-lookup"><span data-stu-id="91310-119">Some storage services also support HTTPS, so it's possible to host resources in storage services that require SSL.</span></span>

- <span data-ttu-id="91310-120">Если требуется обеспечить максимальную производительность и доступность, рекомендуем использовать сеть доставки содержимого (CDN) для кэширования содержимого контейнера хранилища в нескольких центрах обработки данных по всему миру.</span><span class="sxs-lookup"><span data-stu-id="91310-120">For maximum performance and availability, consider using a content delivery network (CDN) to cache the contents of the storage container in multiple datacenters around the world.</span></span> <span data-ttu-id="91310-121">Но за использование CDN может взиматься плата.</span><span class="sxs-lookup"><span data-stu-id="91310-121">However, you'll likely have to pay for using the CDN.</span></span>

- <span data-ttu-id="91310-122">Как правило, учетные записи хранения являются геореплицируемыми по умолчанию. Это позволяет обеспечить устойчивость при возникновении событий, которые могут повлиять на работу центра обработки данных.</span><span class="sxs-lookup"><span data-stu-id="91310-122">Storage accounts are often geo-replicated by default to provide resiliency against events that might affect a datacenter.</span></span> <span data-ttu-id="91310-123">Это означает, что IP-адрес может измениться, но URL-адрес останется тем же.</span><span class="sxs-lookup"><span data-stu-id="91310-123">This means that the IP address might change, but the URL will remain the same.</span></span>

- <span data-ttu-id="91310-124">Когда одна часть содержимого находится в учетной записи хранения, а другая — в размещенном вычислительном экземпляре, развертывание приложения и его обновление усложняется.</span><span class="sxs-lookup"><span data-stu-id="91310-124">When some content is located in a storage account and other content is in a hosted compute instance it becomes more challenging to deploy an application and to update it.</span></span> <span data-ttu-id="91310-125">Возможно, потребуется несколько отдельных развертываний, а также управление версиями приложения и содержимого. Это позволит упростить управление, особенно если статическое содержимое содержит файлы скриптов или компоненты пользовательского интерфейса.</span><span class="sxs-lookup"><span data-stu-id="91310-125">You might have to perform separate deployments, and version the application and content to manage it more easily&mdash;especially when the static content includes script files or UI components.</span></span> <span data-ttu-id="91310-126">Но если требуется обновить только статические ресурсы, их можно просто отправить в учетную запись хранения, не развертывая пакет приложения повторно.</span><span class="sxs-lookup"><span data-stu-id="91310-126">However, if only static resources have to be updated, they can simply be uploaded to the storage account without needing to redeploy the application package.</span></span>

- <span data-ttu-id="91310-127">Службы хранения могут не поддерживать пользовательские доменные имена.</span><span class="sxs-lookup"><span data-stu-id="91310-127">Storage services might not support the use of custom domain names.</span></span> <span data-ttu-id="91310-128">В этом случае необходимо указать полные URL-адреса ресурсов в виде ссылок, так как они будут находиться в домене, отличном от домена для динамического содержимого, содержащего ссылки.</span><span class="sxs-lookup"><span data-stu-id="91310-128">In this case it's necessary to specify the full URL of the resources in links because they'll be in a different domain from the dynamically-generated content containing the links.</span></span>

- <span data-ttu-id="91310-129">Контейнеры хранилища должны быть настроены для открытого доступа на чтение. Но обязательно убедитесь, что они не настроены для открытого доступа на запись, чтобы пользователи не могли отправлять содержимое.</span><span class="sxs-lookup"><span data-stu-id="91310-129">The storage containers must be configured for public read access, but it's vital to ensure that they aren't configured for public write access to prevent users being able to upload content.</span></span> <span data-ttu-id="91310-130">Для управления доступом к ресурсам, которые не должны быть доступны анонимно, рекомендуется использовать ключ камердинера или токен &mdash; см. статью о [шаблоне ключа камердинера](valet-key.md).</span><span class="sxs-lookup"><span data-stu-id="91310-130">Consider using a valet key or token to control access to resources that shouldn't be available anonymously&mdash;see the [Valet Key pattern](valet-key.md) for more information.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="91310-131">Когда следует использовать этот шаблон</span><span class="sxs-lookup"><span data-stu-id="91310-131">When to use this pattern</span></span>

<span data-ttu-id="91310-132">Этот шаблон можно использовать для следующих целей:</span><span class="sxs-lookup"><span data-stu-id="91310-132">This pattern is useful for:</span></span>

- <span data-ttu-id="91310-133">Сокращение затрат на размещение веб-сайтов и приложений, которые содержат некоторые статические ресурсы.</span><span class="sxs-lookup"><span data-stu-id="91310-133">Minimizing the hosting cost for websites and applications that contain some static resources.</span></span>

- <span data-ttu-id="91310-134">Сокращение затрат на размещение веб-сайтов, которые содержат только статическое содержимое и ресурсы.</span><span class="sxs-lookup"><span data-stu-id="91310-134">Minimizing the hosting cost for websites that consist of only static content and resources.</span></span> <span data-ttu-id="91310-135">В зависимости от возможностей системы хранения поставщика услуг размещения можно разместить весь статический веб-сайт в учетной записи хранения.</span><span class="sxs-lookup"><span data-stu-id="91310-135">Depending on the capabilities of the hosting provider’s storage system, it might be possible to entirely host a fully static website in a storage account.</span></span>

- <span data-ttu-id="91310-136">Предоставление статических ресурсов и содержимого для приложений, выполняющихся в других средах размещения или на локальных серверах.</span><span class="sxs-lookup"><span data-stu-id="91310-136">Exposing static resources and content for applications running in other hosting environments or on-premises servers.</span></span>

- <span data-ttu-id="91310-137">Расположение содержимого в нескольких географических областях с использованием сети CDN, которая позволяет кэшировать содержимое учетной записи хранения в нескольких центрах обработки данных по всему миру.</span><span class="sxs-lookup"><span data-stu-id="91310-137">Locating content in more than one geographical area using a content delivery network that caches the contents of the storage account in multiple datacenters around the world.</span></span>

- <span data-ttu-id="91310-138">Мониторинг затрат и использования пропускной способности.</span><span class="sxs-lookup"><span data-stu-id="91310-138">Monitoring costs and bandwidth usage.</span></span> <span data-ttu-id="91310-139">Использование отдельной учетной записи хранения для некоторого или всего статического содержимого позволяет легко отделить затраты на размещение и на среду выполнения.</span><span class="sxs-lookup"><span data-stu-id="91310-139">Using a separate storage account for some or all of the static content allows the costs to be more easily separated from hosting and runtime costs.</span></span>

<span data-ttu-id="91310-140">Этот шаблон неприменим в следующих случаях:</span><span class="sxs-lookup"><span data-stu-id="91310-140">This pattern might not be useful in the following situations:</span></span>

- <span data-ttu-id="91310-141">Приложение должно выполнять обработку статического содержимого перед его доставкой клиенту.</span><span class="sxs-lookup"><span data-stu-id="91310-141">The application needs to perform some processing on the static content before delivering it to the client.</span></span> <span data-ttu-id="91310-142">Например, может потребоваться добавить метку времени в документ.</span><span class="sxs-lookup"><span data-stu-id="91310-142">For example, it might be necessary to add a timestamp to a document.</span></span>

- <span data-ttu-id="91310-143">Объем статического содержимого очень мал.</span><span class="sxs-lookup"><span data-stu-id="91310-143">The volume of static content is very small.</span></span> <span data-ttu-id="91310-144">Затраты на получение этого содержимого из отдельного хранилища могут превысить затраты на отделение содержимого от вычислительных ресурсов.</span><span class="sxs-lookup"><span data-stu-id="91310-144">The overhead of retrieving this content from separate storage can outweigh the cost benefit of separating it out from the compute resource.</span></span>

## <a name="example"></a><span data-ttu-id="91310-145">Пример</span><span class="sxs-lookup"><span data-stu-id="91310-145">Example</span></span>

<span data-ttu-id="91310-146">К статическому содержимому, расположенному в хранилище BLOB-объектов Azure, можно получить доступ непосредственно из браузера.</span><span class="sxs-lookup"><span data-stu-id="91310-146">Static content located in Azure Blob storage can be accessed directly by a web browser.</span></span> <span data-ttu-id="91310-147">Azure предоставляет интерфейс на основе HTTP для хранилища, который может быть открытым для клиентов.</span><span class="sxs-lookup"><span data-stu-id="91310-147">Azure provides an HTTP-based interface over storage that can be publicly exposed to clients.</span></span> <span data-ttu-id="91310-148">Например, содержимое в контейнере хранилища BLOB-объектов Azure предоставляется с использованием URL-адреса в следующем формате:</span><span class="sxs-lookup"><span data-stu-id="91310-148">For example, content in an Azure Blob storage container is exposed using a URL with the following form:</span></span>

`https://[ storage-account-name ].blob.core.windows.net/[ container-name ]/[ file-name ]`


<span data-ttu-id="91310-149">При отправке содержимого необходимо создать один или несколько контейнеров BLOB-объектов, в которых будут храниться файлы и документы.</span><span class="sxs-lookup"><span data-stu-id="91310-149">When uploading the content it's necessary to create one or more blob containers to hold the files and documents.</span></span> <span data-ttu-id="91310-150">Обратите внимание, что по умолчанию для нового контейнера задано разрешение "Закрытый". Это значение нужно заменить на "Открытый", чтобы клиенты могли получить доступ к содержимому.</span><span class="sxs-lookup"><span data-stu-id="91310-150">Note that the default permission for a new container is Private, and you must change this to Public to allow clients to access the contents.</span></span> <span data-ttu-id="91310-151">Если требуется защитить содержимое от анонимного доступа, можно реализовать [шаблон ключа камердинера](valet-key.md). В таком случае для скачивания ресурсов пользователи должны будут предоставить допустимый токен.</span><span class="sxs-lookup"><span data-stu-id="91310-151">If it's necessary to protect the content from anonymous access, you can implement the [Valet Key pattern](valet-key.md) so users must present a valid token to download the resources.</span></span>

> <span data-ttu-id="91310-152">Сведения о хранилище BLOB-объектов, а также о том, как получить к нему доступ и как его использовать, см. в статье [Основные понятия службы BLOB-объектов](https://msdn.microsoft.com/library/azure/dd179376.aspx).</span><span class="sxs-lookup"><span data-stu-id="91310-152">[Blob Service Concepts](https://msdn.microsoft.com/library/azure/dd179376.aspx) has information about blob storage, and the ways that you can access and use it.</span></span>

<span data-ttu-id="91310-153">В ссылках на каждой странице будет указан URL-адрес ресурса, и клиент будет получать к нему доступ прямо из службы хранения.</span><span class="sxs-lookup"><span data-stu-id="91310-153">The links in each page will specify the URL of the resource and the client will access it directly from the storage service.</span></span> <span data-ttu-id="91310-154">На рисунке показана доставка статических частей приложения непосредственно из службы хранения.</span><span class="sxs-lookup"><span data-stu-id="91310-154">The figure illustrates delivering static parts of an application directly from a storage service.</span></span>

![Рис. 1. Доставка статических частей приложения непосредственно из службы хранения](./_images/static-content-hosting-pattern.png)


<span data-ttu-id="91310-156">В ссылках на страницах, которые доставляются клиенту, должны указываться полные URL-адреса ресурса и контейнера BLOB-объектов.</span><span class="sxs-lookup"><span data-stu-id="91310-156">The links in the pages delivered to the client must specify the full URL of the blob container and resource.</span></span> <span data-ttu-id="91310-157">Например, страница со ссылкой на изображение в открытом контейнере может содержать следующий код HTML.</span><span class="sxs-lookup"><span data-stu-id="91310-157">For example, a page that contains a link to an image in a public container might contain the following HTML.</span></span>

```html
<img src="https://mystorageaccount.blob.core.windows.net/myresources/image1.png"
     alt="My image" />
```

> <span data-ttu-id="91310-158">Если ресурсы защищены с помощью ключа камердинера, например подписанного URL-адреса Azure, он должен включаться в URL-адреса в ссылках.</span><span class="sxs-lookup"><span data-stu-id="91310-158">If the resources are protected by using a valet key, such as an Azure shared access signature, this signature must be included in the URLs in the links.</span></span>

<span data-ttu-id="91310-159">Решение StaticContentHosting, в котором продемонстрировано использование внешнего хранилища для статических ресурсов, доступно на сайте [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting).</span><span class="sxs-lookup"><span data-stu-id="91310-159">A solution named StaticContentHosting that demonstrates using external storage for static resources is available from [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting).</span></span> <span data-ttu-id="91310-160">Проект StaticContentHosting.Cloud содержит файлы конфигурации, в которых указаны учетная запись хранения и контейнер, в котором хранится статическое содержимое.</span><span class="sxs-lookup"><span data-stu-id="91310-160">The StaticContentHosting.Cloud project contains configuration files that specify the storage account and container that holds the static content.</span></span>

```xml
<Setting name="StaticContent.StorageConnectionString"
         value="UseDevelopmentStorage=true" />
<Setting name="StaticContent.Container" value="static-content" />
```

<span data-ttu-id="91310-161">Класс `Settings` в файле Settings.cs проекта StaticContentHosting.Web содержит методы для извлечения этих значений и создания строкового значения, содержащего URL-адрес контейнера облачной учетной записи хранения.</span><span class="sxs-lookup"><span data-stu-id="91310-161">The `Settings` class in the file Settings.cs of the StaticContentHosting.Web project contains methods to extract these values and build a string value containing the cloud storage account container URL.</span></span>

```csharp
public class Settings
{
  public static string StaticContentStorageConnectionString {
    get
    {
      return RoleEnvironment.GetConfigurationSettingValue(
                              "StaticContent.StorageConnectionString");
    }
  }

  public static string StaticContentContainer
  {
    get
    {
      return RoleEnvironment.GetConfigurationSettingValue("StaticContent.Container");
    }
  }

  public static string StaticContentBaseUrl
  {
    get
    {
      var account = CloudStorageAccount.Parse(StaticContentStorageConnectionString);

      return string.Format("{0}/{1}", account.BlobEndpoint.ToString().TrimEnd('/'),
                                      StaticContentContainer.TrimStart('/'));
    }
  }
}
```

<span data-ttu-id="91310-162">Класс `StaticContentUrlHtmlHelper` в файле StaticContentUrlHtmlHelper.cs предоставляет метод `StaticContentUrl`, который создает URL-адрес, содержащий путь к облачной учетной записи хранения, если переданный в нее URL-адрес начинается с символа (~) пути к корневой папке ASP.NET.</span><span class="sxs-lookup"><span data-stu-id="91310-162">The `StaticContentUrlHtmlHelper` class in the file StaticContentUrlHtmlHelper.cs exposes a method named `StaticContentUrl` that generates a URL containing the path to the cloud storage account if the URL passed to it starts with the ASP.NET root path character (~).</span></span>

```csharp
public static class StaticContentUrlHtmlHelper
{
  public static string StaticContentUrl(this HtmlHelper helper, string contentPath)
  {
    if (contentPath.StartsWith("~"))
    {
      contentPath = contentPath.Substring(1);
    }

    contentPath = string.Format("{0}/{1}", Settings.StaticContentBaseUrl.TrimEnd('/'),
                                contentPath.TrimStart('/'));

    var url = new UrlHelper(helper.ViewContext.RequestContext);

    return url.Content(contentPath);
  }
}
```

<span data-ttu-id="91310-163">Файл Index.cshtml в папке Views\Home содержит элемент изображения, использующего метод `StaticContentUrl` для создания URL-адреса для его атрибута `src`.</span><span class="sxs-lookup"><span data-stu-id="91310-163">The file Index.cshtml in the Views\Home folder contains an image element that uses the `StaticContentUrl` method to create the URL for its `src` attribute.</span></span>

```html
<img src="@Html.StaticContentUrl("~/media/orderedList1.png")" alt="Test Image" />
```

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="91310-164">Связанные шаблоны и рекомендации</span><span class="sxs-lookup"><span data-stu-id="91310-164">Related patterns and guidance</span></span>

- <span data-ttu-id="91310-165">Пример, демонстрирующий этот шаблон, можно найти на сайте [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting).</span><span class="sxs-lookup"><span data-stu-id="91310-165">A sample that demonstrates this pattern is available on [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting).</span></span>
- <span data-ttu-id="91310-166">[Шаблон ключа камердинера](valet-key.md).</span><span class="sxs-lookup"><span data-stu-id="91310-166">[Valet Key pattern](valet-key.md).</span></span> <span data-ttu-id="91310-167">Если целевые ресурсы не должны быть доступы для анонимных пользователей, нужно обеспечить безопасность на уровне хранилища, которое содержит статическое содержимое.</span><span class="sxs-lookup"><span data-stu-id="91310-167">If the target resources aren't supposed to be available to anonymous users it's necessary to implement security over the store that holds the static content.</span></span> <span data-ttu-id="91310-168">В статье по ссылке объясняется, как использовать токен или ключ, который позволяет клиентам с ограниченным доступом получить прямой доступ к определенному ресурсу или службе, например к размещенной в облаке службе хранения.</span><span class="sxs-lookup"><span data-stu-id="91310-168">Describes how to use a token or key that provides clients with restricted direct access to a specific resource or service such as a cloud-hosted storage service.</span></span>
- <span data-ttu-id="91310-169">[Основные понятия службы BLOB-объектов](https://msdn.microsoft.com/library/azure/dd179376.aspx).</span><span class="sxs-lookup"><span data-stu-id="91310-169">[Blob Service Concepts](https://msdn.microsoft.com/library/azure/dd179376.aspx)</span></span>
