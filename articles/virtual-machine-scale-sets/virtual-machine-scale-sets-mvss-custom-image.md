---
title: Referencia a una imagen personalizada en una plantilla de conjunto de escalado de Azure
description: Obtenga información sobre cómo agregar una imagen personalizada a una plantilla de un conjunto de escalado de máquinas virtuales de Microsoft Azure existente
author: cynthn
ms.author: cynthn
ms.topic: how-to
ms.service: virtual-machine-scale-sets
ms.subservice: shared-image-gallery
ms.date: 04/26/2018
ms.reviewer: mimckitt
ms.openlocfilehash: 56e27c7978ef95ad49e6db7592fe3c755af2b0ab
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108756150"
---
# <a name="add-a-custom-image-to-an-azure-scale-set-template"></a>Incorporación de una imagen personalizada a una plantilla de un conjunto de escalado de Azure

Este artículo le enseña cómo modificar la [plantilla de conjunto de escalado básico](virtual-machine-scale-sets-mvss-start.md) para implementarla a partir de una imagen personalizada.

## <a name="change-the-template-definition"></a>Cambio de la definición de la plantilla
En un [artículo anterior](virtual-machine-scale-sets-mvss-start.md) creamos una plantilla de conjunto de escalado básico. Ahora usaremos esa plantilla anterior y la modificaremos para crear una plantilla que implementa un conjunto de escalado a partir de una imagen personalizada.  

### <a name="creating-a-managed-disk-image"></a>Creación de una imagen de disco administrado

Si ya tiene una imagen de disco administrado personalizada (un recurso del tipo `Microsoft.Compute/images`), puede omitir esta sección.

En primer lugar, agregue un parámetro `sourceImageVhdUri`, que es el identificador URI del blob generalizado de Azure Storage que contiene la imagen personalizada desde la que se va a realizar la implementación.


```diff
     },
     "adminPassword": {
       "type": "securestring"
+    },
+    "sourceImageVhdUri": {
+      "type": "string",
+      "metadata": {
+        "description": "The source of the generalized blob containing the custom image"
+      }
     }
   },
   "variables": {},
```

A continuación, agregue un recurso de tipo `Microsoft.Compute/images`, que es la imagen de disco administrado basada en el blob generalizado ubicado en el URI `sourceImageVhdUri`. Esta imagen debe estar en la misma región que el conjunto de escalado que la utilice. En las propiedades de la imagen, especifique el tipo de sistema operativo, la ubicación del blob (desde el parámetro `sourceImageVhdUri`) y el tipo de cuenta de almacenamiento:

```diff
   "resources": [
     {
+      "type": "Microsoft.Compute/images",
+      "apiVersion": "2019-03-01",
+      "name": "myCustomImage",
+      "location": "[resourceGroup().location]",
+      "properties": {
+        "storageProfile": {
+          "osDisk": {
+            "osType": "Linux",
+            "osState": "Generalized",
+            "blobUri": "[parameters('sourceImageVhdUri')]",
+            "storageAccountType": "Standard_LRS"
+          }
+        }
+      }
+    },
+    {
       "type": "Microsoft.Network/virtualNetworks",
       "name": "myVnet",
       "location": "[resourceGroup().location]",

```

En el recurso del conjunto de escalado, agregue una cláusula `dependsOn` que haga referencia a la imagen personalizada para garantizar que la imagen se crea antes de que el conjunto de escalado se intente implementar desde ella:

```diff
       "location": "[resourceGroup().location]",
       "apiVersion": "2019-03-01-preview",
       "dependsOn": [
-        "Microsoft.Network/virtualNetworks/myVnet"
+        "Microsoft.Network/virtualNetworks/myVnet",
+        "Microsoft.Compute/images/myCustomImage"
       ],
       "sku": {
         "name": "Standard_A1",

```

### <a name="changing-scale-set-properties-to-use-the-managed-disk-image"></a>Cambio en las propiedades del conjunto de escalado para usar la imagen de disco administrado

En `imageReference` del conjunto de escalado `storageProfile`, en lugar de especificar el publicador, la oferta, la SKU y la versión de una imagen de la plataforma, especifique el `id` del recurso `Microsoft.Compute/images`:

```json
         "virtualMachineProfile": {
           "storageProfile": {
             "imageReference": {
              "id": "[resourceId('Microsoft.Compute/images', 'myCustomImage')]"
             }
           },
           "osProfile": {
```

En este ejemplo, use la función `resourceId` para obtener el identificador de recurso de la imagen que se creó en la misma plantilla. Si ha creado la imagen de disco administrado con antelación, deberá proporcionar el identificador de esa imagen en su lugar. Este identificador debe tener el formato: `/subscriptions/<subscription-id>resourceGroups/<resource-group-name>/providers/Microsoft.Compute/images/<image-name>`.


## <a name="next-steps"></a>Pasos siguientes

[!INCLUDE [mvss-next-steps-include](../../includes/mvss-next-steps.md)]
