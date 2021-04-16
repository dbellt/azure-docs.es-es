---
title: 'Tutorial: Prueba de que se poseen certificados de CA en Azure IoT Hub | Microsoft Docs'
description: 'Tutorial: Prueba de que posee un certificado de CA para Azure IoT Hub'
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
ms.openlocfilehash: b7740fa1f6a54dcfcc1181dddedcdd5fdb50402c
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107378234"
---
# <a name="tutorial-proving-possession-of-a-ca-certificate"></a>Tutorial: Prueba de posesión de un certificado de CA

Después de cargar un certificado de una entidad de certificación (CA) raíz o un certificado de CA subordinada en el centro de IoT, debe demostrar que posee el certificado:

1. En Azure Portal, vaya a la instancia de IoT Hub y seleccione **Configuración > Certificados**.

2. Seleccione **Agregar** para agregar un certificado de CA nuevo.

3. Escriba un nombre para mostrar en el campo **Nombre del certificado** y seleccione el archivo de certificado PEM que va a agregar.

4. Seleccione **Guardar**. El certificado se muestra en la lista de certificados con el estado **Sin comprobar**. Este proceso de comprobación determinará que es el poseedor del certificado.

5. Seleccione el certificado para ver el cuadro de diálogo **Detalles de certificado**.

6. Seleccione **Generar código de comprobación** en el cuadro de diálogo.

  :::image type="content" source="media/tutorial-x509-prove-possession/certificate-details.png" alt-text="{cuadro de diálogo Detalles de certificado}":::

7. Copie este código de verificación en el portapapeles. Debe establecer el código de verificación como asunto del certificado. Por ejemplo, si el código de verificación es 75B86466DA34D2B04C0C4C9557A119687ADAE7D4732BDDB3, agréguelo como asunto del certificado, tal como se muestra en el paso siguiente.

8. Hay tres formas de generar un certificado de verificación:

    * Si usa el script de PowerShell proporcionado por Microsoft, ejecute `New-CACertsVerificationCert "75B86466DA34D2B04C0C4C9557A119687ADAE7D4732BDDB3"` para crear un certificado denominado `VerifyCert4.cer`. Para más información, consulte el artículo que trata sobre el [uso de scripts proporcionados por Microsoft](tutorial-x509-scripts.md).

    * Si usa el script de Bash proporcionado por Microsoft, ejecute `./certGen.sh create_verification_certificate "75B86466DA34D2B04C0C4C9557A119687ADAE7D4732BDDB3"` para crear un certificado denominado `verification-code.cert.pem`. Para más información, consulte el artículo que trata sobre el [uso de scripts proporcionados por Microsoft](tutorial-x509-scripts.md).

    * Si usa OpenSSL para generar los certificados, primero debe generar una clave privada y, luego, una solicitud de firma de certificado (CSR):

      ```bash
      $ openssl genpkey -out pop.key -algorithm RSA -pkeyopt rsa_keygen_bits:2048

      $ openssl req -new -key pop.key -out pop.csr

      -----
      Country Name (2 letter code) [XX]:.
      State or Province Name (full name) []:.
      Locality Name (eg, city) [Default City]:.
      Organization Name (eg, company) [Default Company Ltd]:.
      Organizational Unit Name (eg, section) []:.
      Common Name (eg, your name or your server hostname) []:75B86466DA34D2B04C0C4C9557A119687ADAE7D4732BDDB3
      Email Address []:

      Please enter the following 'extra' attributes
      to be sent with your certificate request
      A challenge password []:
      An optional company name []:
 
      ```

      Luego, debe crear un certificado mediante el archivo de configuración de CA raíz (se muestra a continuación) o el archivo de configuración de CA subordinada y el CSR.

      ```bash
      openssl ca -config rootca.conf -in pop.csr -out pop.crt -extensions client_ext

      ```

    Para más información, consulte el artículo en el que se explica cómo [usar OpenSSL para crear certificados de prueba](tutorial-x509-openssl.md).

10. Seleccione el nuevo certificado en la vista **Detalles de certificado**.

11. Una vez que el certificado se carga, seleccione **Comprobar**. El estado del certificado de CA debe cambiar a **Comprobado**.
