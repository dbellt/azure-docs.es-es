---
title: Generación de certificados autofirmados con una entidad de certificación raíz personalizada
titleSuffix: Azure Application Gateway
description: Aprenda a generar un certificado autofirmado de Azure Application Gateway con una entidad de certificación raíz personalizada.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: how-to
ms.date: 07/23/2019
ms.author: victorh
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 602f766429045d91748c0f0873744fe1d35b621d
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110467019"
---
# <a name="generate-an-azure-application-gateway-self-signed-certificate-with-a-custom-root-ca"></a>Generación de un certificado autofirmado de Azure Application Gateway con una entidad de certificación raíz personalizada

La SKU de Application Gateway v2 introduce el uso de certificados raíz de confianza para permitir servidores back-end. De este modo, se quitan los certificados de autenticación necesarios en la SKU v1. El *certificado raíz* es un certificado raíz en formato X.509 (.CER) codificado en base 64 del servidor de certificados de back-end. Identifica la entidad de certificación raíz que emitió el certificado de servidor que, a continuación, se usa para la comunicación TLS/SSL.

Application Gateway confía en el certificado del sitio web de forma predeterminada si está firmado por una entidad de certificación conocida (por ejemplo, GoDaddy o DigiCert). En ese caso, no es necesario cargar explícitamente el certificado raíz. Para obtener más información, consulte [Introducción a la terminación TLS y a TLS de extremo a extremo con Application Gateway](ssl-overview.md). Sin embargo, si tiene un entorno de desarrollo y pruebas y no quiere comprar ningún certificado firmado por una entidad de certificación comprobada, puede crear su propia entidad de certificación personalizada y crear un certificado autofirmado. 

> [!NOTE]
> De forma predeterminada, los certificados autofirmados no son de confianza y pueden ser difíciles de mantener. Además, pueden usar conjuntos hash y de cifrado no actualizados que podrían no ser seguros. Para mejorar la seguridad, compre un certificado firmado por una entidad de certificación conocida.

En este artículo, aprenderá a:

- Crear su propia entidad de certificación personalizada
- Crear un certificado autofirmado firmado por la entidad de certificación personalizada
- Cargar un certificado raíz autofirmado en Application Gateway para autenticar el servidor back-end

## <a name="prerequisites"></a>Prerrequisitos

- **[OpenSSL](https://www.openssl.org/) en un equipo con Windows o Linux** 

   Aunque podría haber otras herramientas disponibles para la administración de certificados, en este tutorial se usa OpenSSL. Puede encontrar OpenSSL incluido en muchas distribuciones de Linux, como Ubuntu.
- **Un servidor web**

   Por ejemplo, Apache, IIS o NGINX para probar los certificados.

- **SKU de Application Gateway v2**
   
  Si no tiene ningún elemento de Application Gateway, consulte [Inicio rápido: Dirección del tráfico web con Azure Application Gateway: Azure Portal](quick-create-portal.md)

## <a name="create-a-root-ca-certificate"></a>Creación de un certificado de entidad de certificación raíz

Cree el certificado de entidad de certificación raíz con OpenSSL.

### <a name="create-the-root-key"></a>Creación de la clave raíz

1. Inicie sesión en el equipo donde está instalado OpenSSL y ejecute el siguiente comando. Este comando crea una clave cifrada.

   ```
   openssl ecparam -out contoso.key -name prime256v1 -genkey
   ```
   
### <a name="create-a-root-certificate-and-self-sign-it"></a>Creación de un certificado raíz y autofirmado

1. Use los siguientes comandos para generar la CSR y el certificado.

   ```
   openssl req -new -sha256 -key contoso.key -out contoso.csr
   ```
   
   ```
   openssl x509 -req -sha256 -days 365 -in contoso.csr -signkey contoso.key -out contoso.crt
   ```
   Los comandos anteriores crean el certificado raíz. Lo usará para firmar el certificado de servidor.

1. Cuando se le solicite, escriba la contraseña de la clave raíz y la información de la organización de la entidad de certificación personalizada, como país o región, estado, organización, unidad organizativa y nombre de dominio completo (es decir, el dominio del emisor).

   ![creación de un certificado raíz](media/self-signed-certificates/root-cert.png)

## <a name="create-a-server-certificate"></a>Crear un certificado de servidor

A continuación, creará un certificado de servidor con OpenSSL.

### <a name="create-the-certificates-key"></a>Crear la clave del certificado

Use el siguiente comando para generar la clave para el certificado de servidor.

   ```
   openssl ecparam -out fabrikam.key -name prime256v1 -genkey
   ```

### <a name="create-the-csr-certificate-signing-request"></a>Cree la CSR (solicitud de firma de certificados).

La CSR es una clave pública que se asigna a una entidad de certificación al solicitar un certificado. La entidad de certificación emite el certificado para esta solicitud específica.

> [!NOTE]
> El nombre común (CN) del certificado de servidor debe ser diferente del dominio del emisor. Por ejemplo, en este caso, el CN del emisor es `www.contoso.com` y el CN del certificado de servidor es `www.fabrikam.com`.


1. Use el comando siguiente para generar la CSR:

   ```
   openssl req -new -sha256 -key fabrikam.key -out fabrikam.csr
   ```

1. Cuando se le solicite, escriba la contraseña de la clave raíz y la información de la organización de la entidad de certificación personalizada: País o región, estado, organización, unidad organizativa y nombre de dominio completo. Este es el dominio del sitio web y debe ser diferente del emisor.

   ![Certificado de servidor](media/self-signed-certificates/server-cert.png)

### <a name="generate-the-certificate-with-the-csr-and-the-key-and-sign-it-with-the-cas-root-key"></a>Generación del certificado con la CSR y la clave y firma con la clave raíz de la entidad de certificación

1. Use el siguiente comando para crear el certificado:

   ```
   openssl x509 -req -in fabrikam.csr -CA  contoso.crt -CAkey contoso.key -CAcreateserial -out fabrikam.crt -days 365 -sha256
   ```
### <a name="verify-the-newly-created-certificate"></a>Comprobación del certificado recién creado

1. Use el siguiente comando para imprimir el resultado del archivo CRT y comprobar su contenido:

   ```
   openssl x509 -in fabrikam.crt -text -noout
   ```

   ![Comprobación de certificados](media/self-signed-certificates/verify-cert.png)

1. Compruebe los archivos del directorio y asegúrese de que tiene los siguientes archivos:

   - contoso.crt
   - contoso.key
   - fabrikam.crt
   - fabrikam.key

## <a name="configure-the-certificate-in-your-web-servers-tls-settings"></a>Configuración del certificado en la configuración de TLS del servidor web

En el servidor web, configure TLS con los archivos fabrikam.crt y fabrikam.key. Si el servidor web no puede recibir dos archivos, puede combinarlos en un único archivo .pem o .pfx mediante comandos de OpenSSL.

### <a name="iis"></a>IIS

Para obtener instrucciones sobre cómo importar el certificado y cargarlo como un certificado de servidor en IIS, consulte [Procedimiento: Instalación de certificados importados en un servidor web en Windows Server 2003](https://support.microsoft.com/help/816794/how-to-install-imported-certificates-on-a-web-server-in-windows-server).

Para obtener instrucciones de enlace TLS, consulte [Configuración de SSL en IIS 7](/iis/manage/configuring-security/how-to-set-up-ssl-on-iis#create-an-ssl-binding-1).

### <a name="apache"></a>Apache

La configuración siguiente es un ejemplo de [host virtual configurado para SSL](https://cwiki.apache.org/confluence/display/HTTPD/NameBasedSSLVHosts) en Apache:

```
<VirtualHost www.fabrikam:443>
      DocumentRoot /var/www/fabrikam
      ServerName www.fabrikam.com
      SSLEngine on
      SSLCertificateFile /home/user/fabrikam.crt
      SSLCertificateKeyFile /home/user/fabrikam.key
</VirtualHost>
```

### <a name="nginx"></a>NGINX

La configuración siguiente es un ejemplo de [bloque de servidor NGINX](https://nginx.org/docs/http/configuring_https_servers.html) con configuración de TLS:

![NGINX con TLS](media/self-signed-certificates/nginx-ssl.png)

## <a name="access-the-server-to-verify-the-configuration"></a>Acceso al servidor para comprobar la configuración

1. Agregue el certificado raíz al almacén raíz de confianza de la máquina. Al acceder al sitio web, asegúrese de que toda la cadena de certificados se vea en el explorador.

   ![Certificados raíz de confianza](media/self-signed-certificates/trusted-root-cert.png)

   > [!NOTE]
   > Se supone que el DNS se configuró para que apunte el nombre del servidor web (en este ejemplo, www.fabrikam.com) a la dirección IP del servidor web. Si no es así, puede editar el [archivo de host](https://answers.microsoft.com/en-us/windows/forum/all/how-to-edit-host-file-in-windows-10/7696f204-2aaf-4111-913b-09d6917f7f3d) para resolver el nombre.
1. Vaya a su sitio web y haga clic en el icono de candado situado en el cuadro de dirección del explorador para comprobar la información del sitio y el certificado.

## <a name="verify-the-configuration-with-openssl"></a>Comprobación de la configuración con OpenSSL

O bien, puede usar OpenSSL para comprobar el certificado.

```
openssl s_client -connect localhost:443 -servername www.fabrikam.com -showcerts
```

![Comprobación de certificado OpenSSL](media/self-signed-certificates/openssl-verify.png)

## <a name="upload-the-root-certificate-to-application-gateways-http-settings"></a>Carga del certificado raíz en la configuración HTTP de Application Gateway

Para cargar el certificado en Application Gateway, debe exportar el certificado .crt con un formato .cer codificado en base 64. Dado que .crt ya contiene la clave pública en el formato codificado en base 64, solo tiene que cambiar el nombre de la extensión del archivo de .crt a .cer. 

### <a name="azure-portal"></a>Portal de Azure

Para cargar el certificado raíz de confianza desde el portal, seleccione **Configuración HTTP** y elija el protocolo **HTTPS**.

![Adición de un certificado mediante el portal](media/self-signed-certificates/portal-cert.png)

### <a name="azure-powershell"></a>Azure PowerShell

O bien, puede usar CLI de Azure o Azure PowerShell para cargar el certificado raíz. El código siguiente es un ejemplo de Azure PowerShell.

> [!NOTE]
> En el ejemplo siguiente se agrega un certificado raíz de confianza a Application Gateway, se crea una nueva configuración de HTTP y se agrega una nueva regla, suponiendo que el grupo de back-end y el cliente de escucha ya existen.

```azurepowershell
## Add the trusted root certificate to the Application Gateway

$gw=Get-AzApplicationGateway -Name appgwv2 -ResourceGroupName rgOne

Add-AzApplicationGatewayTrustedRootCertificate `
   -ApplicationGateway $gw `
   -Name CustomCARoot `
   -CertificateFile "C:\Users\surmb\Downloads\contoso.cer"

$trustedroot = Get-AzApplicationGatewayTrustedRootCertificate `
   -Name CustomCARoot `
   -ApplicationGateway $gw

## Get the listener, backend pool and probe

$listener = Get-AzApplicationGatewayHttpListener `
   -Name basichttps `
   -ApplicationGateway $gw

$bepool = Get-AzApplicationGatewayBackendAddressPool `
  -Name testbackendpool `
  -ApplicationGateway $gw

Add-AzApplicationGatewayProbeConfig `
  -ApplicationGateway $gw `
  -Name testprobe `
  -Protocol Https `
  -HostName "www.fabrikam.com" `
  -Path "/" `
  -Interval 15 `
  -Timeout 20 `
  -UnhealthyThreshold 3

$probe = Get-AzApplicationGatewayProbeConfig `
  -Name testprobe `
  -ApplicationGateway $gw

## Add the configuration to the HTTP Setting and don't forget to set the "hostname" field
## to the domain name of the server certificate as this will be set as the SNI header and
## will be used to verify the backend server's certificate. Note that TLS handshake will
## fail otherwise and might lead to backend servers being deemed as Unhealthy by the probes

Add-AzApplicationGatewayBackendHttpSettings `
  -ApplicationGateway $gw `
  -Name testbackend `
  -Port 443 `
  -Protocol Https `
  -Probe $probe `
  -TrustedRootCertificate $trustedroot `
  -CookieBasedAffinity Disabled `
  -RequestTimeout 20 `
  -HostName www.fabrikam.com

## Get the configuration and update the Application Gateway

$backendhttp = Get-AzApplicationGatewayBackendHttpSettings `
  -Name testbackend `
  -ApplicationGateway $gw

Add-AzApplicationGatewayRequestRoutingRule `
  -ApplicationGateway $gw `
  -Name testrule `
  -RuleType Basic `
  -BackendHttpSettings $backendhttp `
  -HttpListener $listener `
  -BackendAddressPool $bepool

Set-AzApplicationGateway -ApplicationGateway $gw 
```

### <a name="verify-the-application-gateway-backend-health"></a>Comprobación del estado de back-end de una instancia de Application Gateway

1. Haga clic en la vista **Estado de back-end** de la instancia de Application Gateway para comprobar si el estado del sondeo es correcto.
1. Debería ver que el estado es **Correcto** para el sondeo HTTPS.

![Sondeo HTTPS](media/self-signed-certificates/https-probe.png)

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre SSL\TLS en Application Gateway, consulte [Introducción a la terminación TLS y a TLS de extremo a extremo con Application Gateway](ssl-overview.md).
