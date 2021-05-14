---
title: Configuración de OpenSSL para Linux
titleSuffix: Azure Cognitive Services
description: Aprenda a configurar OpenSSL para Linux.
services: cognitive-services
author: jhakulin
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 01/16/2020
ms.author: jhakulin
zone_pivot_groups: programming-languages-set-two
ROBOTS: NOINDEX
ms.openlocfilehash: 0aee0030937a6e11860907e27b775c2671137f71
ms.sourcegitcommit: dd425ae91675b7db264288f899cff6add31e9f69
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2021
ms.locfileid: "108331113"
---
# <a name="configure-openssl-for-linux"></a>Configuración de OpenSSL para Linux

Cuando usa cualquier versión del SDK de Voz anterior a la 1.9.0, [OpenSSL](https://www.openssl.org) se configura dinámicamente en la versión del sistema host. En versiones posteriores del SDK de Voz, OpenSSL (versión [1.1.1 b](https://mta.openssl.org/pipermail/openssl-announce/2019-February/000147.html)) se vincula estáticamente a la biblioteca principal del SDK de Voz.

Para garantizar la conectividad, compruebe que se han instalado los certificados de OpenSSL en el sistema. Ejecute un comando:
```bash
openssl version -d
```

La salida en los sistemas basados en Ubuntu/Debian debe ser:
```
OPENSSLDIR: "/usr/lib/ssl"
```

Compruebe si existe el subdirectorio `certs` en OPENSSLDIR. En el ejemplo anterior, sería `/usr/lib/ssl/certs`.

* Si `/usr/lib/ssl/certs` existe y contiene muchos archivos de certificado individuales (con la extensión `.crt` o `.pem`), no es necesario realizar más acciones.

* Si OPENSSLDIR es algo distinto de `/usr/lib/ssl` y/o hay un único archivo de paquete de certificados en lugar de varios archivos individuales, debe establecer una variable de entorno SSL adecuada para indicar dónde se pueden encontrar los certificados.

## <a name="examples"></a>Ejemplos

- OPENSSLDIR is `/opt/ssl`. Existe un subdirectorio `certs` con muchos archivos `.crt` o `.pem`.
Establezca la variable de entorno `SSL_CERT_DIR` para que apunte a `/opt/ssl/certs` antes de ejecutar un programa que utilice el SDK de Voz. Por ejemplo:
```bash
export SSL_CERT_DIR=/opt/ssl/certs
```

- OPENSSLDIR es `/etc/pki/tls` (como en los sistemas basados en RHEL o CentOS). Existe un subdirectorio `certs` archivo de paquete de certificados, por ejemplo `ca-bundle.crt`.
Establezca la variable de entorno `SSL_CERT_FILE` para que apunte a ese archivo antes de ejecutar un programa que utilice el SDK de Voz. Por ejemplo:
```bash
export SSL_CERT_FILE=/etc/pki/tls/certs/ca-bundle.crt
```

## <a name="certificate-revocation-checks"></a>Comprobaciones de revocación de certificados
Al conectarse al servicio Voz, el SDK de Voz comprobará que el certificado TLS usado por dicho servicio no se ha revocado. Para realizar esta comprobación, el SDK de Voz necesitará acceso a los puntos de distribución de CRL para las entidades de certificación que utiliza Azure. En [este documento](../../security/fundamentals/tls-certificate-changes.md)se puede encontrar una lista de las posibles ubicaciones de descarga de CRL. Si un certificado se ha revocado o no se puede descargar la lista CRL, el SDK de Voz anulará la conexión y generará el evento Cancelado.

Si la red donde se usa el SDK de Voz está configurada para impedir el acceso a las ubicaciones de descarga de CRL, la comprobación de CRL puede deshabilitarse o establecerse para que no genere errores si no se puede recuperar la lista CRL. Esta configuración se realiza a través del objeto de configuración utilizado para crear un objeto de reconocedor.

Para continuar con la conexión cuando no se pueda recuperar una CRL, establezca la propiedad OPENSSL_CONTINUE_ON_CRL_DOWNLOAD_FAILURE.

::: zone pivot="programming-language-csharp"

```csharp
config.SetProperty("OPENSSL_CONTINUE_ON_CRL_DOWNLOAD_FAILURE", "true");
```

::: zone-end

::: zone pivot="programming-language-cpp"

```C++
config->SetProperty("OPENSSL_CONTINUE_ON_CRL_DOWNLOAD_FAILURE", "true");
```

::: zone-end

::: zone pivot="programming-language-java"

```java
config.setProperty("OPENSSL_CONTINUE_ON_CRL_DOWNLOAD_FAILURE", "true");
```

::: zone-end

::: zone pivot="programming-language-python"

```Python
speech_config.set_property_by_name("OPENSSL_CONTINUE_ON_CRL_DOWNLOAD_FAILURE", "true")?
```

::: zone-end

::: zone pivot="programming-language-more"

```ObjectiveC
[config setPropertyTo:@"true" byName:"OPENSSL_CONTINUE_ON_CRL_DOWNLOAD_FAILURE"];
```

::: zone-end
Si se establece en "true", se intenta recuperar la CRL y, si la recuperación se realiza correctamente, se comprueba la revocación del certificado. Si se produce un error en la recuperación, la conexión podrá continuar.

Para deshabilitar completamente las comprobaciones de revocación de certificados, establezca la propiedad OPENSSL_DISABLE_CRL_CHECK en "true".
::: zone pivot="programming-language-csharp"

```csharp
config.SetProperty("OPENSSL_DISABLE_CRL_CHECK", "true");
```

::: zone-end

::: zone pivot="programming-language-cpp"

```C++
config->SetProperty("OPENSSL_DISABLE_CRL_CHECK", "true");
```

::: zone-end

::: zone pivot="programming-language-java"

```java
config.setProperty("OPENSSL_DISABLE_CRL_CHECK", "true");
```

::: zone-end

::: zone pivot="programming-language-python"

```Python
speech_config.set_property_by_name("OPENSSL_DISABLE_CRL_CHECK", "true")?
```

::: zone-end

::: zone pivot="programming-language-more"

```ObjectiveC
[config setPropertyTo:@"true" byName:"OPENSSL_DISABLE_CRL_CHECK"];
```

::: zone-end


> [!NOTE]
> También merece la pena mencionar que algunas distribuciones de Linux no tienen definida una variable de entorno TMP o TMPDIR. Como consecuencia, el SDK de Voz descargará la lista de revocación de certificados (CRL) cada vez, en lugar de almacenarla en caché en el disco para reutilizarla hasta que expire. Para mejorar el rendimiento inicial de la conexión, puede [crear una variable de entorno denominada TMPDIR y establecerla en la ruta de acceso del directorio temporal elegido](https://help.ubuntu.com/community/EnvironmentVariables).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Acerca del SDK de Voz](speech-sdk.md)