---
title: Movimiento de un almacén a una grupo de recursos diferente en Azure Key Vault | Microsoft Docs
description: Guía para mover un almacén de claves a un grupo de recursos diferente.
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 03/31/2021
ms.author: mbaldwin
ms.openlocfilehash: 14ecbcaa35153438601a3dabb70f5b38006ded1b
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107749571"
---
# <a name="moving-an-azure-key-vault-across-resource-groups"></a>Movimiento de un almacén de Azure Key Vault entre grupos de recursos

## <a name="overview"></a>Información general

El movimiento de un almacén de claves entre grupos de recursos es una característica que es compatible con el almacén de claves. El movimiento de un almacén de claves entre grupos de recursos no afectará a las configuraciones de directiva de acceso o firewall del almacén de claves. Las aplicaciones conectadas y las entidades de servicio deben seguir funcionando según lo previsto.

> [!IMPORTANT]
> **No pueden migrarse los almacenes de claves que se usan para el cifrado de discos.**
> Si utiliza Key Vault con el cifrado de discos para una máquina virtual, el almacén de claves no puede migrarse a otro grupo de recursos o a una suscripción mientras esté habilitado el cifrado de discos. Antes de mover el almacén de claves a un nuevo grupo de recursos o a una nueva suscripción, debe deshabilitar el cifrado de discos. 

## <a name="design-considerations"></a>Consideraciones de diseño

Su organización puede haber implementado Azure Policy con cumplimiento o exclusión en el nivel de grupo de recursos. Puede haber un conjunto diferente de asignaciones de directiva en el grupo de recursos en el que se encuentra el almacén de claves y el grupo de recursos al que se va a trasladar el almacén de claves. Un conflicto en los requisitos de la directiva tiene la posibilidad de interrumpir las aplicaciones.

### <a name="example"></a>Ejemplo

Tiene una aplicación conectada al almacén de claves que crea certificados que son válidos durante dos años. El grupo de recursos al que está intentando trasladar el almacén de claves tiene una asignación de directiva que bloquea la creación de certificados que son válidos durante más de un año. Después de trasladar el almacén de claves al nuevo grupo de recursos, una asignación de directiva de Azure bloqueará la operación que crea un certificado válido durante dos años.

### <a name="solution"></a>Solución

Asegúrese de ir a la página Azure Policy del Azure Portal y examine las asignaciones de directiva de su grupo de recursos actual, así como del grupo de recursos al que está pasando, y asegúrese de que no haya ninguna discrepancia.

## <a name="procedure"></a>Procedimiento

1. Iniciar sesión en Azure Portal
2. Vaya a almacén de claves.
3. Haga clic en la pestaña "Introducción".
4. Seleccione el botón "Mover".
5. Seleccione "Mover a otro grupo de recursos" en las opciones de lista desplegable.
6. Seleccione el grupo de recursos a donde quiere mover el almacén de claves
7. Confirme la advertencia relacionada con el traslado de recursos.
8. Seleccione Aceptar.

Key Vault evaluará ahora la validez del movimiento de recursos y le avisará de cualquier error. Si no se encuentra ningún error, se completará el traslado de recursos. 
