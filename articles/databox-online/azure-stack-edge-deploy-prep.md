---
title: Tutorial para preparar la implementación de Azure Stack Edge Pro con FPGA mediante Azure Portal en el centro de datos
description: El primer tutorial para implementar Azure Stack Edge Pro con FPGA trata sobre la preparación de Azure Portal.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 03/16/2021
ms.author: alkohli
ms.openlocfilehash: 1cab6f6f9db0650cee51b3863d521089b500bee9
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110461316"
---
# <a name="tutorial-prepare-to-deploy-azure-stack-edge-pro-fpga"></a>Tutorial: Preparación de la implementación de Azure Stack Edge Pro con FPGA  

Este es el primero de una serie de tutoriales de implementación necesarios para implementar Azure Stack Edge Pro con FPGA en su totalidad. En este tutorial se describe cómo preparar Azure Portal para implementar un recurso de Azure Stack Edge. 

Para completar el proceso de instalación y configuración se necesitan privilegios de administrador. La preparación del portal dura menos de 10 minutos.  

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Crear un nuevo recurso
> * Obtención de la clave de activación

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="get-started"></a>Introducción

Para implementar Azure Stack Edge Pro con FPGA, consulte los siguientes tutoriales en el orden indicado.

| **#** | **En este paso** | **Use estos documentos** |
| --- | --- | --- | 
| 1. |**[Preparación de Azure Portal para Azure Stack Edge Pro con FPGA](azure-stack-edge-deploy-prep.md)** |Cree y configure el recurso de Azure Stack Edge antes de instalar un dispositivo físico de Azure Stack Box Edge. |
| 2. |**[Instalación de Azure Stack Edge Pro con FPGA](azure-stack-edge-deploy-install.md)**|Desempaquete, monte en el bastidor y realice el cableado del dispositivo físico de Azure Stack Edge Pro con FPGA.  |
| 3. |**[Conexión, configuración y activación de Azure Stack Edge Pro con FPGA](azure-stack-edge-deploy-connect-setup-activate.md)** |Conéctese a la interfaz de usuario web local, complete la configuración del dispositivo y active el dispositivo. El dispositivo está listo para configurar recursos compartidos de SMB o NFS.  |
| 4. |**[Transferencia de datos con Azure Stack Edge Pro con FPGA](azure-stack-edge-deploy-add-shares.md)** |Agregue recursos compartidos y conéctese a ellos mediante SMB o NFS. |
| 5. |**[Transformación de datos con Azure Stack Edge Pro con FPGA](azure-stack-edge-deploy-configure-compute.md)** |Configure los módulos de proceso en el dispositivo para que transformen los datos a medida que estos se mueven a Azure. |

Ya puede empezar a configurar Azure Portal.

## <a name="prerequisites"></a>Prerrequisitos

A continuación, encontrará los requisitos previos para configurar su recurso de Azure Stack Edge, el dispositivo de Azure Stack Edge Pro con FPGA y la red de centros de datos.

### <a name="for-the-azure-stack-edge-resource"></a>Para el recurso de Azure Stack Edge

Antes de comenzar, asegúrese de que:

* Su suscripción de Microsoft Azure debe estar habilitada para un recurso de Azure Stack Edge. Asegúrese de que ha usado una suscripción admitida como [Contrato Enterprise (EA) de Microsoft](https://azure.microsoft.com/overview/sales-number/), [Proveedor de soluciones en la nube (CSP)](/partner-center/azure-plan-lp) o [Patrocinio de Microsoft Azure](https://azure.microsoft.com/offers/ms-azr-0036p/). No se admiten las suscripciones de pago por uso.

* Tiene acceso de propietario o colaborador a nivel de grupo de recursos para los recursos de Azure Stack Edge o Data Box Gateway, IoT Hub y Azure Storage.

  * Para conceder acceso de colaborador debe ser **Propietario** en el nivel de suscripción. Para conceder acceso de colaborador a otra persona, en Azure Portal, vaya a **Todos los servicios** > **Suscripciones** > **Control de acceso (IAM)**  >  **+ Agregar** > **Agregar asignación de roles**. Para más información, consulte el [Tutorial: Concesión de acceso de usuario a los recursos de Azure mediante Azure Portal](../role-based-access-control/quickstart-assign-role-user-portal.md).

  * Para crear cualquier recurso de Azure Stack Edge o Data Box Gateway, debe tener permisos de colaborador (o superiores) con ámbito en el nivel del grupo de recursos. También tendrá que asegurarse de que el proveedor de recursos `Microsoft.DataBoxEdge` está registrado. Para obtener información sobre cómo registrar un proveedor de recursos, consulte [Registro de proveedores de recursos](azure-stack-edge-manage-access-power-connectivity-mode.md#register-resource-providers).
  * Para crear cualquier recurso de IoT Hub, asegúrese de que el proveedor Microsoft.Devices está registrado. Para obtener información acerca de cómo registrarse, vaya a [Registro de proveedores de recursos](azure-stack-edge-manage-access-power-connectivity-mode.md#register-resource-providers).
  * Para crear un recurso de una cuenta de Storage también se necesita acceso de colaborador, o superior, con ámbito en el nivel de grupo de recursos. De forma predeterminada Azure Storage es un proveedor de recursos registrado.
* Debe tener acceso de administrador o usuario a Graph API de Azure Active Directory. Para más información, vea [Graph API de Azure Active Directory](/previous-versions/azure/ad/graph/howto/azure-ad-graph-api-permission-scopes#default-access-for-administrators-users-and-guest-users-).
* Tiene una cuenta de almacenamiento de Microsoft Azure con credenciales de acceso.
* No está bloqueado por ninguna directiva de Azure configurada por el administrador del sistema. Para más información sobre las directivas, consulte [Inicio rápido: Creación de una asignación de directiva para identificar recursos no compatibles](../governance/policy/assign-policy-portal.md).

### <a name="for-the-azure-stack-edge-pro-fpga-device"></a>Para el dispositivo de Azure Stack Edge Pro con FPGA

Antes de implementar un dispositivo físico, asegúrese de que:

* Ha revisado la información de seguridad que se incluyó en el paquete de envío.
* Tiene una ranura 1U disponible en un bastidor estándar de 19" en el centro de datos para montar el dispositivo.
* Tiene acceso a una superficie de trabajo plana, estable y nivelada en la que el dispositivo pueda apoyarse con seguridad.
* La ubicación en la que desea efectuar la instalación del dispositivo dispone de corriente alterna estándar de una fuente independiente o una unidad de distribución de energía (PDU) en bastidor con un sistema de alimentación ininterrumpida (UPS).
* Tiene acceso a un dispositivo físico.

### <a name="for-the-datacenter-network"></a>Para la red del centro de datos

Antes de comenzar, asegúrese de que:

* La red del centro de datos está configurada según los requisitos de red para el dispositivo de Azure Stack Edge Pro con FPGA. Para más información, consulte [Requisitos del sistema para Azure Stack Edge Pro con FPGA](azure-stack-edge-system-requirements.md).

* Para que Azure Stack Edge Pro con FPGA funcione normalmente, necesita lo siguiente:

  * Un ancho de banda mínimo de 10 Mbps para garantizar que el dispositivo se mantiene actualizado.
  * Un ancho de banda mínimo de 20 Mbps dedicado a carga y descarga para transferir archivos.

## <a name="create-new-resource-for-existing-device"></a>Creación de un recurso nuevo para un dispositivo existente

Si ya es un cliente de Azure Stack Edge Pro con FPGA, use el procedimiento siguiente para crear un recurso nuevo si necesita reemplazar o restablecer el dispositivo existente.

Si es un cliente nuevo, le recomendamos que explore el uso de los dispositivos de Azure Stack Edge Pro-GPU para sus cargas de trabajo. Para más información, vaya a [¿Qué es Azure Stack Edge Pro con GPU?](azure-stack-edge-gpu-overview.md)? Para más información sobre cómo ordenar un dispositivo de Azure Stack Edge Pro con GPU, vaya al artículo sobre [creación de un nuevo recurso para Azure Stack Edge Pro - GPU](azure-stack-edge-gpu-deploy-prep.md?tabs=azure-portal#create-a-new-resource).

Siga estos pasos en Azure Portal para crear un recurso nuevo de Azure Stack Edge para un dispositivo existente.

1. Use sus credenciales de Microsoft Azure para iniciar sesión en:

    - Azure Portal en esta dirección URL: [https://portal.azure.com](https://portal.azure.com).
    - O bien, el portal de Azure Government en esta dirección URL: [https://portal.azure.us](https://portal.azure.us). Para más información, vaya a [Connect to Azure Government using the portal](../azure-government/documentation-government-get-started-connect-with-portal.md) (Conexión a Azure Government mediante el portal).

1. Seleccione **+ Crear un recurso**. Busque y seleccione **Azure Stack Edge**. Seleccione **Crear**.

1. Seleccione la suscripción del dispositivo de Azure Stack Edge Pro con FPGA y el país al que se enviará el dispositivo en **Enviar a**.

   ![Selección de la suscripción y el país de envío para el dispositivo](media/azure-stack-edge-deploy-prep/create-fpga-existing-resource-01.png)


1. En la lista de tipos de dispositivos que se muestra, seleccione **Azure Stack Edge Pro - FPGA**. Luego, elija **Seleccionar**. 

   El tipo de dispositivo **Azure Stack Edge Pro - FPGA** solo se muestra si tiene un dispositivo existente. Si necesita solicitar un nuevo dispositivo, vaya a [Creación de un nuevo recurso para Azure Stack Edge Pro - GPU](azure-stack-edge-gpu-deploy-prep.md?tabs=azure-portal#create-a-new-resource).

   ![Búsqueda del servicio Azure Stack Edge](media/azure-stack-edge-deploy-prep/create-fpga-existing-resource-02.png)

1. En la pestaña **Aspectos básicos**:

   1. Escriba o seleccione los siguientes **detalles del proyecto**.
    
       |Setting  |Valor  |
       |---------|---------|
       |Subscription    |Este valor se rellena automáticamente según la selección anterior. La suscripción está vinculada a la cuenta de facturación. |
       |Resource group  |Cree un nuevo grupo o seleccione uno existente.<br>Más información sobre los [grupos de recursos de Azure](../azure-resource-manager/management/overview.md).     |

   1. Escriba o seleccione los siguientes **detalles de la instancia**.

       |Configuración  |Valor  |
       |---------|---------|
       |Nombre   | Nombre descriptivo que identifique el recurso.<br>El nombre tiene entre dos y 50 caracteres, que incluyen letras, números y guiones.<br> El nombre comienza y termina con una letra o un número.        |
       |Region     |Para una lista de todas las regiones en las que está disponible el recurso de Azure Stack Edge, consulte [Productos de Azure disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=databox&regions=all). Si usa Azure Government, todas las regiones de gobierno están disponibles como se muestra en las [regiones de Azure](https://azure.microsoft.com/global-infrastructure/regions/).<br> Elija la ubicación más cercana a la región geográfica donde quiera implementar el dispositivo.|

   1. Seleccione **Revisar + crear**.

    ![Detalles del proyecto e instancia](media/azure-stack-edge-deploy-prep/create-fpga-existing-resource-03.png)

1. En la pestaña **Revisar y crear**, revise los **Términos de uso**, los **Detalles de precios** y los detalles del recurso. Seleccione **Crear**.

    ![Revisión de los detalles del recurso y los términos de privacidad de Azure Stack Edge](media/azure-stack-edge-deploy-prep/create-fpga-existing-resource-04.png)

1. Se tarda unos minutos en crear el recurso. Cuando el recurso se haya creado e implementado correctamente, recibirá una notificación. Haga clic en **Go to resource** (Ir al recurso).

   ![Ir al recurso de Azure Stack Edge](media/azure-stack-edge-deploy-prep/data-box-edge-resource-01.png)

Una vez realizado el pedido, Microsoft lo revisa y se pone en contacto con usted (por correo electrónico) para indicarle los detalles del envío.

![Notificación de revisión del pedido de Azure Stack Edge Pro con FPGA](media/azure-stack-edge-deploy-prep/data-box-edge-resource-02.png)


## <a name="get-the-activation-key"></a>Obtención de la clave de activación

Cuando el recurso de Azure Stack Edge esté en funcionamiento, tendrá que obtener la clave de activación. Esta clave se usa para activar y conectar el dispositivo de Azure Stack Edge Pro con FPGA con el recurso. Puede obtener esta clave ahora mientras está en Azure Portal.

1. Vaya al recurso que ha creado y, a continuación, seleccione **Información general**. Se enviará una notificación al efecto para informarle del procesamiento del pedido.

    ![Seleccionar Información general](media/azure-stack-edge-deploy-prep/data-box-edge-select-device-setup.png)

2. Una vez que se procesa el pedido y el dispositivo está de camino, la página **Información general** se actualiza. Acepte el **nombre de la instancia de Azure Key Vault** predeterminada o especifique uno nuevo. Seleccione **Generar código de activación**. Seleccione el icono de copia para copiar la clave y guárdela para su uso posterior.

    ![Obtención de la clave de activación](media/azure-stack-edge-deploy-prep/get-activation-key.png)

> [!IMPORTANT]
>
> * La clave de activación expira tres días después de generarla.
> * Si la clave ha expirado, genere una nueva. La clave antigua no es válida.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha obtenido información acerca de varios temas relacionados con Azure Stack Edge Pro con FPGA, tales como:

> [!div class="checklist"]
>
> * Crear un nuevo recurso
> * Obtención de la clave de activación

En el siguiente tutorial aprenderá a instalar Azure Stack Edge Pro con FPGA.

> [!div class="nextstepaction"]
> [Instalación de Azure Stack Edge Pro con FPGA](./azure-stack-edge-deploy-install.md)