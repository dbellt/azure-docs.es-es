---
title: 'Tutorial: Uso de OpenSSL para crear certificados autofirmados para Azure IoT Hub | Microsoft Docs'
description: 'Tutorial: Uso de OpenSSL para crear certificados X.509 autofirmados para Azure IoT Hub'
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
ms.openlocfilehash: 69bef0938a65777d025e2ffa64de6f133416fc7f
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110067139"
---
# <a name="tutorial-using-openssl-to-create-self-signed-certificates"></a>Tutorial: Uso de OpenSSL para crear certificados autofirmados

Puede autenticar un dispositivo en la instancia de IoT Hub mediante dos certificados de dispositivo autofirmados. Esto se denomina a veces autenticación de huella digital porque los certificados contienen huellas digitales (valores hash) que se envían a IoT Hub. En los pasos siguientes se indica cómo crear dos certificados autofirmados.

## <a name="step-1---create-a-key-for-the-first-certificate"></a>Paso 1: Creación de una clave para el primer certificado

```bash
openssl genpkey -out device1.key -algorithm RSA -pkeyopt rsa_keygen_bits:2048
```

## <a name="step-2---create-a-csr-for-the-first-certificate"></a>Paso 2: Creación de un CSR para el primer certificado

Asegúrese de especificar el identificador de dispositivo cuando se le solicite.

```bash
openssl req -new -key device1.key -out device1.csr

Country Name (2 letter code) [XX]:.
State or Province Name (full name) []:.
Locality Name (eg, city) [Default City]:.
Organization Name (eg, company) [Default Company Ltd]:.
Organizational Unit Name (eg, section) []:.
Common Name (eg, your name or your server hostname) []:{your-device-id}
Email Address []:

```

## <a name="step-3---check-the-csr"></a>Paso 3: Comprobación del puerto del RDP

```bash
openssl req -text -in device1.csr -noout
```

## <a name="step-4---self-sign-certificate-1"></a>Paso 4: Certificado autofirmado 1

```bash
openssl x509 -req -days 365 -in device1.csr -signkey device1.key -out device.crt
```

## <a name="step-5---create-a-key-for-certificate-2"></a>Paso 5: Creación de una clave para el certificado 2

Cuando se le solicite, especifique el mismo identificador de dispositivo que usó para el certificado 1.

```bash
openssl req -new -key device2.key -out device2.csr

Country Name (2 letter code) [XX]:.
State or Province Name (full name) []:.
Locality Name (eg, city) [Default City]:.
Organization Name (eg, company) [Default Company Ltd]:.
Organizational Unit Name (eg, section) []:.
Common Name (eg, your name or your server hostname) []:{your-device-id}
Email Address []:

```

## <a name="step-6---self-sign-certificate-2"></a>Paso 6: Certificado autofirmado 2

```bash
openssl x509 -req -days 365 -in device2.csr -signkey device2.key -out device2.crt
```

## <a name="step-7---retrieve-the-thumbprint-for-certificate-1"></a>Paso 7: Recuperación de la huella digital del certificado 1

```bash
openssl x509 -in device.crt -noout -fingerprint
```

## <a name="step-8---retrieve-the-thumbprint-for-certificate-2"></a>Paso 8: Recuperación de la huella digital del certificado 2

```bash
openssl x509 -in device2.crt -noout -fingerprint
```

## <a name="step-9---create-a-new-iot-device"></a>Paso 9: Creación de una plantilla de dispositivo IoT

Vaya a la instancia de IoT Hub en Azure Portal y cree una identidad del dispositivo IoT con las características siguientes:

* Proporcione el **identificador de dispositivo** que coincida con el nombre de sujeto de los certificados de dispositivo.
* Seleccione el tipo de autenticación **X.509 autofirmado**.
* Pegue las huellas digitales hexadecimales que ha copiado de los certificados principal y secundario del dispositivo. Asegúrese de que las cadenas hexadecimales no tengan ningún delimitador de dos puntos.

## <a name="next-steps"></a>Pasos siguientes

Vaya a la [prueba de la autenticación de certificados](tutorial-x509-test-certificate.md) para determinar si el certificado puede autenticar el dispositivo en el IoT Hub.
