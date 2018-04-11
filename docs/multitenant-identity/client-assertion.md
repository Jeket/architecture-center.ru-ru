---
title: Получение маркеров доступа из Azure AD с помощью утверждений клиентов
description: Сведения об использовании утверждений клиентов для получения маркеров доступа из Azure AD.
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: adfs
pnp.series.next: key-vault
ms.openlocfilehash: 9fe1ee2ec5a540edc41c3a310476507f8d862f0c
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/14/2017
---
# <a name="use-client-assertion-to-get-access-tokens-from-azure-ad"></a>Получение маркеров доступа из Azure AD с помощью утверждений клиентов

[![GitHub](../_images/github.png) Пример кода][sample application]

## <a name="background"></a>Фоновый
При использовании потока кода авторизации или гибридного потока в OpenID Connect клиент обменивается кодом авторизации для маркера доступа. На этом этапе клиент должен пройти проверку подлинности на сервере.

![Секрет клиента](./images/client-secret.png)

Один из способов проверки подлинности клиента заключается в использовании секрета клиента. Вот как приложение [Tailspin Surveys][Surveys] настроено по умолчанию.

Ниже приведен пример запроса клиента на маркер доступа от поставщика удостоверений. Обратите внимание на параметр `client_secret` .

```
POST https://login.microsoftonline.com/b9bd2162xxx/oauth2/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

resource=https://tailspin.onmicrosoft.com/surveys.webapi
  &client_id=87df91dc-63de-4765-8701-b59cc8bd9e11
  &client_secret=i3Bf12Dn...
  &grant_type=authorization_code
  &code=PG8wJG6Y...
```

Секрет представляет собой просто строку, поэтому необходимо защитить его от раскрытия. Секрет клиента рекомендуется хранить вне системы управления версиями. При развертывании в Azure храните секрет клиента в [параметрах приложения][configure-web-app].

Однако все, кто имеет доступ к подписке Azure, могут просматривать параметры приложения. Кроме того, всегда есть возможность проверить секретные данные в системе управления версиями (например, в сценарии развертывания), отправить их по электронной почте и т. д.

Для обеспечения дополнительной безопасности вместо секрета клиента можно использовать [утверждение клиента]. При наличии утверждения клиент использует сертификат X.509 для доказательства получения запроса на маркер от клиента. Сертификат клиента установлен на веб-сервере. Как правило, будет проще ограничить доступ к сертификату, чем предотвратить непреднамеренное раскрытие секрета клиента. Дополнительные сведения о настройке сертификатов в веб-приложении см. в статье [Using Certificates in Azure Websites Applications][using-certs-in-websites] (Использование сертификатов в приложениях веб-сайтов Azure).

Ниже приведен запрос на маркер с помощью утверждения клиента.

```
POST https://login.microsoftonline.com/b9bd2162xxx/oauth2/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

resource=https://tailspin.onmicrosoft.com/surveys.webapi
  &client_id=87df91dc-63de-4765-8701-b59cc8bd9e11
  &client_assertion_type=urn:ietf:params:oauth:client-assertion-type:jwt-bearer
  &client_assertion=eyJhbGci...
  &grant_type=authorization_code
  &code= PG8wJG6Y...
```

Обратите внимание, что параметр `client_secret` больше не используется. Параметр `client_assertion` содержит маркер JWT, который был подписан с помощью сертификата клиента. Параметр `client_assertion_type` указывает тип утверждения, в этом случае &mdash; токен JWT. Сервер проверяет маркер JWT. Если маркер JWT является недопустимым, запрос на маркер возвращает ошибку.

> [!NOTE]
> Сертификаты X.509 не являются единственной формой утверждения клиента. Но мы сосредоточимся на этом, так как сертификаты X.509 поддерживаются службой Azure AD.
> 
> 

Во время выполнения веб-приложение считывает сертификат из хранилища сертификатов. Сертификат должен быть установлен на одном компьютере с веб-приложением.

В приложении Surveys есть вспомогательный класс, который создает [ClientAssertionCertificate](/dotnet/api/microsoft.identitymodel.clients.activedirectory.clientassertioncertificate), передаваемый в метод [AuthenticationContext.AcquireTokenSilentAsync](/dotnet/api/microsoft.identitymodel.clients.activedirectory.authenticationcontext.acquiretokensilentasync) для получения токена из Azure AD.

```csharp
public class CertificateCredentialService : ICredentialService
{
    private Lazy<Task<AdalCredential>> _credential;

    public CertificateCredentialService(IOptions<ConfigurationOptions> options)
    {
        var aadOptions = options.Value?.AzureAd;
        _credential = new Lazy<Task<AdalCredential>>(() =>
        {
            X509Certificate2 cert = CertificateUtility.FindCertificateByThumbprint(
                aadOptions.Asymmetric.StoreName,
                aadOptions.Asymmetric.StoreLocation,
                aadOptions.Asymmetric.CertificateThumbprint,
                aadOptions.Asymmetric.ValidationRequired);
            string password = null;
            var certBytes = CertificateUtility.ExportCertificateWithPrivateKey(cert, out password);
            return Task.FromResult(new AdalCredential(new ClientAssertionCertificate(aadOptions.ClientId, new X509Certificate2(certBytes, password))));
        });
    }

    public async Task<AdalCredential> GetCredentialsAsync()
    {
        return await _credential.Value;
    }
}
```

Сведения о настройке клиентских утверждений в приложении Surveys см. в статье [Use Azure Key Vault to protect application secrets][key vault] (Использование хранилища Azure Key Vault для защиты секретов приложения).

[**Далее**][key vault]

<!-- Links -->
[configure-web-app]: /azure/app-service-web/web-sites-configure/
[azure-management-portal]: https://portal.azure.com
[утверждение клиента]: https://tools.ietf.org/html/rfc7521
[key vault]: key-vault.md
[Setup-KeyVault]: https://github.com/mspnp/multitenant-saas-guidance/blob/master/scripts/Setup-KeyVault.ps1
[Surveys]: tailspin.md
[using-certs-in-websites]: https://azure.microsoft.com/blog/using-certificates-in-azure-websites-applications/

[sample application]: https://github.com/mspnp/multitenant-saas-guidance
