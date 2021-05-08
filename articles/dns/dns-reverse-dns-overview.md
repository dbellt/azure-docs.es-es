---
title: 'Introducción al DNS inverso en Azure: Azure DNS'
description: En esta ruta de aprendizaje, podrá aprender cómo funciona el DNS inverso y cómo se puede usar en Azure
services: dns
documentationcenter: na
author: rohinkoul
manager: KumuD
ms.service: dns
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/26/2021
ms.author: rohink
ms.openlocfilehash: 7f1cde9587615dbbecc450551a3b19fe02db85f5
ms.sourcegitcommit: 5f785599310d77a4edcf653d7d3d22466f7e05e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108065266"
---
# <a name="overview-of-reverse-dns-and-support-in-azure"></a>Introducción a DNS inverso y compatibilidad en Azure

En este artículo se proporciona información general sobre cómo funciona DNS inverso y escenarios en los que se admite DNS inverso en Azure.

## <a name="what-is-reverse-dns"></a>¿Qué es un DNS inverso?

Los registros DNS convencionales asignan un nombre DNS a una dirección IP; por ejemplo, `www.contoso.com` se resuelve en 64.4.6.100. Un DNS inverso hace lo contrario: traduce una dirección IP en un nombre. Por ejemplo, una búsqueda de 64.4.6.100 se resolverá en `www.contoso.com`.

Los registros de DNS inversos se utilizan en distintas situaciones. Por ejemplo, los registros de DNS inverso se usan mucho en la lucha contra el correo basura; para ello, se comprueba el remitente del mensaje de correo electrónico.  El servidor de correo receptor recupera el registro de DNS inverso de la dirección IP del servidor de envío. Luego, comprueba si ese host está autorizado para enviar un correo electrónico desde el dominio de origen.

## <a name="how-reverse-dns-works"></a>Cómo funciona un DNS inverso

Los registros de DNS inverso se hospedan en zonas DNS especiales conocidas como zonas “ARPA”.  Estas zonas forman una jerarquía de DNS independiente paralela a la jerarquía normal que hospeda dominios tales como `contoso.com`.

Por ejemplo, el registro de DNS `www.contoso.com` se implementa mediante un registro "D" de DNS con el nombre "www" en la zona `contoso.com`. Este registro A apunta a la dirección IP correspondiente, en este caso 64.4.6.100.  La búsqueda inversa se implementa por separado, usando un registro "PTR" llamado "100" en la zona "6.4.64.in-addr.arpa". Observe que las direcciones IP de las zonas ARPA se invierten. Este registro PTR, cuando esté configurado correctamente, apuntará al nombre `www.contoso.com`.

Cuando se asigna un bloque de direcciones IP a una organización, adquiere también el derecho a administrar la zona ARPA correspondiente. Microsoft hospeda y administra las zonas ARPA correspondientes a los bloques de direcciones IP usados por Azure. Su ISP puede hospedar la zona ARPA para sus propias direcciones IP automáticamente. También le puede permitir hospedar la zona ARPA en un servicio DNS de su elección, por ejemplo, Azure DNS.

> [!NOTE]
> Las búsquedas DNS directas e inversas se implementan en jerarquías de DNS independientes y paralelas. La búsqueda inversa de "www.contoso.com" **no** se hospeda en la zona "contoso.com"; en su lugar se hospeda en la zona ARPA para el bloque de direcciones IP correspondiente. Se utilizan zonas independientes para los bloques de direcciones IPv4 e IPv6.

### <a name="ipv4"></a>IPv4

El nombre de una zona de búsqueda inversa IPv4 debe tener el formato siguiente: `<IPv4 network prefix in reverse order>.in-addr.arpa`.

Por ejemplo, al crear una zona inversa para hospedar registros para hosts con direcciones IP que están en el prefijo 192.0.2.0/24, el nombre de zona se crearía aislando el prefijo de red de la dirección (192.0.2) e invirtiendo luego el orden (2.0.192) y agregando el sufijo `.in-addr.arpa`.

|Clase de subred|Prefijo de red  |Prefijo de red invertido  |Sufijo estándar  |Nombre de zona inversa |
|-------|----------------|------------|-----------------|---------------------------|
|Clase A|203.0.0.0/8     | 203        | .in-addr.arpa   | `203.in-addr.arpa`        |
|Clase B|198.51.0.0/16   | 51.198     | .in-addr.arpa   | `51.198.in-addr.arpa`     |
|Clase C|192.0.2.0/24    | 2.0.192    | .in-addr.arpa   | `2.0.192.in-addr.arpa`    |

### <a name="classless-ipv4-delegation"></a>Delegación IPv4 sin clases

En algunos casos, el intervalo IP ofrecido a una organización es menor que un intervalo de Clase C (/24). En este caso, el intervalo IP no entra dentro de un límite de zona en la jerarquía de zonas `.in-addr.arpa` y, por lo tanto, no se puede delegar como una zona secundaria.

Se usa un método diferente para transferir cada registro de búsqueda inversa a una zona DNS dedicada. Este método delega una zona secundaria para cada intervalo IP. Luego, asigna cada dirección IP del intervalo individualmente a esa zona secundaria mediante registros CNAME.

Por ejemplo, suponga que su ISP concede a su organización el intervalo IP 192.0.2.128/26. Este bloque de direcciones representa 64 direcciones IP, de 192.0.2.128 a 192.0.2.191. El DNS inverso de este intervalo se implementa del modo siguiente:
- Su organización crea una zona de búsqueda inversa denominada 128-26.2.0.192.in-addr.arpa. El prefijo "128-26" representa el segmento de red asignado a la organización dentro del intervalo de la Clase C (/24).
- El ISP crea registros NS a fin de configurar la delegación de DNS para la zona anterior desde la zona primaria de la Clase C. El ISP también crea registros CNAME en la zona de búsqueda inversa primaria (clase C). A continuación, asigna cada dirección IP del intervalo IP a la nueva zona creada por la organización:

    ```
    $ORIGIN 2.0.192.in-addr.arpa
    ; Delegate child zone
    128-26    NS       <name server 1 for 128-26.2.0.192.in-addr.arpa>
    128-26    NS       <name server 2 for 128-26.2.0.192.in-addr.arpa>
    ; CNAME records for each IP address
    129       CNAME    129.128-26.2.0.192.in-addr.arpa
    130       CNAME    130.128-26.2.0.192.in-addr.arpa
    131       CNAME    131.128-26.2.0.192.in-addr.arpa
    ; etc
    ```

- A continuación, la organización administra los registros PTR individuales dentro de su zona secundaria.

    ```
    $ORIGIN 128-26.2.0.192.in-addr.arpa
    ; PTR records for each UIP address. Names match CNAME targets in parent zone
    129      PTR    www.contoso.com
    130      PTR    mail.contoso.com
    131      PTR    partners.contoso.com
    ; etc
    ```

Una búsqueda inversa para la dirección IP '192.0.2.129' consulta para un registro PTR denominado '129.2.0.192.in-addr.arpa'. Esta consulta se resuelve a través del registro CNAME en la zona primaria hacia el registro PTR en la zona secundaria.

### <a name="ipv6"></a>IPv6

El nombre de una zona de búsqueda inversa IPv6 debe tener el formato siguiente: `<IPv6 network prefix in reverse order>.ip6.arpa`

Por ejemplo, cuando se crea una zona inversa para hospedar registros para hosts con las IP que están en el prefijo 2001:db8:1000:abdc::/64. El nombre de zona se crearía mediante el aislamiento del prefijo de red de la dirección (2001:db8::). A continuación, expanda el prefijo de red IPv6 para quitar la [compresión de ceros](/previous-versions/windows/it-pro/windows-server-2003/cc781672(v=ws.10)), si se usa para acortar el prefijo de dirección IPv6 (2001:0db8:abdc:0000::). Invierta el orden, con un punto como delimitador entre cada número hexadecimal en el prefijo, para generar el prefijo de red invertido (`0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2`) y agregar el sufijo `.ip6.arpa`.


|Prefijo de red  |Prefijo de red expandido e invertido |Sufijo estándar |Nombre de zona inversa  |
|---------|---------|---------|---------|
|2001:db8:abdc::/64    | 0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2        | .ip6.arpa        | `0.0.0.0.c.d.b.a.8.b.d.0.1.0.0.2.ip6.arpa`       |
|2001:db8:1000:9102::/64    | 2.0.1.9.0.0.0.1.8.b.d.0.1.0.0.2        | .ip6.arpa        | `2.0.1.9.0.0.0.1.8.b.d.0.1.0.0.2.ip6.arpa`        |


## <a name="azure-support-for-reverse-dns"></a>Soporte técnico de Azure para DNS inverso

Azure admite dos escenarios independientes con relación al DNS inverso:

**Hospedaje de la zona de búsqueda inversa correspondiente al bloque de direcciones IP**: Azure DNS se puede usar para [hospedar las zonas de búsqueda inversa y administrar los registros PTR](dns-reverse-dns-hosting.md) para IPv4 e IPv6. El proceso de creación de la zona (ARPA) de búsqueda inversa, configuración de la delegación y configuración de los registros PTR es el mismo que para las zonas DNS normales. Las diferencias son que la delegación se debe configurar mediante su ISP en lugar del registrador de DNS, y solo se debe usar el tipo de registro PTR.

**Configuración del registro DNS inverso para la dirección IP asignada al servicio de Azure**: Azure le permite [configurar la búsqueda inversa de las direcciones IP dadas al servicio de Azure](dns-reverse-dns-for-azure-services.md).  Azure configura esta búsqueda inversa como un registro PTR en la zona ARPA correspondiente.  Microsoft hospeda y administra estas zonas ARPA correspondientes a todos los intervalos IP usados por Azure.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener más información sobre el DNS inverso, consulte información sobre la [búsqueda de DNS inverso en Wikipedia](https://en.wikipedia.org/wiki/Reverse_DNS_lookup).
- Aprenda cómo [hospedar la zona de búsqueda inversa para el intervalo IP asignado por el ISP en DNS de Azure](dns-reverse-dns-for-azure-services.md).
- Aprenda a [administrar registros de DNS inversos para servicios de Azure](dns-reverse-dns-for-azure-services.md).
