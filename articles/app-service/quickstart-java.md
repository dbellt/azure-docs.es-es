---
title: 'Inicio rápido: Creación de una aplicación de Java en Azure App Service'
description: Implemente su primera aplicación Hola mundo de Java en Azure App Service en cuestión de minutos. El complemento Aplicación web de Azure para Maven facilita la implementación de aplicaciones Java.
keywords: azure, app service, web app, windows, linux, java, maven, quickstart
author: jasonfreeberg
ms.assetid: 582bb3c2-164b-42f5-b081-95bfcb7a502a
ms.devlang: Java
ms.topic: quickstart
ms.date: 08/01/2020
ms.author: jafreebe
ms.custom: mvc, seo-java-july2019, seo-java-august2019, seo-java-september2019
zone_pivot_groups: app-service-platform-windows-linux
adobe-target: true
adobe-target-activity: DocsExp–386541–A/B–Enhanced-Readability-Quickstarts–2.19.2021
adobe-target-experience: Experience B
adobe-target-content: ./quickstart-java-uiex
ms.openlocfilehash: 7393a8085a2ac597f3fdbcc365608d32956f39b7
ms.sourcegitcommit: 98308c4b775a049a4a035ccf60c8b163f86f04ca
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/30/2021
ms.locfileid: "113111706"
---
# <a name="quickstart-create-a-java-app-on-azure-app-service"></a>Inicio rápido: Creación de una aplicación de Java en Azure App Service

[Azure App Service](overview.md) proporciona un servicio de hospedaje web muy escalable y con aplicación de revisiones de un modo automático.  En este inicio rápido se muestra cómo usar la [CLI de Azure](/cli/azure/get-started-with-azure-cli) con el [complemento Aplicación web de Azure para Maven](https://github.com/Microsoft/azure-maven-plugins/tree/develop/azure-webapp-maven-plugin) para implementar un archivo .jar o un archivo .war. Use las pestañas para cambiar entre las instrucciones para Java SE y Tomcat.


> [!NOTE]
> También se puede hacer lo mismo con IDE populares, como IntelliJ y Eclipse. Consulte nuestros documentos similares en [Inicio rápido de Azure Toolkit for IntelliJ](/azure/developer/java/toolkit-for-intellij/create-hello-world-web-app) o [Inicio rápido de Azure Toolkit for Eclipse](/azure/developer/java/toolkit-for-eclipse/create-hello-world-web-app).


![Aplicación de ejemplo que se ejecuta en Azure App Service](./media/quickstart-java/java-hello-world-in-browser-azure-app-service.png)

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-a-java-app"></a>Creación de una aplicación Java

# <a name="java-se"></a>[Java SE](#tab/javase)

Clone el proyecto de ejemplo [Primeros pasos de Spring Boot](https://github.com/spring-guides/gs-spring-boot).

```azurecli-interactive
git clone https://github.com/spring-guides/gs-spring-boot
```

Cambie de directorio al del proyecto completado.

```azurecli-interactive
cd gs-spring-boot/complete
```

# <a name="tomcat"></a>[Tomcat](#tab/tomcat)

Ejecute el siguiente comando de Maven en el símbolo del sistema de Cloud Shell para crear una aplicación denominada `helloworld`:

```azurecli-interactive
mvn archetype:generate "-DgroupId=example.demo" "-DartifactId=helloworld" "-DarchetypeArtifactId=maven-archetype-webapp" "-Dversion=1.0-SNAPSHOT"
```

A continuación, cambie el directorio de trabajo a la carpeta del proyecto:

```azurecli-interactive
cd helloworld
```

# <a name="jboss-eap"></a>[JBoss EAP](#tab/jbosseap)

::: zone pivot="platform-windows"

JBoss EAP solo está disponible en la versión para Linux de App Service. Seleccione el botón **Linux** de la parte superior de este artículo para ver las instrucciones de inicio rápido de JBoss EAP.

::: zone-end
::: zone pivot="platform-linux"

Clone la aplicación de demostración Pet Store (Tienda para mascotas).

```azurecli-interactive
git clone https://github.com/agoncal/agoncal-application-petstore-ee7.git
```

Cambie de directorio al del proyecto clonado.

```azurecli-interactive
cd agoncal-application-petstore-ee7
```

::: zone-end

---

## <a name="configure-the-maven-plugin"></a>Configuración del complemento Maven

El proceso de implementación en Azure App Service utilizará sus credenciales de Azure desde la CLI de Azure automáticamente. El complemento Maven se autenticará con OAuth o el inicio de sesión del dispositivo si la CLI de Azure no está instalada localmente. Para más información, consulte [Autenticación con los complementos de Maven](https://github.com/microsoft/azure-maven-plugins/wiki/Authentication).

Ejecute el siguiente comando de Maven para configurar la implementación. Este comando le ayudará a configurar el sistema operativo de App Service, la versión de Java y la versión de Tomcat.

```azurecli-interactive
mvn com.microsoft.azure:azure-webapp-maven-plugin:2.0.0:config
```

::: zone pivot="platform-windows"

# <a name="java-se"></a>[Java SE](#tab/javase)

1. Si se le solicita la opción **Suscripción**, escriba el número impreso en el comienzo de línea para seleccionar el valor de `Subscription` adecuado.
2. Cuando se le solicite la opción **Aplicación web**, presione Entrar para seleccionar la opción predeterminada, `<create>`.
3. Cuando se le solicite la opción **SO**, especifique `2` para seleccionar **Windows**.
4. Cuando se le solicite la opción **javaVersion**, escriba `1` para seleccionar **Java 8**.
5. Cuando se le solicite la opción **Plan de tarifa**, especifique `7` para seleccionar **P1v2**.
6. Por último, presione Entrar en la última solicitud para confirmar las selecciones.

    La salida del resumen tendrá un aspecto similar al fragmento de código que se muestra a continuación.

    ```
    Please confirm webapp properties
    Subscription Id : ********-****-****-****-************
    AppName : spring-boot-1599007390755
    ResourceGroup : spring-boot-1599007390755-rg
    Region : westeurope
    PricingTier : Basic_B2
    OS : Windows
    Java : 1.8
    WebContainer : java 8
    Deploy to slot : false
    Confirm (Y/N)? : Y
    [INFO] Saving configuration to pom.
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time: 41.118 s
    [INFO] Finished at: 2020-09-01T17:43:45-07:00
    [INFO] ------------------------------------------------------------------------
    ```

# <a name="tomcat"></a>[Tomcat](#tab/tomcat)

1. Si se le solicita la opción **Suscripción**, escriba el número impreso en el comienzo de línea para seleccionar el valor de `Subscription` adecuado.
2. Cuando se le solicite la opción **Aplicación web**, presione Entrar para seleccionar la opción predeterminada, `<create>`.
3. Cuando se le solicite la opción **SO**, especifique `2` para seleccionar **Windows**.
4. Cuando se le solicite la opción **javaVersion**, escriba `1` para seleccionar **Java 8**.
5. Cuando se le solicite la opción **webContainer**, escriba `3` para seleccionar **Tomcat 8.5**.
6. Cuando se le solicite la opción **Plan de tarifa**, especifique `7` para seleccionar **P1v2**.
7. Por último, presione Entrar en la última solicitud para confirmar las selecciones.

    La salida del resumen tendrá un aspecto similar al fragmento de código que se muestra a continuación.

    ```
    Please confirm webapp properties
    Subscription Id : ********-****-****-****-************
    AppName : helloworld-1599003152123
    ResourceGroup : helloworld-1599003152123-rg
    Region : westeurope
    PricingTier : Basic_B2
    OS : Windows
    Java : 1.8
    WebContainer : tomcat 8.5
    Deploy to slot : false
    Confirm (Y/N)? : Y
    [INFO] Saving configuration to pom.
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time: 03:03 min
    [INFO] Finished at: 2020-09-01T16:35:30-07:00
    [INFO] ------------------------------------------------------------------------
    ```

# <a name="jboss-eap"></a>[JBoss EAP](#tab/jbosseap)

JBoss EAP solo está disponible en la versión para Linux de App Service. Seleccione el botón **Linux** de la parte superior de este artículo para ver las instrucciones de inicio rápido de JBoss EAP.

---

::: zone-end
::: zone pivot="platform-linux"

# <a name="java-se"></a>[Java SE](#tab/javase)

1. Cuando se le solicite la opción **Suscripción**, escriba el número impreso en el comienzo de línea para seleccionar el valor de `Subscription` adecuado.
1. Cuando se le solicite la opción **Aplicación web**, presione Entrar para seleccionar la opción predeterminada, `<create>`.
1. Cuando se le solicite la opción **SO**, presione Entrar para seleccionar **Linux**.
2. Cuando se le solicite la opción **javaVersion**, escriba `1` para seleccionar **Java 8**.
3. Cuando se le solicite la opción **Plan de tarifa**, especifique `6` para seleccionar **P1v2**.
4. Por último, presione Entrar en la última solicitud para confirmar las selecciones.

    ```
    Please confirm webapp properties
    Subscription Id : ********-****-****-****-************
    AppName : spring-boot-1599007116351
    ResourceGroup : spring-boot-1599007116351-rg
    Region : westeurope
    PricingTier : Basic_B2
    OS : Linux
    RuntimeStack : JAVA 8-jre8
    Deploy to slot : false
    Confirm (Y/N)? : Y
    [INFO] Saving configuration to pom.
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time: 20.925 s
    [INFO] Finished at: 2020-09-01T17:38:51-07:00
    [INFO] ------------------------------------------------------------------------
    ```

# <a name="tomcat"></a>[Tomcat](#tab/tomcat)

1. Cuando se le solicite la opción **Suscripción**, escriba el número impreso en el comienzo de línea para seleccionar el valor de `Subscription` adecuado.
1. Cuando se le solicite la opción **Aplicación web**, presione Entrar para seleccionar la opción predeterminada, `<create>`.
1. Cuando se le solicite la opción **SO**, presione Entrar para seleccionar **Linux**.
1. Cuando se le solicite la opción **javaVersion**, escriba `1` para seleccionar **Java 8**.
1. Cuando se le solicite la opción **runtimeStack**, escriba `3` para seleccionar **Tomcat 8.5**.
1. Cuando se le solicite la opción **Plan de tarifa**, especifique `6` para seleccionar **P1v2**.
1. Por último, presione Entrar en la última solicitud para confirmar las selecciones.

    ```
    Please confirm webapp properties
    Subscription Id : ********-****-****-****-************
    AppName : helloworld-1599003744223
    ResourceGroup : helloworld-1599003744223-rg
    Region : westeurope
    PricingTier : Basic_B2
    OS : Linux
    RuntimeStack : TOMCAT 8.5-jre8
    Deploy to slot : false
    Confirm (Y/N)? : Y
    [INFO] Saving configuration to pom.
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time: 50.785 s
    [INFO] Finished at: 2020-09-01T16:43:09-07:00
    [INFO] ------------------------------------------------------------------------
    ```

# <a name="jboss-eap"></a>[JBoss EAP](#tab/jbosseap)

1. Si se le solicita la opción **Suscripción**, escriba el número impreso en el comienzo de línea para seleccionar el valor de `Subscription` adecuado.
1. Cuando aparezca la opción **Aplicación web**, acepte la opción predeterminada `<create>` presionando Entrar.
1. Cuando se le solicite la opción **SO**, presione Entrar para seleccionar **Linux**.
1. Cuando se le solicite la opción **javaVersion**, escriba `1` para seleccionar **Java 8**.
1. Cuando se le solicite la opción **runtimeStack**, escriba `2` para seleccionar **Jbosseap 7**.
1. Cuando se le solicite la opción **Plan de tarifa**, especifique `3` para seleccionar **P1v3**.
1. Por último, presione Entrar en la última solicitud para confirmar las selecciones.

    ```
    Please confirm webapp properties
    Subscription Id : ********-****-****-****-************
    AppName : petstoreee7-1623451825408
    ResourceGroup : petstoreee7-1623451825408-rg
    Region : westeurope
    PricingTier : P1v3
    OS : Linux
    Java : Java 8
    Web server stack: Jbosseap 7.2
    Deploy to slot : false
    Confirm (Y/N) [Y]: y
    [INFO] Saving configuration to pom.
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time: 01:01 min
    [INFO] Finished at: 2021-06-11T15:52:25-07:00
    [INFO] ------------------------------------------------------------------------
    ```

---

::: zone-end

Las configuraciones de App Service se pueden modificar directamente en el archivo `pom.xml` si es necesario. A continuación se enumeran algunas de las más comunes:

Propiedad | Obligatorio | Descripción | Versión
---|---|---|---
`<schemaVersion>` | false | Especifique la versión del esquema de configuración. Los valores admitidos son: `v1`, `v2`. | 1.5.2
`<subscriptionId>` | false | Especifique el identificador de suscripción. | 0.1.0+
`<resourceGroup>` | true | Grupo de recursos de Azure para la aplicación web. | 0.1.0+
`<appName>` | true | El nombre de la aplicación web. | 0.1.0+
`<region>` | true | Especifica la región donde se hospedará la aplicación web. El valor predeterminado es **westeurope**. Todas las regiones válidas en la sección [Regiones admitidas](https://azure.microsoft.com/global-infrastructure/services/?products=app-service). | 0.1.0+
`<pricingTier>` | true | El plan de tarifa de la aplicación web. El valor predeterminado es **P1V2** para cargas de trabajo de producción, mientras que **B2** es el mínimo recomendado para desarrollo y pruebas de Java. [Más información](https://azure.microsoft.com/pricing/details/app-service/linux/)| 0.1.0+
`<runtime>` | true | La configuración del entorno en tiempo de ejecución. Puede ver los detalles [aquí](https://github.com/microsoft/azure-maven-plugins/wiki/Azure-Web-App:-Configuration-Details). | 0.1.0+
`<deployment>` | true | La configuración de implementación. Puede ver los detalles [aquí](https://github.com/microsoft/azure-maven-plugins/wiki/Azure-Web-App:-Configuration-Details). | 0.1.0+

Tenga cuidado con los valores de `<appName>` y `<resourceGroup>` (`helloworld-1590394316693` y `helloworld-1590394316693-rg` en la demostración), ya que se usarán más adelante.

> [!div class="nextstepaction"]
> [He tenido un problema](https://www.research.net/r/javae2e?tutorial=quickstart-java&step=config)

## <a name="deploy-the-app"></a>Implementar la aplicación

El complemento de Maven usa las credenciales de la cuenta desde la CLI de Azure para la implementación en App Services. [Inicie sesión con la CLI de Azure](/cli/azure/authenticate-azure-cli) antes de continuar.

```azurecli-interactive
az login
```

A continuación, puede implementar la aplicación de Java en Azure mediante el siguiente comando.

```azurecli-interactive
mvn package azure-webapp:deploy
```

::: zone pivot="platform-linux"

> [!NOTE]
> Para JBoss EAP, ejecute `mvn package azure-webapp:deploy -DskipTests` para deshabilitar las pruebas, ya que requiere que Wildfly esté instalado localmente. 

::: zone-end

Una vez finalizada la implementación, la aplicación estará lista en `http://<appName>.azurewebsites.net/` (`http://helloworld-1590394316693.azurewebsites.net` en la demostración). Abra la dirección URL con un explorador web local; debería ver

![Aplicación de ejemplo que se ejecuta en Azure App Service](./media/quickstart-java/java-hello-world-in-browser-azure-app-service.png)

**¡Enhorabuena!** Ha implementado su primera aplicación Java en App Service.

> [!div class="nextstepaction"]
> [He tenido un problema](https://www.research.net/r/javae2e?tutorial=app-service-linux-quickstart&step=deploy)

## <a name="clean-up-resources"></a>Limpieza de recursos

En los pasos anteriores, creó recursos de Azure en un grupo de recursos. Si prevé que no va a necesitar estos recursos en el futuro, elimine el grupo de recursos del portal, para lo que debe ejecutar el siguiente comando en Cloud Shell:

```azurecli-interactive
az group delete --name <your resource group name; for example: helloworld-1558400876966-rg> --yes
```

Este comando puede tardar varios segundos en ejecutarse.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Conexión a Azure DB para PostgreSQL con Java](../postgresql/connect-java.md)

> [!div class="nextstepaction"]
> [Configure CI/CD](deploy-continuous-deployment.md)

> [!div class="nextstepaction"]
> [Información de precios](https://azure.microsoft.com/pricing/details/app-service/linux/)

> [!div class="nextstepaction"]
> [Incorporación de registros y métricas](troubleshoot-diagnostic-logs.md)

> [!div class="nextstepaction"]
> [Escalado vertical](manage-scale-up.md)

> [!div class="nextstepaction"]
> [Recursos de Azure para desarrolladores de Java](/java/azure/)

> [!div class="nextstepaction"]
> [Configuración de la aplicación de Java](configure-language-java.md)
