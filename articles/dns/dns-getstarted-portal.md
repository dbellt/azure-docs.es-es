---
title: 'Inicio rápido: Creación de un registro y una zona DNS: Azure Portal'
titleSuffix: Azure DNS
description: Utilice esta guía de inicio rápido paso a paso para aprender a crear una zona Azure DNS y registrarla con Azure Portal.
services: dns
author: rohinkoul
ms.author: rohink
ms.date: 04/23/2021
ms.topic: quickstart
ms.service: dns
ms.custom:
- mode-portal
ms.openlocfilehash: f72dd2be2277754100693c2ebb1ef8658ca6f39a
ms.sourcegitcommit: ad921e1cde8fb973f39c31d0b3f7f3c77495600f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107949421"
---
# <a name="quickstart-create-an-azure-dns-zone-and-record-using-the-azure-portal"></a>Inicio rápido: Creación de una zona y un registro Azure DNS mediante Azure Portal

Puede configurar Azure DNS para resolver nombres de host en el dominio público. Por ejemplo, si ha adquirido el nombre de dominio *contoso.xyz* de un registrador de nombres de dominio, puede configurar Azure DNS para hospedar el dominio *contoso.xyz* y resolver *`www.contoso.xyz`* en la dirección IP del servidor web o la aplicación web.

En esta guía de inicio rápido, creará un dominio de prueba y, después, un registro de dirección llamado *www* para resolverlo en la dirección IP *10.10.10.10*.

:::image type="content" source="media/dns-getstarted-portal/environment-diagram.png" alt-text="Diagrama del entorno de implementación de DNS mediante el uso de Azure Portal." border="false":::

>[!IMPORTANT]
>Todos los nombres y direcciones IP en esta guía de inicio rápido son ejemplos que no representan escenarios del mundo real.

<!---
You can also perform these steps using [Azure PowerShell](dns-getstarted-powershell.md) or the cross-platform [Azure CLI](dns-getstarted-cli.md).
--->

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

Para todos los pasos de portal, inicie sesión en [Azure Portal](https://portal.azure.com).

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en [Azure Portal](https://portal.azure.com) con su cuenta de Azure.

## <a name="create-a-dns-zone"></a>Creación de una zona DNS

Una zona DNS contiene las entradas de DNS para un dominio. Para iniciar el hospedaje de su dominio en Azure DNS, debe crear una zona DNS para ese nombre de dominio. 

**Para crear la zona DNS:**

1. En la parte superior izquierda, seleccione **Crear un recurso**, después **Red** y finalmente **Zona DNS**.

1. En la página **Crear zona DNS**, escriba o seleccione los siguientes valores:

   - **Name**: escriba *contoso.xyz* en este ejemplo de inicio rápido. El nombre de la zona DNS puede ser cualquier valor que no esté ya configurado en los servidores de Azure DNS. Un valor del mundo real sería un dominio que haya comprado a un registrador de nombres de dominio.
   - **Grupo de recursos**: Seleccione **Crear nuevo**, escriba *MyResourceGroup* y seleccione **Aceptar**. El nombre del grupo de recursos debe ser único dentro de la suscripción de Azure. 

1. Seleccione **Crear**.

   ![Zona DNS](./media/dns-getstarted-portal/openzone650.png)

La creación de la zona puede tardar unos minutos.

## <a name="create-a-dns-record"></a>Creación de un registro de DNS

Cree entradas o registros DNS para su dominio dentro de la zona DNS. Cree un nuevo registro de dirección o un registro "A" para resolver un nombre de host a una dirección IPv4.

**Para crear un registro "A":**

1. En Azure Portal, en **Todos los recursos**, abra la zona DNS **contoso.xyz** en el grupo de recursos **MyResourceGroup**. Puede escribir *contoso.xyz* en el cuadro **Filtrar por nombre** para encontrarlo más fácilmente.

1. En la parte superior de la página **Zona DNS**, seleccione **+ Conjunto de registros**.

1. En la página **Agregar conjunto de registros**, escriba o seleccione los siguientes valores:

   - **Name**: escriba *www*. El nombre de registro es el nombre de host que desea resolver en la dirección IP especificada.
   - **Tipo**: seleccione **A**. Los registros "A" son las más comunes, pero hay otros tipos de registros para los servidores de correo (MX), las direcciones IP v6 (AAAA), etc. 
   - **TTL**: escriba *1*. El *período de vida* de la solicitud DNS especifica cuánto tiempo pueden los clientes y servidores DNS almacenar en caché una respuesta.
   - **Unidad de TTL**: seleccione **Horas**. Esta es la unidad de tiempo para el valor **TTL**. 
   - **Dirección IP**: para este ejemplo de inicio rápido, escriba *10.10.10.10*. Este valor es la dirección IP en la que se resuelve el nombre de registro. En un escenario del mundo real, tendría que escribir la dirección IP pública del servidor web.

Como este inicio rápido tiene solamente fines de prueba rápida, no es necesario configurar los servidores de nombres de Azure DNS en un registrador de nombres de dominio. Con un dominio real de producción, normalmente querrá que todos los usuarios de Internet pudieran resolver el nombre de host para conectarse al servidor o aplicación web. Visitará su registrador de nombres de dominio para reemplazar los registros del servidor de nombres con los servidores de nombres de Azure DNS. Para más información, consulte el [Tutorial: Hospedaje del dominio en Azure DNS](dns-delegate-domain-azure-dns.md#delegate-the-domain).

## <a name="test-the-name-resolution"></a>Probar la resolución de nombres

Ahora que tiene una zona DNS de prueba, con un registro "A" de prueba, puede probar la resolución de nombres con una herramienta llamada *nslookup*. 

**Para probar la resolución de nombres DNS:**

1. En Azure Portal, en **Todos los recursos**, abra la zona DNS **contoso.xyz** en el grupo de recursos **MyResourceGroup**. Puede escribir *contoso.xyz* en el cuadro **Filtrar por nombre** para encontrarlo más fácilmente.

1. Copie uno de los nombres de los servidores de nombres de la lista de servidores de nombres en la página **Introducción**. 

   ![zona](./media/dns-getstarted-portal/viewzonens500.png)

1. Abra un símbolo del sistema y ejecute el comando siguiente:

   ```
   nslookup www.contoso.xyz <name server name>
   ```

   Por ejemplo:

   ```
   nslookup www.contoso.xyz ns1-08.azure-dns.com.
   ```

   Debe ver algo parecido a la pantalla siguiente:

   ![La captura de pantalla muestra una ventana del símbolo del sistema con un comando de búsqueda ns y los valores de Servidor, Dirección, Nombre y Dirección.](media/dns-getstarted-portal/nslookup.PNG)

El nombre de host **www\.contoso.xyz** se resuelve en **10.10.10.10**, que es como lo ha configurado. Este resultado confirma que la resolución de nombres funciona correctamente. 

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite los recursos que creó en este inicio rápido, quítelos mediante la eliminación del grupo de recursos **MyResourceGroup**. Abra el grupo de recursos **MyResourceGroup** y seleccione **Eliminar grupo de recursos**.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de registros DNS para una aplicación web en un dominio personalizado](./dns-web-sites-custom-domain.md)
