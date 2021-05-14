---
title: Procedimiento para ejecutar una aplicación con Fortanix Confidential Computing Manager
description: Aprenda a usar Fortanix Confidential Computing Manager para convertir sus imágenes en contenedores
services: virtual-machines
author: JBCook
ms.service: virtual-machines
ms.subservice: confidential-computing
ms.workload: infrastructure
ms.topic: how-to
ms.date: 03/24/2021
ms.author: JenCook
ms.openlocfilehash: 228f66a4dc10e7a518abdace22836c66bc717288
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108204820"
---
# <a name="how-to-run-an-application-with-fortanix-confidential-computing-manager"></a>Instrucciones: ejecución de una aplicación con Fortanix Confidential Computing Manager

Comience a ejecutar la aplicación en la computación confidencial de Azure mediante [Fortanix Confidential Computing Manager](https://azuremarketplace.microsoft.com/marketplace/apps/fortanix.enclave_manager?tab=Overview) y [Fortanix Node Agent](https://azuremarketplace.microsoft.com/marketplace/apps/fortanix.rte_node_agent) de [Fortanix](https://www.fortanix.com/).


Fortanix es un proveedor de software de terceros con productos y servicios basados en la infraestructura de Azure. Hay otros proveedores de terceros que ofrecen servicios de computación confidencial similares en Azure.

> [!Note]
> Los productos a los que se hace referencia en este documento no están bajo el control de Microsoft. Microsoft le proporciona esta información solo para su comodidad y las referencias a estos productos que no son de Microsoft no implica que tengan la aprobación de Microsoft.

En este tutorial se muestra cómo convertir la imagen de la aplicación en una imagen protegida para computación confidencial. Este entorno usa software de [Fortanix](https://www.fortanix.com/) basado en las máquinas virtuales habilitadas para Intel SGX de la serie DCsv2 de Azure. Esta solución organiza directivas de seguridad críticas, como la comprobación de identidades y el control de acceso a datos.

Para obtener el soporte técnico específico de Fortanix, únase a la [comunidad Fortanix en Slack](https://fortanix.com/community/) y use el canal `#enclavemanager`.

## <a name="prerequisites"></a>Requisitos previos

1. Si no tiene una cuenta de Fortanix Confidential Computing Manager, [regístrese](https://ccm.fortanix.com/auth/sign-up) antes de empezar.
1. Un registro de [Docker](https://docs.docker.com/) privado al que se enviarán las imágenes de aplicaciones convertidas.
1. Si no tiene una suscripción a Azure, [cree una cuenta](https://azure.microsoft.com/pricing/purchase-options/pay-as-you-go/) antes de empezar.

> [!NOTE]
> Las cuentas de evaluación gratuita no tienen acceso a las máquinas virtuales que se usan en este tutorial. Actualice a una suscripción de pago por uso.

## <a name="add-an-application-to-fortanix-confidential-computing-manager"></a>Incorporación de una aplicación a Fortanix Confidential Computing Manager

1. Inicie sesión en [Fortanix Confidential Computing Manager (Fortanix CCM)](https://ccm.fortanix.com).
1. Vaya a la página **Cuentas** y seleccione **AGREGAR CUENTA** para crear una cuenta nueva.

   :::image type="content" source="media/how-to-fortanix-confidential-computing-manager-node-agent/create-account-new.png" alt-text="Captura de pantalla que muestra cómo crear una cuenta.":::

1. Una vez creada la cuenta, pulse **SELECCIONAR CUENTA** para seleccionar la cuenta recién creada. Ahora podemos empezar a inscribir los nodos de ejecución y a crear aplicaciones.
1. Vaya a la pestaña **Aplicaciones** y haga clic en **+ APLICACIÓN** para agregar una aplicación. En este ejemplo, agregaremos una aplicación Enclave OS que ejecuta un servidor de Python Flask.

1. Seleccione el botón **ADD** (AGREGAR) para la aplicación Enclave OS.

   :::image type="content" source="media/how-to-fortanix-confidential-computing-manager-node-agent/add-enclave-application.png" alt-text="Captura de pantalla que muestra cómo agregar una aplicación.":::

   > [!NOTE]
   > En este tutorial solo se explica la adición de aplicaciones Enclave OS. [Más información](https://support.fortanix.com/hc/en-us/articles/360044746932-Bringing-EDP-Rust-Apps-to-Confidential-Computing-Manager) sobre cómo migrar aplicaciones EDP Rust a Fortanix Confidential Computing Manager.

1. En este tutorial, usaremos el registro de Docker de Fortanix para la aplicación de ejemplo. Rellene los datos con la información siguiente. Use su registro de Docker privado para mantener la imagen de salida.

    - **Nombre de la aplicación**: Python Application Server (servidor de aplicación de Python)
    - **Descripción**: Python Flask Server (servidor de Flask en Python)
    - **Nombre de la imagen de entrada**: fortanix/python-flask
    - **Nombre de la imagen de salida**: fortanix-private/python-flask-sgx (reemplácelo por su propio registro)
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
      > Se recomienda no usar el registro de Docker privado para almacenar la imagen de salida.

1. Agregue un certificado. Rellene la información con los datos siguientes y, a continuación, seleccione **NEXT** (SIGUIENTE):
    - **Dominio**: myapp.domain.com
    - **Tipo**: Certificado emitido por Confidential Computing Manager
    - **Ruta de acceso de la clave**: /run/key.pem
    - **Tipo de clave**: RSA
    - **Ruta de acceso del certificado**: /run/cert.pem
    - **Tamaño de la clave RSA**: 2048 bits

## <a name="create-an-image"></a>Crear una imagen

Una imagen de Fortanix CCM es una versión de software o de una aplicación. Cada imagen está asociada con un hash de Enclave (MRENCLAVE).

1. En la página **Add Image** (Agregar imagen), escriba las **REGISTRY CREDENTIALS** (CREDENCIALES DEL REGISTRO) para **Output image name** (Nombre de la imagen de salida). Estas credenciales se usan para acceder al registro privado de Docker al que se enviará la imagen. Puesto que la imagen de entrada se almacena en un registro público, no es necesario proporcionar credenciales para esta.
1. Proporcione la etiqueta de imagen y seleccione **CREAR**.

   :::image type="content" source="media/how-to-fortanix-confidential-computing-manager-node-agent/create-image.png" alt-text="Captura de pantalla que muestra cómo crear una imagen.":::


## <a name="domain-and-image-allowlist"></a>Lista de permitidos de dominio e imagen

Una aplicación cuyo dominio se agrega a la lista de permitidos obtendrá un certificado TLS de Fortanix Confidential Computing Manager. Cuando se inicie una aplicación Enclave OS, contactará con la instancia de Fortanix Confidential Computing Manager para recibir ese certificado TLS.

Cambie a la pestaña **Tasks** (Tareas) de la izquierda y apruebe las solicitudes pendientes para permitir el dominio y la imagen.

## <a name="enroll-compute-node-agent-in-azure"></a>Inscripción del agente de nodo de ejecución en Azure

### <a name="generate-and-copy-join-token"></a>Generación y copia del token de unión

En Fortanix Confidential Computing Manager, creará un token. Este permite que un nodo de ejecución de Azure se autentique a sí mismo. Deberá proporcionar este token a la máquina virtual de Azure.

1. Vaya a la pestaña **Nodos de proceso** y haga clic en el botón **+ ENROLL NODE** (INSCRIBIR NODO).
1. Haga clic en el botón **COPIAR** para copiar el token de unión. El nodo de proceso usa este token de unión para autenticarse.

### <a name="enroll-nodes-into-fortanix-node-agent-in-azure-marketplace"></a>Inscripción de nodos en Fortanix Node Agent de Azure Marketplace

Al crear una instancia de Fortanix Node Agent se implementará una máquina virtual, una interfaz de red, una red virtual, un grupo de seguridad de red y una dirección IP pública en el grupo de recursos de Azure. La suscripción a Azure se facturará por hora para la máquina virtual. Antes de crear una instancia de Fortanix Node Agent, revise la [página de precios de máquinas virtuales](https://azure.microsoft.com/pricing/details/virtual-machines/linux/) de Azure de la serie DCsv2. Elimine los recursos de Azure cuando no estén en uso.

1. Vaya a [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/) e inicie sesión con sus credenciales de Azure.
1. En la barra de búsqueda, escriba **Fortanix Confidential Computing Node Agent**. Seleccione la aplicación que aparece en el cuadro de búsqueda denominada **Fortanix Confidential Computing Node Agent** para ir a la página principal de la oferta. Opcionalmente, haga clic en la dirección URL https://azuremarketplace.microsoft.com/marketplace/apps/fortanix.rte_node_agent?tab=OverviewFortanix para acceder a la instancia de Node Agent.

   ![buscar en Marketplace](media/how-to-fortanix-confidential-computing-manager-node-agent/search-fortanix-marketplace.png)
1. Seleccione **Obtenerla ahora**, rellene la información si es necesario y seleccione **Continuar**. Se le redirigirá a Azure Portal.
1. Seleccione **Crear** para entrar en la página de implementación de Fortanix Confidential Computing Node Agent.
1. En esta página, especificará información para implementar una máquina virtual. En concreto, esta máquina virtual es una máquina virtual habilitada para Intel SGX de la serie DCsv2 de Azure con el software de Fortanix Node Agent instalado. Node Agent permitirá que la imagen convertida se ejecute de forma segura en los nodos de Intel SGX en Azure.  Seleccione la **suscripción** y el **grupo de recursos** en los que quiere implementar la máquina virtual y los recursos asociados.

   > [!NOTE]
   > Existen restricciones al implementar máquinas virtuales de la serie DCsv2 en Azure. Quizá deba solicitar la cuota para núcleos adicionales. Para más información, consulte [Soluciones de computación confidencial en máquinas virtuales de Azure](./virtual-machine-solutions.md).

1. Seleccione una región disponible.
1. Introduzca un nombre para la máquina virtual en **Nombre de nodo**.
1. Introduzca un nombre de usuario y contraseña (o clave SSH) para la autenticación en la máquina virtual.
1. Deje el tamaño del disco del SO predeterminado en 200 y seleccione un tamaño de máquina virtual (Standard_DC4s_v2 será suficiente para este tutorial).
1. Pegue el token generado anteriormente en **Token de unión**.

   :::image type="content" source="media/how-to-fortanix-confidential-computing-manager-node-agent/deploy-fortanix-node-agent-protocol.png" alt-text="Captura de pantalla que muestra cómo implementar un recurso.":::

1. Seleccione **Revisar + crear**. Asegúrese de superar la validación y, después, seleccione **Crear**. Cuando se hayan implementado todos los recursos, el nodo de proceso ya estará inscrito en Fortanix Confidential Computing Manager.

## <a name="run-the-application-image-on-the-compute-node"></a>Ejecución de la imagen de aplicación en el nodo de ejecución

Ejecute la aplicación mediante la ejecución del comando siguiente. Asegúrese de cambiar la dirección IP del nodo, el puerto y el nombre de la imagen convertida como entradas para su aplicación específica.

En este tutorial, el comando que se ejecutará es:

```bash
    sudo docker run \
        --device /dev/isgx:/dev/isgx \
        --device /dev/gsgx:/dev/gsgx \
        -v /var/run/aesmd/aesm.socket:/var/run/aesmd/aesm.socket \
        -e NODE_AGENT_BASE_URL=http://52.152.206.164:9092/v1/ fortanix-private/python-flask-sgx
```

Donde:

- *52.152.206.164* es la dirección IP del host de Node Agent.
- *9092* es el puerto predeterminado de escucha de Node Agent.
- *fortanix-private/python-flask-sgx* es la aplicación convertida que se puede encontrar en la pestaña Images (Imágenes), en la columna **Image Name** (Nombre de imagen) de la tabla **Images** (Imágenes) del portal web de Fortanix Confidential Computing Manager.

## <a name="verify-and-monitor-the-running-application"></a>Comprobación y supervisión de la aplicación en ejecución

1. Regresar a [Fortanix Confidential Computing Manager](https://ccm.fortanix.com/console).
1. Asegúrese de que está trabajando dentro de la **Cuenta** en la que inscribió el nodo.
1. Seleccione la pestaña **Aplicaciones**.
1. Compruebe que hay una aplicación en ejecución con un nodo de ejecución asociado.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no sean necesarios, puede eliminar el grupo de recursos, la máquina virtual y todos los recursos asociados. Al eliminar el grupo de recursos, se anulará la inscripción de los nodos asociados con la imagen convertida.

Seleccione el grupo de recursos de la máquina virtual y haga clic en **Eliminar**. Confirme el nombre del grupo de recursos para terminar de eliminar los recursos.

Para eliminar la cuenta de Fortanix Confidential Computing Manager que ha creado, vaya a la página [Cuentas](https://ccm.fortanix.com/accounts) de Fortanix Confidential Computing Manager. Mantenga el puntero sobre la cuenta que quiere eliminar. Seleccione los puntos negros verticales en la esquina superior derecha y seleccione **ELIMINAR CUENTA**.

:::image type="content" source="media/how-to-fortanix-confidential-computing-manager-node-agent/delete-account.png" alt-text="Captura de pantalla que muestra cómo eliminar la cuenta.":::

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha usado las herramientas de Fortanix para convertir la imagen de la aplicación con el fin de que se ejecute en una máquina virtual de computación confidencial. Para más información sobre las máquinas virtuales de computación confidencial en Azure, consulte [Soluciones en Virtual Machines](virtual-machine-solutions.md).

Para más información sobre las ofertas de computación confidencial de Azure, consulte [Introducción a la computación confidencial de Azure](overview.md).

Más información sobre cómo completar tareas similares con otras ofertas de terceros en Azure, como [Anjuna](https://azuremarketplace.microsoft.com/marketplace/apps/anjuna-5229812.aee-az-v1) y [Scone](https://sconedocs.github.io).
