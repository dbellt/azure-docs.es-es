---
title: Registro de errores y excepciones en MSAL para iOS/macOS
titleSuffix: Microsoft identity platform
description: Obtenga información sobre cómo registrar errores y excepciones en MSAL para iOS/macOS.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 01/25/2021
ms.author: marsma
ms.reviewer: saeeda, jmprieur
ms.custom: aaddev
ms.openlocfilehash: 6e3af65c3eefe7f72b9cf3ff79c280012e39289e
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108164334"
---
# <a name="logging-in-msal-for-iosmacos"></a>Inicio de sesión en MSAL para iOS/macOS

[!INCLUDE [MSAL logging introduction](../../../includes/active-directory-develop-error-logging-introduction.md)]

## <a name="objective-c"></a>[Objective-C](#tab/objc)

## <a name="msal-for-ios-and-macos-logging-objc"></a>Registro de MSAL para iOS y macOS: ObjC

Establezca una devolución de llamada para capturar el registro de MSAL e incorporarlo al registro de su propia aplicación. La firma de la devolución de llamada tiene el siguiente aspecto:

```objc
/*!
    The LogCallback block for the MSAL logger

    @param  level           The level of the log message
    @param  message         The message being logged
    @param  containsPII     If the message might contain Personally Identifiable Information (PII)
                            this will be true. Log messages possibly containing PII will not be
                            sent to the callback unless PIllLoggingEnabled is set to YES on the
                            logger.

 */
typedef void (^MSALLogCallback)(MSALLogLevel level, NSString *message, BOOL containsPII);
```

Por ejemplo:

```objc
[MSALGlobalConfig.loggerConfig setLogCallback:^(MSALLogLevel level, NSString *message, BOOL containsPII)
    {
        if (!containsPII)
        {
#if DEBUG
            // IMPORTANT: MSAL logs may contain sensitive information. Never output MSAL logs with NSLog, or print, directly unless you're running your application in debug mode. If you're writing MSAL logs to file, you must store the file securely.
            NSLog(@"MSAL log: %@", message);
#endif
        }
    }];
```

### <a name="personal-data"></a>Datos personales

De forma predeterminada, MSAL no captura ni registra datos personales. La biblioteca permite a los desarrolladores de aplicaciones activar esta opción una propiedad en la clase MSALLogger. Al activar `pii.Enabled`, la aplicación tiene la responsabilidad de controlar la información confidencial y cumplir con los requisitos normativos de forma segura.

```objc
// By default, the `MSALLogger` doesn't capture any PII

// PII will be logged
MSALGlobalConfig.loggerConfig.piiEnabled = YES;

// PII will NOT be logged
MSALGlobalConfig.loggerConfig.piiEnabled = NO;
```

### <a name="logging-levels"></a>Niveles de registro

Para establecer el nivel de registro al iniciar sesión con MSAL para iOS y macOS, use uno de los siguientes valores:

|Nivel  |Descripción |
|---------|---------|
| `MSALLogLevelNothing`| Deshabilitar todos los registros |
| `MSALLogLevelError` | Nivel predeterminado; imprime información solo cuando se producen errores. |
| `MSALLogLevelWarning` | Advertencias |
| `MSALLogLevelInfo` |  Puntos de entrada de la biblioteca, con parámetros y varias operaciones de llavero. |
|`MSALLogLevelVerbose`     |  Seguimiento de API |

Por ejemplo:

```objc
MSALGlobalConfig.loggerConfig.logLevel = MSALLogLevelVerbose;
```

 ### <a name="log-message-format"></a>Formato de los mensajes de registro

En los mensajes de registro de MSAL, la parte del mensaje tiene el formato `TID = <thread_id> MSAL <sdk_ver> <OS> <OS_ver> [timestamp - correlation_id] message`.

Por ejemplo:

`TID = 551563 MSAL 0.2.0 iOS Sim 12.0 [2018-09-24 00:36:38 - 36764181-EF53-4E4E-B3E5-16FE362CFC44] acquireToken returning with error: (MSALErrorDomain, -42400) User cancelled the authorization session.`

Proporcionar los identificadores de correlación y las marcas de tiempo son útiles para el seguimiento de problemas. La información del identificador de correlación y las marcas de tiempo está disponible en el mensaje de registro. El único lugar confiable para recuperarlos es en los mensajes de registro de MSAL.

## <a name="swift"></a>[Swift](#tab/swift)

## <a name="msal-for-ios-and-macos-logging-swift"></a>Registro de MSAL para iOS y macOS: Swift

Establezca una devolución de llamada para capturar el registro de MSAL e incorporarlo al registro de su propia aplicación. La firma (representada en Objective-C) para la devolución de llamada tiene el siguiente aspecto:

```objc
/*!
    The LogCallback block for the MSAL logger

    @param  level           The level of the log message
    @param  message         The message being logged
    @param  containsPII     If the message might contain Personally Identifiable Information (PII)
                            this will be true. Log messages possibly containing PII will not be
                            sent to the callback unless PIllLoggingEnabled is set to YES on the
                            logger.

 */
typedef void (^MSALLogCallback)(MSALLogLevel level, NSString *message, BOOL containsPII);
```

Por ejemplo:

```swift
MSALGlobalConfig.loggerConfig.setLogCallback { (level, message, containsPII) in
    if let message = message, !containsPII
    {
#if DEBUG
    // IMPORTANT: MSAL logs may contain sensitive information. Never output MSAL logs with NSLog, or print, directly unless you're running your application in debug mode. If you're writing MSAL logs to file, you must store the file securely.
    print("MSAL log: \(message)")
#endif
    }
}
```

### <a name="personal-data"></a>Datos personales

De forma predeterminada, MSAL no captura ni registra datos personales. La biblioteca permite a los desarrolladores de aplicaciones activar esta opción una propiedad en la clase MSALLogger. Al activar `pii.Enabled`, la aplicación tiene la responsabilidad de controlar la información confidencial y cumplir con los requisitos normativos de forma segura.

```swift
// By default, the `MSALLogger` doesn't capture any PII

// PII will be logged
MSALGlobalConfig.loggerConfig.piiEnabled = true

// PII will NOT be logged
MSALGlobalConfig.loggerConfig.piiEnabled = false
```

### <a name="logging-levels"></a>Niveles de registro

Para establecer el nivel de registro al iniciar sesión con MSAL para iOS y macOS, use uno de los siguientes valores:

|Nivel  |Descripción |
|---------|---------|
| `MSALLogLevelNothing`| Deshabilitar todos los registros |
| `MSALLogLevelError` | Nivel predeterminado; imprime información solo cuando se producen errores. |
| `MSALLogLevelWarning` | Advertencias |
| `MSALLogLevelInfo` |  Puntos de entrada de la biblioteca, con parámetros y varias operaciones de llavero. |
|`MSALLogLevelVerbose`     |  Seguimiento de API |

Por ejemplo:

```swift
MSALGlobalConfig.loggerConfig.logLevel = .verbose
```

### <a name="log-message-format"></a>Formato de los mensajes de registro

En los mensajes de registro de MSAL, la parte del mensaje tiene el formato `TID = <thread_id> MSAL <sdk_ver> <OS> <OS_ver> [timestamp - correlation_id] message`.

Por ejemplo:

`TID = 551563 MSAL 0.2.0 iOS Sim 12.0 [2018-09-24 00:36:38 - 36764181-EF53-4E4E-B3E5-16FE362CFC44] acquireToken returning with error: (MSALErrorDomain, -42400) User cancelled the authorization session.`

Proporcionar los identificadores de correlación y las marcas de tiempo son útiles para el seguimiento de problemas. La información del identificador de correlación y las marcas de tiempo está disponible en el mensaje de registro. El único lugar confiable para recuperarlos es en los mensajes de registro de MSAL.

---

## <a name="next-steps"></a>Pasos siguientes

Para ver más ejemplos de código, consulte [Ejemplos de código de la Plataforma de identidad de Microsoft](sample-v2-code.md).
