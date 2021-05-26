---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 04/03/2020
ms.author: trbye
ms.openlocfilehash: 1fe353c4a006dba23eda104ba083e0cc20cd2559
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110059185"
---
:::row:::
    :::column span="3":::
        El SDK de voz solo admite **Ubuntu 16.04/18.04/20.04**, **Debian 9/10**, **Red Hat Enterprise Linux (RHEL) 7/8** y **CentOS 7/8** en las siguientes arquitecturas de destino cuando se usa con Linux:
    :::column-end:::
    :::column:::
        <br>
        <div class="icon is-large">
            <img alt="Linux" src="https://docs.microsoft.com/media/logos/logo_linux-color.svg" width="60px">
        </div>
    :::column-end:::
:::row-end:::

- x86 (Debian/Ubuntu), x64, ARM32 (Debian/Ubuntu) y ARM64 (Debian/Ubuntu) para C++
- x64, ARM32 (Debian/Ubuntu) y ARM64 (Debian/Ubuntu) para Java
- x64, ARM32 (Debian/Ubuntu), y ARM64 (Debian/Ubuntu) para .NET Core
- x64 para Python

> [!IMPORTANT]
> Para C# en Linux ARM64, se requiere .NET Core 3.x (paquete dotnet-sdk-3.x).

> [!NOTE]
> Para usar el SDK de voz en Alpine Linux, cree un entorno chroot en Debian como se documenta en la wiki de Alpine Linux en los [programas glibc en ejecución](https://wiki.alpinelinux.org/wiki/Running_glibc_programs) y después siga las instrucciones de Debian aquí.

### <a name="system-requirements"></a>Requisitos del sistema

En el caso de una aplicación nativa, el SDK de Voz se basa en `libMicrosoft.CognitiveServices.Speech.core.so`. Asegúrese de que la arquitectura de destino (x86, x64) coincide con la aplicación. Dependiendo de la versión de Linux, es posible que se requieran dependencias adicionales.

- Las bibliotecas compartidas de la biblioteca GNU C (incluida la biblioteca de programación de subprocesos de POSIX, `libpthreads`)
- La biblioteca OpenSSL (`libssl.so.1.0.0` o `libssl.so.1.0.2`)
- La biblioteca compartida para las aplicaciones ALSA (`libasound.so.2`)

# <a name="ubuntu-160418042004"></a>[Ubuntu 16.04/18.04/20.04](#tab/ubuntu)

```Bash
sudo apt-get update
sudo apt-get install build-essential libssl1.0.0 libasound2 wget
```

> [!NOTE]
> Si libssl1.0.x no está disponible, instale en su lugar libssl1.1.

# <a name="debian-910"></a>[Debian 9/10](#tab/debian)

```Bash
sudo apt-get update
sudo apt-get install build-essential libssl1.0.2 libasound2 wget
```

> [!NOTE]
> Si libssl1.0.x no está disponible, instale en su lugar libssl1.1.

# <a name="rhel-78-and-centos-78"></a>[RHEL 7/8 y CentOS 7/8](#tab/rhel-centos)

```Bash
sudo yum update
sudo yum groupinstall "Development tools"
sudo yum install alsa-lib openssl wget
```

> [!IMPORTANT]
> - En RHEL/CentOS 7, siga las instrucciones sobre [cómo configurar RHEL/CentOS 7 para el SDK de voz](~/articles/cognitive-services/speech-service/how-to-configure-rhel-centos-7.md).
> - En RHEL/CentOS 8, siga las instrucciones sobre [cómo configurar OpenSSL para Linux](~/articles/cognitive-services/speech-service/how-to-configure-openssl-linux.md).

---

### <a name="c"></a>C#

[!INCLUDE [Get .NET Speech SDK](get-speech-sdk-dotnet.md)]

### <a name="c"></a>C++

[!INCLUDE [Get C++ Speech SDK](get-speech-sdk-cpp.md)]

### <a name="python"></a>Python

[!INCLUDE [Get Python Speech SDK](get-speech-sdk-python.md)]

### <a name="java"></a>Java

[!INCLUDE [Get Java Speech SDK](get-speech-sdk-java.md)]
