---
title: Configurar la traducción
titleSuffix: Azure Applied AI Services
description: En este artículo se muestra cómo configurar las distintas opciones de traducción.
author: metanMSFT
manager: guillasi
ms.service: cognitive-services
ms.subservice: immersive-reader
ms.topic: how-to
ms.date: 06/29/2020
ms.author: metang
ms.openlocfilehash: 7ffcc613a9a1e5222f0f812b4a2a9e0eb774ddbc
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110377769"
---
# <a name="how-to-configure-translation"></a>Configuración de la traducción

En este artículo se muestra cómo configurar las distintas opciones de traducción en el Lector inmersivo.

## <a name="configure-translation-language"></a>Configuración del idioma de traducción

El parámetro `options` contiene todas las marcas que se pueden usar para configurar la traducción. Establezca el parámetro `language` en el idioma al que desea traducir. Consulte la [Compatibilidad de idiomas](./language-support.md) para obtener la lista completa de los idiomas admitidos.

```typescript
const options = {
    translationOptions: {
        language: 'fr-FR'
    }
};

ImmersiveReader.launchAsync(YOUR_TOKEN, YOUR_SUBDOMAIN, YOUR_DATA, options);
```

## <a name="automatically-translate-the-document-on-load"></a>Traducción automática del documento en carga

Establezca `autoEnableDocumentTranslation` en `true` para habilitar la traducción automática de todo el documento cuando se cargue el Lector inmersivo.

```typescript
const options = {
    translationOptions: {
        autoEnableDocumentTranslation: true
    }
};
```

## <a name="automatically-enable-word-translation"></a>Habilitación automática de la traducción de palabras

Establezca `autoEnableWordTranslation` en `true` para habilitar la traducción de una sola palabra.

```typescript
const options = {
    translationOptions: {
        autoEnableWordTranslation: true
    }
};
```

## <a name="next-steps"></a>Pasos siguientes

* Lea la [Referencia del SDK del Lector inmersivo](./reference.md)