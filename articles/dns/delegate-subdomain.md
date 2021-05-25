---
title: 'Delegación de un subdominio: Azure DNS'
description: Con esta ruta de aprendizaje, obtendrá información sobre cómo delegar un subdominio de Azure DNS.
services: dns
author: rohinkoul
ms.service: dns
ms.topic: how-to
ms.date: 05/03/2021
ms.author: rohink
ms.openlocfilehash: 11c9fd2e453db37a5aa985bcc53acdabf4a42aaf
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108763314"
---
# <a name="delegate-an-azure-dns-subdomain"></a>Delegación de un subdominio de Azure DNS

Puede usar Azure Portal para delegar un subdominio DNS. Por ejemplo, si es propietario del dominio contoso.com, puede delegar un subdominio llamado *ingeniería* a otra zona independiente que se puede administrar por separado respecto de la zona contoso.com.

Si lo prefiere, también puede delegar un subdominio usando [Azure PowerShell](delegate-subdomain-ps.md).

## <a name="prerequisites"></a>Requisitos previos

Para delegar un subdominio de Azure DNS, primero debe delegar el dominio público en Azure DNS. Consulte [Delegación de un dominio en Azure DNS](./dns-delegate-domain-azure-dns.md) para obtener instrucciones sobre cómo configurar los servidores de nombres para su delegación. Una vez que el dominio se delega a la zona de Azure DNS, puede delegar el subdominio.

> [!NOTE]
> Contoso.com se usa como ejemplo en todo este artículo. Sustituya su propio nombre de dominio por contoso.com.

## <a name="create-a-zone-for-your-subdomain"></a>Creación de una zona para el subdominio

En primer lugar, cree la zona para el subdominio **ingeniería**.

1. En Azure Portal, seleccione **+ Crear un recurso**.

1. Busque la **zona DNS** y, a continuación, seleccione **Crear**.

1. En la página **Crear zona DNS**, seleccione el grupo de recursos de la zona. Es posible que desee usar el mismo grupo de recursos como zona primaria para mantener juntos los recursos similares.

1.  Escriba `engineering.contoso.com` en **Nombre** y seleccione **Crear**.

1. Una vez que la implementación se haya realizado correctamente, vaya a la nueva zona.

## <a name="note-the-name-servers"></a>Observación de los servidores de nombres

A continuación, anote los cuatro servidores de nombres para el subdominio de ingeniería.

En la página de información general de la zona **ingeniería**, observe los cuatro servidores de nombres para la zona. Necesitará estos servidores de nombres más adelante.

## <a name="create-a-test-record"></a>Creación de un registro de prueba

Cree un registro **D** que se usará para las pruebas. Por ejemplo, cree un registro D **www** y configúrelo con una dirección IP **10.10.10.10**.

## <a name="create-an-ns-record"></a>Creación de un registro NS

A continuación, cree un registro de servidor de nombres para la zona **ingeniería**.

1. Vaya a la zona del dominio primario.

1. Seleccione **+ Conjunto de registros** en la parte superior de la página de información general.

1. En la página **Agregar conjunto de registros**, escriba **ingeniería** en el cuadro de texto **Nombre**.

1. En **Tipo**, seleccione **NS**.

1. En **Servidor de nombres**, escriba los cuatro servidores de nombres que anotó anteriormente en la zona **ingeniería**.

1. Seleccione **Aceptar** para guardar el registro.

## <a name="test-the-delegation"></a>Prueba de la delegación

Utilice nslookup para probar la delegación.

1. Abra una ventana de PowerShell.

1. En el símbolo del sistema, escriba `nslookup www.engineering.contoso.com.`.

1. Debería recibir una respuesta no autoritativa que muestra la dirección **10.10.10.10**.

## <a name="next-steps"></a>Pasos siguientes

Aprenda a [configurar búsquedas inversas de DNS para servicios hospedados en Azure](dns-reverse-dns-for-azure-services.md).
