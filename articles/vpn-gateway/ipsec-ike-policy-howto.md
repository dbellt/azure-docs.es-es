---
title: 'Directiva IPsec o IKE para conexiones de VPN de sitio a sitio o de red virtual a red virtual: Azure portal'
titleSuffix: Azure VPN Gateway
description: Obtenga información sobre cómo configurar una directiva IPsec o IKE para conexiones de sitio a sitio o de red virtual a red virtual con instancias de Azure VPN Gateway usando Azure Portal.
services: vpn-gateway
author: yushwang
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 04/28/2021
ms.author: yushwang
ms.openlocfilehash: 04dd5a1e8907d7314f9d96a81ae6965c943dc60c
ms.sourcegitcommit: 49bd8e68bd1aff789766c24b91f957f6b4bf5a9b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108229277"
---
# <a name="configure-ipsecike-policy-for-s2s-vpn-or-vnet-to-vnet-connections-azure-portal"></a>Configuración de una directiva de IPsec o IKE para conexiones VPN S2S o conexiones entre redes virtuales: Azure Portal

Este artículo le guiará por los pasos necesarios para configurar una directiva IPsec o IKE para conexiones VPN de sitio a sitio o de red virtual a red virtual con VPN Gateway mediante Azure Portal. En las siguientes secciones se le ayuda a crear y configurar una directiva IPsec o IKE, y aplicarla a una conexión nueva o existente.

## <a name="about-ipsec-and-ike-policy-parameters"></a><a name="about"></a>Acerca de los parámetros de la directiva IPsec e IKE

El protocolo IPsec e IKE estándar admite una gran variedad de algoritmos criptográficos en diversas combinaciones. Consulte [Acerca de los requisitos criptográficos y las puertas de enlace de VPN de Azure](vpn-gateway-about-compliance-crypto.md) para ver de qué forma esto puede ayudar a garantizar que la conectividad entre locales y de red virtual a red virtual satisface los requisitos de cumplimiento o de seguridad.

En este artículo se proporcionan instrucciones para crear y configurar una directiva IPsec o IKE, y aplicarla a una conexión de VPN Gateway nueva o existente.

### <a name="considerations"></a>Consideraciones

* La directiva IPsec/IKE solo funciona en las siguientes SKU de puerta de enlace:
  * ***VpnGw1~5 y VpnGw1AZ~5AZ***
  * ***Standard** _ y _ *_HighPerformance_**
* Solo se puede especificar ***una*** combinación de directivas para una conexión dada.
* Es preciso especificar todos los algoritmos y parámetros de IKE (modo principal) e IPsec (modo rápido). No se permite la especificación de una directiva parcial.
* Consulte las especificaciones del proveedor de dispositivos VPN para asegurarse de que los dispositivos VPN locales admiten la directiva. No se pueden establecer conexiones de sitio a sitio o de red virtual a red virtual si las directivas son incompatibles.

## <a name="workflow"></a><a name ="workflow"></a>Flujo de trabajo

En esta sección se describe el flujo de trabajo para crear y actualizar una directiva de IPsec o IKE en una conexión VPN de sitio a sitio o de red virtual a red virtual:

1. Cree una red virtual y una puerta de enlace de VPN.
2. Cree una puerta de enlace de red local para la conexión entre locales, u otra red virtual y puerta de enlace para la conexión de red virtual a red virtual.
3. Cree una conexión (IPsec o VNet2VNet).
4. Configure, actualice o elimine la directiva IPsec/IKE en los recursos de la conexión.

Las instrucciones que se dan en este artículo le ayudan a instalar y configurar directivas IPsec o IKE, como se muestra en el diagrama:

:::image type="content" source="./media/ipsec-ike-policy-howto/policy-diagram.png" alt-text="Diagrama de directiva IPsec o IKE" border="false":::

## <a name="supported-cryptographic-algorithms--key-strengths"></a><a name ="params"></a>Algoritmos criptográficos y niveles de seguridad de clave admitidos

### <a name="algorithms-and-keys"></a><a name ="table1"></a>Algoritmos y claves

En la tabla siguiente se enumeran los algoritmos criptográficos y los niveles de las claves admitidos que pueden configurar los clientes:

| **IPsec o IKE**    | **Opciones**    |
| ---              | ---            |
| Cifrado de IKE   | AES256, AES192, AES128, DES3, DES                  |
| Integridad de IKE    | SHA384, SHA256, SHA1, MD5                          |
| Grupo DH         | DHGroup24, ECP384, ECP256, DHGroup14, DHGroup2048, DHGroup2, DHGroup1, No |
| Cifrado IPsec | GCMAES256, GCMAES192, GCMAES128, AES256, AES192, AES128, DES3, DES, No    |
| Integridad de IPsec  | GCMASE256, GCMAES192, GCMAES128, SHA256, SHA1, MD5 |
| Grupo PFS        | PFS24, ECP384, ECP256, PFS2048, PFS2, PFS1, No   |
| Vigencia de SA QM   | (**Opcional**: Se usan los valores predeterminados si no se especifica ningún valor)<br>Segundos (entero; **mín. 300**/predeterminado 27000 segundos)<br>KBytes (entero; **mín. 1024**/predeterminado 102400000 KBytes)    |
| Selector de tráfico | UsePolicyBasedTrafficSelectors** ($True/$False; **Opcional**, valor predeterminado $False si no se especifica)    |
| Tiempo de expiración de DPD      | Segundos (entero; mín. 9/máx. 3600; predeterminado 45 segundos) |
|  |  |

#### <a name="important-requirements"></a>Requisitos importantes

* La configuración de su dispositivo VPN local debe coincidir o contener los siguientes algoritmos y parámetros que se especifican en la directiva de IPsec o IKE de Azure:
  * Algoritmo de cifrado de IKE (modo principal/fase 1)
  * Algoritmo de integridad de IKE (modo principal/fase 1)
  * Grupo DH (modo principal/fase 1)
  * Algoritmo de cifrado de IPsec (modo rápido/fase 2)
  * Algoritmo de integridad de IPsec (modo rápido/fase 2)
  * Grupo PFS (modo rápido/fase 2)>    * Selector de tráfico (si se usa UsePolicyBasedTrafficSelectors)
  * Las vigencias de SA solo son especificaciones locales, no es preciso que coincidan.

* Si GCMAES se usa para el algoritmo de cifrado IPsec, es preciso seleccionar el mismo algoritmo GCMAES y longitud de clave para la integridad de IPsec; por ejemplo, usar GCMAES128 para ambos.

* En la [tabla de algoritmos y claves](#table1) anterior:
  * IKE corresponde al modo principal o fase 1
  * IPsec corresponde al modo rápido o fase 2
  * El Grupo DH especifica el grupo Diffie-Hellmen utilizado en el modo principal o fase 1
  * El grupo PFS especifica el grupo Diffie-Hellmen utilizado en el modo rápido o fase 2

* La vigencia de SA del modo principal de IKE se fija en 28 800 en las puertas de enlace de VPN de Azure.

* Si establece **UsePolicyBasedTrafficSelectors** en $True en una conexión, configurará la puerta de enlace de VPN de Azure para que se conecte al firewall de VPN basado en directivas de forma local. Si habilita PolicyBasedTrafficSelectors, debe asegurarse de que el dispositivo VPN tiene los selectores de tráfico coincidentes definidos con todas las combinaciones de sus prefijos de red local (puerta de enlace de red local) a o desde los prefijos de red virtual de Azure, en lugar de cualquiera a cualquiera. Por ejemplo, si sus prefijos de red local son 10.1.0.0/16 y 10.2.0.0/16, y sus prefijos de red virtual son 192.168.0.0/16 y 172.16.0.0/16, debe especificar los siguientes selectores de tráfico:
  * 10.1.0.0/16 <====> 192.168.0.0/16
  * 10.1.0.0/16 <====> 172.16.0.0/16
  * 10.2.0.0/16 <====> 192.168.0.0/16
  * 10.2.0.0/16 <====> 172.16.0.0/16

   Para obtener más información sobre los selectores de tráfico basados en directivas, consulte [Connect multiple on-premises policy-based VPN devices](vpn-gateway-connect-multiple-policybased-rm-ps.md) (Conectar varios dispositivos VPN basados en directivas locales).

* Tiempo de expiración de DPD: el valor predeterminado es 45 segundos en las puertas de enlace de VPN de Azure. Si se establece el tiempo de expiración en un periodo menor, IKE volverá a especificar la clave de forma más agresiva, lo que provoca que en algunos momentos parezca que la conexión está desconectada, algo que no se desea si las ubicaciones locales están lejos de la región de Azure en la que reside la puerta de enlace de VPN o si el estado de vínculo físico puede llegar a provocar la pérdida de paquetes. La recomendación general es establecer el tiempo de expiración entre **30 y 45** segundos.

### <a name="diffie-hellman-groups"></a>Grupos Diffie-Hellman

En la tabla siguiente se muestran los grupos Diffie-Hellman admitidos en la directiva personalizada:

| **Grupo Diffie-Hellman**  | **Grupo DH**              | **Grupo PFS** | **Longitud de clave** |
| --- | --- | --- | --- |
| 1                         | DHGroup1                 | PFS1         | MODP de 768 bits   |
| 2                         | DHGroup2                 | PFS2         | MODP de 1024 bits  |
| 14                        | DHGroup14<br>DHGroup2048 | PFS2048      | MODP de 2048 bits  |
| 19                        | ECP256                   | ECP256       | ECP de 256 bits    |
| 20                        | ECP384                   | ECP384       | ECP de 384 bits    |
| 24                        | DHGroup24                | PFS24        | MODP de 2048 bits  |

Consulte [RFC3526](https://tools.ietf.org/html/rfc3526) y [RFC5114](https://tools.ietf.org/html/rfc5114) para ver información más detallada.

## <a name="s2s-vpn-with-ipsecike-policy"></a><a name ="S2S"></a>VPN de sitio a sitio con directiva IPsec o IKE

En esta sección se describen los pasos necesarios para crear una conexión VPN de sitio a sitio con una directiva IPsec o IKE. Los pasos siguientes permiten crear la conexión, como se muestra en el diagrama siguiente:

:::image type="content" source="./media/ipsec-ike-policy-howto/site-to-site-diagram.png" alt-text="Directiva de sitio a sitio" border="false":::

### <a name="step-1---create-the-virtual-network-vpn-gateway-and-local-network-gateway"></a><a name="createvnet1"></a>Paso 1: crear la red virtual, la puerta de enlace de VPN y la puerta de enlace de red local

Cree los siguientes recursos, como se muestra en las capturas de pantallas siguientes. Para ver los pasos necesarios, consulte [Creación de una conexión de sitio a sitio mediante Azure Portal](./tutorial-site-to-site-portal.md).

* **Red virtual:**  TestVNet1

   :::image type="content" source="./media/ipsec-ike-policy-howto/testvnet-1.png" alt-text="Red virtual":::

* **Puerta de enlace VPN:** VNet1GW

   :::image type="content" source="./media/ipsec-ike-policy-howto/vnet-1-gateway.png" alt-text="Puerta de enlace":::

* **Puerta de enlace de red local:** Site6

   :::image type="content" source="./media/ipsec-ike-policy-howto/lng-site-6.png" alt-text="Sitio":::

* **Connection:** VNet1 a Site6

    :::image type="content" source="./media/ipsec-ike-policy-howto/connection-site-6.png" alt-text="Connection":::

### <a name="step-2---configure-ipsecike-policy-on-the-s2s-vpn-connection"></a><a name="s2sconnection"></a>Paso 2: configurar la directiva IPsec o IKE en la conexión de VPN de sitio a sitio

En esta sección, configure una directiva de IPsec o IKE con los algoritmos y parámetros siguientes:

* IKE:   AES256, SHA384, DHGroup24, tiempo de expiración de DPD 45 segundos
* IPsec: AES256, SHA256, PFS No, vigencia de SA 30000 segundos y 102400000 KB

1. Vaya al recurso de conexión, **VNet1toSite6**, en Azure Portal. Seleccione la página **Configuración** y seleccione **Personalizada** en Directiva IPsec o IKE para mostrar todas las opciones de configuración. En la captura de pantalla siguiente se muestra la configuración de acuerdo con la lista:

    :::image type="content" source="./media/ipsec-ike-policy-howto/policy-site-6.png" alt-text="Site 6":::

1. Si usa GCMAES para IPsec, debe usar el mismo algoritmo GCMAES y longitud de clave para la integridad y el cifrado IPsec. Por ejemplo, en la siguiente captura de pantalla se especifica GCMAES128 tanto para el cifrado IPsec como para la integridad de IPsec:

   :::image type="content" source="./media/ipsec-ike-policy-howto/gcmaes.png" alt-text="GCMAES para IPsec":::

1. Opcionalmente, puede seleccionar **Habilitar** en la opción **Use policy based traffic selectors** (Usar selectores de tráfico basados en directivas) para que la puerta de enlace de VPN se pueda conectar a dispositivos VPN basados en directivas de forma local, como se ha descrito anteriormente.

   :::image type="content" source="./media/ipsec-ike-policy-howto/policy-based-selector.png" alt-text="Selector de tráfico basado en directiva":::

1. Una vez seleccionadas todas las opciones, seleccione **Guardar** para confirmar los cambios en el recurso de conexión. La directiva se aplicará en un minuto aproximadamente.

> [!IMPORTANT]
>
> * Una vez que se haya especificado una directiva de IPsec o IKE en una conexión, la puerta de enlace de VPN de Azure solo enviará o aceptará la propuesta de IPsec o IKE con los algoritmos criptográficos y los niveles de clave especificados en esa conexión en particular. Asegúrese de que el dispositivo VPN local para la conexión usa o acepta la combinación de directivas exacta. En caso contrario, no se establecerá el túnel VPN de sitio a sitio.
>
> * Las opciones **Selector de tráfico basado en directiva** y **Tiempo de expiración de DPD** se pueden especificar con la directiva **Predeterminada**, sin la directiva IPsec o IKE personalizada, como se muestra en la captura de pantalla anterior.
>

## <a name="vnet-to-vnet-with-ipsecike-policy"></a><a name ="vnet2vnet"></a>De red virtual a red virtual con la directiva IPsec o IKE

Los pasos necesarios para crear una conexión de red virtual a red virtual con una directiva de IPsec o IKE son similares a los pasos para crear una conexión VPN de sitio a sitio.

:::image type="content" source="./media/ipsec-ike-policy-howto/vnet-policy.png" alt-text="Diagrama de las directivas de red virtual a red virtual" border="false":::

1. Siga los pasos del artículo sobre la [creación de una conexión de red virtual a red virtual](vpn-gateway-vnet-vnet-rm-ps.md) para crear la suya.

2. Tras completar los pasos, verá dos conexiones de red virtual a red virtual, como se muestra en la captura de pantalla siguiente del recurso VNet2GW:

   :::image type="content" source="./media/ipsec-ike-policy-howto/vnet-vnet-connections.png" alt-text="Conexiones de red virtual a red virtual":::

3. Vaya al recurso de la conexión y, después, a la **Configuración** del portal. Seleccione **Personalizada** en **Directiva IPsec o IKE** para mostrar las opciones de directiva personalizadas. Seleccione los algoritmos criptográficos con las longitudes de clave correspondientes.

   La captura muestra una directiva de IPsec o IKE diferente con los algoritmos y parámetros siguientes:
   * IKE: AES128, SHA1, DHGroup14, tiempo de expiración de DPD 45 segundos
   * IPsec: GCMAES128, GCMAES128, PFS14, vigencia de SA de 14 400 segundos y 102 400 000 KB

   :::image type="content" source="./media/ipsec-ike-policy-howto/vnet-vnet-policy.png" alt-text="Directiva de conexión":::

4. Seleccione **Guardar** para aplicar los cambios de la directiva en el recurso de conexión.

5. Aplique la misma directiva al otro recurso de conexión, VNet2toVNet1. Si no lo hace, el túnel VPN de IPsec/IKE no se conectará porque no coinciden las directivas.

   > [!IMPORTANT]
   > Una vez que se haya especificado una directiva de IPsec o IKE en una conexión, la puerta de enlace de VPN de Azure solo enviará o aceptará la propuesta de IPsec o IKE con los algoritmos criptográficos y los niveles de clave especificados en esa conexión en particular. Asegúrese de que las directivas de IPsec para ambas conexiones son iguales. En caso contrario, no se establecerá la conexión de red virtual a red virtual.

6. Después de completar estos pasos, la conexión se establecerá al cabo de unos minutos y tendrá la topología de red siguiente:

    :::image type="content" source="./media/ipsec-ike-policy-howto/policy-diagram.png" alt-text="Diagrama de directiva IPsec o IKE" border="false":::

## <a name="to-remove-custom-ipsecike-policy-from-a-connection"></a><a name ="deletepolicy"></a>Eliminación de una directiva de IPsec o IKE de una conexión

1. Para quitar una directiva personalizada de una conexión, vaya hasta el recurso de conexión y, después, a la página **Configuración** para ver la directiva actual.

2. Seleccione **Predeterminada** en la opción **Directiva IPsec o IKE**. Se quitarán todas las directivas personalizadas especificadas previamente en la conexión y se restaurará la configuración predeterminada de IPsec/IKE en esta conexión:

   :::image type="content" source="./media/ipsec-ike-policy-howto/delete-policy.png" alt-text="Eliminar directiva":::

3. Seleccione **Guardar** para quitar la directiva personalizada y restaurar la configuración predeterminada de IPsec o IKE en la conexión.

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre los selectores de tráfico basados en directivas, vea [Connect multiple on-premises policy-based VPN devices](vpn-gateway-connect-multiple-policybased-rm-ps.md) (Conectar varios dispositivos VPN basados en directivas locales).