---
title: Inicio de sesión en el Explorador de Azure Storage | Microsoft Docs
description: Documentación sobre el inicio de sesión en el Explorador de Azure Storage
services: storage
author: MRayermannMSFT
ms.service: storage
ms.topic: article
ms.date: 04/01/2021
ms.author: chuye
ms.openlocfilehash: 26ce15bfe474f96aaef17f186f56e36ce42b7239
ms.sourcegitcommit: ad921e1cde8fb973f39c31d0b3f7f3c77495600f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107950539"
---
# <a name="sign-in-to-storage-explorer"></a>Inicio de sesión en el Explorador de Storage

El inicio de sesión es la manera recomendada de acceder a los recursos de almacenamiento de Azure con el Explorador de Storage. Al iniciar sesión, aprovecha las ventajas de los permisos respaldados por Azure AD, como las listas de control de acceso POSIX de Gen2 y RBAC. 

## <a name="how-to-sign-in"></a>Inicio de sesión

En el Explorador de Storage, abra el **cuadro de diálogo de conexión**. Puede abrir el **cuadro de diálogo de conexión** desde la barra de herramientas vertical izquierda o haciendo clic en **Agregar cuenta...** en el **panel de cuentas**.

Una vez abierto el cuadro de diálogo, elija **Suscripción** como tipo de recurso al que quiere conectarse y haga clic en **Siguiente**.

Ahora debe elegir en qué entorno de Azure quiere iniciar sesión. Puede elegir entre cualquiera de los entornos conocidos, como Azure o Azure China, o puede agregar el suyo propio. Una vez seleccionado el entorno, haga clic en **Siguiente**.

En este momento, se iniciará el **explorador web predeterminado** del sistema operativo y se abrirá una página de inicio de sesión. Para obtener los mejores resultados, deje abierta esta ventana del explorador siempre que use el Explorador de Storage o, al menos, hasta que haya realizado todas las operaciones de MFA esperadas. Cuando haya terminado de iniciar sesión, puede volver al Explorador de Storage.

## <a name="managing-accounts"></a>Administración de cuentas

Puede administrar y quitar las cuentas de Azure en las que ha iniciado sesión desde el **panel de cuentas**. Para abrir el **panel de cuentas**, haga clic en el botón **Administrar cuentas** de la barra de herramientas vertical izquierda.

En el **panel de cuentas**, verá las cuentas en las que ha iniciado sesión. En cada cuenta estarán:
- Los inquilinos a los que pertenece la cuenta
- Las suscripciones a las que tiene acceso en cada inquilino

De forma predeterminada, el Explorador de Storage solo inicia sesión en el inquilino principal. Si quiere ver las suscripciones y los recursos de otro inquilino, deberá activar ese inquilino. Para activar un inquilino, active la casilla situada junto a él. Cuando haya terminado de trabajar con un inquilino, puede desmarcar su casilla para desactivarlo. No puede desactivar el inquilino principal.

Después de activar un inquilino, es posible que tenga que volver a escribir sus credenciales para que el Explorador de Storage pueda cargar las suscripciones o acceder a los recursos desde él. La necesidad de volver a escribir las credenciales suele deberse a una directiva de acceso condicional (CA), como la autenticación multifactor (MFA). Y, aunque puede que ya haya realizado la autenticación multifactor en otro inquilino, es posible que tenga que volver a hacerla. Para volver a escribir las credenciales, simplemente haga clic en **Volver a escribir credenciales...** . También puede hacer clic en **Detalles del error...** para ver exactamente por qué no se pudieron cargar las suscripciones.

Cuando se hayan cargado las suscripciones, puede elegir cuáles quiere filtrar activando o desactivando sus casillas.

Si quiere quitar toda la cuenta de Azure, haga clic en **Quitar** junto a la cuenta.

## <a name="changing-where-sign-in-happens"></a>Cambio del lugar donde se produce el inicio de sesión

De forma predeterminada, el inicio de sesión se realizará en el **explorador web predeterminado** del sistema operativo. El inicio de sesión con el explorador web predeterminado simplifica el modo en que se accede a los recursos protegidos mediante directivas de CA, como MFA. Si, por algún motivo, el inicio de sesión con el **explorador web predeterminado** del sistema operativo no funciona, puede cambiar dónde o cómo el Explorador de Storage realiza el inicio de sesión.

En **Configuración (icono de engranaje de la izquierda)**  > **Aplicación** > **Inicio de sesión**, busque la configuración **Iniciar sesión con**. Hay tres opciones:
- **Explorador web predeterminado**: el inicio de sesión se realizará en el **explorador web predeterminado** del sistema operativo. Esta opción no se recomienda.
- **Inicio de sesión integrado**: el inicio de sesión se realizará en una ventana del Explorador de Storage. Esta opción puede ser útil si está intentando iniciar sesión con varias cuentas Microsoft (MSA) a la vez. Si elige esta opción, es posible que tenga problemas con algunas directivas de CA.
- **Flujo de código de dispositivo**: el Explorador de Storage le dará un código para entrar en una ventana del explorador. No se recomienda esta opción. El flujo de código de dispositivo no es compatible con muchas directivas de CA.

## <a name="troubleshooting-sign-in-issues"></a>Solución de problemas de inicio de sesión de usuario

Si tiene problemas para iniciar sesión o experimenta problemas con una cuenta de Azure después de iniciar sesión, consulte la [sección de inicio de sesión de la guía de solución de problemas del Explorador de Storage](./storage-explorer-troubleshooting.md#sign-in-issues).

## <a name="next-steps"></a>Pasos siguientes

* [Administración de recursos de Azure Blob Storage con el Explorador de Storage](../../vs-azure-tools-storage-explorer-blobs.md)
* [Solución de problemas con el inicio de sesión](./storage-explorer-troubleshooting.md#sign-in-issues)
