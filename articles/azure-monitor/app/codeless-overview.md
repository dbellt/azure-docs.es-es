---
title: 'Supervisión de las aplicaciones sin cambios de código: instrumentación automática para Azure Monitor Application Insights | Microsoft Docs'
description: 'Información general de la instrumentación automática para Azure Monitor Application Insights: administración del rendimiento de aplicaciones sin código'
ms.topic: conceptual
author: MS-jgol
ms.author: jgol
ms.date: 05/31/2020
ms.reviewer: mbullwin
ms.openlocfilehash: df6271f8c036d708b93d7312076f3eee585cfcba
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108287291"
---
# <a name="what-is-auto-instrumentation-or-codeless-attach---azure-monitor-application-insights"></a>¿Qué es la instrumentación automática o la conexión sin código de Azure Monitor Application Insights?

La instrumentación automática, o conexión sin código, permite habilitar la supervisión de las aplicaciones con Application Insights sin modificar el código.  

Application Insights se integra con varios proveedores de recursos y funciona en distintos entornos. En resumen, solo debe habilitar y, en algunos casos, configurar el agente, que recopilará la telemetría automáticamente. En unos instantes, podrá ver las métricas, los datos y las dependencias en el recurso de Application Insights, lo que le permitirá detectar el origen de los posibles problemas antes de que se produzcan y analizar la causa raíz a través de la vista de transacciones integral.

## <a name="supported-environments-languages-and-resource-providers"></a>Entornos, lenguajes y proveedores de recursos admitidos

A medida que agregamos integraciones adicionales, la matriz de funcionalidades de instrumentación automática se vuelve compleja. En la tabla siguiente se muestra el estado actual de la situación en lo que se refiere a la compatibilidad con varios proveedores de recursos, lenguajes y entornos.

|Entorno/proveedor de recursos          | .NET            | .NET Core       | Java            | Node.js         | Python          |
|---------------------------------------|-----------------|-----------------|-----------------|-----------------|-----------------|
|Azure App Service en Windows           | Disponibilidad general, OnBD*       | Disponibilidad general, opcional      | En curso     | En curso     | No compatible   |
|Azure App Service en Linux             | N/D             | No compatible   | En curso     | Vista previa pública  | No compatible   |
|Azure Functions: básico                | Disponibilidad general, OnBD*       | Disponibilidad general, OnBD*       | Disponibilidad general, OnBD*       | Disponibilidad general, OnBD*       | Disponibilidad general, OnBD*       |
|Azure Functions en Windows: dependencias | No compatible   | No compatible   | Vista previa pública  | No compatible   | No compatible   |
|Azure Kubernetes Service               | N/D             | En diseño       | Mediante el agente   | En diseño       | No compatible   |
|VM de Azure con Windows                      | Vista previa pública  | No compatible   | No compatible   | No compatible   | No compatible   |
|VM locales con Windows                | Disponibilidad general, opcional      | No compatible   | Mediante el agente   | No compatible   | No compatible   |
|Agente independiente: cualquier ent.            | No compatible   | No compatible   | Disponibilidad general              | No compatible   | No compatible   |

*OnBD significa "Activado de forma predeterminada"; Application Insights se habilitará automáticamente una vez que implemente la aplicación en los entornos compatibles. 

## <a name="azure-app-service"></a>Azure App Service

### <a name="windows"></a>Windows

#### <a name="net"></a>.NET
La supervisión de aplicaciones de Azure App Service en Windows está disponible para las [aplicaciones .NET](./azure-web-apps.md?tabs=net) y está habilitada de forma predeterminada.

#### <a name="netcore"></a>.NETCore
La supervisión de las [aplicaciones .NETCore](./azure-web-apps.md?tabs=netcore) se puede habilitar con un solo clic.

#### <a name="java"></a>Java
La integración del portal para la supervisión de aplicaciones Java de App Service en Windows no está disponible actualmente. Sin embargo, puede agregar el [agente independiente de Java 3.0](./java-in-process-agent.md) de Application Insights a la aplicación sin realizar ningún cambio de código antes de implementar las aplicaciones en App Service. El agente de Java 3.0 de Application Insights se encuentra disponible con carácter general.

#### <a name="nodejs"></a>Node.js
Actualmente no se puede habilitar la supervisión de aplicaciones Node.js en Windows desde el portal. Para supervisar aplicaciones Node.js, use el [SDK](./nodejs.md).

### <a name="linux"></a>Linux

#### <a name="netcore"></a>.NETCore
Para supervisar las aplicaciones .NETCore que se ejecutan en Linux, use el [SDK](./asp-net-core.md).

#### <a name="java"></a>Java 
La habilitación de la supervisión de aplicaciones Java para App Service en Linux desde el portal no está disponible, pero puede agregar el [agente de Java 3.0 de Application Insights](./java-in-process-agent.md) a la aplicación antes de implementar las aplicaciones en App Service. El agente de Java 3.0 de Application Insights se encuentra disponible con carácter general.

#### <a name="nodejs"></a>Node.js
La [supervisión de las aplicaciones Node.js en App Service en Linux](./azure-web-apps.md?tabs=nodejs) se encuentra en versión preliminar pública y se puede habilitar en Azure Portal, y está disponible en todas las regiones. 

#### <a name="python"></a>Python
Use el SDK para [supervisar la aplicación Python](./opencensus-python.md). 

## <a name="azure-functions"></a>Azure Functions

La supervisión básica de Azure Functions está habilitada de forma predeterminada para recopilar el registro, el rendimiento, los datos de error y las solicitudes HTTP. En el caso de las aplicaciones Java, puede habilitar una supervisión más completa con seguimiento distribuido y obtener los detalles de la transacción de un extremo a otro. Esta funcionalidad de Java está en versión preliminar pública y puede [habilitarla en Azure Portal](./monitor-functions.md).

## <a name="azure-kubernetes-service"></a>Azure Kubernetes Service

La instrumentación sin código de Azure Kubernetes Service está disponible actualmente para aplicaciones Java a través del [agente independiente](./java-in-process-agent.md). 

## <a name="azure-windows-vms-and-virtual-machine-scale-set"></a>Conjunto de escalado de máquinas virtuales y máquinas virtuales de Azure con Windows

La instrumentación automática para máquinas virtuales de Azure y el conjunto de escalado de máquinas virtuales está disponible para [.NET](./azure-vm-vmss-apps.md) y [Java](./java-in-process-agent.md).  

## <a name="on-premises-servers"></a>Servidores locales
Puede habilitar fácilmente la supervisión de sus [servidores de Windows locales para las aplicaciones .NET](./status-monitor-v2-overview.md) y para las [aplicaciones Java](./java-in-process-agent.md).

## <a name="other-environments"></a>Otros entornos
El versátil agente independiente de Java funciona en cualquier entorno, por lo que no es necesario instrumentar el código. [Siga esta guía](./java-in-process-agent.md) para habilitar Application Insights y obtener información sobre las increíbles funcionalidades del agente de Java. El agente se encuentra en versión preliminar pública y está disponible en todas las regiones. 

## <a name="next-steps"></a>Pasos siguientes

* [Información general de Application Insights](./app-insights-overview.md)
* [Mapa de aplicación](./app-map.md)
* [Supervisión del rendimiento de un extremo a otro](../app/tutorial-performance.md)
