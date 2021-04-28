---
title: Creación de una oferta de máquina virtual de Azure en Azure Marketplace mediante su propia imagen
description: Aprenda a publicar una oferta de máquina virtual de Azure en Azure Marketplace mediante su propia imagen.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: krsh
ms.author: krsh
ms.date: 04/16/2021
ms.openlocfilehash: 47fe7b42b68ae42f74a74e5fc69c8d1041d3bf8d
ms.sourcegitcommit: 6f1aa680588f5db41ed7fc78c934452d468ddb84
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107727131"
---
# <a name="how-to-create-a-virtual-machine-using-your-own-image"></a>Procedimiento para crear una máquina virtual mediante su propia imagen

En este artículo se describe cómo crear e implementar una imagen de máquina virtual (VM) proporcionada por el usuario.

> [!NOTE]
> Antes de comenzar este procedimiento, examine los [requisitos técnicos](marketplace-virtual-machines.md#technical-requirements) de las ofertas de máquina virtual de Azure, incluidos los requisitos de disco duro virtual (VHD).

Para usar una imagen base aprobada, siga las instrucciones de [Creación de una imagen de máquina virtual a partir de una base aprobada](azure-vm-create-using-approved-base.md).

## <a name="configure-the-vm"></a>Configuración de la máquina virtual

En esta sección se describe cómo ajustar el tamaño, actualizar y generalizar una máquina virtual de Azure. Estos pasos son necesarios para preparar la máquina virtual para su implementación en Azure Marketplace.

### <a name="size-the-vhds"></a>Ajuste del tamaño de los discos duros virtuales

[!INCLUDE [Discussion of VHD sizing](includes/vhd-size.md)]

### <a name="install-the-most-current-updates"></a>Instalar las actualizaciones más recientes

[!INCLUDE [Discussion of most current updates](includes/most-current-updates.md)]

### <a name="perform-more-security-checks"></a>Realización de comprobaciones de seguridad

[!INCLUDE [Discussion of addition security checks](includes/additional-security-checks.md)]

### <a name="perform-custom-configuration-and-scheduled-tasks"></a>Realizar una configuración personalizada y tareas programadas

[!INCLUDE [Discussion of custom configuration and scheduled tasks](includes/custom-config.md)]

### <a name="generalize-the-image"></a>Generalizar la imagen

Todas las imágenes de Azure Marketplace deben ser reutilizables de forma genérica. Para lograrlo, el VHD del sistema operativo debe estar generalizado, una operación que quita todos los controladores de software y los identificadores específicos de la instancia de una máquina virtual.

## <a name="bring-your-image-into-azure"></a>Incorporación de su imagen en Azure

> [!NOTE]
> La suscripción de Azure que contiene la SIG debe estar en el mismo inquilino que la cuenta de publicador para poder publicar. Además, la cuenta de publicador debe tener al menos acceso de colaborador a la suscripción que contiene la SIG.

Hay tres maneras de llevar la imagen a Azure:

1. Cargue el disco duro virtual en una instancia de Shared Image Gallery (SIG).
1. Cargue el disco duro virtual en una cuenta de Azure Storage.
1. Extraiga el disco duro virtual de una imagen administrada (si se usan servicios de creación de imágenes).

En las siguientes tres secciones se describen estas opciones.

### <a name="option-1-upload-the-vhd-as-shared-image-gallery"></a>Opción 1: carga del disco duro virtual como Shared Image Gallery

1. Cargue los discos duros virtuales en la cuenta de almacenamiento.
2. En Azure Portal, busque la opción **Implementar una plantilla personalizada**.
3. Seleccione **Cree su propia plantilla en el editor**.
4. Copie la siguiente plantilla de Azure Resource Manager (ARM).

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "sourceStorageAccountResourceId": {
          "type": "string",
          "metadata": {
            "description": "Resource ID of the source storage account that the blob vhd resides in."
          }
        },
        "sourceBlobUri": {
          "type": "string",
          "metadata": {
            "description": "Blob Uri of the vhd blob (must be in the storage account provided.)"
          }
        },
        "sourceBlobDataDisk0Uri": {
          "type": "string",
          "metadata": {
            "description": "Blob Uri of the vhd blob (must be in the storage account provided.)"
          }
        },
        "sourceBlobDataDisk1Uri": {
          "type": "string",
          "metadata": {
            "description": "Blob Uri of the vhd blob (must be in the storage account provided.)"
          }
        },
        "galleryName": {
          "type": "string",
          "metadata": {
            "description": "Name of the Shared Image Gallery."
          }
        },
        "galleryImageDefinitionName": {
          "type": "string",
          "metadata": {
            "description": "Name of the Image Definition."
          }
        },
        "galleryImageVersionName": {
          "type": "string",
          "metadata": {
            "description": "Name of the Image Version - should follow <MajorVersion>.<MinorVersion>.<Patch>."
          }
        }
      },
      "resources": [
        {
          "type": "Microsoft.Compute/galleries/images/versions",
          "name": "[concat(parameters('galleryName'), '/', parameters('galleryImageDefinitionName'), '/', parameters('galleryImageVersionName'))]",
          "apiVersion": "2020-09-30",
          "location": "[resourceGroup().location]",
          "properties": {
            "storageProfile": {
              "osDiskImage": {
                "source": {
                  "id": "[parameters('sourceStorageAccountResourceId')]",
                  "uri": "[parameters('sourceBlobUri')]"
                }
              },
    
              "dataDiskImages": [
                {
                  "lun": 0,
                  "source": {
                    "id": "[parameters('sourceStorageAccountResourceId')]",
                    "uri": "[parameters('sourceBlobDataDisk0Uri')]"
                  }
                },
                {
                  "lun": 1,
                  "source": {
                    "id": "[parameters('sourceStorageAccountResourceId')]",
                    "uri": "[parameters('sourceBlobDataDisk1Uri')]"
                  }
                }
              ]
            }
          }
        }
      ]
    }
    
    ```

5. Péguela en el editor.

    :::image type="content" source="media/create-vm/vm-sample-code-screen.png" alt-text="Pantalla de código de ejemplo para la máquina virtual.":::

1. Seleccione **Guardar**.
1. Use los parámetros de esta tabla para completar los campos de la siguiente pantalla.

| Parámetros | Descripción |
| --- | --- |
| sourceStorageAccountResourceId | Id. del recurso de la cuenta de almacenamiento de origen en la que reside el disco duro virtual del blob.<br><br>Para obtener el id. del recurso, vaya a su **cuenta de almacenamiento**  en **Azure Portal**, diríjase a **Propiedades** y copie el valor de **ResourceID**. |
| sourceBlobUri | URI del blob del disco duro virtual del disco del sistema operativo (debe estar en la cuenta de almacenamiento proporcionada).<br><br>Para obtener la URL del blob, vaya a su **cuenta de almacenamiento**  en **Azure Portal**, diríjase al **blob** y copie el valor de **URL**. |
| sourceBlobDataDisk0Uri | URI del blob del disco duro virtual del disco de datos (debe estar en la cuenta de almacenamiento proporcionada). Si no tiene ningún disco de datos, quite este parámetro de la plantilla.<br><br>Para obtener la URL del blob, vaya a su **cuenta de almacenamiento**  en **Azure Portal**, diríjase al **blob** y copie el valor de **URL**. |
| sourceBlobDataDisk1Uri | URI del blob del disco duro virtual del disco de datos adicional (debe estar en la cuenta de almacenamiento proporcionada). Si no tiene ningún disco de datos adicional, quite este parámetro de la plantilla.<br><br>Para obtener la URL del blob, vaya a su **cuenta de almacenamiento**  en **Azure Portal**, diríjase al **blob** y copie el valor de **URL**. |
| galleryName | Nombre de la instancia de Shared Image Gallery |
| galleryImageDefinitionName | Nombre de la definición de la imagen |
| galleryImageVersionName | Nombre de la versión de la imagen que se creará, en este formato: `<MajorVersion>.<MinorVersion>.<Patch>`. |
|

:::image type="content" source="media/create-vm/custom-deployment-window.png" alt-text="Muestra la ventana de implementación personalizada.":::

8. Seleccione **Revisar + crear**. Una vez que finalice la validación, seleccione **Crear**.

> [!TIP]
> La cuenta de editor debe tener acceso de "Propietario" para publicar la imagen SIG. Si es necesario, siga estos pasos para conceder acceso:
>
> 1. Vaya a la instancia de Shared Image Gallery (SIG).
> 2. Seleccione **Control de acceso** (IAM) en el panel de la izquierda.
> 3. Seleccione **Agregar** y, a continuación, **Agregar asignación de roles**.
> 4. En **Rol**, seleccione **Propietario**.
> 5. En **Asignar acceso a**, seleccione **User, group, or service principle** (Usuario, grupo o entidad de servicio).
> 6. Escriba el correo electrónico de Azure de la persona que publicará la imagen.
> 7. Seleccione **Guardar**.<br><br>
> :::image type="content" source="media/create-vm/add-role-assignment.png" alt-text="Se muestra la ventana Agregar asignación de roles.":::

### <a name="option-2-upload-the-vhd-to-a-storage-account"></a>Opción 2: carga del disco duro virtual en una cuenta de almacenamiento

Configure y prepare la máquina virtual para que se cargue como se describe en los artículos sobre [preparación de un VHD o un VHDX de Windows antes de cargarlo en Azure](../virtual-machines/windows/prepare-for-upload-vhd-image.md) o [creación y carga de un disco duro virtual Linux](../virtual-machines/linux/create-upload-generic.md).

### <a name="option-3-extract-the-vhd-from-managed-image-if-using-image-building-services"></a>Opción 3: extracción del disco duro virtual de la imagen administrada (si se usan servicios de creación de imágenes)

Si usa un servicio de creación de imágenes como [Packer](https://www.packer.io/), es posible que deba extraer el disco duro virtual de la imagen. No hay ninguna forma directa de hacerlo. Tendrá que crear una máquina virtual y extraer el disco duro virtual desde el disco de la máquina virtual.

## <a name="create-the-vm-on-the-azure-portal"></a>Creación de la máquina virtual en Azure Portal

Siga estos pasos para crear la imagen base de la máquina virtual en [Azure Portal](https://ms.portal.azure.com/).

1. Inicie sesión en [Azure Portal](https://ms.portal.azure.com/).
2. Seleccione **Máquinas virtuales**.
3. Seleccione **+ Agregar** para abrir la pantalla **Crear una máquina virtual**.
4. Seleccione la imagen en la lista desplegable, o bien **Examinar todas las imágenes públicas y privadas** si desea buscar o examinar todas las imágenes de máquina virtual disponibles.
5. Para crear una máquina virtual de **segunda generación**, vaya a la pestaña **Opciones avanzadas** y seleccione la opción **Generación 2**.

    :::image type="content" source="media/create-vm/vm-gen-option.png" alt-text="Selección Generación 1 o Generación 2.":::

6. Seleccione el tamaño de la máquina virtual que va a implementar.

    :::image type="content" source="media/create-vm/create-virtual-machine-sizes.png" alt-text="Seleccione un tamaño de máquina virtual recomendado para la imagen seleccionada.":::

7. Proporcione los otros detalles necesarios para crear la máquina virtual.
8. Seleccione **Revisar y crear** para revisar sus selecciones. Cuando aparezca el mensaje **Validación superada**, seleccione **Crear**.

Azure comienza el aprovisionamiento de la máquina virtual especificada. Para seguir su progreso, seleccione la pestaña **Máquinas virtuales** en el menú de la izquierda. En cuanto se crea, el estado de la máquina virtual cambia a **En ejecución**.

## <a name="connect-to-your-vm"></a>Conexión a la máquina virtual

Consulte la documentación siguiente para conectarse a una máquina virtual [Windows](../virtual-machines/windows/connect-logon.md) o [Linux](../virtual-machines/linux/ssh-from-windows.md#connect-to-your-vm).

[!INCLUDE [Discussion of addition security checks](includes/size-connect-generalize.md)]

## <a name="next-steps"></a>Pasos siguientes

- [Pruebe la imagen de la máquina virtual](azure-vm-image-test.md) para asegurarse de que cumple los requisitos de publicación de Azure Marketplace. Esto es opcional.
- Si no quiere probar la imagen de la máquina virtual, inicie sesión en el [Centro de partners](https://partner.microsoft.com/) y publique la imagen de SIG (opción 1).
- Si ha seguido la opción 2 o 3, [genere el URI de SAS](azure-vm-get-sas-uri.md).
- Si tiene dificultades para crear un disco duro virtual basado en Azure, consulte las [preguntas frecuentes que pueden surgir al crear una máquina virtual en Azure Marketplace](azure-vm-create-faq.md).
