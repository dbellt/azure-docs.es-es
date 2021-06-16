---
title: Ampliación de la compatibilidad con SQL Server 2008 y SQL Server 2008 R2
description: Amplíe la compatibilidad de SQL Server 2008 y SQL Server 2008 R2 mediante la migración de una instancia de SQL Server a Azure o mediante la compra de compatibilidad ampliada para mantener las instancias locales.
services: virtual-machines-windows
documentationcenter: ''
author: MashaMSFT
tags: azure-service-management
ms.service: virtual-machines-sql
ms.subservice: management
ms.topic: conceptual
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 04/08/2019
ms.author: mathoma
ms.reviewer: jroth
ms.custom: seo-lt-2019
ms.openlocfilehash: 297bb5b3aa464cede71e204133df3e856bb9ba1f
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112079668"
---
# <a name="extend-support-for-sql-server-2008-and-sql-server-2008-r2-with-azure"></a>Ampliar la compatibilidad de SQL Server 2008 y SQL Server 2008 R2 con Azure
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

SQL Server 2008 y SQL Server 2008 R2 han alcanzado el [fin del ciclo de vida de soporte (EOS)](https://www.microsoft.com/sql-server/sql-server-2008). Dado que muchos de nuestros clientes siguen usando ambas versiones, ofrecemos varias opciones para seguir recibiendo soporte técnico. Puede migrar las instancias de SQL Server locales a máquinas virtuales (VM) de Azure, migrar a Azure SQL Database o mantener el entorno local y comprar actualizaciones de seguridad ampliadas.

A diferencia de una instancia administrada, la migración a una VM de Azure no requiere la recertificación de sus aplicaciones. Además, a diferencia de lo que ocurre al mantener un entorno local, al migrar a una máquina virtual de Azure, recibirá revisiones de seguridad ampliadas gratuitas.

En el resto de este artículo se proporcionan consideraciones sobre la migración de la instancia de SQL Server a una VM de Azure.

Para obtener más información sobre las opciones de fin del soporte, consulte [Fin del soporte](/sql/sql-server/end-of-support/sql-server-end-of-life-overview).

## <a name="provisioning"></a>Aprovisionamiento

Hay una imagen de pago por uso de **SQL Server 2008 R2 en Windows Server 2008 R2** disponible en Azure Marketplace.

Los clientes que usan SQL Server 2008 deberán instalarse automáticamente o actualizar a SQL Server 2008 R2. Del mismo modo, los clientes que usan Windows Server 2008 deberán implementar su máquina virtual desde un disco duro virtual personalizado o actualizar a Windows Server 2008 R2.

Las imágenes implementadas con Azure Marketplace tienen la extensión SQL IaaS preinstalada. La extensión SQL IaaS es un requisito para las licencias flexibles y la aplicación de revisiones automatizada. Los clientes que implementen máquinas virtuales autoinstaladas deberán instalar manualmente la extensión SQL IaaS. La extensión SQL IaaS no se admite en Windows Server 2008.

> [!NOTE]
> Si bien el servidor SQL Server y las hojas **Crear** y **Administrar** funcionarán con la imagen de SQL Server 2008 R2 en Azure Portal, las siguientes características _no se admiten_: Copias de seguridad automáticas, integración de Azure Key Vault, R Services y configuración de almacenamiento.

## <a name="licensing"></a>Licencias
Las implementaciones de SQL Server 2008 R2 de pago por uso se pueden convertir en [Ventaja híbrida de Azure](https://azure.microsoft.com/pricing/hybrid-benefit/).

Para convertir una licencia basada en Software Assurance (SA) en una licencia de pago por uso, los clientes deben registrarse con la [extensión del Agente de IaaS de SQL](sql-agent-extension-manually-register-single-vm.md). Después del registro, el tipo de licencia de SQL será intercambiable entre Ventaja híbrida de Azure y "de pago por uso".

Las instancias de SQL Server 2008 o SQL Server 2008 R2 instaladas automáticamente en la máquina virtual de Azure se pueden registrar con la extensión del Agente de IaaS de SQL y convertir su tipo de licencia en de pago por uso.

## <a name="migration"></a>Migración
Puede migrar las instancias de SQL Server con finalización del soporte a una máquina virtual de Azure con métodos manuales de copia de seguridad y restauración. Este es el método de migración más común del entorno local a una máquina virtual de Azure.

### <a name="azure-site-recovery"></a>Azure Site Recovery

Para las migraciones masivas, se recomienda usar el servicio [Azure Site Recovery](../../../site-recovery/site-recovery-overview.md). Con Azure Site Recovery, los clientes pueden replicar la máquina virtual completa, incluido SQL Server desde un entorno local a una máquina virtual de Azure.

SQL Server necesita instantáneas coherentes con la aplicación de Azure Site Recovery para garantizar la recuperación. Azure Site Recovery admite instantáneas coherentes con la aplicación con un intervalo mínimo de una hora. El objetivo de punto de recuperación (RPO) mínimo posible para SQL Server con las migraciones de Azure Site Recovery es de una hora. El objetivo de tiempo de recuperación (RTO) es de dos horas más el tiempo de recuperación de SQL Server.

### <a name="database-migration-service"></a>Database Migration Service

[Azure Database Migration Service](../../../dms/dms-overview.md) es una opción para los clientes si migran desde un entorno local a una máquina virtual de Azure mediante la actualización de SQL Server a SQL Server 2012 y versiones posteriores.

## <a name="disaster-recovery"></a>Recuperación ante desastres

Las soluciones de recuperación ante desastres para SQL Server con finalización del soporte en una máquina virtual de Azure son las siguientes:

- **Copias de seguridad de SQL Server**: Use Azure Backup para ayudar a proteger su instancia de EOS SQL Server 2008 y 2008 R2 contra ransomware, eliminación accidental y daños con un RPO de 15 minutos y una recuperación a un momento dado. Para más información, consulte [este artículo](../../../backup/sql-support-matrix.md#scenario-support).
- **Trasvase de registros**: puede crear una réplica de trasvase de registros en otra zona o región de Azure con restauraciones continuas para reducir el RTO. Tiene que configurar manualmente el trasvase de registros.
- **Azure Site Recovery**: Puede replicar la VM entre zonas y regiones a través de la replicación de Azure Site Recovery. SQL Server necesita instantáneas coherentes con la aplicación para garantizar la recuperación en caso de desastre. Azure Site Recovery ofrece un RPO mínimo de una hora y un RTO de dos horas (más el tiempo de recuperación de SQL Server) para la recuperación ante desastres de SQL Server con finalización del soporte.

## <a name="security-patching"></a>Aplicación de revisiones de seguridad

Las actualizaciones de seguridad ampliadas para las máquinas virtuales con SQL Server se entregarán en los canales de Microsoft Update cuando la máquina virtual con SQL Server se haya registrado con la [extensión del Agente de IaaS de SQL](sql-agent-extension-manually-register-single-vm.md). Las revisiones se pueden descargar manual o automáticamente.

*Automated patching* está habilitada de forma predeterminada. La aplicación de revisiones automatizada permite a Azure aplicar automáticamente las revisiones de SQL Server y del sistema operativo. Puede especificar un día de la semana, una hora y una duración para una ventana de mantenimiento si está instalada la extensión de SQL Server IaaS. Azure realiza la aplicación de revisión en esta ventana de mantenimiento. La programación de la ventana de mantenimiento utiliza la configuración regional de la máquina virtual para la hora. Para más información, consulte [Aplicación de revisiones automatizada para SQL Server en Azure Virtual Machines](automated-patching.md).


## <a name="next-steps"></a>Pasos siguientes

Migración de la máquina virtual con SQL Server a Azure:

* [Migración de una base de datos SQL Server a SQL Server en una VM de Azure](migrate-to-vm-from-sql-server.md)

Introducción a SQL Server en Azure Virtual Machines:

* [Creación de una máquina virtual con SQL Server en Azure Portal](sql-vm-create-portal-quickstart.md)

Obtenga respuestas a las preguntas más habituales acerca de las máquinas virtuales con SQL Server:

* [Preguntas más frecuentes para SQL Server en Azure Virtual Machines](frequently-asked-questions-faq.yml)

Obtenga más información sobre las opciones de fin del soporte y las actualizaciones de seguridad ampliadas:

* [Fin del soporte](/sql/sql-server/end-of-support/sql-server-end-of-life-overview) & [Actualizaciones de seguridad ampliadas](/sql/sql-server/end-of-support/sql-server-extended-security-updates)