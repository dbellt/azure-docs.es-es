---
title: Uso de Application Change Analysis en Azure Monitor para detectar problemas de la aplicación web | Microsoft Docs
description: Use Application Change Analysis en Azure Monitor para solucionar problemas de aplicaciones en sitios activos de Azure App Service.
ms.topic: conceptual
author: cawams
ms.author: cawa
ms.date: 05/04/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 9366b268ca394228a63ecfd18ac6c2f4576ba21a
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108315694"
---
# <a name="use-application-change-analysis-preview-in-azure-monitor"></a>Uso de Application Change Analysis (versión preliminar) en Azure Monitor

Cuando se produce una interrupción o un problema en un sitio activo, determinar rápidamente la causa raíz es fundamental. Las soluciones de supervisión estándares pueden alertarlo sobre un problema. Incluso pueden indicarle qué componente está presentando el error. Sin embargo, esta alerta no siempre explica inmediatamente la causa del error. Sabrá que su sitio funcionaba hace cinco minutos y que ahora no funciona. ¿Qué cambió en los últimos cinco minutos? Esta es la pregunta que Application Change Analysis está diseñado para responder en Azure Monitor.

A partir de la eficacia de [Azure Resource Graph](../../governance/resource-graph/overview.md), Change Analysis proporciona conclusiones sobre los cambios de la aplicación de Azure para aumentar la observación y reducir el tiempo medio para reparación.

> [!IMPORTANT]
> Change Analysis se encuentra actualmente en versión preliminar. Esta versión preliminar se proporciona sin un contrato de nivel de servicio. Esta versión no se recomienda para usarse en cargas de trabajo. Es posible que algunas características no sean compatibles o que sus funcionalidades estén limitadas. Para obtener más información, consulte [Términos de uso complementarios de las Versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="overview"></a>Información general

Change Analysis detecta varios tipos de cambios, desde el nivel de infraestructura hasta la implementación de la aplicación. Es un proveedor de recursos de Azure de nivel de suscripción que comprueba los cambios de recursos en la suscripción. Change Analysis proporciona datos para que varias herramientas de diagnóstico ayuden a los usuarios a comprender qué cambios pueden haber causado los problemas.

El siguiente diagrama muestra la arquitectura de Change Analysis:

![Diagrama de la arquitectura del procedimiento mediante el que Change Analysis obtiene los datos de cambio y los envía a las herramientas de cliente](./media/change-analysis/overview.png)

## <a name="supported-resource-types"></a>Tipos de recurso admitidos

El servicio Application Change Analysis admite cambios en el nivel de propiedad de recursos en todos los tipos de recursos de Azure, incluidos los recursos comunes, como:
- Máquina virtual
- Conjunto de escalado de máquina virtual
- App Service
- Azure Kubernetes Service
- Función de Azure
- Recursos de redes: Grupo de seguridad de red, Virtual Network, Application Gateway, etc.
- Servicios de datos: Storage, SQL, Redis Cache, Cosmos DB, etc.

## <a name="data-sources"></a>Orígenes de datos

Application Change Analysis consulta las propiedades de las que Azure Resource Manager realiza un seguimiento, las configuraciones con proxy y los cambios en el invitado de la aplicación web. Además, el servicio hace un seguimiento de los cambios en las dependencias de los recursos para diagnosticar y supervisar una aplicación completa.

### <a name="azure-resource-manager-tracked-properties-changes"></a>Cambios en la propiedades controladas de Azure Resource Manager

Mediante [Azure Resource Graph](../../governance/resource-graph/overview.md), Change Analysis proporciona un registro histórico de la forma en que los recursos de Azure que hospedan a la aplicación han cambiado con el tiempo. Es posible detectar la configuración controlada, como las identidades administradas, la actualización del sistema operativo de la plataforma y los nombres de host.

### <a name="azure-resource-manager-proxied-setting-changes"></a>Cambios en la configuración con Proxy de Azure Resource Manager

Las opciones de configuración, como la regla de configuración de IP, configuración de TLS y versiones de extensión, todavía no están disponibles en Azure Resource Graph. Por lo tanto, las consultas de Change Analysis se calculan de forma segura para proporcionar más detalles sobre lo que ha cambiado en la aplicación.

### <a name="changes-in-web-app-deployment-and-configuration-in-guest-changes"></a>Cambios en la implementación y la configuración de aplicaciones web (cambios en invitado)

Change Analysis captura el estado de implementación y configuración de una aplicación cada 4 horas. Puede detectar, por ejemplo, los cambios en las variables de entorno de aplicación. La herramienta calcula las diferencias y presenta qué ha cambiado. A diferencia de los cambios de Resource Manager, es posible que la información sobre cambios en la implementación de código no esté disponible en la herramienta inmediatamente. Para ver los cambios más recientes en Change Analysis, seleccione **Actualizar**.

![Captura de pantalla del botón "Scan changes now"](./media/change-analysis/scan-changes.png)

Actualmente se admiten todos los archivos basados en texto en la raíz del sitio **wwwroot**, con las siguientes extensiones:
- *.json
- *.xml
- *.ini
- *.yml
- *.config
- *.properties
- *.html
- *.cshtml
- *.js
- requirements.txt
- Gemfile
- Gemfile.lock
- config.gemspec

### <a name="dependency-changes"></a>Cambios de dependencia

Los cambios realizados en las dependencias de recursos también pueden causar problemas en un recurso. Por ejemplo, si llama una aplicación web a una caché de Redis Cache, la SKU la dicha caché podría afectar al rendimiento de la aplicación de web. Otro ejemplo es si se cerró el puerto 22 en el grupo de seguridad de red de una máquina virtual, lo que provocará errores de conectividad.

#### <a name="web-app-diagnose-and-solve-problems-navigator-preview"></a>Navegador (versión preliminar) de diagnóstico y solución de problemas de una aplicación web

Para detectar los cambios en las dependencias, Change Analysis comprueba el registro DNS de la aplicación web. De este modo, identifica los cambios en todos los componentes de aplicación que podrían ocasionar problemas.
Actualmente, las siguientes dependencias se admiten en **Diagnóstico y solución de problemas de una aplicación web |Navegador (versión preliminar)** :

- Web Apps
- Azure Storage
- Azure SQL

#### <a name="related-resources"></a>Recursos relacionados

Application Change Analysis detecta recursos relacionados. Algunos ejemplos comunes son Grupo de seguridad de red, Virtual Network, Application Gateway y Load Balancer relacionados con una máquina virtual.
Normalmente, los recursos de red se aprovisionan automáticamente en el mismo grupo de recursos que los recursos que lo usan, por lo que el filtrado de los cambios por grupo de recursos mostrará todos los cambios de la máquina virtual y los recursos de red relacionados.

![Captura de pantalla de cambios de red](./media/change-analysis/network-changes.png)

## <a name="application-change-analysis-service-enablement"></a>Habilitación del servicio Application Change Analysis

El servicio Application Change Analysis calcula y agrega los datos modificados de los orígenes de datos mencionados anteriormente. Proporciona un conjunto de análisis para que los usuarios naveguen fácilmente por todos los cambios de los recursos y para identificar qué cambio es pertinente en el contexto de solución de problemas o de supervisión.
Es necesario registrar el proveedor de recursos "Microsoft. ChangeAnalysis" con una suscripción para que esté disponible la configuración de propiedades controladas y datos de cambio de configuración con Proxy de Azure Resource Manager. Al entrar en la herramienta Diagnosticar y solucionar problemas de Web Apps o abrir la hoja independiente de Change Analysis, este proveedor de recursos se registra automáticamente.
En el caso de cambios en el invitado de la aplicación web, se requiere una habilitación independiente para examinar los archivos de código dentro de una aplicación web. Para más información, consulte la sección [Change Analysis en la herramienta Diagnosticar y solucionar problemas](change-analysis-visualizations.md#application-change-analysis-in-the-diagnose-and-solve-problems-tool) que aparece más adelante en este artículo para obtener más detalles.

## <a name="cost"></a>Coste
Application Change Analysis es un servicio gratuito: no incurre en ningún costo de facturación para las suscripciones con el servicio habilitado. El servicio tampoco afecta al rendimiento de la exploración de los cambios de las propiedades de los recursos de Azure. Al habilitar Change Analysis para cambios de archivos invitados de aplicaciones web (o habilitar la herramienta Diagnosticar y resolver problemas), tendrá un impacto insignificante en el rendimiento de la aplicación web y ningún costo de facturación.


## <a name="enable-change-analysis-at-scale-for-web-app-in-guest-file-and-environment-variable-changes"></a>Habilitación de Change Analysis a escala para los cambios de variables de entorno y de archivo en la aplicación web

Si la suscripción incluye varias aplicaciones web, podría resultar ineficaz habilitar el servicio en el nivel de aplicación web. Ejecute el siguiente script para habilitar todas las aplicaciones web de su suscripción.

Requisitos previos:

- Módulo Az de PowerShell. Siga las instrucciones de [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps).

Ejecute el siguiente script:

```PowerShell
# Log in to your Azure subscription
Connect-AzAccount

# Get subscription Id
$SubscriptionId = Read-Host -Prompt 'Input your subscription Id'

# Make Feature Flag visible to the subscription
Set-AzContext -SubscriptionId $SubscriptionId

# Register resource provider
Register-AzResourceProvider -ProviderNamespace "Microsoft.ChangeAnalysis"

# Enable each web app
$webapp_list = Get-AzWebApp | Where-Object {$_.kind -eq 'app'}
foreach ($webapp in $webapp_list)
{
    $tags = $webapp.Tags
    $tags[“hidden-related:diagnostics/changeAnalysisScanEnabled”]=$true
    Set-AzResource -ResourceId $webapp.Id -Tag $tags -Force
}

```

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre las [visualizaciones de Change Analysis](change-analysis-visualizations.md).
- Aprenda a [solucionar problemas de Change Analysis](change-analysis-troubleshoot.md).
- Habilitar Application Insights para [aplicaciones de Azure App Services](azure-web-apps.md).
- Habilitar Application Insights para [aplicaciones hospedadas en IIS en máquina virtual de Azure y conjunto de escalado de máquinas virtuales de Azure](azure-vm-vmss-apps.md).
