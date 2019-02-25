---
title: CAF. Средства основных способов идентификации в Azure
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Средства основных способов идентификации в Azure
author: BrianBlanchard
ms.openlocfilehash: 81b0fa9cfee597da98d8b983fb155eac82d97bf8
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55902452"
---
# <a name="identity-baseline-tools-in-azure"></a>Средства основных способов идентификации в Azure

[Основные способы идентификации](overview.md) — это одна из [пяти дисциплин системы управления облаком](../governance-disciplines.md). Эта дисциплина посвящена способам создания политик, гарантирующих согласованность и непрерывность идентификаторов пользователей, независимо от поставщика облачных служб, который размещает приложение или рабочую нагрузку.

В руководство по обнаружению гибридного удостоверения включены следующие средства.

**Доменные службы Active Directory (локальные)**. Доменные службы Active Directory — это поставщик удостоверений, наиболее часто используемый на предприятии для хранения и проверки учетных данных пользователя.

**Azure Active Directory:** Программное обеспечение как услуга (SaaS), эквивалентное Active Directory, поддерживает федерацию с локальными доменными службами Active Directory.

**Доменные службы Active Directory (IaaS)**. Экземпляр приложения доменных служб Active Directory, запущенных на виртуальной машине в Azure.

Идентификатор представляет собой новую плоскость управления в сфере ИТ-безопасности. Поэтому проверка подлинности — это защитник доступа к организации в облаке. Организации должны определить плоскость управления удостоверениями, которая повышает безопасность и защищает облачные приложения от несанкционированного доступа.

## <a name="cloud-authentication"></a>Облачная аутентификация

Выбор правильного метода аутентификации является первой задачей для организаций, которые хотят переместить свои приложения в облако.

При выборе этого метода Azure AD обрабатывает процесс входа пользователей. В сочетании с простым единым входом (SSO) пользователи могут войти в облачные приложения без необходимости повторного ввода своих учетных данных. При использовании облачной аутентификации вы можете выбрать один из двух вариантов:

**Синхронизация хэша паролей Azure AD**. Это самый простой способ включить аутентификацию объектов локального каталога в Azure AD. Этот метод также можно использовать с любым методом в качестве резервного метода проверки подлинности в случае отказа локального сервера.

**Сквозная проверка подлинности Azure AD**. Обеспечивает постоянную проверку пароля для служб проверки подлинности Azure AD с помощью программного агента, выполняемого на одном или нескольких локальных серверах.

> [!NOTE]
> Этот метод сквозной проверки подлинности следует использовать организациям с требованием безопасности, предписывающим немедленное применение состояний локальных учетных записей пользователей, политик паролей и времени входа.

**Федеративная проверка подлинности**.

При выборе этого метода Azure AD передает процесс проверки подлинности отдельной доверенной системе проверки подлинности, например локальным службам федерации Active Directory (AD FS) или доверенному стороннему поставщику федерации, для проверки пароля пользователя.

Статья [Выбор правильного метода аутентификации для гибридного решения для идентификации Azure Active Directory](/azure/security/azure-ad-choose-authn) содержит дерево принятия решений, которое поможет вам выбрать лучшее решение для вашей организации.

Ниже указана таблица со списком собственных средств Azure, которые могут помочь в совершенствовании политик и процессов, поддерживающих эту дисциплину управления.

<!-- markdownlint-disable MD033 -->

|Рассматриваемый вопрос|Синхронизация хэша паролей и простой единый вход|Сквозная аутентификация и простой единый вход|Федерация с AD FS|
|:-----|:-----|:-----|:-----|
|Где происходит аутентификация?|В облаке|В облаке после безопасного обмена данными проверки пароля с локальным агентом аутентификации|Локальная система|
|Каковы требования к локальному серверу, помимо системы подготовки: Azure AD Connect?|Нет|Один сервер для каждого дополнительного агента аутентификации|Не менее двух серверов AD FS<br><br>Не менее двух WAP-серверов в сети периметра|
|Каковы локальные требования к доступу к Интернету и сетевым подключениям помимо системы подготовки?|Нет|[Исходящий доступ к Интернету](/azure/active-directory/hybrid/how-to-connect-pta-quick-start) с серверов, где работают агенты аутентификации|[Входящий доступ к Интернету](/windows-server/identity/ad-fs/overview/ad-fs-requirements) к WAP-серверам в сети периметра<br><br>Входящий сетевой доступ к серверам AD FS с WAP-серверов в сети периметра<br><br>Балансировка сетевой нагрузки|
|Существует ли требование к SSL-сертификату?|Нет |Нет |Yes|
|Имеется ли решение по наблюдению за работоспособностью?|Не требуется|Состояние агента, предоставляемое [Центром администрирования Azure Active Directory](/azure/active-directory/hybrid/tshoot-connect-pass-through-authentication)|[Azure AD Connect Health](/azure/active-directory/hybrid/how-to-connect-health-adfs)|
|Пользователи получают единый вход в облачные ресурсы с присоединенных к домену устройств в корпоративной сети?|Да, с [простым единым входом](/azure/active-directory/hybrid/how-to-connect-sso)|Да, с [простым единым входом](/azure/active-directory/hybrid/how-to-connect-sso)|Yes|
|Какие типы входа поддерживаются?|UserPrincipalName и пароль<br><br>Встроенная проверка подлинности Windows с [простым единым входом](/azure/active-directory/hybrid/how-to-connect-sso)<br><br>[Альтернативный идентификатор входа](/azure/active-directory/hybrid/how-to-connect-install-custom)|UserPrincipalName и пароль<br><br>Встроенная проверка подлинности Windows с [простым единым входом](/azure/active-directory/hybrid/how-to-connect-sso)<br><br>[Альтернативный идентификатор входа](/azure/active-directory/hybrid/how-to-connect-pta-faq)|UserPrincipalName и пароль<br><br>sAMAccountName и пароль<br><br>Встроенная проверка подлинности Windows<br><br>[Аутентификация с использованием сертификатов и смарт-карт](/windows-server/identity/ad-fs/operations/configure-user-certificate-authentication)<br><br>[Альтернативный идентификатор входа](/windows-server/identity/ad-fs/operations/configuring-alternate-login-id)|
|Поддерживается ли Windows Hello для бизнеса?|[Модель доверия на основе ключей](/windows/security/identity-protection/hello-for-business/hello-identity-verification)<br><br>[Модель доверия на основе сертификатов с Intune](https://blogs.technet.microsoft.com/microscott/setting-up-windows-hello-for-business-with-intune/)|[Модель доверия на основе ключей](/windows/security/identity-protection/hello-for-business/hello-identity-verification)<br><br>[Модель доверия на основе сертификатов с Intune](https://blogs.technet.microsoft.com/microscott/setting-up-windows-hello-for-business-with-intune/)|[Модель доверия на основе ключей](/windows/security/identity-protection/hello-for-business/hello-identity-verification)<br><br>[Модель доверия на основе сертификатов](/windows/security/identity-protection/hello-for-business/hello-key-trust-adfs)|
|Какие варианты многофакторной проверки подлинности существуют?|[Многофакторная идентификация Azure](/azure/multi-factor-authentication/)<br><br>[Пользовательские элементы управления для функции условного доступа*](/azure/active-directory/conditional-access/controls#custom-controls-1)|[Многофакторная идентификация Azure](/azure/multi-factor-authentication/)<br><br>[Пользовательские элементы управления для функции условного доступа*](/azure/active-directory/conditional-access/controls#custom-controls-1)|[Многофакторная идентификация Azure](/azure/multi-factor-authentication/)<br><br>[Сервер Azure MFA](/azure/active-directory/authentication/howto-mfaserver-deploy)<br><br>[Стороннее решение многофакторной идентификации](/windows-server/identity/ad-fs/operations/configure-additional-authentication-methods-for-ad-fs)<br><br>[Пользовательские элементы управления для функции условного доступа*](/azure/active-directory/conditional-access/controls#custom-controls-1)|
|Какие состояния учетной записи пользователя поддерживаются?|Отключенные учетные записи<br>(до 30-минутной задержки)|Отключенные учетные записи<br><br>Учетная запись заблокирована<br><br>Срок действия учетной записи истек<br><br>Срок действия пароля истек<br><br>Время входа|Отключенные учетные записи<br><br>Учетная запись заблокирована<br><br>Срок действия учетной записи истек<br><br>Срок действия пароля истек<br><br>Время входа|
|Какие варианты условного доступа поддерживаются?|[Условный доступ Azure AD](/azure/active-directory/active-directory-conditional-access-azure-portal)|[Условный доступ Azure AD](/azure/active-directory/active-directory-conditional-access-azure-portal)|[Условный доступ Azure AD](/azure/active-directory/active-directory-conditional-access-azure-portal)<br><br>[Правила утверждений AD FS](https://adfshelp.microsoft.com/AadTrustClaims/ClaimsGenerator)|
|Поддерживается ли блокировка устаревших протоколов?|[Да](/azure/active-directory/active-directory-conditional-access-conditions#legacy-authentication)|[Да](/azure/active-directory/active-directory-conditional-access-conditions#legacy-authentication)|[Да](/windows-server/identity/ad-fs/operations/access-control-policies-w2k12)|
|Можно ли настроить логотип, изображение и описание на страницах входа?|[Да, в Azure AD Premium](/azure/active-directory/customize-branding)|[Да, в Azure AD Premium](/azure/active-directory/customize-branding)|[Да](/azure/active-directory/connect/active-directory-aadconnect-federation-management#customlogo)|
|Какие дополнительные сценарии поддерживаются?|[Интеллектуальная блокировка паролей](/azure/active-directory/active-directory-secure-passwords)<br><br>[Отчеты об утерянных учетных данных](/azure/active-directory/active-directory-reporting-risk-events)|[Интеллектуальная блокировка паролей](/azure/active-directory/connect/active-directory-aadconnect-pass-through-authentication-smart-lockout)|Система аутентификации нескольких сайтов с низкой задержкой<br><br>[Блокировка экстрасети AD FS](/windows-server/identity/ad-fs/operations/configure-ad-fs-extranet-soft-lockout-protection)<br><br>[Интеграция со сторонними системами идентификации](/azure/active-directory/connect/active-directory-aadconnect-federation-compatibility)|

<!-- markdownlint-enable MD033 -->

> [!NOTE]
> Пользовательские элементы управления в компоненте условного доступа Azure AD сейчас не поддерживают регистрацию устройств.

## <a name="next-steps"></a>Дополнительная информация

[Гибридная платформа цифрового преобразования для идентификации](https://resources.office.com/ww-landing-M365E-EMS-IDAM-Hybrid-Identity-WhitePaper.html?LCID=EN-US) описывает ряд комбинаций и решений для выбора и интеграции каждого из этих компонентов.

Средство [Azure AD Connect](https://aka.ms/aadconnectwiz) позволяет интегрировать локальные каталоги с Azure AD.
