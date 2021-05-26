---
title: 'Seguridad y autenticación: Azure Event Grid IoT Edge | Microsoft Docs'
description: Seguridad y autenticación de Event Grid en IoT Edge.
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.subservice: iot-edge
ms.date: 05/10/2021
ms.topic: article
ms.openlocfilehash: cfdefe642e7e0f428680340747f825641d19b391
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110378290"
---
# <a name="security-and-authentication"></a>Seguridad y autenticación

La seguridad y la autenticación son un concepto avanzado, por lo que es necesario familiarizarse primero con los conceptos básicos de Event Grid. Comience por [aquí](concepts.md) si no está familiarizado con Event Grid en IoT Edge. El módulo de Event Grid se basa en la infraestructura de seguridad existente en IoT Edge. Consulte [esta documentación ](../../iot-edge/security.md) para obtener más detalles y ajustes.

En las secciones siguientes se describe a detalle cómo se protegen y autentican estas opciones:

* Configuración de TLS
* Autenticación de cliente de entrada
* Autenticación de servidor de salida
* Autenticación de cliente de salida

>[!IMPORTANT]
>La seguridad y autenticación del módulo de Event Grid aprovechan la infraestructura existente disponible en IoT Edge. Se supone que el subsistema de IoT Edge es seguro.

>[!IMPORTANT]
>De forma predeterminada, la configuración Event Grid **es segura**. En las subsecciones siguientes se explican todas las opciones y los valores posibles que puede usar para reemplazar algunos aspectos de la autenticación. Analice el impacto antes de realizar cualquier cambio. Para que los cambios surtan efecto, es necesario volver a implementar el módulo de Event Grid.

## <a name="tls-configuration-aka-server-authentication"></a>Configuración de TLS (o autenticación de servidor)

El módulo de Event Grid hospeda los puntos de conexión HTTP y HTTPS. El demonio de seguridad de IoT Edge le asigna a cada módulo de IoT Edge un certificado de servidor. Usamos el certificado de servidor para proteger el punto de conexión. Al expirar, el módulo se actualiza automáticamente con un nuevo certificado del demonio de seguridad de IoT Edge.

De forma predeterminada, solo se permite la comunicación por HTTPS. Puede sobrescribir este comportamiento mediante la configuración **inbound__serverAuth__tlsPolicy**. En la tabla siguiente se capturan los valores posibles de esta propiedad.

| Valores posibles | Descripción |
| ---------------- | ------------ |
| Strict | Predeterminada. Habilita solo HTTPS.
| habilitado | Habilita HTTP y HTTPS.
| Disabled | Habilita solo HTTP.

## <a name="inbound-client-authentication"></a>Autenticación de cliente de entrada

Los clientes son entidades que realizan operaciones de administración o tiempo de ejecución. Los clientes pueden ser otros módulos de IoT Edge o aplicaciones que no son de IoT.

El módulo de Event Grid admite dos tipos de autenticación de cliente:

* Basada en claves de firma de acceso compartido (SAS).
* Basada en certificados.

De forma predeterminada, el módulo de Event Grid está configurado para aceptar solo la autenticación basada en certificados. En el inicio, el módulo de Event Grid recupera "TrustBundle" del demonio de seguridad de IoT Edge y utiliza ese valor para validar todos los certificados de cliente. Los certificados de cliente que no se resuelven en esta cadena se rechazarán con el error `UnAuthorized`.

### <a name="certificate-based-client-authentication"></a>Autenticación de cliente basada en certificados

La autenticación basada en certificados está activada de forma predeterminada. Puede optar por deshabilitar la autenticación basada en certificados a través de la propiedad **inbound__clientAuth__clientCert__enabled**. En la tabla siguiente, se muestran los valores posibles.

| Valores posibles | Descripción |
| ----------------  | ------------ |
| true | Predeterminada. Requiere que todas las solicitudes del módulo de Event Grid presenten un certificado de cliente. Además, tendrá que configurar **inbound__clientAuth__clientCert__source**.
| false | No obliga al cliente a presentar un certificado.

En la tabla siguiente se capturan los posibles valores de **inbound__clientAuth__clientCert__source**.

| Valores posibles | Descripción |
| ---------------- | ------------ |
| IoT Edge | Predeterminada. Usa el valor de Trustbundle de IoT Edge para validar todos los certificados de cliente.

Si un cliente presenta un certificado autofirmado, de forma predeterminada, el módulo Event Grid rechazará dichas solicitudes. Puede optar por permitir los certificados de cliente autofirmados a través de la propiedad **inbound__clientAuth__clientCert__allowUnknownCA**. En la tabla siguiente, se muestran los valores posibles.

| Valores posibles | Descripción |
| ----------------  | ------------|
| true | Predeterminada. Permite que los certificados autofirmados se presenten correctamente.
| false | Producirá un error en las solicitudes si se presentan certificados autofirmados.

>[!IMPORTANT]
>En escenarios de producción, debería establecer **inbound__clientAuth__clientCert__allowUnknownCA** en **false**.

### <a name="sas-key-based-client-authentication"></a>Autenticación de cliente basada en claves de SAS

Además de la autenticación basada en certificados, el módulo de Event Grid también puede realizar la autenticación basada en claves de SAS. La clave de SAS es como un secreto configurado en el módulo de Event Grid que se debe usar para validar todas las llamadas entrantes. Los clientes deben especificar el secreto en el encabezado HTTP "aeg-sas-key". La solicitud se rechazará con el error `UnAuthorized` si no coincide.

La configuración para controlar la autenticación basada en claves de SAS es **inbound__clientAuth__sasKeys__enabled**.

| Valores posibles | Descripción  |
| ----------------  | ------------ |
| true | Permite la autenticación basada en claves de SAS. Requiere **inbound__clientAuth__sasKeys__key1** o **inbound__clientAuth__sasKeys__key2**
| false | Predeterminada. La autenticación basada en claves de SAS está deshabilitada.

 **inbound__clientAuth__sasKeys__key1** y **inbound__clientAuth__sasKeys__key2** son claves que se configuran en el módulo de Event Grid para comprobar las solicitudes entrantes. Debe configurarse al menos una de las claves. El cliente que realiza la solicitud deberá presentar la clave como parte del encabezado de solicitud "**aeg-sas-key**". Si ambas claves están configuradas, el cliente puede presentar una de las claves.

> [!NOTE]
>Puede configurar ambos métodos de autenticación. En ese caso, la clave de SAS se comprueba primero y solo se realiza la autenticación basada en certificados si se produce un error. Para que una solicitud se realice correctamente, solo es necesario que uno de los métodos de autenticación sea correcto.

## <a name="outbound-client-authentication"></a>Autenticación de cliente de salida

El cliente en el contexto saliente hace referencia al módulo de Event Grid. La operación que se realiza es la entrega de eventos a los suscriptores. Los módulos de suscripción se consideran como el servidor.

El demonio de seguridad de IoT Edge le asigna a cada módulo de IoT Edge un certificado de identidad. Usamos el certificado de identidad para las llamadas salientes. Al expirar, el módulo se actualiza automáticamente con un nuevo certificado del demonio de seguridad de IoT Edge.

La configuración para controlar la autenticación del cliente de salida es **outbound__clientAuth__clientCert__enabled**.

| Valores posibles | Descripción |
| ----------------  | ------------ |
| true | Predeterminada. Requiere que todas las solicitudes de salida del módulo de Event Grid presenten un certificado. Necesita configurar **outbound__clientAuth__clientCert__source**.
| false | No requiere que el módulo de Event Grid presente su certificado.

La configuración que controla el origen del certificado es **outbound__clientAuth__clientCert__source**.

| Valores posibles | Descripción |
| ---------------- | ------------ |
| IoT Edge | Predeterminada. Usa el certificado de identidad del módulo que ha configurado el demonio de seguridad de IoT Edge.

### <a name="outbound-server-authentication"></a>Autenticación de servidor de salida

Uno de los tipos de destino de un suscriptor de Event Grid es "webhook". De forma predeterminada, solo se aceptan puntos de conexión HTTPS para estos suscriptores.

La configuración para controlar la directiva de destino de webhook es **outbound__webhook__httpsOnly**.

| Valores posibles | Descripción |
| ----------------  | ------------ |
| true | Predeterminada. Solo admite a los suscriptores con el punto de conexión HTTPS.
| false | Admite a los suscriptores con un punto de conexión HTTP o HTTPS.

De forma predeterminada, el módulo de Event Grid validará el certificado de servidor del suscriptor. Puede omitir la validación si sobrescribe **outbound__webhook__skipServerCertValidation**. Los valores posibles son:

| Valores posibles | Descripción |
| ----------------  | ------------ |
| true | No valida el certificado de servidor del suscriptor.
| false | Predeterminada. Valida el certificado de servidor del suscriptor.

Si el certificado del suscriptor está autofirmado, de forma predeterminada el módulo de Event Grid rechazará a los suscriptores. Para permitir los certificados autofirmados, puede sobrescribir **outbound__webhook__allowUnknownCA**. En la tabla siguiente, se muestran los posibles valores.

| Valores posibles | Descripción |
| ----------------  | ------------ |
| true | Predeterminada. Permite que los certificados autofirmados se presenten correctamente.
| false | Producirá un error en las solicitudes si se presentan certificados autofirmados.

>[!IMPORTANT]
>En escenarios de producción, debería establecer **outbound__webhook__allowUnknownCA** en **false**.

> [!NOTE]
>El entorno de IoT Edge genera certificados autofirmados. Se recomienda generar certificados emitidos por entidades de certificación autorizadas para las cargas de trabajo de producción y establecer la propiedad **allowUnknownCA** en **false** para las de entrada y salida.

## <a name="summary"></a>Resumen

De forma predeterminada, el módulo de Event Grid **es seguro**. Se recomienda mantener estos valores predeterminados para las implementaciones de producción.

A continuación se indican los principios orientativos que se usan para la configuración:

* Permitir solo las solicitudes HTTPS al módulo.
* Permitir solo la autenticación de cliente basada en certificados. Permitir solo aquellos certificados que sean emitidos por entidades de certificación conocidas. No permitir los certificados autofirmados.
* No permitir la autenticación de cliente basada en claves de SAS.
* Presentar siempre el certificado de identidad del módulo de Event Grid en las llamadas salientes.
* Permitir solo a los suscriptores HTTPS para los tipos de destino de webhook.
* Validar siempre el certificado de servidor del suscriptor para los tipos de destino de webhook. Permitir solo los certificados emitidos por entidades de certificación conocidas. No permitir los certificados autofirmados.

De forma predeterminada, el módulo de Event Grid se implementa con la configuración siguiente:

 ```json
 {
  "Env": [
    "inbound__serverAuth__tlsPolicy=strict",
    "inbound__serverAuth__serverCert__source=IoTEdge",
    "inbound__clientAuth__sasKeys__enabled=false",
    "inbound__clientAuth__clientCert__enabled=true",
    "inbound__clientAuth__clientCert__source=IoTEdge",
    "inbound__clientAuth__clientCert__allowUnknownCA=true",
    "outbound__clientAuth__clientCert__enabled=true",
    "outbound__clientAuth__clientCert__source=IoTEdge",
    "outbound__webhook__httpsOnly=true",
    "outbound__webhook__skipServerCertValidation=false",
    "outbound__webhook__allowUnknownCA=true"
  ],
  "HostConfig": {
    "PortBindings": {
      "4438/tcp": [
        {
          "HostPort": "4438"
        }
      ]
    }
  }
}
```
