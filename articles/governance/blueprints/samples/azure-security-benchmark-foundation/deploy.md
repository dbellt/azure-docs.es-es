---
title: Implementación del ejemplo de plano técnico de Azure Security Benchmark Foundation
description: Pasos de implementación para el ejemplo de plano técnico de Azure Security Benchmark Foundation, incluidos los detalles de los parámetros del artefacto de plano técnico.
ms.date: 03/12/2021
ms.topic: sample
ms.openlocfilehash: 6afdc94624d39ce60b96ad33639721f130a056ef
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108765186"
---
# <a name="deploy-the-azure-security-benchmark-foundation-blueprint-sample"></a>Implementación del ejemplo de plano técnico de Azure Security Benchmark Foundation

Para implementar el ejemplo de plano técnico de Azure Security Benchmark Foundation, debe realizar los pasos siguientes:

> [!div class="checklist"]
> - Crear un plano técnico a partir del ejemplo
> - Marcar la copia del ejemplo como **publicada**
> - Asignar la copia del plano técnico a una suscripción existente

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free) antes de empezar.

## <a name="create-blueprint-from-sample"></a>Creación de un plano técnico a partir del ejemplo

En primer lugar, implemente el ejemplo de plano técnico mediante la creación de un plano técnico en su entorno tomando el ejemplo como punto de partida.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. En la página **Introducción** de la izquierda, seleccione el botón **Crear** en _Crear un plano técnico_.

1. Busque el ejemplo de plano técnico de **Azure Security Benchmark Foundation** en _Otras muestras_ y seleccione **Usar esta muestra**.

1. Escriba los _Aspectos básicos_ del ejemplo de plano técnico:

   - **Nombre del plano técnico**: proporcione un nombre para su copia del ejemplo de plano técnico de Azure Security Benchmark Foundation.
   - **Ubicación de definición**: use los puntos suspensivos y seleccione el grupo de administración donde guardar la copia del ejemplo.

1. Seleccione la pestaña _Artefactos_ en la parte superior de la página **Siguiente: Artefactos** en la parte inferior de la página.

1. Revise la lista de artefactos que componen el ejemplo de plano técnico. Muchos de los artefactos tienen parámetros que se definirán más tarde. Seleccione **Guardar borrador** cuando haya terminado de revisar el ejemplo de plano técnico.

## <a name="publish-the-sample-copy"></a>Publicación de la copia del ejemplo

La copia del ejemplo de plano técnico ahora se ha creado en el entorno. Se crea en el modo **Borrador** y debe **publicarse** antes de que se pueda asignar e implementar. La copia del ejemplo de plano técnico se puede personalizar para adaptarla a su entorno y sus necesidades, pero esa modificación puede apartarla del plano técnico de Azure Security Benchmark Foundation.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. En la parte izquierda, seleccione la página **Definiciones del plano técnico**. Use los filtros para buscar su copia del ejemplo de plano técnico y, a continuación, selecciónela.

1. Seleccione **Publicar plano técnico** en la parte superior de la página. En la nueva página de la derecha, especifique una **versión** para la copia del ejemplo de plano técnico. Esta propiedad es útil si realiza una modificación posteriormente. Escriba **Notas de cambios**, como "Primera versión publicada del ejemplo de plano técnico de Azure Security Benchmark Foundation". A continuación, seleccione **Publicar** en la parte inferior de la página.

## <a name="assign-the-sample-copy"></a>Asignación de la copia de ejemplo

Una vez que la copia del ejemplo de plano técnico se haya **publicado** correctamente, se podrá asignar a una suscripción dentro del grupo de administración donde se guardó. En este paso se proporcionan los parámetros para hacer que cada implementación de la copia del ejemplo de plano técnico sea única.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. En la parte izquierda, seleccione la página **Definiciones del plano técnico**. Use los filtros para buscar su copia del ejemplo de plano técnico y, a continuación, selecciónela.

1. Seleccione **Asignar plano técnico** en la parte superior de la página de definición del plano técnico.

1. Proporcione los valores de parámetro para la asignación de plano técnico:

   - Aspectos básicos
       - **Suscripciones**: seleccione una o varias de las suscripciones que están en el grupo de administración donde guardó la copia del ejemplo de plano técnico. Si selecciona más de una suscripción, se creará una asignación para cada una mediante los parámetros especificados.
     - **Nombre de asignación**: el nombre se rellena de antemano de forma automática en función del nombre del plano técnico.
       Cámbielo si fuera necesario o déjelo tal cual.
     - **Ubicación**: seleccione una región para la identidad administrada en la que se va a crear.
     - Azure Blueprint usa esta identidad administrada para implementar todos los artefactos del plano técnico asignado.
       Para más información, consulte [Identidades administradas para recursos de Azure](../../../../active-directory/managed-identities-azure-resources/overview.md).
     - **Versión de definición de Blueprint**: Elija una versión **publicada** de la copia del ejemplo de plano técnico.

   - Asignación de bloqueo

     Seleccione el valor de bloqueo de plano técnico para el entorno. Para más información, consulte [Bloqueo de recursos en planos técnicos](../../concepts/resource-locking.md).

   - Identidad administrada

     Elija la opción de identidad administrada predeterminada _asignada por el sistema_ o la opción de identidad _asignada por el usuario_.

   - Parámetros de plano técnico

     Muchos de los artefactos de la definición de plano técnico usan los parámetros definidos en esta sección para proporcionar coherencia.

     - **Prefix for resources and resource groups** (Prefijo para recursos y grupos de recursos): esta cadena se usa como prefijo para todos los nombres de recursos y grupos de recursos.
     - **Nombre del centro**: nombre del centro.
     - **Log retention (days)** (Retención del registro [días]): número de días que se retienen los registros; si introduce "0", se conservan indefinidamente.
     - **Deploy hub** (Implementar centro): escriba "true" o "false" para especificar si la asignación implementa los componentes de centro de la arquitectura.
     - **Ubicación del concentrador**: ubicación del grupo de recursos de centro.
     - **Direcciones IP de destino**: direcciones IP de destino de la conectividad de salida; es una lista separada por comas de direcciones IP o prefijos de intervalos IP.
     - **Nombre de Network Watcher**: nombre del recurso de Network Watcher.
     - **Network Watcher resource group name** (Nombre del grupo de recursos de Network Watcher): nombre del grupo de recursos de Network Watcher.
     - **Enable DDoS protection** (Habilitar protección contra DDoS): especifique "true" o "false" para indicar si la protección contra DDoS está habilitada o no en la red virtual.

     > [!NOTE]
     > Si Network Watcher ya está habilitado, se recomienda usar el grupo de recursos de Network Watcher existente. También debe proporcionar la ubicación del grupo de recursos de Network Watcher existente para el parámetro de artefacto **Ubicación del grupo de recursos de Network Watcher**.

   - Parámetros de artefacto

     Los parámetros definidos en esta sección se aplican al artefacto en el que se define. Estos parámetros son [parámetros dinámicos](../../concepts/parameters.md#dynamic-parameters), ya que se definen durante la asignación del plano técnico. Para obtener una lista completa de los parámetros de los artefactos y sus descripciones, consulte la [tabla de parámetros de los artefactos](#artifact-parameters-table).

1. Una vez que se hayan especificado todos los parámetros, seleccione **Asignar** en la parte inferior de la página. Se crea la asignación del plano técnico y comienza la implementación del artefacto. La implementación tarda aproximadamente una hora. Para comprobar el estado de implementación, abra la asignación del plano técnico.

> [!WARNING]
> El servicio Azure Blueprints y los ejemplos de plano técnico incorporados son **gratuitos**. El precio de los recursos de Azure se [calcula por producto](https://azure.microsoft.com/pricing/). Use la [calculadora de precios](https://azure.microsoft.com/pricing/calculator/) para estimar el costo de la ejecución de los recursos implementados en este ejemplo de plano técnico.

## <a name="artifact-parameters-table"></a>Tabla de parámetros de los artefactos

En la tabla siguiente se proporciona una lista de los parámetros del plano técnico:

|Nombre del artefacto|Tipo de artefacto|Nombre de parámetro|Descripción|
|-|-|-|-|
|Grupo de recursos del centro|Resource group|Definición de un nombre de grupo de recursos|Bloqueado: concatena el prefijo con el nombre del centro|
|Grupo de recursos del centro|Resource group|Ubicación del grupo de recursos|Bloqueado: usa la ubicación del centro|
|Plantilla de Azure Firewall|Plantilla de Resource Manager|Azure Firewall private IP address (Dirección IP privada de Azure Firewall)||
|Plantillas de Azure Log Analytics y Diagnostics|Plantilla de Resource Manager|Ubicación del área de trabajo de Log Analytics|Ubicación donde se crea el área de trabajo de Log Analytics; ejecute `Get-AzLocation | Where-Object Providers -like 'Microsoft.OperationalInsights' | Select DisplayName` en Azure PowersShell para ver las regiones disponibles|
|Plantillas de Azure Log Analytics y Diagnostics|Plantilla de Resource Manager|Azure Automation account ID (Id. de cuenta de Azure Automation) (opcional) |Identificador de recurso de la cuenta de Automation; se usa para crear un servicio vinculado entre Log Analytics y una cuenta de Automation|
|Plantilla del grupo de seguridad de red de Azure|Plantilla de Resource Manager|Enable NSG flow logs (Habilitar registros de flujo del grupo de seguridad de red)|Escriba "true" o "false" para habilitar o deshabilitar los registros de flujo de grupo de seguridad de red|
|Plantilla de centro de Azure Virtual Network|Plantilla de Resource Manager|Virtual network address prefix (Prefijo de dirección de red virtual)|Prefijo de dirección de red virtual para la red virtual de centro|
|Plantilla de centro de Azure Virtual Network|Plantilla de Resource Manager|Firewall subnet address prefix (Prefijo de dirección de subred de Firewall)|Prefijo de dirección de subred de Firewall para la red virtual de centro|
|Plantilla de centro de Azure Virtual Network|Plantilla de Resource Manager|Bastion subnet address prefix (Prefijo de dirección de subred de Bastion)|Prefijo de dirección de subred de Bastion para la red virtual de centro|
|Plantilla de centro de Azure Virtual Network|Plantilla de Resource Manager|Gateway subnet address prefix (Prefijo de dirección de subred de puerta de enlace)|Prefijo de dirección de la subred de puerta de enlace para la red virtual de centro|
|Plantilla de centro de Azure Virtual Network|Plantilla de Resource Manager|Management subnet address prefix (Prefijo de dirección de subred de administración)|Prefijo de dirección de subred de administración para la red virtual de centro|
|Plantilla de centro de Azure Virtual Network|Plantilla de Resource Manager|Jump box subnet address prefix (Prefijo de dirección de subred de jumpbox)|Prefijo de dirección de subred de jumpbox para la red virtual de centro|
|Plantilla de centro de Azure Virtual Network|Plantilla de Resource Manager|Subnet address names (Nombres de direcciones de subred) (opcional)|Matriz de nombres de subred que se va a implementar en la red virtual del centro; por ejemplo, "subnet1", "subnet2"|
|Plantilla de centro de Azure Virtual Network|Plantilla de Resource Manager|Subnet address prefixes (Prefijos de direcciones de subred) (opcional)|Matriz de prefijos de direcciones IP para las subredes opcionales de la red virtual del centro; por ejemplo, "10.0.7.0/24", "10.0.8.0/24"|
|Grupo de recursos de radio|Resource group|Definición de un nombre de grupo de recursos|Bloqueado: concatena el prefijo con el nombre del radio|
|Grupo de recursos de radio|Resource group|Ubicación del grupo de recursos|Bloqueado: usa la ubicación del centro|
|Plantilla de radio de Azure Virtual Network|Plantilla de Resource Manager|Deploy spoke (Implementar radio)|Escriba "true" o "false" para especificar si la asignación implementa los componentes de radio de la arquitectura|
|Plantilla de radio de Azure Virtual Network|Plantilla de Resource Manager|Hub subscription ID (Id. de suscripción al centro)|Identificador de la suscripción donde se implementa el centro; el valor predeterminado es la suscripción en la que se encuentra la definición del plano técnico|
|Plantilla de radio de Azure Virtual Network|Plantilla de Resource Manager|Spoke name (Nombre de radio)|Nombre del radio|
|Plantilla de radio de Azure Virtual Network|Plantilla de Resource Manager|Prefijo de dirección de Virtual Network|Prefijo de dirección de Virtual Network para la red virtual de radios|
|Plantilla de radio de Azure Virtual Network|Plantilla de Resource Manager|Subnet address prefix (Prefijo de dirección de subred)|Prefijo de dirección de subred de la red virtual de radios|
|Plantilla de radio de Azure Virtual Network|Plantilla de Resource Manager|Subnet address names (Nombres de direcciones de subred) (opcional)|Matriz de nombres de subred que se va a implementar en la red virtual de radios; por ejemplo, "subnet1", "subnet2".|
|Plantilla de radio de Azure Virtual Network|Plantilla de Resource Manager|Subnet address prefixes (Prefijos de direcciones de subred) (opcional)|Matriz de prefijos de direcciones IP para las subredes opcionales de la red virtual de radios; por ejemplo, "10.0.7.0/24", "10.0.8.0/24"|
|Plantilla de radio de Azure Virtual Network|Plantilla de Resource Manager|Deploy spoke (Implementar radio)|Escriba "true" o "false" para especificar si la asignación implementa los componentes de radio de la arquitectura|
|Plantilla de Azure Network Watcher|Plantilla de Resource Manager|Network Watcher location (Ubicación de Network Watcher)|Ubicación para el recurso de Network Watcher|
|Plantilla de Azure Network Watcher|Plantilla de Resource Manager|Network Watcher resource group location (Ubicación del grupo de recursos de Network Watcher)|Si Network Watcher ya está habilitado, este valor de parámetro **debe** coincidir con la ubicación del grupo de recursos de Network Watcher existente.|

## <a name="troubleshooting"></a>Solución de problemas

Si aparece el error `The resource group 'NetworkWatcherRG' failed to deploy due to the
following error: Invalid resource group location '{location}'. The Resource group already exists in
location '{location}'.`, compruebe que el parámetro de plano técnico **Nombre de grupo de recursos de Network Watcher** especifica el nombre del grupo de recursos de Network Watcher existente y que el parámetro de artefacto **Ubicación de grupo de recursos de Network Watcher Network** especifica la ubicación del grupo de recursos de Network Watcher existente.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha revisado los pasos para implementar el ejemplo de plano técnico de Azure Security Benchmark Foundation, consulte el siguiente artículo para conocer la arquitectura:

> [!div class="nextstepaction"]
> [Introducción al ejemplo de plano técnico de Azure Security Benchmark Foundation](./index.md)

Artículos adicionales sobre planos técnicos y cómo utilizarlos:

- Información acerca del [ciclo de vida del plano técnico](../../concepts/lifecycle.md).
- Descubra cómo utilizar [parámetros estáticos y dinámicos](../../concepts/parameters.md).
- Aprenda a personalizar el [orden de secuenciación de planos técnicos](../../concepts/sequencing-order.md).
- Averigüe cómo usar el [bloqueo de recursos de planos técnicos](../../concepts/resource-locking.md).
- Aprenda a [actualizar las asignaciones existentes](../../how-to/update-existing-assignments.md).
