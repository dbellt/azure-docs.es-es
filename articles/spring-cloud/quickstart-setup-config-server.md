---
title: 'Inicio rápido: configuración del servidor de configuración de Azure Spring Cloud'
description: Describe la configuración de Azure Spring Cloud Config Server para la implementación de aplicaciones.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: quickstart
ms.date: 09/08/2020
ms.custom: devx-track-java, fasttrack-edit
zone_pivot_groups: programming-languages-spring-cloud
ms.openlocfilehash: 8c71e76213011beaa63deeaadfa3d6d2dc0d4ce2
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108286550"
---
# <a name="quickstart-set-up-azure-spring-cloud-configuration-server"></a>Inicio rápido: configuración del servidor de configuración de Azure Spring Cloud

El servidor de configuración de Azure Spring Cloud es un servicio de configuración centralizado para sistemas distribuidos. Usa una capa de repositorio conectable que actualmente admite el almacenamiento local, Git y Subversion. En esta guía de inicio rápido, configurará el servidor de configuración para obtener datos de un repositorio de Git.

::: zone pivot="programming-language-csharp"

## <a name="prerequisites"></a>Requisitos previos

* Complete la guía de inicio rápido anterior de esta serie: [Aprovisionamiento del servicio Azure Spring Cloud](./quickstart-provision-service-instance.md).

## <a name="azure-spring-cloud-config-server-procedures"></a>Procedimientos de Azure Spring Cloud Config Server

Ejecute el comando siguiente para configurar el servidor de configuración con la ubicación del repositorio de Git del proyecto. Reemplace `<service instance name>` por el nombre del servicio que creó anteriormente. El valor predeterminado para el nombre de instancia de servicio que estableció en la guía de inicio rápido anterior no funciona con este comando.

```azurecli
az spring-cloud config-server git set -n <service instance name> --uri https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples --search-paths steeltoe-sample/config
```

Este comando indica al servidor de configuración que busque los datos de configuración en la carpeta [steeltoe-sample/config](https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples/tree/master/steeltoe-sample/config) del repositorio de la aplicación de ejemplo. Dado que el nombre de la aplicación que va a obtener los datos de configuración es `planet-weather-provider`, el archivo que se usará es [planet-weather-provider.yml](https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples/blob/master/steeltoe-sample/config/planet-weather-provider.yml).

::: zone-end

::: zone pivot="programming-language-java"
Azure Spring Cloud Config Server es un servicio de configuración centralizado para sistemas distribuidos. Usa una capa de repositorio conectable que actualmente admite el almacenamiento local, Git y Subversion.  Configure el servidor de configuración para implementar aplicaciones de microservicios en Azure Spring Cloud.

## <a name="prerequisites"></a>Requisitos previos

* [Instalación de JDK 8](/java/azure/jdk/)
* [Registro para obtener una suscripción a Azure](https://azure.microsoft.com/free/)
* (Opcional) [Instale la versión 2.0.67 de la CLI de Azure, o cualquier versión superior](/cli/azure/install-azure-cli), e instale la extensión de Azure Spring Cloud con el comando `az extension add --name spring-cloud`
* (Opcional) [Instale Azure Toolkit for IntelliJ](https://plugins.jetbrains.com/plugin/8053-azure-toolkit-for-intellij/) e [inicie sesión](/azure/developer/java/toolkit-for-intellij/create-hello-world-web-app#installation-and-sign-in)

## <a name="azure-spring-cloud-config-server-procedures"></a>Procedimientos de Azure Spring Cloud Config Server

#### <a name="portal"></a>[Portal](#tab/Azure-portal)

En el procedimiento siguiente se configura el servidor de configuración mediante Azure Portal para implementar el [ejemplo de PetClinic](https://github.com/azure-samples/spring-petclinic-microservices).

1. Vaya a la página **Información general** del servicio y seleccione **Config Server**.

2. En la sección **Default repository** (Repositorio predeterminado), en **URI** seleccione "https://github.com/azure-samples/spring-petclinic-microservices-config".

3. Haga clic en **Validar**.

    ![Ir al servidor de configuración](media/spring-cloud-quickstart-launch-app-portal/portal-config.png)

4. Cuando finalice la validación, haga clic en **Aplicar** para guardar los cambios.

    ![Validación del servidor de configuración](media/spring-cloud-quickstart-launch-app-portal/validate-complete.png)

5. La actualización de la configuración puede tardar unos minutos.
 
    ![Actualización del servidor de configuración](media/spring-cloud-quickstart-launch-app-portal/updating-config.png) 

6. Cuando se haya completado la configuración, debería recibir una notificación.

#### <a name="cli"></a>[CLI](#tab/Azure-CLI)


En el procedimiento siguiente se usa la CLI de Azure para configurar el servidor de configuración para que implemente el [ejemplo de PetClinic](https://github.com/azure-samples/spring-petclinic-microservices).

Ejecute el siguiente comando para establecer el repositorio predeterminado.

```azurecli

az spring-cloud config-server git set -n <service instance name> --uri https://github.com/azure-samples/spring-petclinic-microservices-config
::: zone-end

> [!TIP]
> If you are using a private repository for config server, please refer to our [tutorial on setting up authentication](./how-to-config-server.md).

## Troubleshooting of Azure Spring Cloud config server

The following procedure explains how to troubleshoot config server settings.

1. In the Azure portal, go to the service **Overview** page and select **Logs**. 
1. Select **Queries** and **Show the application logs that contain the "error" or "exception" terms"**. 
1. Click **Run**. 
1. If you find the error **java.lang.illegalStateException** in logs, this indicates that spring cloud service cannot locate properties from config server.

    [ ![ASC portal run query](media/spring-cloud-quickstart-setup-config-server/setup-config-server-query.png) ](media/spring-cloud-quickstart-setup-config-server/setup-config-server-query.png)

1. Go to the service **Overview** page.
1. Select **Diagnose and solve problems**. 
1. Select **Config Server** detector.

    [ ![ASC portal diagnose problems](media/spring-cloud-quickstart-setup-config-server/setup-config-server-diagnose.png) ](media/spring-cloud-quickstart-setup-config-server/setup-config-server-diagnose.png)

3. Click **Config Server Health Check**.

    [ ![ASC portal genie](media/spring-cloud-quickstart-setup-config-server/setup-config-server-genie.png) ](media/spring-cloud-quickstart-setup-config-server/setup-config-server-genie.png)

4. Click **Config Server Status** to see more details from the detector.

    [ ![ASC portal health status](media/spring-cloud-quickstart-setup-config-server/setup-config-server-health-status.png) ](media/spring-cloud-quickstart-setup-config-server/setup-config-server-health-status.png)

## Next steps

In this quickstart, you created Azure resources that will continue to accrue charges if they remain in your subscription. If you don't intend to continue on to the next quickstart, see [Clean up resources](./quickstart-logs-metrics-tracing.md#clean-up-resources). Otherwise, advance to the next quickstart:

> [!div class="nextstepaction"]
> [Build and deploy apps](./quickstart-deploy-apps.md)
