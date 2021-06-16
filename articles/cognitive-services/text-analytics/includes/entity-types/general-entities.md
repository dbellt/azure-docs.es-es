---
title: Entidades con nombre generales
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: include
ms.date: 01/15/2021
ms.author: aahi
ms.openlocfilehash: 36f05d2803d4ac347ef9dc8bf8e840e3f4a63cac
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111761288"
---
La característica NER para Text Analytics devuelve las siguientes categorías de entidad generales (no de identificación). Por ejemplo, al enviar solicitudes al punto de conexión `/entities/recognition/general`.


| Category | Descripción                          |
|------------|-------------|--------------------------------------|-------------------------------------------------------------|--------------------------------------|
| [Person](#category-person)     | Nombres de personas.  |
| [PersonType](#category-persontype) | Tipos de trabajo o roles mantenido por una persona. |
| [Ubicación](#category-location)    | Puntos de referencia naturales y humanos, estructuras, características geográficas y entidades geopolíticas. |
| [Organización](#category-organization)  | Empresas, grupos políticos, bandas musicales, clubs deportivos, organismos gubernamentales y organizaciones públicas.  |
| [Evento](#category-event)  | Eventos históricos, sociales y naturales. |
| [Producto](#category-product) | Objetos físicos de varias categorías. |
| [Habilidad](#category-skill) | Capacidad, aptitud o experiencia.  |
| [Dirección](#category-address) | Dirección de correo postal completa.  |
| [Número de teléfono](#category-phonenumber) | Números de teléfono. |
| [Correo electrónico](#category-email) | Direcciones de correo. |
| [URL](#category-url) | Direcciones URL de sitios web. |
| [IP](#category-ip) | Direcciones IP de red. |
| [DateTime](#category-datetime) | Fechas y horas del día. |
| [Cantidad](#category-quantity) | Unidades y medidas numéricas. |


### <a name="category-person"></a>Categoría: Person

Este categoría contiene la entidad siguiente:

:::row:::
    :::column span="":::
        **Entidad**

        Person

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Nombres de personas.
      
    :::column-end:::
    :::column span="2":::
      **Idiomas de documento admitidos**

      `ar`, `cs`, `da`, `nl`, `en`, `fi`, `fr`, `de`, `he`, <br> `hu`, `it`, `ja`, `ko`, `no`, `pl`, `pt-br`, `pt`-`pt`, `ru`, `es`, `sv`, `tr`   
      
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

        Tipos de trabajo o roles que tiene una persona
      
    :::column-end:::
    :::column span="2":::
      **Idiomas de documento admitidos**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt`, `pt-br`  
      
   :::column-end:::
:::row-end:::

### <a name="category-location"></a>Categoría: Location

Este categoría contiene la entidad siguiente:

:::row:::
    :::column span="":::
        **Entidad**

        Location

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Puntos de referencia naturales y humanos, estructuras, características geográficas y entidades geopolíticas.
      
    :::column-end:::
    :::column span="2":::
      **Idiomas de documento admitidos**

      `ar`, `cs`, `da`, `nl`, `en`, `fi`, `fr`, `de`, `he`, `hu`, `it`, `ja`, `ko`, `no`, `pl`, `pt-br`, `pt-pt`, `ru`, `es`, `sv`, `tr`   
      
   :::column-end:::
:::row-end:::

#### <a name="subcategories"></a>Subcategorías

La entidad de esta categoría puede tener las subcategorías siguientes.

:::row:::
    :::column span="":::
        **Subcategoría de la entidad**

        Entidad geopolítica (GPE)

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Ciudades, países o regiones, estados.
      
    :::column-end:::
    :::column span="2":::
      **Idiomas de documento admitidos**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt`, `pt-br`   
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Estructural

    :::column-end:::
    :::column span="2":::

        Estructuras hechas por el hombre. 
      
    :::column-end:::
    :::column span="2":::

      `en`   
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Geográfica

    :::column-end:::
    :::column span="2":::

        Características geográficas y naturales, como ríos, océanos y desiertos.
      
    :::column-end:::
    :::column span="2":::

      `en`   
      
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

        Empresas, grupos políticos, bandas musicales, clubs deportivos, organismos gubernamentales y organizaciones públicas. Las nacionalidades y las religiones no se incluyen en este tipo de entidad.
      
    :::column-end:::
    :::column span="2":::
      **Idiomas de documento admitidos**

      `ar`, `cs`, `da`, `nl`, `en`, `fi`, `fr`, `de`, `he`, `hu`, `it`, `ja`, `ko`, `no`, `pl`, `pt-br`, `pt-pt`, `ru`, `es`, `sv`, `tr`   
      
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
      
    :::column-end:::
    :::column span="2":::
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
      
    :::column-end:::
    :::column span="2":::

      `en`   
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Deportes

    :::column-end:::
    :::column span="2":::

        Organizaciones relacionadas con los deportes.
      
    :::column-end:::
    :::column span="2":::

      `en`   
      
   :::column-end:::
:::row-end:::

### <a name="category-event"></a>Categoría: Evento

Este categoría contiene la entidad siguiente:

:::row:::
    :::column span="":::
        **Entidad**

        Evento

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Eventos históricos, sociales y naturales.
      
    :::column-end:::
    :::column span="2":::
      **Idiomas de documento admitidos**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt` y `pt-br`  
      
   :::column-end:::
:::row-end:::

#### <a name="subcategories"></a>Subcategorías

La entidad de esta categoría puede tener las subcategorías siguientes.

:::row:::
    :::column span="":::
        **Subcategoría de la entidad**

        Cultural

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Eventos culturales y días festivos.
      
    :::column-end:::
    :::column span="2":::
      **Idiomas de documento admitidos**

      `en`   
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Natural

    :::column-end:::
    :::column span="2":::

        Eventos de origen natural.
      
    :::column-end:::
    :::column span="2":::

      `en`   
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Deportes

    :::column-end:::
    :::column span="2":::

        Eventos deportivos.
      
    :::column-end:::
    :::column span="2":::

      `en`   
      
   :::column-end:::
:::row-end:::

### <a name="category-product"></a>Categoría: Producto

Este categoría contiene la entidad siguiente:

:::row:::
    :::column span="":::
        **Entidad**

        Producto

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Objetos físicos de varias categorías.
      
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

        Productos informáticos
    :::column-end:::
    :::column span="2":::
        **Detalles**

        Productos informáticos.
      
    :::column-end:::
    :::column span="2":::
      **Idiomas de documento admitidos**

      `en`   
      
   :::column-end:::
:::row-end:::

### <a name="category-skill"></a>Categoría: Habilidad

Este categoría contiene la entidad siguiente:

:::row:::
    :::column span="":::
        **Entidad**

        Habilidad

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Capacidad, aptitud o experiencia.
      
    :::column-end:::
    :::column span="2":::
      **Idiomas de documento admitidos**

      `en` , `es`, `fr`, `de`, `it`, `pt-pt`, `pt-br` 
      
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

        Dirección de correo postal completa.
      
    :::column-end:::
    :::column span="2":::
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

        Números de teléfono (solo números de teléfono de EE. UU y la UE).
      
    :::column-end:::
    :::column span="2":::
      **Idiomas de documento admitidos**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt` `pt-br`
      
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

        Direcciones de correo.
      
    :::column-end:::
    :::column span="2":::
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

        Direcciones URL de sitios web. 
      
    :::column-end:::
    :::column span="2":::
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

        Direcciones IP de red. 
      
    :::column-end:::
    :::column span="2":::
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
      
    :::column-end:::
    :::column span="2":::
      **Idiomas de documento admitidos**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt`, `pt-br`
      
   :::column-end:::
:::row-end:::

Las entidades de esta categoría pueden tener las subcategorías siguientes

#### <a name="subcategories"></a>Subcategorías

La entidad de esta categoría puede tener las subcategorías siguientes.

:::row:::
    :::column span="":::
        **Subcategoría de la entidad**

        Fecha

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Fechas calendario.
      
    :::column-end:::
    :::column span="2":::
      **Idiomas de documento admitidos**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `pt-pt`, `pt-br`   
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Time

    :::column-end:::
    :::column span="2":::

        Horas del día.
      
    :::column-end:::
    :::column span="2":::

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `pt-pt`, `pt-br`   
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        DateRange

    :::column-end:::
    :::column span="2":::

        Intervalos de fechas.
      
    :::column-end:::
    :::column span="2":::

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `pt-pt`, `pt-br`  
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        TimeRange

    :::column-end:::
    :::column span="2":::

        Intervalos de horas.
      
    :::column-end:::
    :::column span="2":::

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `pt-pt`, `pt-br`  
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Duration

    :::column-end:::
    :::column span="2":::

        Duraciones.
      
    :::column-end:::
    :::column span="2":::

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `pt-pt`, `pt-br`  
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Set

    :::column-end:::
    :::column span="2":::

        Establecer varias veces repetidas.
      
    :::column-end:::
    :::column span="2":::

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `pt-pt`, `pt-br`  
      
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

        Number

    :::column-end:::
    :::column span="2":::
        **Detalles**

        Números.
      
    :::column-end:::
    :::column span="2":::
      **Idiomas de documento admitidos**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `pt-pt`, `pt-br`   
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::
        Porcentaje

    :::column-end:::
    :::column span="2":::

        Porcentajes
      
    :::column-end:::
    :::column span="2":::

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `pt-pt`, `pt-br`   
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::
        Números ordinales

    :::column-end:::
    :::column span="2":::

        Números ordinales.
      
    :::column-end:::
    :::column span="2":::

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `pt-pt`, `pt-br`   
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::
        Age

    :::column-end:::
    :::column span="2":::

        Edades.
      
    :::column-end:::
    :::column span="2":::

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `pt-pt`, `pt-br`   
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::
        Moneda

    :::column-end:::
    :::column span="2":::

        Monedas
      
    :::column-end:::
    :::column span="2":::

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `pt-pt`, `pt-br`   
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::
        Dimensions

    :::column-end:::
    :::column span="2":::

        Dimensiones y medidas.
      
    :::column-end:::
    :::column span="2":::

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `pt-pt`, `pt-br`   
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::
        Temperatura

    :::column-end:::
    :::column span="2":::

        Temperaturas.
      
    :::column-end:::
    :::column span="2":::

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `pt-pt`, `pt-br`   
      
   :::column-end:::
:::row-end:::
