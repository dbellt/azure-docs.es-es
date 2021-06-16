---
title: Implementación de recursos con Azure Portal
description: Utilice Azure Portal y Azure Resource Manager para implementar los recursos en un grupo de recursos de su suscripción.
ms.topic: conceptual
ms.date: 05/05/2021
ms.openlocfilehash: 718d8be4e51bf41467bea398440b24f98ac1f6e9
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111957900"
---
# <a name="deploy-resources-with-arm-templates-and-azure-portal"></a>Implementación de recursos con plantillas de ARM y Azure Portal

Aprenda a usar [Azure Portal](https://portal.azure.com) con [plantillas de Azure Resource Manager (plantillas de Resource Manager)](overview.md) para implementar los recursos de Azure. Para más información sobre cómo administrar los recursos, consulte [Administración de recursos de Azure con Azure Portal](../management/manage-resources-portal.md).

La implementación de recursos de Azure mediante Azure Portal implica normalmente dos pasos:

- Cree un grupo de recursos.
- Implemente recursos en el grupo de recursos.

Además, puede crear una plantilla de Resource Manager personalizada para implementar recursos de Azure.

En el artículo se muestran ambos métodos.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

1. Para crear un grupo de recursos, seleccione **Grupos de recursos** en [Azure Portal](https://portal.azure.com).

   ![Seleccionar Grupos de recursos](./media/deploy-portal/select-resource-groups.png)

1. En Grupo de recursos, seleccione **Agregar**.

   ![Agregar un grupo de recursos](./media/deploy-portal/add-resource-group.png)

1. Seleccione o escriba los siguientes valores de propiedad:

    - **Suscripción**: Seleccione una suscripción de Azure.
    - **Grupo de recursos**: asigne un nombre al grupo de recursos.
    - **Región**: seleccione una ubicación de Azure. Esta ubicación es donde los grupos de recursos almacenan los metadatos sobre los recursos. Por motivos de cumplimiento, debería especificar dónde se almacenan esos metadatos. Por lo general, se recomienda especificar una ubicación donde estará la mayoría de los recursos. Si usa la misma ubicación, puede simplificar la plantilla.

   ![Establecer valores de grupo](./media/deploy-portal/set-group-properties.png)

1. Seleccione **Revisar + crear**.
1. Revise los valores y luego seleccione **Crear**.
1. Para poder ver el nuevo grupo de recursos en la lista, seleccione **Actualizar**.

## <a name="deploy-resources-to-a-resource-group"></a>Implementación de recursos en un grupo de recursos

Una vez creado el grupo de recursos, puede implementar recursos en él desde Marketplace. Marketplace proporciona soluciones predefinidas para escenarios habituales.

1. Para iniciar una implementación, seleccione **Crear un recurso** en [Azure Portal](https://portal.azure.com).

   ![Nuevo recurso](./media/deploy-portal/new-resources.png)

1. Busque el tipo de recurso que quiera implementar. Los recursos se organizan en categorías. Si no ve la solución específica que gustaría implementar, búsquela en Marketplace. En la siguiente captura de pantalla se muestra que se ha seleccionado Ubuntu Server.

   ![Selección del tipo de recurso](./media/deploy-portal/select-resource-type.png)

1. Según el tipo del recurso seleccionado, tiene una colección de propiedades pertinentes que debe establecer antes de la implementación. Para todos los tipos, debe seleccionar un grupo de recursos de destino. En la imagen siguiente se muestra cómo crear una máquina virtual Linux e implementarla en el grupo de recursos que ha creado.

   ![Creación de un grupo de recursos](./media/deploy-portal/select-existing-group.png)

   Puede decidir crear un nuevo grupo de recursos al implementar los recursos. Seleccione **Crear nuevo** y asígnele un nombre al grupo de recursos.

1. Comenzará la implementación. Esta operación puede tardar varios minutos. Algunos recursos tardan más que otros. Cuando haya terminado la implementación, verá una notificación. Seleccione **Ir a recurso**.

   ![Ver la notificación](./media/deploy-portal/view-notification.png)

1. Después de implementar los recursos, puede agregar más recursos al grupo de recursos si hace clic en **Agregar**.

   ![Agregar recurso](./media/deploy-portal/add-resource.png)

Aunque no lo vio, el portal usó una plantilla de Resource Manager para implementar los recursos seleccionados. Puede encontrar la plantilla en el historial de implementación. Para obtener más información, consulte [Exportación de la plantilla después de la implementación](export-template-portal.md#export-template-after-deployment).

## <a name="deploy-resources-from-custom-template"></a>Implementación de recursos desde plantilla personalizada

Si desea ejecutar una implementación sin usar las plantillas de Marketplace, puede crear una plantilla personalizada que defina la infraestructura para la solución. Para obtener información sobre cómo crear plantillas, consulte [Nociones sobre la estructura y la sintaxis de las plantillas de ARM](./syntax.md).

> [!NOTE]
> La interfaz del portal no admite referencias a un [secreto desde Key Vault](key-vault-parameter.md). En su lugar, use [PowerShell](deploy-powershell.md) o la [CLI de Azure](deploy-cli.md) para implementar la plantilla localmente o desde un URI externo.

1. Para implementar una plantilla personalizada mediante el portal, seleccione **Crear un recurso** y busque **plantilla**. Luego, seleccione **Implementación de plantillas**.

   ![Buscar la implementación de plantilla](./media/deploy-portal/search-template.png)

1. Seleccione **Crear**.
1. Tiene varias opciones para crear una plantilla:

    - **Crear su propia plantilla en el editor**: cree su propia plantilla en el editor de plantillas del portal.
    - **Plantillas comunes**: seleccione entre soluciones comunes.
    - **Cargar una plantilla de inicio rápido de GitHub**: seleccione entre las [plantillas de inicio rápido](https://azure.microsoft.com/resources/templates/).

   ![Ver las opciones](./media/deploy-portal/see-options.png)

    En este tutorial se proporcionan las instrucciones para cargar una plantilla de inicio rápido.

1. En **Cargar una plantilla de inicio rápido de GitHub**, escriba o seleccione **storage-account-create**.

    Tiene dos opciones:

    - **Seleccionar plantilla**: implemente la plantilla.
    - **Editar plantilla**: edite la plantilla de inicio rápido antes de implementarla.

1. Seleccione **Editar plantilla** para explorar el editor de plantillas del portal. La plantilla se carga en el editor. Observe que hay dos parámetros: `storageAccountType` y `location`.

   ![Creación de una plantilla](./media/deploy-portal/show-json.png)

1. Realice un cambio menor en la plantilla. Por ejemplo, actualice la variable `storageAccountName` a:

    ```json
    "storageAccountName": "[concat('azstore', uniquestring(resourceGroup().id))]"
    ```

1. Seleccione **Guardar**. Ahora verá la interfaz de implementación de plantillas del portal. Observe los dos parámetros que definió en la plantilla.
1. Escriba o seleccione los valores de propiedad:

    - **Suscripción**: Seleccione una suscripción de Azure.
    - **Grupo de recursos**: seleccione **Crear nuevo** y asígnele un nombre.
    - **Ubicación**: Seleccione una ubicación de Azure.
    - **Tipo de cuenta de almacenamiento**: Use el valor predeterminado. El nombre del parámetro con mayúsculas y minúsculas Camel, *storageAccountType*, definido en la plantilla se convierte en una cadena separada por espacios cuando se muestra en el portal.
    - **Ubicación**: Use el valor predeterminado.
    - **Acepto los términos y condiciones indicados anteriormente**: (seleccionar)

1. Seleccione **Comprar**.

## <a name="next-steps"></a>Pasos siguientes

- Para ver los registros de auditoría, consulte [Operaciones de auditoría con Resource Manager](../management/view-activity-logs.md).
- Para solucionar errores de implementación, vea [View deployment operations](deployment-history.md) (Ver operaciones de implementación).
- Para exportar una plantilla de una implementación o un grupo de recursos, consulte [Exportación de plantillas de ARM](export-template-portal.md).