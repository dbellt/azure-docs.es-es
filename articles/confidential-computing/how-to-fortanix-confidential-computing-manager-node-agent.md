---
title: Ejecución de una aplicación con Fortanix Confidential Computing Manager
description: Aprenda a usar Fortanix Confidential Computing Manager para convertir sus imágenes en contenedores.
services: virtual-machines
author: JBCook
ms.service: virtual-machines
ms.subservice: confidential-computing
ms.workload: infrastructure
ms.topic: how-to
ms.date: 03/24/2021
ms.author: JenCook
ms.openlocfilehash: 6ad2bbb206d1765d4070e744eeab9708ea0db366
ms.sourcegitcommit: ba8f0365b192f6f708eb8ce7aadb134ef8eda326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2021
ms.locfileid: "109633328"
---
# <a name="run-an-application-by-using-fortanix-confidential-computing-manager"></a>Ejecución de una aplicación mediante Fortanix Confidential Computing Manager

Aprenda cómo ejecutar la aplicación en la computación confidencial de Azure mediante [Fortanix Confidential Computing Manager](https://azuremarketplace.microsoft.com/marketplace/apps/fortanix.enclave_manager?tab=Overview) y [Node Agent](https://azuremarketplace.microsoft.com/marketplace/apps/fortanix.rte_node_agent) de [Fortanix](https://www.fortanix.com/).


Fortanix es un proveedor de software externo que brinda productos y servicios que funcionan con la infraestructura de Azure. Hay otros proveedores externos que ofrecen servicios de computación confidencial similares para Azure.

> [!Note]
> Microsoft no proporciona algunos de los productos a los que se hace referencia en este documento. Microsoft proporciona esta información solo por comodidad. Las referencias a estos productos que no son de Microsoft no implican la aprobación por parte de Microsoft.

En este tutorial se muestra cómo convertir la imagen de una aplicación en una imagen protegida por computación confidencial. El entorno usa software de [Fortanix](https://www.fortanix.com/) basado en tecnología de máquinas virtuales habilitadas para Intel SGX de la serie DCsv2 de Azure. La solución organiza directivas de seguridad críticas, como la verificación de identidades y el control del acceso a datos.

Para obtener soporte técnico de Fortanix, únase a la [comunidad Fortanix en Slack](https://fortanix.com/community/). Use el canal `#enclavemanager`.

## <a name="prerequisites"></a>Requisitos previos

- Si no tiene una cuenta de Fortanix Confidential Computing Manager, [regístrese](https://ccm.fortanix.com/auth/sign-up) antes de comenzar.
- Necesita un registro de [Docker](https://docs.docker.com/) privado al que se enviarán las imágenes de la aplicación convertidas.
- Si no tiene una suscripción a Azure, [cree una cuenta](https://azure.microsoft.com/pricing/purchase-options/pay-as-you-go/) antes de comenzar.

> [!NOTE]
> Las cuentas de evaluación gratuita no tienen acceso a las máquinas virtuales que se usan en este tutorial. Para completar el tutorial, se necesita una suscripción de pago por uso.

## <a name="add-an-application-to-fortanix-confidential-computing-manager"></a>Incorporación de una aplicación a Fortanix Confidential Computing Manager

1. Inicie sesión en [Fortanix Confidential Computing Manager (Fortanix CCM)](https://ccm.fortanix.com).
1. Vaya a la página **Accounts** (Cuentas) y seleccione **ADD ACCOUNT** (Agregar cuenta) para crear una cuenta nueva:

   :::image type="content" source="media/how-to-fortanix-confidential-computing-manager-node-agent/create-account-new.png" alt-text="Captura de pantalla que muestra cómo crear una cuenta.":::

1. Una vez creada la cuenta, haga clic en **SELECT ACCOUNT** (Seleccionar cuenta) para seleccionar la cuenta recién creada. Ahora puede empezar a inscribir los nodos de ejecución y a crear aplicaciones.
1. Vaya a la pestaña **Applications** (Aplicaciones) y seleccione **+ APPLICATION** (Aplicación) para agregar una aplicación. En este ejemplo, agregaremos una aplicación Enclave OS que ejecuta un servidor de Python Flask.

1. Seleccione el botón **ADD** (Agregar) para la **aplicación Enclave OS**:

   :::image type="content" source="media/how-to-fortanix-confidential-computing-manager-node-agent/add-enclave-application.png" alt-text="Captura de pantalla que muestra cómo agregar una aplicación.":::

   > [!NOTE]
   > En este tutorial se explica la adición de aplicaciones de Enclave OS únicamente. Para obtener información sobre cómo agregar aplicaciones de EDP Rust, consulte [Bringing EDP Rust Apps to Confidential Computing Manager](https://support.fortanix.com/hc/en-us/articles/360044746932-Bringing-EDP-Rust-Apps-to-Confidential-Computing-Manager) (Traslado de aplicaciones de EDP Rust a Confidential Computing Manager).

1. En este tutorial, usaremos el registro de Docker de Fortanix para la aplicación de ejemplo. Escriba los valores especificados para la siguiente configuración. Use su registro de Docker privado para almacenar la imagen de salida.

    - **Nombre de la aplicación**: Python Application Server (servidor de aplicación de Python)
    - **Descripción**: Python Flask Server (servidor de Flask en Python)
    - **Nombre de la imagen de entrada**: fortanix/python-flask
    - **Nombre de la imagen de salida**: fortanix-private/python-flask-sgx (reemplácelo por su propio registro).
    - **ISVPRODID**: 1
    - **ISVSVM**: 1
    - **Tamaño de memoria**: 1 GB
    - **Número de subprocesos**: 128

      *Opcional*: ejecute la aplicación no convertida.
    - **Docker Hub**: [https://hub.docker.com/u/fortanix](https://hub.docker.com/u/fortanix)
    - **Aplicación**: fortanix/python-flask

      Ejecute el siguiente comando:

      ```bash
         sudo docker run fortanix/python-flask
      ```
      > [!NOTE]
      > No se recomienda usar el registro de Docker privado para almacenar la imagen de salida.

1. Agregue un certificado. Escriba los valores siguientes y, después, seleccione **NEXT** (Siguiente):
    - **Dominio**: myapp.domain.com
    - **Tipo**: Certificado emitido por Confidential Computing Manager
    - **Ruta de acceso de la clave**: /run/key.pem
    - **Tipo de clave**: RSA
    - **Ruta de acceso del certificado**: /run/cert.pem
    - **Tamaño de la clave RSA**: 2048 bits

## <a name="create-an-image"></a>Crear una imagen

Una imagen de Fortanix CCM es una versión de software o de una aplicación. Cada imagen está asociada con un hash de Enclave (MRENCLAVE).

1. En la página **Add Image** (Agregar imagen), escriba las credenciales del registro para **Output image name** (Nombre de la imagen de salida). Estas credenciales se usan para acceder al registro de Docker privado donde se insertará la imagen. Dado que la imagen de entrada se almacena en un registro público, no es necesario que proporcione credenciales para la imagen de entrada.
1. Escriba la etiqueta de imagen y seleccione **CREATE** (Crear).

   :::image type="content" source="media/how-to-fortanix-confidential-computing-manager-node-agent/create-image.png" alt-text="Captura de pantalla que muestra cómo crear una imagen.":::


## <a name="domain-and-image-allowlist"></a>Lista de permitidos de dominio e imagen

Una aplicación cuyo dominio se agrega a la lista de permitidos obtendrá un certificado TLS de Fortanix Confidential Computing Manager. Cuando se inicie una aplicación Enclave OS, contactará con la instancia de Fortanix Confidential Computing Manager para recibir ese certificado TLS.

En la pestaña **Tasks** (Tareas) a la izquierda de la pantalla, apruebe las solicitudes pendientes para permitir el dominio y la imagen.

## <a name="enroll-the-compute-node-agent-in-azure"></a>Inscripción del agente de nodo de ejecución en Azure

### <a name="create-and-copy-a-join-token"></a>Creación y copia de un token de unión

Ahora creará un token en Fortanix Confidential Computing Manager. Este permite que un nodo de ejecución de Azure se autentique a sí mismo. La máquina virtual de Azure necesitará este token.

1. En la pestaña **Compute Nodes** (Nodos de ejecución), seleccione **ENROLL NODE** (Inscribir nodo).
1. Seleccione el botón **COPY** (Copiar) para copiar el token de unión. El nodo de ejecución usa este token de unión para autenticarse.

### <a name="enroll-nodes-into-fortanix-node-agent"></a>Inscripción de nodos en Node Agent de Fortanix

Al crear una instancia de Node Agent de Fortanix, se implementará una máquina virtual, una interfaz de red, una red virtual, un grupo de seguridad de red y una dirección IP pública en el grupo de recursos de Azure. La suscripción a Azure se facturará por hora para la máquina virtual. Antes de crear una instancia de Node Agent de Fortanix, revise la [página de precios de máquinas virtuales](https://azure.microsoft.com/pricing/details/virtual-machines/linux/) de Azure de la serie DCsv2. Elimine los recursos de Azure que no use.

1. Vaya a [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/) e inicie sesión con sus credenciales de Azure.
1. En la barra de búsqueda, escriba **Fortanix Confidential Computing Node Agent**. En los resultados de la búsqueda, seleccione **Fortanix Confidential Computing Node Agent** para ir a la [página principal de la aplicación](https://azuremarketplace.microsoft.com/marketplace/apps/fortanix.rte_node_agent?tab=OverviewFortanix):

    :::image type="content" source="media/how-to-fortanix-confidential-computing-manager-node-agent/search-fortanix-marketplace.png" alt-text="Captura de pantalla que muestra cómo llegar a la página principal de la aplicación.":::
1. Seleccione **Obtenerla ahora**, rellene la información si es necesario y seleccione **Continuar**. Se le redirigirá a Azure Portal.
1. Seleccione **Crear** para ir en la página de implementación de Fortanix Confidential Computing Node Agent.
1. En esta página, escribirá la información para implementar una máquina virtual. La VM es una máquina virtual habilitada para Intel SGX de la serie DCsv2 de Azure con el software Node Agent de Fortanix instalado. Node Agent permitirá que la imagen convertida se ejecute con mayor seguridad en los nodos de Intel SGX en Azure. Seleccione la suscripción y el grupo de recursos en los que quiere implementar la máquina virtual y los recursos asociados.

   > [!NOTE]
   > Las restricciones se aplican al implementar máquinas virtuales de la serie DCsv2 en Azure. Es posible que tenga que solicitar la cuota para núcleos adicionales. Para más información, consulte [Soluciones de computación confidencial en máquinas virtuales de Azure](./virtual-machine-solutions.md).

1. Seleccione una región disponible.
1. En el cuadro **Nombre de nodo**, escriba un nombre para la máquina virtual.
1. Escriba un nombre de usuario y contraseña (o clave SSH) para la autenticación en la máquina virtual.
1. Deje el **Tamaño de disco del SO** predeterminado de **200**. Seleccione un **Tamaño de VM**. (**Standard_DC4s_v2** servirá para este tutorial).
1. En el cuadro **Token de unión**, pegue el token que creó anteriormente en este tutorial:

   :::image type="content" source="media/how-to-fortanix-confidential-computing-manager-node-agent/deploy-fortanix-node-agent-protocol.png" alt-text="Captura de pantalla que muestra cómo implementar un recurso.":::

1. Seleccione **Revisar + crear**. Asegúrese de que supere la validación y, después, seleccione **Crear**. Cuando se hayan implementado todos los recursos, el nodo de proceso estará inscrito en Fortanix Confidential Computing Manager.

## <a name="run-the-application-image-on-the-compute-node"></a>Ejecución de la imagen de aplicación en el nodo de ejecución

Para ejecutar la aplicación, ejecute el siguiente comando. Asegúrese de cambiar la dirección IP del nodo, el puerto y el nombre de la imagen convertida a los valores de su aplicación.

En este tutorial, este es el comando que se debe ejecutar:

```bash
    sudo docker run \
        --device /dev/isgx:/dev/isgx \
        --device /dev/gsgx:/dev/gsgx \
        -v /var/run/aesmd/aesm.socket:/var/run/aesmd/aesm.socket \
        -e NODE_AGENT_BASE_URL=http://52.152.206.164:9092/v1/ fortanix-private/python-flask-sgx
```

En este comando:

- `52.152.206.164` es la dirección IP del host de Node Agent.
- `9092` es el puerto predeterminado donde escucha Node Agent.
- `fortanix-private/python-flask-sgx` es la aplicación convertida. Puede encontrarla en el portal web de Fortanix Confidential Computing Manager. Se encuentra en la pestaña **Images** (Imágenes), en la columna **Image Name** (Nombre de la imagen) de la tabla **Images** (Imágenes).

## <a name="verify-and-monitor-the-running-application"></a>Comprobación y supervisión de la aplicación en ejecución

1. Regresar a [Fortanix Confidential Computing Manager](https://ccm.fortanix.com/console).
1. Asegúrese de estar trabajando dentro de la **Cuenta** en la que inscribió el nodo.
1. En la pestaña **Applications** (Aplicaciones), compruebe que hay una aplicación en ejecución con un nodo de ejecución asociado.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ya no necesita el grupo de recursos, la máquina virtual y los recursos asociados, puede eliminarlos. Al eliminar el grupo de recursos, se anulará la inscripción de los nodos asociados con la imagen convertida.

Seleccione el grupo de recursos de la máquina virtual y haga clic en **Delete** (Eliminar). Confirme el nombre del grupo de recursos para terminar de eliminar los recursos.

Para eliminar la cuenta de Fortanix Confidential Computing Manager que ha creado, vaya a la página [Accounts](https://ccm.fortanix.com/accounts) (Cuentas) de Fortanix Confidential Computing Manager. Mantenga el puntero sobre la cuenta que quiere eliminar. Seleccione los puntos negros verticales en la esquina superior derecha y seleccione **DELETE ACCOUNT** (Eliminar cuenta).

:::image type="content" source="media/how-to-fortanix-confidential-computing-manager-node-agent/delete-account.png" alt-text="Captura de pantalla que muestra cómo eliminar una cuenta.":::

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha usado las herramientas de Fortanix para convertir la imagen de una aplicación para que se ejecute en una máquina virtual de computación confidencial. Para obtener más información sobre las máquinas virtuales de computación confidencial en Azure, consulte [Soluciones en máquinas virtuales](virtual-machine-solutions.md).

Para obtener más información sobre las ofertas de computación confidencial de Azure, consulte [Introducción a la computación confidencial de Azure](overview.md).

También puede obtener más información sobre cómo completar tareas similares con otras ofertas de terceros en Azure, como [Anjuna](https://azuremarketplace.microsoft.com/marketplace/apps/anjuna-5229812.aee-az-v1) y [Scone](https://sconedocs.github.io).
