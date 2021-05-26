---
title: Entidades con nombre de información personal
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: include
ms.date: 04/14/2021
ms.author: aahi
ms.openlocfilehash: 20334f48866875614e313ea35403f3d2694ad275
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110164860"
---
> [!NOTE]
> Para detectar información de salud protegida (PHI), use el parámetro `domain=phi` y la versión del modelo `2020-04-01` o posterior.
>
> Por ejemplo: `https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.1-preview.5/entities/recognition/pii?domain=phi&model-version=2021-01-15`
 
Se devuelven las siguientes categorías de entidad al enviar solicitudes al punto de conexión `/v3.1-preview.5/entities/recognition/pii`.


| Category   |  Descripción                          |
|------------|-------------|
| [Person](#category-person)      |  Nombres de personas.  |
| [PersonType](#category-persontype) | Tipos de trabajo o roles mantenido por una persona. |
| [Número de teléfono](#category-phonenumber) |Números de teléfono (solo números de teléfono de EE. UU y la UE). |
| [Organización](#category-organization) |  Compañías, grupos, organismos gubernamentales y otras organizaciones.  |
| [Dirección](#category-address) | Dirección de correo postal completa.  |
| [Correo electrónico](#category-email) | Direcciones de correo.   |
| [URL](#category-url) | Direcciones URL de sitios web.  |
| [IP](#category-ip) | Direcciones IP de red.  |
| [DateTime](#category-datetime) | Fechas y horas del día. | 
| [Cantidad](#category-quantity) | Números y cantidades numéricas.  |
| [Información de Azure](#azure-information) | Información de Azure identificable, como la información de autenticación.  |
| [Identificación](#identification) | Identificación específica fiscal y del país.  |

### <a name="category-person"></a>Categoría: Person

Este categoría contiene la entidad siguiente:

:::row:::
    :::column span="":::
        **Entidad**

        Person

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Nombres de personas. También se devuelve con `domain=phi`.

        Para obtener esta categoría de entidad, agregue `Person` al parámetro `pii-categories`. `Person` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    
    :::column span="":::
      **Idiomas de documento admitidos**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt`, `pt-br`   
      
   :::column-end:::
:::row-end:::

### <a name="category-persontype"></a>Categoría: PersonType

Este categoría contiene la entidad siguiente:


:::row:::
    :::column span="":::
        **Entidad**

        PersonType

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Tipos de trabajo o roles mantenido por una persona.

        Para obtener esta categoría de entidad, agregue `PersonType` al parámetro `pii-categories`. `PersonType` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::

    :::column span="":::
      **Idiomas de documento admitidos**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt`, `pt-br`  
      
   :::column-end:::
:::row-end:::

### <a name="category-phonenumber"></a>Categoría: PhoneNumber

Este categoría contiene la entidad siguiente:

:::row:::
    :::column span="":::
        **Entidad**

        PhoneNumber

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Números de teléfono (solo números de teléfono de EE. UU y la UE). También se devuelve con `domain=phi`.

        Para obtener esta categoría de entidad, agregue `PhoneNumber` al parámetro `pii-categories`. `PhoneNumber` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::

    :::column span="":::
      **Idiomas de documento admitidos**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt` `pt-br`
      
   :::column-end:::

:::row-end:::


### <a name="category-organization"></a>Categoría: Organización

Este categoría contiene la entidad siguiente:

:::row:::
    :::column span="":::
        **Entidad**

        Organización

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Empresas, grupos políticos, bandas musicales, clubs deportivos, organismos gubernamentales y organizaciones públicas. Las nacionalidades y las religiones no se incluyen en este tipo de entidad. También se devuelve con `domain=phi`.

        Para obtener esta categoría de entidad, agregue `Organization` al parámetro `pii-categories`. `Organization` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::

    :::column span="":::
      **Idiomas de documento admitidos**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt`, `pt-br`  
      
   :::column-end:::

:::row-end:::

#### <a name="subcategories"></a>Subcategorías

La entidad de esta categoría puede tener las subcategorías siguientes.

:::row:::
    :::column span="":::
        **Subcategoría de la entidad**

        Medicina    

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Empresas y grupos médicos.

        Para obtener esta categoría de entidad, agregue `OrganizationMedical` al parámetro `pii-categories`. `OrganizationMedical` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::

    :::column span="":::
      **Idiomas de documento admitidos**

      `en`   
      
   :::column-end:::

:::row-end:::
:::row:::
    :::column span="":::

        Bolsa de valores

    :::column-end:::
    :::column span="2":::

        Grupos de bolsa de valores. 

        Para obtener esta categoría de entidad, agregue `OrganizationStockExchange` al parámetro `pii-categories`. `OrganizationStockExchange` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::

    :::column span="":::

      `en`   
      
   :::column-end:::

:::row-end:::
:::row:::
    :::column span="":::

        Deportes

    :::column-end:::
    :::column span="2":::

        Organizaciones relacionadas con los deportes.

        Para obtener esta categoría de entidad, agregue `OrganizationSports` al parámetro `pii-categories`. `OrganizationSports` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::

    :::column span="":::

      `en`   
      
   :::column-end:::

:::row-end:::


### <a name="category-address"></a>Categoría: Dirección

Este categoría contiene la entidad siguiente:

:::row:::
    :::column span="":::
        **Entidad**

        Dirección

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Dirección de correo postal completa. También se devuelve con `domain=phi`.

        Para obtener esta categoría de entidad, agregue `Address` al parámetro `pii-categories`. `Address` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::

    :::column span="":::
      **Idiomas de documento admitidos**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt`, `pt-br`
      
    :::column-end:::

:::row-end:::

### <a name="category-email"></a>Categoría: Email

Este categoría contiene la entidad siguiente:

:::row:::
    :::column span="":::
        **Entidad**

        Email

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Direcciones de correo. También se devuelve con `domain=phi`.
      
        Para obtener esta categoría de entidad, agregue `Email` al parámetro `pii-categories`. `Email` se devolverá en la respuesta de la API si se detecta.

    :::column-end:::
    :::column span="":::
      **Idiomas de documento admitidos**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt`, `pt-br`
      
    :::column-end:::
:::row-end:::


### <a name="category-url"></a>Categoría: URL

Este categoría contiene la entidad siguiente:

:::row:::
    :::column span="":::
        **Entidad**

        URL

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Direcciones URL de sitios web. También se devuelve con `domain=phi`.

        Para obtener esta categoría de entidad, agregue `URL` al parámetro `pii-categories`. `URL` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::

    :::column span="":::
      **Idiomas de documento admitidos**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt`, `pt-br`
      
    :::column-end:::

:::row-end:::

### <a name="category-ip"></a>Categoría: IP

Este categoría contiene la entidad siguiente:

:::row:::
    :::column span="":::
        **Entidad**

        IP

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Direcciones IP de red. También se devuelve con `domain=phi`.

        Para obtener esta categoría de entidad, agregue `IP` al parámetro `pii-categories`. `IP` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::

    :::column span="":::
      **Idiomas de documento admitidos**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt`, `pt-br`
      
    :::column-end:::
:::row-end:::

### <a name="category-datetime"></a>Categoría: DateTime

Este categoría contiene las entidades siguientes:

:::row:::
    :::column span="":::
        **Entidad**

        DateTime

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Fechas y horas del día. 

        Para obtener esta categoría de entidad, agregue `DateTime` al parámetro `pii-categories`. `DateTime` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
:::column span="":::
      **Idiomas de documento admitidos**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt`, `pt-br`
      
   :::column-end:::
:::row-end:::

#### <a name="subcategories"></a>Subcategorías

La entidad de esta categoría puede tener las subcategorías siguientes.

:::row:::
    :::column span="":::
        **Subcategoría de la entidad**

        Fecha

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Fechas calendario. También se devuelve con `domain=phi`.

        Para obtener esta categoría de entidad, agregue `Date` al parámetro `pii-categories`. `Date` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="2":::
      **Idiomas de documento admitidos**
      
      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt`, `pt-br`   
      
    :::column-end:::
:::row-end:::

### <a name="category-quantity"></a>Categoría: Cantidad

Este categoría contiene las entidades siguientes:

:::row:::
    :::column span="":::
        **Entidad**

        Cantidad

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Números y cantidades numéricas.

        Para obtener esta categoría de entidad, agregue `Quantity` al parámetro `pii-categories`. `Quantity` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="2":::
      **Idiomas de documento admitidos**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt`, `pt-br`
      
    :::column-end:::
:::row-end:::

#### <a name="subcategories"></a>Subcategorías

La entidad de esta categoría puede tener las subcategorías siguientes.

:::row:::
    :::column span="":::
        **Subcategoría de la entidad**

        Age

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Edades. 

        Para obtener esta categoría de entidad, agregue `Age` al parámetro `pii-categories`. `Age` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="2":::
        **Idiomas de documento admitidos**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt`, `pt-br`  
      
   :::column-end:::
:::row-end:::

### <a name="azure-information"></a>Información de Azure

Estas categorías de entidad incluyen información de Azure identificable entre la que se incluye la información de autenticación y las cadenas de conexión. No se devuelve con el parámetro `domain=phi`.

:::row:::
    :::column span="":::
        **Entidad**

        Clave de autorización de Azure DocumentDB 

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Clave de autorización para un servidor de Azure Cosmos DB.   

        Para obtener esta categoría de entidad, agregue `AzureDocumentDBAuthKey` al parámetro `pii-categories`. `AzureDocumentDBAuthKey` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::
      **Idiomas de documento admitidos**

      `en` 

    :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Cadena de conexión de base de datos de IAAS de Azure y cadena de conexión de Azure SQL.
        

    :::column-end:::
    :::column span="2":::

        Cadena de conexión de una base de datos de infraestructura como servicio (IaaS) de Azure y una cadena de conexión SQL.

        Para obtener esta categoría de entidad, agregue `AzureIAASDatabaseConnectionAndSQLString` al parámetro `pii-categories`. `AzureIAASDatabaseConnectionAndSQLString` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `en` 

    :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Cadena de conexión de Azure IoT  

    :::column-end:::
    :::column span="2":::

        Cadena de conexión para Azure IoT. 
      
        Para obtener esta categoría de entidad, agregue `AzureIoTConnectionString` al parámetro `pii-categories`. `AzureIoTConnectionString` se devolverá en la respuesta de la API si se detecta.

    :::column-end:::
    :::column span="":::

      `en` 

    :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Contraseña de configuración de publicación de Azure  

    :::column-end:::
    :::column span="2":::

        Contraseña para la configuración de publicación de Azure.

        Para obtener esta categoría de entidad, agregue `AzurePublishSettingPassword` al parámetro `pii-categories`. `AzurePublishSettingPassword` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `en` 

    :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Cadena de conexión de Azure Redis Cache 

    :::column-end:::
    :::column span="2":::

        Cadena de conexión para una instancia de Redis Cache.

        Para obtener esta categoría de entidad, agregue `AzureRedisCacheString` al parámetro `pii-categories`. `AzureRedisCacheString` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `en` 

    :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        SAS de Azure 

    :::column-end:::
    :::column span="2":::

        Cadena de conexión para software como servicio (SaaS) de Azure.

        Para obtener esta categoría de entidad, agregue `AzureSAS` al parámetro `pii-categories`. `AzureSAS` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `en` 

    :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Cadena de conexión de Azure Service Bus

    :::column-end:::
    :::column span="2":::

        Cadena de conexión de una instancia de Azure Service Bus.

        Para obtener esta categoría de entidad, agregue `AzureServiceBusString` al parámetro `pii-categories`. `AzureServiceBusString` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `en` 

    :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Clave de la cuenta de Azure Storage 

    :::column-end:::
    :::column span="2":::

        Clave de una cuenta de Azure Storage. 

        Para obtener esta categoría de entidad, agregue `AzureStorageAccountKey` al parámetro `pii-categories`. `AzureStorageAccountKey` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `en` 

    :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Clave de la cuenta de Azure Storage (genérica)

    :::column-end:::
    :::column span="2":::

        Clave de cuenta genérica para una cuenta de Azure Storage.

        Para obtener esta categoría de entidad, agregue `AzureStorageAccountGeneric` al parámetro `pii-categories`. `AzureStorageAccountGeneric` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `en` 

    :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Cadena de conexión de SQL Server 

    :::column-end:::
    :::column span="2":::

        Cadena de conexión para un equipo donde se ejecuta SQL Server.

        Para obtener esta categoría de entidad, agregue `SQLServerConnectionString` al parámetro `pii-categories`. `SQLServerConnectionString` se devolverá en la respuesta de la API si se detecta.
      
    :::column-end:::
    :::column span="":::

      `en` 

    :::column-end:::
:::row-end:::

### <a name="identification"></a>Identificación

[!INCLUDE [supported identification entities](./identification-entities.md)]
