---
title: Configuración personalizada
description: Configure los valores que se aplican a todo el entorno de Azure App Service. Aprenda a hacerlo con plantillas de Azure Resource Manager.
author: ccompy
ms.assetid: 1d1d85f3-6cc6-4d57-ae1a-5b37c642d812
ms.topic: tutorial
ms.date: 01/29/2021
ms.author: ccompy
ms.custom: mvc, seodec18
ms.openlocfilehash: e871778576266d772f37c23ee174dda7df6a6a75
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112027432"
---
# <a name="custom-configuration-settings-for-app-service-environments"></a>Opciones de configuración personalizada para Entornos de App Service
## <a name="overview"></a>Información general
Dado que las instancias de App Service Environment (ASE) están aisladas en un solo cliente, hay ciertos valores de configuración que se pueden aplicar exclusivamente a ellas. En este artículo se documentan las distintas personalizaciones específicas disponibles para los entornos de App Service.

Si no cuenta con un entorno de App Service, consulte [Creación de un entorno de App Service](app-service-web-how-to-create-an-app-service-environment.md).

Puede almacenar las personalizaciones del entorno de App Service mediante el uso de una matriz en el nuevo atributo **clusterSettings** . Este atributo se encuentra en el diccionario de "Propiedades" de la entidad *hostingEnvironments* de Azure Resource Manager.

La siguiente plantilla abreviada de Resource Manager muestra el atributo **clusterSettings** :

```json
"resources": [
{
    "apiVersion": "2015-08-01",
    "type": "Microsoft.Web/hostingEnvironments",
    "name": ...,
    "location": ...,
    "properties": {
        "clusterSettings": [
            {
                "name": "nameOfCustomSetting",
                "value": "valueOfCustomSetting"
            }
        ],
        "workerPools": [ ...],
        etc...
    }
}
```

El atributo **clusterSettings** se puede incluir en una plantilla de Resource Manager para actualizar el entorno de App Service.

## <a name="use-azure-resource-explorer-to-update-an-app-service-environment"></a>Uso del Explorador de recursos de Azure para actualizar un entorno de App Service
Como alternativa, puede actualizar el entorno de App Service mediante el [Explorador de recursos de Azure](https://resources.azure.com).  

1. En el Explorador de recursos, vaya al nodo de App Service Environment (**subscriptions** > **resourceGroups** > **providers** > **Microsoft.Web** > **hostingEnvironments**). A continuación, haga clic en el entorno de App Service específico que quiere actualizar.
2. En el panel derecho, haga clic en **Lectura/escritura** en la barra de herramientas superior para permitir la edición interactiva en el Explorador de recursos.  
3. Haga clic en el botón azul **Editar** para que la plantilla de Resource Manager se pueda editar.
4. Desplácese hasta la parte inferior del panel derecho. El atributo **clusterSettings** se encuentra abajo del todo, donde puede especificar o actualizar su valor.
5. Escriba (o copie y pegue) la matriz de valores de configuración que quiera en el atributo **clusterSettings** .  
6. Haga clic en el botón verde **PUT** situado en la parte superior del panel derecho para confirmar el cambio en el entorno de App Service.

Una vez enviado el cambio, tarda en aplicarse aproximadamente 30 minutos multiplicado por el número de front-ends en el entorno de App Service.
Por ejemplo, si un entorno de App Service tiene cuatro front-ends, tardará aproximadamente dos horas en finalizar la actualización de la configuración. Mientras se implementa el cambio de configuración, no será posible realizar otras operaciones de escalado o de cambio de configuración en el entorno de App Service.

## <a name="enable-internal-encryption"></a>Habilitación del cifrado interno

App Service Environment funciona como sistema de caja negra en el que no se pueden ver los componentes internos ni la comunicación dentro del sistema. Para permitir mayor capacidad de proceso, no se habilita el cifrado de forma predeterminada entre los componentes internos. El sistema es seguro, ya que no se puede acceder al tráfico ni supervisarlo. Si hay alguna medida de cumplimiento normativo que requiera el cifrado completo de toda la ruta de acceso a los datos, una forma de hacerlo es con clusterSetting.  

```json
"clusterSettings": [
    {
        "name": "InternalEncryption",
        "value": "true"
    }
],
```
Si InternalEncryption se establece en True, se cifrará el tráfico de red interno del ASE entre los servidores front-end y los trabajos, así como el archivo de paginación y los discos de trabajo. La habilitación de clusterSetting de InternalEncryption puede afectar al rendimiento del sistema. Al realizar el cambio para habilitar InternalEncryption, ASE tendrá un estado inestable hasta que se propague el cambio totalmente. La propagación completa del cambio puede tardar unas horas en completarse, en función de cuántas instancias tenga en ASE. Le recomendamos encarecidamente que no habilite InternalEncryption en un ASE si este se encuentra en uso. Si necesita habilitar InternalEncryption en un ASE que se esté utilizando activamente, lo mejor es desviar el tráfico a un entorno de copia de seguridad hasta que se complete la operación. 


## <a name="disable-tls-10-and-tls-11"></a>Deshabilitación de TLS 1.0 y TLS 1.1

Si desea administrar la configuración de TLS aplicación por aplicación, luego puede usar la guía que se proporciona en el tutorial [Enlazar un certificado SSL personalizado a Azure App Service](../configure-ssl-bindings.md#enforce-tls-versions). 

Si desea deshabilitar todos los tráfico de TLS 1.0 y TLS 1.1 entrante de todas las aplicaciones de ASE, puede establecer la siguiente entrada de **clusterSettings**:

```json
"clusterSettings": [
    {
        "name": "DisableTls1.0",
        "value": "1"
    }
],
```

El nombre de la configuración dice 1.0, pero cuando se configura, se deshabilitan tanto TLS 1.0 como TLS 1.1.

## <a name="change-tls-cipher-suite-order"></a>Cambio del orden del conjunto de aplicaciones de cifrado TLS
El ASE permite cambiar el conjunto de cifrado predeterminado por otro. El conjunto predeterminado de cifrado es el mismo conjunto que se usa en el servicio multiinquilino. Si se modifican los conjuntos de cifrado, toda la implementación de App Service se verá afectada, por lo que solo puede hacerse en los ASE de un solo inquilino. Los ASE requieren dos conjuntos de cifrado: TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 y TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256. Si desea utilizar el ASE con el conjunto de cifrado más seguro y el conjunto de cifrado más liviano, use solo los dos cifrados requeridos. Si desea configurar el ASE para que solo utilice los cifrados requeridos, modifique **clusterSettings** tal y como se muestra a continuación. 

```json
"clusterSettings": [
    {
        "name": "FrontEndSSLCipherSuiteOrder",
        "value": "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256"
    }
],
```

> [!WARNING]
> Nota: Si se establecen valores incorrectos para el conjunto de aplicaciones de cifrado que SChannel no entiende, toda la comunicación TLS en el servidor podría dejar de funcionar. En tal caso, debe quitar la entrada *FrontEndSSLCipherSuiteOrder* de **clusterSettings** y enviar la plantilla de Resource Manager actualizada para volver a la configuración de conjunto de cifrado predeterminada.  Utilice esta funcionalidad con precaución.

## <a name="get-started"></a>Primeros pasos
El sitio de inicio rápido de plantillas de Azure Resource Manager incluye una plantilla con la definición base para [crear un entorno de App Service](https://azure.microsoft.com/resources/templates/web-app-ase-create/).

<!-- LINKS -->

<!-- IMAGES -->
