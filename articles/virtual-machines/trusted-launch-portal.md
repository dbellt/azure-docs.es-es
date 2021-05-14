---
title: 'Versión preliminar: implementación de una máquina virtual con inicio seguro'
description: Implemente una máquina virtual que use el inicio seguro.
author: khyewei
ms.author: khwei
ms.reviewer: cynthn
ms.service: virtual-machines
ms.subservice: trusted-launch
ms.topic: how-to
ms.date: 04/06/2021
ms.custom: template-how-to
ms.openlocfilehash: 295579d17f3b24adcf43f6907cc4b1aca01dcae2
ms.sourcegitcommit: 590f14d35e831a2dbb803fc12ebbd3ed2046abff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107565923"
---
# <a name="deploy-a-vm-with-trusted-launch-enabled-preview"></a>Implementación de una máquina virtual con el inicio seguro habilitado (versión preliminar)

El [inicio seguro](trusted-launch.md) es una manera de mejorar la seguridad de las máquinas virtuales de [generación 2](generation-2.md). Protege frente a técnicas de ataque persistentes y avanzadas, gracias a la combinación de tecnologías de infraestructura como vTPM y el arranque seguro.

> [!IMPORTANT]
> El inicio seguro está actualmente en versión preliminar pública.
> 
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
>
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="deploy-using-the-portal"></a>Implementación mediante Azure Portal

Cree una máquina virtual con el inicio seguro habilitado.

1. Inicie sesión en [Azure Portal](https://aka.ms/TL_preview).
   > [!NOTE] 
   > El vínculo del portal es único de la versión preliminar de inicio seguro.
   >  
2. Busque **Máquinas virtuales**.
3. En **Servicios**, seleccione **Máquinas virtuales**.
4. En la página **Máquinas virtuales**, seleccione **Agregar** y, después, **Máquina virtual**.
5. En **Detalles del proyecto**, asegúrese de que está seleccionada la suscripción correcta.
6. En **Grupo de recursos**, seleccione **Crear nuevo** y especifique un nombre para el grupo de recursos, o bien seleccione un grupo de recursos existente en la lista desplegable.
7. En **Detalles de la instancia**, escriba un nombre para la máquina virtual y elija una región que admita el [inicio seguro](trusted-launch.md#public-preview-limitations).
8. En **Imagen**, seleccione una [imagen de segunda generación que admita el inicio seguro](trusted-launch.md#public-preview-limitations). Asegúrese de que aparece el siguiente mensaje: **Esta imagen admite la versión preliminar del inicio seguro. Configúrela en la pestaña Avanzado**.
   > [!TIP]
   > Si no ve la versión de segunda generación de la imagen que desea en la lista desplegable, seleccione **Ver todas las imágenes** y cambie el filtro **Generación de VM** para que se muestren solo imágenes de segunda generación. Busque la imagen en la lista y, después, use la lista desplegable **Seleccionar** para seleccionar la versión de segunda generación.

    :::image type="content" source="media/trusted-launch/gen-2-image.png" alt-text="Captura de pantalla que muestra el mensaje que confirma que se trata de una imagen de generación 2 compatible con el inicio seguro.":::

13. Seleccione un tamaño de máquina virtual que admita el inicio seguro. Consulte la lista de [tamaños compatibles](trusted-launch.md#public-preview-limitations).
14. Rellene la información de **Cuenta de administrador** y **Reglas del puerto de entrada**. 
1. Seleccione **Avanzado** en la parte superior de la página para pasar a esta pestaña.
1. Desplácese hacia abajo hasta la sección **Generación de VM**. Asegúrese de **Gen 2** está seleccionado.
1. Todavía en la pestaña **Avanzado**, desplácese hacia abajo hasta **Inicio seguro** y active la casilla **Inicio seguro**. Esto hará que aparezcan dos opciones más: Arranque seguro y vTPM. Seleccione las opciones adecuadas para su implementación.

    :::image type="content" source="media/trusted-launch/trusted-launch-portal.png" alt-text="Captura de pantalla que muestra las opciones de inicio seguro.":::

15. En la parte inferior de la página, seleccione **Revisar y crear**.
16. En la página **Crear una máquina virtual** verá los detalles de la máquina virtual que va a implementar. Cuando esté preparado, seleccione **Crear**.

    :::image type="content" source="media/trusted-launch/validation.png" alt-text="Captura de pantalla de la página de validación, donde se muestran las opciones de inicio seguro incluidas.":::


La implementación de la máquina virtual tardará unos minutos. 

## <a name="deploy-using-a-template"></a>Implementación mediante una plantilla

Puede implementar máquinas virtuales con inicio seguro mediante una plantilla de inicio rápido:

**Linux**:    
[![Implementar en Azure](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.svg?sanitize=true)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-vm-trustedlaunch-linux%2Fazuredeploy.json/createUIDefinitionUri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-vm-trustedlaunch-linux%2FcreateUiDefinition.json)

**Windows**:    
[![Implementar en Azure](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.svg?sanitize=true)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-vm-trustedlaunch-windows%2Fazuredeploy.json/createUIDefinitionUri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-vm-trustedlaunch-windows%2FcreateUiDefinition.json)

## <a name="view-and-update"></a>Consulta y actualización

Puede ver la configuración del inicio seguro de una máquina virtual existente en su página **Información general** en el portal.

Para cambiar la configuración del inicio seguro, en el menú de la izquierda, seleccione **Configuración** en la sección **Configuración**. En la sección **Inicio seguro**, puede habilitar o deshabilitar el arranque seguro y vTPM. Cuando haya terminado, seleccione **Guardar** en la parte superior de la página. 

:::image type="content" source="media/trusted-launch/configuration.png" alt-text="Captura de pantalla sobre cómo cambiar la configuración de inicio seguro.":::

Si la máquina virtual se está ejecutando, aparecerá un mensaje que le indicará que la máquina virtual se reiniciará para aplicar la configuración de inicio seguro modificada. Seleccione **Sí** y espere a que se reinicie la máquina virtual para que los cambios surtan efecto.


## <a name="verify-secure-boot-and-vtpm"></a>Comprobación del arranque seguro y vTPM

Puede comprobar si el arranque seguro y vTPM están habilitados en la máquina virtual.
    
### <a name="linux-validate-if-secure-boot-is-running"></a>Linux: comprobar si está habilitado el arranque seguro

Conéctese mediante SSH a la máquina virtual y, a continuación, ejecute el siguiente comando: 

```bash
mokutil --sb-state
```

Si el arranque seguro está habilitado, el comando devolverá:
 
```bash
SecureBoot enabled 
```

### <a name="linux-validate-if-vtpm-is-enabled"></a>Linux: comprobar si está habilitado vTPM

Utilice SSH en la máquina virtual. Compruebe si el dispositivo tpm0 está presente: 

```bash
ls /dev/tpm0
```

Si vTPM está habilitado, el comando devolverá:

```output
/dev/tpm0
```

Si vTPM está deshabilitado, el comando devolverá:

```output
ls: cannot access '/dev/tpm0': No such file or directory
```

### <a name="windows-validate-that-secure-boot-is-running"></a>Windows: comprobar si está habilitado el arranque seguro

Conéctese a la máquina virtual mediante Escritorio remoto y, después, ejecute `msinfo32.exe`.

En el panel derecho, compruebe que Estado de arranque seguro muestra **Activado**.

## <a name="enable-the-azure-security-center-experience"></a>Habilitación de la experiencia de Azure Security Center

Para que Azure Security Center muestre información sobre las máquinas virtuales con inicio seguro, debe habilitar varias directivas. La forma más fácil de habilitar las directivas es mediante la implementación de esta [plantilla de Resource Manager](https://github.com/prash200/azure-quickstart-templates/tree/master/101-asc-trustedlaunch-policies) en su suscripción. 

Seleccione el botón siguiente para implementar las directivas en la suscripción:

[![Implementar en Azure](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.svg?sanitize=true)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fprash200%2Fazure-quickstart-templates%2Fmaster%2F101-asc-trustedlaunch-policies%2Fazuredeploy.json)

La plantilla solo debe implementarse una vez por cada suscripción. Instala automáticamente las extensiones `GuestAttestation` y `AzureSecurity` en todas las máquinas virtuales compatibles. Si aparecen errores, intente volver a implementar la plantilla.

Para conocer las recomendaciones sobre vTPM y arranque seguro en máquinas virtuales con inicio seguro, consulte [Agregar una iniciativa personalizada a la suscripción](../security-center/custom-security-policies.md#to-add-a-custom-initiative-to-your-subscription).
 
## <a name="sign-things-for-secure-boot-on-linux"></a>Firma de elementos para el arranque seguro en Linux

En algunos casos, puede que necesite firmar algunos elementos para disponer del arranque seguro de UEFI.  Por ejemplo, puede que tenga que seguir el [procedimiento para firmar elementos de arranque seguro](https://ubuntu.com/blog/how-to-sign-things-for-secure-boot) para Ubuntu. En estos casos, debe especificar las claves de inscripción de la utilidad MOK para la máquina virtual. Deberá usar la consola serie de Azure para acceder a la utilidad MOK.

1. Habilite la consola serie de Azure para Linux. Para obtener más información, consulte [Consola serie para Linux](/troubleshoot/azure/virtual-machines/serial-console-linux).
1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Busque **Máquinas virtuales** y seleccione la máquina virtual en la lista.
1. En **Soporte técnico y solución de problemas**, seleccione **Consola serie**. Se abrirá una página a la derecha, con la consola serie.
1. Inicie sesión en la máquina virtual mediante la consola serie de Azure. Como **inicio de sesión**, escriba el nombre de usuario que usó al crear la máquina virtual. Por ejemplo, *azureuser*. Cuando se le solicite, escriba la contraseña asociada con el nombre de usuario.
1. Una vez que haya iniciado sesión, use `mokutil` para importar el archivo de clave pública `.der`.

    ```bash
    sudo mokutil –import <path to public key.der> 
    ```
1. Reinicie la máquina desde la consola serie de Azure; para ello, escriba `sudo reboot`. Se iniciará una cuenta atrás de 10 segundos.
1. Presione la tecla de flecha arriba o abajo para interrumpir la cuenta atrás y esperar en el modo de consola UEFI. Si no se interrumpe el temporizador, el proceso de arranque continúa y se pierden todos los cambios de MOK.
1. Seleccione la acción adecuada en el menú de la utilidad MOK.

    :::image type="content" source="media/trusted-launch/mok-mangement.png" alt-text="Captura de pantalla que muestra las opciones disponibles en el menú de administración de MOK en la consola serie.":::


## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre las máquinas virtuales con [inicio seguro](trusted-launch.md) y de [generación 2](generation-2.md).
