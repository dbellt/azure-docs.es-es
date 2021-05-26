---
title: Almacenamiento de las preferencias del usuario
titleSuffix: Azure Applied AI Services
description: En este artículo se muestra cómo almacenar las preferencias del usuario.
author: metanMSFT
manager: guillasi
ms.service: cognitive-services
ms.subservice: immersive-reader
ms.topic: how-to
ms.date: 06/29/2020
ms.author: metang
ms.openlocfilehash: b3ac552d60515d907157e9d4d778aef9bd7bb380
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110373864"
---
# <a name="how-to-store-user-preferences"></a>Procedimiento: Almacenamiento de las preferencias del usuario

En este artículo se muestra cómo almacenar la configuración de la interfaz del usuario, también conocida usualmente como **Preferencias de usuario**, a través de las opciones [-preferences](./reference.md#options) y [-onPreferencesChanged](./reference.md#options) del SDK del Lector inmersivo.

Cuando la opción de SDK [CookiePolicy](./reference.md#cookiepolicy-options) se establece en *Habilitada*, la aplicación Lector inmersivo almacena las **preferencias de usuario** (tamaño de texto, color del tema, fuente, etc.) en las cookies, que son locales en un explorador y un dispositivo específicos. Cada vez que el usuario inicia el Lector inmersivo en el mismo explorador y dispositivo, se abrirá con las preferencias de usuario desde la última sesión iniciada en ese dispositivo. Sin embargo, si el usuario abre el Lector inmersivo en otro explorador o dispositivo, las opciones se configurarán inicialmente con los valores predeterminados del Lector inmersivo y el usuario tendrá que establecer sus preferencias de nuevo; deberá hacer esto en cada dispositivo que use. Las opciones de SDK `-preferences` y `-onPreferencesChanged` del Lector inmersivo proporcionan a las aplicaciones una manera de llevar las preferencias de un usuario a varios exploradores y dispositivos, de modo que el usuario tenga una experiencia coherente dondequiera que use la aplicación.

En primer lugar, al proporcionar la opción de SDK de devolución de llamada `-onPreferencesChanged` al iniciar la aplicación Lector inmersivo, esta última enviará una cadena `-preferences` a la aplicación host cada vez que el usuario cambie sus preferencias durante la sesión del Lector inmersivo. A continuación, la aplicación host será la responsable de almacenar las preferencias de usuario en su propio sistema. Después, cuando el mismo usuario vuelve a iniciar el Lector inmersivo, la aplicación host puede recuperar las preferencias de usuario del almacenamiento y suministrarlas como una opción de SDK de cadena `-preferences` al iniciar la aplicación Lector inmersivo, de modo que se restauren las preferencias del usuario.

Esta funcionalidad se puede usar como medio alternativo para almacenar las **preferencias de usuario** cuando el uso de cookies no sea deseable o factible.

> [!CAUTION]
> **IMPORTANTE** No intente cambiar mediante programación los valores de la cadena `-preferences` enviada a la aplicación del Lector inmersivo o desde ella, ya que puede provocar un comportamiento inesperado y dar lugar a una experiencia de usuario degradada para los clientes. Recuerde que las aplicaciones host nunca deben asignar un valor personalizado a la cadena `-preferences` ni manipularla. Al utilizar la opción de cadena `-preferences`, use solo el valor exacto que se devolvió en la opción de devolución de llamada `-onPreferencesChanged`.

## <a name="how-to-enable-storing-user-preferences"></a>Cómo habilitar el almacenamiento de preferencias del usuario

El parámetro [launchAsync](./reference.md#launchasync) del SDK de Lector inmersivo `options` contiene la devolución de llamada `-onPreferencesChanged`. Se llamará a esta función siempre que el usuario cambie sus preferencias. El parámetro `value` contiene una cadena, que representa las preferencias actuales del usuario. A continuación, la aplicación host almacena la cadena del usuario.

```typescript
const options = {
    onPreferencesChanged: (value: string) => {
        // Store user preferences here
    }
};

ImmersiveReader.launchAsync(YOUR_TOKEN, YOUR_SUBDOMAIN, YOUR_DATA, options);
```

## <a name="how-to-load-user-preferences-into-the-immersive-reader"></a>Cómo cargar las preferencias del usuario en el Lector inmersivo

Pase las preferencias del usuario al Lector inmersivo mediante la opción `-preferences`. A continuación encontrará un ejemplo trivial para almacenar y cargar las preferencias del usuario:

```typescript
const storedUserPreferences = localStorage.getItem("USER_PREFERENCES");
let userPreferences = storedUserPreferences === null ? null : storedUserPreferences;
const options = {
    preferences: userPreferences,
    onPreferencesChanged: (value: string) => {
        userPreferences = value;
        localStorage.setItem("USER_PREFERENCES", userPreferences);
    }
};
```

## <a name="next-steps"></a>Pasos siguientes

* Lea la [Referencia del SDK del Lector inmersivo](./reference.md)