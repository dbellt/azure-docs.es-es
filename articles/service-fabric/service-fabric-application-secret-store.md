---
title: Servicio central de secretos de Azure Service Fabric
description: En este artículo se indica cómo usar el servicio central de secretos en Azure Service Fabric.
author: amenarde
ms.topic: conceptual
ms.date: 04/06/2021
ms.author: anmenard
ms.openlocfilehash: 1c920631d671ad2af9fa2e723fc98bf96462b007
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111950111"
---
# <a name="central-secret-service-in-azure-service-fabric"></a>Servicio central de secretos en Azure Service Fabric 
El Servicio central de secretos (CSS), también conocido como Almacén central de secretos, es un servicio del sistema de Service Fabric diseñado para proteger los secretos de un clúster. CSS facilita la administración de secretos de aplicaciones SF, lo que elimina la necesidad de usar parámetros cifrados.

El Servicio central de secretos es una caché de secretos en clúster duradera y replicada; los secretos almacenados en CSS se cifran en reposo en un certificado de cifrado proporcionado por el cliente. Asimismo, CSS proporciona API de cliente para la administración de secretos, que serán accesibles para las entidades que se autentican como el clúster o un usuario administrador de clústeres. El modelo de aplicación en tiempo de ejecución de Service Fabric se integra con CSS, lo que permite realizar la declaración de parámetros de la aplicación como referencias secretas de CSS. 

CSS también es fundamental para el aprovisionamiento de secretos de aplicaciones declarados como [URI de secretos de KeyVault](service-fabric-keyvault-references.md), en combinación con la [Identidad administrada para aplicaciones de Service Fabric implementadas en Azure](concepts-managed-identity.md).

El Servicio central de secretos no está pensado para sustituir a un servicio de administración de secretos externo y dedicado, como Azure Key Vault. 

  > [!NOTE] 
  > Al activar CSS en un clúster de SF que ejecuta una versión anterior a [7.1. CU3](service-fabric-versions.md#service-fabric-version-name-and-number-reference), la activación puede producir un error y dejar CSS en un estado incorrecto permanentemente si el clúster está configurado en la autenticación de Windows o si `EncryptionCertificateThumbprint` se declara incorrectamente o si el certificado correspondiente no está instalado. En cualquier caso, es aconsejable actualizar el clúster a una versión en tiempo de ejecución de SF más reciente que la 7.1. antes de continuar. 
  
## <a name="enable-central-secrets-service"></a>Habilitación del Servicio central de secretos
Para habilitar Servicio central de secretos, actualice la configuración del clúster, tal como se describe a continuación. Se recomienda usar un certificado de cifrado diferente del certificado de clúster. Este certificado debe estar instalado en todos los nodos.
```json
{ 
    "fabricSettings": [
        {
            "name":  "CentralSecretService",
            "parameters":  [
                {
                    "name":  "DeployedState",
                    "value":  "enabled"
                },
                {
                    "name" : "EncryptionCertificateThumbprint",
                    "value": "<thumbprint>"
                },
                {
                    "name":  "MinReplicaSetSize",
                    "value":  "1"
                },
                {
                    "name":  "TargetReplicaSetSize",
                    "value":  "3"
                }
            ]
        }
    ]
}
```
  > [!NOTE] 
  > El valor de configuración "DeployedState", introducido en la [versión 8.0](service-fabric-versions.md#service-fabric-version-name-and-number-reference) de Service Fabric es el mecanismo preferido para habilitar o deshabilitar CSS; esta función estaba disponible en versiones anteriores mediante el valor de configuración "IsEnabled", que ahora se considera obsoleto.

## <a name="central-secret-service-secret-model"></a>Modelo de secreto del Servicio central de secretos
La API del Servicio central de secretos expone dos tipos: el recurso secreto y la versión secreta. El tipo del recurso secreto representa, conceptualmente, una familia de versiones de un único secreto que se usa para un propósito específico; entre los ejemplos se incluyen: una cadena de conexión, una contraseña y un certificado de punto de conexión. Un objeto del tipo de recurso de secreto contiene metadatos asociados a ese secreto, específicamente el tipo, el tipo de contenido y la descripción. El tipo de versión del secreto representa una instancia determinada del secreto asociado y almacena el texto no cifrado del secreto (cifrado); siguiendo con los ejemplos anteriores, una versión secreta contiene el valor de contraseña actual, un objeto de certificado válido hasta el final del mes, etc. Tras renovar estos secretos, se deben generar nuevas versiones de secretos (y agregarlas a CSS).

Al formalizar el modelo, la información siguiente indica las reglas implementadas y aplicadas en la implementación de CSS:

- Un recurso de secreto puede tener cero o más versiones.
- Cada versión de secreto es un elemento secundario de un recurso de secreto determinado; una versión solo puede tener un recurso primario.
- Se puede eliminar una versión de secreto individual, sin que ello afecte a otras versiones del mismo secreto.
- Tenga en cuenta que la eliminación de un recurso de secreto provoca la eliminación de todas sus versiones.  
- El valor de una versión de secreto es inmutable.
    
El conjunto completo de las API de administración de REST para los recursos de secretos se puede encontrar [aquí](/rest/api/servicefabric/sfclient-index-meshsecrets) y, para las versiones secretas, [aquí](/rest/api/servicefabric/sfclient-index-meshsecretvalues).

### <a name="declare-a-secret-resource"></a>Declaración de un recurso como secreto
Puede crear un recurso secreto mediante la API REST.

  > [!NOTE] 
  > Si el clúster usa la autenticación de Windows sin un certificado HttpGateway, la solicitud de REST se envía a través de un canal HTTP no seguro. Para habilitar TLS en este canal, la definición del clúster debe actualizarse para especificar un certificado de servidor de puerta de enlace de HTTP.

Para crear un recurso secreto `supersecret` mediante la API REST, realice una solicitud PUT a `https://<clusterfqdn>:19080/Resources/Secrets/supersecret?api-version=6.4-preview`. Tendrá que autenticarse mediante un certificado de clúster o un certificado de cliente de administrador para crear un recurso secreto.
```powershell
$json = '{"properties": {"kind": "inlinedValue", "contentType": "text/plain", "description": "supersecret"}}'
Invoke-WebRequest  -Uri https://<clusterfqdn>:19080/Resources/Secrets/supersecret?api-version=6.4-preview -Method PUT -CertificateThumbprint <CertThumbprint> -Body $json
```

### <a name="set-the-secret-value"></a>Establecimiento del valor secreto
Con este script puede usar la API REST para establecer el valor secreto.
```powershell
$Params = '{"properties": {"value": "mysecretpassword"}}'
Invoke-WebRequest -Uri https://<clusterfqdn>:19080/Resources/Secrets/supersecret/values/ver1?api-version=6.4-preview -Method PUT -Body $Params -CertificateThumbprint <ClusterCertThumbprint>
```

### <a name="examine-the-secret-value"></a>Examen del valor del secreto
```powershell
Invoke-WebRequest -CertificateThumbprint <ClusterCertThumbprint> -Method POST -Uri "https:<clusterfqdn>/Resources/Secrets/supersecret/values/ver1/list_value?api-version=6.4-preview"
```

## <a name="use-the-secret-in-your-application"></a>Uso del secreto en la aplicación
Una aplicación puede consumir un secreto de CSS declarándose como una variable de entorno o especificando una ruta de acceso donde se serializará el texto no cifrado secreto. Siga estos pasos para hacer referencia a un secreto de CSS:

1. Agregue una sección al archivo **settings.xml** con el fragmento de código siguiente. Tenga en cuenta que aquí el valor tiene el formato {`secretname:version`}.
```xml
     <Section Name="testsecrets">
      <Parameter Name="TopSecret" Type="SecretsStoreRef" Value="supersecret:ver1"/
     </Section>
```

2. Importe la sección en **ApplicationManifest.xml**.
```xml
<ServiceManifestImport>
<ServiceManifestRef ServiceManifestName="testservicePkg" ServiceManifestVersion="1.0.0" />
<ConfigOverrides />
<Policies>
 <ConfigPackagePolicies CodePackageRef="Code">
   <ConfigPackage Name="Config" SectionName="testsecrets" EnvironmentVariableName="SecretPath" />
   </ConfigPackagePolicies>
</Policies>
</ServiceManifestImport>
```
   
Ejemplo 1: monte los secretos en un contenedor. El único cambio necesario para que los secretos pasen a estar disponibles dentro del contenedor consiste en aplicar `specify` a un punto de montaje en `<ConfigPackage>`.
El fragmento de código siguiente es el archivo **ApplicationManifest.xml** modificado.  
```xml
   <ServiceManifestImport>
       <ServiceManifestRef ServiceManifestName="testservicePkg" ServiceManifestVersion="1.0.0" />
       <ConfigOverrides />
       <Policies>
         <ConfigPackagePolicies CodePackageRef="Code">
           <ConfigPackage Name="Config" SectionName="testsecrets" MountPoint="C:\secrets" EnvironmentVariableName="SecretPath" />
           <!-- Linux Container
            <ConfigPackage Name="Config" SectionName="testsecrets" MountPoint="/mnt/secrets" EnvironmentVariableName="SecretPath" />
           -->
         </ConfigPackagePolicies>
       </Policies>
     </ServiceManifestImport>
```
Los secretos están disponibles en el punto de montaje dentro del contenedor.

Ejemplo 2: enlace un secreto a una variable de entorno de proceso si especifica `Type='SecretsStoreRef`. El fragmento siguiente es un ejemplo de cómo enlazar la versión `ver1` de `supersecret` a la variable de entorno `MySuperSecret` de **ServiceManifest.xml**.

```xml
   <EnvironmentVariables>
     <EnvironmentVariable Name="MySuperSecret" Type="SecretsStoreRef" Value="supersecret:ver1"/>
   </EnvironmentVariables>
```

La variable de entorno `SecretPath` apuntará al directorio donde se almacenan todos los secretos. Cada parámetro incluido en la sección `testsecrets` se almacena en un archivo independiente. Ahora la aplicación puede usar el secreto como se muestra aquí:
```C#
secretValue = IO.ReadFile(Path.Join(Environment.GetEnvironmentVariable("SecretPath"),  "TopSecret"))
```
   
## <a name="rotating-the-central-secret-service-encryption-certificate"></a>Rotación del certificado de cifrado del Servicio central de secretos
Es importante tener en cuenta que los certificados siguen siendo válidos para realizar descifrados, incluso más allá de su expiración. En este momento, se recomienda seguir aprovisionando certificados de cifrado anteriores después de realizar la rotación, para reducir la posibilidad de un bloqueo. La rotación del certificado de cifrado CSS requiere los pasos siguientes:

1. Aprovisione el nuevo certificado en cada nodo del clúster. Recuerde que no debe quitar ni continuar aprovisionando el certificado de cifrado anterior.
2. Inicie una actualización de configuración del clúster para cambiar el valor de `EncryptionCertificateThumbprint` a la huella digital SHA-1 del nuevo certificado.
Tras la finalizar la actualización, CSS volverá a cifrar el contenido existente en el nuevo certificado de cifrado. Todos los secretos agregados a CSS después de este punto se cifrarán directamente en el nuevo certificado de cifrado. Dado que la convergencia para tener todos los secretos protegidos por el nuevo certificado es asincrónica, es importante que el certificado de cifrado anterior permanezca instalado en todos los nodos y esté disponible para CSS.

## <a name="removing-central-secret-service-from-your-cluster"></a>Eliminación del Servicio central de secretos del clúster
La eliminación segura del Servicio central de secretos de un clúster requiere dos actualizaciones. La primera actualización deshabilita CSS de forma funcional, mientras que la segunda actualización elimina el servicio de la definición del clúster, lo que incluye la eliminación permanente de su contenido. Este proceso de dos fases evita la eliminación accidental del servicio y le permite garantizar que no haya dependencias huérfanas en CSS durante el proceso de eliminación. Esta característica está disponible a partir de la versión 8.0 de SF.

### <a name="step-1-update-css-deployedstate-to-removing"></a>Paso 1: actualización de CSS DeployedState para quitar elementos
Actualice la definición de clúster de `"IsEnabled" = "true"` o `"DeployedState" = "enabled"` de a
```json
{
    "name":  "DeployedState",
    "value":  "removing"
}
```
Una vez que el Servicio central de secretos entre en el estado desplegado `Removing`, se rechazarán todas las llamadas API secretas entrantes, ya sean llamadas de REST directas o mediante activaciones de servicios que incluyen los valores SecretStoreRefs o KeyVaultReferences. Cualquier aplicación o componente del clúster que todavía dependa de CSS en este momento pasará al estado de Advertencia. Si esto ocurre, se debe revertir la actualización al estado implementado `Removing`; si esa actualización ya se ha producido correctamente, se debe iniciar una nueva actualización para volver a cambiar CSS a DeployedState = `Enabled`. Si el Servicio central de secretos recibe una solicitud mientras está en el estado implementado `Removing`, devolverá el código HTTP 401 (no autorizado) y se colocará en un estado de mantenimiento de advertencia.

### <a name="step-2-update-css-deployedstate-to-disabled"></a>Paso 2: actualización de CSS DeployedState para deshabilitar elementos
Actualice la definición de clúster de `"DeployedState" = "removing"` a
```json
{
    "name":  "DeployedState",
    "value":  "disabled"
}
```
El Servicio central de secretos ya no debe ejecutarse en el clúster, y no estará presente en la lista de servicios del sistema. Tenga en cuenta que el contenido de CSS se pierde y no se puede recuperar. 

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre la [seguridad de aplicaciones y servicios](service-fabric-application-and-service-security.md).
- Información sobre la [Identidad administrada para las aplicaciones de Service Fabric](concepts-managed-identity.md).
- Expansión de la funcionalidad de CSS con [KeyVaultReferences](service-fabric-keyvault-references.md)