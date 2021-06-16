---
title: Características de seguridad usadas con máquinas virtuales de Azure
titleSuffix: Azure security
description: Este artículo ofrece una visión general de las principales características de seguridad de Azure que se pueden usar con Azure Virtual Machines.
services: security
documentationcenter: na
author: TerryLanfear
manager: rkarlin
editor: TomSh
ms.assetid: 467b2c83-0352-4e9d-9788-c77fb400fe54
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/2/2019
ms.author: terrylan
ms.openlocfilehash: 6092476180614fe11f8c1199c079b0599f829779
ms.sourcegitcommit: 832e92d3b81435c0aeb3d4edbe8f2c1f0aa8a46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111554433"
---
# <a name="azure-virtual-machines-security-overview"></a>Información general de seguridad de Azure Virtual Machines
Este artículo ofrece una introducción a las principales características de seguridad de Azure que pueden usarse con máquinas virtuales.

Puede usar las máquinas virtuales de Azure para implementar una amplia gama de soluciones informáticas con facilidad. El servicio es compatible con Microsoft Windows, Linux, Microsoft SQL Server, Oracle, IBM, SAP y Azure BizTalk Services. Gracias a ello, puede implementar cualquier carga de trabajo y cualquier lenguaje en casi cualquier sistema operativo.

Una máquina virtual de Azure le ofrece la flexibilidad de la virtualización sin necesidad de adquirir y mantener el hardware físico que ejecuta la máquina virtual. Puede compilar e implementar aplicaciones con la tranquilidad de saber que sus datos están protegidos en centros de datos de alta seguridad.

Con Azure, puede crear soluciones compatibles con seguridad mejorada que:

* Protegen sus máquinas virtuales de virus y malware.
* Protegen con cifrado su información confidencial.
* Protegen el tráfico de red.
* Identifican y detectan amenazas.
* Satisfacen los requisitos de cumplimiento.  

## <a name="antimalware"></a>Antimalware

Con Azure, puede usar el software antimalware de proveedores de seguridad como Microsoft, Symantec, Trend Micro y Kaspersky. Este software le ayudará a proteger las máquinas virtuales de archivos malintencionados, adware y otras amenazas.

Microsoft Antimalware para Azure Cloud Services y Virtual Machines es una funcionalidad de protección en tiempo real que permite identificar y eliminar virus, spyware y otro software malintencionado.  Microsoft Antimalware para Azure activa alertas configurables cuando software no deseado o malintencionado intenta instalarse o ejecutarse en los sistemas de Azure.

Microsoft Antimalware para Azure es una solución de un único agente dirigida a entornos de aplicaciones e inquilinos. Está concebida para su ejecución en segundo plano sin intervención del usuario. Puede implementar la protección en función de las necesidades de sus cargas de trabajo de aplicaciones, con configuración de protección básica o personalizada avanzada que incluye supervisión antimalware.

Obtenga más información sobre [Microsoft Antimalware para Azure](antimalware.md) y las características principales disponibles.

Obtenga más información acerca del software antimalware para proteger las máquinas virtuales:

* [Implementación de soluciones antimalware en Azure Virtual Machines](https://azure.microsoft.com/blog/deploying-antimalware-solutions-on-azure-virtual-machines/)
* [Instalación y configuración de Trend Micro Deep Security como servicio en una máquina virtual de Azure](/previous-versions/azure/virtual-machines/extensions/trend)
* [Instalación y configuración de Endpoint Protection en una máquina virtual de Azure](../../virtual-machines/extensions/symantec.md)
* [Soluciones de seguridad en Azure Marketplace](https://azure.microsoft.com/marketplace/?term=security)

Para una protección aún más eficaz, considere el uso de la [Protección contra amenazas avanzada de Windows Defender](/mem/configmgr/protect/deploy-use/defender-advanced-threat-protection). Con ATP de Windows Defender, obtiene lo siguiente:

* [Reducción de la superficie expuesta a ataques](/windows/security/threat-protection/windows-defender-atp/overview-attack-surface-reduction)  
* [Protección de próxima generación](/windows/security/threat-protection/windows-defender-antivirus/windows-defender-antivirus-in-windows-10)  
* [Protección y respuesta de punto de conexión](/windows/security/threat-protection/windows-defender-atp/overview-endpoint-detection-response)
* [Investigación y corrección automatizadas](/windows/security/threat-protection/windows-defender-atp/automated-investigations-windows-defender-advanced-threat-protection)
* [Puntuación segura](/windows/security/threat-protection/microsoft-defender-atp/tvm-microsoft-secure-score-devices)
* [Búsqueda avanzada](/windows/security/threat-protection/windows-defender-atp/overview-hunting-windows-defender-advanced-threat-protection)
* [Administración y API](/windows/security/threat-protection/windows-defender-atp/management-apis)
* [Protección contra amenazas de Microsoft](/windows/security/threat-protection/windows-defender-atp/threat-protection-integration)

Más información:

* [Introducción a protección contra amenazas avanzada de Windows Defender](/windows/security/threat-protection/microsoft-defender-atp/microsoft-defender-advanced-threat-protection)  
* [Información general sobre las capacidades de ATP de Windows Defender](/microsoft-365/security/defender-endpoint/whats-new-in-microsoft-defender-atp)  

## <a name="hardware-security-module"></a>Módulo de seguridad de hardware

Las protecciones del cifrado y autenticación se pueden mejorar si se mejora la clave de seguridad. Puede simplificar la administración y la seguridad de claves y secretos críticos guardándolos en Azure Key Vault.

Key Vault permite guardar claves en módulos de seguridad de hardware (HSM) que tienen la certificación FIPS 140-2 nivel 2. Sus claves de cifrado de SQL Server para copias de seguridad o [cifrado de datos transparente](/sql/relational-databases/security/encryption/transparent-data-encryption) se pueden almacenar en Key Vault con otras claves y secretos de sus aplicaciones. Los permisos y el acceso a estos elementos protegidos se administran con [Azure Active Directory](https://azure.microsoft.com/documentation/services/active-directory/).

Más información:

* [¿Qué es Azure Key Vault?](../../key-vault/general/overview.md)
* [Blog de Azure Key Vault](/archive/blogs/kv/)

## <a name="virtual-machine-disk-encryption"></a>Cifrado de discos de máquinas virtuales

Azure Disk Encryption es una nueva funcionalidad que permite cifrar los discos de las máquinas virtuales de Windows y Linux. Azure Disk Encryption usa la característica [BitLocker](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc732774(v=ws.11)) estándar del sector de Windows y la característica [dm-crypt](https://en.wikipedia.org/wiki/Dm-crypt) de Linux para ofrecer cifrado de volumen para el sistema operativo y los discos de datos.

La solución se integra con Azure Key Vault para controlar y administrar los secretos y las claves de cifrado de los discos en la suscripción de Key Vault. Gracias a ello, se garantiza que todos los datos de los discos de máquinas virtuales se cifren en reposo en Azure Storage.

Más información:

* [Azure Disk Encryption para máquinas virtuales IaaS](./azure-disk-encryption-vms-vmss.md)
* [Inicio rápido: Cifrado de una máquina virtual IaaS Windows con Azure PowerShell](../../virtual-machines/linux/disk-encryption-powershell-quickstart.md)

## <a name="virtual-machine-backup"></a>Copia de seguridad de máquina virtual

Azure Backup es una solución escalable que protege los datos de su aplicación sin necesidad de realizar ninguna inversión y afrontando unos costos operativos mínimos. Los errores de una aplicación pueden dañar los datos, y los errores humanos pueden crear errores en las aplicaciones. Con Azure Backup, las máquinas virtuales que ejecutan Windows y Linux están protegidas.

Más información:

* [¿Qué es Azure Backup?](../../backup/backup-overview.md)
* [P+F del servicio Azure Backup](../../backup/backup-azure-backup-faq.yml)

## <a name="azure-site-recovery"></a>Azure Site Recovery

Una parte importante de la estrategia de BCDR de la organización es averiguar cómo mantener las aplicaciones y cargas de trabajo corporativas listas para su funcionamiento cuando se producen interrupciones planificadas e imprevistas. Azure Site Recovery le ayuda a coordinar la replicación, la conmutación por error y la recuperación de aplicaciones y cargas de trabajo para que estén disponibles desde una ubicación secundaria si la ubicación principal deja de funcionar.

Site Recovery:

* **Simplificar su estrategia de BCDR**: Site Recovery facilita el control de la replicación, la conmutación por error y la recuperación de varias cargas de trabajo y aplicaciones de negocios desde una única ubicación. Site Recovery organiza la replicación y la conmutación por error pero no intercepta los datos de la aplicación ni obtiene información alguna sobre ella.
* **Proporcionar replicación flexible**: con Site Recovery puede replicar las cargas de trabajo que se ejecutan en máquinas virtuales de Hyper-V, máquinas virtuales de VMware y servidores físicos con Windows o Linux.
* **Admite recuperación y conmutación por error**: Site Recovery proporciona conmutaciones por error de prueba que admiten maniobras de recuperación ante desastres sin que los entornos de producción se vean afectados. También puede ejecutar conmutaciones por error planeadas sin pérdidas de datos para interrupciones previstas o conmutaciones por error con una pérdida de datos mínima (según la frecuencia de replicación) ante desastres inesperados. Después de la conmutación por error puede ejecutar una conmutación por recuperación a los sitios principales. Site Recovery proporciona planes de recuperación que pueden incluir scripts y libros de Azure Automation para que pueda personalizar la conmutación por error y la recuperación de aplicaciones de varios niveles.
* **Eliminar centros de datos secundarios**: puede replicar en un sitio local secundario o en Azure. Usar Azure como destino de la recuperación ante desastres elimina el costo y la complejidad de mantener un sitio secundario. Los datos replicados se almacenan en Azure Storage.
* **Se integra con tecnologías de BCDR existentes**: Site Recovery se asocia con otras características de BCDR de las aplicaciones. Por ejemplo, puede usar Site Recovery para proteger el back-end de SQL Server de las cargas de trabajo corporativas. Esto incluye la compatibilidad nativa con SQL Server AlwaysOn para administrar la conmutación por error de los grupos de disponibilidad.

Más información:

* [¿Qué es Azure Site Recovery?](../../site-recovery/site-recovery-overview.md)
* [¿Cómo funciona Azure Site Recovery?](../../site-recovery/azure-to-azure-architecture.md)
* [¿Qué cargas de trabajo se pueden proteger con Azure Site Recovery?](../../site-recovery/site-recovery-workload.md)

## <a name="virtual-networking"></a>Redes virtuales

Las máquinas virtuales necesita conectividad de red. Para satisfacer este requisito, es necesario que las máquinas virtuales de Azure estén conectadas a una instancia de Azure Virtual Network.

Azure Virtual Network es una construcción lógica creada encima del tejido de red físico de Azure. Cada instancia lógica de Azure Virtual Network está aislada de todas las demás redes virtuales de Azure. Este aislamiento contribuye a garantizar que otros clientes de Microsoft Azure no puedan acceder al tráfico de red de sus implementaciones.

Más información:

* [Azure Network Security Overview (Información general sobre Azure Network Security)](network-overview.md)
* [Información general sobre Virtual Network](../../virtual-network/virtual-networks-overview.md)
* [Networking features and partnerships for Enterprise scenarios (Características de red y asociaciones para escenarios empresariales)](https://azure.microsoft.com/blog/networking-enterprise/)

## <a name="security-policy-management-and-reporting"></a>Informes y administración de directivas de seguridad

Azure Security Center ayuda a evita y a detectar las amenazas, además de a responder a ellas. Security Center aporta visibilidad mejorada y control sobre la seguridad de los recursos de Azure. Proporciona una supervisión de la seguridad y una administración de directivas integradas en suscripciones de Azure. Le ayuda a detectar amenazas que podrían pasar desapercibidas, y funciona con un amplio ecosistema de soluciones de seguridad.

Security Center le ayuda a optimizar y supervisar la seguridad de las máquinas virtuales de la siguiente manera:

* Proporcionando [recomendaciones de seguridad](../../security-center/security-center-recommendations.md) para las máquinas virtuales. Las recomendaciones de ejemplo incluyen lo siguiente: aplicar actualizaciones del sistema, configurar puntos de conexión de listas de control de acceso, habilitar antimalware, habilitar grupos de seguridad de red y aplicar cifrado de discos.
* Supervisando el estado de las máquinas virtuales.

Más información:

* [Introducción al Centro de seguridad de Azure](../../security-center/security-center-introduction.md)
* [Preguntas más frecuentes sobre el Azure Security Center](../../security-center/faq-general.yml)
* [Guía de planeamiento y operaciones de Azure Security Center](../../security-center/security-center-planning-and-operations-guide.md)

## <a name="compliance"></a>Cumplimiento normativo

Azure Virtual Machines tiene las certificaciones de FISMA, FedRAMP, HIPAA, PCI DSS nivel 1 y otros programas de cumplimiento fundamentales. Esta certificación facilita en gran medida que sus propias aplicaciones de Azure cumplan los requisitos de cumplimiento y que su empresa satisfaga una amplia variedad de requisitos normativos nacionales e internacionales.

Más información:

* [Centro de confianza de Microsoft: Cumplimiento normativo](https://www.microsoft.com/en-us/trustcenter/compliance)
* [Trusted Cloud: Microsoft Azure Security, Privacy, and Compliance (La nube de confianza: Seguridad, privacidad y cumplimiento de Microsoft Azure)](https://download.microsoft.com/download/1/6/0/160216AA-8445-480B-B60F-5C8EC8067FCA/WindowsAzure-SecurityPrivacyCompliance.pdf)

## <a name="confidential-computing"></a>Computación confidencial

Si bien la computación confidencial técnicamente no forma parte de la seguridad de la máquina virtual, esta seguridad pertenece al tema de nivel superior de la seguridad "computacional". La computación confidencial pertenece a la categoría de seguridad "computacional".

La computación confidencial garantiza que los datos, cuando están "descubiertos" (lo cual es necesario para un procesamiento eficiente), se mantienen protegidos dentro de un entorno de ejecución de confianza https://en.wikipedia.org/wiki/Trusted_execution_environment (también conocido como enclave). En la ilustración siguiente se muestra un ejemplo.  

Los entornos de ejecución de confianza garantizan que no hay ninguna manera de ver datos u operaciones desde el exterior, ni siquiera con un depurador. Incluso aseguran que solo se permite el acceso a los datos al código autorizado. Si el código está alterado o modificado, se deniegan las operaciones y se deshabilita el entorno. El entorno de ejecución de confianza aplica estas protecciones a todo el proceso de ejecución del código que contiene.

Más información:

* [Introducing Azure confidential computing](https://azure.microsoft.com/blog/introducing-azure-confidential-computing/) (Introducción a la computación confidencial de Azure)  
* [Azure confidential computing](https://azure.microsoft.com/blog/azure-confidential-computing/) (Computación confidencial de Azure)  

## <a name="next-steps"></a>Pasos siguientes

Aprenda sobre los [procedimientos recomendados de seguridad](iaas.md) para máquinas virtuales y sistemas operativos.
