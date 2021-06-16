---
title: Preparación del contenido HTML para el Lector inmersivo
titleSuffix: Azure Applied AI Services
description: Aprenda a iniciar el Lector inmersivo con HTML, JavaScript, Python, Android o iOS. El Lector inmersivo emplea técnicas demostradas para mejorar la comprensión lectora de estudiantes de idiomas, lectores en ciernes y personas con diferencias en el aprendizaje.
author: erhopf
manager: nitinme
ms.service: applied-ai-services
ms.subservice: immersive-reader
ms.topic: include
ms.date: 03/04/2021
ms.author: erhopf
ms.openlocfilehash: 4efadfaed4980afd035f36d530f87a971eeb4964
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111888201"
---
# <a name="how-to-prepare-html-content-for-immersive-reader"></a>Preparación del contenido HTML para el Lector inmersivo

En este artículo se muestra cómo estructurar el código HTML y recuperar el contenido para que lo pueda usar el Lector inmersivo.

## <a name="prepare-the-html-content"></a>Preparación del contenido HTML

Coloque el contenido que desea representar en el Lector inmersivo dentro de un elemento contenedor. Asegúrese de que el elemento contenedor tenga un `id`único. El Lector inmersivo brinda compatibilidad con los elementos HTML básicos, consulte la [referencia](reference.md#html-support) para obtener más información.

```html
<div id='immersive-reader-content'>
    <b>Bold</b>
    <i>Italic</i>
    <u>Underline</u>
    <strike>Strikethrough</strike>
    <code>Code</code>
    <sup>Superscript</sup>
    <sub>Subscript</sub>
    <ul><li>Unordered lists</li></ul>
    <ol><li>Ordered lists</li></ol>
</div>
```

## <a name="get-the-html-content-in-javascript"></a>Obtención del contenido HTML en JavaScript

Use el `id` del elemento contenedor para obtener el contenido HTML en el código de JavaScript.

```javascript
const htmlContent = document.getElementById('immersive-reader-content').innerHTML;
```

## <a name="launch-the-immersive-reader-with-your-html-content"></a>Inicio del Lector inmersivo con contenido HTML

Al llamar a `ImmersiveReader.launchAsync`, establezca la propiedad `mimeType` del fragmento en `text/html` para habilitar la representación HTML.

```javascript
const data = {
    chunks: [{
        content: htmlContent,
        mimeType: 'text/html'
    }]
};

ImmersiveReader.launchAsync(YOUR_TOKEN, YOUR_SUBDOMAIN, data, YOUR_OPTIONS);
```

## <a name="next-steps"></a>Pasos siguientes

* Lea la [Referencia del SDK del Lector inmersivo](reference.md)