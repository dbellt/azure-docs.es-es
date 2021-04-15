---
author: aahill
ms.service: cognitive-services
ms.topic: include
ms.date: 06/24/2019
ms.author: aahi
ms.openlocfilehash: 95bd83575809f6ecda716ff751b47b7bb499cae3
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "85073366"
---
## <a name="configure-an-environment-variable-for-authentication"></a>Configuración de una variable de entorno para la autenticación

Las aplicaciones tienen que autenticar el acceso a los servicios Cognitive Services que usan. Para realizar la autenticación, se recomienda crear una variable de entorno para almacenar las claves de los recursos de Azure. 

Una vez que tenga la clave, escríbala en una variable de entorno nueva en la máquina local que ejecuta la aplicación. Para establecer la variable de entorno, abra una ventana de consola y siga las instrucciones de su sistema operativo. Reemplace `your-key` por una de las claves del recurso.

#### <a name="windows"></a>[Windows](#tab/windows)

```console
setx COGNITIVE_SERVICE_KEY "your-key"
```

Después de agregar la variable de entorno, puede que tenga que reiniciar todos los programas en ejecución que necesiten leer la variable de entorno, incluida la ventana de consola. Por ejemplo, si usa Visual Studio como editor, reinícielo antes de ejecutar el ejemplo.

#### <a name="linux"></a>[Linux](#tab/linux)

```bash
export COGNITIVE_SERVICE_KEY=your-key
```

Después de agregar la variable de entorno, ejecute `source ~/.bashrc` desde la ventana de consola para que los cambios surtan efecto.

#### <a name="macos"></a>[macOS](#tab/unix)

Edite .bash_profile y agregue la variable de entorno:

```bash
export COGNITIVE_SERVICE_KEY=your-key
```

Después de agregar la variable de entorno, ejecute `source ~/.bash_profile` desde la ventana de consola para que los cambios surtan efecto.

***
