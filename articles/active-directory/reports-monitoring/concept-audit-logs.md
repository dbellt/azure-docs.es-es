---
title: Registros de auditoría en Azure Active Directory | Microsoft Docs
description: Información general de los registros de auditoría en Azure Active Directory.
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: mtillman
editor: ''
ms.assetid: a1f93126-77d1-4345-ab7d-561066041161
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 04/26/2021
ms.author: markvi
ms.reviewer: besiler
ms.collection: M365-identity-device-management
ms.openlocfilehash: 7da93343e7380a327b7d2586b90dd5a5df1e006f
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108136866"
---
# <a name="audit-logs-in-azure-active-directory"></a>Registros de auditoría en Azure Active Directory 

Como administrador de TI, quiere saber cómo funciona el entorno de TI. La información sobre el estado del sistema le permite evaluar si es necesario responder a posibles problemas y cómo hacerlo. 

Para ayudarle a conseguir este objetivo, el portal de Azure Active Directory le proporciona acceso a tres registros de actividad:

- **[Inicios de sesión](concept-sign-ins.md)** : Información sobre los inicios de sesión y cómo los usuarios emplean los recursos.
- **[Auditoría](concept-audit-logs.md)** : información sobre los cambios aplicados al inquilino, como la administración de usuarios y grupos o las actualizaciones aplicadas a los recursos del inquilino.
- **[Aprovisionamiento](concept-provisioning-logs.md)** : actividades realizadas por el servicio de aprovisionamiento, como la creación de un grupo en ServiceNow o un usuario importado de Workday.

En este artículo se proporciona información general de los informes de auditoría.


## <a name="what-can-you-do-with-it"></a>¿Para qué sirven el informe?

Con los registros de auditoría de Azure AD, puede obtener registros de las actividades del sistema de cara al cumplimiento.
Las vistas más comunes de este registro se basan en las siguientes categorías:

- Administración de usuarios

- Administración de grupos
 
- Administración de aplicaciones  


Con una vista centrada en usuarios o grupos, puede obtener respuestas a preguntas como:

- ¿Qué tipos de actualizaciones se han aplicado a los usuarios?

- ¿Cuántos usuarios han cambiado?

- ¿Cuántas contraseñas han cambiado?

- ¿Qué ha hecho un administrador en un directorio?

- ¿Cuáles son los grupos que se han agregado?

- ¿Hay grupos con cambios de pertenencia?

- ¿Se han cambiado los propietarios del grupo?

- ¿Qué licencias se han asignado a un grupo o un usuario?


Con una vista centrada en la aplicación, puede obtener respuestas a preguntas como:

- ¿Qué aplicaciones se han agregado o actualizado?

- ¿Qué aplicaciones se han quitado?

- ¿Ha cambiado la entidad de servicio de una aplicación?

- ¿Se han cambiado los nombres de las aplicaciones?

- ¿Quién dio el consentimiento a una aplicación?

 
## <a name="who-can-access-it"></a>¿Quién puede acceder a ellos?

Para acceder a los registros de auditoría, debe tener uno de los siguientes roles: 

- Administrador de seguridad
- Lector de seguridad
- Lector de informes
- Lector global
- Administrador global

## <a name="where-can-you-find-it-in-the-azure-portal"></a>¿Dónde puede encontrarlo en Azure Portal?

Azure Portal ofrece varias opciones para acceder al registro. Por ejemplo, en el menú Azure Active Directory, puede abrir el registro en la sección **Supervisión**.  

![Apertura de los registros de aprovisionamiento](./media/concept-provisioning-logs/provisioning-logs-menu.png)

Además, puede acceder directamente a los registros de auditoría mediante este vínculo: [https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/ProvisioningEvents](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/ProvisioningEvents).


## <a name="what-is-the-default-view"></a>¿Cuál es la vista predeterminada?

Un registro de auditoría tiene una vista de lista predeterminada que muestra:

- la fecha y hora de la repetición
- el servicio que ha registrado la repetición
- la categoría y el nombre de la actividad (*qué*) 
- el estado de la actividad (correcto o incorrecto)
- el destino
- el iniciador o actor (quién) de una actividad

![Registros de auditoría](./media/concept-audit-logs/listview.png "Registros de auditoría")

Puede personalizar la vista de lista, haga clic en **Columnas** en la barra de herramientas.

![Columnas de auditoría](./media/concept-audit-logs/columns.png "Columnas de auditoría")

Esto le permite mostrar los campos adicionales o quitar los campos que ya se están mostrando.

![Quitar campos](./media/concept-audit-logs/columnselect.png "Quitar campos")

Seleccione un elemento de la vista de lista para obtener información más detallada.

![seleccionar elemento](./media/concept-audit-logs/details.png "Seleccionar elemento")


## <a name="filtering-audit-logs"></a>Filtrado de registros de auditoría

Puede filtrar los datos de auditoría por los siguientes campos:

- Servicio
- Category
- Actividad
- Status
- Destino
- Iniciado por (actor)
- Intervalo de fechas

![Objeto de filtro](./media/concept-audit-logs/filter.png "Filter, objeto")

El filtro **Service** (Servicio) le permite seleccionar los siguientes servicios de una lista desplegable:

- All
- Experiencia de usuario de administración de AAD
- Revisiones de acceso
- Account Provisioning (Aprovisionamiento de cuentas)
- Proxy de aplicación
- Métodos de autenticación
- B2C
- Acceso condicional
- Core Directory (Directorio principal)
- Administración de derechos
- Autenticación híbrida
- Protección de identidad
- Invited Users (Usuarios invitados)
- MIM Service (Servicio MIM)
- MyApps
- PIM
- Self-service Group Management (Administración de grupos de autoservicio)
- Self-service Password Management (Administración de contraseñas de autorservicio)
- Términos de uso

El filtro **Category** (Categoría) le permite seleccionar uno de los filtros siguientes:

- All
- AdministrativeUnit
- ApplicationManagement
- Authentication
- Authorization
- Contacto
- Dispositivo
- DeviceConfiguration
- DirectoryManagement
- EntitlementManagement
- GroupManagement
- KerberosDomain
- KeyManagement
- Etiqueta
- Otros
- PermissionGrantPolicy
- Directiva
- ResourceManagement
- RoleManagement
- UserManagement

El filtro **Activity** (Actividad) se basa en la selección de la categoría y el tipo de recurso de actividad que realice. Puede seleccionar la actividad específica que desea ver o elegir todas. 

Puede obtener la lista de todas las actividades de auditoría mediante Graph API: `https://graph.windows.net/<tenantdomain>/activities/auditActivityTypesV2?api-version=beta`

El filtro **Status** (Estado) le permite filtrar en función del estado de una operación de auditoría. El estado puede ser uno de los siguientes:

- All
- Correcto
- Error

El filtro **Target** (Destino) le permite buscar un destino determinado por nombre o nombre principal de usuario (UPN). El nombre de destino y el UPN distinguen mayúsculas de minúsculas. 

El filtro **Iniciado por** (Initiated by) le permite definir por qué empieza el nombre de un actor o un nombre principal universal (UPN). El nombre y el UPN distinguen mayúsculas de minúsculas.

El filtro **Date range** (Intervalo de fechas) permite definir un período de tiempo para los datos devueltos.  
Los valores posibles son:

- 7 días
- 24 horas
- Personalizado

Cuando se selecciona un intervalo de tiempo personalizado, puede configurar una hora de inicio y una hora de finalización.

También puede descargar los datos filtrados, hasta 250 000 registros, si selecciona el botón **Download** (Descargar). Puede descargar los registros en formato CSV o JSON. El número de registros que se puede descargar también está restringido por las [directivas de retención de informes de Azure Active Directory](reference-reports-data-retention.md).

![Descarga de datos](./media/concept-audit-logs/download.png "Descarga de datos")




## <a name="microsoft-365-activity-logs"></a>Registros de actividad de Microsoft 365

Puede ver los registros de actividad de Microsoft 365 desde el [Centro de administración de Microsoft 365](/office365/admin/admin-overview/about-the-admin-center). Aunque los registros de actividad de Microsoft 365 y Azure AD comparten muchos de los recursos del directorio, solo el Centro de administración de Microsoft 365 proporciona una vista completa de los registros de actividad de Microsoft 365. 

También puede acceder a los registros de actividad de Microsoft 365 mediante programación con las [API de administración de Office 365](/office/office-365-management-api/office-365-management-apis-overview).

## <a name="next-steps"></a>Pasos siguientes

- [Referencia sobre actividades de auditoría de Azure AD](reference-audit-activities.md)
- [Referencia sobre la retención de registros de Azure AD](reference-reports-data-retention.md)
- [Referencia sobre las latencias de registro de Azure AD](reference-reports-latencies.md)
- [Actores desconocidos en el informe de auditoría](/troubleshoot/azure/active-directory/unknown-actors-in-audit-reports)