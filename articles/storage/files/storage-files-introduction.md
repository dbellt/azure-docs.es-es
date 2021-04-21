---
title: Introducción a Azure Files | Microsoft Docs
description: Información general sobre Azure Files, el servicio que permite crear y utilizar recursos compartidos de archivos en red en la nube mediante el protocolo SMB estándar del sector.
author: roygara
ms.service: storage
ms.topic: overview
ms.date: 09/15/2020
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 7279c5578d969db9178ce85734e3a43f9eccdf8d
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107780054"
---
# <a name="what-is-azure-files"></a>¿Qué es Azure Files?
Azure Files ofrece recursos compartidos de archivos totalmente administrados en la nube a los que se puede acceder mediante el [protocolo SMB (Bloque de mensajes del servidor)](/windows/win32/fileio/microsoft-smb-protocol-and-cifs-protocol-overview) o el [protocolo NFS (Network File System)](https://en.wikipedia.org/wiki/Network_File_System) estándar del sector. Los recursos compartido de archivos de Azure se pueden montar simultáneamente mediante implementaciones locales o en la nube. A los recursos compartidos de archivos SMB de Azure Files se puede acceder desde clientes Windows, Linux y macOS. A los recursos compartidos de archivos NFS de Azure Files se puede acceder desde clientes Linux y macOS. Además, los recursos compartidos de archivos SMB de Azure Files se pueden almacenar en la caché de los servidores de Windows Server con Azure File Sync, lo que permite un acceso rápido allí donde se utilizan los datos.

Estos son algunos vídeos sobre los casos de uso comunes de Azure Files:
* [Reemplazo del servidor de archivos por un recurso compartido de archivos de Azure sin servidor](https://sec.ch9.ms/ch9/3358/0addac01-3606-4e30-ad7b-f195f3ab3358/ITOpsTalkAzureFiles_high.mp4)
* [Introducción a los contenedores de perfiles de FSLogix en Azure Files en Windows Virtual Desktop mediante el aprovechamiento de la autenticación de AD](https://www.youtube.com/embed/9S5A1IJqfOQ)

## <a name="why-azure-files-is-useful"></a>¿Por qué es útil Azure Files?
Los recursos compartidos de archivos de Azure se pueden usar para:

* **Reemplazar o complementar servidores de archivos locales**:  
    Azure Files puede utilizarse para reemplazar totalmente o complementar los servidores de archivos tradicionales locales o en dispositivos NAS. Desde sistemas operativos tan extendidos como Windows, macOS y Linux se puede montar directamente un recurso compartido de Azure Files desde cualquier lugar del mundo. Los recursos compartidos archivos SMB de Azure Files se pueden replicar también con Azure File Sync en servidores de Windows Server, locales o en la nube, para obtener un almacenamiento en caché eficiente y distribuido de los datos allí donde se usan. Con el reciente lanzamiento de la [autenticación con AD para Azure Files](storage-files-active-directory-overview.md), los recursos compartidos de archivos SMB de Azure Files pueden seguir funcionando con AD hospedado en un entorno local para el control de acceso. 

* **Aplicaciones "Lift-and-shift"** :  
    Azure Files facilita la migración mediante "lift and shift" de aplicaciones a la nube que espera un recurso compartido de archivos para almacenar datos de la aplicación de archivos o de un usuario. Azure Files permite la migración clásica mediante "lift and shift" en la que tanto la aplicación como sus datos se mueven a Azure, y la migración híbrida mediante "lift and shift" en la que los datos de la aplicación se mueven a Azure Files pero la aplicación continúa ejecutándose de forma local. 

* **Simplificar el desarrollo en la nube**:  
    Azure Files también se puede utilizar de muchas formas para simplificar los nuevos proyectos de desarrollo en la nube. Por ejemplo:
    * **Configuración de aplicaciones compartidas**  
        Un patrón habitual entre las aplicaciones distribuidas es contar con archivos de configuración en una ubicación centralizada que permite tener acceso a ellos desde muchas instancias de aplicaciones. Las instancias de la aplicación pueden cargar su configuración mediante la API de REST de Azure Files y los usuarios pueden acceder a ella según sea necesario montando el recurso compartido SMB localmente.

    * **Recurso compartido de diagnóstico**:  
        Un recurso compartido de Azure Files es un lugar adecuado para que las aplicaciones en la nube escriban sus registros, métricas y volcados de memoria. Las instancias de la aplicación pueden escribir los registros mediante la API de REST de Azure Files, y los desarrolladores pueden acceder a ellos montando el recurso compartido de archivos en su máquina local. Esto permite una gran flexibilidad, ya que los desarrolladores pueden adoptar el desarrollo en la nube sin tener que abandonar las herramientas existentes que ya conocen y disfrutan.

    * **Desarrollo, pruebas y depuración**:  
        Cuando los desarrolladores o administradores están trabajando en máquinas virtuales en la nube, a menudo necesitan diversas herramientas o utilidades. Copiar estas utilidades y herramientas en cada máquina virtual puede ser una tarea que consuma mucho tiempo. Mediante el montaje de un recurso compartido de Azure Files localmente en las máquinas virtuales, un desarrollador o administrador pueden acceder rápidamente a sus herramientas y utilidades, sin tener que copiarlos.
* **Contenedorización**:  
    los recursos compartidos de archivos de Azure se pueden usar como volúmenes persistentes para contenedores con estado. Los contenedores ofrecen funcionalidades de "compilar una vez, ejecutarse en cualquier lugar" que permiten a los desarrolladores acelerar la innovación. En el caso de los contenedores que acceden a datos sin procesar en cada inicio, se requiere un sistema de archivos compartidos para que estos contenedores puedan acceder al sistema de archivos, independientemente de la instancia en que se ejecuten.

## <a name="key-benefits"></a>Ventajas principales
* **Acceso compartido**. Los recursos compartidos de Azure Files admiten los protocolos SMB y NFS estándar del sector, lo que significa que puede reemplazar perfectamente los recursos compartidos de archivos en local por recursos compartidos de archivos de Azure sin preocuparse de compatibilidad de aplicaciones. La posibilidad de compartir un sistema de archivos entre varias máquinas, aplicaciones o instancias es una ventaja importante de Azure Files para aquellas aplicaciones que necesitan la posibilidad de compartir. 
* **Completamente administrado**. Los recursos compartidos de Azure Files pueden crearse sin necesidad de administrar ni el hardware ni un sistema operativo. Esto significa que no tiene que tratar con la aplicación de actualizaciones de seguridad críticas en el sistema operativo del servidor ni ocuparse de reemplazar discos duros defectuosos.
* **Herramientas y scripting**. Como parte de la administración de las aplicaciones de Azure, se pueden usar cmdlets de PowerShell y la CLI de Azure para crear, montar y administrar recursos compartidos de archivos de Azure. Los recursos compartidos de archivos de Azure se pueden crear y administrar mediante Azure Portal y el Explorador de Azure Storage. 
* **Resistencia**. Azure Files se creó desde sus orígenes para estar siempre disponible. Reemplazar los recursos compartidos de archivos en local por Azure Files significa que ya no tendrá que tratar con problemas de red o interrupciones del suministro eléctrico local. 
* **Programación amigable**. Las aplicaciones que se ejecutan en Azure pueden tener acceso a los datos en el recurso compartido mediante las [API de E/S del sistema](/dotnet/api/system.io.file). Por tanto, los desarrolladores pueden aprovechar el código y los conocimientos que ya tienen para migrar las aplicaciones actuales. Además de las API de E/S del sistema, puede usar las [Bibliotecas de cliente de Azure Storage](/previous-versions/azure/dn261237(v=azure.100)) o la [API de REST de Azure Storage](/rest/api/storageservices/file-service-rest-api).

## <a name="next-steps"></a>Pasos siguientes
* [Más información sobre los protocolos de recursos compartidos de archivos disponibles](storage-files-compare-protocols.md)
* [Creación de un recurso compartido de Azure Files](storage-how-to-create-file-share.md)
* [Conexión y montaje de un recurso compartido de archivos SMB en Windows](storage-how-to-use-files-windows.md)
* [Conexión y montaje de un recurso compartido de archivos SMB en Linux](storage-how-to-use-files-linux.md)
* [Conexión y montaje de un recurso compartido de archivos SMB en macOS](storage-how-to-use-files-mac.md)
* [Procedimiento para crear un recurso compartido de NFS](storage-files-how-to-create-nfs-shares.md)
