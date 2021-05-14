---
title: 'Inicio rápido de la CLI de Voz: servicio Voz'
titleSuffix: Azure Cognitive Services
description: Introducción a la CLI de Voz de Azure. Es posible interactuar con los servicios Voz como conversión de voz en texto, texto a voz y traducción de voz sin tener que escribir ni una sola línea de código.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: quickstart
ms.date: 04/28/2021
ms.author: trbye
ms.openlocfilehash: e0a2646dad0be024573c8d02bf4e2749eaeb20c2
ms.sourcegitcommit: 49bd8e68bd1aff789766c24b91f957f6b4bf5a9b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108227997"
---
# <a name="get-started-with-the-azure-speech-cli"></a>Introducción a la CLI de Voz de Azure

En este artículo, aprenderá a usar la CLI de voz, una interfaz de línea de comandos, para tener acceso a servicios Voz como conversión de voz en texto, texto a voz y traducción de voz sin necesidad de escribir código. La CLI de Voz está lista para producción y se puede usar para automatizar flujos de trabajo sencillos en el servicio Voz mediante scripts de shell o `.bat`.

En este artículo se supone que tiene conocimientos prácticos del símbolo del sistema, un terminal o PowerShell.

[!INCLUDE [](includes/spx-setup.md)]

## <a name="basic-usage"></a>Uso básico

En esta sección se muestran algunos comandos básicos de SPX que a menudo resultan útiles para la prueba y la experimentación iniciales. Para empezar, vea la ayuda integrada en la herramienta, para lo que debe ejecutar el siguiente comando.

```console
spx
```

Puede buscar temas de ayuda por palabra clave. Por ejemplo, escriba el siguiente comando para ver una lista de ejemplos de uso de la CLI de Voz:

```console
spx help find --topics "examples"
```

Escriba el siguiente comando para ver las opciones del comando recognize:

```console
spx help recognize
```

Los comandos de ayuda adicionales se enumeran en la columna derecha. Escríbalos si desea obtener ayuda detallada acerca de los subcomandos.

## <a name="speech-to-text-speech-recognition"></a>Conversión de voz a texto (reconocimiento de voz)

Vamos a usar la CLI de Voz para convertir voz en texto (reconocimiento de voz), para lo que usaremos el micrófono predeterminado del sistema. Una vez que se escriba el comando, SPX comenzará a escuchar el audio en el dispositivo de entrada activo actual y se detendrá cuando se presione **ENTRAR**. La voz grabada se reconoce y se convierte a texto en la salida de la consola.

>[!IMPORTANT]
> Si usa un contenedor de Docker, `--microphone` no funcionará.

Ejecute este comando:

```console
spx recognize --microphone
```

Con la CLI de Voz también se puede reconocer voz procedente de un archivo de audio.

```console
spx recognize --file /path/to/file.wav
```

> [!TIP]
> Si va a reconocer la voz a partir de un archivo de audio en un contenedor de Docker, asegúrese de que el archivo de audio se encuentra en el directorio que ha montado en el paso anterior.

No olvide que, si se atasca o desea obtener más información sobre las opciones de reconocimiento de la CLI de Voz, solo tiene que escribir:

```console
spx help recognize
```

## <a name="text-to-speech-speech-synthesis"></a>Conversión de texto a voz (síntesis de voz)

Al ejecutar el comando siguiente, se toma el texto como entrada y se genera la voz sintetizada en el dispositivo de salida activo actual (por ejemplo, los altavoces del equipo).

```console
spx synthesize --text "Testing synthesis using the Speech CLI" --speakers
```

La salida sintetizada también se puede guardar en un archivo. En este ejemplo, se va a crear un archivo denominado `my-sample.wav` en el directorio en el que se ejecuta el comando.

```console
spx synthesize --text "Enjoy using the Speech CLI." --audio output my-sample.wav
```

En estos ejemplos se presupone que las pruebas se realizan en inglés. Sin embargo, se admite la síntesis de voz en muchos idiomas. Para desplegar una lista completa de voces, use este comando o visite la [página de idiomas compatibles](./language-support.md).

```console
spx synthesize --voices
```

Aquí se muestra cómo se usa una de las voces que ha descubierto.

```console
spx synthesize --text "Bienvenue chez moi." --voice fr-CA-Caroline --speakers
```

No olvide que, si se atasca o desea obtener más información sobre las opciones de síntesis de la CLI de Voz, solo tiene que escribir:

```console
spx help synthesize
```

## <a name="speech-to-text-translation"></a>Conversión de voz en texto

Con la CLI de Voz, también se puede realizar una conversión de voz a texto. Ejecute este comando para capturar audio desde el micrófono predeterminado y generar la conversión en texto. Tenga en cuenta que debe especificar el idioma `source` y `target` con el comando `translate`.

```console
spx translate --microphone --source en-US --target ru-RU
```

Al realizar la conversión a varios idiomas, separe los códigos de idioma con `;`.

```console
spx translate --microphone --source en-US --target ru-RU;fr-FR;es-ES
```

Si desea guardar la salida de la conversión, use la marca de `--output`. En este ejemplo, también se realizará la lectura de un archivo.

```console
spx translate --file /some/file/path/input.wav --source en-US --target ru-RU --output file /some/file/path/russian_translation.txt
```

> [!NOTE]
> Consulte el [artículo sobre los idiomas y las configuraciones regionales](language-support.md) para obtener una lista de todos los idiomas compatibles con sus códigos de configuración regional correspondientes.

No olvide que, si se atasca o desea obtener más información sobre las opciones de conversión de la CLI de Voz, solo tiene que escribir:

```console
spx help translate
```

## <a name="next-steps"></a>Pasos siguientes

* [Opciones de configuración de la CLI de Voz](./spx-data-store-configuration.md)
* [Operaciones por lotes con la CLI de Voz](./spx-batch-operations.md)
