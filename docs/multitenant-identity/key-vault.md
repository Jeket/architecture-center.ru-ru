---
title: Использование Key Vault для защиты секретов приложения
description: Как использовать службу Key Vault для хранения секретов приложения.
author: MikeWasson
ms.date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: client-assertion
ms.openlocfilehash: dc471ca5fa090270465624548ffe7335363d6cb7
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54112759"
---
# <a name="use-azure-key-vault-to-protect-application-secrets"></a><span data-ttu-id="fa6fc-103">Использование Azure Key Vault для защиты секретов приложений</span><span class="sxs-lookup"><span data-stu-id="fa6fc-103">Use Azure Key Vault to protect application secrets</span></span>

<span data-ttu-id="fa6fc-104">[![GitHub](../_images/github.png) Пример кода][sample application]</span><span class="sxs-lookup"><span data-stu-id="fa6fc-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="fa6fc-105">Довольно часто используются параметры приложения, которые являются конфиденциальными и должны быть защищены, например:</span><span class="sxs-lookup"><span data-stu-id="fa6fc-105">It's common to have application settings that are sensitive and must be protected, such as:</span></span>

* <span data-ttu-id="fa6fc-106">строки подключения к базе данных;</span><span class="sxs-lookup"><span data-stu-id="fa6fc-106">Database connection strings</span></span>
* <span data-ttu-id="fa6fc-107">Пароли</span><span class="sxs-lookup"><span data-stu-id="fa6fc-107">Passwords</span></span>
* <span data-ttu-id="fa6fc-108">Криптографические ключи</span><span class="sxs-lookup"><span data-stu-id="fa6fc-108">Cryptographic keys</span></span>

<span data-ttu-id="fa6fc-109">Из соображений безопасности не следует размещать эти секреты в системе управления версиями.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-109">As a security best practice, you should never store these secrets in source control.</span></span> <span data-ttu-id="fa6fc-110">Вероятность утечки будет слишком высокой, даже если вы используете частный репозиторий исходного кода.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-110">It's too easy for them to leak &mdash; even if your source code repository is private.</span></span> <span data-ttu-id="fa6fc-111">И вопрос не только в защите секретов от общего доступа.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-111">And it's not just about keeping secrets from the general public.</span></span> <span data-ttu-id="fa6fc-112">В больших проектах может потребоваться ограничить доступ к производственным секретам, предоставив его только определенным разработчикам и операторам.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-112">On larger projects, you might want to restrict which developers and operators can access the production secrets.</span></span> <span data-ttu-id="fa6fc-113">(Для сред тестирования и разработки используются разные параметры.)</span><span class="sxs-lookup"><span data-stu-id="fa6fc-113">(Settings for test or development environments are different.)</span></span>

<span data-ttu-id="fa6fc-114">Более безопасный вариант — хранить эти секреты в [Azure Key Vault][KeyVault].</span><span class="sxs-lookup"><span data-stu-id="fa6fc-114">A more secure option is to store these secrets in [Azure Key Vault][KeyVault].</span></span> <span data-ttu-id="fa6fc-115">Хранилище ключей является облачной службой для управления криптографическими ключами и другими секретными данными.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-115">Key Vault is a cloud-hosted service for managing cryptographic keys and other secrets.</span></span> <span data-ttu-id="fa6fc-116">В этой статье описано, как использовать Key Vault для хранения параметров конфигурации приложения.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-116">This article shows how to use Key Vault to store configuration settings for your app.</span></span>

<span data-ttu-id="fa6fc-117">В приложении [Tailspin Surveys][Surveys] секретами являются следующие параметры:</span><span class="sxs-lookup"><span data-stu-id="fa6fc-117">In the [Tailspin Surveys][Surveys] application, the following settings are secret:</span></span>

* <span data-ttu-id="fa6fc-118">строка подключения к базе данных;</span><span class="sxs-lookup"><span data-stu-id="fa6fc-118">The database connection string.</span></span>
* <span data-ttu-id="fa6fc-119">строка подключения к Redis;</span><span class="sxs-lookup"><span data-stu-id="fa6fc-119">The Redis connection string.</span></span>
* <span data-ttu-id="fa6fc-120">секрет клиента для веб-приложения.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-120">The client secret for the web application.</span></span>

<span data-ttu-id="fa6fc-121">Приложение Surveys загружает параметры конфигурации из следующих расположений:</span><span class="sxs-lookup"><span data-stu-id="fa6fc-121">The Surveys application loads configuration settings from the following places:</span></span>

* <span data-ttu-id="fa6fc-122">файл appsettings.json;</span><span class="sxs-lookup"><span data-stu-id="fa6fc-122">The appsettings.json file</span></span>
* <span data-ttu-id="fa6fc-123">[хранилище секретов пользователя][user-secrets] (только в среде разработки; для тестирования);</span><span class="sxs-lookup"><span data-stu-id="fa6fc-123">The [user secrets store][user-secrets] (development environment only; for testing)</span></span>
* <span data-ttu-id="fa6fc-124">среда размещения (параметры веб-приложений Azure);</span><span class="sxs-lookup"><span data-stu-id="fa6fc-124">The hosting environment (app settings in Azure web apps)</span></span>
* <span data-ttu-id="fa6fc-125">Key Vault (если служба подключена).</span><span class="sxs-lookup"><span data-stu-id="fa6fc-125">Key Vault (when enabled)</span></span>

<span data-ttu-id="fa6fc-126">В этом списке хранилища перечислены в порядке увеличения приоритета, то есть параметры из Key Vault используются всегда.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-126">Each of these overrides the previous one, so any settings stored in Key Vault take precedence.</span></span>

> [!NOTE]
> <span data-ttu-id="fa6fc-127">По умолчанию поставщик конфигурации хранилища ключей отключен.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-127">By default, the Key Vault configuration provider is disabled.</span></span> <span data-ttu-id="fa6fc-128">Он не требуется для локального запуска приложения.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-128">It's not needed for running the application locally.</span></span> <span data-ttu-id="fa6fc-129">Его можно будет активировать в рабочей среде.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-129">You would enable it in a production deployment.</span></span>

<span data-ttu-id="fa6fc-130">При запуске приложение считывает параметры каждого зарегистрированного поставщика конфигураций и использует их для заполнения строго типизированного объекта параметров.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-130">At startup, the application reads settings from every registered configuration provider, and uses them to populate a strongly typed options object.</span></span> <span data-ttu-id="fa6fc-131">Дополнительные сведения см. в статье об [использовании параметров и объектов конфигурации][options].</span><span class="sxs-lookup"><span data-stu-id="fa6fc-131">For more information, see [Using Options and configuration objects][options].</span></span>

## <a name="setting-up-key-vault-in-the-surveys-app"></a><span data-ttu-id="fa6fc-132">Настройка хранилища ключей в приложении Surveys</span><span class="sxs-lookup"><span data-stu-id="fa6fc-132">Setting up Key Vault in the Surveys app</span></span>

<span data-ttu-id="fa6fc-133">Предварительные требования:</span><span class="sxs-lookup"><span data-stu-id="fa6fc-133">Prerequisites:</span></span>

* <span data-ttu-id="fa6fc-134">Установите [командлеты Azure Resource Manager][azure-rm-cmdlets].</span><span class="sxs-lookup"><span data-stu-id="fa6fc-134">Install the [Azure Resource Manager Cmdlets][azure-rm-cmdlets].</span></span>
* <span data-ttu-id="fa6fc-135">Настройте приложение Surveys, как описано в статье [Run the Surveys application][readme] (Запуск приложения Surveys).</span><span class="sxs-lookup"><span data-stu-id="fa6fc-135">Configure the Surveys application as described in [Run the Surveys application][readme].</span></span>

<span data-ttu-id="fa6fc-136">Основные действия:</span><span class="sxs-lookup"><span data-stu-id="fa6fc-136">High-level steps:</span></span>

1. <span data-ttu-id="fa6fc-137">Настройте в клиенте учетную запись пользователя с правами администратора.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-137">Set up an admin user in the tenant.</span></span>
2. <span data-ttu-id="fa6fc-138">Настройте сертификат клиента.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-138">Set up a client certificate.</span></span>
3. <span data-ttu-id="fa6fc-139">Создать хранилище ключей.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-139">Create a key vault.</span></span>
4. <span data-ttu-id="fa6fc-140">Добавьте параметры конфигурации в хранилище ключей.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-140">Add configuration settings to your key vault.</span></span>
5. <span data-ttu-id="fa6fc-141">Раскомментируйте код, который активирует хранилище ключей.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-141">Uncomment the code that enables key vault.</span></span>
6. <span data-ttu-id="fa6fc-142">Обновите секреты пользователя приложения.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-142">Update the application's user secrets.</span></span>

### <a name="set-up-an-admin-user"></a><span data-ttu-id="fa6fc-143">Настройка учетной записи пользователя с правами администратора</span><span class="sxs-lookup"><span data-stu-id="fa6fc-143">Set up an admin user</span></span>

> [!NOTE]
> <span data-ttu-id="fa6fc-144">Для создания хранилища ключей используется учетная запись, с помощью которой можно управлять подпиской Azure.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-144">To create a key vault, you must use an account which can manage your Azure subscription.</span></span> <span data-ttu-id="fa6fc-145">Кроме того, любое приложение, которому вы предоставляете права на чтение данных из хранилища ключей, должно быть зарегистрировано в том же клиенте, что и эта учетная запись.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-145">Also, any application that you authorize to read from the key vault must be registered in the same tenant as that account.</span></span>

<span data-ttu-id="fa6fc-146">На этом этапе следует удостовериться, что вы имеете возможность создать хранилище ключей, находясь в системе в качестве пользователя, который выполнил вход из клиента с зарегистрированным приложением Surveys.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-146">In this step, you will make sure that you can create a key vault while signed in as a user from the tenant where the Surveys app is registered.</span></span>

<span data-ttu-id="fa6fc-147">Создайте учетную запись пользователя с правами администратора в клиенте Azure AD, в котором зарегистрировано приложение Surveys.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-147">Create an administrator user within the Azure AD tenant where the Surveys application is registered.</span></span>

1. <span data-ttu-id="fa6fc-148">Войдите на [портал Azure][azure-portal].</span><span class="sxs-lookup"><span data-stu-id="fa6fc-148">Log into the [Azure portal][azure-portal].</span></span>
2. <span data-ttu-id="fa6fc-149">Выберите клиент Azure AD, в котором зарегистрировано приложение.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-149">Select the Azure AD tenant where your application is registered.</span></span>
3. <span data-ttu-id="fa6fc-150">Последовательно щелкните **More service** (Дополнительные службы) > **БЕЗОПАСНОСТЬ И ИДЕНТИФИКАЦИЯ** > **Azure Active Directory** > **Пользователи и группы** > **All users** (Все пользователи).</span><span class="sxs-lookup"><span data-stu-id="fa6fc-150">Click **More service** > **SECURITY + IDENTITY** > **Azure Active Directory** > **User and groups** > **All users**.</span></span>
4. <span data-ttu-id="fa6fc-151">Щелкните команду **Новый пользователь** в верхней части портала.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-151">At the top of the portal, click **New user**.</span></span>
5. <span data-ttu-id="fa6fc-152">Заполните все поля и назначьте пользователю роль **глобального администратора** для каталога.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-152">Fill in the fields and assign the user to the **Global administrator** directory role.</span></span>
6. <span data-ttu-id="fa6fc-153">Нажмите кнопку **Создать**.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-153">Click **Create**.</span></span>

![Пользователь "Глобальный администратор"](./images/running-the-app/global-admin-user.png)

<span data-ttu-id="fa6fc-155">Теперь назначьте этого пользователя владельцем подписки.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-155">Now assign this user as the subscription owner.</span></span>

1. <span data-ttu-id="fa6fc-156">В главном меню выберите **Подписки**.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-156">On the Hub menu, select **Subscriptions**.</span></span>

    ![Снимок экрана: главное меню портала Azure](./images/running-the-app/subscriptions.png)

2. <span data-ttu-id="fa6fc-158">Выберите подписку, доступ к которой нужно предоставить этому администратору.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-158">Select the subscription that you want the administrator to access.</span></span>
3. <span data-ttu-id="fa6fc-159">В колонке подписки выберите **Управление доступом (IAM)**.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-159">In the subscription blade, select **Access control (IAM)**.</span></span>
4. <span data-ttu-id="fa6fc-160">Щелкните **Добавить**.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-160">Click **Add**.</span></span>
5. <span data-ttu-id="fa6fc-161">В разделе **Роль**выберите **Владелец**.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-161">Under **Role**, select **Owner**.</span></span>
6. <span data-ttu-id="fa6fc-162">Укажите адрес электронной почты пользователя, для которого хотите добавить роль "Владелец".</span><span class="sxs-lookup"><span data-stu-id="fa6fc-162">Type the email address of the user you want to add as owner.</span></span>
7. <span data-ttu-id="fa6fc-163">Выберите нужного пользователя и щелкните **Сохранить**.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-163">Select the user and click **Save**.</span></span>

### <a name="set-up-a-client-certificate"></a><span data-ttu-id="fa6fc-164">Настройка сертификата клиента</span><span class="sxs-lookup"><span data-stu-id="fa6fc-164">Set up a client certificate</span></span>

1. <span data-ttu-id="fa6fc-165">Запустите скрипт PowerShell [/Scripts/Setup-KeyVault.ps1][Setup-KeyVault] следующим образом:</span><span class="sxs-lookup"><span data-stu-id="fa6fc-165">Run the PowerShell script [/Scripts/Setup-KeyVault.ps1][Setup-KeyVault] as follows:</span></span>

    ```powershell
    .\Setup-KeyVault.ps1 -Subject <<subject>>
    ```
    <span data-ttu-id="fa6fc-166">Для параметра `Subject` введите любое имя, например "surveysapp".</span><span class="sxs-lookup"><span data-stu-id="fa6fc-166">For the `Subject` parameter, enter any name, such as "surveysapp".</span></span> <span data-ttu-id="fa6fc-167">Сценарий создаст самозаверяющий сертификат и сохранит его в хранилище сертификатов "Текущий пользователь/Личные".</span><span class="sxs-lookup"><span data-stu-id="fa6fc-167">The script generates a self-signed certificate and stores it in the "Current User/Personal" certificate store.</span></span> <span data-ttu-id="fa6fc-168">Выходные данные этого сценария представлены фрагментом JSON.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-168">The output from the script is a JSON fragment.</span></span> <span data-ttu-id="fa6fc-169">Скопируйте это значение.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-169">Copy this value.</span></span>

2. <span data-ttu-id="fa6fc-170">На [портале Azure][azure-portal] перейдите к каталогу, где зарегистрировано приложение Surveys, выбрав свою учетную запись в правом верхнем углу портала.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-170">In the [Azure portal][azure-portal], switch to the directory where the Surveys application is registered, by selecting your account in the top right corner of the portal.</span></span>

3. <span data-ttu-id="fa6fc-171">Выберите **Azure Active Directory** > **Регистрация приложений** > Surveys.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-171">Select **Azure Active Directory** > **App Registrations** > Surveys</span></span>

4. <span data-ttu-id="fa6fc-172">Щелкните **Манифест** и выберите **Изменить**.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-172">Click **Manifest** and then **Edit**.</span></span>

5. <span data-ttu-id="fa6fc-173">Вставьте выходные данные сценария в свойство `keyCredentials` .</span><span class="sxs-lookup"><span data-stu-id="fa6fc-173">Paste the output from the script into the `keyCredentials` property.</span></span> <span data-ttu-id="fa6fc-174">Это должно выглядеть следующим образом:</span><span class="sxs-lookup"><span data-stu-id="fa6fc-174">It should look similar to the following:</span></span>

    ```json
    "keyCredentials": [
        {
        "type": "AsymmetricX509Cert",
        "usage": "Verify",
        "keyId": "29d4f7db-0539-455e-b708-....",
        "customKeyIdentifier": "ZEPpP/+KJe2fVDBNaPNOTDoJMac=",
        "value": "MIIDAjCCAeqgAwIBAgIQFxeRiU59eL.....
        }
    ],
    ```

6. <span data-ttu-id="fa6fc-175">Выберите команду **Сохранить**.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-175">Click **Save**.</span></span>

7. <span data-ttu-id="fa6fc-176">Повторите шаги 3–6 и добавьте этот же фрагмент JSON в манифест приложения веб-API (Surveys.WebAPI).</span><span class="sxs-lookup"><span data-stu-id="fa6fc-176">Repeat steps 3-6 to add the same JSON fragment to the application manifest of the web API (Surveys.WebAPI).</span></span>

8. <span data-ttu-id="fa6fc-177">Откройте окно PowerShell и выполните следующую команду, чтобы получить отпечаток сертификата:</span><span class="sxs-lookup"><span data-stu-id="fa6fc-177">From the PowerShell window, run the following command to get the thumbprint of the certificate.</span></span>

    ```powershell
    certutil -store -user my [subject]
    ```

    <span data-ttu-id="fa6fc-178">Для `[subject]` укажите значение, которое вы использовали для Subject в скрипте PowerShell.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-178">For `[subject]`, use the value that you specified for Subject in the PowerShell script.</span></span> <span data-ttu-id="fa6fc-179">Отпечаток указан в разделе Cert Hash(sha1).</span><span class="sxs-lookup"><span data-stu-id="fa6fc-179">The thumbprint is listed under "Cert Hash(sha1)".</span></span> <span data-ttu-id="fa6fc-180">Скопируйте это значение.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-180">Copy this value.</span></span> <span data-ttu-id="fa6fc-181">Отпечаток будет использоваться позднее.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-181">You will use the thumbprint later.</span></span>

### <a name="create-a-key-vault"></a><span data-ttu-id="fa6fc-182">Создайте хранилище ключей.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-182">Create a key vault</span></span>

1. <span data-ttu-id="fa6fc-183">Запустите скрипт PowerShell [/Scripts/Setup-KeyVault.ps1][Setup-KeyVault] следующим образом:</span><span class="sxs-lookup"><span data-stu-id="fa6fc-183">Run the PowerShell script [/Scripts/Setup-KeyVault.ps1][Setup-KeyVault] as follows:</span></span>

    ```powershell
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name>> -ResourceGroupName <<resource group name>> -Location <<location>>
    ```

    <span data-ttu-id="fa6fc-184">При появлении запроса на ввод учетных данных войдите в систему с учетной записью пользователя Azure AD, созданной ранее.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-184">When prompted for credentials, sign in as the Azure AD user that you created earlier.</span></span> <span data-ttu-id="fa6fc-185">Сценарий создаст новую группу ресурсов с новым хранилищем ключей.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-185">The script creates a new resource group, and a new key vault within that resource group.</span></span>

2. <span data-ttu-id="fa6fc-186">Повторно запустите Setup-KeyVault.ps1 следующим образом:</span><span class="sxs-lookup"><span data-stu-id="fa6fc-186">Run Setup-KeyVault.ps1 again as follows:</span></span>

    ```powershell
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name>> -ApplicationIds @("<<Surveys app id>>", "<<Surveys.WebAPI app ID>>")
    ```

    <span data-ttu-id="fa6fc-187">Установите следующие значения параметров:</span><span class="sxs-lookup"><span data-stu-id="fa6fc-187">Set the following parameter values:</span></span>

       * <span data-ttu-id="fa6fc-188">key vault name — имя, присвоенное хранилищу ключей на предыдущем этапе;</span><span class="sxs-lookup"><span data-stu-id="fa6fc-188">key vault name = The name that you gave the key vault in the previous step.</span></span>
       * <span data-ttu-id="fa6fc-189">Surveys app ID — идентификатор веб-приложения Surveys;</span><span class="sxs-lookup"><span data-stu-id="fa6fc-189">Surveys app ID = The application ID for the Surveys web application.</span></span>
       * <span data-ttu-id="fa6fc-190">Surveys.WebApi app ID — идентификатор приложения Surveys.WebAPI.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-190">Surveys.WebApi app ID = The application ID for the Surveys.WebAPI application.</span></span>

    <span data-ttu-id="fa6fc-191">Пример:</span><span class="sxs-lookup"><span data-stu-id="fa6fc-191">Example:</span></span>

    ```powershell
     .\Setup-KeyVault.ps1 -KeyVaultName tailspinkv -ApplicationIds @("f84df9d1-91cc-4603-b662-302db51f1031", "8871a4c2-2a23-4650-8b46-0625ff3928a6")
    ```

    <span data-ttu-id="fa6fc-192">Этот скрипт разрешает веб-приложению и веб-API получать секреты из хранилища ключей.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-192">This script authorizes the web app and web API to retrieve secrets from your key vault.</span></span> <span data-ttu-id="fa6fc-193">Дополнительные сведения см. в статье [Приступая к работе с хранилищем ключей Azure](/azure/key-vault/key-vault-get-started/).</span><span class="sxs-lookup"><span data-stu-id="fa6fc-193">See [Get started with Azure Key Vault](/azure/key-vault/key-vault-get-started/) for more information.</span></span>

### <a name="add-configuration-settings-to-your-key-vault"></a><span data-ttu-id="fa6fc-194">Добавьте параметры конфигурации в хранилище ключей.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-194">Add configuration settings to your key vault</span></span>

1. <span data-ttu-id="fa6fc-195">Запустите Setup-KeyVault.ps1 следующим образом:</span><span class="sxs-lookup"><span data-stu-id="fa6fc-195">Run Setup-KeyVault.ps1 as follows:</span></span>

    ```powershell
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name> -KeyName Redis--Configuration -KeyValue "<<Redis DNS name>>.redis.cache.windows.net,password=<<Redis access key>>,ssl=true"
    ```
    <span data-ttu-id="fa6fc-196">где:</span><span class="sxs-lookup"><span data-stu-id="fa6fc-196">where</span></span>

   * <span data-ttu-id="fa6fc-197">key vault name — имя, присвоенное хранилищу ключей на предыдущем этапе;</span><span class="sxs-lookup"><span data-stu-id="fa6fc-197">key vault name = The name that you gave the key vault in the previous step.</span></span>
   * <span data-ttu-id="fa6fc-198">Redis DNS name — DNS-имя экземпляра кэша Redis;</span><span class="sxs-lookup"><span data-stu-id="fa6fc-198">Redis DNS name = The DNS name of your Redis cache instance.</span></span>
   * <span data-ttu-id="fa6fc-199">Redis access key — ключ доступа для экземпляра кэша Redis.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-199">Redis access key = The access key for your Redis cache instance.</span></span>

2. <span data-ttu-id="fa6fc-200">На этом этапе рекомендуется проверить, успешно ли сохранены секреты в хранилище ключей.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-200">At this point, it's a good idea to test whether you successfully stored the secrets to key vault.</span></span> <span data-ttu-id="fa6fc-201">Выполните следующую команду PowerShell:</span><span class="sxs-lookup"><span data-stu-id="fa6fc-201">Run the following PowerShell command:</span></span>

    ```powershell
    Get-AzureKeyVaultSecret <<key vault name>> Redis--Configuration | Select-Object *
    ```

3. <span data-ttu-id="fa6fc-202">Повторно запустите Setup-KeyVault.ps1, чтобы добавить строку подключения к базе данных:</span><span class="sxs-lookup"><span data-stu-id="fa6fc-202">Run Setup-KeyVault.ps1 again to add the database connection string:</span></span>

    ```powershell
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name> -KeyName Data--SurveysConnectionString -KeyValue <<DB connection string>> -ConfigName "Data:SurveysConnectionString"
    ```

    <span data-ttu-id="fa6fc-203">где `<<DB connection string>>` — значение строки подключения к базе данных.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-203">where `<<DB connection string>>` is the value of the database connection string.</span></span>

    <span data-ttu-id="fa6fc-204">Для тестирования с локальной базой данных скопируйте строку подключения из файла Tailspin.Surveys.Web/appsettings.json.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-204">For testing with the local database, copy the connection string from the Tailspin.Surveys.Web/appsettings.json file.</span></span> <span data-ttu-id="fa6fc-205">При этом обязательно замените две обратные косые черты ("\\\\") одной обратной косой чертой.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-205">If you do that, make sure to change the double backslash ('\\\\') into a single backslash.</span></span> <span data-ttu-id="fa6fc-206">Двойная обратная косая черта является escape-символом в JSON-файле.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-206">The double backslash is an escape character in the JSON file.</span></span>

    <span data-ttu-id="fa6fc-207">Пример:</span><span class="sxs-lookup"><span data-stu-id="fa6fc-207">Example:</span></span>

    ```powershell
    .\Setup-KeyVault.ps1 -KeyVaultName mykeyvault -KeyName Data--SurveysConnectionString -KeyValue "Server=(localdb)\MSSQLLocalDB;Database=Tailspin.SurveysDB;Trusted_Connection=True;MultipleActiveResultSets=true"
    ```

### <a name="uncomment-the-code-that-enables-key-vault"></a><span data-ttu-id="fa6fc-208">Раскомментирование кода, который активирует хранилище ключей</span><span class="sxs-lookup"><span data-stu-id="fa6fc-208">Uncomment the code that enables Key Vault</span></span>

1. <span data-ttu-id="fa6fc-209">Откройте решение Tailspin.Surveys.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-209">Open the Tailspin.Surveys solution.</span></span>
2. <span data-ttu-id="fa6fc-210">В файле Tailspin.Surveys.Web/Startup.cs найдите следующий блок кода и раскомментируйте его:</span><span class="sxs-lookup"><span data-stu-id="fa6fc-210">In Tailspin.Surveys.Web/Startup.cs, locate the following code block and uncomment it.</span></span>

    ```csharp
    //var config = builder.Build();
    //builder.AddAzureKeyVault(
    //    $"https://{config["KeyVault:Name"]}.vault.azure.net/",
    //    config["AzureAd:ClientId"],
    //    config["AzureAd:ClientSecret"]);
    ```
3. <span data-ttu-id="fa6fc-211">В файле Tailspin.Surveys.Web/Startup.cs найдите блок кода, который регистрирует `ICredentialService`.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-211">In Tailspin.Surveys.Web/Startup.cs, locate the code that registers the `ICredentialService`.</span></span> <span data-ttu-id="fa6fc-212">Раскомментируйте строку, в которой используется `CertificateCredentialService`, и закомментируйте строку, в которой используется `ClientCredentialService`:</span><span class="sxs-lookup"><span data-stu-id="fa6fc-212">Uncomment the line that uses `CertificateCredentialService`, and comment out the line that uses `ClientCredentialService`:</span></span>

    ```csharp
    // Uncomment this:
    services.AddSingleton<ICredentialService, CertificateCredentialService>();
    // Comment out this:
    //services.AddSingleton<ICredentialService, ClientCredentialService>();
    ```

    <span data-ttu-id="fa6fc-213">Благодаря этому изменению веб-приложение сможет использовать [утверждение клиента][client-assertion] для получения маркера доступа OAuth.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-213">This change enables the web app to use [Client assertion][client-assertion] to get OAuth access tokens.</span></span> <span data-ttu-id="fa6fc-214">Утверждение клиента устраняет необходимость в секрете клиента OAuth.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-214">With client assertion, you don't need an OAuth client secret.</span></span> <span data-ttu-id="fa6fc-215">Либо же секрет клиента можно сохранить в хранилище ключей.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-215">Alternatively, you could store the client secret in key vault.</span></span> <span data-ttu-id="fa6fc-216">Однако и хранилище ключей, и утверждение клиента используют сертификат клиента, поэтому при активации хранилища ключей рекомендуется также включить утверждение клиента.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-216">However, key vault and client assertion both use a client certificate, so if you enable key vault, it's a good practice to enable client assertion as well.</span></span>

### <a name="update-the-user-secrets"></a><span data-ttu-id="fa6fc-217">Обновление секретов пользователя</span><span class="sxs-lookup"><span data-stu-id="fa6fc-217">Update the user secrets</span></span>

<span data-ttu-id="fa6fc-218">В обозревателе решений щелкните правой кнопкой мыши проект Tailspin.Surveys.Web и выберите пункт **Управление секретами пользователей**.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-218">In Solution Explorer, right-click the Tailspin.Surveys.Web project and select **Manage User Secrets**.</span></span> <span data-ttu-id="fa6fc-219">В файле secrets.json удалите существующий код JSON и вставьте следующий:</span><span class="sxs-lookup"><span data-stu-id="fa6fc-219">In the secrets.json file, delete the existing JSON and paste in the following:</span></span>

```json
{
  "AzureAd": {
    "ClientId": "[Surveys web app client ID]",
    "ClientSecret": "[Surveys web app client secret]",
    "PostLogoutRedirectUri": "https://localhost:44300/",
    "WebApiResourceId": "[App ID URI of your Surveys.WebAPI application]",
    "Asymmetric": {
      "CertificateThumbprint": "[certificate thumbprint. Example: 105b2ff3bc842c53582661716db1b7cdc6b43ec9]",
      "StoreName": "My",
      "StoreLocation": "CurrentUser",
      "ValidationRequired": "false"
    }
  },
  "KeyVault": {
    "Name": "[key vault name]"
  }
}
```

<span data-ttu-id="fa6fc-220">Замените записи в [квадратных скобках] надлежащими значениями.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-220">Replace the entries in [square brackets] with the correct values.</span></span>

* <span data-ttu-id="fa6fc-221">`AzureAd:ClientId`: идентификатор клиента приложения Surveys.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-221">`AzureAd:ClientId`: The client ID of the Surveys app.</span></span>
* <span data-ttu-id="fa6fc-222">`AzureAd:ClientSecret`: ключ, созданный при регистрации приложения Surveys в Azure AD.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-222">`AzureAd:ClientSecret`: The key that you generated when you registered the Surveys application in Azure AD.</span></span>
* <span data-ttu-id="fa6fc-223">`AzureAd:WebApiResourceId`: код URI идентификатора приложения, указанный при создании приложения Surveys.WebAPI в Azure AD.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-223">`AzureAd:WebApiResourceId`: The App ID URI that you specified when you created the Surveys.WebAPI application in Azure AD.</span></span>
* <span data-ttu-id="fa6fc-224">`Asymmetric:CertificateThumbprint`: отпечаток сертификата, полученный ранее при создании сертификата клиента.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-224">`Asymmetric:CertificateThumbprint`: The certificate thumbprint that you got previously, when you created the client certificate.</span></span>
* <span data-ttu-id="fa6fc-225">`KeyVault:Name`: имя хранилища ключей.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-225">`KeyVault:Name`: The name of your key vault.</span></span>

> [!NOTE]
> <span data-ttu-id="fa6fc-226">`Asymmetric:ValidationRequired` имеет значение False, так как созданный ранее сертификат не подписан корневым центром сертификации.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-226">`Asymmetric:ValidationRequired` is false because the certificate that you created previously was not signed by a root certificate authority (CA).</span></span> <span data-ttu-id="fa6fc-227">В рабочей среде используйте сертификат, подписанный центром сертификации, и измените значение `ValidationRequired` на True.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-227">In production, use a certificate that is signed by a root CA and set `ValidationRequired` to true.</span></span>

<span data-ttu-id="fa6fc-228">Сохраните обновленный файл secrets.json.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-228">Save the updated secrets.json file.</span></span>

<span data-ttu-id="fa6fc-229">Затем в обозревателе решений щелкните правой кнопкой мыши проект Tailspin.Surveys.WebApi и выберите пункт **Управление секретами пользователей**.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-229">Next, in Solution Explorer, right-click the Tailspin.Surveys.WebApi project and select **Manage User Secrets**.</span></span> <span data-ttu-id="fa6fc-230">Удалите существующий код JSON и вставьте следующий:</span><span class="sxs-lookup"><span data-stu-id="fa6fc-230">Delete the existing JSON and paste in the following:</span></span>

```json
{
  "AzureAd": {
    "ClientId": "[Surveys.WebAPI client ID]",
    "WebApiResourceId": "https://tailspin5.onmicrosoft.com/surveys.webapi",
    "Asymmetric": {
      "CertificateThumbprint": "[certificate thumbprint]",
      "StoreName": "My",
      "StoreLocation": "CurrentUser",
      "ValidationRequired": "false"
    }
  },
  "KeyVault": {
    "Name": "[key vault name]"
  }
}
```

<span data-ttu-id="fa6fc-231">Замените записи в [квадратных скобках] и сохраните файл secrets.json.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-231">Replace the entries in [square brackets] and save the secrets.json file.</span></span>

> [!NOTE]
> <span data-ttu-id="fa6fc-232">В веб-API обязательно должен использоваться идентификатор клиента для приложения Surveys.WebAPI, а не для приложения Surveys.</span><span class="sxs-lookup"><span data-stu-id="fa6fc-232">For the web API, make sure to use the client ID for the Surveys.WebAPI application, not the Surveys application.</span></span>

<span data-ttu-id="fa6fc-233">[**Далее**][adfs]</span><span class="sxs-lookup"><span data-stu-id="fa6fc-233">[**Next**][adfs]</span></span>

<!-- links -->

[adfs]: ./adfs.md
[authorize-app]: /azure/key-vault/key-vault-get-started//#authorize
[azure-portal]: https://portal.azure.com
[azure-rm-cmdlets]: https://msdn.microsoft.com/library/mt125356.aspx
[client-assertion]: client-assertion.md
[configuration]: /aspnet/core/fundamentals/configuration
[KeyVault]: https://azure.microsoft.com/services/key-vault/
[key-tags]: https://msdn.microsoft.com/library/azure/dn903623.aspx#BKMK_Keytags
[Microsoft.Azure.KeyVault]: https://www.nuget.org/packages/Microsoft.Azure.KeyVault/
[options]: /aspnet/core/fundamentals/configuration#using-options-and-configuration-objects
[readme]: ./run-the-app.md
[Setup-KeyVault]: https://github.com/mspnp/multitenant-saas-guidance/blob/master/scripts/Setup-KeyVault.ps1
[Surveys]: tailspin.md
[user-secrets]: /aspnet/core/security/app-secrets
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
