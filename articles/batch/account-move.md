---
title: Traslado de una cuenta de Azure Batch a otra región
description: Obtenga información sobre cómo mover una cuenta de Azure Batch a otra región.
ms.topic: how-to
ms.date: 05/05/2021
ms.custom: subject-moving-resources
ms.openlocfilehash: 9c0e45b62f6dd6152ab80beaa751925702b1fa37
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108777072"
---
# <a name="move-an-azure-batch-account-to-another-region"></a>Traslado de una cuenta de Azure Batch a otra región

Hay escenarios en los que puede resultar útil mover una [cuenta de Batch](accounts.md) existente de una región a otra. También puede realizar el traslado a otra región como parte del planeamiento de la recuperación ante desastres.

Las cuentas de Azure Batch no se pueden mover directamente de una región a otra. Sin embargo, puede usar una plantilla de Azure Resource Manager para exportar la configuración actual de la cuenta de Batch. Después, para preparar el recurso en otra región, puede exportar la cuenta de Batch a una plantilla, modificar los parámetros para que coincidan con la región de destino y, a continuación, implementar la plantilla en la nueva región. A continuación, puede volver a crear trabajos y otras características en la cuenta.

 Para más información sobre Resource Manager y las plantillas, consulte [Inicio rápido: Creación e implementación de plantillas de Azure Resource Manager mediante Azure Portal](../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md).

En este tema se explica cómo mover una cuenta de Batch entre regiones mediante Azure Portal.

## <a name="prerequisites"></a>Requisitos previos

- Mueva la cuenta de almacenamiento asociada a la cuenta de Batch a la nueva región de destino siguiendo los pasos descritos en [Traslado de una cuenta de Azure Storage a otra región](../storage/common/storage-account-move.md). Si lo prefiere, puede dejar la cuenta de almacenamiento en la región original. Sin embargo, se recomienda moverla, ya que normalmente se obtendrá un mejor rendimiento si se encuentra en la misma región que la cuenta de Batch. En las instrucciones siguientes se supone que ya ha migrado la cuenta de almacenamiento.
- Asegúrese de que los servicios y las características que usa su cuenta de Batch se admitan en la región de destino.

## <a name="prepare"></a>Preparación

Para empezar, deberá exportar y luego modificar una plantilla de Resource Manager.

### <a name="export-a-template"></a>Exportación de una plantilla

En primer lugar, exporte una plantilla que contenga la configuración y la información de la cuenta de Batch.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

2. Seleccione **Todos los recursos** y seleccione su cuenta de Batch.

3. Seleccione > **Automation** > **Exportar plantilla**.

4. Elija **Descargar** en la hoja **Exportar plantilla**.

5. Busque el archivo ZIP que descargó desde el portal y descomprímalo en la carpeta que prefiera.

   Este archivo ZIP contiene los archivos .json que componen la plantilla y los scripts para implementar la plantilla.

### <a name="modify-the-template"></a>Modificación de la plantilla

A continuación, cargue y modifique la plantilla para poder crear una nueva cuenta de Batch en la región de destino.

1. En Azure Portal, haga clic en **Crear un recurso**.

1. En **Buscar en Marketplace**, escriba **implementación de plantillas** y, después, presione **ENTRAR**.

1. Seleccione **Implementación de plantillas (implementar mediante plantillas personalizadas)** .

1. Seleccione **Crear**.

1. Seleccione **Cree su propia plantilla en el editor**.

1. Seleccione **Cargar archivo** y, después, siga las instrucciones para cargar el archivo **template.json** que descargó en la última sección.

1. En el archivo **template.json** cargado, asigne un nombre a la cuenta de Batch de destino. Para ello, escriba un nuevo valor de **defaultValue** para el nombre de la cuenta de Batch. En este ejemplo se establece el valor de **defaultValue** del nombre de la cuenta de Batch en `mytargetaccount`. y se reemplaza la cadena en **defaultValue** por el identificador de recurso de `mytargetstorageaccount`.

    ```json
    {
        "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "batchAccounts_mysourceaccount_name": {
                "defaultValue": "mytargetaccount",
                "type": "String"
            }
        },
   ```

1. A continuación, actualice el valor de **defaultValue** de la cuenta de almacenamiento con el identificador de recurso de la cuenta de almacenamiento migrada. Para obtener este valor, vaya a la cuenta de almacenamiento en Azure Portal, seleccione **Vista JSON** cerca de la parte superior de la pantalla y, a continuación, copie el valor que se muestra en **Id. de recurso**. En este ejemplo se devuelve el identificador de recurso de la cuenta de almacenamiento denominada `mytargetstorageaccount` en el grupo de recursos `mytargetresourcegroup`.

   ```json
           "storageAccounts_mysourcestorageaccount_externalid": {
            "defaultValue": "/subscriptions/{subscriptionID}/resourceGroups/mytargetresourcegroup/providers/Microsoft.Storage/storageAccounts/mytargetstorageaccount",
            "type": "String"
        }
    },
   ```

1. Por último, edite la propiedad **location** para usar la región de destino. En este ejemplo, la región de destino se establece en `centralus`.

```json
    {
        "resources": [
            {
                "type": "Microsoft.Batch/batchAccounts",
                "apiVersion": "2021-01-01",
                "name": "[parameters('batchAccounts_mysourceaccount_name')]",
                "location": "centralus",  
```

Para obtener los códigos de ubicación de la región, consulte [Ubicaciones de Azure](https://azure.microsoft.com/global-infrastructure/locations/).  El código de una región es el nombre de la región sin espacios. Por ejemplo, **Centro de EE. UU** = **centralus**.

## <a name="move"></a>Mover

Implemente la plantilla para crear una nueva cuenta de Batch en la región de destino.

1. Ahora que ha realizado las modificaciones, seleccione **Guardar** debajo del archivo **template.json**.

1. Escriba o seleccione los valores de propiedad:
   - **Suscripción**: Seleccione una suscripción de Azure.
   - **Grupo de recursos**: seleccione el grupo de recursos que creó al mover la cuenta de almacenamiento asociada.
   - **Región**: seleccione la región de Azure a la que va a mover la cuenta.

1. Seleccione **Revisar y crear** y, luego, **Crear**.

### <a name="configure-the-new-batch-account"></a>Configuración de la cuenta de Batch nueva

Algunas características no se exportarán a una plantilla, por lo que tendrá que volver a crearlas en la nueva cuenta de Batch. Entre ellas, figuran:

- Trabajos
- Programación de trabajos
- Certificados
- paquetes de aplicación

Asegúrese de configurar estos valores en la nueva cuenta según sea necesario. Puede ver cómo ha configurado estas características en la cuenta de Batch de origen como referencia.

## <a name="discard-or-clean-up"></a>Descarte o limpieza

Una vez que haya confirmado que la nueva cuenta de Batch funciona correctamente en la nueva región y haya restaurado las características necesarias, puede eliminar la cuenta de Batch de origen.

Para quitar una cuenta de Batch mediante Azure Portal:

1. En Azure Portal, expanda el menú de la izquierda para abrir el menú de servicios y elija **Cuentas de Batch**.

2. Busque la cuenta de Batch que se va a eliminar y haga clic con el botón derecho en el botón **Más** ( **...** ) situado en la parte derecha de la lista. Asegúrese de que se trata de la cuenta de Batch de origen original, no la nueva que ha creado.

3. Seleccione **Eliminar** y, luego, confirme.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre el [traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](../azure-resource-manager/management/move-resource-group-and-subscription.md).
- Obtenga más información sobre el [Traslado de máquinas virtuales de Azure a otra región](../site-recovery/azure-to-azure-tutorial-migrate.md).
