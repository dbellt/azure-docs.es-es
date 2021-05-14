---
title: Diseño de archivos para la conversión
description: Recomendaciones sobre cómo colocar archivos en el contenedor de entrada de forma óptima.
author: MalcolmTyrrell
ms.author: matyrr
ms.date: 09/03/2020
ms.topic: how-to
ms.openlocfilehash: ab01143fcab5ceb5468ef8fde233905b8d3e05d1
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108124552"
---
# <a name="laying-out-files-for-conversion"></a>Diseño de archivos para la conversión

Para procesar correctamente un recurso, el servicio de conversión debe ser capaz de encontrar todos los archivos de entrada.
Estos constan del archivo de recursos principal que se va a convertir y, normalmente, otros archivos a los que hacen referencia las rutas de acceso dentro del archivo de recursos.
A la solicitud para convertir un recurso se le proporcionan dos parámetros que determinan cómo encuentra el servicio de conversión estos archivos: El parámetro `settings.inputLocation.blobPrefix` (que es opcional) y el parámetro `settings.inputLocation.relativeInputAssetPath`.
Estos parámetros se documentan completamente en la página [API REST de conversión](conversion-rest-api.md).
Al colocar archivos, es importante tener en cuenta que `BlobPrefix` determina el conjunto completo de archivos que están disponibles para el servicio de conversión al procesar el recurso.

> [!Note]
> El servicio descargará todos los archivos en input.BlobPrefix. Asegúrese de que los nombres de archivo y las rutas de acceso no superan los [límites de longitud de la ruta de acceso de Windows](/windows/win32/fileio/maximum-file-path-limitation) para evitar problemas en el servicio. 

## <a name="placing-files-so-they-can-be-found"></a>Colocación de archivos para que se puedan encontrar

Cuando un recurso de origen hace uso de archivos externos, las rutas de acceso a esos archivos se almacenan en el recurso.
El servicio de conversión tiene que interpretarlas en un sistema de archivos diferente del sistema de archivos original del recurso.
Si las rutas de acceso se almacenan como rutas de acceso relativas y la ubicación relativa entre el recurso de origen y el archivo al que hace referencia no cambia, es fácil que el servicio de conversión encuentre dicho archivo.

> [!Note]
> Se recomienda colocar los archivos en el contenedor de entrada para que las ubicaciones relativas de los archivos sean las mismas que tenían cuando se creó el recurso.

> [!Note]
> Dé preferencia a crear recursos que incluyan rutas de acceso relativas.
> En el tutorial sobre la [configuración de materiales para 3ds Max](../../tutorials/modeling/3dsmax-material-setup.md) se ofrece un ejemplo de 3ds Max de cómo asegurarse de que un recurso utiliza rutas de acceso relativas.

## <a name="finding-textures"></a>Búsqueda de texturas

Debido a las muchas maneras en que se pueden generar los recursos, el servicio de conversión debe ser flexible.
En concreto, debe que controlar situaciones en las que las rutas de acceso del recurso y la ubicación de las texturas no coinciden con precisión.
Este es el caso cuando se generan recursos que contienen rutas de acceso absolutas, ya que estas rutas nunca coincidirán con el sistema de archivos utilizado por el servicio de conversión.
Para tratar esta situación entre otras, se usa el principio de máximo esfuerzo para buscar texturas.

El algoritmo para buscar texturas es el siguiente: Dada una ruta de acceso almacenada en un recurso, busque el sufijo de subruta de acceso más largo que, cuando se use como ruta de acceso relativa desde la ubicación del recurso de origen, se dirija a un archivo que exista.
Si ninguna de estas subrutas (incluida la ruta de acceso completa) se dirige a un archivo, se considera que la textura falta.

Considere el siguiente sistema de archivos ficticio: 
```
G:\CONVERSION
├───Assets
│   │   myAsset.fbx
│   │   myTexture.png            <- A
│   │
│   └───Textures
│       │   myTexture.png        <- B
│       │
│       └───MyAssetTextures
│               myTexture.png    <- C
│
└───Textures
    │   myTexture.png            <- D
    │
    └───MyAssetTextures
            myTexture.png        <- E
```
Si myAsset.fbx hace referencia a una textura con la ruta de acceso relativa `..\Textures\MyAssetTextures\myTexture.png`, el servicio de conversión usará el archivo E. De hecho, puede usar cualquiera de los archivos, A, C y E, si existen, pero el archivo E es el preferido dado que se encuentra con el sufijo más largo.
Nunca se usarán los archivos B y D, ya que `Textures\myTexture.png` no forma parte de ningún sufijo de la ruta de acceso almacenada.
Si el recurso contiene en cambio las rutas de acceso `H:\Foo\Bar\Textures\MyAssetTextures\myTexture.png` o `..\..\..\Foo\Bar\Textures\MyAssetTextures\myTexture.png`, el servicio de conversión podrá encontrar los archivos A y C, si existen (siendo C el preferido por encima de A). Sin embargo, E no se puede encontrar de esta manera y el archivo tendrá que reubicarse.
Para corregirlo, se podría mover la carpeta Textures situada junto al recurso.

> [!Note]
> Si no se encuentran las texturas, una posible solución es asegurarse de que el recurso sea un elemento relacionado de algún subárbol que las contenga.

## <a name="next-steps"></a>Pasos siguientes

- [Conversión de modelos](model-conversion.md)