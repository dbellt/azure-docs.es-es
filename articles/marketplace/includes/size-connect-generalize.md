---
title: archivo de inclusión
description: archivo
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: include
author: mingshen-ms
ms.author: krsh
ms.date: 04/16/2021
ms.openlocfilehash: 7d94bd0a4a9fb50cb211fd227c3022a46beef502
ms.sourcegitcommit: 70ce9237435df04b03dd0f739f23d34930059fef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/05/2021
ms.locfileid: "111527559"
---
## <a name="generalize-the-image"></a>Generalizar la imagen

Todas las imágenes de Azure Marketplace deben ser reutilizables de forma genérica. Para lograrlo, el VHD del sistema operativo debe estar generalizado, una operación que quita todos los controladores de software y los identificadores específicos de la instancia de una máquina virtual.

### <a name="for-windows"></a>Para Windows

Los discos de sistema operativo Windows se generalizan con la herramienta [sysprep](/windows-hardware/manufacture/desktop/sysprep--system-preparation--overview). Si más adelante actualiza o vuelve a configurar el sistema operativo, debe ejecutar sysprep de nuevo.

> [!WARNING]
> Después de ejecutar sysprep, desactive la máquina virtual hasta que se implemente, ya que las actualizaciones pueden ejecutarse automáticamente. Este apagado evita que las actualizaciones posteriores realicen cambios específicos de la instancia en el sistema operativo o los servicios instalados. Para más información sobre la ejecución de sysprep, vea [Pasos para generalizar un disco duro virtual](../../virtual-machines/windows/capture-image-resource.md#generalize-the-windows-vm-using-sysprep).

### <a name="for-linux"></a>Para Linux

El siguiente proceso generaliza una máquina virtual Linux y la vuelve a implementar como una máquina virtual independiente. Para obtener información detallada, vea [Creación de una imagen de una máquina virtual o un disco duro virtual](../../virtual-machines/linux/capture-image.md). Puede detenerse cuando llegue a la sección denominada "Crear una máquina virtual a partir de la imagen capturada".

1. Quite el agente Linux de Azure.
    1. Conexión a una máquina virtual Linux de Azure mediante un cliente SSH.
    2. En la ventana de SSH, escriba este comando: `sudo waagent –deprovision+user`.
    3. Escriba Y para continuar (puede agregar el parámetro -force al comando anterior para evitar el paso de confirmación).
    4. Una vez finalizado el comando, escriba **Exit** para cerrar el cliente de SSH.
2. Detenga la máquina virtual.
    1. En Azure Portal, seleccione el grupo de recursos (RG) y anule la asignación de la máquina virtual.
    2. La máquina virtual ahora se ha generalizado y puede crear una nueva mediante este disco de máquina virtual.

### <a name="capture-image"></a>Capture la imagen

> [!NOTE]
> La suscripción de Azure que contiene la SIG debe estar en el mismo inquilino que la cuenta de publicador para poder publicar. Además, la cuenta de publicador debe tener al menos acceso de colaborador a la suscripción que contiene la SIG.

Una vez que la VM esté lista, puede capturarla en una instancia de Azure Shared Image Gallery. Siga estos pasos para realizar la captura:

1. En [Azure Portal](https://ms.portal.azure.com/), vaya a la página de la máquina virtual.
2. Seleccione **Capturar**.
3. En **Share image to Shared image gallery** (Compartir imagen en Shared Image Gallery), seleccione **Yes, share it to a gallery as an image version** (Sí, compartirla en una galería como una versión de la imagen).
4. En **Estado del sistema operativo**, seleccione Generalizado.
5. Seleccione una galería de imágenes de destino o la opción para **Crear nuevo**.
6. Seleccione una definición de imágenes de destino o la opción para **Crear nuevo**.
7. Proporcione un **Número de versión** para la imagen.
8. Seleccione **Revisar y crear** para revisar sus selecciones.
9. Una vez que pase la validación, seleccione **Crear**.

## <a name="set-the-right-permissions"></a>Definición de los permisos correctos

Si la cuenta del Centro de partners es el propietario de la suscripción que hospeda Shared Image Gallery, no se necesita nada más para los permisos.

Si solo tiene acceso de lectura a la suscripción, use una de las dos opciones siguientes.

### <a name="option-one--ask-the-owner-to-grant-owner-permission"></a>Opción uno: pedir al propietario que conceda permiso de propietario

Pasos para que el propietario conceda permiso de propietario:

1. Vaya a la instancia de Shared Image Gallery (SIG).
2. Seleccione **Control de acceso** (IAM) en el panel de la izquierda.
3. Seleccione **Agregar** y, a continuación, **Agregar asignación de roles**.<br>
    :::image type="content" source="../media/create-vm/add-role-assignment.png" alt-text="Se muestra la ventana Agregar asignación de roles.":::
1. En **Rol**, seleccione **Propietario**.
1. En **Asignar acceso a**, seleccione **User, group, or service principle** (Usuario, grupo o entidad de servicio).
1. En **Seleccionar**, escriba el correo electrónico de Azure de la persona que publicará la imagen.
1. Seleccione **Guardar**.

### <a name="option-two--run-a-command"></a>Opción dos: ejecutar un comando

Pida al propietario que ejecute cualquiera de estos comandos (en cualquier caso, use el susbscriptionId de la suscripción en la que creó Shared Image Gallery).

```azurecli
az login
az provider register --namespace Microsoft.PartnerCenterIngestion --subscription {subscriptionId}
```
 
```powershell
Connect-AzAccount
Select-AzSubscription -SubscriptionId {subscriptionId}
Register-AzResourceProvider -ProviderNamespace Microsoft.PartnerCenterIngestion
```

> [!NOTE]
> No es necesario generar URI de SAS, ya que ahora puede publicar una imagen de SIG en el Centro de partners. Sin embargo, si todavía necesita hacer referencia a los pasos de generación de URI de SAS, consulte [Generación de un URI de SAS para una imagen de máquina virtual](../azure-vm-get-sas-uri.md).
