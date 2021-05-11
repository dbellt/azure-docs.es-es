---
title: 'Conexión de un equipo a una red virtual mediante P2S: autenticación de certificado: Azure Portal (clásico)'
titleSuffix: Azure VPN Gateway
description: Obtenga información sobre cómo crear una conexión de VPN Gateway de punto a sitio clásica mediante Azure Portal.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 10/08/2020
ms.author: cherylmc
ms.openlocfilehash: 90f2c6dd8d82b40f6a12195c2ce89b28ca4e9305
ms.sourcegitcommit: 49bd8e68bd1aff789766c24b91f957f6b4bf5a9b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108228917"
---
# <a name="configure-a-point-to-site-connection-by-using-certificate-authentication-classic"></a>Configuración de una conexión de punto a sitio mediante la autenticación de certificado (clásica)

[!INCLUDE [deployment models](../../includes/vpn-gateway-classic-deployment-model-include.md)]

En este artículo se explica cómo crear una red virtual con una conexión de punto a sitio. Cree esta red virtual con el modelo de implementación clásica en Azure Portal. Esta configuración utiliza certificados para autenticar al cliente que se conecta, ya sean autofirmados o emitidos por una entidad de certificación. También puede crear esta configuración con un modelo o una herramienta de implementación diferente mediante las opciones que se describen en estos artículos:

> [!div class="op_single_selector"]
> * [Azure Portal](vpn-gateway-howto-point-to-site-resource-manager-portal.md)
> * [PowerShell](vpn-gateway-howto-point-to-site-rm-ps.md)
> * [Portal de Azure clásico](vpn-gateway-howto-point-to-site-classic-azure-portal.md)
>

Use una puerta de enlace de VPN de punto a sitio (P2S) para crear una conexión segura a la red virtual desde un equipo cliente individual. Las conexiones VPN de punto a sitio son útiles cuando desea conectarse a la red virtual desde una ubicación remota. Cuando hay solo unos pocos clientes que necesitan conectarse a una red virtual, una VPN P2S es una solución útil en lugar de una VPN de sitio a sitio. Se establece una conexión VPN P2S al iniciar la conexión desde el equipo cliente.

> [!IMPORTANT]
> El modelo de implementación clásica admite solo clientes de VPN de Windows y usa el Protocolo de túnel de sockets seguros (SSTP), un protocolo de VPN basado en SSL. Para admitir clientes de VPN que no sean de Windows, debe crear la red privada virtual con el modelo de implementación de Resource Manager. El modelo de implementación de Resource Manager admite IKEv2 VPN, además de SSTP. Para más información, consulte [Acerca de las conexiones P25](point-to-site-about.md).
>
>

![Diagrama de punto a sitio](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/point-to-site-connection-diagram.png)

## <a name="settings-and-requirements"></a>Configuración y requisitos

### <a name="requirements"></a>Requisitos

Las conexiones de autenticación de certificado de punto a sitio requieren los siguientes elementos. En este artículo se describen los pasos que le ayudarán a crearlos.

* Una puerta de enlace VPN dinámica.
* La clave pública (archivo .cer) de un certificado raíz, que se carga en Azure. Esta clave se considera un certificado de confianza y se usa para la autenticación.
* Se genera un certificado de cliente a partir del certificado raíz y se instala en cada equipo cliente que se vaya a conectar. Este certificado se usa para la autenticación de cliente.
* Se debe generar un paquete de configuración de cliente de VPN e instalarlo en todos los equipos cliente que se conectan. El paquete de configuración de cliente configura el cliente de VPN nativo que ya está en el sistema operativo con la información necesaria para conectarse a la red virtual.

Las conexiones de punto a sitio no requieren un dispositivo VPN ni una dirección IP de acceso público local. Se crea la conexión VPN sobre SSTP (Protocolo de túnel de sockets seguros). En el lado servidor, se admiten las versiones 1.0, 1.1 y 1.2 de SSTP. El cliente decide qué versión va a usar. Para Windows 8.1 y versiones posteriores, SSTP usa 1.2 de forma predeterminada.

Para más información, consulte [Acerca de las conexiones de punto a sitio](point-to-site-about.md) y las [Preguntas más frecuentes](#faq).

### <a name="example-settings"></a>Configuración de ejemplo

Use los siguientes valores para crear un entorno de prueba o hacer referencia a ellos para comprender mejor los ejemplos de este artículo:

* **Grupos de recursos:** TestRG
* **Nombre de la red virtual:** VNet1
* **Espacio de direcciones**: 192.168.0.0/16 <br>En este ejemplo, se utiliza solo un espacio de direcciones. Puede tener más de un espacio de direcciones para la red virtual.
* **Nombre de subred:** FrontEnd
* **Intervalo de direcciones de subred:** 192.168.1.0/24
* **GatewaySubnet:** 10.11.255.0/27
* **Región:** (EE. UU.) Este de EE. UU.
* **Espacio de direcciones de cliente:** 172.16.201.0/24 <br> Los clientes de VPN que se conectan a la red virtual mediante esta conexión de punto a sitio reciben una dirección IP del grupo especificado.
* **Tipo de conexión**: seleccione **Punto a sitio**.
* **Intervalo de direcciones de GatewaySubnet (bloque CIDR):** 192.168.200.0/24

Antes de empezar, compruebe que tiene una suscripción a Azure. Si todavía no la tiene, puede activar sus [ventajas como suscriptor de MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details) o registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial).

## <a name="create-a-virtual-network"></a><a name="vnet"></a>Creación de una red virtual

Si ya dispone de una red virtual, compruebe que la configuración sea compatible con el diseño de la puerta de enlace de VPN. Preste especial atención a las subredes que se pueden superponer con otras redes.

[!INCLUDE [basic classic vnet](../../includes/vpn-gateway-vnet-classic.md)]

[!INCLUDE [basic classic DNS](../../includes/vpn-gateway-dns-classic.md)]

## <a name="create-a-vpn-gateway"></a><a name="gateway"></a>Creación de una puerta de enlace de VPN

1. Navegue hasta la red virtual que ha creado.
1. En la página VNet, en Configuración, seleccione **Puerta de enlace**. En la página **Puerta de enlace**, puede ver la puerta de enlace de la red virtual. Esta red virtual todavía no tiene una puerta de enlace. Haga clic en la nota que indica **Haga clic aquí para agregar una conexión y una puerta de enlace**.
1. En la página **Configurar una conexión VPN y una puerta de enlace**, seleccione la configuración siguiente:

   * Tipo de conexión: de punto a sitio
   * Espacio de direcciones de cliente: agregue el intervalo de direcciones IP del que los clientes de VPN reciben una dirección IP al conectarse. Use un intervalo de direcciones IP privadas que no se superponga a la ubicación local desde la que se va a conectar ni a la red virtual a la que va a conectarse.
1. Deje la casilla **No configurar una puerta de enlace en este momento** sin seleccionar. Crearemos una puerta de enlace.
1. En la parte inferior de la página, seleccione **Next: Puerta de enlace >** .
1. En la pestaña **Puerta de enlace**, seleccione los siguientes valores:

   * **Size:** El tamaño es la SKU de la puerta de enlace para la puerta de enlace de red virtual. En Azure Portal, la SKU predeterminada es **Predeterminada**. Para más información acerca de las SKU de puerta de enlace, vea [Acerca de la configuración de VPN Gateway](vpn-gateway-about-vpn-gateway-settings.md#gwsku).
   * **Tipo de enrutamiento:** Debe seleccionar **Dinámico** para una configuración de punto a sitio. El enrutamiento estático no funcionará.
   * **Subred de puerta de enlace:** Este campo ya se ha autorrellenado. No puede cambiar el nombre. Si intenta cambiar el nombre mediante PowerShell o cualquier otro medio, la puerta de enlace no funcionará correctamente.
   * **Intervalo de direcciones (bloque CIDR):** Aunque es posible crear una subred de puerta de enlace tan pequeña como /29, se recomienda que cree una subred mayor que incluya más direcciones seleccionando al menos /28 o /27. Esto permitirá suficientes direcciones para dar cabida a posibles configuraciones adicionales que desee en el futuro. Cuando trabaje con subredes de la puerta de enlace, evite asociar un grupo de seguridad de red (NSG) a la subred de la puerta de enlace. La asociación de un grupo de seguridad de red a esta subred puede causar que la puerta de enlace de VPN no funcione según lo esperado.
1. Seleccione **Revisar y crear** para validar la configuración.
1. Una vez pasada la validación, seleccione **Crear**. Una puerta de enlace de VPN puede tardar hasta 45 minutos en completarse, según la SKU de puerta de enlace que seleccione.

## <a name="create-certificates"></a><a name="generatecerts"></a>Creación de certificados

Azure usa certificados para autenticar a los clientes VPN en VPN de punto a sitio. Cargue la información de clave pública del certificado raíz en Azure. La clave pública se considerará *de confianza*. Deben generarse certificados de cliente a partir del certificado raíz de confianza e instalarse en cada equipo cliente en el almacén de certificados Certificados-Usuario actual/Personal/Certificados. El certificado se utiliza para autenticar al cliente cuando se conecta a la red virtual. 

Si usa certificados autofirmados, se deben crear con parámetros específicos. Puede crear un certificado autofirmado con las instrucciones para [PowerShell y Windows 10](vpn-gateway-certificates-point-to-site.md), o bien [MakeCert](vpn-gateway-certificates-point-to-site-makecert.md). Es importante que siga los pasos descritos en estas instrucciones cuando use certificados raíz autofirmados y genere certificados de cliente a partir del certificado raíz autofirmado. En caso contrario, los certificados que cree no serán compatibles con las conexiones P2S y recibirá un error de conexión.

### <a name="acquire-the-public-key-cer-for-the-root-certificate"></a>Obtención de la clave pública (.cer) para el certificado raíz

[!INCLUDE [vpn-gateway-basic-vnet-rm-portal](../../includes/vpn-gateway-p2s-rootcert-include.md)]

### <a name="generate-a-client-certificate"></a>Generación de un certificado de cliente

[!INCLUDE [vpn-gateway-basic-vnet-rm-portal](../../includes/vpn-gateway-p2s-clientcert-include.md)]

## <a name="upload-the-root-certificate-cer-file"></a>Carga del archivo .cer de certificado raíz

Una vez creada la puerta de enlace, cargue el archivo .cer (que contiene la información de clave pública) para un certificado raíz de confianza en el servidor de Azure. No cargue la clave privada para el certificado raíz. Una vez cargado el certificado, Azure lo usa para autenticar a los clientes que tienen instalado un certificado de cliente generado a partir del certificado raíz de confianza. Más adelante, puede cargar más archivos de certificado raíz de confianza, hasta un total de 20, si es necesario.

1. Vaya a la red virtual que creó.
1. En **Configuración**, seleccione **Conexiones de punto a sitio**.
1. Seleccione **Administrar certificado**.
1. Seleccione **Cargar**.
1. En el panel **Cargar un certificado**, seleccione el icono de la carpeta y navegue hasta el certificado que quiere cargar.
1. Seleccione **Cargar**.
1. Una vez que el certificado se haya cargado correctamente, puede verlo en la página Administrar certificado. Es posible que tenga que seleccionar **Actualizar** para ver el certificado que acaba de cargar.

## <a name="configure-the-client"></a>Configurar el cliente

Para conectarse a una red virtual mediante una VPN de punto a sitio, cada cliente debe instalar un paquete para configurar el cliente de VPN de Windows nativo. El paquete de configuración configura el cliente de VPN nativo de Windows con los parámetros necesarios para conectarse a la red virtual.

Puede utilizar el mismo paquete de configuración de cliente VPN en todos los equipos cliente, siempre que la versión coincida con la arquitectura del cliente. Para obtener la lista de sistemas operativos cliente compatibles, consulte el documento [Acerca de las conexiones de punto a sitio](point-to-site-about.md) y las [Preguntas más frecuentes](#faq).

### <a name="generate-and-install-a-vpn-client-configuration-package"></a>Generación e instalación de un paquete de configuración de cliente de VPN

1. Vaya a la configuración **Conexiones de punto a sitio** de la red virtual.
1. En la parte superior de la página, seleccione el paquete de descarga que se corresponde con el sistema operativo cliente en el que se va a instalar:

   * Para los clientes de 64 bits, seleccione **Cliente de VPN (64 bits)** .
   * Para los clientes de 32 bits, seleccione **Cliente de VPN (32 bits)** .

1. Azure genera un paquete con la configuración específica que requiere el cliente. Cada vez que realice cambios en la red virtual o la puerta de enlace, deberá descargar un nuevo paquete de configuración de cliente e instalarlo en los equipos cliente.
1. Una vez generado el paquete, seleccione **Descargar**.
1. Instale el paquete de configuración del cliente en el equipo cliente. Al realizar la instalación, si ve una ventana emergente de SmartScreen que indica que Windows protegió su equipo, seleccione **Más información** y, después, seleccione **Ejecutar de todas formas**. También puede guardar el paquete para instalarlo en otros equipos cliente.

### <a name="install-a-client-certificate"></a>Instalación de un certificado de cliente

Para este ejercicio, al generar el certificado de cliente, se instaló automáticamente en el equipo. Para crear una conexión P2S desde un equipo cliente distinto del que usó para generar los certificados de cliente, debe instalar un certificado de cliente en ese equipo.

Al instalar un certificado de cliente, necesita la contraseña que se creó cuando se exportó el certificado de cliente. Normalmente, puede instalar el certificado con tan solo hacer doble clic en él. Para más información, consulte [Instalación de un certificado de cliente exportado](vpn-gateway-certificates-point-to-site.md#install).

## <a name="connect-to-your-vnet"></a>Conexión a la red virtual

>[!NOTE]
>Debe tener derechos de administrador en el equipo cliente desde el que se va a conectar.
>

1. En el equipo cliente, vaya a la configuración de VPN.
1. Seleccione la VPN que ha creado. Si usó la configuración de ejemplo, la conexión estará etiquetada como **Grupo TestRG VNet1**.
1. Seleccione **Conectar**.
1. En el cuadro de Windows Azure Virtual Network, seleccione **Conectar**. Si aparece un mensaje emergente sobre el certificado, seleccione **Continuar** para usar privilegios elevados y **Sí** para aceptar los cambios en la configuración.
1. Si la conexión se realiza correctamente, verá una notificación **Conectado**.

[!INCLUDE [verify-client-certificates](../../includes/vpn-gateway-certificates-verify-client-cert-include.md)]

## <a name="verify-the-vpn-connection"></a>Comprobación de la conexión VPN

1. Verifique que la conexión VPN está activa. Abra un símbolo del sistema con privilegios elevados en el equipo cliente y ejecute **ipconfig/all**.
1. Vea los resultados. Observe que la dirección IP recibida es una de las direcciones en el intervalo de direcciones de conectividad de punto a sitio que especificó cuando creó la red virtual. Los resultados deben ser parecidos a los del ejemplo siguiente:

   ```
    PPP adapter VNet1:
        Connection-specific DNS Suffix .:
        Description.....................: VNet1
        Physical Address................:
        DHCP Enabled....................: No
        Autoconfiguration Enabled.......: Yes
        IPv4 Address....................: 192.168.130.2(Preferred)
        Subnet Mask.....................: 255.255.255.255
        Default Gateway.................:
        NetBIOS over Tcpip..............: Enabled
   ```

## <a name="to-connect-to-a-virtual-machine"></a>Para conectarse a una máquina virtual

[!INCLUDE [Connect to a VM](../../includes/vpn-gateway-connect-vm-p2s-classic-include.md)]

## <a name="to-add-or-remove-trusted-root-certificates"></a>Para agregar o quitar certificados raíz de confianza

Puede agregar y quitar certificados raíz de confianza de Azure. Al quitar un certificado raíz, los clientes que tienen un certificado generado a partir de dicha raíz ya no podrán autenticarse ni tampoco conectarse. Para que esos clientes puedan volver a autenticarse y conectarse, debe instalar un nuevo certificado de cliente generado a partir de un certificado raíz que sea de confianza en Azure.

### <a name="add-a-trusted-root-certificate"></a>Adición de un certificado raíz de confianza

Puede agregar hasta 20 archivos .cer de certificado raíz de confianza en Azure mediante el mismo proceso que usó para agregar el primer certificado raíz de confianza.

### <a name="remove-a-trusted-root-certificate"></a>Eliminación de un certificado raíz de confianza

1. En la sección **Conexiones de punto a sitio** de la página de la red virtual, seleccione **Administrar certificado**.
1. Seleccione los puntos suspensivos junto al certificado que quiere quitar y, después, seleccione **Eliminar**.

## <a name="to-revoke-a-client-certificate"></a>Para revocar un certificado de cliente

Si es necesario, puede revocar un certificado de cliente. La lista de revocación de certificados permite denegar de forma selectiva la conectividad de punto a sitio basada en certificados de cliente individuales. Este método difiere de la forma en que se quita un certificado raíz de confianza. Si quita de Azure un archivo .cer de certificado raíz de confianza, se revoca el acceso para todos los certificados de cliente generados y firmados con el certificado raíz revocado. Al revocarse un certificado de cliente, en lugar del certificado raíz, se permite que el resto de los certificados que se generaron con el certificado raíz sigan usándose para la autenticación de la conexión de punto a sitio.

Lo más habitual es usar el certificado raíz para administrar el acceso a nivel de equipo u organización, mientras que los certificados de cliente revocados se usan para un control de acceso específico para usuarios individuales.

Puede revocar un certificado de cliente si agrega la huella digital a la lista de revocación.

1. Recupere la huella digital del certificado de cliente. Para más información, consulte [Cómo recuperar la huella digital de un certificado](/dotnet/framework/wcf/feature-details/how-to-retrieve-the-thumbprint-of-a-certificate).
1. Copie la información en un editor de texto y quite sus espacios de forma que sea una sola cadena continua.
1. Vaya a **Conexión VPN de punto a sitio** y seleccione **Administrar certificado**.
1. Seleccione **Lista de revocación** para abrir la página **Lista de revocación**.
1. En **Huella digital**, pegue la huella digital del certificado como una línea continua de texto, sin espacios.
1. Seleccione **+ Agregar a la lista** para agregar la huella digital a la lista de revocación de certificados (CRL).

Una vez finalizada la actualización, el certificado no se puede usar para conectarse. Los clientes que intenten conectarse con este certificado recibirán un mensaje que indica que el certificado ya no es válido.

## <a name="faq"></a><a name="faq"></a>P+F

[!INCLUDE [Point-to-Site FAQ](../../includes/vpn-gateway-faq-point-to-site-classic-include.md)]

## <a name="next-steps"></a>Pasos siguientes

* Una vez completada la conexión, puede agregar máquinas virtuales a las redes virtuales. Consulte [Virtual Machines](../index.yml) para más información.

* Para más información sobre las redes y las máquinas virtuales Linux, consulte [Información general sobre las redes de máquina virtual con Linux y Azure](../virtual-machines/network-overview.md).

* Para información de solución de problemas de P2S, consulte el artículo de [solución de problemas de conexión de punto a sitio de Azure](vpn-gateway-troubleshoot-vpn-point-to-site-connection-problems.md).