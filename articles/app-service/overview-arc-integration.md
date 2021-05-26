---
title: App Service en Azure Arc
description: Introducción a la integración de App Service con Azure Arc para operadores de Azure.
ms.topic: article
ms.date: 05/03/2021
ms.openlocfilehash: 8119d983a891ac6a671920c745d6395f4418ce24
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110386423"
---
# <a name="app-service-functions-and-logic-apps-on-azure-arc-preview"></a>App Service, Functions y Logic Apps en Azure Arc (versión preliminar)

Puede ejecutar App Service, Functions y Logic Apps en un clúster de Kubernetes habilitado para Azure Arc. El clúster de Kubernetes puede ser local o estar hospedado en una nube de terceros. Este enfoque permite a los desarrolladores de aplicaciones aprovechar las características de App Service. Al mismo tiempo, permite a los administradores de TI mantener el cumplimiento corporativo hospedando las aplicaciones de App Service en la infraestructura interna. También permite que otros operadores de TI protejan sus inversiones anteriores en otros proveedores de nube mediante la ejecución de App Service en clústeres de Kubernetes existentes.

> [!NOTE]
> Para obtener información sobre cómo configurar el clúster de Kubernetes para App Service, Functions y Logic Apps, consulte [Creación de un entorno de Kubernetes de App Service (versión preliminar)](manage-create-arc-environment.md).

En la mayoría de los casos, los desarrolladores de aplicaciones no necesitan saber nada más que cómo implementar en la región correcta de Azure que representa el entorno de Kubernetes implementado. Para los operadores que proporcionan el entorno y mantienen la infraestructura subyacente de Kubernetes, debe tener en cuenta los siguientes recursos de Azure:

- El clúster conectado, que es una proyección en Azure de la infraestructura de Kubernetes. Para más información, consulte [¿Qué es Kubernetes habilitado para Azure Arc?](../azure-arc/kubernetes/overview.md).
- Una extensión de clúster, que es un subrecurso del recurso de clúster conectado. La extensión de App Service [instala los pods necesarios en el clúster conectado](#pods-created-by-the-app-service-extension). Para más información sobre las extensiones de clúster, consulte [Extensiones de clúster en Kubernetes habilitado para Azure Arc](../azure-arc/kubernetes/conceptual-extensions.md).
- Una ubicación personalizada, que agrupa un grupo de extensiones y las asigna a un espacio de nombres para los recursos creados. Para más información, consulte [Ubicaciones personalizadas basadas en Kubernetes habilitado para Azure Arc](../azure-arc/kubernetes/conceptual-custom-locations.md).
- Un entorno de Kubernetes de App Service, que permite una configuración común entre aplicaciones pero no relacionada con las operaciones del clúster. Conceptualmente, se implementa en el recurso de ubicación personalizada y los desarrolladores de aplicaciones crean aplicaciones en este entorno. Esto se describe con más detalle en [Entorno de Kubernetes de App Service](#app-service-kubernetes-environment).

## <a name="public-preview-limitations"></a>Limitaciones de la vista preliminar pública

Las siguientes limitaciones de la versión preliminar pública se aplican los entornos de Kubernetes de App Service. Se actualizarán cuando se validen distribuciones adicionales y se admitan más regiones.

| Limitación                                              | Detalles                                                                          |
|---------------------------------------------------------|----------------------------------------------------------------------------------|
| Regiones de Azure compatibles                                 | Este de EE. UU., Oeste de Europa                                                             |
| Distribuciones de Kubernetes validadas                      | Azure Kubernetes Service                                                         |
| Característica: redes                                     | [No disponible (se basa en las redes del clúster)](#are-networking-features-supported) |
| Característica: identidades administradas                             | [No disponible](#are-managed-identities-supported)                               |
| Característica: referencias de Key Vault                           | No disponible (depende de las identidades administradas)                                    |
| Característica: extracción de imágenes de ACR con identidad administrada     | No disponible (depende de las identidades administradas)                                    |
| Característica: edición en el portal para Functions y Logic Apps | No disponible                                                                    |
| Característica: publicación FTP                                 | No disponible                                                                    |
| Registros                                                    | Log Analytics se debe configurar con la extensión de clúster; no por cada sitio.            |

## <a name="pods-created-by-the-app-service-extension"></a>Pods creados por la extensión de App Service

Cuando la extensión de App Service se instala en el clúster de Kubernetes habilitado para Arc, verá varios pods creados en el espacio de nombres de versión que se especificó. Estos pods permiten que el clúster de Kubernetes sea una extensión del proveedor de recursos `Microsoft.Web` en Azure y ayudan en la administración y el funcionamiento de las aplicaciones. Opcionalmente, puede optar por que la extensión instale [KEDA](https://keda.sh/) para el escalado controlado por eventos.
 <!-- You can only have one installation of KEDA on the cluster. If you have one already, you must disable this behavior during installation of the cluster extension `TODO`. -->

En la tabla siguiente se describe el rol de cada pod que se crea de manera predeterminada:

| Pod                                   | Descripción                                                                                                                       |
|---------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| `<extensionName>-k8se-app-controller` | Pod de operador principal que crea recursos en el clúster y mantiene el estado de los componentes.                                |
| `<extensionName>-k8se-envoy`          | Una capa de proxy de front-end para todas las solicitudes del plano de datos. Enruta el tráfico entrante a las aplicaciones correctas.                           |
| `<extensionName>-k8se-activator`      | Un destino de enrutamiento alternativo para ayudar con las aplicaciones que se han escalado a cero mientras el sistema obtiene la primera instancia disponible. |
| `<extensionName>-k8se-build-service`  | Admite operaciones de implementación y ofrece la [característica Herramientas avanzadas](resources-kudu.md).                                        |
| `<extensionName>-k8se-http-scaler`    | Supervisa el volumen de solicitudes entrantes para proporcionar información de escalado a [KEDA](https://keda.sh).                               |
| `<extensionName>-k8se-img-cacher`     | Extrae imágenes de marcador de posición y de aplicación en una caché local en el nodo.                                                                  |
| `<extensionName>-k8se-log-processor`  | Recopila registros de las aplicaciones y otros componentes y los envía a Log Analytics.                                                      |
| `placeholder-azure-functions-*`       | Se usa para acelerar los arranques en frío para Azure Functions.                                                                                 |

## <a name="app-service-kubernetes-environment"></a>Entorno de Kubernetes de App Service

El recurso de entorno de Kubernetes de App Service es necesario antes de que se puedan crear aplicaciones. Habilita la configuración común de las aplicaciones de la ubicación personalizada, como el sufijo DNS predeterminado.

Solo se puede crear un recurso de entorno de Kubernetes en una ubicación personalizada. En la mayoría de los casos, un desarrollador que crea e implementa aplicaciones no necesita conocer directamente el recurso. Se puede deducir directamente del identificador de ubicación personalizada proporcionado. Sin embargo, al definir plantillas de Azure Resource Manager, cualquier recurso de plan debe hacer referencia directamente al identificador de recurso del entorno. Los valores de ubicación personalizada del plan y el entorno especificado deben coincidir.

## <a name="faq-for-app-service-functions-and-logic-apps-on-azure-arc-preview"></a>Preguntas frecuentes sobre App Service, Functions y Logic Apps en Azure Arc (versión preliminar)

- [¿Cuánto cuesta?](#how-much-does-it-cost)
- [¿Se admiten aplicaciones windows y Linux?](#are-both-windows-and-linux-apps-supported)
- [¿Qué pilas de aplicaciones integradas se admiten?](#which-built-in-application-stacks-are-supported)
- [¿Se admiten todos los tipos de implementación de aplicaciones?](#are-all-app-deployment-types-supported)
- [¿Qué características de App Service se admiten?](#which-app-service-features-are-supported)
- [¿Se admiten las características de redes?](#are-networking-features-supported)
- [¿Se admiten identidades administradas?](#are-managed-identities-supported)
- [¿Qué registros se recopilan?](#what-logs-are-collected)

### <a name="how-much-does-it-cost"></a>¿Cuánto cuesta?

App Service en Azure Arc es gratis durante la versión preliminar pública.

### <a name="are-both-windows-and-linux-apps-supported"></a>¿Se admiten aplicaciones windows y Linux?

Solo se admiten aplicaciones basadas en Linux, tanto de código como de contenedores personalizados. No se admiten aplicaciones de Windows.

### <a name="which-built-in-application-stacks-are-supported"></a>¿Qué pilas de aplicaciones integradas se admiten?

Se admiten todas las pilas de Linux integradas.

### <a name="are-all-app-deployment-types-supported"></a>¿Se admiten todos los tipos de implementación de aplicaciones?

No se admite la implementación mediante FTP. Actualmente tampoco se admite `az webapp up`. Se admiten otros métodos de implementación, como Git, ZIP, CI/CD, Visual Studio y Visual Studio Code.

### <a name="which-app-service-features-are-supported"></a>¿Qué características de App Service se admiten?

Durante el período de versión preliminar, se están validando determinadas características de App Service. Cuando se admitan, se activarán sus opciones en la barra de navegación izquierda de Azure Portal. Las características que aún no se admiten permanecen atenuadas.

### <a name="are-networking-features-supported"></a>¿Se admiten las características de redes?

No. No se admiten características de redes, como las conexiones híbridas, la integración de red virtual o las restricciones de dirección IP. Las redes se deben controlar directamente en las reglas de red del propio clúster de Kubernetes.

### <a name="are-managed-identities-supported"></a>¿Se admiten identidades administradas?

No. No se pueden asignar identidades administradas a las aplicaciones cuando se ejecutan en Azure Arc. Si la aplicación necesita una identidad para trabajar con otro recurso de Azure, considere la posibilidad de usar una [entidad de servicio de aplicación](../active-directory/develop/app-objects-and-service-principals.md#service-principal-object) en su lugar.

### <a name="what-logs-are-collected"></a>¿Qué registros se recopilan?

Los registros de los componentes del sistema y las aplicaciones se escriben en la salida estándar. Ambos tipos de registro se pueden recopilar para su análisis mediante las herramientas estándar de Kubernetes. También puede configurar la extensión de clúster de App Service con un [área de trabajo de Log Analytics](../azure-monitor/logs/log-analytics-overview.md) y se enviarán todos los registros a esa área de trabajo.

De manera predeterminada, los registros de los componentes del sistema se envían al equipo de Azure. Los registros de aplicación no se envían. Para evitar que estos registros se transfieran, establezca `logProcessor.enabled=false` como un valor de configuración de extensión. Esto también deshabilitará el reenvío de la aplicación al área de trabajo de Log Analytics. Deshabilitar el procesador de registro puede afectar al tiempo necesario para cualquier caso de soporte técnico y se le pedirá que recopile registros de la salida estándar mediante otros medios.

## <a name="next-steps"></a>Pasos siguientes

[Creación de un entorno de Kubernetes de App Service (versión preliminar)](manage-create-arc-environment.md)