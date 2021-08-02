---
title: 'Opciones de almacenamiento para el contenedor de perfiles de FSLogix de Azure Virtual Desktop: Azure'
description: Opciones para almacenar el perfil de FSLogix de Azure Virtual Desktop en Azure Storage.
author: Heidilohr
ms.topic: conceptual
ms.date: 04/27/2021
ms.author: helohr
manager: femila
ms.openlocfilehash: ba1ae8c108f02dbd104361e1efe39f5583011124
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111755706"
---
# <a name="storage-options-for-fslogix-profile-containers-in-azure-virtual-desktop"></a>Opciones de almacenamiento para los contenedores de perfiles de FSLogix de Azure Virtual Desktop

Azure ofrece varias soluciones de almacenamiento que puede usar para almacenar el contenedor de perfiles de FSLogix. En este artículo se comparan las soluciones de almacenamiento que Azure ofrece para los contenedores de perfiles de usuario de FSLogix de Azure Virtual Desktop. Se recomienda almacenar contenedores de perfil de FSLogix en Azure Files en el caso de la mayoría de nuestros clientes.

Azure Virtual Desktop ofrece contenedores de perfiles de FSLogix como solución recomendada para los perfiles de usuario. FSLogix está diseñado para utilizar perfiles itinerantes en entornos informáticos remotos, como Azure Virtual Desktop. Al iniciar sesión, este contenedor se adjunta dinámicamente al entorno informático mediante el disco duro virtual (VHD) compatible de forma nativa y el disco duro virtual de Hyper-V (VHDX). El perfil de usuario está disponible inmediatamente y aparece en el sistema exactamente como un perfil de usuario nativo.

En las siguientes tablas se comparan las soluciones de almacenamiento que Azure Storage ofrece para los perfiles de usuario del contenedor de perfiles de FSLogix de Azure Virtual Desktop.

## <a name="azure-platform-details"></a>Detalles de la plataforma Azure

|Características|Azure Files|Azure NetApp Files|Espacios de almacenamiento directo|
|--------|-----------|------------------|---------------------|
|Caso de uso|Uso general|Ultrarrendimiento o migración desde NetApp local|Multiplataforma|
|Servicio de plataforma|Sí, solución nativa de Azure|Sí, solución nativa de Azure|No, administración automática|
|Disponibilidad regional|Todas las regiones|[Seleccionar regiones](https://azure.microsoft.com/global-infrastructure/services/?products=netapp&regions=all)|Todas las regiones|
|Redundancia|Redundancia local/redundancia de zona/redundancia geográfica/redundancia de zona geográfica|Redundancia local|Redundancia local/de zona/geográfica|
|Niveles y rendimiento| Estándar (optimizado para transacciones)<br>Premium<br>Hasta un máximo de 100 000 IOPS por recurso compartido con 10 GBps por recurso compartido a aproximadamente 3 ms de latencia|Estándar<br>Premium<br>Ultra<br>Hasta 4,5 Gbps por volumen a aproximadamente 1 ms de latencia. Para obtener información detallada sobre IOPS y rendimiento, consulte las [consideraciones de rendimiento de Azure NetApp Files](../azure-netapp-files/azure-netapp-files-performance-considerations.md) y las [preguntas más frecuentes](../azure-netapp-files/azure-netapp-files-faqs.md#how-do-i-convert-throughput-based-service-levels-of-azure-netapp-files-to-iops).|HDD estándar: hasta 500 IOPS por disco<br>SSD estándar: hasta 4000 IOPS por disco<br>SSD Premium: hasta 20 000 IOPS por disco<br>Se recomiendan discos Premium para Espacios de almacenamiento directo|
|Capacity|100 TiB por recurso compartido, hasta 5 PiB por cuenta de uso general |100 TiB por volumen, hasta 12,5 PiB por suscripción|32 TiB por disco como máximo|
|Infraestructura necesaria|Tamaño mínimo del recurso compartido: 1 GiB|Grupo de capacidad mínima: 4 TiB, tamaño de volumen mínimo: 100 GiB|Dos máquinas virtuales en Azure IaaS (+ testigo de la nube) o al menos tres máquinas virtuales sin él y costos por los discos|
|Protocolos|SMB 3.0/2.1, NFSv4.1 (versión preliminar), REST|NFSv3, NFSv 4.1 (versión preliminar), SMB 3.x/2.x|NFSv3, NFS v4.1, SMB 3.1|

## <a name="azure-management-details"></a>Detalles de administración de Azure

|Características|Azure Files|Azure NetApp Files|Espacios de almacenamiento directo|
|--------|-----------|------------------|---------------------|
|Acceso|En la nube, locales e híbridos (Azure File Sync)|En la nube, locales (mediante Express Route)|En la nube o en el entorno local|
|Copia de seguridad|Integración de instantáneas de copia de seguridad de Azure|Instantáneas de Azure NetApp Files|Integración de instantáneas de copia de seguridad de Azure|
|Seguridad y cumplimiento normativo|[Todos los certificados admitidos en Azure](https://www.microsoft.com/trustcenter/compliance/complianceofferings)|Norma ISO completada|[Todos los certificados admitidos en Azure](https://www.microsoft.com/trustcenter/compliance/complianceofferings)|
|Integración de Azure Active Directory|[Active Directory nativo y Azure Active Directory Domain Services](../storage/files/storage-files-active-directory-overview.md)|[Azure Active Directory Domain Services y Active Directory nativo](../azure-netapp-files/azure-netapp-files-faqs.md#does-azure-netapp-files-support-azure-active-directory)|Solo compatibilidad con Active Directory nativo o Azure Active Directory Domain Services|

Una vez que haya elegido el método de almacenamiento, consulte [Precios de Azure Virtual Desktop](https://azure.microsoft.com/pricing/details/virtual-desktop/) para información sobre los planes de precios.

## <a name="azure-files-tiers"></a>Niveles de Azure Files

Azure Files ofrece dos niveles diferentes de almacenamiento: Premium y Standard. Estos niveles permiten personalizar el rendimiento y el costo de los recursos compartidos de archivos para satisfacer los requisitos de su escenario.

- Los recursos compartidos de archivos Premium están respaldados por unidades de estado sólido (SSD) y se implementan en el tipo de cuenta de almacenamiento de FileStorage. Los recursos compartidos de archivos Premium proporcionan un alto rendimiento y una baja latencia coherentes para cargas de trabajo intensivas de entrada y salida (E/S). 

- Los recursos compartidos de archivos estándar están respaldados por unidades de disco duro (HDD) y se implementan en el tipo de cuenta de almacenamiento de uso general versión 2 (GPv2). Los recursos compartidos de archivos Estándar ofrecen un rendimiento confiable para cargas de trabajo de E/S menos sensibles a la variabilidad del rendimiento, como recursos compartidos de archivos de uso general y entornos de desarrollo y pruebas. Los recursos compartidos de archivos estándar solo están disponibles en un modelo de facturación de pago por uso.

En la tabla siguiente se enumeran las recomendaciones para el nivel de rendimiento que se debe utilizar en función de la carga de trabajo. Estas recomendaciones le ayudarán a seleccionar el nivel de rendimiento que se ajuste a sus objetivos de rendimiento, presupuesto y consideraciones regionales. Hemos basado estas recomendaciones en los escenarios de ejemplo de [tipos de carga de trabajo de Escritorio remoto](/windows-server/remote/remote-desktop-services/remote-desktop-workloads). 

| Tipo de carga de trabajo | Nivel de archivo recomendado |
|--------|-----------|
| Ligero (menos de 200 usuarios) | Recursos compartidos de archivos estándar |
| Ligero (más de 200 usuarios) | Recursos compartidos de archivos Premium o Estándar con varios recursos compartidos de archivos |
|Media|Recursos compartidos de archivos Prémium|
|Pesado|Recursos compartidos de archivos Prémium|
|Power|Recursos compartidos de archivos Prémium|

Para más información sobre el rendimiento de Azure Files, consulte el artículo sobre [objetivos de escalado de archivos y recursos compartido de archivos](../storage/files/storage-files-scale-targets.md#azure-files-scale-targets). Para más información sobre los precios, consulte [Precios de Azure Files](https://azure.microsoft.com/pricing/details/storage/files/).

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre los contenedores de perfiles de FSLogix, los discos de perfil de usuario y otras tecnologías de perfil de usuario, consulte la tabla de [Contenedores de perfiles de FSLogix y archivos de Azure ](fslogix-containers-azure-files.md).

Si está listo para crear sus propios contenedores de perfiles de FSLogix, comience con uno de estos tutoriales:

- [Introducción a los contenedores de perfiles de FSLogix en Azure Files en Azure Virtual Desktop](create-file-share.md)
- [Creación de un contenedor de perfiles de FSLogix para un grupo host mediante Azure NetApp Files](create-fslogix-profile-container.md)
- Las instrucciones de [Implementar un servidor de archivos de escalabilidad horizontal de Espacios de almacenamiento directo de dos nodos para almacenar UPD en Azure](/windows-server/remote/remote-desktop-services/rds-storage-spaces-direct-deployment/) también se aplican cuando se usa un contenedor de perfiles de FSLogix en lugar de un disco de perfil de usuario.

También puede empezar desde el principio y configurar su propia solución de Azure Virtual Desktop en [Creación de un inquilino en Azure Virtual Desktop](./virtual-desktop-fall-2019/tenant-setup-azure-active-directory.md).
