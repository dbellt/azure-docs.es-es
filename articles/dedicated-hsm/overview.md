---
title: ¿Qué es Dedicated HSM? - Azure Dedicated HSM | Microsoft Docs
description: Conozca cómo Azure Dedicated HSM ofrece almacenamiento de claves criptográficas en Azure.
services: dedicated-hsm
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: overview
ms.custom: mvc, seodec18
ms.date: 03/25/2021
ms.author: keithp
ms.openlocfilehash: d4c21835cfb9d3e665f3617047c6934282282c1e
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108125956"
---
# <a name="what-is-azure-dedicated-hsm"></a>¿Qué es Azure Dedicated HSM?

Azure Dedicated HSM ofrece almacenamiento de claves criptográficas en Azure. Dedicated HSM cumple los requisitos de seguridad más estrictos. Es la solución idónea para los clientes que necesitan dispositivos validados con la certificación FIPS 140-2 nivel 3 y un control completo y exclusivo del dispositivo HSM. 

 Los dispositivos HSM se implementan de manera global en varias regiones de Azure. Se pueden aprovisionar fácilmente como un par de dispositivos y configurar para alta disponibilidad. Los dispositivos HSM también se pueden aprovisionar entre regiones para protección frente a conmutaciones por error en el nivel regional. Microsoft ofrece el servicio Dedicated HSM mediante el uso de los dispositivos [Thales Luna 7 HSM modelo A790](https://cpl.thalesgroup.com/encryption/hardware-security-modules/network-hsms). Este dispositivo ofrece los mayores niveles de rendimiento y opciones de integración criptográficas. 

Una vez aprovisionados, los dispositivos HSM se conectan directamente a la red virtual de un cliente. También son accesibles mediante una aplicación local y herramientas de administración si se configura la conectividad de punto a sitio o de VPN de sitio a sitio. Los clientes obtienen el software y la documentación para configurar y administrar los dispositivos HSM desde el [portal de soporte técnico al cliente de Thales](https://supportportal.thalesgroup.com/csm).

## <a name="why-use-azure-dedicated-hsm"></a>Motivos para usar Azure Dedicated HSM

### <a name="fips-140-2-level-3-compliance"></a>Cumplimiento de la norma FIPS 140-2 nivel 3

Muchas organizaciones se rigen por estrictas regulaciones del sector que dictan que las claves criptográficas deben almacenarse en los HSM validados conforme a [FIPS 140-2 (nivel 3)](https://csrc.nist.gov/publications/detail/fips/140/2/final). Azure Dedicated HSM y una nueva oferta de inquilino único, [HSM administrado por Azure Key Vault](../key-vault/managed-hsm/index.yml), ayudan a los clientes de diversos segmentos del sector, como el sector de los servicios financieros, las agencias gubernamentales y otros a cumplir los requisitos del nivel 3 de FIPS 140-2. No obstante, el servicio [Azure Key Vault](../key-vault/index.yml) multiinquilino de Microsoft usa actualmente los HSM validados conforme a FIPS 140-2 (nivel 2). 

### <a name="single-tenant-devices"></a>Dispositivos de un único inquilino

Muchos de nuestros clientes requieren un único inquilino en el dispositivo de almacenamiento criptográfico. El servicio Azure Dedicated HSM les permite aprovisionar un dispositivo físico desde uno de los centros de datos distribuidos globalmente de Microsoft. Una vez que se aprovisiona a un cliente, solo ese cliente puede acceder al dispositivo.

### <a name="full-administrative-control"></a>Control administrativo completo

Muchos clientes requieren un control administrativo completo y acceso exclusivo a su dispositivo para fines administrativos. Una vez que se aprovisiona un dispositivo, solo el cliente tiene acceso de nivel administrativo o de aplicación al mismo.

 Microsoft no tiene control administrativo una vez que el cliente accede por primera vez al dispositivo, momento en el cual el cliente cambia la contraseña. A partir de ese punto, el cliente es un único inquilino verdadero con control administrativo completo y funcionalidad de administración de aplicaciones. Microsoft sí conserva el acceso a nivel de supervisión (no un rol de administrador) para la telemetría a través de la conexión de puerto serie. Este acceso abarca indicadores de hardware, como temperatura, estado del suministro eléctrico y estado del ventilador. 
 
 Si lo desea, el cliente puede deshabilitar esta supervisión necesaria pero, si lo hace, no recibirá alertas de estado proactivas de parte de Microsoft.

### <a name="high-performance"></a>Alto rendimiento

Para este servicio, se seleccionó el dispositivo Thales por diversos motivos. Ofrece una amplia variedad de compatibilidad con algoritmos criptográficos, una variedad de sistemas operativos compatibles y una amplia compatibilidad con API. El modelo específico implementado ofrece un rendimiento excelente con 10 000 operaciones por segundo para RSA-2048. Admite 10 particiones que se pueden usar para las instancias de aplicaciones únicas. Este dispositivo es de baja latencia, alta capacidad y alto rendimiento.

### <a name="unique-cloud-based-offering"></a>Oferta única basada en la nube

Microsoft reconoció una necesidad concreta en un conjunto exclusivo de clientes y es el único proveedor de servicios en la nube que ofrece un servicio HSM que cumple con la certificación FIPS 140-2 nivel 3 y que ofrece una integración tan amplia de aplicaciones basadas en la nube y locales.

## <a name="is-azure-dedicated-hsm-right-for-you"></a>¿Es Azure Dedicated HSM adecuado para usted?

Azure Dedicated HSM es un servicio especializado que permite cumplir los requisitos únicos de un tipo específico de organización a gran escala. Como resultado, es de esperar que la mayoría de los clientes de Azure no se encuadren en el perfil de cliente que usará este servicio. Muchas descubrirán que el servicio Azure Key Vault es más adecuado y rentable. Para ayudarlo a decidir si es adecuado para sus requisitos, hemos identificado los siguientes criterios.

### <a name="best-fit"></a>Casos en los que está indicado

Azure Dedicated HSM es el más adecuado para escenarios de "migración mediante lift-and-shift" que requieren un acceso directo y exclusivo a los dispositivos HSM. Entre los ejemplos se incluyen:

- Migración de aplicaciones locales a Azure Virtual Machines
- Migración de aplicaciones de Amazon AWS EC2 a máquinas virtuales que usan el servicio AWS Cloud HSM Classic (Amazon no ofrece este servicio para los nuevos clientes)
- Ejecución de un software empaquetado como Apache/Ngnix SSL Offload, Oracle TDE y ADCS en Azure Virtual Machines 

### <a name="not-a-fit"></a>Casos en los que no está indicado

Azure Dedicated HSM no es idóneo para el siguiente tipo de escenario: Los servicios en la nube de Microsoft que admiten el cifrado con claves administradas por los clientes (como Azure Information Protection, Azure Disk Encryption, Azure Data Lake Storage, Azure Storage, Azure SQL Database y clave de cliente para Office 365) no están integrados con Azure Dedicated HSM.

### <a name="it-depends"></a>Depende

Si Azure Dedicated HSM funcionará para usted depende de una posible combinación compleja de requisitos y compromisos que puede o no asumir. Un ejemplo es el requisito de la certificación FIPS 140-2 nivel 3. Se trata de un requisito común, y Azure Dedicated HSM y la nueva oferta de inquilino único, [HSM administrado por Azure Key Vault](../key-vault/managed-hsm/index.yml) son actualmente las únicas opciones que lo cumplen. Si estos requisitos obligatorios no proceden, a menudo se puede elegir entre Azure Key Vault y Azure Dedicated HSM. Evalúe sus requisitos antes de tomar una decisión.

Las situaciones en las que tendrá que ponderar sus opciones incluyen las siguientes: 

- Nuevo código que se ejecuta en una máquina virtual de Azure del cliente
- TDE de SQL Server en una máquina virtual de Azure
- Cifrado en el lado de cliente de Azure Storage
- SQL Server y Azure SQL Database Always Encrypted

## <a name="next-steps"></a>Pasos siguientes

Se trata de un servicio altamente especializado. Por lo tanto, se recomienda que comprenda totalmente los conceptos clave de este conjunto de documentación, incluidos los precios, el soporte técnico y los contratos de nivel de servicio. 

Las [guías de integración de Thales](https://cpl.thalesgroup.com/partners/overview) le permiten facilitar el aprovisionamiento de dispositivos HSM en un entorno de red virtual existente. También existen guías de procedimientos para ayudarle a determinar cómo configurar la arquitectura de implementación.

* [Alta disponibilidad](high-availability.md)
* [Seguridad física](physical-security.md)
* [Redes](networking.md)
* [Compatibilidad](supportability.md)
* [Supervisión](monitoring.md)