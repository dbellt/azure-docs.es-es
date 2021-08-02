---
title: Uso de identidades administradas para acceder a Event Hubs desde un trabajo de Azure Stream Analytics (versión preliminar)
description: En este artículo se describe cómo usar identidades administradas para autenticar su trabajo de Azure Stream Analytics en la entrada y la salida de Azure Event Hubs.
author: sidramadoss
ms.author: sidram
ms.service: stream-analytics
ms.topic: how-to
ms.date: 06/07/2021
ms.openlocfilehash: c0f81c5abdffe307550d2b0b3e1f86f67f867f5a
ms.sourcegitcommit: b11257b15f7f16ed01b9a78c471debb81c30f20c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2021
ms.locfileid: "111591303"
---
# <a name="use-managed-identities-to-access-event-hubfrom-an-azure-stream-analytics-job-preview"></a>Uso de identidades administradas para acceder a Event Hubs desde un trabajo de Azure Stream Analytics (versión preliminar)

Azure Stream Analytics admite la autenticación de identidad administrada tanto para la entrada como para la salida de Azure Event Hubs. Las identidades administradas eliminan las limitaciones de los métodos de autenticación basada en el usuario, como la necesidad de volver a realizar la autenticación debido a los cambios de contraseña o la expiración de tokens de usuario que se produce cada 90 días. Cuando se elimina la necesidad de autenticarse manualmente, las implementaciones de Stream Analytics se pueden automatizar completamente.  

Una identidad administrada es una aplicación administrada registrada en Azure Active Directory que representa un trabajo de Stream Analytics determinado. La aplicación administrada se usa para autenticarse en un recurso de destino, incluido los centros de eventos que están detrás de un firewall o una red virtual (VNet). Para más información sobre cómo pasar por alto los firewalls, consulte [Permiso para acceder a los espacios de nombres de Azure Event Hubs a través de puntos de conexión privados](../event-hubs/private-link-service.md#trusted-microsoft-services).

En este artículo se muestra cómo habilitar la identidad administrada para la salida o entrada de Event Hubs de un trabajo de Stream Analytics a través de Azure Portal. Antes de habilitar la identidad administrada, primero debe tener un trabajo de Stream Analytics y un recurso de centro de eventos.

### <a name="limitation"></a>Limitación
En la versión preliminar, el muestreo de la entrada desde Event Hubs en Azure Portal no funcionará si se usa el modo de autenticación de identidad administrada.

## <a name="create-a-managedidentity"></a>Creación de una entidad administrada  

En primer lugar, debe crear una identidad administrada para el trabajo de Azure Stream Analytics.  

1. En Azure Portal, abra el trabajo de Azure Stream Analytics.  

1. En el menú de navegación de la izquierda, seleccione  **Identidad administrada** en  *Configurar*. A continuación, active la casilla situada junto a  **Usar la identidad administrada asignada por el sistema** y seleccione  **Guardar**.

   :::image type="content" source="media/event-hubs-managed-identity/system-assigned-managed-identity.png" alt-text="Identidad administrada asignada por el sistema":::  

1. Se crea una entidad de servicio para la identidad del trabajo de Stream Analytics en Azure Active Directory. El ciclo de vida de la identidad recién creada lo administrará Azure. Cuando se elimina el trabajo de Stream Analytics, Azure elimina automáticamente la identidad asociada (es decir, la entidad de servicio).  

   Cuando se guarda la configuración, el id. de objeto (OID) de la entidad de servicio aparece como id. de entidad de seguridad, tal como se muestra a continuación:  

   :::image type="content" source="media/event-hubs-managed-identity/principal-id.png" alt-text="Identificador de entidad de seguridad":::

   La entidad de servicio se llama igual que el trabajo de Stream Analytics. Por ejemplo, si el nombre del trabajo es  `MyASAJob`, el nombre de la entidad de servicio también será  `MyASAJob`.  

## <a name="grant-the-stream-analytics-job-permissionsto-access-the-event-hub"></a>Concesión de acceso al trabajo de Stream Analytics al centro de eventos

Para que el trabajo de Stream Analytics tenga acceso al centro de eventos con una identidad administrada, la entidad de servicio que creó debe tener permisos especiales para el centro de eventos.

1. Vaya a **Access Control (IAM)** en el centro de eventos.

1. Seleccione **+ Agregar** y **Agregar asignación de roles**.

1. En la página **Agregar asignación de roles**, especifique las opciones siguientes:

   |Parámetro|Valor|
   |---------|-----|
   |Role|Propietario de los datos de Azure Event Hubs|
   |Asignar acceso a|Usuario, grupo o entidad de servicio|
   |Seleccionar|Escriba el nombre del trabajo de Stream Analytics|

   :::image type="content" source="media/event-hubs-managed-identity/add-role-assignment.png" alt-text="Agregar asignación de roles":::

1. Seleccione **Guardar** y espere que los cambios se propaguen.

También puede conceder este rol en el nivel de espacio de nombres del centro de eventos, que propagará de forma natural los permisos a todos los centros de eventos creados en él. Es decir, todos los centros de eventos de un espacio de nombres se pueden usar como recurso de autenticación de identidad administrada en el trabajo de Stream Analytics.

> [!NOTE]
> Debido a la replicación global o la latencia de almacenamiento en caché, puede haber un retraso cuando se revocan o se conceden permisos. Los cambios deben reflejarse en un plazo de ocho minutos.

## <a name="create-anevent-hub-input-or-output"></a>Creación de una entrada o salida del centro de eventos  

Ahora que la identidad administrada está configurada, está listo para agregar el recurso del centro de eventos como entrada o salida a su trabajo de Stream Analytics.  

### <a name="add-the-event-hub-as-an-input"></a>Incorporación del centro de eventos como entrada 

1. Vaya al trabajo de Stream Analytics y navegue a la página **Entradas** en **Topología de trabajo**.

1. Seleccione **Agregar entrada de flujo > Centro de eventos**. En la ventana de propiedades de entrada, busque y seleccione el centro de eventos y seleccione **Identidad administrada** en el menú desplegable *Modo de autenticación*.

1. Rellene el resto de las propiedades y seleccione **Guardar**.

### <a name="add-the-event-hub-as-an-output"></a>Incorporación del centro de eventos como salida

1. Vaya al trabajo de Stream Analytics y navegue a la página **Salidas** en **Topología de trabajo**.

1. Seleccione **Agregar > Centro de eventos**. En la ventana de propiedades de salida, busque y seleccione el centro de eventos y seleccione **Identidad administrada** en el menú desplegable *Modo de autenticación*.

1. Rellene el resto de las propiedades y seleccione **Guardar**.

## <a name="next-steps"></a>Pasos siguientes

* [Salida de Event Hubs desde Azure Stream Analytics](event-hubs-output.md)
* [Datos de flujo de Event Hubs](stream-analytics-define-inputs.md#stream-data-from-event-hubs)
