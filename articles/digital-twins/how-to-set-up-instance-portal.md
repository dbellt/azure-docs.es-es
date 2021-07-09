---
title: Configuración de una instancia y autenticación (portal)
titleSuffix: Azure Digital Twins
description: Vea cómo configurar una instancia del servicio Azure Digital Twins mediante Azure Portal.
author: baanders
ms.author: baanders
ms.date: 7/23/2020
ms.topic: how-to
ms.service: digital-twins
ms.custom: contperf-fy21q2, subject-rbac-steps
ms.openlocfilehash: 203c53daf07949e343e9aa7e5cf56e89a7b07b1f
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110474549"
---
# <a name="set-up-an-azure-digital-twins-instance-and-authentication-portal"></a>Configuración de una instancia de Azure Digital Twins y autenticación (portal)

[!INCLUDE [digital-twins-setup-selector.md](../../includes/digital-twins-setup-selector.md)]

En este artículo se describen los pasos para **configurar una nueva instancia de Azure Digital Twins**, incluidas la creación de la instancia y la configuración de la autenticación. Después de completar este artículo, tendrá una instancia de Azure Digital Twins lista para empezar a programar.

En esta versión de este artículo se realizan los pasos manualmente, uno por uno, mediante Azure Portal. Azure Portal es una consola unificada basada en web que proporciona una alternativa a las herramientas de línea de comandos.
* Para realizar estos pasos manualmente mediante la CLI, consulte la versión de CLI de este artículo: [Procedimiento: Configuración de una instancia y autenticación (CLI)](how-to-set-up-instance-cli.md) .
* Para ejecutar una configuración automatizada mediante un script de implementación de ejemplo, consulte la versión con scripts de este artículo: [Procedimiento: Configuración de una instancia y autenticación (con scripts)](how-to-set-up-instance-scripted.md) .

[!INCLUDE [digital-twins-setup-steps.md](../../includes/digital-twins-setup-steps.md)]

## <a name="create-the-azure-digital-twins-instance"></a>Creación de una instancia de Azure Digital Twins

En esta sección, **creará una instancia de Azure Digital Twins** mediante [Azure Portal](https://ms.portal.azure.com/). Vaya al portal e inicie sesión con sus credenciales.

Una vez en el portal, seleccione _Crear un recurso_ en el menú de la página principal de servicios de Azure, para comenzar.

:::image type="content" source= "media/how-to-set-up-instance/portal/create-resource.png" alt-text="Captura de pantalla de Azure Portal, en la que se resalta el icono &quot;Crear un recurso&quot; de la página principal.":::

Busque *Azure Digital Twins* en el cuadro de búsqueda y elija el servicio **Azure Digital Twins** en los resultados. Seleccione el botón _Crear_ para crear una nueva instancia del servicio.

:::image type="content" source= "media/how-to-set-up-instance/portal/create-azure-digital-twins.png" alt-text="Captura de pantalla de Azure Portal, en la que se resalta el botón &quot;Crear&quot; de la página del servicio Azure Digital Twins.":::

En la siguiente página **Crear recurso**, rellene los valores indicados a continuación:
* **Suscripción**: la suscripción de Azure que está usando.
  - **Grupo de recursos**: grupo de recursos en el que implementar la instancia. Si aún no tiene un grupo de recursos existente en mente, puede crear uno aquí mediante la selección del vínculo *Crear nuevo* y la introducción de un nombre para el nuevo grupo de recursos.
* **Ubicación**: región habilitada para Azure Digital Twins para la implementación. Para obtener más información sobre la compatibilidad regional, visite [Productos de Azure disponibles por región (Azure Digital Twins)](https://azure.microsoft.com/global-infrastructure/services/?products=digital-twins) .
* **Nombre del recurso**: nombre para la instancia de Azure Digital Twins. Si su suscripción tiene otra instancia de Azure Digital Twins en la región que ya usa el nombre especificado, se le pedirá que elija un nombre diferente.
* **Conceder acceso al recurso**: al marcar la casilla de esta sección se concederá permiso a la cuenta de Azure para acceder a los datos de la instancia y administrarlos. Si es quien va a administrar la instancia, debe marcar esta casilla ahora. Si está atenuada porque no tiene permiso en la suscripción, puede seguir creando el recurso y hacer que alguien con los permisos necesarios le conceda el rol más adelante. Para obtener más información sobre este rol y la asignación de roles a la instancia, consulte la sección siguiente [Configuración de permisos de acceso de usuarios](#set-up-user-access-permissions).

:::image type="content" source= "media/how-to-set-up-instance/portal/create-azure-digital-twins-2.png" alt-text="Captura de pantalla del proceso Crear recurso para Azure Digital Twins en Azure Portal. Los valores descritos aparecen rellenos.":::

Cuando haya terminado, puede seleccionar **Revisar y crear** si no quiere configurar más opciones para la instancia. Esta acción le llevará a una página de resumen, donde puede revisar los detalles de la instancia que ha introducido y finalizar con **Crear**. 

Si quiere configurar más detalles para la instancia, en la sección siguiente se describen las pestañas de configuración restantes.

### <a name="additional-setup-options"></a>Opciones de configuración adicionales

Estas son las opciones adicionales que puede configurar durante la instalación, mediante las demás pestañas del proceso **Crear recurso**.

* **Redes**: en esta pestaña, puede habilitar puntos de conexión privados con [Azure Private Link](../private-link/private-link-overview.md) para eliminar la exposición de la red pública a la instancia. Para instrucciones, consulte [Procedimiento: para habilitar el acceso privado con Private Link (versión preliminar)](./how-to-enable-private-link-portal.md#add-a-private-endpoint-during-instance-creation) .
* **Avanzadas**: en esta pestaña, puede habilitar una identidad administrada por el sistema para la instancia que se puede usar al reenviar eventos a [puntos de conexión](concepts-route-events.md). Para más información sobre el uso de identidades administradas por el sistema con Azure Digital Twins, consulte [Protección de Azure Digital Twins](concepts-security.md#managed-identity-for-accessing-other-resources-preview).
* **Etiquetas**: en esta pestaña, puede agregar etiquetas a la instancia a fin de facilitar su organización entre los recursos de Azure. Para obtener más información sobre las etiquetas de recurso de Azure, vea [Etiquetado de recursos, grupos de recursos y suscripciones para una organización lógica](../azure-resource-manager/management/tag-resources.md).

### <a name="verify-success-and-collect-important-values"></a>Comprobación de que la operación es correcta y recopilación de valores importantes

Después de finalizar la configuración de la instancia con la selección de **Crear**, puede ver el estado de la implementación en las notificaciones de Azure que hay en la barra de iconos del portal. La notificación le indicará cuándo se ha realizado la implementación correctamente y podrá seleccionar el botón _Ir al recurso_ para ver la instancia creada.

:::image type="content" source="media/how-to-set-up-instance/portal/notifications-deployment.png" alt-text="Captura de pantalla de las notificaciones de Azure que muestra una implementación correcta y se resalta el botón &quot;Ir al recurso&quot; en Azure Portal.":::

Como alternativa, si se produce un error con la implementación, la notificación le indicará el motivo. Observe el consejo del mensaje de error y vuelva a intentar crear la instancia.

>[!TIP]
>Una vez creada la instancia, para volver a su página en cualquier momento, busque el nombre de la instancia en la barra de búsqueda de Azure Portal.

A partir de la página *Información general* de la instancia, anote su *Nombre*, *Grupo de recursos* y *Nombre de host*. Estos son todos los valores importantes que puede necesitar a medida que sigue trabajando con la instancia de Azure Digital Twins. Si otros usuarios van a programar en la instancia, debe compartirlos con ellos.

:::image type="content" source="media/how-to-set-up-instance/portal/instance-important-values.png" alt-text="Captura de pantalla de Azure Portal en la que se resaltan los valores importantes de la página de información general de la instancia de Azure Digital Twins.":::

Ahora tiene lista una instancia de Azure Digital Twins. A continuación, debe proporcionar los permisos de usuario de Azure adecuados para administrarla.

## <a name="set-up-user-access-permissions"></a>Configuración de permisos de acceso de usuarios

[!INCLUDE [digital-twins-setup-role-assignment.md](../../includes/digital-twins-setup-role-assignment.md)]

Hay dos maneras de crear una asignación de roles para un usuario en Azure Digital Twins:
* [Durante la creación de la instancia de Azure Digital Twins](#assign-the-role-during-instance-creation)
* [Mediante Azure Identity Management (IAM)](#assign-the-role-using-azure-identity-management-iam)

Ambas opciones requieren los mismos permisos.

### <a name="prerequisites-permission-requirements"></a>Requisitos previos: Requisitos de permisos

[!INCLUDE [digital-twins-setup-permissions.md](../../includes/digital-twins-setup-permissions.md)]

### <a name="assign-the-role-during-instance-creation"></a>Asignación del rol durante la creación de la instancia

Al crear el recurso de Azure Digital Twins mediante el proceso descrito [anteriormente en este artículo](#create-the-azure-digital-twins-instance), seleccione **Asignar rol de propietario de datos de Azure Digital Twins** en **Conceder acceso al recurso**. Esto le concederá acceso completo a las API del plano de datos.

:::image type="content" source= "media/how-to-set-up-instance/portal/create-azure-digital-twins-2-role.png" alt-text="Captura de pantalla del proceso Crear recurso para Azure Digital Twins en Azure Portal. La casilla situada debajo de Conceder acceso al recurso está resaltada.":::

Si no tiene permiso para asignar un rol a una identidad, el cuadro aparecerá atenuado.

:::image type="content" source= "media/how-to-set-up-instance/portal/create-azure-digital-twins-2-role-greyed.png" alt-text="Captura de pantalla del proceso Crear recurso para Azure Digital Twins en Azure Portal. La casilla situada debajo de Conceder acceso al recurso está deshabilitada.":::

En ese caso, todavía puede continuar con la creación correcta del recurso de Azure Digital Twins, pero alguien con los permisos adecuados deberá asignarle este rol o a la persona que va a administrar los datos de la instancia.

### <a name="assign-the-role-using-azure-identity-management-iam"></a>Asignación del rol mediante Azure Identity Management (IAM)

También puede asignar el **rol de propietario de datos de Azure Digital Twins**  mediante las opciones de control de acceso de Azure Identity Management (IAM).

1. En primer lugar, abra la página de la instancia de Azure Digital Twins en Azure Portal. 

1. Seleccione **Access Control (IAM)** .

1. Seleccione **Agregar** > **Agregar asignación de roles** para abrir la página Agregar asignación de roles.

1. Asigne el rol **Propietario de datos de Azure Digital Twins**. Para asignar roles, consulte [Asignación de roles de Azure mediante Azure Portal](../role-based-access-control/role-assignments-portal.md).
    
    | Configuración | Valor |
    | --- | --- |
    | Role | [Propietario de datos de Azure Digital Twins](../role-based-access-control/built-in-roles.md#azure-digital-twins-data-owner) |
    | Asignar acceso a | Usuario, grupo o entidad de servicio |
    | Miembros | busque el nombre o la dirección de correo electrónico del usuario para realizar la asignación. |
    
    ![Página Agregar asignación de roles](../../includes/role-based-access-control/media/add-role-assignment-page.png)

### <a name="verify-success"></a>Comprobación de que la operación se ha completado correctamente

Puede ver la asignación de roles que ha configurado en *Control de acceso (IAM) > Asignaciones de roles*. El usuario debe aparecer en la lista con el rol *Propietario de datos de Azure Digital Twins*. 

:::image type="content" source="media/how-to-set-up-instance/portal/verify-role-assignment.png" alt-text="Captura de pantalla de las asignaciones de roles para una instancia de Azure Digital Twins en Azure Portal.":::

Ahora tiene lista una instancia de Azure Digital Twins y los permisos asignados para administrarla.

## <a name="next-steps"></a>Pasos siguientes

Pruebe las llamadas individuales de la API de REST en su instancia mediante los comandos de la CLI de Azure Digital Twins: 
* [Referencia de az dt](/cli/azure/dt?view=azure-cli-latest&preserve-view=true)
* [Conceptos: Conjunto de comandos de la CLI de Azure Digital Twins](concepts-cli.md)

O bien consulte cómo conectar una aplicación cliente a la instancia mediante el código de autenticación:
* [Procedimiento: Escritura de código de autenticación de aplicación](how-to-authenticate-client.md)