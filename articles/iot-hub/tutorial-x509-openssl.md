---
title: 'Tutorial: Uso de OpenSSL para crear certificados de prueba X.509 para Azure IoT Hub | Microsoft Docs'
description: 'Tutorial: Uso de OpenSSL para crear certificados de entidad de certificación y de dispositivo para Azure IoT Hub'
author: v-gpettibone
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: tutorial
ms.date: 02/26/2021
ms.author: robinsh
ms.custom:
- mvc
- 'Role: Cloud Development'
- 'Role: Data Analytics'
ms.openlocfilehash: 0843e5d3a5e91cb4acdf18ad6bdf6f4f0c214f72
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107378302"
---
# <a name="tutorial-using-openssl-to-create-test-certificates"></a>Tutorial: Uso de OpenSSL para crear certificados de prueba

Aunque puede adquirir certificados X.509 de una entidad de certificación de confianza, la creación de su propia jerarquía de certificados de prueba o el uso de certificados autofirmados es adecuado para probar la autenticación de dispositivos de IoT Hub. En el ejemplo siguiente se usa [OpenSSL](https://www.openssl.org/) y [OpenSSL Cookbook](https://www.feistyduck.com/library/openssl-cookbook/online/ch-openssl.html) para crear una entidad de certificación, una entidad de certificación subordinada y un certificado de dispositivo. A continuación, en el ejemplo se firma la entidad de certificación subordinada y el certificado de dispositivo en una jerarquía de certificados. Esto se presenta únicamente con fines de ejemplo.

## <a name="step-1---create-the-root-ca-directory-structure"></a>Paso 1: Creación de la estructura de directorios de la entidad de certificación raíz

Cree una estructura de directorios para la entidad de certificación.

* El directorio **certs** almacena los certificados nuevos.
* El directorio **db** se utiliza para la base de datos de certificados.
* El directorio **private** almacena la clave privada de la entidad de certificación.

```bash
  mkdir rootca
  cd rootca
  mkdir certs db private
  touch db/index
  openssl rand -hex 16 > db/serial
  echo 1001 > db/crlnumber
```

## <a name="step-2---create-a-root-ca-configuration-file"></a>Paso 2: Creación de un archivo de configuración de la entidad de certificación raíz

Antes de crear una entidad de certificación, cree un archivo de configuración y guárdelo como `rootca.conf` en el directorio rootca.

```xml
[default]
name                     = rootca
domain_suffix            = example.com
aia_url                  = http://$name.$domain_suffix/$name.crt
crl_url                  = http://$name.$domain_suffix/$name.crl
default_ca               = ca_default
name_opt                 = utf8,esc_ctrl,multiline,lname,align

[ca_dn]
commonName               = "Test Root CA"

[ca_default]
home                     = ../rootca 
database                 = $home/db/index
serial                   = $home/db/serial
crlnumber                = $home/db/crlnumber
certificate              = $home/$name.crt
private_key              = $home/private/$name.key
RANDFILE                 = $home/private/random
new_certs_dir            = $home/certs
unique_subject           = no
copy_extensions          = none
default_days             = 3650
default_crl_days         = 365
default_md               = sha256
policy                   = policy_c_o_match

[policy_c_o_match]
countryName              = optional
stateOrProvinceName      = optional
organizationName         = optional
organizationalUnitName   = optional
commonName               = supplied
emailAddress             = optional

[req]
default_bits             = 2048
encrypt_key              = yes
default_md               = sha256
utf8                     = yes
string_mask              = utf8only
prompt                   = no
distinguished_name       = ca_dn
req_extensions           = ca_ext

[ca_ext]
basicConstraints         = critical,CA:true
keyUsage                 = critical,keyCertSign,cRLSign
subjectKeyIdentifier     = hash

[sub_ca_ext]
authorityKeyIdentifier   = keyid:always
basicConstraints         = critical,CA:true,pathlen:0
extendedKeyUsage         = clientAuth,serverAuth
keyUsage                 = critical,keyCertSign,cRLSign
subjectKeyIdentifier     = hash

[client_ext]
authorityKeyIdentifier   = keyid:always
basicConstraints         = critical,CA:false
extendedKeyUsage         = clientAuth
keyUsage                 = critical,digitalSignature
subjectKeyIdentifier     = hash

```

## <a name="step-3---create-a-root-ca"></a>Paso 3: Creación de una entidad de certificación raíz

En primer lugar, genere la clave y la solicitud de firma de certificado (CSR) en el directorio rootca.

```bash
  openssl req -new -config rootca.conf -out rootca.csr -keyout private/rootca.key
```

A continuación, cree un certificado de entidad de certificación autofirmado. La firma automática es adecuada para la realización de pruebas. Especifique las extensiones ca_ext del archivo de configuración en la línea de comandos. Estas indican que el certificado es para una entidad de certificación raíz y se pueden usar para firmar certificados y listas de revocación de certificados (CRL). Firme el certificado y confírmelo en la base de datos.

```bash
  openssl ca -selfsign -config rootca.conf -in rootca.csr -out rootca.crt -extensions ca_ext
```

## <a name="step-4---create-the-subordinate-ca-directory-structure"></a>Paso 4: Creación de la estructura de directorios de la entidad de certificación subordinada

Cree una estructura de directorios para la entidad de certificación subordinada.

```bash
  mkdir subca
  cd subca
  mkdir certs db private
  touch db/index
  openssl rand -hex 16 > db/serial
  echo 1001 > db/crlnumber
```

## <a name="step-5---create-a-subordinate-ca-configuration-file"></a>Paso 5: Creación de un archivo de configuración de entidad de certificación subordinada

Cree un archivo de configuración y guárdelo como subca.conf en el directorio `subca`.

```bash
[default]
name                     = subca
domain_suffix            = example.com
aia_url                  = http://$name.$domain_suffix/$name.crt
crl_url                  = http://$name.$domain_suffix/$name.crl
default_ca               = ca_default
name_opt                 = utf8,esc_ctrl,multiline,lname,align

[ca_dn]
commonName               = "Test Subordinate CA"

[ca_default]
home                     = . 
database                 = $home/db/index
serial                   = $home/db/serial
crlnumber                = $home/db/crlnumber
certificate              = $home/$name.crt
private_key              = $home/private/$name.key
RANDFILE                 = $home/private/random
new_certs_dir            = $home/certs
unique_subject           = no
copy_extensions          = copy 
default_days           
default_crl_days         = 90 
default_md               = sha256
policy                   = policy_c_o_match

[policy_c_o_match]
countryName              = optional
stateOrProvinceName      = optional
organizationName         = optional
organizationalUnitName   = optional
commonName               = supplied
emailAddress             = optional

[req]
default_bits             = 2048
encrypt_key              = yes
default_md               = sha256
utf8                     = yes
string_mask              = utf8only
prompt                   = no
distinguished_name       = ca_dn
req_extensions           = ca_ext

[ca_ext]
basicConstraints         = critical,CA:true
keyUsage                 = critical,keyCertSign,cRLSign
subjectKeyIdentifier     = hash

[sub_ca_ext]
authorityKeyIdentifier   = keyid:always
basicConstraints         = critical,CA:true,pathlen:0
extendedKeyUsage         = clientAuth,serverAuth
keyUsage                 = critical,keyCertSign,cRLSign
subjectKeyIdentifier     = hash

[client_ext]
authorityKeyIdentifier   = keyid:always
basicConstraints         = critical,CA:false
extendedKeyUsage         = clientAuth
keyUsage                 = critical,digitalSignature
subjectKeyIdentifier     = hash
```

## <a name="step-6---create-a-subordinate-ca"></a>Paso 6: Creación de una entidad de certificación subordinada

Cree un nuevo número de serie en el archivo `rootca/db/serial` del certificado de la entidad de certificación subordinada.

```bash
  openssl rand -hex 16 > db/serial
```

>[!IMPORTANT]
>Debe crear un nuevo número de serie para cada certificado de entidad de certificación subordinada y para cada certificado de dispositivo que cree. Los distintos certificados no pueden tener el mismo número de serie.

En este ejemplo se muestra cómo crear una entidad de certificación subordinada o de registro. Dado que puede usar la entidad de certificación raíz para firmar certificados, la creación de una entidad de certificación subordinada no es estrictamente necesaria. Sin embargo, tener una entidad de certificación subordinada imita las jerarquías de certificados del mundo real en las que la entidad de certificación raíz se mantiene sin conexión y las entidades de certificación subordinadas emiten certificados de cliente.

Use el archivo de configuración para generar una clave y una solicitud de firma de certificado (CSR).

```bash
  openssl req -new -config subca.conf -out subca.csr -keyout private/subca.key
```

Envíe la solicitud de firma de certificado a la entidad de certificación raíz y use esta para emitir y firmar el certificado de la entidad de certificación subordinada. Especifique sub_ca_ext para el modificador de extensiones en la línea de comandos. Las extensiones indican que el certificado es para una entidad de certificación raíz que puede firmar certificados y listas de revocación de certificados (CRL). Cuando se le solicite, firme el certificado y confírmelo en la base de datos.

```bash
  openssl ca -config ../rootca/rootca.conf -in subca.csr -out subca.crt -extensions sub_ca_ext
```

## <a name="step-7---demonstrate-proof-of-possession"></a>Paso 7: Demostración de la prueba de posesión

Ahora tiene un certificado de entidad de certificación raíz y un certificado de entidad de certificación subordinada. Puede usar cualquiera de ellos para firmar los certificados de dispositivo. El que elija debe cargarse en la instancia de IoT Hub. En los pasos siguientes se da por supuesto que está usando el certificado de la entidad de certificación subordinada. Para cargar y registrar el certificado de la entidad de certificación subordinada en la instancia de IoT Hub:

1. En Azure Portal, vaya a la instancia de IoT Hub y seleccione **Configuración > Certificados**.

1. Seleccione **Agregar** para agregar el nuevo certificado de la entidad de certificación subordinada.

1. Escriba un nombre para mostrar en el campo **Nombre del certificado** y seleccione el archivo de certificado PEM que creó anteriormente.

1. Seleccione **Guardar**. El certificado se muestra en la lista de certificados con el estado **Sin comprobar**. El proceso de comprobación demostrará que es el propietario del certificado.

   
1. Seleccione el certificado para ver el cuadro de diálogo **Detalles del certificado**.

1. Seleccione **Generar código de verificación**. Para más información, consulte [Demostración de la posesión de un certificado de entidad de certificación](tutorial-x509-prove-possession.md).

1. Copie este código de verificación en el portapapeles. Debe establecer el código de verificación como asunto del certificado. Por ejemplo, si el código de verificación es BB0C656E69AF75E3FB3C8D922C1760C58C1DA5B05AAA9D0A, agréguelo como firmante del certificado, tal como se muestra en el paso 9.

1. Genere una clave privada.

  ```bash
    $ openssl genpkey -out pop.key -algorithm RSA -pkeyopt rsa_keygen_bits:2048
  ```

9. Genere una solicitud de firma de certificado (CSR) para una clave privada existente. Agregue el código de verificación como firmante del certificado.

  ```bash
  openssl req -new -key pop.key -out pop.csr
  
    -----
    Country Name (2 letter code) [XX]:.
    State or Province Name (full name) []:.
    Locality Name (eg, city) [Default City]:.
    Organization Name (eg, company) [Default Company Ltd]:.
    Organizational Unit Name (eg, section) []:.
    Common Name (eg, your name or your server hostname) []:BB0C656E69AF75E3FB3C8D922C1760C58C1DA5B05AAA9D0A
    Email Address []:

    Please enter the following 'extra' attributes
    to be sent with your certificate request
    A challenge password []:
    An optional company name []:
 
  ```

10. Cree un certificado con el archivo de configuración de la entidad de certificación raíz y el CSR como certificado de prueba de posesión.

  ```bash
    openssl ca -config rootca.conf -in pop.csr -out pop.crt -extensions client_ext

  ```

11. Seleccione el nuevo certificado en la vista **Detalles de certificado**. Para buscar el archivo PEM, vaya a la carpeta certs.

12. Una vez que el certificado se carga, seleccione **Comprobar**. El estado del certificado de la entidad de certificación debe cambiar a **Comprobado**.

## <a name="step-8---create-a-device-in-your-iot-hub"></a>Paso 8: Creación de un dispositivo en la instancia de IoT Hub

Vaya a la instancia de IoT Hub en Azure Portal y cree una identidad del dispositivo IoT con los valores siguientes:

1. Proporcione el **identificador de dispositivo** que coincida con el nombre de sujeto de los certificados de dispositivo.

1. Seleccione el tipo de autenticación **X.509 firmado por CA**.

1. Seleccione **Guardar**.

## <a name="step-9---create-a-client-device-certificate"></a>Paso 9: Creación de un certificado de dispositivo cliente

Para generar un certificado de cliente, primero debe generar una clave privada. El siguiente comando muestra cómo usar OpenSSL para crear una clave privada. Cree la clave en el directorio subca.

```bash
openssl genpkey -out device.key -algorithm RSA -pkeyopt rsa_keygen_bits:2048
```

Cree una solicitud de firma de certificado para la clave. No es necesario que escriba una contraseña de comprobación ni un nombre de empresa opcional. Sin embargo, debe escribir el identificador del dispositivo en el campo Nombre común.

```bash
openssl req -new -key device.key -out device.csr

-----
Country Name (2 letter code) [XX]:.
State or Province Name (full name) []:.
Locality Name (eg, city) [Default City]:.
Organization Name (eg, company) [Default Company Ltd]:.
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your server hostname) []:`<your device ID>`
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:

```

Compruebe que la solicitud de firma de certificado es la que espera.

```bash
openssl req -text -in device.csr -noout
```

Envíe dicha solicitud a la entidad de certificación subordinada para iniciar sesión en la jerarquía de certificados. Especifique `client_ext` en el modificador `-extensions`. Tenga en cuenta que `Basic Constraints` en el certificado emitido indica que este certificado no es para una entidad de certificación. Si va a firmar varios certificados, asegúrese de actualizar el número de serie antes de generar cada certificado mediante el comando `rand -hex 16 > db/serial` de OpenSSL.

```bash
openssl ca -config subca.conf -in device.csr -out device.crt -extensions client_ext
```

## <a name="next-steps"></a>Pasos siguientes

Vaya a la [prueba de la autenticación de certificados](tutorial-x509-test-certificate.md) para determinar si el certificado puede autenticar el dispositivo en el IoT Hub.
