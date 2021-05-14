---
title: Adición manual de una API mediante Azure Portal| Microsoft Docs
description: En este tutorial se muestra cómo utilizar la API de administración (APIM) para agregar manualmente una API.
services: api-management
documentationcenter: ''
author: mikebudzynski
ms.service: api-management
ms.topic: tutorial
ms.date: 04/26/2021
ms.author: apimpm
ms.openlocfilehash: 4419bca71e3523d4b1bf6c803a96fe8190bda780
ms.sourcegitcommit: 49bd8e68bd1aff789766c24b91f957f6b4bf5a9b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108230753"
---
# <a name="add-an-api-manually"></a>Adición manual de una API

En los pasos de este artículo se explica cómo usar Azure Portal para agregar manualmente una API a la instancia de API Management (APIM). Un escenario común es si quiere crear una API en blanco y definirla manualmente cuando desee simular la API. Para obtener más información sobre la simulación de una API, consulte el artículo sobre cómo [simular respuestas de API](mock-api-responses.md).

Si desea importar una API existente, consulte la sección de [temas relacionados](#related-topics).

En este artículo, creará una API en blanco y especificará [httpbin.org](https://httpbin.org) (un servicio de prueba público) como API de back-end.

## <a name="prerequisites"></a>Prerrequisitos

Complete el siguiente inicio rápido: [Creación de una instancia de Azure API Management](get-started-create-service-instance.md)

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-navigate-to-instance.md)]

## <a name="create-an-api"></a>Creación de una API

1. Vaya al servicio API Management en Azure Portal y seleccione **API** en el menú.
2. En el menú izquierdo, seleccione **+ Agregar API**.
3. Seleccione **API en blanco** en la lista.  
    ![API en blanco](media/add-api-manually/blank-api.png)  
4. Escriba la configuración de la API. Los valores de configuración se explican en el tutorial [Importación y publicación de la primera API](import-and-publish.md#import-and-publish-a-backend-api).
5. Seleccione **Crear**.

En este momento, no tiene ninguna operación en API Management que se asigne a las operaciones en la API de back-end. Si llama a una operación que se expone a través del back-end, pero no a través de API Management, obtendrá una respuesta **404**.

>[!NOTE] 
> De manera predeterminada, cuando agrega una API, aunque esté conectado a algún servicio back-end, APIM no expondrá ninguna operación hasta que la permita. Para permitir una operación del servicio de back-end, cree una operación de APIM que se asigne a la operación de back-end.

## <a name="add-and-test-an-operation"></a>Adición y prueba de una operación

En esta sección se muestra cómo agregar una operación "/get" para asignarla a la operación de back-end "http://httpbin.org/get".

### <a name="add-an-operation"></a>Agregar una operación

1. Seleccione la API que creó en los pasos anteriores.
2. Haga clic en **+ Agregar operación**.
3. En la **URL**, seleccione **GET** y escriba `/get` en el recurso.
4. Escriba "*FetchData*" en **Nombre para mostrar**.
5. Seleccione **Guardar**.

### <a name="test-an-operation"></a>Probar una operación

Pruebe la operación en Azure Portal. También puede probarla en el **Portal para desarrolladores**.

1. Seleccione la pestaña **Prueba**.
2. Seleccione **FetchData**.
3. Presione **Enviar**.

Se muestra la respuesta que genera la operación "http://httpbin.org/get". Si desea transformar las operaciones, consulte [Transformación y protección de una API](transform-api.md).

## <a name="add-and-test-a-parameterized-operation"></a>Adición y prueba de una operación con parámetros

En esta sección se explica cómo agregar una operación que toma un parámetro. En este caso, se asigna la operación a "http://httpbin.org/status/200".

### <a name="add-the-operation"></a>Adición de la operación

1. Seleccione la API que creó en los pasos anteriores.
2. Haga clic en **+ Agregar operación**.
3. En la **URL**, seleccione **GET** y escriba `*/status/{code}` en el recurso. Si lo desea, puede proporcionar cierta información asociada a este parámetro. Por ejemplo, escriba "*Número*" en **TYPE** y "*200*" (valor predeterminado) en **VALUES**.
4. Escriba "WildcardGet" como **Nombre para mostrar**.
5. Seleccione **Guardar**.

### <a name="test-the-operation"></a>Prueba de la operación 

Pruebe la operación en Azure Portal.  También puede probarla en el **Portal para desarrolladores**.

1. Seleccione la pestaña **Prueba**.
2. Seleccione **WildcardGet**. De forma predeterminada, el valor de código se establece en "*200*". Puede cambiarlo para probar otros valores. Por ejemplo, escriba "*418*".
3. Presione **Enviar**.

    Se muestra la respuesta que genera la operación "http://httpbin.org/status/200". Si desea transformar las operaciones, consulte [Transformación y protección de una API](transform-api.md).

## <a name="add-and-test-a-wildcard-operation"></a>Adición y prueba de una operación con caracteres comodín

En esta sección se indica cómo agregar una operación con caracteres comodín. Una operación con caracteres comodín le permite pasar un valor arbitrario con una solicitud de API. En lugar de crear operaciones GET independientes como se muestra en las secciones anteriores, podría crear una operación GET con caracteres comodín.

### <a name="add-the-operation"></a>Adición de la operación

1. Seleccione la API que creó en los pasos anteriores.
2. Haga clic en **+ Agregar operación**.
3. En la **URL**, seleccione **GET** y escriba `/*` en el recurso.
4. Escriba "*WildcardGet*" como **Nombre para mostrar**.
5. Seleccione **Guardar**.

### <a name="test-the-operation"></a>Prueba de la operación 

Pruebe la operación en Azure Portal.  También puede probarla en el **Portal para desarrolladores**.

1. Seleccione la pestaña **Prueba**.
2. Seleccione **WildcardGet**. Pruebe una o varias de las operaciones GET que ha probado en secciones anteriores o pruebe otra operación GET compatible. 

    Por ejemplo, en **Parámetros de la plantilla**, actualice el valor junto al nombre del carácter comodín (*) a `headers`. La operación devuelve los encabezados HTTP de la solicitud entrante.
1. Presione **Enviar**.

    Se muestra la respuesta que genera la operación "http://httpbin.org/headers". Si desea transformar las operaciones, consulte [Transformación y protección de una API](transform-api.md).

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-append-apis.md)]

[!INCLUDE [api-management-define-api-topics.md](../../includes/api-management-define-api-topics.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Transformación y protección de una API publicada](transform-api.md)
