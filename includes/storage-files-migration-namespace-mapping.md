---
title: archivo de inclusión
description: archivo de inclusión
services: storage
author: fauhse
ms.service: storage
ms.topic: include
ms.date: 2/20/2020
ms.author: fauhse
ms.custom: include file
ms.openlocfilehash: ffa5c899686596256622a6ea334e486c52ec710d
ms.sourcegitcommit: ba8f0365b192f6f708eb8ce7aadb134ef8eda326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2021
ms.locfileid: "109645047"
---
En este paso, establecerá cuántos recursos compartidos de archivos de Azure se necesitan. Una sola instancia de Windows Server (o clúster) puede sincronizar hasta 30 recursos compartidos de archivos de Azure.

Es posible que tenga más carpetas en los volúmenes que actualmente comparte localmente como recursos compartidos de SMB en sus usuarios y aplicaciones. La manera más sencilla de visualizar este escenario es imaginar un recurso compartido local que asigne 1:1 a un recurso compartido de archivos de Azure. Si tiene un número suficientemente pequeño de recursos compartidos, por debajo de 30 para una sola instancia de Windows Server, se recomienda una asignación 1:1.

Si tiene más de 30 recursos compartidos, a menudo no es necesaria la asignación 1:1 de recursos compartidos locales a un recurso compartido de archivos de Azure. Considere las opciones siguientes.

#### <a name="share-grouping"></a>Agrupación de recursos compartidos

Por ejemplo, si el departamento de RR. HH. tiene 15 recursos compartidos, podría considerar la posibilidad de almacenar todos los datos de RR. HH. en un solo recurso compartido de archivos de Azure. El almacenamiento de varios recursos compartidos locales en un recurso compartido de archivos de Azure no evita que tenga que crear los 15 recursos compartidos de SMB habituales en la instancia local de Windows Server. Solo significa que organiza las carpetas raíz de estos 15 recursos compartidos como subcarpetas en una carpeta común. A continuación, sincronizará esta carpeta común con un recurso compartido de archivos de Azure. De este modo, solo se necesita un único recurso compartido de archivos de Azure en la nube para este grupo de recursos compartidos locales.

#### <a name="volume-sync"></a>Sincronización de volúmenes

Azure File Sync admite la sincronización de la raíz de un volumen con un recurso compartido de archivos de Azure. Si sincroniza la raíz del volumen, todas las subcarpetas y los archivos van al mismo recurso compartido de archivos de Azure.

La sincronización de la raíz del volumen no siempre es la mejor opción. La sincronización de varias ubicaciones ofrece varias ventajas. Por ejemplo, ayuda a disminuir el número de elementos por ámbito de sincronización. Probamos los recursos compartidos de archivos de Azure y Azure File Sync con 100 millones de elementos (archivos y carpetas) por recurso compartido. Pero un procedimiento es intentar mantener el número por debajo de 20 o 30 millones en un solo recurso compartido. La configuración de Azure File Sync con un número de elementos menor no solo es beneficiosa para la sincronización de archivos. Un menor número de elementos también beneficia a escenarios como estos:

* El examen inicial del contenido de la nube puede realizarse más rápido, lo que a su vez reduce la espera de que aparezca el espacio de nombres en un servidor habilitado para Azure File Sync.
* La restauración en la nube a partir de una instantánea de recursos compartidos de archivos de Azure se hará con mayor rapidez.
* La recuperación ante desastres de un servidor local puede acelerarse de forma considerable.
* Los cambios hechos directamente en un recurso compartido de archivos de Azure (sin sincronización) se pueden detectar y sincronizar más rápido.

> [!TIP]
> Si no está seguro de cuántos archivos y carpetas tiene, consulte la herramienta TreeSize de JAM Software GmbH.

#### <a name="a-structured-approach-to-a-deployment-map"></a>Un enfoque estructurado de una asignación de implementación

Antes de implementar el almacenamiento en la nube en un paso posterior, es importante crear una asignación entre carpetas locales y recursos compartidos de archivos de Azure. Esta asignación informará de cuántos recursos del *grupo de sincronización* de Azure File Sync se van a aprovisionar y de cuáles van a ser. Un grupo de sincronización está relacionado con el recurso compartido de archivos de Azure y la carpeta de su servidor, y establece una conexión de sincronización.

Para decidir cuántos recursos compartidos de archivos de Azure necesita, revise los límites y procedimientos recomendados siguientes. Eso le va a ayudar a optimizar la asignación.

* Un servidor donde está instalado el agente de Azure File Sync puede sincronizarse con hasta 30 recursos compartidos de archivos de Azure.
* Un recurso compartido de archivos de Azure se implementa en una cuenta de almacenamiento. Esta disposición hace que la cuenta de almacenamiento sea un destino de escalado para los números de rendimiento, como IOPS y rendimiento.

  En teoría, un recurso compartido de archivos de Azure estándar puede saturar el rendimiento máximo que puede proporcionar una cuenta de almacenamiento. Si coloca varios recursos compartidos en una única cuenta de almacenamiento, está creando un grupo compartido de IOPS y rendimiento para esos recursos compartidos. Si solo tiene previsto asociar Azure File Sync a estos recursos compartidos de archivos, la agrupación de varios recursos compartidos de archivos de Azure en la misma cuenta de almacenamiento no plantea ningún problema. Revise los destinos de rendimiento para obtener información más detallada sobre las métricas pertinentes. Estas limitaciones no se aplican al almacenamiento prémium, donde el rendimiento se aprovisiona y garantiza explícitamente para cada recurso compartido.

  Si tiene previsto mover una aplicación a Azure que usará el recurso compartido de archivos de Azure de forma nativa, es posible que necesite un mayor rendimiento del recurso compartido de archivos de Azure. Si este tipo de uso es una posibilidad, incluso en el futuro, lo mejor es crear un único recurso compartido de archivos de Azure estándar en su propia cuenta de almacenamiento.
* Hay un límite de 250 cuentas de almacenamiento por suscripción por cada región de Azure.

> [!TIP]
> Teniendo en cuenta esta información, suele ser necesario agrupar varias carpetas de nivel superior de sus volúmenes en un nuevo directorio raíz común. Luego se sincroniza este nuevo directorio raíz y todas las carpetas agrupadas en él, en un solo recurso compartido de archivos de Azure. Esta técnica permite permanecer dentro del límite de 30 sincronizaciones de recursos compartidos de archivos de Azure por servidor.
>
> Esta agrupación bajo una raíz común no afecta al acceso a sus datos. Las ACL se mantienen como están. Solo tiene que ajustar algunas rutas de acceso a los recursos compartidos (como las de los recursos compartidos SMB o NFS) que podría haber en las carpetas de servidor locales que ahora han cambiado a una raíz común. No cambia nada más.

> [!IMPORTANT]
> El vector de escala más importante para Azure File Sync es el número de elementos (archivos y carpetas) que tienen que sincronizarse. Para más información, revise los [objetivos de escala de Azure File Sync](../articles/storage/files/storage-files-scale-targets.md#azure-file-sync-scale-targets).

Se recomienda mantener bajo el número de elementos por ámbito de sincronización. Ese es un factor importante que se debe tener en cuenta en la asignación de carpetas a recursos compartidos de archivos de Azure. Azure File Sync se prueba con 100 millones elementos (archivos y carpetas) por recurso compartido. Pero a menudo es mejor mantener el número de elementos por debajo de 20 o 30 millones en un solo recurso compartido. Divida el espacio de nombres en varios recursos compartidos si empieza a superar estos números. Puede seguir agrupando varios recursos compartidos locales en el mismo recurso compartido de archivos de Azure, siempre y cuando se mantenga aproximadamente por debajo de estos números. Esto le proporcionará más espacio para crecer.

En una situación tal, es posible que un conjunto de carpetas pueda sincronizarse de forma lógica con el mismo recurso compartido de archivos de Azure (mediante el nuevo enfoque de carpeta raíz común mencionado anteriormente). Pero puede que siga siendo mejor reagrupar carpetas de modo que se sincronicen con dos recursos compartidos de archivos de Azure en lugar de uno. Puede usar este enfoque para mantener equilibrado el número de archivos y carpetas por recurso compartido de archivos en el servidor. También puede dividir los recursos compartidos locales y sincronizarlos entre más servidores locales, lo que agrega la posibilidad de sincronizar 30 recursos compartidos de archivos de Azure más por cada servidor adicional.

#### <a name="create-a-mapping-table"></a>Creación de una tabla de asignación

:::row:::
    :::column:::
        [![Diagrama que muestra un ejemplo de una tabla de asignación. Descargue el archivo siguiente para experimentar y usar el contenido de esta imagen.](media/storage-files-migration-namespace-mapping/namespace-mapping.png)](media/storage-files-migration-namespace-mapping/namespace-mapping-expanded.png#lightbox)
    :::column-end:::
    :::column:::
        Use la información anterior para decidir cuántos recursos compartidos de archivos de Azure necesita, y qué partes de los datos existentes terminarán en cuál recurso compartido de archivos de Azure.
        
        Cree una tabla para registrar sus ideas, de modo que pueda consultarla cuando sea necesario. La organización es importante, ya que puede ser fácil perder detalles del plan de asignación cuando se aprovisionan muchos recursos de Azure a la vez. Descargue el siguiente archivo de Excel para usarlo como plantilla para ayudar a crear la asignación.

[//]: # (HTML aparece como la única forma de realizar la adición de una tabla anidada de dos columnas con análisis de imágenes de trabajo y texto/hipervínculo en la misma línea.)

<br>
<table>
    <tr>
        <td>
            <img src="media/storage-files-migration-namespace-mapping/excel.png" alt="Excel icon that sets the context for the download.">
        </td>
        <td>
            <a href="https://download.microsoft.com/download/1/8/D/18DC8184-E7E2-45EF-823F-F8A36B9FF240/Azure File Sync - Namespace Mapping.xlsx">Descargue una plantilla de asignación de espacios de nombres.</a>
        </td>
    </tr>
</table>
    :::column-end:::
:::row-end:::
