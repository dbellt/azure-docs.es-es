---
title: Ejecución de VM con SQL Server en una instancia de Azure Dedicated Host
description: Aprenda a ejecutar una VM con SQL Server en una instancia de Azure Dedicated Host.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
tags: azure-resource-manager
ms.service: virtual-machines-sql
ms.subservice: management
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 08/12/2019
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: 7b1259843a1195301ff66e7c8d1ad7650f9ca686
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112078751"
---
# <a name="run-sql-server-vm-on-an-azure-dedicated-host"></a>Ejecución de VM con SQL Server en una instancia de Azure Dedicated Host 
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

En este artículo se especifican los detalles del uso de una máquina virtual (VM) con SQL Server mediante [Azure Dedicated Host](../../../virtual-machines/dedicated-hosts.md). Puede encontrar más información sobre Azure Dedicated Host en la entrada de blog [Presentación de Azure Dedicated Host](https://azure.microsoft.com/blog/introducing-azure-dedicated-host/). 

## <a name="overview"></a>Información general
[Azure Dedicated Host](../../../virtual-machines/dedicated-hosts.md) es un servicio que proporciona servidores físicos (capaces de hospedar una o varias máquinas virtuales) dedicados a una suscripción de Azure. Los hosts dedicados son los mismos servidores físicos que se usan en los centros de datos de Microsoft y se proporcionan como un recurso. Puede aprovisionar hosts dedicados dentro de una región, zona de disponibilidad y dominio de error. Después, puede colocar las máquinas virtuales directamente en los hosts aprovisionados, en la configuración que más se ajuste a sus necesidades.

## <a name="limitations"></a>Limitaciones

- No todas las series de máquinas virtuales se admiten en los hosts dedicados, y la disponibilidad de las series de máquinas virtuales varía según la región. Para obtener más información, consulte la [Introducción a hosts dedicados de Azure](../../../virtual-machines/dedicated-hosts.md).

## <a name="licensing"></a>Licencias

Al colocar una VM con SQL Server en una instancia de Azure Dedicated Host, puede elegir entre dos opciones de licencia diferentes. 

  - **Licencia de máquina virtual de SQL**: es la opción de licencia existente, en la que se paga por cada licencia de VM con SQL Server existente individualmente. 
  - **Licencia de host dedicado**: el nuevo modelo de licencia disponible para Azure Dedicated Host, donde las licencias de SQL Server se agrupan y pagan en el nivel de host. 


Opciones de nivel de host para usar licencias de SQL Server existentes: 
  - Ventaja híbrida de Azure (AHB) de SQL Server Enterprise Edition
    - Disponible para los clientes con asociación de seguridad o suscripción.
    - Disfrute de licencia en todos los núcleos físicos disponibles y de una virtualización ilimitada (hasta el número máximo de vCPU admitido por el host).
        - Para obtener más información sobre cómo aplicar la Ventaja híbrida de Azure a Azure Dedicated Host, consulte las [preguntas más frecuentes sobre Ventaja híbrida de Azure](https://azure.microsoft.com/pricing/hybrid-benefit/faq/). 
  - Licencias de SQL Server adquiridas antes del 1 de octubre
      - La edición SQL Server Enterprise tiene opciones de licencia de nivel de host y por VM. 
      - La edición SQL Server Standard solo tiene disponible una opción de licencia por VM. 
          - Para obtener más información, consulte los [términos del producto de Microsoft](https://www.microsoft.com/licensing/product-licensing/products). 
  - Si no hay ninguna opción de nivel de host dedicado de SQL Server seleccionada, puede seleccionar Ventaja híbrida de Azure de SQL Server en el nivel de máquinas virtuales individuales, igual que haría en las máquinas virtuales con varios inquilinos.



## <a name="provisioning"></a>Aprovisionamiento  
El aprovisionamiento de una VM con SQL Server en el host dedicado no es diferente al de cualquier otra máquina virtual de Azure. Para ello, puede usar [Azure PowerShell](../../../virtual-machines/windows/dedicated-hosts-powershell.md), [Azure Portal](../../../virtual-machines/dedicated-hosts-portal.md) y la [CLI de Azure](../../../virtual-machines/linux/dedicated-hosts-cli.md).

El proceso de agregar una VM con SQL Server existente al host dedicado requiere tiempo de inactividad, pero no afectará a los datos y no habrá pérdida de datos. Sin embargo, antes de la migración es preciso realizar una copia de seguridad de todas las bases de datos, incluidas las bases de datos del sistema.

## <a name="virtualization"></a>Virtualización 

Una de las ventajas de un host dedicado es la virtualización ilimitada. Por ejemplo, puede tener licencias para 64 núcleos virtuales, pero puede configurar el host para que tenga 128 núcleos virtuales, así que tiene el doble de núcleos virtuales, pero solo paga la mitad de lo que pagaría por las licencias de SQL Server. 

Al ser su host, puede establecer la virtualización con una relación de 1:2. 

## <a name="faq"></a>Preguntas más frecuentes

**P: ¿Cómo funciona la Ventaja híbrida de Azure en las licencias de Windows Server/SQL Server en Azure Dedicated Host?**

A. Los clientes pueden usar el valor de sus licencias actuales de Windows Server y SQL Server con Software Assurance, o bien licencias de suscripción cualificadas, para pagar menos en Azure Dedicated Host mediante Ventaja híbrida de Azure. Los clientes de Windows Server Datacenter y SQL Server Enterprise Edition obtienen una virtualización ilimitada (se implementan tantas máquinas virtuales de Windows Server como sea posible en el host, en función de la capacidad física del servidor subyacente) cuando tienen licencia de todo el host y usan Ventaja híbrida de Azure.  Todas las cargas de trabajo de Windows Server y SQL Server de Azure Dedicated Host también pueden disfrutar de actualizaciones de seguridad ampliadas para Windows Server y SQL Server 2008/R2 sin costo adicional. 

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte los siguientes artículos. 

* [Introducción a SQL Server en máquinas virtuales Windows](sql-server-on-azure-vm-iaas-what-is-overview.md)
* [Preguntas más frecuentes de SQL Server en máquinas virtuales Windows](frequently-asked-questions-faq.yml)
* [Orientación de precios de SQL Server para máquinas virtuales de Azure](pricing-guidance.md)
* [Notas de la versión de SQL Server en Azure Virtual Machines](doc-changes-updates-release-notes.md)