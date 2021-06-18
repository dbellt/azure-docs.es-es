---
title: Configuración de un entorno de ensayo en Azure Spring Cloud | Microsoft Docs
description: Aprenda a usar la implementación blue-green con Azure Spring Cloud
author: MikeDodaro
ms.service: spring-cloud
ms.topic: conceptual
ms.date: 01/14/2021
ms.author: brendm
ms.custom: devx-track-java, devx-track-azurecli
ms.openlocfilehash: 9f8f09b61998c0b9c2d46291e4559741beac8acc
ms.sourcegitcommit: a434cfeee5f4ed01d6df897d01e569e213ad1e6f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111812498"
---
# <a name="set-up-a-staging-environment-in-azure-spring-cloud"></a>Configuración de un entorno de ensayo en Azure Spring Cloud

**Este artículo se aplica a:** ✔️ Java

En este artículo se explica cómo configurar una implementación de ensayo con el patrón de implementación azul-verde en Azure Spring Cloud. La implementación azul-verde es un patrón de entrega continua de Azure DevOps que se basa en mantener una versión existente (azul) activa mientras se implementa una nueva (verde). En este artículo se muestra cómo llevar la implementación de ensayo a producción sin cambiar la implementación de producción.

## <a name="prerequisites"></a>Requisitos previos

* Instancia de Azure Spring Cloud con un plan de tarifa Estándar
* [Extensión de Azure Spring Cloud](/cli/azure/azure-cli-extensions-overview) para la CLI de Azure

En este artículo se usa una aplicación compilada a partir de Spring Initializr. Si quiere usar otra aplicación en este ejemplo, deberá realizar un sencillo cambio en una parte pública de la aplicación para diferenciar la implementación de ensayo de la de producción.

>[!TIP]
> [Azure Cloud Shell](https://shell.azure.com) es un shell interactivo gratuito que puede usar para ejecutar las instrucciones de este artículo.  Incluye herramientas comunes de Azure preinstaladas, entre las que se incluyen las versiones más recientes de Git, JDK, Maven y la CLI de Azure. Si ha iniciado sesión en su suscripción a Azure, inicie la instancia de Cloud Shell. Para obtener más información, consulte [Información general de Azure Cloud Shell](../cloud-shell/overview.md).

Para configurar la implementación azul-verde en Azure Spring Cloud, siga las instrucciones de las secciones siguientes.

## <a name="install-the-azure-cli-extension"></a>Instalación de la extensión de la CLI de Azure

Para instalar la extensión de Azure Spring Cloud para la CLI de Azure, use el siguiente comando:

```azurecli
az extension add --name spring-cloud
```
## <a name="prepare-the-app-and-deployments"></a>Preparación de la aplicación y las implementaciones
Para compilar la aplicación, siga estos pasos:

1. Genere el código de la aplicación de ejemplo mediante Spring Initializr con [esta configuración](https://start.spring.io/#!type=maven-project&language=java&platformVersion=2.3.4.RELEASE&packaging=jar&jvmVersion=1.8&groupId=com.example&artifactId=hellospring&name=hellospring&description=Demo%20project%20for%20Spring%20Boot&packageName=com.example.hellospring&dependencies=web,cloud-eureka,actuator,cloud-starter-sleuth,cloud-starter-zipkin,cloud-config-client).

2. Descargue el código.
3. Agregue el siguiente archivo de código fuente HelloController.java a la carpeta `\src\main\java\com\example\hellospring\`:

   ```java
   package com.example.hellospring; 
   import org.springframework.web.bind.annotation.RestController; 
   import org.springframework.web.bind.annotation.RequestMapping; 

   @RestController 

   public class HelloController { 

   @RequestMapping("/") 

     public String index() { 

         return "Greetings from Azure Spring Cloud!"; 
     } 

   } 
   ```
4. Compile el archivo .jar:

   ```azurecli
   mvn clean package -DskipTests
   ```
5. Cree la aplicación en la instancia de Azure Spring Cloud:

   ```azurecli
   az spring-cloud app create -n demo -g <resourceGroup> -s <Azure Spring Cloud instance> --assign-endpoint
   ```
6. Implemente la aplicación en Azure Spring Cloud:

   ```azurecli
   az spring-cloud app deploy -n demo -g <resourceGroup> -s <Azure Spring Cloud instance> --jar-path target\hellospring-0.0.1-SNAPSHOT.jar
   ```
7. Modifique el código para la implementación de ensayo:

   ```java
   package com.example.hellospring; 
   import org.springframework.web.bind.annotation.RestController; 
   import org.springframework.web.bind.annotation.RequestMapping; 

   @RestController 

   public class HelloController { 

   @RequestMapping("/") 

     public String index() { 

         return "Greetings from Azure Spring Cloud! THIS IS THE GREEN DEPLOYMENT"; 
     } 

   } 
   ```
8. Vuelva a compilar el archivo .jar:

   ```azurecli
   mvn clean packge -DskipTests
   ```
9. Cree la implementación verde: 

   ```azurecli
   az spring-cloud app deployment create -n green --app demo -g <resourceGroup> -s <Azure Spring Cloud instance> --jar-path target\hellospring-0.0.1-SNAPSHOT.jar 
   ```

## <a name="view-apps-and-deployments"></a>Visualización de aplicaciones e implementaciones

Vea las aplicaciones implementadas siguiendo los procedimientos siguientes:

1. Vaya a la instancia de Azure Spring Cloud en Azure Portal.

1. Desde el panel izquierdo, abra el panel **Aplicaciones** para ver las aplicaciones de la instancia de servicio.

   ![Captura de pantalla del panel para abrir aplicaciones.](media/spring-cloud-blue-green-staging/app-dashboard.png)

1. Puede seleccionar una aplicación y ver los detalles.

   ![Captura de pantalla de los detalles de una aplicación.](media/spring-cloud-blue-green-staging/app-overview.png)

1. Abra **Implementaciones** para ver todas las implementaciones de la aplicación. La cuadrícula muestra las implementaciones de producción y de ensayo.

   ![Captura de pantalla que muestra las implementaciones enumeradas de la aplicación.](media/spring-cloud-blue-green-staging/deployments-dashboard.png)

1. Seleccione la dirección URL para abrir la aplicación implementada actualmente.
    
   ![Captura de pantalla que muestra la URL de la aplicación implementada.](media/spring-cloud-blue-green-staging/running-blue-app.png)

1. Seleccione **Producción** en la columna **Estado** para ver la aplicación predeterminada.
    
   ![Captura de pantalla que muestra la URL de la aplicación predeterminada.](media/spring-cloud-blue-green-staging/running-default-app.png)

1. Seleccione **Ensayo** en la columna **Estado** para ver la aplicación de ensayo.
    
   ![Captura de pantalla que muestra la URL de la aplicación de ensayo.](media/spring-cloud-blue-green-staging/running-staging-app.png)

>[!TIP]
> * Confirme que el punto de conexión de prueba termina con una barra diagonal (/), con el fin de asegurarse de que el archivo CSS se ha cargado correctamente.  
> * Si el explorador requiere que escriba las credenciales de inicio de sesión para ver la página, use [Descodificar como dirección URL ](https://www.urldecoder.org/) para descodificar el punto de conexión de prueba. La descodificación de URL devuelve una dirección URL con el formato *https://\<username>:\<password>@\<cluster-name>.test.azureapps.io/gateway/green*. Use este formato para acceder al punto de conexión.

>[!NOTE]    
> Los valores del servidor de configuración se aplican tanto al entorno de ensayo como al de producción. Por ejemplo, si establece la ruta de acceso del contexto (*server.servlet.context-path*) para la puerta de enlace de aplicaciones en el servidor de configuración como *somepath*, la ruta de acceso a la implementación green cambia a *https://\<username>:\<password>@\<cluster-name>.test.azureapps.io/gateway/green/somepath/...* .
 
Si va a la puerta de enlace de aplicaciones pública en este momento, debería ver la página anterior sin el nuevo cambio.

## <a name="set-the-green-deployment-as-the-production-environment"></a>Establecimiento de la implementación green como entorno de producción

1. Una vez que haya verificado el cambio en el entorno de ensayo, puede enviarlo a producción. En la página **Aplicaciones** > **Implementaciones**, seleccione la aplicación que se encuentra en **Producción**.

1. Seleccione los puntos suspensivos que hay al lado de **Estado de registro** de la implementación verde y, a continuación, seleccione **Set as production** (Establecer como producción). 

   ![Captura de pantalla que muestra las selecciones para establecer la compilación de ensayo en producción.](media/spring-cloud-blue-green-staging/set-staging-deployment.png)

1. Confirme que la dirección URL de la aplicación muestra los cambios.

   ![Captura de pantalla que muestra la URL de la aplicación ahora en producción.](media/spring-cloud-blue-green-staging/new-production-deployment.png)

>[!NOTE]
> Después de haber establecido la implementación green como entorno de producción, la implementación anterior se convierte en la implementación de ensayo.

## <a name="modify-the-staging-deployment"></a>Modificación de la implementación de ensayo

Si no está satisfecho con el cambio, puede modificar el código de la aplicación, compilar un nuevo paquete .jar y cargarlo en la implementación verde mediante la CLI de Azure:

```azurecli
az spring-cloud app deploy  -g <resource-group-name> -s <service-instance-name> -n gateway -d green --jar-path gateway.jar
```

## <a name="delete-the-staging-deployment"></a>Eliminación de la implementación de ensayo

Para eliminar la implementación de ensayo desde Azure Portal, vaya a la página de la implementación de ensayo y seleccione el botón **Eliminar**.

Como alternativa, también puede eliminarla desde la CLI de Azure con el siguiente comando:

```azurecli
az spring-cloud app deployment delete -n <staging-deployment-name> -g <resource-group-name> -s <service-instance-name> --app gateway
```

## <a name="next-steps"></a>Pasos siguientes

* [CI/CD para Azure Spring Cloud](./how-to-cicd.md?pivots=programming-language-java)
