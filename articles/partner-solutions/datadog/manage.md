---
title: 'Administración de un recurso de Datadog: soluciones de asociados de Azure'
description: En este artículo se describe la administración de un recurso de Datadog en Azure Portal. Cómo configurar el inicio de sesión único, eliminar una organización de Confluent y obtener soporte técnico.
ms.service: partner-services
ms.topic: conceptual
ms.date: 05/28/2021
author: tfitzmac
ms.author: tomfitz
ms.openlocfilehash: e964fd436795e19cc77a25efa95de5f6fcac87ca
ms.sourcegitcommit: 1b698fb8ceb46e75c2ef9ef8fece697852c0356c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110652458"
---
# <a name="manage-the-datadog-resource"></a>Administración del recurso Datadog

En este artículo se muestra cómo administrar la configuración de la integración de Azure con Datadog.

## <a name="resource-overview"></a>Información general sobre recursos

Para ver los detalles del recurso de Datadog, seleccione **Overview** (Información general) en el panel izquierdo.

:::image type="content" source="media/manage/resource-overview.png" alt-text="Información general del recurso de Datadog" border="true" lightbox="media/manage/resource-overview.png":::

Los detalles incluyen:

- Definición de un nombre de grupo de recursos
- Ubicación (región)
- Subscription
- Etiquetas
- Vínculo de inicio de sesión único en la organización de Datadog
- Oferta o plan de Datadog
- Período de facturación

También proporciona vínculos a paneles, registros y mapas de host de Datadog.

La pantalla de información general proporciona un resumen de los recursos que envían registros y métricas a Datadog.

- Tipo de recurso: tipo de recurso de Azure.
- Total de recursos: recuento de todos los recursos del tipo de recurso.
- Recursos que envían registros: recuento de los recursos que envían registros a Datadog mediante la integración.
- Recursos que envían métricas: recuento de los recursos que envían métricas a Datadog mediante la integración.

## <a name="reconfigure-rules-for-metrics-and-logs"></a>Volver a configurar las reglas para métricas y registros

Para cambiar las reglas de configuración de métricas y registros, seleccione **Metrics and Logs** (Métricas y registros) en el panel izquierdo.

:::image type="content" source="media/manage/reconfigure-metrics-and-logs.png" alt-text="Modificación de la configuración de registros y métricas del recurso de Datadog." border="true":::

Para más información, consulte [Configuración de métricas y registros](create.md#configure-metrics-and-logs).

## <a name="view-monitored-resources"></a>Vista de los recursos supervisados

Para ver la lista de recursos que emiten registros a Datadog, seleccione **Monitored Resources** (Recursos supervisados) en el panel izquierdo.

:::image type="content" source="media/manage/view-monitored-resources.png" alt-text="Vista de los recursos supervisados por Datadog" border="true":::

Puede filtrar la lista de recursos por tipo de recurso, nombre de grupo de recursos, ubicación y en función de si el recurso envía registros y métricas.

La columna **Logs to Datadog** (Registros a Datadog) indica si el recurso envía los registros a Datadog. Si el recurso no envía registros, este campo indica por qué no se envían los registros a Datadog. Los motivos pueden ser:

- El recurso no admite el envío de registros. Solo se pueden configurar para enviar registros a Datadog los tipos de recursos con categorías de registros de supervisión.
- Se ha alcanzado el límite de cinco configuraciones de diagnóstico. Cada recurso de Azure puede tener un máximo de cinco configuraciones de diagnóstico. Para más información, consulte [Creación de una configuración de diagnóstico para enviar los registros y las métricas de la plataforma a diferentes destinos](../../azure-monitor/essentials/diagnostic-settings.md).
- Error. El recurso está configurado para enviar los registros a Datadog, pero está bloqueado por un error.
- Registros no configurados. Solo los recursos de Azure con las etiquetas de recurso adecuadas están configurados para enviar los registros a Datadog.
- Región no admitida. El recurso de Azure está en una región que actualmente no admite el envío de registros a Datadog.
- Agente de Datadog no configurado. Las máquinas virtuales sin el agente de Datadog instalado no emiten registros a Datadog.

## <a name="api-keys"></a>Claves de API

Para ver la lista de claves de API del recurso de Datadog, seleccione **Keys** (Claves) en el panel izquierdo. Verá información acerca de las claves.

:::image type="content" source="media/manage/keys.png" alt-text="Claves de API de la organización de Datadog." border="true":::

Azure Portal proporciona una vista de solo lectura de las claves de API. Para administrar las claves, seleccione el vínculo al portal de Datadog. Después de realizar cambios en el portal de Datadog, actualice la vista de Azure Portal.

La integración de Datadog de Azure ofrece la posibilidad de instalar el agente de Datadog en una máquina virtual o en una instancia de App Service. Si no se selecciona una clave predeterminada, se produce un error en la instalación del agente de Datadog.

## <a name="monitor-virtual-machines-using-the-datadog-agent"></a>Supervisión de máquinas virtuales con el agente de Datadog

Puede instalar agentes de Datadog en las máquinas virtuales como una extensión. Vaya a **Virtual machine agent** (Agente de máquina virtual) en la configuración de la organización de Datadog en el panel izquierdo. Esta pantalla muestra la lista de todas las máquinas virtuales de la suscripción.

Para cada máquina virtual, se muestran los datos siguientes:

- Nombre de recurso: nombre de la máquina virtual
- Estado del recurso: indica si la máquina virtual está detenida o en ejecución. El agente de Datadog solo se puede instalar en máquinas virtuales que estén en ejecución. Si la máquina virtual está detenida, se deshabilitará la instalación del agente de Datadog.
- Versión del agente: número de versión del agente de Datadog.
- Estado del agente: indica si el agente de Datadog está en ejecución en la máquina virtual.
- Integraciones habilitadas: métricas principales que recopila el agente de Datadog.
- Método de instalación: herramienta específica utilizada para instalar el agente de Datadog. Por ejemplo, Chef o script.
- Envío de registros: indica si el agente de Datadog envía registros a Datadog.

Seleccione la máquina virtual en la que se va a instalar el agente de Datadog. Seleccione **Install Agent** (Instalar agente).

El portal le pide confirmación de que desea instalar el agente con la clave predeterminada. Seleccione **OK** (Aceptar) para comenzar la instalación. Azure muestra el estado **Installing** (Instalando) hasta que el agente se instala y aprovisiona.

Una vez instalado el agente de Datadog, el estado cambia a Installed (Instalado).

Para comprobar que se ha instalado el agente de Datadog, seleccione la máquina virtual y vaya a la ventana Extensions (Extensiones).

Para desinstalar los agentes de Datadog de una máquina virtual, vaya a **Virtual Machine Agent** (Agente de máquina virtual). Seleccione la máquina virtual y, a continuación, seleccione **Uninstall agent** (Desinstalar el agente).

## <a name="monitor-app-services-using-the-datadog-agent-as-an-extension"></a>Supervisión de App Services con el agente de Datadog como una extensión

Puede instalar agentes de Datadog en App Services como una extensión. Vaya a **App Service extension** (Extensión de App Service) en el panel izquierdo. Esta pantalla muestra la lista de todas las instancias de App Service de la suscripción.

Para cada instancia de App Service, se muestran los siguientes elementos de datos:

- Nombre de recurso: nombre de la máquina virtual.
- Estado del recurso: indica si la instancia de App Service está detenida o en ejecución. El agente de Datadog solo se puede instalar en instancias de App Service que estén en ejecución. Si la instancia de App Service está detenida, se deshabilitará la instalación del agente de Datadog.
- Plan de App Service: plan específico configurado para la instancia de App Service.
- Versión del agente: número de versión del agente de Datadog.

Para instalar el agente de Datadog, seleccione la instancia de App Service y, a continuación, seleccione **Install Extension** (Instalar extensión). El agente de Datadog más reciente se instala en la instancia de App Service como una extensión.

El portal confirma que desea instalar el agente de Datadog. Además, la configuración de la aplicación de la instancia de App Service específica se actualiza con la clave predeterminada. La instancia de App Service se reinicia una vez finalizada la instalación del agente de Datadog.

Seleccione **OK** (Aceptar) para comenzar el proceso de instalación del agente de Datadog. El portal muestra el estado **Installing** (Instalando) hasta que se instala el agente. Una vez instalado el agente de Datadog, el estado cambia a Installed (Instalado).

Para desinstalar los agentes de Datadog de la instancia de App Service, vaya a **App Service Extension** (Extensión de App Service). Seleccione la instancia de App Service y, a continuación, seleccione **Uninstall Extension** (Desinstalar extensión).

## <a name="reconfigure-single-sign-on"></a>Volver a configurar el inicio de sesión único

Si desea volver a configurar el inicio de sesión único, seleccione **Single sign-on** (Inicio de sesión único) en el panel izquierdo.

Para establecer el inicio de sesión único mediante Azure Active Directory, seleccione **Enable single sign-on through Azure Active Directory** (Habilitar el inicio de sesión único mediante Azure Active Directory).

El portal recupera la aplicación de Datadog adecuada desde Azure Active Directory. La aplicación proviene del nombre de la aplicación empresarial que seleccionó al configurar la integración. Seleccione el nombre de la aplicación de Datadog como se muestra a continuación:
 
:::image type="content" source="media/manage/reconfigure-single-sign-on.png" alt-text="Volver a configurar la aplicación de inicio de sesión único." border="true":::
 
## <a name="change-plan"></a>Cambio de plan

Para cambiar el plan de facturación de Datadog, vaya a **Información general** y seleccione **Cambiar plan**.

:::image type="content" source="media/manage/datadog-select-change-plan.png" alt-text="Selección del cambio de plan de facturación de Datadog." border="true":::

El portal recupera todos los planes de Datadog disponibles para el inquilino. Seleccione el plan adecuado y haga clic en **Cambiar plan**.

:::image type="content" source="media/manage/datadog-change-plan.png" alt-text="Selección del plan de facturación de Datadog para cambiar." border="true":::
  
## <a name="disable-or-enable-integration"></a>Deshabilitación o habilitación de la integración

Puede dejar de enviar registros y métricas desde Azure a Datadog. Se le seguirán facturando otros servicios de Datadog que no estén relacionados con la supervisión de métricas y registros.

Para deshabilitar la integración de Azure con Datadog, vaya a **Overview** (Información general). Seleccione **Disable** (Deshabilitar) y **OK** (Aceptar).
 
:::image type="content" source="media/manage/disable.png" alt-text="Deshabilitación del recurso Datadog." border="true":::

Para habilitar la integración de Azure con Datadog, vaya a **Overview** (Información general). Seleccione **Enable** (Habilitar) y **OK** (Aceptar). Al seleccionar **Enable** (Habilitar), se recupera cualquier configuración anterior de métricas y registros. La configuración determina qué recursos de Azure emiten métricas y registros a Datadog. Después de completar el paso, los registros y las métricas se envían a Datadog.
 
:::image type="content" source="media/manage/enable.png" alt-text="Habilitación del recurso Datadog." border="true":::

## <a name="delete-datadog-resource"></a>Eliminación del recurso de Datadog

Vaya a **Overview** (Información general) en el panel izquierdo y seleccione **Delete** (Eliminar). Confirme que desea eliminar el recurso de Datadog. Seleccione **Eliminar**.

:::image type="content" source="media/manage/delete.png" alt-text="Eliminación del recurso de Datadog" border="true":::

Si solo se asigna un recurso de Datadog a una organización de Datadog, los registros y las métricas ya no se envían a Datadog. Se detiene toda la facturación para Datadog mediante Azure Marketplace.

Si se asigna más de un recurso de Datadog a la organización de Datadog, la eliminación del recurso de Datadog solo deja de enviar los registros y las métricas de ese recurso de Datadog. Dado que la organización de Datadog está vinculada a otros recursos de Azure, la facturación continúa mediante Azure Marketplace.

## <a name="next-steps"></a>Pasos siguientes

Para obtener ayuda para solucionar problemas, consulte [Solución de problemas de Datadog en Azure](troubleshoot.md).
