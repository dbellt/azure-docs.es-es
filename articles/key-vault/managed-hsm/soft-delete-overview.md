---
title: Eliminación temporal de HSM administrado de Azure Key Vault | Microsoft Docs
description: La eliminación temporal en HSM administrado permite recuperar las claves y las instancias de HSM eliminadas.
ms.service: key-vault
ms.subservice: managed-hsm
ms.topic: conceptual
author: amitbapat
ms.author: ambapat
ms.date: 06/01/2021
ms.openlocfilehash: 673260fb8b78cb08ccd8581862238dc2e0341362
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111415403"
---
# <a name="managed-hsm-soft-delete-overview"></a>Visión general de la eliminación temporal de HSM administrado

> [!IMPORTANT]
> La eliminación temporal está activada de manera predeterminada para los recursos de HSM administrado. No puede desactivarse.

> [!IMPORTANT]
> Los recursos de HSM administrado eliminados temporalmente se seguirán facturando según su tarifa por hora completa hasta que se purguen.

La característica de eliminación temporal de HSM administrado permite la recuperación de las claves y HSM eliminados. Específicamente, esta seguridad ofrece las siguientes protecciones:

- Una vez que se elimina un HSM o una clave, se podrá recuperar durante un período configurable de 7 a 90 días naturales. El período de retención se puede establecer durante la creación de HSM. Si no se especifica ningún valor, el período de retención predeterminado se establecerá en 90 días. De esta forma, los usuarios tendrán tiempo suficiente para darse cuenta de la eliminación accidental de una clave o HSM y responder a ella.
- Para eliminar de forma permanente una clave, se deben realizar dos operaciones. En primer lugar, el usuario debe eliminar la clave, que la coloca en estado de eliminación temporal. En segundo lugar, el usuario debe purgar la clave en estado de eliminación temporal. La operación de purga requiere que el usuario tenga asignado un rol "Managed HSM Crypto Officer". Estas protecciones adicionales reducen el riesgo de que un usuario elimine una clave o HSM por accidente o de forma malintencionada.


## <a name="soft-delete-behavior"></a>Comportamiento de eliminación temporal

La eliminación temporal está activada de manera predeterminada para HSM administrado. No puede crear un recurso de HSM administrado con la eliminación temporal deshabilitada.

Con la eliminación temporal habilitada, los recursos marcados como eliminados se retienen durante un período especificado (de forma predeterminada, 90 días). El servicio proporciona además un mecanismo para recuperar las claves o HSM eliminados, lo que permite deshacer la eliminación.

El período de retención predeterminado es de 90 días, sin embargo, durante la creación de recursos de HSM, es posible establecer el intervalo de la directiva de retención en un valor de 7 a 90 días. La directiva de retención de protección de purgas usa el mismo intervalo. Una vez establecido, el intervalo de la directiva de retención no se puede cambiar.

No puede volver a usar el nombre de un recurso de HSM que se ha eliminado temporalmente hasta que haya transcurrido el período de retención y se haya purgado el recurso de HSM.

## <a name="purge-protection"></a>Protección de purgas

La protección contra purgas es un comportamiento opcional y **no está habilitado de forma predeterminada**. Se puede activar a través de la [CLI](./recovery.md?tabs=azure-cli) o [Powershell](./recovery.md?tabs=azure-powershell).

Cuando la protección contra purgas está activada, un HSM o una clave en estado eliminado no se pueden purgar hasta que haya transcurrido el período de retención. Los HSM y claves eliminados temporalmente todavía se pueden recuperar, lo que garantiza que se seguirá la directiva de retención.

El período de retención predeterminado es de 90 días, pero es posible establecer el intervalo de la directiva de retención en un valor de 7 a 90 días en el momento de creación de un HSM. El intervalo de directiva de retención solo se puede establecer en el momento de crear un HSM. No se puede modificar más tarde.

Vea [Uso de la eliminación flexible de HSM administrado con la CLI](./recovery.md?tabs=azure-cli#managed-hsm-cli) o [Uso de la eliminación flexible de HSM administrado con PowerShell](./recovery.md?tabs=azure-powershell#managed-hsm-powershell).

## <a name="managed-hsm-recovery"></a>Recuperación del HSM administrado

Al eliminar un HSM, el servicio crea un recurso de proxy en la suscripción, agregando metadatos suficientes para la recuperación. El recurso de proxy es un objeto almacenado, disponible en la misma ubicación que el HSM eliminado. 

## <a name="key-recovery"></a>Recuperación de claves

Cuando se elimina una clave, el servicio la colocará en un estado eliminado, de manera que no será accesible para ninguna operación. En este estado, las claves se pueden enumerar, recuperar o purgar (eliminar de forma permanente). Para ver los objetos, use el comando `az keyvault key list-deleted` de la CLI de Azure (como se documenta en [Eliminación temporal de HSM administrado y protección contra purgas con la CLI](./recovery.md?tabs=azure-cli#keys-cli)) o el parámetro `-InRemovedState` de Azure PowerShell (como se describe en [Eliminación temporal de HSM administrado y protección contra purgas con PowerShell](./recovery.md?tabs=azure-powershell#keys-powershell)).  

Al mismo tiempo, HSM administrado programará la eliminación de los datos subyacentes correspondientes al HSM o clave eliminados para su ejecución tras un intervalo de retención predeterminado. El registro DNS correspondiente al HSM también se retiene durante el intervalo de retención.

## <a name="soft-delete-retention-period"></a>Período de retención de la eliminación temporal

Los recursos eliminados temporalmente se conservan durante un período de tiempo determinado: 90 días. Durante el intervalo de retención de eliminación temporal, se dan las circunstancias siguientes:

- Puede confeccionar una lista de todos los HSM y claves que se encuentran en estado de eliminación temporal en su suscripción, así como acceder a la información de eliminación y recuperación sobre ellos.
  - Solo los usuarios con el rol de colaborador de HSM administrado pueden enumerar los HSM eliminados. Se recomienda que los usuarios creen un rol personalizado con estos permisos especiales para tratar los almacenes eliminados.
- No es posible crear un HSM administrado con el mismo nombre en la misma ubicación; en consecuencia, no se puede crear una clave en un HSM determinado si contiene una clave con el mismo nombre y está en estado eliminado.
- Solo los usuarios con el rol "Managed HSM Contributor" pueden enumerar, ver, recuperar y purgar HSM administrados.
- Solo los usuarios con el rol "Managed HSM Crypto Officer" pueden enumerar, ver, recuperar y purgar claves.
  
A menos que se recupere un HSM administrado o clave, al final del intervalo de retención, el servicio realiza una purga del HSM o clave eliminados temporalmente. La eliminación de recursos no se puede volver a programar.

### <a name="billing-implications"></a>Implicaciones de facturación

HSM administrado es un servicio de un solo inquilino. Al crear un HSM administrado, el servicio reserva los recursos subyacentes asignados al HSM. Estos recursos permanecen asignados incluso cuando el HSM está en estado eliminado. Por lo tanto, se le facturará el HSM mientras se encuentre en estado eliminado.

## <a name="next-steps"></a>Pasos siguientes

Las dos guías siguientes ofrecen los escenarios de uso principal para uso de la eliminación temporal.

- [Uso de la eliminación temporal de HSM administrado con PowerShell](./recovery.md?tabs=azure-powershell) 
- [Uso de la eliminación temporal de HSM administrado con la CLI](./recovery.md?tabs=azure-cli)
