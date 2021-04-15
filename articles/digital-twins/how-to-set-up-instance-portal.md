---
title: Configuración de una instancia y autenticación (portal)
titleSuffix: Azure Digital Twins
description: Vea cómo configurar una instancia del servicio Azure Digital Twins mediante Azure Portal.
author: baanders
ms.author: baanders
ms.date: 7/23/2020
ms.topic: how-to
ms.service: digital-twins
ms.custom: contperf-fy21q2
ms.openlocfilehash: 7247aa45ae164cd4f2eb5f210e0ec6ec91a0a25f
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107104072"
---
# <a name="set-up-an-azure-digital-twins-instance-and-authentication-portal"></a>Configuración de una instancia de Azure Digital Twins y autenticación (portal)

[!INCLUDE [digital-twins-setup-selector.md](../../includes/digital-twins-setup-selector.md)]

En este artículo se describen los pasos para **configurar una nueva instancia de Azure Digital Twins**, incluidas la creación de la instancia y la configuración de la autenticación. Después de completar este artículo, tendrá una instancia de Azure Digital Twins lista para empezar a programar.

En esta versión de este artículo se realizan los pasos manualmente, uno por uno, mediante Azure Portal. Azure Portal es una consola unificada basada en web que proporciona una alternativa a las herramientas de línea de comandos.
* Para realizar estos pasos manualmente mediante la CLI, consulte la versión de CLI de este artículo: [*Procedimiento: Configuración de una instancia y autenticación (CLI)*](how-to-set-up-instance-cli.md).
* Para ejecutar una configuración automatizada mediante un script de implementación de ejemplo, consulte la versión con scripts de este artículo: [*Procedimiento: Configuración de una instancia y autenticación (con scripts)*](how-to-set-up-instance-scripted.md).

[!INCLUDE [digital-twins-setup-steps.md](../../includes/digital-twins-setup-steps.md)]
[!INCLUDE [digital-twins-setup-permissions.md](../../includes/digital-twins-setup-permissions.md)]

## <a name="create-the-azure-digital-twins-instance"></a>Creación de una instancia de Azure Digital Twins

En esta sección, **creará una instancia de Azure Digital Twins** mediante [Azure Portal](https://ms.portal.azure.com/). Vaya al portal e inicie sesión con sus credenciales.

Una vez en el portal, seleccione _Crear un recurso_ en el menú de la página principal de servicios de Azure, para comenzar.

:::image type="content" source= "media/how-to-set-up-instance/portal/create-resource.png" alt-text="Selección de Crear un recurso en la página principal de Azure Portal":::

Busque *Azure Digital Twins* en el cuadro de búsqueda y elija el servicio **Azure Digital Twins** en los resultados. Seleccione el botón _Crear_ para crear una nueva instancia del servicio.

:::image type="content" source= "media/how-to-set-up-instance/portal/create-azure-digital-twins.png" alt-text="Selección de Crear en la página del servicio de Azure Digital Twins":::

En la siguiente página **Crear recurso**, rellene los valores indicados a continuación:
* **Suscripción**: la suscripción de Azure que está usando.
  - **Grupo de recursos**: grupo de recursos en el que implementar la instancia. Si aún no tiene un grupo de recursos existente en mente, puede crear uno aquí mediante la selección del vínculo *Crear nuevo* y la introducción de un nombre para el nuevo grupo de recursos.
* **Ubicación**: región habilitada para Azure Digital Twins para la implementación. Para obtener más información sobre la compatibilidad regional, visite [*Productos de Azure disponibles por región (Azure Digital Twins)*](https://azure.microsoft.com/global-infrastructure/services/?products=digital-twins).
* **Nombre del recurso**: nombre para la instancia de Azure Digital Twins. Si su suscripción tiene otra instancia de Azure Digital Twins en la región que ya usa el nombre especificado, se le pedirá que elija un nombre diferente.

:::image type="content" source= "media/how-to-set-up-instance/portal/create-azure-digital-twins-2.png" alt-text="Paso para rellenar los valores descritos para crear un recurso de Azure Digital Twins":::

Cuando haya terminado, puede seleccionar **Revisar y crear** si no quiere configurar más opciones para la instancia. Esta acción le llevará a una página de resumen, donde puede revisar los detalles de la instancia que ha introducido y finalizar con **Crear**. 

Si quiere configurar más detalles para la instancia, en la sección siguiente se describen las pestañas de configuración restantes.

### <a name="additional-setup-options"></a>Opciones de configuración adicionales

Estas son las opciones adicionales que puede configurar durante la instalación, mediante las demás pestañas del proceso **Crear recurso**.

* **Redes**: en esta pestaña, puede habilitar puntos de conexión privados con [Azure Private Link](../private-link/private-link-overview.md) para eliminar la exposición de la red pública a la instancia. Para instrucciones, consulte [*Procedimiento: para habilitar el acceso privado con Private Link (versión preliminar)*](./how-to-enable-private-link-portal.md#add-a-private-endpoint-during-instance-creation).
* **Avanzadas**: en esta pestaña, puede habilitar una [identidad administrada por el sistema](../active-directory/managed-identities-azure-resources/overview.md) para la instancia que se puede usar al reenviar eventos a [puntos de conexión](concepts-route-events.md). Para instrucciones, consulte [*Procedimiento: para habilitar identidades administradas para eventos de enrutamiento (versión preliminar)*](./how-to-enable-managed-identities-portal.md#add-a-system-managed-identity-during-instance-creation).
* **Etiquetas**: en esta pestaña, puede agregar etiquetas a la instancia a fin de facilitar su organización entre los recursos de Azure. Para obtener más información sobre las etiquetas de recurso de Azure, vea [*Etiquetado de recursos, grupos de recursos y suscripciones para una organización lógica*](../azure-resource-manager/management/tag-resources.md).

### <a name="verify-success-and-collect-important-values"></a>Comprobación de que la operación es correcta y recopilación de valores importantes

Después de finalizar la configuración de la instancia con la selección de **Crear**, puede ver el estado de la implementación en las notificaciones de Azure que hay en la barra de iconos del portal. La notificación le indicará cuándo se ha realizado la implementación correctamente y podrá seleccionar el botón _Ir al recurso_ para ver la instancia creada.

:::image type="content" source="media/how-to-set-up-instance/portal/notifications-deployment.png" alt-text="Vista de las notificaciones de Azure que muestra una implementación correcta y resalta el botón Ir al recurso":::

Como alternativa, si se produce un error con la implementación, la notificación le indicará el motivo. Observe el consejo del mensaje de error y vuelva a intentar crear la instancia.

>[!TIP]
>Una vez creada la instancia, para volver a su página en cualquier momento, busque el nombre de la instancia en la barra de búsqueda de Azure Portal.

A partir de la página *Información general* de la instancia, anote su *Nombre*, *Grupo de recursos* y *Nombre de host*. Estos son todos los valores importantes que puede necesitar a medida que sigue trabajando con la instancia de Azure Digital Twins. Si otros usuarios van a programar en la instancia, debe compartirlos con ellos.

:::image type="content" source="media/how-to-set-up-instance/portal/instance-important-values.png" alt-text="Resaltado de los valores importantes en la página de información general de la instancia":::

Ahora tiene lista una instancia de Azure Digital Twins. A continuación, debe proporcionar los permisos de usuario de Azure adecuados para administrarla.

## <a name="set-up-user-access-permissions"></a>Configuración de permisos de acceso de usuarios

[!INCLUDE [digital-twins-setup-role-assignment.md](../../includes/digital-twins-setup-role-assignment.md)]

En primer lugar, abra la página de la instancia de Azure Digital Twins en Azure Portal. En el menú de la instancia, seleccione *Control de acceso (IAM)* . Seleccione el botón **+ Agregar** para agregar una nueva asignación de roles.

:::image type="content" source="media/how-to-set-up-instance/portal/add-role-assignment-1.png" alt-text="Selección de la opción para agregar una asignación de roles en la página Control de acceso (IAM)":::.

En la siguiente página *Add role assignment* (Agregar asignación de roles), rellene los valores (el usuario que los complete debe tener [permisos suficientes](#prerequisites-permission-requirements) en la suscripción de Azure):
* **Rol**: seleccione *Propietario de datos de Azure Digital Twins* en el menú desplegable.
* **Asignar acceso a**: Use *Usuario, grupo o entidad de servicio*
* **Select**: busque el nombre o la dirección de correo electrónico del usuario para realizar la asignación. Cuando seleccione el resultado, el usuario aparecerá en la sección *Miembros seleccionados*.

:::row:::
    :::column:::
        :::image type="content" source="media/how-to-set-up-instance/portal/add-role-assignment-2.png" alt-text="Relleno de los campos indicados en el diálogo Agregar una asignación de roles":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

Cuando acabe de introducir los detalles, pulse el botón *Guardar*.

### <a name="verify-success"></a>Comprobación de que la operación se ha completado correctamente

Puede ver la asignación de roles que ha configurado en *Control de acceso (IAM) > Asignaciones de roles*. El usuario debe aparecer en la lista con el rol *Propietario de datos de Azure Digital Twins*. 

:::image type="content" source="media/how-to-set-up-instance/portal/verify-role-assignment.png" alt-text="Vista de las asignaciones de roles para una instancia de Azure Digital Twins en Azure Portal":::

Ahora tiene lista una instancia de Azure Digital Twins y los permisos asignados para administrarla.

## <a name="next-steps"></a>Pasos siguientes

Pruebe las llamadas individuales de la API de REST en su instancia mediante los comandos de la CLI de Azure Digital Twins: 
* [Referencia de az dt](/cli/azure/dt)
* [*Procedimiento: Uso de la CLI de Azure Digital Twins*](how-to-use-cli.md).

O bien consulte cómo conectar una aplicación cliente a la instancia mediante el código de autenticación:
* [*Procedimiento: Escritura de código de autenticación de aplicación*](how-to-authenticate-client.md)