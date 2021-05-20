---
title: Actualización de clústeres de Azure Service Fabric
description: Más información sobre las opciones para actualizar el clúster de Azure Service Fabric
ms.topic: conceptual
ms.date: 03/26/2021
ms.openlocfilehash: 9af6a9b4d515fcede5bc0444a93ae8118077e7dd
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109738759"
---
# <a name="upgrading-and-updating-azure-service-fabric-clusters"></a>Actualización de clústeres de Azure Service Fabric

Un clúster de Azure Service Fabric es un recurso de su propiedad, pero que está parcialmente administrado por Microsoft. En este artículo se describen las opciones para cuándo y cómo actualizar el clúster de Azure Service Fabric.

## <a name="automatic-versus-manual-upgrades"></a>Actualizaciones automáticas frente a manuales

Es fundamental asegurarse de que el clúster de Service Fabric ejecute siempre una [versión en tiempo de ejecución compatible](service-fabric-versions.md). Cada vez que Microsoft anuncia el lanzamiento de una nueva versión de Service Fabric, se marca la versión anterior para que *finalice el soporte técnico* después de un mínimo de 60 días a partir de esa fecha. Las nuevas versiones se anuncian en el [blog del equipo de Service Fabric](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric).

Catorce días antes de la expiración de la versión que se está ejecutando en el clúster, se genera un evento de mantenimiento que coloca el clúster en un estado de mantenimiento de *advertencia*. El clúster permanece en un estado de advertencia hasta que actualiza a una versión en tiempo de ejecución compatible.

Puede configurar el clúster para recibir actualizaciones automáticas de Service Fabric a medida que se publican en Microsoft o elegir manualmente en una lista de versiones admitidas en la actualidad. Estas opciones están disponibles en la sección **Actualizaciones de tejido** del recurso de clúster de Service Fabric.

:::image type="content" source="./media/service-fabric-cluster-upgrade/fabric-upgrade-mode.png" alt-text="Seleccione actualizaciones automáticas o manuales en la sección &quot;Actualizaciones de tejido&quot; del recurso de clúster en Azure Portal.":::

También puede establecer el modo de actualización del clúster y seleccionar una versión en tiempo de ejecución [mediante una plantilla de Resource Manager](service-fabric-cluster-upgrade-version-azure.md#resource-manager-template).

Las actualizaciones automáticas son el modo de actualización recomendado, ya que esta opción garantiza que el clúster permanezca en un estado compatible y se beneficia de las correcciones y características más recientes, a la vez que permite programar actualizaciones de una manera que sea menos problemática para las cargas de trabajo mediante una estrategia de [implementación en lanzamientos](#wave-deployment-for-automatic-upgrades).

## <a name="wave-deployment-for-automatic-upgrades"></a>Implementación en lanzamientos para actualizaciones automáticas

Con la implementación en lanzamientos, puede minimizar la interrupción de una actualización en el clúster seleccionando el nivel de madurez de una actualización, en función de la carga de trabajo. Por ejemplo, puede configurar una canalización de implementación en lanzamientos de *Prueba* -> *Fase* -> *Producción* para los distintos clústeres de Service Fabric con el fin de probar la compatibilidad de una actualización en tiempo de ejecución antes de aplicarla a las cargas de trabajo de producción.

Para participar en una implementación en lanzamientos, especifique uno de los siguientes valores de lanzamiento para el clúster (en su plantilla de implementación):

* **Lanzamiento 0**: los clústeres se actualizan en cuanto se publica una nueva compilación de Service Fabric. Diseñado para clústeres de prueba y desarrollo.
* **Lanzamiento 1**: los clústeres se actualizan una semana (siete días) después de que se lance una nueva compilación. Diseñado para clústeres de preproducción y almacenamiento provisional.
* **Lanzamiento 2**: los clústeres se actualizan dos semanas (14 días) después de que se lance una nueva compilación. Diseñado para clústeres de producción.

Puede registrarse para recibir notificaciones por correo electrónico con vínculos a ayuda adicional si se produce un error en una actualización del clúster. Consulte [Implementación en lanzamientos para actualizaciones automáticas](service-fabric-cluster-upgrade-version-azure.md#wave-deployment-for-automatic-upgrades) para comenzar.

## <a name="phases-of-automatic-upgrade"></a>Fases de actualización automática

Microsoft mantiene el código en tiempo de ejecución de Service Fabric y la configuración que se ejecuta en un clúster de Azure. Realizamos actualizaciones supervisadas automáticas del software según se vayan necesitando. Estas actualizaciones podrían ser de código, de configuración o ambas. Para minimizar el impacto de estas actualizaciones en las aplicaciones, se realizan en las fases siguientes:

### <a name="phase-1-an-upgrade-is-performed-by-using-all-cluster-health-policies"></a>Fase 1: La actualización se realiza con todas las directivas de mantenimiento de clústeres

Durante esta fase, las actualizaciones se realizan en un dominio de actualización cada vez, y las aplicaciones que se ejecutaban en el clúster continúan ejecutándose sin tiempo de inactividad. Las directivas de mantenimiento de clústeres (en cuanto al estado del nodo y el estado de la aplicación) se aplican durante la actualización.

Si las directivas del clúster no se cumplen, la actualización se revierte y se envía un correo electrónico al propietario de la suscripción. El correo electrónico contiene la siguiente información:

* Notificación de que tuvimos que revertir una actualización de clúster.
* Acciones correctoras sugeridas, si hay alguna.
* Número de días (*n*) hasta que ejecutemos la fase 2.

Vamos a intentar ejecutar la misma actualización unas cuantas veces más por si se produjeron errores en las actualizaciones por razones de infraestructura. Después de *n* días desde la fecha en que se envió el correo electrónico, continuamos con la fase 2.

Si se cumplen las directivas de mantenimiento del clúster, la actualización se considera correcta y se marca como completada. Esta situación puede ocurrir durante la ejecución inicial de la actualización o cualquiera de las ejecuciones posteriores de la actualización en esta fase. No se recibe ninguna confirmación por correo electrónico de una ejecución correcta, para evitar el envío de demasiados mensajes de correo electrónico. La recepción de un correo electrónico indica una excepción a las operaciones normales. Esperamos que la mayoría de las actualizaciones de clúster funcionen sin afectar a la disponibilidad de las aplicaciones.

### <a name="phase-2-an-upgrade-is-performed-by-using-default-health-policies-only"></a>Fase 2: La actualización se realiza solo con las directivas de mantenimiento predeterminadas

En esta fase, las directivas de mantenimiento se establecen de forma que el número de aplicaciones que tenían un estado correcto al principio de la actualización siga siendo el mismo durante el proceso de actualización. Al igual que en la fase 1, en la fase 2 las actualizaciones se realizan en un dominio de actualización cada vez, y las aplicaciones que se ejecutaban en el clúster continúan ejecutándose sin tiempo de inactividad. Las directivas de mantenimiento del clúster (una combinación del estado del nodo y el estado de todas las aplicaciones que se ejecutan en el clúster) se cumplen mientras dura la actualización.

Si no se cumplen las directivas de mantenimiento del clúster, la actualización se revierte. Después, se envía un correo electrónico al propietario de la suscripción. El correo electrónico contiene la siguiente información:

* Notificación de que tuvimos que revertir una actualización de clúster.
* Acciones correctoras sugeridas, si hay alguna.
* El número de días (n) hasta que ejecutemos la fase 3.

Vamos a intentar ejecutar la misma actualización unas cuantas veces más por si se produjeron errores en las actualizaciones por razones de infraestructura. Se envía un recordatorio por correo electrónico un par de días antes de llegar a n días. Después de los n días desde la fecha en que se envió el correo electrónico, continuamos con la fase 3. Es necesario tomarse en serio los correos electrónicos que le enviamos en la fase 2 y realizar acciones de corrección.

Si se cumplen las directivas de mantenimiento del clúster, la actualización se considera correcta y se marca como completada. Esto puede ocurrir durante la ejecución inicial de la actualización o cualquiera de las ejecuciones posteriores de la actualización en esta fase. No hay ningún correo electrónico de confirmación de una ejecución correcta.

### <a name="phase-3-an-upgrade-is-performed-by-using-aggressive-health-policies"></a>Fase 3: La actualización se realiza con directivas de mantenimiento agresivas

En esta fase, estas directivas de mantenimiento están orientadas a la finalización de la actualización en lugar de al mantenimiento de las aplicaciones. Pocas actualizaciones del clúster terminarán en esta fase. Si el clúster llega a esta fase, es probable que la aplicación no tenga el mantenimiento adecuado o pierda disponibilidad.

Igual que las otras dos fases, las actualizaciones de la fase 3 se realizan en un dominio de actualización cada vez.

Si no se cumplen las directivas de mantenimiento del clúster, la actualización se revierte. Vamos a intentar ejecutar la misma actualización unas cuantas veces más por si se produjeron errores en las actualizaciones por razones de infraestructura. Después, el clúster se ancla y ya no recibirán soporte técnico ni actualizaciones.

Se enviará un correo electrónico con esta información al propietario de la suscripción, junto con las acciones correctoras. No esperamos que los clústeres entren en un estado en el que la fase 3 produzca errores.

Si se cumplen las directivas de mantenimiento del clúster, la actualización se considera correcta y se marca como completada. Esto puede ocurrir durante la ejecución inicial de la actualización o cualquiera de las ejecuciones posteriores de la actualización en esta fase. No hay ningún correo electrónico de confirmación de una ejecución correcta.

## <a name="custom-policies-for-manual-upgrades"></a>Directivas personalizadas para actualizaciones manuales

Puede especificar directivas personalizadas para actualizaciones manuales de clústeres. Estas directivas se aplican cada vez que se selecciona una nueva versión en tiempo de ejecución, lo que desencadena el sistema para iniciar la actualización del clúster. Si no reemplaza las directivas, se usarán los valores predeterminados. Para más información, consulte [Establecimiento de directivas personalizadas para actualizaciones manuales](service-fabric-cluster-upgrade-version-azure.md#custom-policies-for-manual-upgrades).

## <a name="other-cluster-updates"></a>Otras actualizaciones del clúster

Fuera de la actualización en tiempo de ejecución, hay una serie de acciones que es posible que tenga que realizar para mantener el clúster actualizado, incluidas las siguientes:

### <a name="managing-certificates"></a>Administrar certificados

Service Fabric usa los [certificados de servidor X.509](service-fabric-cluster-security.md) que especifica al crear un clúster para proteger las comunicaciones entre los nodos del clúster y autenticar a los clientes. Puede agregar, actualizar o eliminar certificados para el clúster y el cliente en [Azure Portal](https://portal.azure.com) o mediante la CLI de Azure o PowerShell.  Para obtener más información, lea [Agregar o quitar certificados](service-fabric-cluster-security-update-certs-azure.md)

### <a name="opening-application-ports"></a>Apertura de puertos de aplicación

Puede cambiar los puertos de aplicación cambiando las propiedades del recurso del equilibrador de carga asociado al tipo de nodo. Puede usar Azure Portal, PowerShell o el CLI de Azure. Para obtener más información, lea [Open application ports for a cluster](create-load-balancer-rule.md) (Abrir los puertos de aplicación para un clúster).

### <a name="defining-node-properties"></a>Definición de propiedades de nodo

A veces, es posible que quiera asegurarse de que ciertas cargas de trabajo solo se ejecuten en determinados tipos de nodos en el clúster. Por ejemplo, algunas cargas de trabajo pueden requerir GPU o SSD, al contrario que otras. Para cada uno de los tipos de nodo de un clúster, puede agregar propiedades de nodo personalizados a los nodos del clúster. Las restricciones de posición son las instrucciones adjuntas a los servicios individuales que se seleccionan para una o más propiedades de nodo. Las restricciones de posición definen dónde deben ejecutarse los servicios.

Para obtener más información acerca de las restricciones de selección de ubicación, de las propiedades del nodo y de cómo definirlas, lea [Restricciones de ubicación y propiedades de nodo](service-fabric-cluster-resource-manager-cluster-description.md#node-properties-and-placement-constraints).

### <a name="adding-capacity-metrics"></a>Incorporación de métricas de capacidad

Para cada uno de los tipos de nodo, puede agregar las métricas de capacidad personalizadas que desee usar en las aplicaciones para la carga de informes. Para obtener más información sobre el uso de las métricas de capacidad para notificar la carga, consulte los documentos del Administrador de recursos de clúster de Service Fabric que se describen en [Descripción de un clúster de Service Fabric](service-fabric-cluster-resource-manager-cluster-description.md) y sobre [métricas y cargas](service-fabric-cluster-resource-manager-metrics.md).

### <a name="customizing-settings-for-your-cluster"></a>Personalización de la configuración del clúster

Pueden personalizarse muchas opciones de configuración diferentes en un clúster, como el nivel de confiabilidad de las propiedades del nodo y del clúster. Para más información, lea [Service Fabric cluster fabric settings](service-fabric-cluster-fabric-settings.md) (Configuración de un clúster de Service Fabric).

### <a name="upgrading-os-images-for-cluster-nodes"></a>Actualización de imágenes del sistema operativo para nodos de clúster

Se recomienda habilitar las actualizaciones automáticas de imágenes del sistema operativo para los nodos de clúster de Service Fabric. Para ello, hay que seguir varios pasos y cumplir con los requisitos del clúster. Otra opción es utilizar la aplicación de orquestación de revisiones (POA), una aplicación de Service Fabric que automatiza la aplicación de revisiones del sistema operativo en un clúster de Service Fabric sin tiempo de inactividad. Para más información sobre estas opciones, consulte [Revisión del sistema operativo Windows en el clúster de Service Fabric](how-to-patch-cluster-nodes-windows.md).

## <a name="next-steps"></a>Pasos siguientes

* [Administración de actualizaciones de Service Fabric](service-fabric-cluster-upgrade-version-azure.md)
* [Personalización de la configuración de un clúster de Service Fabric](service-fabric-cluster-fabric-settings.md)
* [Escalar o reducir horizontalmente el clúster](service-fabric-cluster-scale-in-out.md)
* Obtenga información sobre [actualizaciones de aplicaciones](service-fabric-application-upgrade.md)
