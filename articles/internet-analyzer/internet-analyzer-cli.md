---
title: Creación de una prueba de Internet Analyzer mediante la CLI | Microsoft Docs
description: En este artículo, aprenderá a crear su primera prueba de Internet Analyzer mediante la CLI de Azure.
services: internet-analyzer
author: diego-perez-botero
ms.service: internet-analyzer
ms.topic: tutorial
ms.date: 10/16/2019
ms.author: mebeatty
ms.openlocfilehash: 46caae53ed81de335c2b9d5ddbd3fd7f89424fdd
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107780422"
---
# <a name="create-an-internet-analyzer-test-using-cli-preview"></a>Creación de una prueba de Internet Analyzer mediante la CLI (versión preliminar)

Hay dos maneras de crear un recurso de Internet Analyzer: mediante [Azure Portal](internet-analyzer-create-test-portal.md) o mediante el uso de la CLI. Esta sección le ayuda a crear un nuevo recurso de Azure Internet Analyzer con la experiencia de la CLI. 


> [!IMPORTANT]
> Esta versión preliminar pública se proporciona sin un acuerdo de nivel de servicio y no debe usarse para cargas de trabajo de producción. Puede que algunas características no se admitan, que tengan funcionalidades limitadas o que no estén disponibles en todas las ubicaciones de Azure. Para más información, consulte [Términos de uso complementarios de las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).
>

## <a name="before-you-begin"></a>Antes de empezar

La versión preliminar pública está disponible para su uso global; sin embargo, el almacenamiento de datos está limitado a *Oeste de EE. UU. 2* durante la versión preliminar.

## <a name="object-model"></a>Modelo de objetos
La CLI de Internet Analyzer expone los siguientes tipos de recursos:
* **Pruebas**: una prueba compara el rendimiento de usuario final de dos puntos de conexión de Internet (A y B) a lo largo del tiempo.
* **Perfiles**: las pruebas se crean en un perfil de Internet Analyzer. Los perfiles permiten agrupar las pruebas relacionadas; un único perfil puede contener una o varias pruebas.
* **Puntos de conexión preconfigurados**: hemos configurado puntos de conexión con una variedad de configuraciones (regiones, tecnologías de aceleración, etc.). Puede usar cualquiera de estos puntos de conexión preconfigurados en las pruebas.
* **Cuadros de mandos**: un cuadro de mandos proporciona resúmenes rápidos y significativos de los resultados de las mediciones. Consulte [Interpretación del cuadro de mandos](internet-analyzer-scorecard.md).
* **Series temporales**: una serie temporal muestra cómo cambia una métrica con el tiempo.

## <a name="profile-and-test-creation"></a>Creación de perfiles y pruebas
1. Obtenga acceso a la versión preliminar de Internet Analyzer siguiendo las instrucciones de **¿Cómo participar en la versión preliminar?** de las [preguntas frecuentes sobre Azure Internet Analyzer](internet-analyzer-faq.md).
2. [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).
3. Ejecute el comando `login` para iniciar una sesión de la CLI:
    ```azurecli-interactive
    az login
    ```

    Si la CLI puede abrir el explorador predeterminado, lo hará y cargará una página de inicio de sesión de Azure.
    En caso contrario, abra una página del explorador en https://aka.ms/devicelogin y escriba el código de autorización que se muestra en el terminal.

4. Inicie sesión con las credenciales de su cuenta en el explorador.

5. Seleccione el identificador de la suscripción a la que se ha concedido acceso a la versión preliminar pública de Internet Analyzer.

    Después de iniciar sesión, verá una lista de suscripciones asociadas con su cuenta de Azure. La información de suscripción con `isDefault: true` es la suscripción actualmente activada después de iniciar sesión. Para seleccionar otra suscripción, utilice el comando [az account set](/cli/azure/account#az_account_set) con el identificador de suscripción al que desea cambiar. Para más información sobre la selección de suscripción, consulte [Uso de varias suscripciones de Azure](/cli/azure/manage-azure-subscriptions-azure-cli).

    Hay varias formas de iniciar sesión de una manera no interactiva, que se describen con detalle en [Inicio de sesión con la CLI de Azure](/cli/azure/authenticate-azure-cli).

6. **[Opcional]** Creación de un nuevo grupo de recursos de Azure:
    ```azurecli-interactive
    az group create --location eastus --name "MyInternetAnalyzerResourceGroup"
    ```

7. Instale la extensión de Internet Analyzer de la CLI de Azure:
     ```azurecli-interactive
    az extension add --name internet-analyzer
    ```

8. Cree un nuevo perfil de Internet Analyzer:
    ```azurecli-interactive
    az internet-analyzer profile create --location eastus --resource-group "MyInternetAnalyzerResourceGroup" --name "MyInternetAnalyzerProfile" --enabled-state Enabled
    ```

9. Enumere todos los puntos de conexión preconfigurados disponibles para el perfil recién creado:
    ```azurecli-interactive
    az internet-analyzer preconfigured-endpoint list --resource-group "MyInternetAnalyzerResourceGroup" --profile-name "MyInternetAnalyzerProfile"
    ```

10. Cree una nueva prueba en el perfil de Internet Analyzer recién creado:
    ```azurecli-interactive
    az internet-analyzer test create --resource-group "MyInternetAnalyzerResourceGroup" --profile-name "MyInternetAnalyzerProfile" --endpoint-a-name "contoso" --endpoint-a-endpoint "www.contoso.com/some/path/to/trans.gif" --endpoint-b-name "microsoft" --endpoint-b-endpoint "www.microsoft.com/another/path/to/trans.gif" --name "MyFirstInternetAnalyzerTest" --enabled-state Enabled
    ```

    En el comando anterior se supone que tanto `www.contoso.com` como `www.microsoft.com` hospedan la imagen de un píxel ([trans.gif](https://fpc.msedge.net/apc/trans.gif)) en rutas de acceso personalizadas. Si no se especifica explícitamente una ruta de acceso del objeto, Internet Analyzer usará `/apc/trans.gif` como ruta de acceso del objeto de forma predeterminada, que es donde los puntos de conexión preconfigurados hospedan la imagen de un píxel. Tenga en cuenta también que no es necesario especificar el esquema (https/http); Internet Analyzer solo admite puntos de conexión HTTPS, por lo que se presupone HTTPS.

11. La nueva prueba debe aparecer en el perfil de Internet Analyzer:
    ```azurecli-interactive
    az internet-analyzer test list --resource-group "MyInternetAnalyzerResourceGroup" --profile-name "MyInternetAnalyzerProfile"
    ```

    Salida de ejemplo:
    ````
    [
        {
            "description": null,
            "enabledState": "Enabled",
            "endpointA": {
            "endpoint": "www.contoso.com/some/path/to/1k.jpg",
            "name": "contoso"
            },
            "endpointB": {
            "endpoint": "www.microsoft.com/another/path/to/1k.jpg",
            "name": "microsoft"
            },
            "id": "/subscriptions/faa9ddd0-9137-4659-99b7-cdc55a953342/resourcegroups/MyInternetAnalyzerResourceGroup/providers/Microsoft.Network/networkexperimentprofiles/MyInternetAnalyzerProfile/experiments/MyFirstInternetAnalyzerTest",
            "location": null,
            "name": "MyFirstInternetAnalyzerTest",
            "resourceGroup": "MyInternetAnalyzerResourceGroup",
            "resourceState": "Enabled",
            "scriptFileUri": "https://fpc.msedge.net/client/v2/d8c6fc64238d464c882cee4a310898b2/ab.min.js",
            "status": "Created",
            "tags": null,
            "type": "Microsoft.Network/networkexperimentprofiles/experiments"
        }
    ]
    ````

12. Para empezar a generar medidas, el archivo JavaScript al que apunta el elemento **scriptFileUri** de la prueba debe estar insertado en la aplicación web. Puede encontrar instrucciones específicas en la página [Inserción del cliente de Internet Analyzer](internet-analyzer-embed-client.md).

13. Puede supervisar el progreso de la prueba realizando un seguimiento del valor "status":
    ```azurecli-interactive
    az internet-analyzer test show --resource-group "MyInternetAnalyzerResourceGroup" --profile-name "MyInternetAnalyzerProfile" --name "MyFirstInternetAnalyzerTest"
    ```

14. Puede inspeccionar los resultados recopilados de la prueba mediante la generación de series temporales o cuadros de mandos para ella:
    ```azurecli-interactive
    az internet-analyzer show-scorecard --resource-group "MyInternetAnalyzerResourceGroup" --profile-name "MyInternetAnalyzerProfile" --name "MyFirstInternetAnalyzerTest" --aggregation-interval "Daily" --end-date-time-utc "2019-10-24T00:00:00"
    ```

    ```azurecli-interactive
    az internet-analyzer show-timeseries --resource-group "MyInternetAnalyzerResourceGroup" --profile-name "MyInternetAnalyzerProfile" --name "MyFirstInternetAnalyzerTest" --aggregation-interval "Hourly" --start-date-time-utc "2019-10-23T00:00:00" --end-date-time-utc "2019-10-24T00:00:00" --timeseries-type MeasurementCounts
    ```


## <a name="next-steps"></a>Pasos siguientes

* Examine la [Referencia de la CLI de Internet Analyzer](/cli/azure/ext/internet-analyzer/internet-analyzer) para ver la lista completa de comandos y ejemplos de uso admitidos.
* Consulte las [preguntas frecuentes sobre Internet Analyzer](internet-analyzer-faq.md).
* Más información sobre la inserción del [cliente de Internet Analyzer](internet-analyzer-embed-client.md) y la creación de un [punto de conexión personalizado](internet-analyzer-custom-endpoint.md).