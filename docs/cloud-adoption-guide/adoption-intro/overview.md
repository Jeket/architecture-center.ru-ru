---
title: Внедрение Azure. Базовый уровень
description: Описание базовых понятий, которые организация должна знать для внедрения Azure
author: petertay
ms.openlocfilehash: 3f522d1662849d651423d8022ad152c64692b823
ms.sourcegitcommit: b3d74d8a89b2224fc796ce0e89cea447af43a0d4
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/11/2018
ms.locfileid: "35290481"
---
# <a name="adopting-azure-foundational"></a>Внедрение Azure. Базовый уровень

Внедрение Azure является первым этапом зрелости организации. К концу этого этапа сотрудники вашей организации смогут развернуть простые рабочие нагрузки в Azure.

В списке ниже приводятся задачи для прохождения базового этапа внедрения. В списке задачи представлены в определенной последовательности, поэтому выполняйте их по порядку. Если до этого вы выполнили задачу, переходите к следующей задаче в списке. 

1. Основные сведения о внутренних компонентах Azure.
    - **Пояснения.** [Принцип работы Azure](azure-explainer.md)
    - **Пояснения.** [Знакомство с системой управления облачными ресурсами](governance-explainer.md)
2. Основные сведения о цифровых удостоверениях организации в Azure.
    - **Пояснения.** [Знакомство с клиентом Azure Active Directory](tenant-explainer.md)
    - **Инструкции.** [Получение клиента Azure Active Directory](/azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - **Руководство.** [Разработка клиента Azure AD](tenant.md)
    - **Инструкции.** [Добавление новых пользователей в Azure Active Directory](/azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json)    
3. Общие сведения о подписках в Azure.
    - **Пояснения.** [Знакомство с подпиской Azure](subscription-explainer.md)
    - **Руководство.** [Разработка подписки Azure](subscription.md)
4. Основные сведения об управлении ресурсами в Azure. 
    - **Пояснения.** [Знакомство с Azure Resource Manager](resource-manager-explainer.md)
    - **Пояснения.** [Знакомство с группой ресурсов Azure](resource-group-explainer.md)
    - **Пояснения.** [Основные сведения о доступе к ресурсам в Azure](/azure/active-directory/active-directory-understanding-resource-access?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - **Инструкции.** [Создание группы ресурсов Azure с помощью портала Azure](/azure/azure-resource-manager/resource-group-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - **Руководство.** [Разработка группы ресурсов Azure](resource-group.md)
    - **Руководство.** [Соглашения об именовании для ресурсов Azure](/azure/architecture/best-practices/naming-conventions?toc=/azure/architecture/cloud-adoption-guide/toc.json)
5. Развертывание базовой архитектуры Azure.
    - Узнайте о различных типах вычислительных служб Azure, например инфраструктура как услуга (IaaS) и платформа как услуга (PaaS), в статье [Обзор вычислительных служб в Azure](/azure/architecture/guide/technology-choices/compute-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json).
    - Когда вы ознакомитесь с различными типами вычислительных служб Azure, выберите веб-приложение (PaaS) или виртуальную машину (IaaS) в качестве первого ресурса в Azure.
    - PaaS: общие сведения о платформе как услуге в Azure.
        - **Инструкции.** [Развертывание базового веб-приложения в Azure](/azure/app-service/app-service-web-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **Руководство.** Проверенные методики по развертыванию [основного веб-приложения](/azure/architecture/reference-architectures/app-service-web-app/basic-web-app?toc=/azure/architecture/cloud-adoption-guide/toc.json) в Azure
    - IaaS: введение в виртуальную сеть.
        - **Пояснения.** [Виртуальная сеть Azure](/azure/virtual-network/virtual-networks-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **Инструкции.** [Развертывание виртуальной сети в Azure с помощью портала](/azure/virtual-network/virtual-networks-create-vnet-arm-pportal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - IaaS. Развертывание рабочей нагрузки на одной виртуальной машине (Windows и Linux).
        - **Инструкции.** [Развертывание виртуальной машины Windows в Azure с помощью портала](/azure/virtual-machines/windows/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **Руководство.** [Проверенные методики по запуску виртуальных машин Windows в Azure](/azure/architecture/reference-architectures/virtual-machines-windows/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **Инструкции.** [Развертывание виртуальной машины Linux в Azure с помощью портала](/azure/virtual-machines/linux/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **Руководство.** [Проверенные методики по запуску виртуальных машин Linux в Azure](/azure/architecture/reference-architectures/virtual-machines-linux/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json)
