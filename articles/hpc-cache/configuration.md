---
title: Configuración de Azure HPC Cache
description: Explica cómo configurar opciones adicionales para la caché, como MTU, NTP personalizado y los valores de DNS, y cómo acceder a las instantáneas rápidas desde destinos de Azure Blob Storage.
author: ekpgh
ms.service: hpc-cache
ms.topic: how-to
ms.date: 04/08/2021
ms.author: v-erkel
ms.openlocfilehash: 0b3996df3c75ff31d0825be1d332dbd055305963
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107259768"
---
# <a name="configure-additional-azure-hpc-cache-settings"></a>Configuración de valores adicionales de Azure HPC Cache

La página **Redes** de Azure Portal tiene opciones para personalizar varias configuraciones. La mayoría de los usuarios no tienen que cambiar estos valores predeterminados.

Además, en este artículo se describe cómo usar la característica de instantánea para destinos de almacenamiento de blobs de Azure. La característica de instantánea no tiene valores configurables.

Para ver la configuración, abra la página **Redes** de la caché en Azure Portal.

![Captura de pantalla de la página Redes de Azure Portal](media/networking-page.png)

> [!NOTE]
> Una versión anterior de esta página incluía un valor de configuración de squash raíz para el nivel de caché, pero esta configuración se pasó a [directivas de acceso de cliente](access-policies.md).

<!-- >> [!TIP]
> The [Managing Azure HPC Cache video](https://azure.microsoft.com/resources/videos/managing-hpc-cache/) shows the networking page and its settings. -->

## <a name="adjust-mtu-value"></a>Ajuste del valor de MTU
<!-- linked from troubleshoot-nas article -->

Puede seleccionar el tamaño máximo de la unidad de transmisión para la memoria caché mediante el menú desplegable con la etiqueta **Tamaño de MTU**.

El valor predeterminado es 1500 bytes, pero puede cambiarlo a 1400.

> [!NOTE]
> Si reduce el tamaño de MTU de la memoria caché, asegúrese de que los clientes y los sistemas de almacenamiento que se comunican con la memoria caché tengan la misma configuración de MTU o un valor inferior.

Reducir el valor de MTU de la caché puede ayudarle a solucionar las restricciones de tamaño de paquete en el resto de la red de la memoria caché. Por ejemplo, algunas VPN no pueden transmitir correctamente paquetes de tamaño completo de 1500 bytes. Reducir el tamaño de los paquetes enviados a través de la VPN puede eliminar ese problema. Sin embargo, tenga en cuenta que una configuración inferior de MTU de la memoria caché significa que cualquier otro componente que se comunique con la memoria caché (incluidos los clientes y sistemas de almacenamiento) también debe tener una configuración inferior para evitar problemas de comunicación.

Si no desea cambiar la configuración de MTU en otros componentes del sistema, no debe reducir la configuración de MTU de la memoria caché. Existen otras soluciones alternativas para las restricciones de tamaño de paquete de VPN. Para obtener más información sobre cómo diagnosticar y solucionar este problema, consulte [Ajuste de restricciones de tamaño de paquete de VPN](troubleshoot-nas.md#adjust-vpn-packet-size-restrictions) en el artículo solución de problemas de NAS.

Para obtener más información sobre la configuración de MTU en redes virtuales de Azure, lea [Optimización del rendimiento de TCP/IP para máquinas virtuales de Azure](../virtual-network/virtual-network-tcpip-performance-tuning.md).

## <a name="customize-ntp"></a>Personalización de NTP

De manera predeterminada, la caché usa el servidor horario basado en Azure time.windows.com. Si quiere que la caché use un servidor NTP diferente, especifíquelo en la sección **Configuración de NTP**. Use un nombre de dominio completo o una dirección IP.

## <a name="set-a-custom-dns-configuration"></a>Configuración de un valor de DNS personalizado

> [!CAUTION]
> No cambie la configuración de DNS de la caché si no es necesario. Los errores de configuración pueden tener graves consecuencias. Si la configuración no puede resolver los nombres de servicio de Azure, la instancia de HPC Cache se volverá inaccesible de manera permanente.
>
> Consulte con los representantes de Azure antes de intentar configurar un valor de DNS personalizado.

La Azure HPC Cache se configura automáticamente para usar el sistema de Azure DNS, que es seguro y práctico. Sin embargo, algunas configuraciones inusuales requieren que la caché use un sistema DNS local e independiente, en lugar del sistema de Azure. La sección **Configuración de DNS** de la página **Redes** se usa para especificar este tipo de sistema.

Póngase en contacto con sus representantes de Azure o con el servicio y soporte técnico de Microsoft para determinar si necesita usar una configuración DNS de caché personalizada.

Si configura su propio sistema DNS local para uso de Azure HPC Cache, debe asegurarse de que el servidor DNS local pueda resolver directamente los nombres de los puntos de conexión de servicio de Azure. HPC Cache funcionará si el servidor DNS está restringido a la resolución de nombres públicos.

Compruebe que la configuración de DNS puede resolver correctamente estos elementos antes de usarla para una instancia de Azure HPC Cache:

* ``*.core.windows.net``
* Descarga de la lista de revocación de certificados (CRL) y servicios de comprobación del protocolo de estado de certificados en línea (OCSP). Se especifica una lista parcial en el [elemento Reglas de firewall](../security/fundamentals/tls-certificate-changes.md#will-this-change-affect-me) al final de este [artículo de Azure TLS](../security/fundamentals/tls-certificate-changes.md), pero debe consultar a un representante técnico de Microsoft para conocer todos los requisitos.
* El nombre de dominio completo del servidor NTP (time.windows.com o un servidor personalizado).

Si necesita establecer un servidor DNS personalizado para la caché, use los campos proporcionados:

* **Dominio de búsqueda de DNS** (opcional): escriba su dominio de búsqueda; por ejemplo, ``contoso.com``. Se permite un solo valor, o puede dejarlo en blanco.
* **Servidores DNS**: escriba hasta tres servidores DNS. Indíquelos por dirección IP.

<!-- 
  > [!NOTE]
  > The cache will use only the first DNS server it successfully finds. -->

Considere la posibilidad de usar una caché de prueba para comprobar y refinar la configuración de DNS antes de usarla en un entorno de producción.

### <a name="refresh-storage-target-dns"></a>Actualización del DNS de destino de almacenamiento

Si el servidor DNS actualiza las direcciones IP, los destinos de almacenamiento NFS asociados dejarán de estar disponibles de manera temporal. Lea cómo actualizar las direcciones IP del sistema DNS personalizado en [Edición de destinos de almacenamiento](hpc-cache-edit-storage.md#update-ip-address-custom-dns-configurations-only).

## <a name="view-snapshots-for-blob-storage-targets"></a>Visualización de instantáneas de destinos de almacenamiento de blobs

Azure HPC Cache guarda automáticamente las instantáneas de almacenamiento para los destinos de almacenamiento de blobs de Azure. Las instantáneas proporcionan un punto de referencia rápido para el contenido del contenedor de almacenamiento de back-end.

Las instantáneas no son un reemplazo de las copias de seguridad de los datos y no incluyen ninguna información sobre el estado de los datos en caché.

> [!NOTE]
> Esta característica de instantánea es diferente de la incluida en el software de almacenamiento NetApp o Isilon. Esas implementaciones de instantáneas vacían los cambios de la memoria caché en el sistema de almacenamiento de back-end antes de tomar la instantánea.
>
> Por motivos de eficacia, la instantánea de Azure HPC Cache no vacía primero los cambios y solo registra los datos que se han escrito en el contenedor de blobs. Esta instantánea no representa el estado de los datos en caché, por lo que es posible que se excluyan los cambios recientes.

Esta característica solo está disponible para destinos de almacenamiento de blobs de Azure, y su configuración no se puede cambiar.

Las instantáneas se toman cada ocho horas, a las 0:00, 08:00 y 16:00 UTC.

Azure HPC Cache almacena instantáneas diarias, semanales y mensuales hasta que se sustituyen por otras nuevas. Los límites de retención de instantáneas son:

* Hasta 20 instantáneas diarias.
* Hasta 8 instantáneas semanales.
* Hasta 3 instantáneas mensuales.

Acceda a las instantáneas desde el directorio `.snapshot` en la raíz de su destino de almacenamiento de blobs montado.
