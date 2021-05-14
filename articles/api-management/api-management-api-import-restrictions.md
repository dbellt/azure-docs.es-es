---
title: Restricciones y detalles de los formatos de API compatibles
titleSuffix: Azure API Management
description: Los detalles de los problemas conocidos y las restricciones en los formatos de API Open, WSDL y WADL se admiten en Azure API Management.
services: api-management
documentationcenter: ''
author: vladvino
manager: vlvinogr
editor: ''
ms.assetid: 7a5a63f0-3e72-49d3-a28c-1bb23ab495e2
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 01/02/2020
ms.author: apimpm
ms.openlocfilehash: 3205d4c4c6d8d7d821cb65400a4c1f19d6484005
ms.sourcegitcommit: 49bd8e68bd1aff789766c24b91f957f6b4bf5a9b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108226541"
---
# <a name="api-import-restrictions-and-known-issues"></a>Restricciones de importación de API y problemas conocidos

## <a name="about-this-list"></a>Acerca de esta lista

Al importar una API, es posible que se encuentre con algunas restricciones o que identifique los problemas que se deben rectificar antes de poder realizar la importación correctamente. En este artículo se documentan estas limitaciones, organizadas por el formato de importación de la API. También se describe cómo funciona la exportación de OpenAPI.

## <a name="openapiswagger-import-limitations"></a><a name="open-api"> </a>Limitaciones de la importación de OpenAPI/Swagger

Si recibe errores al importar el documento de OpenAPI, asegúrese de haberlo validado con antelación. Para ello, use el diseñador en Azure Portal (Diseño - Front-End - OpenAPI Specification Editor [Editor de especificaciones de OpenAPI]) o con una herramienta de terceros como <a href="https://editor.swagger.io">Swagger Editor</a>.

### <a name="general"></a><a name="open-api-general"> </a>General

-   Los parámetros necesarios en la ruta de acceso y en la consulta deben tener nombres únicos. (En OpenAPI, un nombre de parámetro solo debe ser único dentro de una ubicación; por ejemplo, ruta de acceso, consulta, encabezado. Pero en API Management se permite que los parámetros tanto de la ruta de acceso como de la consulta [que OpenAPI no admite] discriminen las operaciones. Por eso es necesario que los nombres de los parámetros sean únicos en toda la plantilla de la dirección URL).
-   `\$ref`Los punteros no pueden hacer referencia a archivos externos.
-   `x-ms-paths` y `x-servers` son las únicas extensiones admitidas.
-   Las extensiones personalizadas se omiten durante la importación y no se guardan ni conservan para la exportación.
-   `Recursion` -API Management no admite definiciones establecidas de forma recursiva (por ejemplo, esquemas que hacen referencia a ellos mismos).
-   La dirección URL (si está disponible) del archivo de origen se aplica a direcciones URL del servidor relativas.
-   Se omiten las definiciones de seguridad.
-   No se admiten las definiciones de esquemas alineadas para las operaciones de API. Las definiciones de esquema se definen en el ámbito de la API y se puede hacer referencia a ellas en los ámbitos de solicitud o respuesta de operaciones de API.
-   Un parámetro de dirección URL definido debe formar parte de la plantilla de dirección URL.
-   El objeto `server` no se admite en el nivel de operación de API.
-   `Produces` no se admite la palabra clave, que describe los tipos MIME devueltos por una API. 

### <a name="openapi-version-2"></a><a name="open-api-v2"> </a>OpenAPI versión 2

-   Solo se admite el formato JSON.

### <a name="openapi-version-3"></a><a name="open-api-v3"> </a>OpenAPI versión 3

-   Si se especifican muchos `servers`, API Management intentará seleccionar la primera dirección URL HTTPs. Si no hay ninguna dirección URL HTTPS: la primera dirección URL HTTP. Si no hay ninguna dirección URL HTTP: la dirección URL del servidor estará vacía.
-   `Examples` no se admite, pero `example` está.

## <a name="openapi-import-update-and-export-mechanisms"></a>OpenAPI mecanismos de importación, actualización y exportación

### <a name="general"></a><a name="open-import-export-general"> </a>General

-   Las definiciones de API exportadas desde el servicio API Management están pensadas principalmente para las aplicaciones externas al servicio API Management que necesitan llamar a la API hospedada en este servicio. Las definiciones de API exportadas no están destinadas a importarse en este u otro servicio de API Management. Para más información sobre la administración de la configuración de las definiciones de API en los diferentes servicios o entornos, consulte la documentación relativa al uso del servicio API Management con Git. 

### <a name="add-new-api-via-openapi-import"></a>Incorporación de una nueva API a través de OpenAPI Import

Para cada operación encontrada en el documento OpenAPI, se creará una nueva operación con el nombre de recurso de Azure y el nombre para mostrar establecidos en `operationId` y `summary` respectivamente. `operationId` el valor se normaliza según las reglas que se describen a continuación. `summary` el valor se importa tal cual y su longitud está limitada a 300 caracteres.

Si no se especifica `operationId` (es decir, no está presente, `null`, o vacío), el valor del nombre del recurso de Azure se generará combinando el método HTTP y la plantilla de ruta de acceso, por ejemplo, `get-foo`.

Si no se especifica `summary` (es decir, no está presente, `null` o está vacío), el valor de `display name` se establecerá en `operationId`. Si no se especifica `operationId`, el valor de nombre para mostrar se generará combinando el método HTTP y la plantilla de ruta de acceso, por ejemplo, `Get - /foo`.

### <a name="update-an-existing-api-via-openapi-import"></a>Actualización de una API existente a través de la importación de OpenAPI

Durante la importación, se cambia la API existente para que coincida con la API descrita en el documento OpenAPI. Cada operación del documento OpenAPI se asocia a una operación existente comparando su valor `operationId` con el nombre de recurso de Azure de la operación existente.

Si se encuentra una coincidencia, las propiedades de la operación existente se actualizarán "en contexto".

Si no se encuentra ninguna coincidencia, se creará una nueva operación con las reglas descritas en la sección anterior. Para cada nueva operación, la importación intentará copiar las directivas de una operación existente con el mismo método HTTP y la misma plantilla de ruta de acceso.

Se eliminarán todas las operaciones no coincidentes existentes.

Para hacer que la importación sea más predecible, siga estas instrucciones:

- Asegúrese de especificar `operationId` propiedad para cada operación.
- Absténgase de cambiar `operationId` después de la importación inicial.
- Nunca cambie `operationId` y la plantilla del método HTTP o de la ruta de acceso al mismo tiempo.

### <a name="export-api-as-openapi"></a>Exportar API como OpenAPI

Para cada operación, su nombre de recurso de Azure se exportará como un `operationId` y el nombre para mostrar se exportará como un `summary`.
Reglas de normalización para operationId

- Convertir en minúsculas.
- Reemplace cada secuencia de caracteres no alfanuméricos por un solo guión; por ejemplo, `GET-/foo/{bar}?buzz={quix}` se transformará en `get-foo-bar-buzz-quix-`.
- Recortar los guiones en ambos lados, por ejemplo, `get-foo-bar-buzz-quix-` se convertirán en `get-foo-bar-buzz-quix`
- Se trunca para ajustar 76 caracteres, cuatro caracteres menos que el límite máximo de un nombre de recurso.
- Utilice los cuatro caracteres restantes para un sufijo de desduplicación, si es necesario, en forma de `-1, -2, ..., -999`.


## <a name="wsdl"></a><a name="wsdl"> </a>WSDL

Los archivos WSDL se usan para crear API de paso a través de SOAP y de SOAP a REST.

-   **Enlaces SOAP**: solo se admiten los enlaces SOAP cuya codificación es de tipo "documento" o "literal". No se admiten las codificaciones de estilo "rpc" o SOAP.
-   **WSDL:Import**: no se admite este atributo. Los clientes deben combinar las importaciones en un único documento.
-   **Mensajes con varias partes**: en la actualidad no se admiten estos tipos de mensajes.
-   **WCF wsHttpBinding**: los servicios SOAP creados con Windows Communication Foundation deben usar basicHttpBinding. No se admite el uso de wsHttpBinding.
-   **MTOM**: los servicios que usan MTOM <em>podrían</em> funcionar. No se ofrece soporte técnico oficial en este momento.
-   **Recursión**: APIM no admite los tipos que se definen de modo recursivo (por ejemplo, hacen referencia a una matriz formada por ellos mismos).
-   **Varios espacios de nombres**: en un esquema se pueden usar varios espacios de nombres, pero el único que se puede usar para definir las partes del mensaje es el de destino. Los espacios de nombres distintos del destino, que se usan para definir otros elementos de entrada o salida, no se conservan. Aunque se puede importar un documento WSDL, en la exportación todas las partes del mensaje tendrá el espacio de nombres de destino de WSDL.
-   **Matrices**: la transformación de SOAP a REST admite solo las matrices encapsuladas que se muestran en el siguiente ejemplo:

```xml
    <complexType name="arrayTypeName">
        <sequence>
            <element name="arrayElementValue" type="arrayElementType" minOccurs="0" maxOccurs="unbounded"/>
        </sequence>
    </complexType>
    <complexType name="typeName">
        <sequence>
            <element name="element1" type="someTypeName" minOccurs="1" maxOccurs="1"/>
            <element name="element2" type="someOtherTypeName" minOccurs="0" maxOccurs="1" nillable="true"/>
            <element name="arrayElement" type="arrayTypeName" minOccurs="1" maxOccurs="1"/>
        </sequence>
    </complexType>
```

## <a name="wadl"></a><a name="wadl"> </a>WADL

Actualmente, no hay ningún problema de importación conocido de WADL.
