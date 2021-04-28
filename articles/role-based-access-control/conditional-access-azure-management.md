---
title: Administración el acceso a la administración de Azure con acceso condicional en Azure AD
description: Obtenga más información sobre el uso del acceso condicional en Azure AD para administrar el acceso a la administración de Azure.
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
editor: skwan
ms.assetid: 0adc8b11-884e-476c-8c43-84f9bf12a34b
ms.service: role-based-access-control
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 07/15/2019
ms.author: rolyon
ms.reviewer: skwan
ms.openlocfilehash: 547dd3f039f2205d5d83c4b5b5f6105482f94d87
ms.sourcegitcommit: 5f785599310d77a4edcf653d7d3d22466f7e05e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108065050"
---
# <a name="manage-access-to-azure-management-with-conditional-access"></a>Administración el acceso a la administración de Azure con acceso condicional

> [!CAUTION]
> Asegúrese de comprender cómo funciona el acceso condicional antes de configurar una directiva para administrar el acceso a la administración de Azure. Asegúrese de no crear condiciones que pudieran bloquear su propio acceso al portal.

El acceso condicional en Azure Active Directory (Azure AD) controla el acceso a las aplicaciones en la nube en función de las condiciones concretas que especifique. Para permitir el acceso, cree directivas de acceso condicional que permitan o bloqueen el acceso en función de si se cumplen los requisitos de la directiva. 

Normalmente, se utiliza el acceso condicional para controlar el acceso a las aplicaciones en la nube. También puede configurar directivas para controlar el acceso a la administración de Azure en función de ciertas condiciones (como el riesgo de inicio de sesión, la ubicación o el dispositivo) y para exigir requisitos como la autenticación multifactor.

Para crear una directiva de administración de Azure, seleccione **Microsoft Azure Management** en **Aplicaciones en la nube** al elegir la aplicación a la que se va a aplicar la directiva.

![Acceso condicional para la administración de Azure](./media/conditional-access-azure-management/conditional-access-azure-mgmt.png)

La Directiva que se crea se aplica a todos los puntos de conexión de administración de Azure, incluidos los siguientes:

- Azure Portal
- Proveedor de Azure Resource Manager
- Service Management API clásicas
- Azure PowerShell
- Portal de administrador de suscripciones de Visual Studio
- Azure DevOps
- Portal de Azure Data Factory
- [Azure SQL Database](https://docs.microsoft.com/azure/azure-sql/database/conditional-access-configure)
- Instancia administrada de SQL
- Azure Synapse

Tenga en cuenta que la directiva se aplica a Azure PowerShell, que llama a la API de Azure Resource Manager. No es aplicable a [Azure AD PowerShell](/powershell/azure/active-directory/install-adv2), que llama a Microsoft Graph.

Para obtener más información sobre cómo configurar una directiva de ejemplo a fin de habilitar el acceso condicional para la administración de Microsoft Azure, consulte el artículo [Acceso condicional: Exigir autenticación multifactor para la administración de Azure](../active-directory/conditional-access/howto-conditional-access-policy-azure-management.md)
