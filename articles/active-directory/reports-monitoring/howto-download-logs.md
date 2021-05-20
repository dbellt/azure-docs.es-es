---
title: Descarga de registros en Azure Active Directory | Microsoft Docs
description: Aprenda a descargar registros de actividad en Azure Active Directory.
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: mtillman
editor: ''
ms.service: active-directory
ms.topic: how-to
ms.workload: identity
ms.subservice: report-monitor
ms.date: 05/05/2021
ms.author: markvi
ms.reviewer: besiler
ms.collection: M365-identity-device-management
ms.openlocfilehash: e1c23f5ee8472b889929c513199a46022b6eaa3c
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108776984"
---
# <a name="how-to-download-logs-in-azure-active-directory"></a>Descarga de registros en Azure Active Directory

El portal de Azure Active Directory (Azure AD) ofrece acceso a tres tipos de registros de actividad:

- **[Inicios de sesión](concept-sign-ins.md)** : Información sobre los inicios de sesión y cómo los usuarios emplean los recursos.
- **[Auditoría](concept-audit-logs.md)** : información sobre los cambios aplicados al inquilino, como la administración de usuarios y grupos o las actualizaciones aplicadas a los recursos del inquilino.
- **[Aprovisionamiento](concept-provisioning-logs.md)** : actividades realizadas por el servicio de aprovisionamiento, como la creación de un grupo en ServiceNow o un usuario importado de Workday.

Azure AD almacena los datos en estos registros durante un período de tiempo limitado. Como administrador de TI, puede descargar los registros de actividad para realizar una copia de seguridad a largo plazo.

En este artículo, se explica cómo descargar registros de actividad en Azure AD.  

## <a name="what-you-should-know"></a>Qué debería saber

- En el portal de Azure AD, puede encontrar varios puntos de entrada a los registros de actividad. Por ejemplo, la sección **Actividad** de la página [Usuarios](https://portal.azure.com/#blade/Microsoft_AAD_IAM/UsersManagementMenuBlade/MsGraphUsers) o [grupos](https://portal.azure.com/#blade/Microsoft_AAD_IAM/GroupsManagementMenuBlade/AllGroups). Sin embargo, solo hay una ubicación que proporciona una vista inicial sin filtrar de los registros: la sección **Supervisión** de la página [Azure AD](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview).    

- Azure AD solo almacena los registros de actividad durante un período específico. Para obtener más información, vea [¿Durante cuánto tiempo Azure AD almacena los datos de informes?](reference-reports-data-retention.md) 

- Al descargar los registros, puede controlar durante cuánto tiempo se almacenan los registros. 

- Puede descargar hasta 250 000 registros. Si desea descargar más datos, use la API de generación de informes.

- La descarga se basa en el filtro establecido. 

- Azure AD admite los siguientes formatos para la descarga:

    - **CSV** 

    - **JSON** 

- Las marcas de tiempo de los archivos descargados siempre se basan en UTC.



## <a name="what-license-do-you-need"></a>¿Qué licencia necesita?

La opción para descargar los datos de un registro de actividad está disponible en todas las ediciones de Azure AD.


## <a name="who-can-do-it"></a>¿Quién puede hacerlo?

Mientras el administrador global funciona, debe usar una cuenta con menos privilegios para realizar esta tarea. Para acceder a los registros de auditoría, sirven los siguientes roles:

- Lector de informes
- Lector global
- Administrador de seguridad
- Lector de seguridad


## <a name="how-to-do-it"></a>Cómo hacerlo


**Para descargar un registro de actividad**:

1. Vaya a la vista del registro de actividad que le interesa:
 
    - [Registro de inicios de sesión](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/SignIns)
    
    - [Registro de auditoría](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/SignIns)    
       
    - [Registro de aprovisionamiento](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/ProvisioningEvents)    
   

2.  **Agregue** el filtro necesario.  

    ![Adición de filtro](./media/\howto-download-logs/add-filter.png)    

3. **Descargue** los datos.

    ![Descargar registro](./media/\howto-download-logs/download-log.png)

## <a name="next-steps"></a>Pasos siguientes

- [Registros de inicios de sesión en Azure AD](concept-sign-ins.md)
- [Registros de auditoría en Azure AD](concept-audit-logs.md)