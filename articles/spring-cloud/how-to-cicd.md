---
title: Automatización de implementaciones de aplicaciones en Azure Spring Cloud
description: Describe cómo usar la tarea de Azure Spring Cloud para Azure Pipelines.
author: bmitchell287
ms.service: spring-cloud
ms.topic: conceptual
ms.date: 05/12/2021
ms.author: brendm
ms.custom: devx-track-java
zone_pivot_groups: programming-languages-spring-cloud
ms.openlocfilehash: 2df1c8f0e091e553e045efb8aea5665d29bbc53f
ms.sourcegitcommit: 1ee13b62c094a550961498b7a52d0d9f0ae6d9c0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109839336"
---
# <a name="automate-application-deployments-to-azure-spring-cloud"></a>Automatización de implementaciones de aplicaciones en Azure Spring Cloud

Las herramientas de integración continua y entrega continua permiten implementar rápidamente las actualizaciones en las aplicaciones existentes con el mínimo esfuerzo y riesgo. Azure DevOps ayuda a organizar y controlar estos trabajos clave. 

En este artículo se muestra cómo usar la [tarea de Azure Spring Cloud para Azure Pipelines](/azure/devops/pipelines/tasks/deploy/azure-spring-cloud) para implementar aplicaciones.

## <a name="create-an-azure-resource-manager-service-connection"></a>Creación de una conexión de servicio de Azure Resource Manager

Lea [este artículo](/azure/devops/pipelines/library/connect-to-azure) para aprender a crear una conexión de servicio de Azure Resource Manager con su proyecto de Azure DevOps. Asegúrese de seleccionar la misma suscripción que usa para la instancia de servicio de Azure Spring Cloud.

## <a name="build-and-deploy-apps"></a>Compilación e implementación de aplicaciones

::: zone pivot="programming-language-csharp"
### <a name="deploy-artifacts"></a>Implementación de artefactos

Puede compilar e implementar los proyectos mediante una serie de tareas. Este fragmento de código define variables, una tarea de .NET Core para compilar la aplicación y una tarea de Azure Spring Cloud para implementar la aplicación.

```yaml
variables:
  workingDirectory: './steeltoe-sample'
  planetMainEntry: 'Microsoft.Azure.SpringCloud.Sample.PlanetWeatherProvider.dll'
  solarMainEntry: 'Microsoft.Azure.SpringCloud.Sample.SolarSystemWeather.dll'
  planetAppName: 'planet-weather-provider'
  solarAppName: 'solar-system-weather'
  serviceName: '<your service name>'


steps:
# Restore, build, publish and package the zipped planet app
- task: DotNetCoreCLI@2
  inputs:
    command: 'publish'
    publishWebProjects: false
    arguments: '--configuration Release'
    zipAfterPublish: false
    modifyOutputPath: false
    workingDirectory: $(workingDirectory)

# Deploy the planet app
- task: AzureSpringCloud@0
  inputs:
    azureSubscription: '<Service Connection Name>'
    Action: 'Deploy'
    AzureSpringCloud: $(serviceName)
    AppName: 'testapp'
    UseStagingDeployment: false
    DeploymentName: 'default'
    Package: $(workingDirectory)/src/$(planetAppName)/publish-deploy-planet.zip
    RuntimeVersion: 'NetCore_31'
    DotNetCoreMainEntryPath: $(planetMainEntry)

# Deploy the solar app
- task: AzureSpringCloud@0
  inputs:
    azureSubscription: '<Service Connection Name>'
    Action: 'Deploy'
    AzureSpringCloud: $(serviceName)
    AppName: 'testapp'
    UseStagingDeployment: false
    DeploymentName: 'default'
    Package: $(workingDirectory)/src/$(solarAppName)/publish-deploy-solar.zip
    RuntimeVersion: 'NetCore_31'
    DotNetCoreMainEntryPath: $(solarMainEntry)
```

::: zone-end
::: zone pivot="programming-language-java"
### <a name="deploy-artifacts"></a>Implementación de artefactos

#### <a name="to-production"></a>En producción

Puede compilar e implementar los proyectos mediante una serie de tareas. En primer lugar, este fragmento de código define una tarea de Maven para compilar la aplicación, seguida de una segunda tarea que implementa el archivo JAR mediante la tarea de Azure Spring Cloud para Azure Pipelines.

```yaml
steps:
- task: Maven@3
  inputs:
    mavenPomFile: 'pom.xml'
- task: AzureSpringCloud@0
  inputs:
    azureSubscription: '<your service connection name>'
    Action: 'Deploy'
    AzureSpringCloud: <your Azure Spring Cloud service>
    AppName: <app-name>
    UseStagingDeployment: false
    DeploymentName: 'default'
    Package: ./target/your-result-jar.jar
```

#### <a name="blue-green-deployments"></a>Implementaciones azul-verde

La implementación que se muestra en la sección anterior recibe inmediatamente el tráfico de la aplicación tras la implementación. A veces, los desarrolladores quieren probar sus aplicaciones en el entorno de producción, pero antes de que la aplicación reciba cualquier tráfico de cliente.

El fragmento de código siguiente crea la aplicación de la misma manera que la anterior y, a continuación, la implementa en una implementación de almacenamiento provisional. En este ejemplo, la implementación de almacenamiento provisional ya debe existir. Para obtener un enfoque alternativo, consulte [Estrategias de implementación azul-verde](concepts-blue-green-deployment-strategies.md).


```yaml
steps:
- task: Maven@3
  inputs:
    mavenPomFile: 'pom.xml'
- task: AzureSpringCloud@0
  inputs:
    azureSubscription: '<your service connection name>'
    Action: 'Deploy'
    AzureSpringCloud: <your Azure Spring Cloud service>
    AppName: <app-name>
    UseStagingDeployment: true
    Package: ./target/your-result-jar.jar
- task: AzureSpringCloud@0
  inputs:
    azureSubscription: '<your service connection name>'
    Action: 'Set Production'
    AzureSpringCloud: <your Azure Spring Cloud service>
    AppName: <app-name>
    UseStagingDeployment: true
```

### <a name="deploy-from-source"></a>Implementación desde el origen

Se puede implementar directamente en Azure sin un paso de compilación independiente.

```yaml
- task: AzureSpringCloud@0
  inputs:
    azureSubscription: '<your service connection name>'
    Action: 'Deploy'
    AzureSpringCloud: <your Azure Spring Cloud service>
    AppName: <app-name>
    UseStagingDeployment: false
    DeploymentName: 'default'
    Package: $(Build.SourcesDirectory)
```
::: zone-end

## <a name="next-steps"></a>Pasos siguientes

* [Inicio rápido: Implementación de la primera aplicación de Azure Spring Cloud](./quickstart.md)

