---
title: Comunicación segura con proxy inverso de Azure Service Fabric
description: Configure el proxy inverso para habilitar la comunicación segura de un extremo a otro en una aplicación de Azure Service Fabric.
ms.topic: conceptual
ms.date: 08/10/2017
ms.openlocfilehash: b87ec5d1c43d15c36a3188678efbdfdb72509cd4
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111956739"
---
# <a name="connect-to-a-secure-service-with-the-reverse-proxy"></a>Conexión a un servicio seguro con el proxy inverso

En este artículo se explica cómo establecer una conexión segura entre el proxy inverso y los servicios, lo que crea un canal seguro de un extremo a otro. Para más información sobre el proxy inverso, consulte [Proxy inverso en Azure Service Fabric](service-fabric-reverseproxy.md)

> [!IMPORTANT]
> La conexión a servicios seguros solo se admite cuando se configura el proxy inverso para escuchar en HTTPS. En este artículo se da por supuesto que este es el caso. Consulte [Configuración del proxy inverso en Azure Service Fabric](service-fabric-reverseproxy-setup.md) para configurar el proxy inverso en Service Fabric.

## <a name="secure-connection-establishment-between-the-reverse-proxy-and-services"></a>Establecimiento de una conexión segura entre el proxy inverso y los servicios 

### <a name="reverse-proxy-authenticating-to-services"></a>Autenticación del proxy inverso con los servicios:
El proxy inverso se identifica él mismo con los servicios mediante su certificado. Para los clústeres de Azure, el certificado se especifica con la propiedad **reverseProxyCertificate** de la sección [Tipo de recurso](../azure-resource-manager/templates/syntax.md) de [_ *Microsoft.ServiceFabric/clusters**](/azure/templates/microsoft.servicefabric/clusters) de la plantilla de Resource Manager. Para los clústeres independientes, el certificado se especifica con las propiedades **_ReverseProxyCertificate_*o*_ReverseProxyCertificateCommonNames_*en la sección* Seguridad** de ClusterConfig.json. Para más información, consulte [Habilitación del proxy inverso en clústeres independientes](service-fabric-reverseproxy-setup.md#enable-reverse-proxy-on-standalone-clusters). 

Los servicios pueden implementar la lógica para comprobar el certificado presentado por el proxy inverso. Los servicios pueden especificar los detalles del certificado de cliente aceptado como valores de configuración en el paquete de configuración. Esto se puede leer en tiempo de ejecución y puede utilizarse para validar el certificado presentado por el proxy inverso. Consulte [Administración de los parámetros de la aplicación en varios entornos](service-fabric-manage-multiple-environment-app-configuration.md) para agregar los valores de configuración. 

### <a name="reverse-proxy-verifying-the-services-identity-via-the-certificate-presented-by-the-service"></a>Comprobación por parte del proxy inverso de la identidad del servicio mediante el certificado presentado por el servicio:
El proxy inverso admite las siguientes directivas para realizar la validación de los certificados de servidor presentados por los servicios: Ninguna, ServiceCommonNameAndIssuer y ServiceCertificateThumbprints.
Para seleccionar la directiva del proxy inverso, especifique **ApplicationCertificateValidationPolicy** en la sección **ApplicationGateway/Http** en [fabricSettings](service-fabric-cluster-fabric-settings.md).

En la siguiente sección se muestran los detalles de configuración para cada una de estas opciones.

### <a name="service-certificate-validation-options"></a>Opciones de validación de certificados de servicio 

- **Ninguna**: el proxy inverso omite la comprobación del certificado de los servicios con proxy y establece la conexión segura. Este es el comportamiento predeterminado.
Especifique **ApplicationCertificateValidationPolicy** con el valor **None** en la sección [**ApplicationGateway/Http**](./service-fabric-cluster-fabric-settings.md#applicationgatewayhttp).

   ```json
   {
   "fabricSettings": [
             ...
             {
               "name": "ApplicationGateway/Http",
               "parameters": [
                 {
                   "name": "ApplicationCertificateValidationPolicy",
                   "value": "None"
                 }
               ]
             }
           ],
           ...
   }
   ```

- **ServiceCommonNameAndIssuer**: el proxy inverso comprueba el certificado presentado por el servicio según el nombre común del certificado y la huella digital del emisor inmediato: Especifique **ApplicationCertificateValidationPolicy** con el valor **ServiceCommonNameAndIssuer** en la sección [**ApplicationGateway/Http**](./service-fabric-cluster-fabric-settings.md#applicationgatewayhttp).

   ```json
   {
   "fabricSettings": [
             ...
             {
               "name": "ApplicationGateway/Http",
               "parameters": [
                 {
                   "name": "ApplicationCertificateValidationPolicy",
                   "value": "ServiceCommonNameAndIssuer"
                 }
               ]
             }
           ],
           ...
   }
   ```

   Para especificar la lista de nombres comunes de servicio y huellas digitales de emisor, agregue una sección [**ApplicationGateway/Http/ServiceCommonNameAndIssuer**](./service-fabric-cluster-fabric-settings.md#applicationgatewayhttpservicecommonnameandissuer) en **fabricSettings**, tal y como se muestra a continuación. Pueden agregarse varios pares de huella digital de emisor y nombre común de certificado en la matriz **parameters**. 

   Si el proxy inverso de punto de conexión al que se está conectando presenta un certificado cuya huella digital de emisor y nombre común coinciden con cualquiera de los valores especificados aquí, se establece el canal TLS.
   Si no se puede realizar la coincidencia de los detalles del certificado, el proxy inverso produce un error en la solicitud del cliente con el código de estado 502 (Puerta de enlace incorrecta). La línea de estado HTTP también contendrá la frase "Certificado SSL no válido". 

   ```json
   {
   "fabricSettings": [
             ...
             {
               "name": "ApplicationGateway/Http/ServiceCommonNameAndIssuer",
               "parameters": [
                 {
                   "name": "WinFabric-Test-Certificate-CN1",
                   "value": "b3 44 9b 01 8d 0f 68 39 a2 c5 d6 2b 5b 6c 6a c8 22 b4 22 11"
                 },
                 {
                   "name": "WinFabric-Test-Certificate-CN2",
                   "value": "b3 44 9b 01 8d 0f 68 39 a2 c5 d6 2b 5b 6c 6a c8 22 11 33 44"
                 }
               ]
             }
           ],
           ...
   }
   ```

- **ServiceCertificateThumbprints**: el proxy inverso comprobará el certificado de servicio con proxy en función de su huella digital. Puede elegir ir a esta ruta cuando los servicios estén configurados con certificados autofirmados: Especifique **ApplicationCertificateValidationPolicy** con el valor **ServiceCertificateThumbprints** en la sección [**ApplicationGateway/Http**](./service-fabric-cluster-fabric-settings.md#applicationgatewayhttp).

   ```json
   {
   "fabricSettings": [
             ...
             {
               "name": "ApplicationGateway/Http",
               "parameters": [
                 {
                   "name": "ApplicationCertificateValidationPolicy",
                   "value": "ServiceCertificateThumbprints"
                 }
               ]
             }
           ],
           ...
   }
   ```

   Especifique también las huellas digitales con una entrada **ServiceCertificateThumbprints** en la sección **ApplicationGateway/Http**. Pueden especificarse varias huellas digitales como una lista separada por comas en el campo de valor, tal y como se muestra a continuación:

   ```json
   {
   "fabricSettings": [
             ...
             {
               "name": "ApplicationGateway/Http",
               "parameters": [
                   ...
                 {
                   "name": "ServiceCertificateThumbprints",
                   "value": "78 12 20 5a 39 d2 23 76 da a0 37 f0 5a ed e3 60 1a 7e 64 bf,78 12 20 5a 39 d2 23 76 da a0 37 f0 5a ed e3 60 1a 7e 64 b9"
                 }
               ]
             }
           ],
           ...
   }
   ```

   Si la huella digital del certificado de servidor se muestra en esta entrada de configuración, el proxy inverso realiza correctamente la conexión TLS. En caso contrario, se termina la conexión y se produce un error en la solicitud del cliente con un 502 (Puerta de enlace incorrecta). La línea de estado HTTP también contendrá la frase "Certificado SSL no válido".

## <a name="endpoint-selection-logic-when-services-expose-secure-as-well-as-unsecured-endpoints"></a>Lógica de selección de punto de conexión cuando los servicios exponen tanto puntos de conexión seguros como no seguros
Service Fabric admite la configuración de varios puntos de conexión para un servicio. Para más información, vea [Especificación de recursos en un manifiesto de servicio](service-fabric-service-manifest-resources.md).

El proxy inverso selecciona uno de los puntos de conexión al que reenviar la solicitud en función del parámetro de consulta **ListenerName** del [URI de servicio](./service-fabric-reverseproxy.md#uri-format-for-addressing-services-by-using-the-reverse-proxy). Si no se especifica el parámetro **ListenerName**, el proxy inverso puede elegir cualquier punto de conexión en la lista de puntos de conexión. Según los puntos de conexión configurados para el servicio, el punto de conexión seleccionado puede ser un punto de conexión HTTP o HTTPS. Puede que haya escenarios o requisitos en la que quiera que el proxy inverso funcione en "modo solo seguro"; es decir, no quiere que el proxy inverso seguro reenvíe solicitudes a puntos de conexión no seguros. Para establecer el proxy inverso en modo solo seguro, especifique la entrada de configuración **SecureOnlyMode** con el valor **true** en la sección [**ApplicationGateway/Http**](./service-fabric-cluster-fabric-settings.md#applicationgatewayhttp).   

```json
{
"fabricSettings": [
          ...
          {
            "name": "ApplicationGateway/Http",
            "parameters": [
                ...
              {
                "name": "SecureOnlyMode",
                "value": true
              }
            ]
          }
        ],
        ...
}
```

> [!NOTE]
> Cuando se trabaja en **SecureOnlyMode**, si el cliente ha especificado un **ListenerName** correspondiente a un punto de conexión HTTP (no seguro), el proxy inverso produce un error en la solicitud con un código de estado HTTP 404 (No encontrado).

## <a name="setting-up-client-certificate-authentication-through-the-reverse-proxy"></a>Configuración de la autenticación de certificados de cliente mediante el proxy inverso
Se produce la terminación TLS en el proxy inverso y se pierden todos los datos del certificado de cliente. Para que los servicios autentiquen el certificado de cliente, establezca el valor **ForwardClientCertificate** en la sección [**ApplicationGateway/Http**](./service-fabric-cluster-fabric-settings.md#applicationgatewayhttp).

1. Cuando **ForwardClientCertificate** está establecido en **false**, el proxy inverso no solicitará el certificado de cliente durante el protocolo de enlace TLS con el cliente.
Este es el comportamiento predeterminado.

2. Cuando **ForwardClientCertificate** está establecido en **true**, el proxy inverso solicita el certificado del cliente durante el protocolo de enlace TLS con el cliente.
Luego, reenvía los datos del certificado de cliente en un encabezado HTTP personalizado denominado **X-Client-Certificate**. El valor del encabezado es la cadena en formato PEM con codificación base64 del certificado del cliente. El servicio puede completar correctamente la solicitud o no con el código de estado adecuado después de inspeccionar los datos del certificado.
Si el cliente no presenta un certificado, el proxy inverso reenvía un encabezado vacío y deja que el servicio gestione el caso.

> [!NOTE]
> El proxy inverso actúa únicamente como servicio de reenvío. No validará en modo alguno el certificado del cliente.


## <a name="next-steps"></a>Pasos siguientes
* [Instalación y configuración del proxy inverso en un clúster](service-fabric-reverseproxy-setup.md)
* Consulte [Configuración de un proxy inverso para conectarse a servicios seguros](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/Reverse-Proxy-Sample#configure-reverse-proxy-to-connect-to-secure-services)
* Vea un ejemplo de comunicación HTTP entre los servicios de un [proyecto de ejemplo en GitHub](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started).
* [Llamadas a procedimiento remoto con la comunicación remota de Reliable Services](service-fabric-reliable-services-communication-remoting.md)
* [API web que usa OWIN en Reliable Services](./service-fabric-reliable-services-communication-aspnetcore.md)
* [Administración de certificados del clúster](service-fabric-cluster-security-update-certs-azure.md)