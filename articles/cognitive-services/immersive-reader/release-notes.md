---
title: Notas de la versión del SDK del Lector inmersivo
titleSuffix: Azure Applied AI Services
description: Obtenga más información sobre las novedades del SDK de JavaScript del Lector inmersivo.
services: cognitive-services
author: dylankil
manager: guillasi
ms.service: cognitive-services
ms.subservice: immersive-reader
ms.topic: reference
ms.date: 10/12/2020
ms.author: dylankil
ms.openlocfilehash: 8aa1282234a324fae294274e7720c6138d9b059b
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110370777"
---
# <a name="immersive-reader-javascript-sdk-release-notes"></a>Notas de la versión del SDK de JavaScript para el Lector inmersivo

## <a name="version-110"></a>Versión 1.1.0

Esta versión contiene nuevas características, correcciones de vulnerabilidades de seguridad, correcciones de errores, actualizaciones de ejemplos de código y opciones de configuración.

#### <a name="new-features"></a>Características nuevas

* Posibilidad de guardar y cargar las preferencias de usuario en diferentes exploradores y dispositivos.
* Posibilidad de configurar opciones de visualización predeterminadas.
* Se ha agregado una opción para establecer el idioma de traducción y permitir la traducción de palabras y documentos al iniciar el Lector inmersivo.
* Se ha agregado compatibilidad con la configuración de Lectura en voz alta mediante opciones.
* Se ha agregado la posibilidad deshabilitar la primera experiencia de ejecución.
* Se ha agregado ImmersiveReaderView para UWP.

#### <a name="improvements"></a>Mejoras

* Se ha actualizado el código HTML de ejemplo de Android para que funcione con el SDK más reciente.
* Se ha actualizado la respuesta de inicio para que devuelva el número de caracteres procesados.
* Se han actualización los ejemplos de código para usar v1.1.0.
* No se permite la llamada a launchAsync cuando ya se está cargando.
* Comprobación de contenido no válido mediante la omisión de mensajes en los que los datos no son una cadena.
* Encapsulado de llamada a ventana en una cláusula If para comprobar la compatibilidad del explorador de Promise.

#### <a name="fixes"></a>Correcciones

* Se ha corregido dependabot mediante la eliminación de yarn.lock de gitignore.
* Se ha corregido una vulnerabilidad de seguridad mediante la actualización de Pug a v3.0.0 en el ejemplo de código de quickstart-nodejs.
* Se han corregido varias vulnerabilidades de seguridad mediante la actualización de los paquetes de Jest y TypeScript.
* Se ha corregido una vulnerabilidad de seguridad mediante la actualización de Microsoft.IdentityModel.Clients.ActiveDirectory a v5.2.0.

<br>

## <a name="version-100"></a>Versión 1.0.0

Esta versión contiene cambios importantes, nuevas características, mejoras de ejemplos de código y correcciones de errores.

#### <a name="breaking-changes"></a>Últimos cambios

* Se exigen el token y el subdominio de Azure AD y se dejan de usar los tokens empleados en versiones anteriores.
* Se establece CookiePolicy como deshabilitado. La retención de las preferencias de usuario está deshabilitada de forma predeterminada. El Lector se inicia siempre con la configuración predeterminada, a menos que CookiePolicy se establezca en habilitado.

#### <a name="new-features"></a>Características nuevas

* Se ha agregado compatibilidad para habilitar o deshabilitar las cookies.
* Se ha agregado un ejemplo de código de inicio rápido de Kotlin para Android.
* Se ha agregado un ejemplo de código de inicio rápido de Java para Android.
* Se ha agregado un ejemplo de código de inicio rápido de Node.js.

#### <a name="improvements"></a>Mejoras

* Se ha actualizado el archivo README.md avanzado de Node.js.
* Se ha cambiado el ejemplo de código de Python de avanzado a inicio rápido.
* Se ha trasladado el ejemplo de código de Swift para iOS a js/samples.
* Se han actualizado los ejemplos de código para usar v1.0.0.

#### <a name="fixes"></a>Correcciones

* Se ha corregido el ejemplo de código avanzado de Node.js.
* Se han agregado archivos que faltan en advanced-csharp-multiple-resources.
* Se ha quitado en-us de los hipervínculos.

<br>

## <a name="version-003"></a>Versión 0.0.3

Esta versión contiene nuevas características, mejoras de los ejemplos de código, correcciones de vulnerabilidades de seguridad y correcciones de errores.

#### <a name="new-features"></a>Características nuevas

* Se ha agregado un ejemplo de código de Swift para iOS.
* Se ha agregado un ejemplo de código avanzado de C# que muestra el uso de varios recursos. 
* Se ha agregado compatibilidad para deshabilitar la característica de alternancia de pantalla completa.
* Se ha agregado compatibilidad para ocultar el botón de salida de la aplicación del Lector inmersivo.
* Se ha agregado una función de devolución de llamada que pueda usar la aplicación host al salir del Lector inmersivo.
* Se han actualizado los ejemplos de código para usar la autenticación de Azure Active Directory.

#### <a name="improvements"></a>Mejoras

* Se ha actualizado el ejemplo de código avanzado de C# para incluir documentos de Word.
* Se han actualizado los ejemplos de código para usar v0.0.3.

#### <a name="fixes"></a>Correcciones

* Se ha actualizado lodash a la versión 4.17.14 para corregir una vulnerabilidad de seguridad.
* Se ha actualizado la biblioteca MSAL de C# para corregir una vulnerabilidad de seguridad.
* Se ha actualizado mixin-deep a la versión 1.3.2 para corregir una vulnerabilidad de seguridad.
* Se han actualizado jest, webpack y webpack-cli, que usaban versiones vulnerables de set-value y mixin-deep, para corregir una vulnerabilidad de seguridad.

<br>

## <a name="version-002"></a>Versión 0.0.2

Esta versión contiene nuevas características, mejoras de los ejemplos de código, correcciones de vulnerabilidades de seguridad y correcciones de errores.

#### <a name="new-features"></a>Características nuevas

* Se ha agregado un ejemplo de código avanzado de Python.
* Se ha agregado un ejemplo de código de inicio rápido de Java.
* Se ha agregado un ejemplo de código simple.

#### <a name="improvements"></a>Mejoras

* Se ha cambiado el nombre de resourceName por cogSvcsSubdomain.
* Se han sacado los secretos del código y se usan variables de entorno.
* Se han actualizado los ejemplos de código para usar v0.0.2.

#### <a name="fixes"></a>Correcciones

* Se han corregido los errores de accesibilidad del botón del Lector inmersivo.
* Se ha corregido el desplazamiento interrumpido.
* Se ha actualizado el paquete de Handlebars a la versión 4.1.2 para solucionar una vulnerabilidad de seguridad.
* Se han corregido errores de las pruebas unitarias del SDK.
* Se han corregido errores de compatibilidad de JavaScript en Internet Explorer 11.
* Se han actualizado las direcciones URL del SDK.

<br>

## <a name="version-001"></a>Versión 0.0.1

La versión inicial del SDK de JavaScript del Lector inmersivo.

* Se ha agregado el SDK de JavaScript del Lector inmersivo.
* Se ha agregado compatibilidad para especificar el idioma de la interfaz de usuario.
* Se ha agregado un tiempo de espera para determinar cuándo debe producirse un error de tiempo de espera en la función launchAsync.
* Se ha agregado compatibilidad para especificar el índice z del iframe del Lector inmersivo.
* Se ha agregado compatibilidad para usar una etiqueta de vista web en lugar de un iframe, para la compatibilidad con aplicaciones de Chrome.
* Se han agregado pruebas unitarias del SDK.
* Se ha agregado un ejemplo de código avanzado de Node.js.
* Se ha agregado un ejemplo de código avanzado de C#.
* Se ha agregado un ejemplo de código de inicio rápido de C#.
* Se han agregado archivos de configuración de paquetes, archivos YARN y otros archivos de compilación.
* Se han agregado archivos de configuración de Git.
* Se han agregado archivos README.md a ejemplos de código y al SDK.
* Se ha agregado una licencia de MIT.
* Se han agregado instrucciones de colaborador.
* Se han agregado recursos de SVG para el botón de icono estático.

## <a name="next-steps"></a>Pasos siguientes

Introducción a Lector inmersivo:

* Lea la [referencia sobre la biblioteca cliente de Lector inmersivo](./reference.md).
* Explore la [biblioteca cliente del Lector inmersivo en GitHub](https://github.com/microsoft/immersive-reader-sdk).
