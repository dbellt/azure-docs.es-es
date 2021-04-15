---
title: archivo de inclusión
description: archivo de inclusión
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 10/29/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: faab8aa124ca2f290938cb6cff0a2f4d072caffd
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/31/2021
ms.locfileid: "106073340"
---
## <a name="create-a-self-signed-root-certificate"></a><a name="rootcert"></a>Creación de un certificado raíz autofirmado

Use el cmdlet New-SelfSignedCertificate para crear un certificado raíz autofirmado. Para obtener información adicional sobre los parámetros, consulte [New-SelfSignedCertificate](/powershell/module/pkiclient/new-selfsignedcertificate).

1. En un equipo con Windows 10 o Windows Server 2016, abra una consola de Windows PowerShell con privilegios elevados. Estos ejemplos no funcionan en Azure Cloud Shell "Try It". Debe ejecutar estos ejemplos localmente.
1. Utilice el ejemplo siguiente para crear el certificado raíz autofirmado. En el ejemplo siguiente se crea un certificado raíz firmado automáticamente con el nombre "P2SRootCert" que se instala automáticamente en "Certificates-Current User\Personal\Certificates". Puede ver el certificado si abre *certmgr.msc*, o bien *Administrar certificados de usuario*.

   Utilice el cmdlet `Connect-AzAccount` para iniciar sesión. Luego, ejecute el siguiente ejemplo con las modificaciones necesarias.

   ```powershell
   $cert = New-SelfSignedCertificate -Type Custom -KeySpec Signature `
   -Subject "CN=P2SRootCert" -KeyExportPolicy Exportable `
   -HashAlgorithm sha256 -KeyLength 2048 `
   -CertStoreLocation "Cert:\CurrentUser\My" -KeyUsageProperty Sign -KeyUsage CertSign
   ```

1. Deje abierta la consola de PowerShell y continúe con los pasos siguientes para generar un certificado de cliente.

## <a name="generate-a-client-certificate"></a><a name="clientcert"></a>Generación de un certificado de cliente

Cada equipo cliente que se conecta a una red virtual con una conexión de punto a sitio debe tener instalado un certificado de cliente. Puede generarlo desde un certificado raíz autofirmado y, luego, exportar e instalar el certificado de cliente. Si no está instalado el certificado de cliente, se produce un error de autenticación. 

Los pasos siguientes lo llevan por el proceso de generación de un certificado de cliente a partir de un certificado autofirmado. Puede generar varios certificados de cliente desde el mismo certificado raíz. Al generar certificados de cliente mediante los pasos siguientes, el certificado de cliente se instala automáticamente en el equipo que se usó para generar el certificado. Si desea instalar un certificado de cliente en otro equipo cliente, puede exportar el certificado.

Los ejemplos usan el cmdlet New-SelfSignedCertificate para generar un certificado de cliente que expira en un año. Para obtener información adicional sobre los parámetros (por ejemplo, cómo establecer un valor de expiración diferente para el certificado de cliente), consulte [New-SelfSignedCertificate](/powershell/module/pkiclient/new-selfsignedcertificate).

### <a name="example-1---powershell-console-session-still-open"></a>Ejemplo 1: sesión de consola de PowerShell aún abierta

Use este ejemplo si no ha cerrado la consola de PowerShell después de crear el certificado raíz autofirmado. Este ejemplo continúa desde la sección anterior y usa la variable "$cert" declarada. Si cerró la consola de PowerShell después de crear el certificado raíz autofirmado o va a crear más certificados de cliente en una nueva sesión de consola de PowerShell, siga los pasos del [Ejemplo 2](#ex2).

Modifique y ejecute el ejemplo para generar un certificado de cliente. Si ejecuta el ejemplo siguiente sin modificación alguna, el resultado es un certificado de cliente con el nombre "P2SChildCert".  Si quiere agregar algo más al nombre del certificado secundario, modifique el valor CN. No cambie el valor de TextExtension cuando ejecute este ejemplo. El certificado de cliente que genera se instala automáticamente en la ruta del equipo "Certificates - Current User\Personal\Certificates".

```powershell
New-SelfSignedCertificate -Type Custom -DnsName P2SChildCert -KeySpec Signature `
-Subject "CN=P2SChildCert" -KeyExportPolicy Exportable `
-HashAlgorithm sha256 -KeyLength 2048 `
-CertStoreLocation "Cert:\CurrentUser\My" `
-Signer $cert -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.2")
```

### <a name="example-2---new-powershell-console-session"></a><a name="ex2"></a>Ejemplo 2: nueva sesión de consola de PowerShell

Si va a crear más certificados de cliente, o bien no está usando la misma sesión de PowerShell que empleó para crear el certificado raíz autofirmado, siga estos pasos:

1. Identifique el certificado raíz autofirmado que se instaló en el equipo. Este cmdlet devuelve una lista de certificados que están instalados en el equipo.

   ```powershell
   Get-ChildItem -Path "Cert:\CurrentUser\My"
   ```

1. Busque el nombre del firmante de la lista devuelta y, luego, copie la huella digital que se encuentra junto a él en un archivo de texto. En el ejemplo siguiente, hay dos certificados. El nombre CN es el nombre del certificado raíz autofirmado a partir del que va a generar un certificado secundario. En este caso, "P2SRootCert".

   ```
   Thumbprint                                Subject
  
   AED812AD883826FF76B4D1D5A77B3C08EFA79F3F  CN=P2SChildCert4
   7181AA8C1B4D34EEDB2F3D3BEC5839F3FE52D655  CN=P2SRootCert
   ```

1. Declare una variable para el certificado raíz con la huella digital del paso anterior. Reemplace la huella digital con la del certificado raíz a partir del que va a generar un certificado secundario.

   ```powershell
   $cert = Get-ChildItem -Path "Cert:\CurrentUser\My\THUMBPRINT"
   ```

   Por ejemplo, al usar la huella digital de P2SRootCert del paso anterior, la variable tendrá el aspecto similar al siguiente:

   ```powershell
   $cert = Get-ChildItem -Path "Cert:\CurrentUser\My\7181AA8C1B4D34EEDB2F3D3BEC5839F3FE52D655"
   ```

1. Modifique y ejecute el ejemplo para generar un certificado de cliente. Si ejecuta el ejemplo siguiente sin modificación alguna, el resultado es un certificado de cliente con el nombre "P2SChildCert". Si quiere agregar algo más al nombre del certificado secundario, modifique el valor CN. No cambie el valor de TextExtension cuando ejecute este ejemplo. El certificado de cliente que genera se instala automáticamente en la ruta del equipo "Certificates - Current User\Personal\Certificates".

   ```powershell
   New-SelfSignedCertificate -Type Custom -DnsName P2SChildCert -KeySpec Signature `
   -Subject "CN=P2SChildCert" -KeyExportPolicy Exportable `
   -HashAlgorithm sha256 -KeyLength 2048 `
   -CertStoreLocation "Cert:\CurrentUser\My" `
   -Signer $cert -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.2")
   ```

## <a name="export-the-root-certificate-public-key-cer"></a><a name="cer"></a>Exportación de la clave pública del certificado raíz (.cer)

[!INCLUDE [Export public key](vpn-gateway-certificates-export-public-key-include.md)]

### <a name="export-the-self-signed-root-certificate-and-private-key-to-store-it-optional"></a>Exportación del certificado raíz autofirmado y clave privada para almacenarlo (opcional)

Puede que desee exportar el certificado autofirmado y almacenarlo de manera segura como copia de seguridad. Si es necesario, más adelante puede instalarlo en otro equipo y generar más certificados de cliente. Para exportar el certificado raíz autofirmado como archivo .pfx, seleccione el certificado raíz y use los mismos pasos descritos en [Exportación de un certificado de cliente](#clientexport).

## <a name="export-the-client-certificate"></a><a name="clientexport"></a>Exportación del certificado de cliente

[!INCLUDE [Export client certificate](vpn-gateway-certificates-export-client-cert-include.md)]