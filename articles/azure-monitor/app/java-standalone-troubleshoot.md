---
title: Solución de problemas de Application Insights de Azure Monitor para Java
description: Obtenga información sobre cómo solucionar problemas del agente de Java para Application Insights de Azure Monitor
ms.topic: conceptual
ms.date: 11/30/2020
ms.custom: devx-track-java
ms.openlocfilehash: cea6e93999477f7f33daaf5440e161a0da6fb2a2
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112027846"
---
# <a name="troubleshooting-guide-azure-monitor-application-insights-for-java"></a>Guía de solución de problemas: Application Insights de Azure Monitor para Java

En este artículo se tratan algunos de los problemas comunes que podría encontrar al instrumentar una aplicación Java mediante el agente de Java para Application Insights. También se explican los pasos para resolver estos problemas. Application Insights es una característica del servicio de plataforma Azure Monitor.

## <a name="check-the-self-diagnostic-log-file"></a>Comprobación del archivo de registro de autodiagnóstico

De forma predeterminada, Application Insights Java 3.x genera un archivo de registro denominado `applicationinsights.log` en el mismo directorio que contiene el archivo `applicationinsights-agent-3.1.1.jar`.

Este archivo de registro es la primera ubicación en la que buscar sugerencias sobre cualquier problema que se experimente.

## <a name="jvm-fails-to-start"></a>JVM no se inicia

Si JVM no se inicia con el mensaje "Error al abrir el archivo ZIP o falta el manifiesto JAR", intente volver a descargar el archivo JAR del agente, porque puede que se haya dañado durante la transferencia de archivos.

## <a name="upgrade-from-the-application-insights-java-2x-sdk"></a>Actualización del SDK de Application Insights para Java 2.x

Si ya está usando el SDK de Application Insights para Java 2.x en la aplicación, puede seguir usándolo.
El agente de Application Insights Java 3.x lo detectará, y capturará y correlacionará cualquier telemetría personalizada que envíe a través del SDK 2.x, al mismo tiempo que suprimirá cualquier recopilación automática que realice el SDK 2.x para evitar la telemetría duplicada.
Para obtener más información, consulte [Actualización desde el SDK para Java 2.x](./java-standalone-upgrade-from-2x.md).

## <a name="upgrade-from-application-insights-java-30-preview"></a>Actualización de Application Insights para Java 3.0 (versión preliminar)

Si va a actualizar desde el agente para Java 3.0 (versión preliminar), revise detenidamente todas las [opciones de configuración](./java-standalone-config.md). La estructura JSON ha cambiado completamente en la versión 3.0 de disponibilidad general (GA).

Entre estos cambios se incluyen los siguientes:

-  El propio nombre del archivo de configuración ha cambiado de `ApplicationInsights.json` a `applicationinsights.json`.
-  El nodo `instrumentationSettings` ya no está presente. Todo el contenido de `instrumentationSettings` se ha movido al nivel raíz. 
-  Los nodos de configuración, como `sampling`, `jmxMetrics`, `instrumentation` y `heartbeat`, se han movido de `preview` al nivel raíz.

## <a name="some-logging-is-not-auto-collected"></a>Algunos registros no se recopilan automáticamente

El registro solo se captura si primero cumple el nivel que está configurado para la plataforma de registro, y segundo, también cumple el nivel configurado para Application Insights.

Por ejemplo, si la plataforma de registro está configurada para registrar `WARN` (y versiones posteriores) del paquete `com.example`, y Application Insights está configurado para capturar `INFO` (y superiores), Application Insights solo capturará `WARN` (y superiores) del paquete `com.example`.

La mejor manera de saber si una instrucción de registro determinada cumple el umbral configurado de las plataformas de registro es confirmar que se muestra en el registro de aplicaciones normal (por ejemplo, archivo o consola).

Tenga en cuenta también que, si se pasa un objeto de excepción al registrador, el mensaje de registro (y los detalles del objeto de excepción) se mostrarán en Azure Portal en la tabla `exceptions`, en lugar de la tabla `traces`.

Para obtener más información, consulte la [configuración de registro de recopilación automática](./java-standalone-config.md#auto-collected-logging).

## <a name="import-ssl-certificates"></a>Importación de certificados SSL

Esta sección le ayuda a solucionar problemas y, posiblemente, a corregir las excepciones relacionadas con los certificados de SSL al usar el agente de Java.

Hay dos formas distintas de resolver este problema:
* Si usa un almacén de claves de Java predeterminado
* Si usa un almacén de claves de Java personalizado

Si no está seguro de la ruta de acceso que debe seguir, compruebe si tiene un argumento de JVM `-Djavax.net.ssl.trustStore=...`.
Si _no_ tiene ningún argumento de JVM, probablemente está usando el almacén de claves de Java predeterminado.
Si _tiene_ este argumento de JVM, es probable que esté usando un almacén de claves personalizado y el argumento de JVM señalará al almacén de claves personalizado.

### <a name="if-using-the-default-java-keystore"></a>Si usa un almacén de claves de Java predeterminado:

Normalmente, el almacén de claves de Java predeterminado ya tendrá todos los certificados raíz de CA. Sin embargo, puede haber algunas excepciones, como que el certificado del punto de conexión de ingesta podría estar firmado por un certificado raíz diferente. Por ello, se recomiendan los tres pasos siguientes para resolver este problema.

1.  Compruebe si el certificado raíz que se usó para firmar el punto de conexión de Application Insights ya está presente en el almacén de claves predeterminado. De forma predeterminada, los certificados de CA de confianza se almacenan en `$JAVA_HOME/jre/lib/security/cacerts`. Para enumerar los certificados de un almacén de claves de Java, use el siguiente comando:
    > `keytool -list -v -keystore $PATH_TO_KEYSTORE_FILE`
 
    Puede redirigir la salida a un archivo temporal como este (será fácil de buscar más adelante).
    > `keytool -list -v -keystore $JAVA_HOME/jre/lib/security/cacerts > temp.txt`

2. Una vez que tenga la lista de certificados, siga estos [pasos](#steps-to-download-ssl-certificate) para descargar el certificado raíz que se usó para firmar el punto de conexión de Application Insights.

    Una vez descargado el certificado, genere un hash SHA-1 en el certificado con el siguiente comando:
    > `keytool -printcert -v -file "your_downloaded_root_certificate.cer"`
 
    Copie el valor SHA-1 y compruebe si este valor está presente en el archivo "temp.txt" que guardó anteriormente.  Si no puede encontrar el valor SHA-1 en el archivo temporal, esto significa que falta el certificado raíz descargado en el almacén de claves de Java predeterminado.


3. Importe el certificado raíz al almacén de claves de Java predeterminado con el siguiente comando:
    >   `keytool -import -file "the cert file" -alias "some meaningful name" -keystore "path to cacerts file"`
 
    En este caso, será
 
    > `keytool -import -file "your downloaded root cert file" -alias "some meaningful name" $JAVA_HOME/jre/lib/security/cacerts`


### <a name="if-using-a-custom-java-keystore"></a>Si usa un almacén de claves de Java personalizado:

Si usa un almacén de claves de Java personalizado, es posible que tenga que importar en él los certificados SSL raíz de los puntos de conexión de Application Insights.
Se recomiendan los pasos siguientes para resolver este problema.
1. Siga estos [pasos](#steps-to-download-ssl-certificate) para descargar el certificado raíz desde el punto de conexión de Application Insights.
2. Use el siguiente comando para importar el certificado SSL raíz en el almacén de claves de Java personalizado:
    > `keytool -importcert -alias your_ssl_certificate -file "your downloaded SSL certificate name.cer" -keystore "Your KeyStore name" -storepass "Your keystore password" -noprompt`

### <a name="steps-to-download-ssl-certificate"></a>Pasos para descargar el certificado SSL

1.  Abra el explorador que prefiera y vaya a la dirección URL de `IngestionEndpoint` presente en la cadena de conexión que se usa para instrumentar la aplicación.

    :::image type="content" source="media/java-ipa/troubleshooting/ingestion-endpoint-snippet.png" alt-text="Captura de pantalla que muestra una cadena de conexión de Application Insights." lightbox="media/java-ipa/troubleshooting/ingestion-endpoint-snippet.png":::

2.  Seleccione el icono de **Ver información del sitio** (candado) en el explorador y, a continuación, seleccione la opción **Certificado**.

    :::image type="content" source="media/java-ipa/troubleshooting/certificate-icon-capture.png" alt-text="Captura de pantalla de la opción Certificado en Ver información del sitio." lightbox="media/java-ipa/troubleshooting/certificate-icon-capture.png":::

3.  En lugar de descargar el certificado de "hoja", debe descargar el certificado "raíz" como se muestra a continuación. Más adelante, tendrá que hacer clic en "Ruta de acceso del certificado"- > seleccionar el certificado raíz > hacer clic en "Ver certificado". Se abrirá un menú de nuevo certificado y desde ahí podrá descargar el certificado.

    :::image type="content" source="media/java-ipa/troubleshooting/root-certificate-selection.png" alt-text="Captura de pantalla de cómo seleccionar el certificado raíz." lightbox="media/java-ipa/troubleshooting/root-certificate-selection.png":::

4.  Vaya a la pestaña **Detalles** y seleccione **Copiar en archivo**.
5.  Seleccione el botón **Siguiente**, luego el formato **X.509 codificado base 64 (.CER)** formato y, a continuación, vuelva a seleccionar **Siguiente**.

    :::image type="content" source="media/java-ipa/troubleshooting/certificate-export-wizard.png" alt-text="Captura de pantalla del Asistente para exportación de certificados, con un formato seleccionado." lightbox="media/java-ipa/troubleshooting/certificate-export-wizard.png":::

6.  Especifique el archivo en el que quiere guardar el certificado SSL. Después, seleccione **Siguiente** > **Finalizar**. Debería ver el mensaje "La exportación se realizó correctamente".

> [!WARNING]
> Tiene que repetir estos pasos para obtener el nuevo certificado antes de que expire el actual. Puede encontrar la información de expiración en la pestaña **Detalles** del cuadro de diálogo **Certificado**.
>
> :::image type="content" source="media/java-ipa/troubleshooting/certificate-details.png" alt-text="Captura de pantalla que muestra los detalles del certificado SSL." lightbox="media/java-ipa/troubleshooting/certificate-details.png":::
