---
title: Шаблон размещения статического содержимого
titleSuffix: Cloud Design Patterns
description: Разверните статическое содержимое в облачной службе хранения для предоставления непосредственно клиенту.
keywords: Конструктивный шаблон
author: dragon119
ms.date: 01/04/2019
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 719f0221ecc8d52267cba3136eec20dadef30b99
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54483451"
---
# <a name="static-content-hosting-pattern"></a><span data-ttu-id="c57a7-104">Шаблон размещения статического содержимого</span><span class="sxs-lookup"><span data-stu-id="c57a7-104">Static Content Hosting pattern</span></span>

<span data-ttu-id="c57a7-105">Развертывайте статическое содержимое в облачной службе хранения, чтобы предоставлять его непосредственно клиенту.</span><span class="sxs-lookup"><span data-stu-id="c57a7-105">Deploy static content to a cloud-based storage service that can deliver them directly to the client.</span></span> <span data-ttu-id="c57a7-106">Это позволяет снизить потребность в использовании дорогостоящих вычислительных экземпляров.</span><span class="sxs-lookup"><span data-stu-id="c57a7-106">This can reduce the need for potentially expensive compute instances.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="c57a7-107">Контекст и проблема</span><span class="sxs-lookup"><span data-stu-id="c57a7-107">Context and problem</span></span>

<span data-ttu-id="c57a7-108">Как правило, веб-приложения содержат элементы статического содержимого.</span><span class="sxs-lookup"><span data-stu-id="c57a7-108">Web applications typically include some elements of static content.</span></span> <span data-ttu-id="c57a7-109">Это статическое содержимое может включать HTML-страницы и другие ресурсы, такие как изображения и документы, доступные для клиента либо в составе HTML-страниц (например, встроенные изображения, таблицы стилей и клиентские файлы JavaScript), либо как отдельно загружаемые страницы (например, PDF-документы).</span><span class="sxs-lookup"><span data-stu-id="c57a7-109">This static content might include HTML pages and other resources such as images and documents that are available to the client, either as part of an HTML page (such as inline images, style sheets, and client-side JavaScript files) or as separate downloads (such as PDF documents).</span></span>

<span data-ttu-id="c57a7-110">Хотя веб-серверы и оптимизированы для динамического отображения и кэширования вывода, они по-прежнему должны обрабатывать запросы на скачивание статического содержимого.</span><span class="sxs-lookup"><span data-stu-id="c57a7-110">Although web servers are optimized for dynamic rendering and output caching, they still have to handle requests to download static content.</span></span> <span data-ttu-id="c57a7-111">Для этого задействуются циклы обработки, которые могли бы использоваться для других задач.</span><span class="sxs-lookup"><span data-stu-id="c57a7-111">This consumes processing cycles that could often be put to better use.</span></span>

## <a name="solution"></a><span data-ttu-id="c57a7-112">Решение</span><span class="sxs-lookup"><span data-stu-id="c57a7-112">Solution</span></span>

<span data-ttu-id="c57a7-113">В большинстве сред облачного размещения вы можете поместить отдельные ресурсы и статические страницы приложения в службу хранилища.</span><span class="sxs-lookup"><span data-stu-id="c57a7-113">In most cloud hosting environments, you can put some of an application's resources and static pages in a storage service.</span></span> <span data-ttu-id="c57a7-114">Служба хранилища может обрабатывать запросы для этих ресурсов, уменьшая нагрузку на вычислительные ресурсы, обрабатывающие другие веб-запросы.</span><span class="sxs-lookup"><span data-stu-id="c57a7-114">The storage service can serve requests for these resources, reducing load on the compute resources that handle other web requests.</span></span> <span data-ttu-id="c57a7-115">Как правило, облачные хранилища намного дешевле, чем вычислительные экземпляры.</span><span class="sxs-lookup"><span data-stu-id="c57a7-115">The cost for cloud-hosted storage is typically much less than for compute instances.</span></span>

<span data-ttu-id="c57a7-116">Основные проблемы при размещении некоторых частей приложения в службе хранения связаны c развертыванием приложения и защитой ресурсов, которые не должны быть доступны для анонимных пользователей.</span><span class="sxs-lookup"><span data-stu-id="c57a7-116">When hosting some parts of an application in a storage service, the main considerations are related to deployment of the application and to securing resources that aren't intended to be available to anonymous users.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="c57a7-117">Проблемы и рекомендации</span><span class="sxs-lookup"><span data-stu-id="c57a7-117">Issues and considerations</span></span>

<span data-ttu-id="c57a7-118">При принятии решения о реализации этого шаблона необходимо учитывать следующие моменты.</span><span class="sxs-lookup"><span data-stu-id="c57a7-118">Consider the following points when deciding how to implement this pattern:</span></span>

- <span data-ttu-id="c57a7-119">Размещенная служба хранения должна предоставлять конечную точку HTTP, к которой пользователи смогут получать доступ для скачивания статических ресурсов.</span><span class="sxs-lookup"><span data-stu-id="c57a7-119">The hosted storage service must expose an HTTP endpoint that users can access to download the static resources.</span></span> <span data-ttu-id="c57a7-120">Некоторые службы хранения также поддерживают протокол HTTPS, что позволяет размещать в службах хранения ресурсы, для которых требуется протокол SSL.</span><span class="sxs-lookup"><span data-stu-id="c57a7-120">Some storage services also support HTTPS, so it's possible to host resources in storage services that require SSL.</span></span>

- <span data-ttu-id="c57a7-121">Если требуется обеспечить максимальную производительность и доступность, рекомендуем использовать сеть доставки содержимого (CDN) для кэширования содержимого контейнера хранилища в нескольких центрах обработки данных по всему миру.</span><span class="sxs-lookup"><span data-stu-id="c57a7-121">For maximum performance and availability, consider using a content delivery network (CDN) to cache the contents of the storage container in multiple datacenters around the world.</span></span> <span data-ttu-id="c57a7-122">Но за использование CDN может взиматься плата.</span><span class="sxs-lookup"><span data-stu-id="c57a7-122">However, you'll likely have to pay for using the CDN.</span></span>

- <span data-ttu-id="c57a7-123">Как правило, учетные записи хранения являются геореплицируемыми по умолчанию. Это позволяет обеспечить устойчивость при возникновении событий, которые могут повлиять на работу центра обработки данных.</span><span class="sxs-lookup"><span data-stu-id="c57a7-123">Storage accounts are often geo-replicated by default to provide resiliency against events that might affect a datacenter.</span></span> <span data-ttu-id="c57a7-124">Это означает, что IP-адрес может измениться, но URL-адрес останется тем же.</span><span class="sxs-lookup"><span data-stu-id="c57a7-124">This means that the IP address might change, but the URL will remain the same.</span></span>

- <span data-ttu-id="c57a7-125">Когда одна часть содержимого находится в учетной записи хранения, а другая — в размещенном вычислительном экземпляре, развертывание и обновление приложения усложняется.</span><span class="sxs-lookup"><span data-stu-id="c57a7-125">When some content is located in a storage account and other content is in a hosted compute instance, it becomes more challenging to deploy and update the application.</span></span> <span data-ttu-id="c57a7-126">Возможно, потребуется несколько отдельных развертываний, а также управление версиями приложения и содержимого. Это позволит упростить управление, особенно если статическое содержимое содержит файлы скриптов или компоненты пользовательского интерфейса.</span><span class="sxs-lookup"><span data-stu-id="c57a7-126">You might have to perform separate deployments, and version the application and content to manage it more easily&mdash;especially when the static content includes script files or UI components.</span></span> <span data-ttu-id="c57a7-127">Но если требуется обновить только статические ресурсы, их можно просто отправить в учетную запись хранения, не развертывая пакет приложения повторно.</span><span class="sxs-lookup"><span data-stu-id="c57a7-127">However, if only static resources have to be updated, they can simply be uploaded to the storage account without needing to redeploy the application package.</span></span>

- <span data-ttu-id="c57a7-128">Службы хранения могут не поддерживать пользовательские доменные имена.</span><span class="sxs-lookup"><span data-stu-id="c57a7-128">Storage services might not support the use of custom domain names.</span></span> <span data-ttu-id="c57a7-129">В этом случае необходимо указать полные URL-адреса ресурсов в виде ссылок, так как они будут находиться в домене, отличном от домена для динамического содержимого, содержащего ссылки.</span><span class="sxs-lookup"><span data-stu-id="c57a7-129">In this case it's necessary to specify the full URL of the resources in links because they'll be in a different domain from the dynamically-generated content containing the links.</span></span>

- <span data-ttu-id="c57a7-130">Контейнеры хранилища должны быть настроены для открытого доступа на чтение. Но обязательно убедитесь, что они не настроены для открытого доступа на запись, чтобы пользователи не могли отправлять содержимое.</span><span class="sxs-lookup"><span data-stu-id="c57a7-130">The storage containers must be configured for public read access, but it's vital to ensure that they aren't configured for public write access to prevent users being able to upload content.</span></span>

- <span data-ttu-id="c57a7-131">Для управления доступом к ресурсам, которые не должны быть доступны анонимно, рекомендуется использовать ключ ограниченного доступа или маркер проверки подлинности.</span><span class="sxs-lookup"><span data-stu-id="c57a7-131">Consider using a valet key or token to control access to resources that shouldn't be available anonymously.</span></span> <span data-ttu-id="c57a7-132">Дополнительные сведения см. в описании [шаблона ключа ограниченного доступа](./valet-key.md).</span><span class="sxs-lookup"><span data-stu-id="c57a7-132">See the [Valet Key pattern](./valet-key.md) for more information.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="c57a7-133">Когда следует использовать этот шаблон</span><span class="sxs-lookup"><span data-stu-id="c57a7-133">When to use this pattern</span></span>

<span data-ttu-id="c57a7-134">Этот шаблон можно использовать для следующих целей:</span><span class="sxs-lookup"><span data-stu-id="c57a7-134">This pattern is useful for:</span></span>

- <span data-ttu-id="c57a7-135">Сокращение затрат на размещение веб-сайтов и приложений, которые содержат некоторые статические ресурсы.</span><span class="sxs-lookup"><span data-stu-id="c57a7-135">Minimizing the hosting cost for websites and applications that contain some static resources.</span></span>

- <span data-ttu-id="c57a7-136">Сокращение затрат на размещение веб-сайтов, которые содержат только статическое содержимое и ресурсы.</span><span class="sxs-lookup"><span data-stu-id="c57a7-136">Minimizing the hosting cost for websites that consist of only static content and resources.</span></span> <span data-ttu-id="c57a7-137">В зависимости от возможностей системы хранилища поставщика услуг размещения можно разместить весь статический веб-сайт в учетной записи хранения.</span><span class="sxs-lookup"><span data-stu-id="c57a7-137">Depending on the capabilities of the hosting provider's storage system, it might be possible to entirely host a fully static website in a storage account.</span></span>

- <span data-ttu-id="c57a7-138">Предоставление статических ресурсов и содержимого для приложений, выполняющихся в других средах размещения или на локальных серверах.</span><span class="sxs-lookup"><span data-stu-id="c57a7-138">Exposing static resources and content for applications running in other hosting environments or on-premises servers.</span></span>

- <span data-ttu-id="c57a7-139">Расположение содержимого в нескольких географических областях с использованием сети CDN, которая позволяет кэшировать содержимое учетной записи хранения в нескольких центрах обработки данных по всему миру.</span><span class="sxs-lookup"><span data-stu-id="c57a7-139">Locating content in more than one geographical area using a content delivery network that caches the contents of the storage account in multiple datacenters around the world.</span></span>

- <span data-ttu-id="c57a7-140">Мониторинг затрат и использования пропускной способности.</span><span class="sxs-lookup"><span data-stu-id="c57a7-140">Monitoring costs and bandwidth usage.</span></span> <span data-ttu-id="c57a7-141">Использование отдельной учетной записи хранения для некоторого или всего статического содержимого позволяет легко отделить затраты на размещение и на среду выполнения.</span><span class="sxs-lookup"><span data-stu-id="c57a7-141">Using a separate storage account for some or all of the static content allows the costs to be more easily separated from hosting and runtime costs.</span></span>

<span data-ttu-id="c57a7-142">Этот шаблон неприменим в следующих случаях:</span><span class="sxs-lookup"><span data-stu-id="c57a7-142">This pattern might not be useful in the following situations:</span></span>

- <span data-ttu-id="c57a7-143">Приложение должно выполнять обработку статического содержимого перед его доставкой клиенту.</span><span class="sxs-lookup"><span data-stu-id="c57a7-143">The application needs to perform some processing on the static content before delivering it to the client.</span></span> <span data-ttu-id="c57a7-144">Например, может потребоваться добавить метку времени в документ.</span><span class="sxs-lookup"><span data-stu-id="c57a7-144">For example, it might be necessary to add a timestamp to a document.</span></span>

- <span data-ttu-id="c57a7-145">Объем статического содержимого очень мал.</span><span class="sxs-lookup"><span data-stu-id="c57a7-145">The volume of static content is very small.</span></span> <span data-ttu-id="c57a7-146">Затраты на получение этого содержимого из отдельного хранилища могут превысить затраты на отделение содержимого от вычислительных ресурсов.</span><span class="sxs-lookup"><span data-stu-id="c57a7-146">The overhead of retrieving this content from separate storage can outweigh the cost benefit of separating it out from the compute resource.</span></span>

## <a name="example"></a><span data-ttu-id="c57a7-147">Пример</span><span class="sxs-lookup"><span data-stu-id="c57a7-147">Example</span></span>

<span data-ttu-id="c57a7-148">Служба хранилища Azure поддерживает обслуживание статического содержимого непосредственно из контейнера хранилища.</span><span class="sxs-lookup"><span data-stu-id="c57a7-148">Azure Storage supports serving static content directly from a storage container.</span></span> <span data-ttu-id="c57a7-149">Файлы обслуживаются путем выполнения анонимных запросов на доступ.</span><span class="sxs-lookup"><span data-stu-id="c57a7-149">Files are served through anonymous access requests.</span></span> <span data-ttu-id="c57a7-150">По умолчанию файлы имеют URL-адрес в поддомене `core.windows.net`, например `https://contoso.z4.web.core.windows.net/image.png`.</span><span class="sxs-lookup"><span data-stu-id="c57a7-150">By default, files have a URL in a subdomain of `core.windows.net`, such as `https://contoso.z4.web.core.windows.net/image.png`.</span></span> <span data-ttu-id="c57a7-151">Вы можете настроить пользовательское доменное имя и использовать Azure CDN для доступа к файлам по протоколу HTTPS.</span><span class="sxs-lookup"><span data-stu-id="c57a7-151">You can configure a custom domain name, and use Azure CDN to access the files over HTTPS.</span></span> <span data-ttu-id="c57a7-152">Дополнительные сведения см. в руководстве по [размещению статических веб-сайтов в службе хранилища Azure](/azure/storage/blobs/storage-blob-static-website).</span><span class="sxs-lookup"><span data-stu-id="c57a7-152">For more information, see [Static website hosting in Azure Storage](/azure/storage/blobs/storage-blob-static-website).</span></span>

![Доставка статических частей приложения непосредственно из службы хранения](./_images/static-content-hosting-pattern.png)

<span data-ttu-id="c57a7-154">Размещение статического веб-сайта обеспечивает анонимный доступ к файлам.</span><span class="sxs-lookup"><span data-stu-id="c57a7-154">Static website hosting makes the files available for anonymous access.</span></span> <span data-ttu-id="c57a7-155">Если необходимо контролировать доступ пользователей к файлам, можно поместить файлы в хранилище BLOB-объектов Azure, а затем создать [подписанные URL-адреса](/azure/storage/common/storage-dotnet-shared-access-signature-part-1) для ограничения доступа.</span><span class="sxs-lookup"><span data-stu-id="c57a7-155">If you need to control who can access the files, you can store files in Azure blob storage and then generate [shared access signatures](/azure/storage/common/storage-dotnet-shared-access-signature-part-1) to limit access.</span></span>

<span data-ttu-id="c57a7-156">В ссылках на страницах, которые доставляются клиенту, должен указываться полный URL-адрес ресурса.</span><span class="sxs-lookup"><span data-stu-id="c57a7-156">The links in the pages delivered to the client must specify the full URL of the resource.</span></span> <span data-ttu-id="c57a7-157">Если ресурс защищен с помощью ключа ограниченного доступа, например подписанного URL-адреса, он должен включаться в URL-адрес.</span><span class="sxs-lookup"><span data-stu-id="c57a7-157">If the resource is protected with a valet key, such as a shared access signature, this signature must be included in the URL.</span></span>

<span data-ttu-id="c57a7-158">Пример приложения, в котором продемонстрировано использование внешнего хранилища для статических ресурсов, доступен на сайте [GitHub][sample-app].</span><span class="sxs-lookup"><span data-stu-id="c57a7-158">A sample application that demonstrates using external storage for static resources is available on [GitHub][sample-app].</span></span> <span data-ttu-id="c57a7-159">В этом примере используются файлы конфигурации, чтобы указать учетную запись хранения и контейнер, в котором хранится статическое содержимое.</span><span class="sxs-lookup"><span data-stu-id="c57a7-159">This sample uses configuration files to specify the storage account and container that holds the static content.</span></span>

```xml
<Setting name="StaticContent.StorageConnectionString"
         value="UseDevelopmentStorage=true" />
<Setting name="StaticContent.Container" value="static-content" />
```

<span data-ttu-id="c57a7-160">Класс `Settings` в файле Settings.cs проекта StaticContentHosting.Web содержит методы для извлечения этих значений и создания строкового значения, содержащего URL-адрес контейнера облачной учетной записи хранения.</span><span class="sxs-lookup"><span data-stu-id="c57a7-160">The `Settings` class in the file Settings.cs of the StaticContentHosting.Web project contains methods to extract these values and build a string value containing the cloud storage account container URL.</span></span>

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

<span data-ttu-id="c57a7-161">Класс `StaticContentUrlHtmlHelper` в файле StaticContentUrlHtmlHelper.cs предоставляет метод `StaticContentUrl`, который создает URL-адрес, содержащий путь к облачной учетной записи хранения, если переданный в нее URL-адрес начинается с символа (~) пути к корневой папке ASP.NET.</span><span class="sxs-lookup"><span data-stu-id="c57a7-161">The `StaticContentUrlHtmlHelper` class in the file StaticContentUrlHtmlHelper.cs exposes a method named `StaticContentUrl` that generates a URL containing the path to the cloud storage account if the URL passed to it starts with the ASP.NET root path character (~).</span></span>

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

<span data-ttu-id="c57a7-162">Файл Index.cshtml в папке Views\Home содержит элемент изображения, использующего метод `StaticContentUrl` для создания URL-адреса для его атрибута `src`.</span><span class="sxs-lookup"><span data-stu-id="c57a7-162">The file Index.cshtml in the Views\Home folder contains an image element that uses the `StaticContentUrl` method to create the URL for its `src` attribute.</span></span>

```html
<img src="@Html.StaticContentUrl("~/media/orderedList1.png")" alt="Test Image" />
```

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="c57a7-163">Связанные шаблоны и рекомендации</span><span class="sxs-lookup"><span data-stu-id="c57a7-163">Related patterns and guidance</span></span>

- <span data-ttu-id="c57a7-164">[Пример размещения статического содержимого][sample-app].</span><span class="sxs-lookup"><span data-stu-id="c57a7-164">[Static Content Hosting sample][sample-app].</span></span> <span data-ttu-id="c57a7-165">Пример приложения, иллюстрирующий этот шаблон.</span><span class="sxs-lookup"><span data-stu-id="c57a7-165">A sample application that demonstrates this pattern.</span></span>
- <span data-ttu-id="c57a7-166">[Шаблон ключа камердинера](./valet-key.md).</span><span class="sxs-lookup"><span data-stu-id="c57a7-166">[Valet Key pattern](./valet-key.md).</span></span> <span data-ttu-id="c57a7-167">Если целевые ресурсы не должны быть доступными для анонимных пользователей, используйте этот шаблон, чтобы запретить прямой доступ.</span><span class="sxs-lookup"><span data-stu-id="c57a7-167">If the target resources aren't supposed to be available to anonymous users, use this pattern to restrict direct access.</span></span>
- <span data-ttu-id="c57a7-168">[Бессерверное веб-приложение в Azure](../reference-architectures/serverless/web-app.md).</span><span class="sxs-lookup"><span data-stu-id="c57a7-168">[Serverless web application on Azure](../reference-architectures/serverless/web-app.md).</span></span> <span data-ttu-id="c57a7-169">Эталонная архитектура, в которой реализовано бессерверное веб-приложение путем размещения статического веб-сайта с помощью Функций Azure.</span><span class="sxs-lookup"><span data-stu-id="c57a7-169">A reference architecture that uses static website hosting with Azure Functions to implement a serverless web app.</span></span>

[sample-app]: https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting
