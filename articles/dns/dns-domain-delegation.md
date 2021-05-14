---
title: Introducción a la delegación de Azure DNS
description: Información sobre cómo cambiar la delegación de dominios y usar los servidores de nombres DNS de Azure para ofrecer hospedaje de dominios.
services: dns
author: rohinkoul
ms.service: dns
ms.date: 04/19/2021
ms.author: rohink
ms.topic: conceptual
ms.openlocfilehash: 4753b07cc2f3ccd998c26a3392eb08c8761dd6f7
ms.sourcegitcommit: 425420fe14cf5265d3e7ff31d596be62542837fb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107738853"
---
# <a name="delegation-of-dns-zones-with-azure-dns"></a>Delegación de zonas DNS con Azure DNS

DNS de Azure le permite hospedar una zona DNS y administrar los registros DNS de un dominio en Azure. Para que las consultas DNS de un dominio lleguen a DNS de Azure, es necesario haber delegado dicho dominio en DNS de Azure desde el dominio primario. Tenga en cuenta que Azure DNS no es el registrador de dominios. En este artículo se explica cómo funciona la delegación de dominios y cómo delegar dominios en DNS de Azure.

## <a name="how-dns-delegation-works"></a>Funcionamiento de la delegación de DNS

### <a name="domains-and-zones"></a>Dominios y zonas

El sistema de nombres de dominio es una jerarquía de dominios. La jerarquía empieza por el dominio "raíz", cuyo nombre es sencillamente " **.** ".  Después de él, se encuentran los dominios de primer nivel, a saber, “com”, “net”, “org”, “uk” o “jp”.  A continuación, se colocan los dominios de segundo nivel, como “org.uk” o “co.jp”.  y así sucesivamente. Los dominios de la jerarquía de DNS se hospedan en zonas DNS independientes. Estas zonas están distribuidas globalmente, hospedadas por servidores de nombres DNS de todo el mundo.

**Zona DNS**: un dominio es un nombre exclusivo dentro del sistema de nombres de dominio, como por ejemplo “contoso.com”. Una zona DNS se usa para hospedar los registros DNS de un dominio concreto. Por ejemplo, puede que el dominio "contoso.com" contenga varios registros DNS, como "mail.contoso.com" (para un servidor de correo) y "www.contoso.com" (para un sitio web).

**Registrador de dominios**: un registrador de dominios es una empresa que puede ofrecer nombres de dominio de Internet. Verificará si el dominio de Internet deseado se encuentra disponible para que pueda adquirirlo. Tras haber registrado el nombre de dominio, será el titular legal de dicho nombre. Si ya dispone de un dominio de Internet, utilizará al registrador de dominios actual para delegar en Azure DNS.

Para más información sobre registradores de dominios acreditados, consulte [ICANN: Registradores acreditados](https://www.icann.org/registrar-reports/accredited-list.html).

### <a name="resolution-and-delegation"></a>Resolución y delegación

Existen dos tipos de servidores DNS:

* Un servidor DNS *autoritativo* hospeda zonas DNS. Solo responde a las consultas DNS de los registros de dichas zonas.
* Un servidor DNS *recursivo* no hospeda zonas DNS. Responde a todas las consultas DNS, para lo que realiza una llamada a los servidores DNS autoritativos a fin de recopilar los datos que necesita.

DNS de Azure proporciona un servicio DNS autoritativo.  No proporciona un servicio DNS recursivo. Las instancias de Azure Cloud Services y Virtual Machines se configuran automáticamente para que utilicen un servicio DNS recursivo que se proporciona por separado como parte de la infraestructura de Azure. Para más información sobre cómo cambiar esta configuración de DNS, consulte [Resolución de nombres de Azure](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server).

Los clientes DNS de equipos o dispositivos móviles suelen llamar a un servidor DNS recursivo para realizar todas las consultas de DNS que las aplicaciones cliente necesitan.

Cuando un servidor DNS recursivo recibe una consulta para un registro DNS, como “www.contoso.com”, primero necesita encontrar el servidor de nombres que hospeda la zona para el dominio “contoso.com”. Para encontrar el servidor de nombres, empieza por los servidores de nombres raíz y, a partir de ahí, busca los servidores de nombres que hospedan la zona “com”. A continuación, consulta los servidores de nombres “com” para encontrar los servidores de nombres que hospedan la zona “contoso.com”.  Por último, puede consultar estos servidores de nombres para "www.contoso.com".

A este procedimiento se le denomina resolución del nombre DNS. En el sentido estricto, la resolución DNS incluye más pasos, como seguir los CNAME, pero no es importante saber cómo funciona la delegación DNS.

¿Cómo “apunta” una zona primaria a los servidores de nombres de una zona secundaria? Se consigue usando un tipo especial de registro DNS, denominado registro NS (NS, por sus siglas en inglés, significa “servidor de nombres”). Por ejemplo, la zona raíz contiene registros NS para “com”, donde se muestran los servidores de nombres de la zona “com”. A su vez, la zona “com” contiene registros NS para “contoso.com”, donde se muestran los servidores de nombres para la zona “contoso.com”. A la acción que conlleva configurar los registros NS para una zona secundaria en una zona primaria se le denomina delegación de dominios.

La imagen siguiente muestra una consulta de DNS de ejemplo. contoso.net y partners.contoso.net son zonas DNS de Azure.

![Dns-nameserver](./media/dns-domain-delegation/image1.png)

1. El cliente solicita `www.partners.contoso.net` desde su servidor DNS local.
2. El servidor DNS local no tiene el registro, por lo que realiza una solicitud a su servidor de nombres raíz.
3. El servidor de nombres raíz no tiene el registro, pero conoce la dirección del servidor de nombres `.net` y proporciona esa dirección al servidor DNS.
4. El servidor DNS local envía la solicitud al servidor de nombres `.net`.
5. El servidor de nombres `.net` no tiene el registro pero no conoce la dirección del servidor de nombres `contoso.net`. En este caso, responde con la dirección del servidor de nombres de la zona DNS hospedada en Azure DNS.
6. El servidor DNS local envía la solicitud al servidor de nombres para la zona `contoso.net` hospedada en Azure DNS.
7. La zona `contoso.net` no tiene el registro, pero conoce el servidor de nombres de `partners.contoso.net` y responde con la dirección. En este caso es una zona DNS hospedada en Azure DNS.
8. El servidor DNS local envía la solicitud al servidor de nombres para la zona `partners.contoso.net`.
9. La zona `partners.contoso.net` contiene el registro A y responde con la dirección IP.
10. El servidor DNS local proporciona la dirección IP al cliente.
11. El cliente se conecta al sitio web `www.partners.contoso.net`.

Cada delegación realmente contiene dos copias de los registros NS, una en la zona primaria que apunta a la secundaria y otra en la misma zona secundaria. La zona “contoso.net” contiene los registros NS para “contoso.net” (además de los registros NS de “net”). Estos registros se denominan registros NS autoritativos y se sitúan en la cúspide de la zona secundaria.

## <a name="next-steps"></a>Pasos siguientes

Aprenda cómo [delegar el dominio a Azure DNS](dns-delegate-domain-azure-dns.md).
