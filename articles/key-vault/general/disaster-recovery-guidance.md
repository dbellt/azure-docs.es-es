---
title: 'Disponibilidad y redundancia de Azure Key Vault: Azure Key Vault | Microsoft Docs'
description: Obtenga información sobre la disponibilidad y redundancia de Azure Key Vault.
services: key-vault
author: msmbaldwin
ms.service: key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 03/31/2021
ms.author: mbaldwin
ms.openlocfilehash: 7a23522da2bf6d1c5f9c6a76ab3d9c92cbc64897
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108749544"
---
# <a name="azure-key-vault-availability-and-redundancy"></a>Redundancia y disponibilidad de Azure Key Vault

Azure Key Vault tiene varias capas de redundancia para garantizar la disponibilidad de las claves y los secretos para su aplicación, aunque se produzcan errores de componentes individuales del servicio.

> [!NOTE]
> Esta guía se aplica a los almacenes. Los grupos de HSM administrados usan un modelo diferente de alta disponibilidad y recuperación ante desastres. Para más información, consulte [Guía de recuperación ante desastres de HSM administrado](../managed-hsm/disaster-recovery-guide.md).

El contenido del almacén de claves se replica dentro de la región y en una región secundaria que se encuentre a una distancia mínima de 241 km pero dentro de la misma ubicación geográfica para mantener una alta durabilidad de las claves y los secretos. Consulte [Regiones emparejadas de Azure](../../best-practices-availability-paired-regions.md) para más información sobre pares de regiones específicas. La excepción al modelo de regiones emparejadas es una sola región geográfica, por ejemplo, Sur de Brasil, Centro de Catar. Estas regiones solo permiten la opción de mantener los datos residentes dentro de la misma región. Tanto Sur de Brasil como Centro de Catar usan el almacenamiento con redundancia de zona (ZRS) para replicar los datos tres veces dentro de la misma ubicación o región. Para AKV Premium, solo se usan dos de las tres regiones para replicar los datos de la del HSM.

Si se produce un error en algún componente individual dentro del servicio del almacén de claves, los componentes alternativos de la región se encargan de atender la solicitud para garantizar que no se pierde funcionalidad. No es necesario realizar ninguna acción para iniciar este proceso, ya que se realiza automáticamente y es transparente para el usuario.

En el caso excepcional de que toda una región de Azure pase a no estar disponible, las solicitudes efectuadas a Azure Key Vault de esa región se enrutarán automáticamente (un proceso conocido como *conmutación por error*) a una región secundaria excepto en el caso de las regiones Sur de Brasil y Centro de Catar. Cuando la región primaria vuelva a estar disponible, las solicitudes se enrutarán a ella nuevamente (*conmutación por recuperación*). Conviene insistir en que no es necesaria ninguna acción, ya que este proceso se realiza automáticamente.

En las regiones Sur de Brasil y Centro de Catar, debe planear la recuperación de los almacenes de claves de Azure en un escenario de errores de la región. Para realizar una copia de seguridad y restaurar el almacén de claves de Azure en una región de su elección, complete los pasos que se detallan en [Copia de seguridad de Azure Key Vault](backup.md). 

En este diseño de alta disponibilidad, Azure Key Vault no requiere ningún tiempo de inactividad para las actividades de mantenimiento.

Hay algunas advertencias que deben tenerse en cuenta:

* La conmutación por error de región, en caso de producirse, puede tardar varios minutos en completarse. Es posible que se produzca un error en las solicitudes realizadas durante este tiempo, antes de la conmutación por error.
* Si usa un vínculo privado para conectarse al almacén de claves, la conexión puede tardar hasta 20 minutos en restablecerse si se produce una conmutación por error. 
* Durante la conmutación por error, el almacén de claves está en modo de solo lectura. Las solicitudes compatibles con este modo son las siguientes:
  * Enumeración de certificados
  * Obtención de certificados
  * Enumeración de secretos
  * Obtención de secretos
  * Enumeración de claves
  * Obtención de (propiedades de) claves
  * Cifrado
  * Descifrado
  * Encapsulado
  * Desencapsulado
  * Verify
  * Firma
  * Copia de seguridad

* Durante la conmutación por error, no podrá realizar cambios en las propiedades del almacén de claves. No podrá cambiar la directiva de acceso ni los valores y la configuración del firewall.

* Cuando tenga lugar la conmutación por error, todos los tipos de solicitudes (incluidas de lectura *y* escritura) pasarán a estar disponibles.
