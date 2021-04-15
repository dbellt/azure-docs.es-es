---
title: Exportación de cadenas de certificados de CA de cliente de confianza para la autenticación de clientes
titleSuffix: Azure Application Gateway
description: Aprenda a exportar cadenas de certificados de CA de cliente de confianza para la autenticación de clientes en Azure Application Gateway
services: application-gateway
author: mscatyao
ms.service: application-gateway
ms.topic: how-to
ms.date: 03/31/2021
ms.author: caya
ms.openlocfilehash: 2329dc7426b223ef2c81dd0e2e607bccf73192e6
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/02/2021
ms.locfileid: "106231040"
---
# <a name="export-a-trusted-client-ca-certificate-chain-to-use-with-client-authentication"></a>Exportación de cadenas de certificados de CA de cliente de confianza para usarlos en la autenticación de clientes
Para configurar la autenticación mutua con el cliente o la autenticación de clientes, Application Gateway requiere que se cargue una cadena de certificados de CA de cliente de confianza en la puerta de enlace. Si tiene varias cadenas de certificados, deberá crear las cadenas por separado y cargarlas como archivos diferentes en Application Gateway. En este artículo, aprenderá a exportar una cadena de certificados de CA de cliente de confianza, que puede usar en la configuración de autenticación de clientes en la puerta de enlace.  

## <a name="prerequisites"></a>Requisitos previos

Para generar la cadena de certificados de CA de cliente de confianza se requiere un certificado de cliente existente. 

## <a name="export-trusted-client-ca-certificate"></a>Exportación de certificados de CA de cliente de confianza

Se requiere un certificado de CA de cliente de confianza para permitir la autenticación de clientes en Application Gateway. En este ejemplo, se usará un certificado TLS/SSL como certificado de cliente, se exportará su clave pública y, luego, se exportarán los certificados de CA de la clave pública en formato codificado para obtener los certificado de CA de cliente de confianza. Luego, se concatenaran todos los certificados de CA de cliente en una cadena de certificados de CA de cliente de confianza. 

Los siguientes pasos le ayudan a exportar el archivo .pm o .cer para el certificado:

### <a name="export-public-certificate"></a>Exportación de un certificado público 

1. Para obtener un archivo .cer del certificado, abra **Administrar certificados de usuario**. Busque el certificado; normalmente se encuentra en "Certificados - Usuario actual\Personal\Certificados" y haga clic con el botón derecho. Haga clic en **Todas las tareas** y, luego, en **Exportar**. Se abre el **Asistente para exportar certificados**. Si no encuentra el certificado en Usuario actual\Personal\Certificados, puede que haya abierto de forma accidental "Certificados – equipo Local", en lugar de "Certificados - Usuario actual"). Si desea abrir el Administrador de certificados en el ámbito del usuario actual mediante PowerShell, escriba *certmgr* en la ventana de la consola.

    > [!div class="mx-imgBorder"]
    > ![Instantánea en la que aparece el Administrador de certificados con la opción Certificados seleccionada y con un menú contextual en el que están seleccionadas las opciones Todas las tareas y Exportar.](./media/certificates-for-backend-authentication/export.png)

2. En el asistente, haga clic en **Siguiente**.
    > [!div class="mx-imgBorder"]
    > ![Exportar un certificado](./media/certificates-for-backend-authentication/exportwizard.png)

3. Seleccione **No exportar la clave privada** y, después, haga clic en **Siguiente**.
    > [!div class="mx-imgBorder"]
    > ![No exportar la clave privada](./media/certificates-for-backend-authentication/notprivatekey.png)

4. En la página **Formato de archivo de exportación**, seleccione **X.509 codificado base 64 (.CER)** y, luego, haga clic en **Siguiente**.
    > [!div class="mx-imgBorder"]
    > ![Codificado con Base 64](./media/certificates-for-backend-authentication/base64.png)

5. En **Archivo que se va a exportar**, haga clic en **Examinar** para ir a la ubicación a la que desea exportar el certificado. En **Nombre de archivo**, asígnele un nombre al archivo de certificado. A continuación, haga clic en **Siguiente**.

    > [!div class="mx-imgBorder"]
   > ![Instantánea en la que aparece el Asistente para exportar certificados, donde puede especificar el archivo que va a exportar.](./media/certificates-for-backend-authentication/browse.png)

6. Haga clic en **Finalizar** para exportar el certificado.

    > [!div class="mx-imgBorder"]
    > ![Instantánea en la que aparece el Asistente para exportar certificados después de completar la exportación del archivo.](./media/certificates-for-backend-authentication/finish.png)

7. El certificado se exportó correctamente.

    > [!div class="mx-imgBorder"]
    > ![Instantánea en la que aparece el Asistente para exportar certificados con un mensaje que indica que la operación se ha realizado correctamente.](./media/certificates-for-backend-authentication/success.png)

   El certificado exportado tiene un aspecto similar al siguiente:

    > [!div class="mx-imgBorder"]
    > ![Instantánea en la que aparece un símbolo de certificado.](./media/certificates-for-backend-authentication/exported.png)

### <a name="export-ca-certificates-from-the-public-certificate"></a>Exportación de certificados de CA desde el certificado público

Una vez exportado el certificado público, exportará los certificados de CA desde el certificado público. Si solo tiene una CA raíz, únicamente tendrá que exportar ese certificado. Sin embargo, si tiene varias CA intermedias, deberá exportar todas ellas también. 

1. Cuando se haya exportado la clave pública, abra el archivo.

    > [!div class="mx-imgBorder"]
    > ![Abrir el certificado de autorización](./media/certificates-for-backend-authentication/openAuthcert.png)

    > [!div class="mx-imgBorder"]
    > ![acerca de certificado](./media/mutual-authentication-certificate-management/general.png)

1. Seleccione la pestaña Ruta de certificación para ver la entidad de certificación.

    > [!div class="mx-imgBorder"]
    > ![información del certificado](./media/mutual-authentication-certificate-management/cert-details.png) 

1. Seleccione el certificado raíz y haga clic en **Ver certificado**.

    > [!div class="mx-imgBorder"]
    > ![ruta de acceso del certificado](./media/mutual-authentication-certificate-management/root-cert.png) 

   Debería ver los detalles del certificado raíz.

    > [!div class="mx-imgBorder"]
    > ![información de certificado](./media/mutual-authentication-certificate-management/root-cert-details.png)

1. Seleccione la pestaña **Detalles** y haga clic en **Copiar en archivo...**

    > [!div class="mx-imgBorder"]
    > ![copia del certificado raíz](./media/mutual-authentication-certificate-management/root-cert-copy-to-file.png)

1. En este punto, ha extraído los detalles del certificado de CA del certificado de back-end. Verá el **Asistente para exportación de certificados**. Siga los pasos 2 a 7 de la sección anterior ([Exportación de un certificado público](./mutual-authentication-certificate-management.md#export-public-certificate)) para completar el Asistente para exportación de certificados. 

1. Ahora repita los pasos 2 a 6 de esta sección ([Exportación de certificados de CA desde el certificado público](./mutual-authentication-certificate-management.md#export-ca-certificates-from-the-public-certificate)) en todas las CA intermedias para exportar todos los certificados de CA intermedios en formato X.509 codificado con Base 64 (.CER).

    > [!div class="mx-imgBorder"]
    > ![certificado intermedio](./media/mutual-authentication-certificate-management/intermediate-cert.png)

    Por ejemplo, repita los pasos 2 a 6 de esta sección en la CA intermedia *MSIT CAZ2* para extraerlo como su propio certificado. 

### <a name="concatenate-all-your-ca-certificates-into-one-file"></a>Concatenación de todos los certificados de CA en un archivo

1. Ejecute el siguiente comando con todos los certificados CA que extrajo anteriormente. 

    Windows:
    ```console
    type intermediateCA.cer rootCA.cer > combined.cer
    ```
    
    Linux:
    ```console
    cat intermediateCA.cer rootCA.cer >> combined.cer
    ```

    El certificado combinado resultante debería ser similar al siguiente:
    
    > [!div class="mx-imgBorder"]
    > ![certificado combinado](./media/mutual-authentication-certificate-management/combined-cert.png)

## <a name="next-steps"></a>Pasos siguientes

Ya tiene la cadena de certificados de CA de cliente de confianza. Puede agregarla a la configuración de autenticación del cliente en Application Gateway para permitir la autenticación mutua con la puerta de enlace. Consulte la secciones sobre [configuración de la autenticación mutua mediante Application Gateway con Azure Portal](./mutual-authentication-portal.md) o [configuración de la autenticación mutua mediante Application Gateway con PowerShell](./mutual-authentication-powershell.md).

