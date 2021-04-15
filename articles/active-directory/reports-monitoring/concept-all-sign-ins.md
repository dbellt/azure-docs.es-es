---
title: 'Informes de actividad de inicio de sesión de Azure Active Directory: versión preliminar | Microsoft Docs'
description: Introducción a los informes de actividad de inicio de sesión en el portal de Azure Active Directory
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
ms.date: 03/16/2021
ms.author: markvi
ms.reviewer: besiler
ms.collection: M365-identity-device-management
ms.openlocfilehash: b17e7f34da1428eec219a393417b0a2a7b461078
ms.sourcegitcommit: b0557848d0ad9b74bf293217862525d08fe0fc1d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106552740"
---
# <a name="azure-active-directory-sign-in-activity-reports---preview"></a>Informes de actividad de inicio de sesión de Azure Active Directory: versión preliminar

La arquitectura de los informes de Azure Active Directory (Azure AD) consta de los siguientes componentes:

- **Actividad** 
    - **Inicios de sesión**: información sobre cuándo los usuarios, las aplicaciones y los recursos administrados inician sesión en Azure AD y acceden a los recursos.
    - **Registros de auditoría** - [: los](concept-audit-logs.md)registros de auditoría proporcionan información de la actividad del sistema sobre la administración de usuarios y grupos, aplicaciones administradas y actividades de directorio.
- **Seguridad** 
    - **Inicios de sesión de riesgo**: un [inicio de sesión de riesgo](../identity-protection/overview-identity-protection.md) es un indicador de un intento de inicio de sesión de alguien que no es el propietario legítimo de una cuenta de usuario.
    - **Usuarios marcados en riesgo**: un [usuario en riesgo](../identity-protection/overview-identity-protection.md) es un indicador de una cuenta de usuario que puede haber estado en peligro.

El informe inicios de sesión clásico de Azure Active Directory proporciona información general sobre los inicios de sesión de usuario interactivos. Además, ahora tiene acceso a tres informes de inicio de sesión adicionales que están en versión preliminar:

- Inicios de sesión de usuario no interactivos

- Inicios de sesión de entidad de servicio

- Inicios de sesión de identidades administradas para recursos de Azure

En este artículo se ofrece información general sobre el informe de actividad de inicio de sesión con la versión preliminar de los inicios de sesión no interactivos, de aplicaciones y de identidades administradas para recursos de Azure. Para obtener información sobre el informe de inicio de sesión sin las características en versión preliminar, consulte [Informes de actividad de inicio de sesión en el portal de Azure Active Directory](concept-sign-ins.md).



## <a name="prerequisites"></a>Requisitos previos

Antes de poder empezar a usar esta característica, debe conocer las respuestas a:

- ¿Quién puede acceder a los datos?

- ¿Qué licencia de Azure AD se necesita para acceder a la actividad de inicio de sesión?

### <a name="who-can-access-the-data"></a>¿Quién puede acceder a los datos?

- Usuarios de los roles Administrador de seguridad, Lector de seguridad o Lector de informes

- Administradores globales

- Cualquier usuario (no administradores) puede acceder a sus propios inicios de sesión 

### <a name="what-azure-ad-license-do-you-need-to-access-sign-in-activity"></a>¿Qué licencia de Azure AD se necesita para acceder a la actividad de inicio de sesión?

El inquilino debe tener una licencia de Azure AD Premium asociada para ver las actividades de inicio de sesión. Consulte [Introducción a Azure Active Directory Premium](../fundamentals/active-directory-get-started-premium.md) para actualizar la edición de Azure Active Directory. Los datos tardarán un par de días en aparecer en los informes después de actualizar a una licencia Premium sin actividades de datos antes de la actualización.



## <a name="sign-ins-report"></a>Informe de inicios de sesión

El informe de inicios de sesión proporciona respuestas a las preguntas siguientes:

- ¿Cuál es el patrón de inicio de sesión de un usuario, una aplicación o un servicio?
- ¿Cuántos usuarios, aplicaciones o servicios han iniciado sesión durante una semana?
- ¿Cuál es el estado de estos inicios de sesión?


En la hoja del informe de inicios de sesión, puede cambiar entre:

- **Inicios de sesión de usuario interactivos**: inicios de sesión en los que un usuario proporciona un factor de autenticación, como una contraseña, una respuesta mediante una aplicación de MFA, un factor biométrico o un código QR.

- **Inicios de sesión de usuario no interactivos**: inicios de sesión realizados por un cliente en nombre de un usuario. Estos inicios de sesión no requieren ninguna interacción ni factor de autenticación del usuario. Por ejemplo, la autenticación y la autorización mediante tokens de acceso y actualización, que no requieren que un usuario escriba las credenciales.

- **Inicios de sesión de entidad de servicio**: inicios de sesión realizados por aplicaciones y entidades de servicio que no impliquen a ningún usuario. En estos inicios de sesión, la aplicación o el servicio proporcionan una credencial en su propio nombre para autenticarse o acceder a los recursos.

- **Inicios de sesión de identidades administradas para recursos de Azure**: inicios de sesión realizados por recursos de Azure que tienen secretos administrados por Azure. Para obtener más información, consulte [¿Qué son las identidades administradas para recursos de Azure?](../managed-identities-azure-resources/overview.md) 


![Tipos de informes de inicios de sesión](./media/concept-all-sign-ins/sign-ins-report-types.png)












## <a name="user-sign-ins&quot;></a>Inicios de sesión de usuario

Cada pestaña de la hoja de inicios de sesión muestra las columnas predeterminadas que aparecen a continuación. Algunas pestañas tienen columnas adicionales:

- Fecha de inicio de sesión

- Id. de solicitud

- Nombre de usuario o identificador de usuario

- Nombre de aplicación o identificador de aplicación

- Estado del inicio de sesión

- Dirección IP del dispositivo usado para el inicio de sesión



### <a name=&quot;interactive-user-sign-ins&quot;></a>Inicios de sesión de usuario interactivos


Los inicios de sesión de usuario interactivos son inicios de sesión en los que un usuario proporciona un factor de autenticación para Azure AD o interactúa directamente con Azure AD o una aplicación auxiliar, como la aplicación Microsoft Authenticator. Los factores que los usuarios proporcionan incluyen las contraseñas, las respuestas a los desafíos de MFA, los factores biométricos o los códigos QR que un usuario proporciona a Azure AD o a una aplicación auxiliar.

> [!NOTE]
> Este informe también incluye los inicios de sesión federados de proveedores de identidades que están federados con Azure AD.  



Nota: El informe de inicios de sesión de usuario interactivo se usa para incluir algunos inicios de sesión no interactivos de los clientes de Microsoft Exchange. Aunque esos inicios de sesión no son interactivos, se han incluido en el informe de inicios de sesión de usuarios interactivos para obtener mayor visibilidad. Una vez que el informe de inicios de sesión de usuarios no interactivos se introdujo en la versión preliminar pública en noviembre de 2020, los registros de eventos de inicio de sesión no interactivos se movieron al informe de inicio de sesión de usuario no interactivo para aumentar la precisión. 


**Tamaño del informe:** pequeño <br> 
**Ejemplos:**

- Un usuario proporciona el nombre de usuario y la contraseña en la pantalla de inicio de sesión de Azure AD.

- Un usuario pasa un desafío de MFA por SMS.

- Un usuario proporciona un gesto biométrico para desbloquear su PC Windows con Windows Hello para empresas.

- Un usuario que está federado con Azure AD con una aserción de SAML de AD FS.


Además de los campos predeterminados, el informe de inicios de sesión interactivos también muestra: 

- La ubicación de inicio de sesión

- Si se ha aplicado el acceso condicional



Puede personalizar la vista de lista, haga clic en **Columnas** en la barra de herramientas.

![Columnas de inicio de sesión de usuario interactivo](./media/concept-all-sign-ins/columns-interactive.png &quot;Columnas del inicio de sesión de usuario interactivo")





La personalización de la vista permite mostrar campos adicionales o eliminar campos que ya se muestran.

![Todas las columnas interactivas](./media/concept-all-sign-ins/all-interactive-columns.png)


Seleccione un elemento de la vista de lista para obtener información más detallada sobre el inicio de sesión relacionado.

![Actividad de inicio de sesión](./media/concept-all-sign-ins/interactive-user-sign-in-details.png "Inicios de sesión de usuario interactivos")



### <a name="non-interactive-user-sign-ins"></a>Inicios de sesión de usuario no interactivos

Los inicios de sesión de usuario no interactivos son inicios de sesión que se realizaron mediante una aplicación cliente o componentes del sistema operativo en nombre de un usuario. Al igual que los inicios de sesión de usuario interactivos, estos inicios de sesión se realizan en nombre de un usuario. A diferencia de los inicios de sesión de usuario interactivos, estos inicios de sesión no requieren que el usuario proporcione un factor de autenticación. En su lugar, el dispositivo o la aplicación cliente usa un token o un código para autenticarse o acceder a un recurso en nombre de un usuario. En general, el usuario percibe estos inicios de sesión como algo que sucede en segundo plano de la actividad del usuario.


**Tamaño del informe:** grande <br>
**Ejemplos:** 

- Una aplicación cliente usa un token de actualización de OAuth 2.0 para obtener un token de acceso.

- Un cliente utiliza un código de autorización de OAuth 2.0 para obtener un token de acceso y un token de actualización.

- Un usuario realiza el inicio de sesión único (SSO) en una aplicación web o una aplicación de Windows en un equipo unido a Azure AD.

- Un usuario inicia sesión en una segunda aplicación de Microsoft Office mientras tiene una sesión en un dispositivo móvil que usa FOCI (Familia de identificadores de cliente).




Además de los campos predeterminados, el informe de inicios de sesión no interactivos también muestra: 

- Id. de recurso

- Número de inicios de sesión agrupados




No se pueden personalizar los campos que se muestran en este informe.


![Columnas deshabilitadas](./media/concept-all-sign-ins/disabled-columns.png "Columnas deshabilitadas")

Para facilitar la síntesis de los datos, se agrupan los eventos de inicio de sesión no interactivos. A menudo, los clientes crean muchos inicios de sesión no interactivos en nombre del mismo usuario en un breve período de tiempo, que comparten las mismas características excepto la hora en que se intentó iniciar sesión. Por ejemplo, un cliente puede obtener un token de acceso una vez por hora en nombre de un usuario. Si el usuario o el cliente no cambian de estado, la dirección IP, el recurso y el resto de la información es la misma para cada solicitud de token de acceso. Cuando Azure AD registra varios inicios de sesión idénticos salvo por la fecha y la hora, dichos inicios de sesión de la misma entidad se agregan en una sola fila. Una fila con varios inicios de sesión idénticos (excepto por la fecha y la hora de emisión) tendrá un valor mayor que 1 en la columna Número de inicios de sesión. Puede expandir la fila para ver todos los inicios de sesión diferentes y sus diferentes marcas de tiempo. Los inicios de sesión se agregan para los usuarios no interactivos cuando coinciden los datos siguientes:


- Application

- Usuario

- Dirección IP

- Status

- Id. de recurso


Puede:

- Expandir un nodo para ver los elementos individuales de un grupo.  

- Hacer clic en un elemento individual para ver todos los detalles. 


![Detalles de los inicios de sesión de usuario no interactivos](./media/concept-all-sign-ins/non-interactive-sign-ins-details.png)




## <a name="service-principal-sign-ins"></a>Inicios de sesión de entidad de servicio

A diferencia de los inicios de sesión de usuario interactivos y no interactivos, los inicios de sesión de entidad de servicio no implican a un usuario. En su lugar, son inicios de sesión realizados por una cuenta que no es de usuario, como las aplicaciones o las entidades de servicio (excepto el inicio de sesión de identidad administrada, que se incluye solo en el informe de inicios de sesión de identidad administrada). En estos inicios de sesión, la aplicación o el servicio proporcionan su propia credencial, como un certificado o un secreto, para autenticarse o acceder a los recursos.


**Tamaño del informe:** grande <br>
**Ejemplos:**

- Una entidad de servicio utiliza un certificado para autenticarse y acceder a Microsoft Graph. 

- Una aplicación utiliza un secreto de cliente para autenticarse en el flujo de credenciales del cliente de OAuth. 


Este informe tiene una vista de lista predeterminada que muestra:

- Fecha de inicio de sesión

- Id. de solicitud

- Nombre o identificador de la entidad de servicio

- Status

- Dirección IP

- Nombre del recurso

- Id. de recurso

- Número de inicios de sesión

No se pueden personalizar los campos que se muestran en este informe.

![Columnas deshabilitadas](./media/concept-all-sign-ins/disabled-columns.png "Columnas deshabilitadas")

Para facilitar la síntesis de los datos de los registros de inicio de sesión de la entidad de servicio, se agrupan los eventos de inicio de sesión de la entidad de servicio. Los inicios de sesión de la misma entidad en las mismas condiciones se agregan en una sola fila. Puede expandir la fila para ver todos los inicios de sesión diferentes y sus diferentes marcas de tiempo. Los inicios de sesión se agregan en el informe por entidad de servicio cuando coinciden los datos siguientes:

- Nombre o identificador de la entidad de servicio

- Status

- Dirección IP

- Nombre o identificador del recurso

Puede:

- Expandir un nodo para ver los elementos individuales de un grupo.  

- Hacer clic en un elemento individual para ver todos los detalles. 


![Detalles de las columnas](./media/concept-all-sign-ins/service-principals-sign-ins-view.png "Detalles de las columnas")




## <a name="managed-identity-for-azure-resources-sign-ins"></a>Inicios de sesión de identidad administrada para recursos de Azure 

Los inicios de sesión de identidad administrada para recursos de Azure son inicios de sesión realizados por los recursos cuyos secretos son administrados por Azure para simplificar la administración de credenciales.

**Tamaño del informe:** Pequeña <br> 
**Ejemplos:**

Una máquina virtual con credenciales administradas usa Azure AD para obtener un token de acceso.   


Este informe tiene una vista de lista predeterminada que muestra:


- Identificador de la identidad administrada

- Nombre de la identidad administrada

- Resource

- Id. de recurso

- Número de inicios de sesión agrupados

No se pueden personalizar los campos que se muestran en este informe.

Para que sea más fácil sintetizar los datos, se agrupan los eventos no interactivos en los registros de inicio de sesión de identidades administradas para recursos de Azure. Los inicios de sesión de la misma entidad se agregan en una sola fila. Puede expandir la fila para ver todos los inicios de sesión diferentes y sus diferentes marcas de tiempo. Los inicios de sesión se agregan en el informe de identidades administradas cuando coinciden los datos siguientes:

- Nombre o identificador de la identidad administrada

- Status

- Dirección IP

- Nombre o identificador del recurso

Seleccione un elemento en la vista de lista para mostrar todos los inicios de sesión que se han agrupado en un nodo.

Seleccione un elemento agrupado para ver todos los detalles del inicio de sesión. 


## <a name="filter-sign-in-activities"></a>Filtrado de las actividades de inicio de sesión

Al establecer un filtro, puede restringir el ámbito de los datos de inicio de sesión devueltos. Azure AD proporciona una amplia variedad de filtros adicionales que puede establecer. Al establecer el filtro, siempre debe prestar especial atención al filtro de intervalo del campo **Fecha** configurado. Un filtro de intervalo de fechas adecuado garantiza que Azure AD solo devuelva los datos que realmente le interesen.     

El filtro de intervalo del campo **Fecha** permite definir un período de tiempo para los datos devueltos.
Los valores posibles son:

- Un mes

- Siete días

- Veinticuatro horas

- Personalizado

![Filtro de intervalo de fechas](./media/concept-all-sign-ins/date-range-filter.png)





### <a name="filter-user-sign-ins"></a>Filtrado de los inicios de sesión de usuario

El filtro para los inicios de sesión interactivos y no interactivos es el mismo. Por este motivo, el filtro que haya configurado para los inicios de sesión interactivos se conserva para los inicios de sesión no interactivos y viceversa. 






## <a name="access-the-new-sign-in-activity-reports"></a>Acceso a los nuevos informes de actividad de inicio de sesión 

El informe actividad de inicios de sesión de Azure Portal proporciona un método sencillo para activar y desactivar el informe de la versión preliminar. Si tiene habilitados los informes de la versión preliminar, obtendrá un nuevo menú que le proporcionará acceso a todos los tipos de informes de actividad de inicio de sesión.     


Para acceder a los nuevos informes de inicio de sesión con los inicios de sesión no interactivos y de aplicaciones: 

1. En [Azure Portal](https://portal.azure.com), seleccione **Azure Active Directory**.

    ![Selección de Azure AD](./media/concept-all-sign-ins/azure-services.png)

2. En la sección **Supervisión**, haga clic en **Inicios de sesión**.

    ![Selección de Inicios de sesión](./media/concept-all-sign-ins/sign-ins.png)

3. Haga clic en la barra **Versión preliminar**.

    ![Habilitación de la nueva vista](./media/concept-all-sign-ins/enable-new-preview.png)

4. Para volver a la vista predeterminada, vuelva a hacer clic en la barra **Versión preliminar**. 

    ![Restauración de la vista clásica](./media/concept-all-sign-ins/switch-back.png)







## <a name="download-sign-in-activity-reports"></a>Descarga de los informes de actividad de inicio de sesión

Cuando se descarga un informe de actividad de inicio de sesión, se cumple lo siguiente:

- Puede descargar el informe de inicio de sesión como archivo CSV o JSON.

- Puede descargar hasta 100 000 registros. Si desea descargar más datos, use la API de generación de informes.

- La descarga se basa en la selección de filtros que ha realizado.

- El número de registros que se puede descargar también está restringido por las [directivas de retención de informes de Azure Active Directory](reference-reports-data-retention.md). 


![Descarga de informes](./media/concept-all-sign-ins/download-reports.png "Descarga de informes")


Cada descarga en formato CSV consta de seis archivos diferentes:

- Inicios de sesión interactivos

- Detalles de autenticación de los inicios de sesión interactivos

- Inicios de sesión no interactivos

- Detalles de autenticación de los inicios de sesión no interactivos

- Inicios de sesión de entidad de servicio

- Inicios de sesión de identidad administrada para recursos de Azure

Cada descarga en formato JSON consta de cuatro archivos diferentes:

- Inicios de sesión interactivos (incluye los detalles de autenticación)

- Inicios de sesión no interactivos (incluye los detalles de autenticación)

- Inicios de sesión de entidad de servicio

- Inicios de sesión de identidad administrada para recursos de Azure

![Descarga de archivos](./media/concept-all-sign-ins/download-files.png "Descarga de archivos")




## <a name="next-steps"></a>Pasos siguientes

* [Códigos de error de los informes de actividad de inicio de sesión](reference-sign-ins-error-codes.md)
* [Directivas de retención de datos de Azure AD](reference-reports-data-retention.md)
* [Latencias de informes de Azure AD](reference-reports-latencies.md)
