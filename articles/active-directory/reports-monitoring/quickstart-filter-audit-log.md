---
title: Filtrado del registro de auditoría de Azure AD
description: En este inicio rápido, aprenderá a filtrar las entradas del registro de auditoría de Azure AD.
services: active-directory
ms.service: active-directory
ms.subservice: report-monitor
ms.topic: quickstart
ms.date: 06/11/2021
ms.author: markvi
author: MarkusVi
manager: mtillman
ms.reviewer: besiler
ms.collection: M365-identity-device-management
ms.openlocfilehash: c684db974dfd939287cd15d63079693b4cf1d668
ms.sourcegitcommit: 190658142b592db528c631a672fdde4692872fd8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112008175"
---
# <a name="quickstart-filter-your-azure-ad-audit-log"></a>Inicio rápido: Filtrado del registro de auditoría de Azure AD 

Con la información de los registros de auditoría de Azure AD, obtiene acceso a registros de actividades del sistema de cumplimiento. En este inicio rápido se muestra cómo encontrar una cuenta de usuario recién creada en el registro de auditoría.


## <a name="prerequisites"></a>Requisitos previos

Para completar el escenario en este inicio rápido, necesita:

- **Acceso a un inquilino de Azure AD**: si no tiene acceso a uno, consulte [Cree su cuenta de Azure gratuita hoy mismo](https://azure.microsoft.com/free/?WT.mc_id=A261C142F). 
- **Una cuenta de prueba denominada Isabella Simonsen**: si no sabe cómo crear una cuenta de prueba, consulte [Agregar usuarios basados en la nube](../fundamentals/add-users-azure-active-directory.md#add-a-new-user).

## <a name="find-the-new-user-account"></a>Búsqueda de la nueva cuenta de usuario

En esta sección se proporcionan los pasos para filtrar el registro de auditoría.


**Para buscar el nuevo usuario:**

1. Vaya al [registro de auditoría](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Audit).

2. Para mostrar solo los registros de Isabella Simonsen:

    a. En la barra de herramientas, haga clic en **Agregar filtros**.
    
    ![Adición de un filtro de usuario](./media/quickstart-analyze-sign-in/add-filters.png)   

    b. En la lista **Elija un campo**, seleccione **Destino** y, luego, haga clic en **Aplicar**.

    c. En el cuadro de texto **Destino**, en **Nombre principal de usuario**, escriba **Isabella Simonsen** y, luego, haga clic en **Aplicar**.

3. Haga clic en el elemento filtrado.

    ![Elementos filtrados](./media/quickstart-filter-audit-log/audit-log-list.png)  

4.  Revise los **detalles del registro de auditoría**.
 
    ![Detalles del registro de auditoría](./media/quickstart-filter-audit-log/audit-log-details.png)  
 
  

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no lo necesite, elimine el usuario de prueba. Si no sabe cómo eliminar un usuario de Azure AD, consulte [Eliminación de usuarios desde Azure AD](../fundamentals/add-users-azure-active-directory.md#delete-a-user).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [¿Qué son los informes de Azure Active Directory?](overview-reports.md)
