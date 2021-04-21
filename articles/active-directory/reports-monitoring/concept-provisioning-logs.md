---
title: Información general sobre los registros de aprovisionamiento en Azure Portal (versión preliminar) | Microsoft Docs
description: Obtenga una introducción a los informes de registros de aprovisionamiento en Azure Active Directory a través de Azure Portal.
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: 4b18127b-d1d0-4bdc-8f9c-6a4c991c5f75
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 1/29/2021
ms.author: markvi
ms.reviewer: arvinh
ms.collection: M365-identity-device-management
ms.openlocfilehash: 468e885bab6aab4becb5aaaec7b4d52ce5ef5e07
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107535989"
---
# <a name="overview-of-provisioning-logs-in-the-azure-portal-preview"></a>Información general sobre los registros de aprovisionamiento en Azure Portal (versión preliminar)

La arquitectura de los informes de Azure Active Directory (Azure AD) consta de los siguientes componentes:

- Actividad: 
    - **Inicios de sesión**: información sobre el uso de aplicaciones administradas y actividades de inicio de sesión de usuario.
    - [Registros de auditoría](concept-audit-logs.md): información de la actividad del sistema sobre los usuarios y la administración de grupos, sus aplicaciones administradas y actividades de directorio.
    - **Registros de aprovisionamiento**: la actividad del sistema sobre los usuarios, los grupos y los roles aprovisionados por el servicio de aprovisionamiento de Azure AD. 

- Seguridad: 
    - **Inicios de sesión de riesgo**: un [inicio de sesión de riesgo](../identity-protection/overview-identity-protection.md) es un indicador de un intento de inicio de sesión que puede haber realizado alguien que no es el propietario legítimo de una cuenta de usuario.
    - **Usuarios marcados de riesgo**: un [usuario de riesgo](../identity-protection/overview-identity-protection.md) es un indicador de una cuenta de usuario que puede haber estado en peligro.

Este tema ofrece una visión general de los registros de aprovisionamiento. Los registros responden preguntas como las siguientes: 

* ¿Qué grupos se han creado correctamente en ServiceNow?
* ¿Qué usuarios se han quitado correctamente de Adobe?
* ¿Qué usuarios de Workday se crearon correctamente en Active Directory? 

## <a name="prerequisites"></a>Prerrequisitos

Estos usuarios pueden acceder a los datos de los registros de aprovisionamiento:

* Propietarios de aplicaciones (registros para sus propias aplicaciones)
* Los usuarios con los roles Administrador de seguridad, Lector de seguridad, Lector de informes, Operador de seguridad, Administrador de aplicaciones y Administrador de aplicaciones en la nube
* Usuarios en un rol personalizado con el [permiso provisioningLogs](../roles/custom-enterprise-app-permissions.md#full-list-of-permissions)
* Administradores globales


Para ver el informe de actividades de aprovisionamiento, el inquilino debe tener una licencia de Azure AD Premium asociada. Para actualizar la edición de Azure AD, consulte [Introducción a Azure Active Directory Premium](../fundamentals/active-directory-get-started-premium.md). 


## <a name="ways-of-interacting-with-the-provisioning-logs"></a>Maneras de interactuar con los registros de aprovisionamiento 
Los clientes pueden interactuar con los registros de aprovisionamiento de cuatro maneras:

- Acceder a los registros desde Azure Portal, como se describe en la siguiente sección.
- Transmitir los registros de aprovisionamiento a [Azure Monitor](../app-provisioning/application-provisioning-log-analytics.md). Este método permite la retención extendida de datos y la creación de paneles, alertas y consultas personalizados.
- Consultar la [Microsoft Graph API](/graph/api/resources/provisioningobjectsummary) para los registros de aprovisionamiento.
- Descargar los registros de aprovisionamiento como un archivo CSV o JSON.

## <a name="access-the-logs-from-the-azure-portal"></a>Acceso a los registros desde Azure Portal
Se puede acceder a los registros de aprovisionamiento si se selecciona **Registros de aprovisionamiento** en la sección **Supervisión** del panel **Azure Active Directory** en [Azure Portal](https://portal.azure.com). Algunos registros de aprovisionamiento pueden tardar hasta dos horas en aparecer en el portal.

![Captura de pantalla que muestra las selecciones para acceder a los registros de aprovisionamiento.](./media/concept-provisioning-logs/access-provisioning-logs.png "Registros de aprovisionamiento")


Un registro de aprovisionamiento tiene una vista de lista predeterminada que muestra:

- La identidad
- La acción
- El sistema de origen
- El sistema de destino
- El estado
- La fecha


![Captura de pantalla que muestra las columnas predeterminadas en un registro de aprovisionamiento.](./media/concept-provisioning-logs/default-columns.png "Columnas predeterminadas")

Puede personalizar la vista de lista seleccionando **Columnas** en la barra de herramientas.

![Captura de pantalla que muestra el botón para personalizar las columnas.](./media/concept-provisioning-logs/column-chooser.png "Selector de columnas")

Esta área le permite mostrar campos adicionales o quitar campos que ya se muestran.

![Captura de pantalla que muestra las columnas disponibles con algunas seleccionadas.](./media/concept-provisioning-logs/available-columns.png "Columnas disponibles")

Seleccione un elemento de la vista de lista para obtener información más detallada.

![Captura de pantalla que muestra información detallada.](./media/concept-provisioning-logs/steps.png "Filter")


## <a name="filter-provisioning-activities"></a>Filtrado de las actividades de aprovisionamiento

Puede filtrar los datos de aprovisionamiento. Algunos valores de los filtros se rellenan dinámicamente en función del inquilino. Por ejemplo, si no tiene ningún evento “crear” en el inquilino, no habrá una opción de filtro **Crear**.

En la vista predeterminada, puede seleccionar los siguientes filtros:

- **Identidad**
- **Fecha**
- **Estado**
- **Acción**


![Captura de pantalla que muestra valores de filtro.](./media/concept-provisioning-logs/default-filter.png "Filter")

El filtro **Identidad** le permite especificar el nombre o la identidad que le interesa. Esta identidad puede ser un usuario, un grupo, un rol u otro objeto. 

Puede buscar por nombre o por identificador de objeto. El identificador varía según el escenario. Por ejemplo, al aprovisionar un objeto de Azure AD a Salesforce, el id. de origen es el id. de objeto del usuario en Azure AD. El id. de destino es el id. del usuario en Salesforce. Al aprovisionar desde Workday a Active Directory, el id. de origen es el id. de empleado del trabajador de Workday. 

> [!NOTE]
> El nombre del usuario puede no estar siempre presente en la columna **Identidad**. Siempre habrá un identificador. 


El filtro **Fecha** le permite definir un período de tiempo para los datos devueltos. Los valores posibles son:

- 1 mes
- 7 días
- 30 días
- 24 horas
- Intervalo de tiempo personalizado

Cuando se selecciona un período de tiempo personalizado, puede configurar una fecha de inicio y una fecha de finalización.

El filtro **Estado** le permite seleccionar:

- **Todo**
- **Success**
- **Error**
- **Omitido**

El filtro **Acción** permite filtrar estas acciones:

- **Crear** 
- **Actualizar**
- **Eliminar**
- **Deshabilitar**
- **Otros**

Además de los filtros de la vista predeterminada, también puede establecer los siguientes filtros.

![Captura de pantalla que muestra los campos que se pueden agregar como filtros.](./media/concept-provisioning-logs/add-filter.png "Seleccionar un campo")

- **Id. del trabajo**: un id. del trabajo único se asocia a cada aplicación para la que se ha habilitado el aprovisionamiento.   

- **Id. de ciclo**: el id. de ciclo identifica de forma única el ciclo de aprovisionamiento. Puede compartir este id. con el soporte técnico del producto para buscar el ciclo en el que se ha producido este evento.

- **Id. de cambio**: el id. de cambio es un identificador único para el evento de aprovisionamiento. Puede compartir este id. con el soporte técnico del producto para buscar el evento de aprovisionamiento.   

- **Sistema de origen**: puede especificar desde dónde se aprovisiona la identidad. Por ejemplo, al aprovisionar un objeto desde Azure AD a ServiceNow, el sistema de origen es Azure AD. 

- **Sistema de destino**: puede especificar adónde se aprovisiona la identidad. Por ejemplo, al aprovisionar un objeto desde Azure AD a ServiceNow, el sistema de destino es ServiceNow. 

- **Aplicación**: puede mostrar solo los registros de aplicaciones con un nombre para mostrar que contenga una cadena específica.

## <a name="provisioning-details"></a>Detalles de aprovisionamiento 

Al seleccionar un elemento en la vista de lista de aprovisionamiento, obtendrá más información sobre dicho elemento. Los detalles se agrupan en las siguientes pestañas.

![Captura de pantalla que muestra cuatro pestañas que contienen los detalles del aprovisionamiento.](./media/concept-provisioning-logs/provisioning-tabs.png "Pestañas")

- **Pasos**: se describen los pasos necesarios para aprovisionar un objeto. El aprovisionamiento de un objeto puede constar de cuatro pasos:
  
  1. Importación del objeto.
  1. Determinación de si el objeto está en el ámbito.
  1. Coincidencia del objeto entre el origen y el destino.
  1. Aprovisionamiento del objeto (creación, actualización, eliminación o deshabilitación).

  ![Captura de pantalla que muestra los pasos de aprovisionamiento en la pestaña Pasos.](./media/concept-provisioning-logs/steps.png "Filter")

- **Troubleshooting & Recommendations** (Solución de problemas y recomendaciones): proporciona el código de error y el motivo. La información de error solo está disponible si se produce uno.

- **Propiedades modificadas**: se muestran el valor anterior y el nuevo. Si no hay ningún valor anterior, esa columna está en blanco.

- **Resumen**: se proporciona información general sobre lo que sucedió y los identificadores del objeto en los sistemas de origen y de destino.

## <a name="download-logs-as-csv-or-json"></a>Descarga de registros como archivo CSV o JSON

Puede descargar los registros de aprovisionamiento para usarlos más adelante. Para ello, vaya a los registros en Azure Portal y seleccione **Descargar**. El archivo se filtrará en función de los criterios de filtro seleccionados. Haga que los filtros sean lo más específicos posible para reducir el tamaño y el tiempo de la descarga. 

La descarga de CSV incluye tres archivos:

* **ProvisioningLogs**: descarga todos los registros, excepto los pasos de aprovisionamiento y las propiedades modificadas.
* **ProvisioningLogs_ProvisioningSteps**: contiene los pasos de aprovisionamiento y el id. de cambio. El id. de cambio se puede usar para unir el evento con los otros dos archivos.
* **ProvisioningLogs_ModifiedProperties**: contiene los atributos que se cambiaron y el id. de cambio. El id. de cambio se puede usar para unir el evento con los otros dos archivos.

#### <a name="open-the-json-file"></a>Apertura del archivo JSON
Para abrir el archivo JSON, use un editor de texto como [Microsoft Visual Studio Code](https://aka.ms/vscode). Visual Studio Code facilita la lectura del archivo gracias al resaltado de sintaxis. También puede abrir el archivo JSON mediante exploradores en un formato no editable, como [Microsoft Edge](https://aka.ms/msedge). 

#### <a name="prettify-the-json-file"></a>Mejora del archivo JSON
El archivo JSON se descarga en formato reducido para reducir el tamaño de la descarga. Este formato puede hacer que la carga sea difícil de leer. Consulte dos opciones para mejorar el formato del archivo:

- Uso de [Visual Studio Code para dar formato al JSON](https://code.visualstudio.com/docs/languages/json#_formatting).

- Uso de PowerShell para dar formato al JSON. Este script generará el archivo JSON en un formato que incluya tabulaciones y espacios: 

  ` $JSONContent = Get-Content -Path "<PATH TO THE PROVISIONING LOGS FILE>" | ConvertFrom-JSON`

  `$JSONContent | ConvertTo-Json > <PATH TO OUTPUT THE JSON FILE>`

#### <a name="parse-the-json-file"></a>Análisis del archivo JSON

Estos son algunos comandos de ejemplo para trabajar con el archivo JSON mediante PowerShell. Puede usar cualquier lenguaje de programación con el que esté familiarizado.  

En primer lugar, para [leer el archivo JSON](/powershell/module/microsoft.powershell.utility/convertfrom-json), ejecute este comando:

` $JSONContent = Get-Content -Path "<PATH TO THE PROVISIONING LOGS FILE>" | ConvertFrom-JSON`

Ahora puede analizar los datos de acuerdo a su escenario. Estos son algunos ejemplos: 

- Generación de todos los id. de trabajo en el archivo JSON:

  `foreach ($provitem in $JSONContent) { $provitem.jobId }`

- Generación de todos los id. de cambio para los eventos en los que la acción era "crear":

  `foreach ($provitem in $JSONContent) { `
  `   if ($provItem.action -eq 'Create') {`
  `       $provitem.changeId `
  `   }`
  `}`

## <a name="what-you-should-know"></a>Qué debería saber

Estas son algunas sugerencias y consideraciones para el aprovisionamiento de informes:

- Azure Portal almacena durante 30 días los datos de aprovisionamiento notificados si tiene una edición prémium y durante siete días si tiene una edición gratuita. Puede publicar los registros de aprovisionamiento en [Log Analytics](../app-provisioning/application-provisioning-log-analytics.md) para una retención superior a 30 días. 

- Puede usar el atributo de id. de cambio como identificador único. Esto es útil, por ejemplo, al interactuar con el soporte técnico del producto.

- Es posible que vea eventos omitidos para usuarios que no están en el ámbito. Esto es normal, especialmente cuando el ámbito de sincronización se establece en todos los usuarios y grupos. El servicio evaluará todos los objetos del inquilino, incluso los que están fuera del ámbito. 

- Los registros de aprovisionamiento no están disponibles actualmente en la nube de administración pública. Si no puede acceder a los registros de aprovisionamiento, use los registros de auditoría como una solución temporal. 

- Los registros de aprovisionamiento no muestran importaciones de roles (se aplica a AWS, Salesforce y Zendesk). Puede encontrar los registros de importaciones de roles en los registros de auditoría. 

## <a name="error-codes"></a>Códigos de error

Use la tabla siguiente para entender mejor cómo resolver los errores que se encuentre en los registros de aprovisionamiento. En el caso de los códigos de error que faltan, proporcione comentarios mediante el vínculo situado en la parte inferior de esta página. 

|Código de error|Descripción|
|---|---|
|Conflict, EntryConflict|Corrija los valores de atributo en conflicto en Azure AD o en la aplicación. O bien revise la configuración de atributo correspondiente si se supone que la cuenta de usuario en conflicto debería coincidir y tomarse. Revise la [documentación](../app-provisioning/customize-application-attributes.md) para obtener más información sobre cómo configurar atributos coincidentes.|
|TooManyRequests|La aplicación de destino rechazó este intento de actualizar el usuario porque está sobrecargado y recibe demasiadas solicitudes. No hay nada que hacer. Este intento se retirará automáticamente. También se ha notificado a Microsoft de este problema.|
|InternalServerError |La aplicación de destino devolvió un error inesperado. Un problema de servicio con la aplicación de destino puede estar impidiendo que esto funcione. Este intento se retirará automáticamente en 40 minutos.|
|InsufficientRights, MethodNotAllowed, NotPermitted, Unauthorized| Azure AD se autenticó con la aplicación de destino, pero no tenía autorización para realizar la actualización. Revise las instrucciones proporcionadas por la aplicación de destino, junto con el [tutorial](../saas-apps/tutorial-list.md) de la aplicación correspondiente.|
|UnprocessableEntity|La aplicación de destino devolvió una respuesta inesperada. Es posible que la configuración de la aplicación de destino no sea correcta o que haya un problema de servicio con la aplicación de destino que impida que esto funcione.|
|WebExceptionProtocolError |Se produjo un error de protocolo HTTP al conectarse a la aplicación de destino. No hay nada que hacer. Este intento se retirará automáticamente en 40 minutos.|
|InvalidAnchor|Ya no existe un usuario que se había creado previamente o que coincidía con el servicio de aprovisionamiento. Asegúrese de que el usuario existe. Para forzar una nueva coincidencia de todos los usuarios, use Microsoft Graph API para [reiniciar el trabajo](/graph/api/synchronization-synchronizationjob-restart?tabs=http&view=graph-rest-beta&preserve-view=true). <br><br>Reiniciar el aprovisionamiento desencadenará un ciclo inicial, que puede tardar en completarse. Al reiniciar el aprovisionamiento también se elimina la memoria caché que usa el servicio de aprovisionamiento para funcionar. Esto significa que todos los usuarios y grupos del inquilino tendrán que volver a evaluarse y que se pueden quitar determinados eventos de aprovisionamiento.|
|NotImplemented | La aplicación de destino devolvió una respuesta inesperada. Es posible que la configuración de la aplicación no sea correcta o que haya un problema de servicio con la aplicación de destino que impida que esto funcione. Revise las instrucciones proporcionadas por la aplicación de destino, junto con el [tutorial](../saas-apps/tutorial-list.md) de la aplicación correspondiente. |
|MandatoryFieldsMissing, MissingValues |No se pudo crear el usuario porque faltan los valores necesarios. Corrija los valores de atributo que faltan en el registro de origen o revise la configuración de atributo coincidente para asegurarse de que no se omitan los campos obligatorios. [Más información](../app-provisioning/customize-application-attributes.md) sobre cómo configurar atributos coincidentes.|
|SchemaAttributeNotFound |La operación no se pudo realizar porque se especificó un atributo que no existe en la aplicación de destino. Consulte la [documentación](../app-provisioning/customize-application-attributes.md) sobre la personalización de atributos y asegúrese de que la configuración sea correcta.|
|InternalError |Se produjo un error de servicio interno en el servicio de aprovisionamiento de Azure AD. No hay nada que hacer. Este intento se retirará automáticamente en 40 minutos.|
|InvalidDomain |No se pudo realizar la operación porque un valor de atributo contiene un nombre de dominio no válido. Actualice el nombre de dominio en el usuario o agréguelo a la lista de valores permitidos en la aplicación de destino. |
|Tiempo de espera |No se pudo completar la operación porque la aplicación de destino tardó demasiado tiempo en responder. No hay nada que hacer. Este intento se retirará automáticamente en 40 minutos.|
|LicenseLimitExceeded|No se pudo crear el usuario en la aplicación de destino porque no hay licencias disponibles para este usuario. Adquiera más licencias para la aplicación de destino. O bien revise las asignaciones de usuario y la configuración de asignación de atributos para asegurarse de que se asignan los atributos correctos a los usuarios correctos.|
|DuplicateTargetEntries  |No se pudo completar la operación porque se encontró más de un usuario en la aplicación de destino con los atributos coincidentes configurados. Quite el usuario duplicado de la aplicación de destino o [vuelva a configurar las asignaciones de atributos](../app-provisioning/customize-application-attributes.md).|
|DuplicateSourceEntries | No se pudo completar la operación porque se encontró más de un usuario con los atributos coincidentes configurados. Quite el usuario duplicado o [vuelva a configurar las asignaciones de atributos](../app-provisioning/customize-application-attributes.md).|
|ImportSkipped | Cuando se evalúa a cada usuario, el sistema intenta importar el usuario desde el sistema de origen. Este error suele producirse cuando al usuario que se importará le falta la propiedad coincidente definida en las asignaciones de atributos. Sin un valor presente en el objeto user para el atributo coincidente, el sistema no puede evaluar los cambios de ámbito, coincidencia o exportación. Tenga en cuenta que la presencia de este error no indica que el usuario está en el ámbito, porque aún no ha evaluado el ámbito para el usuario.|
|EntrySynchronizationSkipped | El servicio de aprovisionamiento ha consultado correctamente el sistema de origen y ha identificado al usuario. No se realizó ninguna acción adicional en el usuario y se omitió. Puede que el usuario estuviera fuera del ámbito o que ya existiera en el sistema de destino sin necesidad de realizar más cambios.|
|SystemForCrossDomainIdentityManagementMultipleEntriesInResponse| Una solicitud GET para recuperar un usuario o grupo recibió varios usuarios o grupos en la respuesta. El sistema espera solo un usuario o grupo en la respuesta. [Por ejemplo](../app-provisioning/use-scim-to-provision-users-and-groups.md#get-group), si realiza una solicitud GET para recuperar un grupo y proporciona un filtro para excluir miembros y el punto de conexión System for Cross-Domain Identity Management (SCIM) devuelve los miembros, obtendrá este error.|

## <a name="next-steps"></a>Pasos siguientes

* [Comprobación del estado de aprovisionamiento de usuarios](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md)
* [Problema al configurar el aprovisionamiento de usuarios para una aplicación de la galería de Azure AD](../app-provisioning/application-provisioning-config-problem.md)
* [Graph API para registros de aprovisionamiento](/graph/api/resources/provisioningobjectsummary)