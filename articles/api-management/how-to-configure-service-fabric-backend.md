---
title: Configuración de back-end de Service Fabric en Azure API Management | Microsoft Docs
description: Creación de un back-end del servicio Service Fabric en Azure API Management mediante Azure Portal
services: api-management
documentationcenter: ''
author: dlepow
editor: ''
ms.service: api-management
ms.topic: article
ms.date: 01/29/2021
ms.author: apimpm
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 3dda6f18c2bf92b537c2f4be1c6a0a3b70cdc28a
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107815889"
---
# <a name="set-up-a-service-fabric-backend-in-api-management-using-the-azure-portal"></a>Configuración de un back-end de Service Fabric en API Management mediante Azure Portal

En este artículo se muestra cómo configurar el servicio [Service Fabric](../service-fabric/service-fabric-api-management-overview.md) como un back-end de API personalizado mediante Azure Portal. Con fines de demostración, se muestra cómo configurar un servicio básico confiable de ASP.NET Core sin estado como back-end de Service Fabric.

Para obtener información general, consulte [Back-ends en API Management](backends.md).

## <a name="prerequisites"></a>Requisitos previos

Requisitos previos para configurar un servicio de ejemplo en un clúster de Service Fabric que ejecuta Windows como un back-end personalizado:

* **Entorno de desarrollo de Windows**: instale [Visual Studio 2019](https://www.visualstudio.com) y las cargas de trabajo de **desarrollo de Azure**, **desarrollo web y ASP.NET** y **desarrollo a través de plataformas .NET Core**. Después, configure un [entorno de desarrollo .NET](../service-fabric/service-fabric-get-started.md).

* **Clúster de Service Fabric**: consulte [Tutorial: Implementación de un clúster de Service Fabric con Windows una red virtual de Azure](../service-fabric/service-fabric-tutorial-create-vnet-and-windows-cluster.md). Puede crear un clúster con un certificado X.509 existente o, con fines de prueba, crear un nuevo certificado autofirmado. El clúster se crea en una red virtual.

* **Ejemplo de aplicación de Service Fabric**: cree una aplicación de API web e impleméntela en el clúster de Service Fabric como se describe en [Integrar API Management con Service Fabric en Azure](../service-fabric/service-fabric-tutorial-deploy-api-management.md).

    Estos pasos crean un servicio básico y confiable de ASP.NET Core sin estado mediante la plantilla de proyecto de API web predeterminada. Posteriormente, expone el punto de conexión HTTP de este servicio mediante Azure API Management.

    Anote el nombre de la aplicación, por ejemplo `fabric:/myApplication/myService`. 

* **Instancia de API Management**: una instancia de API Management existente o nueva en el nivel **Premium** o **Desarrollador** y en la misma región que el clúster de Service Fabric. Si lo necesita, [cree una instancia de API Management](get-started-create-service-instance.md).

* **Red virtual**: agregue la instancia de API Management a la red virtual que creó para el clúster de Service Fabric. API Management requiere una subred dedicada en la red virtual.

  Para conocer los pasos necesarios para habilitar la conectividad de la red virtual para la instancia de API Management, consulte [Usar Azure API Management con redes virtuales](api-management-using-with-vnet.md).

## <a name="create-backend---portal"></a>Creación de back-end: portal

### <a name="add-service-fabric-cluster-certificate-to-api-management"></a>Incorporación de un certificado de clúster de Service Fabric a API Management

El certificado de clúster de Service Fabric se almacena y administra en un almacén de claves de Azure asociado al clúster. Agregue este certificado a la instancia de API Management como un certificado de cliente.

Para conocer los pasos para agregar un certificado a la instancia de API Management, consulte [Protección de servicios back-end mediante la autenticación de certificados de cliente en Azure API Management](api-management-howto-mutual-certificates.md). 

> [!NOTE]   
> Se recomienda agregar el certificado a API Management haciendo referencia al certificado del almacén de claves. 

### <a name="add-service-fabric-backend"></a>Adición de un back-end de Service Fabric

1. Vaya a la instancia de API Management en [Azure Portal](https://portal.azure.com).
1. En **API**, seleccione **Back-ends** >  **+ Agregar**.
1. Especifique un nombre de back-end y una descripción opcional.
1. En **Tipo**, seleccione **Service Fabric**.
1. En **URL de tiempo de ejecución**, escriba el nombre del servicio back-end de Service Fabric al que API Management reenviará las solicitudes. Ejemplo: `fabric:/myApplication/myService`. 
1. En **Maximum number of partition resolution retries** (Número máximo de reintentos de resolución de particiones), escriba un número entre 0 y 10.
1. Especifique el punto de conexión de administración del clúster de Service Fabric. Este punto de conexión es la dirección URL del clúster en el puerto `19080`, por ejemplo, `https://mysfcluster.eastus.cloudapp.azure.com:19080`.
1. En **Certificados de cliente**, seleccione el certificado del clúster de Service Fabric que agregó a la instancia de API Management en la sección anterior.
1. En **Management endpoint authorization method** (Método de autorización del punto de conexión de administración), escriba un nombre de huella digital o servidor X509 de un certificado que el servicio de administración de clústeres de Service Fabric use para la comunicación con TLS.
1. Habilite las opciones **Validate certificate chain** (Validar cadena de certificados) y **Validate certificate name** (Validar nombre del certificado).
1. En **Authorization credentials** (Credenciales de autorización), proporcione credenciales, si es necesario, para alcanzar el servicio back-end configurado en Service Fabric. En el caso de la aplicación de ejemplo que se usa en este escenario, no se necesitan credenciales de autorización.
1. Seleccione **Crear**.

:::image type="content" source="media/backends/create-service-fabric-backend.png" alt-text="Creación de un back-end de Service Fabric":::

## <a name="use-the-backend"></a>Uso del back-end

Para usar un back-end personalizado, haga referencia a él mediante la directiva [`set-backend-service`](api-management-transformation-policies.md#SetBackendService). Esta directiva transforma la dirección URL base del servicio back-end predeterminado de una solicitud de API entrante en un back-end específico, en este caso el back-end de Service Fabric. 

La directiva `set-backend-service` puede ser útil con una API existente para transformar una solicitud entrante en otro back-end distinto del especificado en la configuración de la API. Para los fines de demostración de este artículo, cree una API de prueba y establezca la directiva para dirigir las solicitudes de API al back-end de Service Fabric. 

### <a name="create-api"></a>Create API

Siga los pasos descritos en [Adición manual de una API](add-api-manually.md) para crear una API en blanco.

* En la configuración de la API, deje **Dirección URL del servicio web** en blanco.
* Agregue un **sufijo de dirección URL de API**, como *fabric*.

  :::image type="content" source="media/backends/create-blank-api.png" alt-text="Crear API en blanco":::

### <a name="add-get-operation-to-the-api"></a>Adición de una operación GET a la API

Como se muestra en [Implementación de un servicio de back-end de Service Fabric](../service-fabric/service-fabric-tutorial-deploy-api-management.md#deploy-a-service-fabric-back-end-service), el servicio ASP.NET Core de ejemplo implementado en el clúster de Service Fabric admite una única operación GET de HTTP en la ruta de acceso de la dirección URL `/api/values`.

La respuesta predeterminada en esa ruta de acceso es una matriz JSON de dos cadenas:

```json
["value1", "value2"]
```

Para probar la integración de API Management con el clúster, agregue la operación GET correspondiente a la API en la ruta de acceso `/api/values`:

1. Seleccione la API que creó en los pasos anteriores.
1. Seleccione **+ Agregar operación**.
1. En la ventana **Front-end**, escriba los siguientes valores y seleccione **Guardar**.

     | Valor             | Value                             | 
    |---------------------|-----------------------------------|
    | **Nombre para mostrar**    | *Probar back-end*                       |  
    | **URL** | GET                               | 
    | **URL**             | `/api/values`                           | 
    
    :::image type="content" source="media/backends/configure-get-operation.png" alt-text="Adición de una operación GET a la API":::

### <a name="configure-set-backend-policy"></a>Configuración de la directiva `set-backend`

Agregue la directiva [`set-backend-service`](api-management-transformation-policies.md#SetBackendService) a la API de prueba.

1. En la pestaña **Diseño**, en la sección **Procesamiento de entrada**, seleccione el icono del editor de código ( **</>** ). 
1. Coloque el cursor dentro del elemento **&lt;inbound&gt;** .
1. Agregue la siguiente instrucción de directiva. En `backend-id`, sustituya el nombre del back-end de Service Fabric.

   `sf-resolve-condition` es una condición de reintento si no se resuelve la partición del clúster. El número de reintentos se estableció al configurar el back-end.

    ```xml
    <set-backend-service backend-id="mysfbackend" sf-resolve-condition="@(context.LastError?.Reason == "BackendConnectionFailure")"  />
    ```
1. Seleccione **Guardar**.

    :::image type="content" source="media/backends/set-backend-service.png" alt-text="Configuración de la directiva set-backend-service":::

### <a name="test-backend-api"></a>Prueba de la API de back-end

1. En la pestaña **Probar** seleccione la operación **GET** que creó en una sección anterior.
1. Seleccione **Enviar**.

Cuando se configura correctamente, la respuesta HTTP muestra un código de operación correcta de HTTP y muestra el JSON devuelto desde el servicio back-end de Service Fabric.

:::image type="content" source="media/backends/test-backend-service.png" alt-text="Probar un back-end de Service Fabric":::

## <a name="next-steps"></a>Pasos siguientes

* Aprenda a [configurar directivas](api-management-advanced-policies.md) para reenviar solicitudes a un back-end
* Los back-ends también se pueden configurar mediante la [API REST](/rest/api/apimanagement/2020-06-01-preview/backend) de API Management, [Azure PowerShell](/powershell/module/az.apimanagement/new-azapimanagementbackend) o [plantillas de Azure Resource Manager](../service-fabric/service-fabric-tutorial-deploy-api-management.md).

