---
title: Conexión de máquinas que no son de Azure a Azure Security Center
description: Obtenga información sobre cómo conectar máquinas que no son de Azure a Security Center.
author: memildin
ms.author: memildin
ms.date: 11/16/2020
ms.topic: quickstart
ms.service: security-center
manager: rkarlin
zone_pivot_groups: non-azure-machines
ms.openlocfilehash: 8ca3f7c9104fa655536762ac38f50c78cb96ad3a
ms.sourcegitcommit: f3b930eeacdaebe5a5f25471bc10014a36e52e5e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/16/2021
ms.locfileid: "112237933"
---
# <a name="connect-your-non-azure-machines-to-security-center"></a>Conexión de máquinas que no son de Azure a Security Center

Defender puede supervisar la situación de seguridad de los equipos que no son de Azure, pero para ello antes hay que conectarlos a Azure.

Puede conectar equipos que no son de Azure de alguna de las maneras siguientes:

- Mediante servidores habilitados para Azure Arc (**recomendable**)
- Desde las páginas de Security Center en Azure Portal (**Introducción** e **Inventario**)

En esta página se describen cada una de ellas.

::: zone pivot="azure-arc"

## <a name="add-non-azure-machines-with-azure-arc"></a>Incorporación de máquinas que no son de Azure con Azure Arc

El uso de los servidores habilitados de Azure Arc es la manera preferida de agregar máquinas que no son de Azure a Azure Security Center.

Una máquina con servidores habilitados de Azure Arc, se convierte en un recurso de Azure y aparece en Security Center con recomendaciones igual que los otros recursos de Azure.

Además, los servidores habilitados de Azure Arc proporcionan funcionalidades mejoradas, como la opción de habilitar directivas de configuración de invitados en la máquina, implementar el agente de Log Analytics como una extensión, simplificar la implementación con otros servicios de Azure y mucho más. Para obtener información general sobre las ventajas, consulte [Escenarios admitidos](../azure-arc/servers/overview.md#supported-scenarios).

[Obtenga más información sobre los servidores habilitados de Azure Arc](../azure-arc/servers/overview.md).

**Para implementar Azure Arc:**

- En el caso de una máquina, siga las instrucciones de [Inicio rápido: Conexión de máquinas híbridas con servidores habilitados para Azure Arc](../azure-arc/servers/learn/quick-enable-hybrid-vm.md).
- Consulte [Conexión de máquinas híbridas a Azure a gran escala](../azure-arc/servers/onboard-service-principal.md) para conectar varias máquinas a gran escala a servidores habilitados de Azure Arc.

> [!TIP]
> Si va a incorporar máquinas que se ejecutan en AWS, el conector de Security Center para AWS controla la implementación de Azure Arc sin que se perciba. Obtenga más información en [Conexión de las cuentas de AWS a Azure Security Center](quickstart-onboard-aws.md).

::: zone-end

::: zone pivot="azure-portal"

## <a name="add-non-azure-machines-from-the-azure-portal"></a>Adición de máquinas que no son de Azure desde Azure Portal

1. En el menú de Security Center, abra la página **Introducción**.
1. Seleccione la pestaña **Introducción**.

    :::image type="content" source="./media/security-center-onboarding/onboarding-get-started-tab.png" alt-text="Pestaña Introducción de la página de introducción." lightbox="./media/security-center-onboarding/onboarding-get-started-tab.png":::

1. En **Servidores que no son de Azure**, seleccione **Configurar**.

    > [!TIP]
    > También puede abrir el menú para agregar máquinas en el botón **Agregar servidores que no sean de Azure** de la página **Inventario**.
    > 
    > :::image type="content" source="./media/security-center-onboarding/onboard-inventory.png" alt-text="Incorporación de máquinas que no son de Azure desde la página de inventario de recursos.":::

    Aparecerá una lista de las áreas de trabajo de Log Analytics. La lista incluye, si procede, el área de trabajo predeterminada que Security Center crea automáticamente si el aprovisionamiento automático está habilitado. Seleccione esta área de trabajo u otra que desee usar.

    Puede agregar equipos a un área de trabajo existente o crear un área de trabajo.

1. Opcionalmente, para crear un área de trabajo, seleccione **Crear área de trabajo nueva**.

1. En la lista de áreas de trabajo, seleccione **Agregar servidores** para el área de trabajo correspondiente.
    Se mostrará la página **Administración de agentes**.

    Aquí, elija el procedimiento correspondiente que se indica a continuación en función del tipo de máquinas en las que se va a incorporar:

    - [Incorporación de máquinas virtuales de Azure Stack Hub](#onboard-your-azure-stack-hub-vms)
    - [Incorporación de máquinas Linux](#onboard-your-linux-machines)
    - [Incorporación de máquinas Windows](#onboard-your-windows-machines)

### <a name="onboard-your-azure-stack-hub-vms"></a>Incorporación de máquinas virtuales de Azure Stack Hub

Para agregar máquinas virtuales de Azure Stack Hub, se necesita no solo la información de la página **Administración de agentes**, sino también configurar la extensión de máquina virtual **Azure Monitor, Update and Configuration Management** en las máquinas virtuales que se ejecutan en la instancia de Azure Stack Hub.

1. En la página **Administración de agentes**, copie los valores de **Id. del área de trabajo** y **Clave principal** en el Bloc de notas.
1. Inicie sesión en el portal de **Azure Stack Hub** y abra la página **Máquinas virtuales**.
1. Seleccione la máquina virtual que desea proteger con Security Center.
    >[!TIP]
    > Para obtener información acerca de cómo crear una máquina virtual en Azure Stack Hub, consulte [este inicio rápido para máquinas virtuales Windows](/azure-stack/user/azure-stack-quick-windows-portal) o [este inicio rápido para máquinas virtuales Linux](/azure-stack/user/azure-stack-quick-linux-portal).
1. Seleccione **Extensiones**. Se muestra la lista de extensiones de máquina virtual instaladas en esta máquina virtual.
1. Seleccione la pestaña **Agregar**. En el menú **Nuevo recurso** se muestra la lista de extensiones de máquina virtual disponibles.
1. Seleccione la extensión **Azure Monitor, Update and Configuration Management** y seleccione **Crear**. Se abre la página de configuración **Instalar extensión**.
    >[!NOTE]
    > Si no ve la extensión **Azure Monitor, Update and Configuration Management** en Marketplace, póngase en contacto con su operador de Azure Stack Hub para que se la proporcione.
1. En la página de configuración **Instalar extensión**, pegue el **Id. del área de trabajo** y la **Clave del área de trabajo (clave principal)** que copió en el Bloc de notas en el paso anterior.
1. Una vez completada la configuración, seleccione **Aceptar**. El estado de la extensión se mostrará como **Aprovisionamiento realizado correctamente**. La máquina virtual puede tardar hasta una hora en aparecer en Security Center.

### <a name="onboard-your-linux-machines"></a>Incorporación de máquinas Linux

Para agregar máquinas Linux, necesita el comando WGET de la página **Administración de agentes**.

1. Desde la página **Administración de agentes**, copie el comando **WGET** en el Bloc de notas. Guarde este archivo en una ubicación a la que se pueda acceder desde el equipo Linux.
1. En el equipo Linux, abra el archivo con el comando WGET. Seleccione todo el contenido, cópielo y péguelo en una consola del terminal.
1. Una vez finalizada la instalación, puede validar que *omsagent* está instalado mediante la ejecución del comando *pgrep*. El comando devolverá el PID de *omsagent*.
    Los registros del agente se pueden encontrar en: */var/opt/microsoft/omsagent/\<workspace id>/log/* . La nueva máquina Linux puede tardar hasta 30 minutos en aparecer en Security Center.

### <a name="onboard-your-windows-machines"></a>Incorporación de máquinas Windows

Para agregar máquinas Windows, necesita la información de la página **Administración de agentes** y descargar el archivo de agente adecuado (32/64 bits).
1. Seleccione el vínculo **Descargar Agente para Windows** correspondiente a su tipo de procesador del equipo para descargar el archivo del programa de instalación.
1. En la página **Administración de agentes**, copie los valores de **Id. del área de trabajo** y **Clave principal** en el Bloc de notas.
1. Copie el archivo de instalación descargado en el equipo de destino y ejecútelo.
1. Siga el Asistente para instalación (**Siguiente**, **Acepto**, **Siguiente**, **Siguiente**).
    1. En la página **Azure Log Analytics**, pegue el **identificador del área de trabajo** y la **clave del área de trabajo (clave principal)** que copió en el Bloc de notas.
    1. Si el equipo tiene que notificar a un área de trabajo de Log Analytics en Azure Government Cloud, seleccione **Azure Gobierno de EE.UU.** en la lista desplegable **Azure Cloud**.
    1. Si el equipo necesita comunicarse a través de un servidor proxy con el servicio de Log Analytics, seleccione **Avanzado** y proporcione la dirección URL y el número de puerto del servidor proxy.
    1. Una establecida toda la configuración, seleccione **Siguiente**.
    1. En la página **Listo para instalar**, revise la configuración que se va a aplicar y seleccione **Instalar**.
    1. En la página **Configuración completada correctamente**, seleccione **Finalizar**.

Una vez completado el proceso, el **Agente de administración de Microsoft** aparece en el **Panel de control**. Puede revisar ahí la configuración y verificar que el agente esté conectado.

Para más información sobre cómo instalar y configurar el agente, vea [Conexión de máquinas Windows](../azure-monitor/agents/agent-windows.md#install-agent-using-setup-wizard).

::: zone-end

## <a name="verifying"></a>Comprobando

¡Enhorabuena! Ahora puede ver las máquinas de Azure y las que no son de Azure en un solo lugar. Abra la [página del inventario de recursos](asset-inventory.md) y filtre por los tipos de recursos correspondientes. Estos dos iconos distinguen los tipos:

  ![Icono de ASC para una máquina que no es de Azure.](./media/quick-onboard-linux-computer/security-center-monitoring-icon1.png) Máquina que no es de Azure

  ![Icono de ASC para una máquina de Azure.](./media/quick-onboard-linux-computer/security-center-monitoring-icon2.png) Azure VM

  ![Icono de ASC para un servidor de Azure Arc.](./media/quick-onboard-linux-computer/arc-enabled-machine-icon.png) Servidor habilitado para Azure Arc

## <a name="next-steps"></a>Pasos siguientes

En esta página se muestra cómo agregar las máquinas que no son de Azure a Azure Security Center. Para supervisar su estado, utilice las herramientas de inventario, como se explica en la siguiente página:

- [Exploración y administración de los recursos con Asset Inventory](asset-inventory.md)