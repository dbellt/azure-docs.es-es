---
title: Conexión de máquinas híbridas a Azure a gran escala
description: En este artículo, obtendrá información sobre cómo conectar máquinas a Azure mediante servidores habilitados para Azure Arc con una entidad de servicio.
ms.date: 03/04/2021
ms.topic: conceptual
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 4aad01fd6991c059b2cf891fd4f06ae83a78a0e4
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107831608"
---
# <a name="connect-hybrid-machines-to-azure-at-scale"></a>Conexión de máquinas híbridas a Azure a gran escala

Puede activar los servidores habilitados para Azure Arc para varias máquinas Windows o Linux en su entorno, con varias opciones flexibles, en función de sus requisitos. Con el script de plantilla que proporcionamos, puede automatizar cada paso de la instalación, incluido el establecimiento de la conexión a Azure Arc. Sin embargo, es necesario ejecutar este script de forma interactiva con una cuenta que tenga permisos elevados en la máquina de destino y en Azure.

Para conectar las máquinas a los servidores habilitados para Azure Arc, puede usar una [entidad de servicio](../../active-directory/develop/app-objects-and-service-principals.md) de Azure Active Directory en lugar de usar su identidad con privilegios para [conectar interactivamente la máquina](onboard-portal.md). Una entidad de servicio es una identidad de administración limitada especial a la que solo se concede el permiso mínimo necesario para conectar máquinas a Azure con el comando `azcmagent`. Es más seguro que usar una cuenta con más privilegios, como un Administrador de inquilinos, y sigue nuestros procedimientos recomendados de seguridad de control de acceso. La entidad de servicio se usa solo durante la incorporación; no se usa para ningún otro propósito.  

Los métodos de instalación para instalar y configurar el agente Connected Machine requieren que el método automatizado que se use tenga permisos de administrador en las máquinas. En Linux, mediante la cuenta raíz y, en Windows, como miembro del grupo local de administradores.

Antes de comenzar, asegúrese de revisar los [requisitos previos](agent-overview.md#prerequisites) y compruebe que su suscripción y sus recursos los cumplen. Para obtener información sobre las regiones admitidas y otras consideraciones relacionadas, consulte [Regiones de Azure admitidas](overview.md#supported-regions). Revise también nuestra [guía de planeamiento a gran escala](plan-at-scale-deployment.md) para comprender los criterios de diseño e implementación, así como nuestras recomendaciones de administración y supervisión.  

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="create-a-service-principal-for-onboarding-at-scale"></a>Creación de una entidad de servicio para la incorporación a gran escala

Puede usar [Azure PowerShell](/powershell/azure/install-az-ps) para crear una entidad de servicio con el cmdlet [New-AzADServicePrincipal](/powershell/module/Az.Resources/New-AzADServicePrincipal). O bien, puede seguir los pasos que se indican en [Creación de una entidad de servicio con Azure Portal](../../active-directory/develop/howto-create-service-principal-portal.md) para completar esta tarea.

> [!NOTE]
> Antes de crear una entidad de servicio, la cuenta debe ser miembro del rol **Propietario** o **Administrador de acceso de usuario** en la suscripción que quiere usar para la incorporación. Si no tiene permisos suficientes para crear asignaciones de roles, es posible que se cree la entidad de servicio, pero no podrá incorporar máquinas.
>

Para crear la entidad de servicio mediante PowerShell, realice estos pasos:

1. Ejecute el siguiente comando: Debe almacenar la salida del cmdlet [`New-AzADServicePrincipal`](/powershell/module/az.resources/new-azadserviceprincipal) en una variable o no podrá recuperar la contraseña necesaria en un paso posterior.

    ```azurepowershell-interactive
    $sp = New-AzADServicePrincipal -DisplayName "Arc-for-servers" -Role "Azure Connected Machine Onboarding"
    $sp
    ```

    ```output
    Secret                : System.Security.SecureString
    ServicePrincipalNames : {ad9bcd79-be9c-45ab-abd8-80ca1654a7d1, https://Arc-for-servers}
    ApplicationId         : ad9bcd79-be9c-45ab-abd8-80ca1654a7d1
    ObjectType            : ServicePrincipal
    DisplayName           : Hybrid-RP
    Id                    : 5be92c87-01c4-42f5-bade-c1c10af87758
    Type                  :
    ```

2. Para recuperar la contraseña almacenada en la variable `$sp`, ejecute el siguiente comando:

    ```azurepowershell-interactive
    $credential = New-Object pscredential -ArgumentList "temp", $sp.Secret
    $credential.GetNetworkCredential().password
    ```

3. En la salida, busque el valor de la contraseña en el campo **password** y cópiela. Busque también el valor del campo **ApplicationId** y cópielo también. Guárdelos para más adelante en un lugar seguro. Si olvida o pierde la contraseña de la entidad de servicio, puede restablecerla mediante el cmdlet [`New-AzADSpCredential`](/powershell/module/azurerm.resources/new-azurermadspcredential).

Los valores de las siguientes propiedades se usan con los parámetros que se pasan a `azcmagent`:

* El valor de la propiedad **ApplicationId** se usa para el valor del parámetro `--service-principal-id`.
* El valor de la propiedad **password** se usa para el parámetro `--service-principal-secret` que se usa para conectar el agente.

> [!NOTE]
> Asegúrese de usar la propiedad **ApplicationId** de la entidad de servicio, no la propiedad **Id**.
>

El rol **Incorporación de Azure Connected Machine** contiene solo los permisos necesarios para incorporar una máquina. Puede asignar a la entidad de servicio el permiso para permitir que su ámbito incluya un grupo de recursos o una suscripción. Para agregar una asignación de roles, consulte [Asignación de roles de Azure mediante Azure Portal](../../role-based-access-control/role-assignments-portal.md) o [Asignación de roles de Azure mediante la CLI de Azure](../../role-based-access-control/role-assignments-cli.md).

## <a name="generate-the-installation-script-from-the-azure-portal"></a>Generación del script de instalación desde Azure Portal

El script para automatizar la descarga y la instalación, y para establecer la conexión con Azure Arc, está disponible en Azure Portal. Para completar el proceso, haga lo siguiente:

1. En el explorador, vaya a [Azure Portal](https://portal.azure.com).

1. En la página **Servidores: Azure Arc**, seleccione **Agregar** en la parte superior izquierda.

1. En la página **Seleccionar un método**, elija el icono **Add multiple servers** (Agregar varios servidores) y, luego, seleccione **Generar script**.

1. En la página **Generar script**, seleccione la suscripción y el grupo de recursos en el que desea que se administre la máquina en Azure. Seleccione la ubicación de Azure en la que se almacenarán los metadatos de la máquina. Esta ubicación puede ser la misma u otra diferente que la del grupo de recursos.

1. En la página **Requisitos previos**, revise la información y, luego, seleccione **Siguiente: Detalles del recurso**.

1. En la página **Detalles del recurso**, proporcione lo siguiente:

    1. En la lista desplegable **Grupo de recursos**, seleccione el grupo de recursos desde el que se administrará la máquina.
    1. En la lista desplegable **Región**, seleccione la región de Azure en la que se almacenarán los metadatos de los servidores.
    1. En la lista desplegable **Sistema operativo**, seleccione el sistema operativo en el que se ejecutará el script.
    1. Si la máquina se comunica mediante un servidor proxy para conectarse a Internet, especifique la dirección IP del servidor proxy o el nombre y el número de puerto que usará la máquina para comunicarse con el servidor proxy. Escriba el valor con el formato `http://<proxyURL>:<proxyport>`.
    1. Seleccione **Siguiente: Autenticación**.

1. En la página **Autenticación**, en la lista desplegable **Entidad de servicio**, elija **Arc-for-servers**.  Después, seleccione **Siguiente: Etiquetas**.

1. En la página **Etiquetas**, revise las **etiquetas de ubicación física** predeterminadas sugeridas y escriba un valor, o bien especifique una o varias **etiquetas personalizadas** para que cumplan sus estándares.

1. Seleccione **Siguiente: Descargar y ejecutar el script**.

1. En la pestaña **Descargar y ejecutar el script**, revise la información del resumen y seleccione **Descargar**. Si todavía necesita realizar cambios, seleccione **Anterior**.

En Windows, se le pedirá que guarde en el equipo `OnboardingScript.ps1`, y en Linux `OnboardingScript.sh`.

## <a name="install-the-agent-and-connect-to-azure"></a>Instalación del agente y conexión a Azure

Tomando la plantilla de script creada anteriormente, puede instalar y configurar el agente de Connected Machine en varias máquinas híbridas de Linux y Windows mediante la herramienta de automatización preferida de su organización. El script realiza pasos similares que se describen en el artículo [Conexión de máquinas híbridas a Azure desde Azure Portal](onboard-portal.md). La diferencia radica en el paso final en el que se establece la conexión a Azure Arc con el comando `azcmagent` mediante la entidad de servicio.

A continuación, se muestran los valores que se configuran en el comando `azcmagent` que se va a usar para la entidad de servicio.

* `service-principal-id`: identificador único (GUID) que representa el identificador de aplicación de la entidad de servicio.
* `service-principal-secret`: contraseña de la entidad de servicio.
* `tenant-id`: el identificador único (GUID) que representa la instancia dedicada de Azure AD.
* `subscription-id`: el identificador de suscripción (GUID) de la suscripción de Azure en la que quiere que estén las máquinas.
* `resource-group`: el nombre del grupo de recursos al que desea que pertenezcan las máquinas conectadas.
* `location`: consulte [Regiones de Azure compatibles](overview.md#supported-regions). Esta ubicación puede ser la misma u otra diferente que la del grupo de recursos.
* `resource-name`: (*opcional*) Se usa para la representación de recursos de Azure de la máquina local. Si no especifica este valor, se usa el nombre de host de la máquina.

Para obtener más información sobre la herramienta de línea de comandos `azcmagent`, revise la [referencia de Azcmagent](./manage-agent.md).

>[!NOTE]
>El script de Windows PowerShell solo admite la ejecución desde una versión de 64 bits de Windows PowerShell.
>

Después de instalar el agente y configurarlo para que se conecte a los servidores habilitados para Azure Arc, vaya a Azure Portal para comprobar que el servidor se ha conectado correctamente. Vea las máquinas en [Azure Portal](https://aka.ms/hybridmachineportal).

![Una conexión de servidor correcta](./media/onboard-portal/arc-for-servers-successful-onboard.png)

## <a name="next-steps"></a>Pasos siguientes

- Puede encontrar información sobre la solución de problemas en la guía [Solución de problemas de conexión del agente de Connected Machine](troubleshoot-agent-onboard.md).

- Aprenda a administrar la máquina con [Azure Policy](../../governance/policy/overview.md) para, por ejemplo, la [configuración de invitado](../../governance/policy/concepts/guest-configuration.md) de máquina virtual, la comprobación de que la máquina informa al área de trabajo de Log Analytics esperada, la habilitación de la supervisión con [Azure Monitor con máquinas virtuales](../../azure-monitor/vm/vminsights-enable-policy.md) y mucho más.

- Más información sobre el [agente de Log Analytics](../../azure-monitor/agents/log-analytics-agent.md). El agente de Log Analytics para Windows y Linux es necesario si quiere recopilar datos de supervisión del sistema operativo y de las cargas de trabajo con Azure Monitor para VM, administrarlos con runbooks de Automation o con características como Update Management, o usar otros servicios de Azure, como [Azure Security Center](../../security-center/security-center-introduction.md).
