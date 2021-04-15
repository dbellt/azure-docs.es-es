---
title: Errores comunes y problemas conocidos al migrar a Azure Cloud Services (soporte extendido)
description: Información general de los errores comunes al migrar desde Cloud Services (clásico) a Cloud Services (soporte extendido)
ms.topic: how-to
ms.service: cloud-services-extended-support
ms.subservice: classic-to-arm-migration
author: tanmaygore
ms.author: tagore
ms.reviewer: mimckitt
ms.date: 2/08/2021
ms.custom: ''
ms.openlocfilehash: 58203730793202649c401d96182469fa1eac6ef1
ms.sourcegitcommit: b8995b7dafe6ee4b8c3c2b0c759b874dff74d96f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/03/2021
ms.locfileid: "106286815"
---
# <a name="common-errors-and-known-issues-when-migration-to-azure-cloud-services-extended-support"></a>Errores comunes y problemas conocidos al migrar a Azure Cloud Services (soporte extendido)

En este artículo se tratan los problemas conocidos y los errores comunes que pueden surgir al migrar desde Cloud Services (clásico) a Cloud Services (soporte extendido). 

## <a name="known-issues"></a>Problemas conocidos
Los siguientes problemas son conocidos y se están abordando.

| Problemas conocidos | Mitigación | 
|---|---|
| Instancias de rol que se reinician UD por UD después de una confirmación correcta. | La operación de reinicio sigue el mismo método que los lanzamientos mensuales de SO invitado. No confirme la migración de Cloud Services con una sola instancia de rol o afectada por un reinicio.| 
| Azure Portal no puede leer el estado de migración después de una actualización del explorador. | Vuelva a ejecutar las operaciones de validación y preparación para volver al estado de migración original. | 
| El certificado se muestra como recurso secreto en Key Vault. | Después de la migración, vuelva a cargar el certificado como recurso de certificado para simplificar la operación de actualización en Cloud Services (soporte extendido). | 
| Las etiquetas de implementación no se guardan como etiquetas como parte de la migración. | Cree manualmente las etiquetas después de la migración para mantener esta información.
| El nombre del grupo de recursos está todo en mayúsculas. | No afecta. La solución todavía no está disponible. |
| El nombre del bloqueo en Cloud Services (soporte extendido) es incorrecto. | No afecta. La solución todavía no está disponible. | 
| El nombre de la dirección IP es incorrecto en la hoja del portal de Cloud Services (soporte extendido). | No afecta. La solución todavía no está disponible. | 
| Se muestra un nombre DNS no válido para la dirección IP virtual después de la operación de actualización en un servicio en la nube migrado. | No afecta. La solución todavía no está disponible. | 
| Después de una preparación correcta, no se permite la vinculación de una nueva implementación de Cloud Services (soporte extendido) como intercambiable. | No vincule un nuevo servicio en la nube como intercambiable a un servicio en la nube preparado. | 
| Los mensajes de error tienen que actualizarse. | No afecta. | 

## <a name="common-migration-errors"></a>Errores comunes en la migración
Errores de migración comunes y pasos de mitigación. 

| Mensaje de error | Detalles | 
|---|---|
| No se pudo encontrar el tipo de recurso en el espacio de nombres `Microsoft.Compute` del valor api version "2020-10-01-preview". | [Registre la suscripción](in-place-migration-overview.md#setup-access-for-migration) para la marca de características CloudServices para acceder a la versión preliminar pública. | 
| Se produjo un error interno en el servidor. Vuelva a intentarlo. | Vuelva a intentar la operación, use [Microsoft Q&A](https://docs.microsoft.com/answers/topics/azure-cloud-services-extended-support.html) o póngase en contacto con el soporte técnico. | 
| El servidor detectó un error inesperado al intentar asignar recursos de red para el servicio en la nube. Vuelva a intentarlo. | Vuelva a intentar la operación, use [Microsoft Q&A](https://docs.microsoft.com/answers/topics/azure-cloud-services-extended-support.html) o póngase en contacto con el soporte técnico. | 
| La implementación nombre-de-implementación en el servicio en la nube nombre-del-servicio-en-la-nube debe estar dentro de una red virtual para poder migrarla. | La implementación no se encuentra en una red virtual. Consulte [este](in-place-migration-technical-details.md#migration-of-deployments-not-in-a-virtual-network) documento para obtener más detalles. | 
| No se admite la migración de la implementación nombre-de-implementación en el servicio en la nube nombre-del-servicio-en-la-nube porque está en la región nombre-de-región. Regiones permitidas: [lista de regiones disponibles]. | La región todavía no se admite para la migración. | 
| No se puede migrar la implementación nombre-de-implementación en el servicio en la nube nombre-del-servicio-en-la-nube porque no hay subredes asociadas a los roles nombre-de-rol. Asocie todos los roles con una subred y vuelva a intentar la migración del servicio en la nube. | Para actualizar la implementación de Cloud Services (clásico), colóquela en una subred antes de la migración. |  
| No se puede migrar la implementación nombre-de-implementación en el servicio en la nube nombre-del-servicio-en-la-nube porque la implementación requiere al menos una característica que no esté registrada en la suscripción en Azure Resource Manager. Registre todas las características necesarias para migrar esta implementación. Características que faltan: [lista de características que faltan]. | Póngase en contacto con el soporte técnico para registrar las marcas de características. | 
| No se puede migrar la implementación porque el servicio en la nube de la implementación tiene dos espacios ocupados. La migración de servicios en la nube solo se admite para implementaciones que son la única implementación en su servicio en la nube. Elimine la otra implementación en el servicio en la nube para continuar con la migración de esta implementación. | Consulte la lista de [escenarios no admitidos](in-place-migration-overview.md#unsupported-configurations--migration-scenarios) para obtener más información. | 
| La implementación nombre-de-implementación en HostedService nombre-del-servicio-en-la-nube está en estado intermedio: estado. No se permite la migración. | La implementación se está creando, eliminando o actualizando. Espere a que se complete la operación y vuelva a intentarlo. | 
| La implementación nombre-de-implementación en el servicio hospedado nombre-del-servicio-en-la-nube tiene IP reservadas, pero ningún nombre de IP reservada. Para resolver este problema, actualice el nombre de la IP reservada o póngase en contacto con el servicio de asistencia al cliente de Microsoft Azure. | Actualice la implementación del servicio en la nube. | 
| La implementación nombre-de-implementación en el servicio hospedado nombre-del-servicio-en-la-nube tiene IP reservadas nombre-de-ip-reservada, pero ningún punto de conexión en la IP reservada. Para resolver este problema, agregue al menos un punto de conexión a la IP reservada. | Agregue un punto de conexión a la IP reservada. | 
| La migración de la implementación{0} en HostedService {1} está en proceso de confirmarse y no puede cambiarse hasta que se complete correctamente.  | Espere o vuelva a intentar la operación. | 
| La migración de la implementación {0} en HostedService {1} está en proceso de anularse y no puede cambiarse hasta que se complete correctamente. | Espere o vuelva a intentar la operación. |
| Una o más VM en la implementación {0} de HostedService {1} está llevando a cabo una operación de actualización. No se puede migrar hasta que la operación anterior se complete correctamente. Inténtelo de nuevo más tarde. | Espere a que la operación finalice. | 
| La migración no es compatible con la implementación {0} del servicio HostedService {1} porque usa las características siguientes que aún no se admiten para la migración: implementación no de red virtual.| La implementación no se encuentra en una red virtual. Consulte [este](in-place-migration-technical-details.md#migration-of-deployments-not-in-a-virtual-network) documento para obtener más detalles. | 
| El nombre de la red virtual no puede ser nulo ni estar vacío. | Proporcionar el nombre de la red virtual en el cuerpo de la solicitud de REST. | 
| El nombre de la subred no puede ser nulo ni estar vacío. | Proporcione el nombre de la subred en el cuerpo de la solicitud de REST. | 
| DestinationVirtualNetwork debe establecerse en uno de los valores siguientes: Default (Predeterminado), New (Nuevo) o Existing (Existente). | Proporcione la propiedad DestinationVirtualNetwork en el cuerpo de la solicitud de REST. | 
| Opción de destino de red virtual predeterminada no implementada. | El valor "default" no se admite para la propiedad DestinationVirtualNetwork en el cuerpo de la solicitud de REST. | 
| No se puede migrar la implementación {0} porque CSPKG no está disponible. | Actualice la implementación y vuelva a intentarlo. | 
| La subred con el identificador "{0}" está en una ubicación diferente a la de la implementación "{1}" en el servicio hospedado "{2}". La ubicación de la subred es "{3}"y la ubicación del servicio hospedado es "{4}".  Especifique una subred en la misma ubicación que la implementación. | Actualice el servicio en la nube para que tenga tanto la subred como el servicio en la nube en la misma ubicación antes de la migración. | 
| La migración de la implementación {0} en HostedService {1} está en proceso de anularse y no puede cambiarse hasta que se complete correctamente. | Espere a que se complete la anulación o vuelva a intentar la anulación. Use [Microsoft Q&A](https://docs.microsoft.com/answers/topics/azure-cloud-services-extended-support.html) o, de lo contrario, póngase en contacto con el soporte técnico. | 
| La implementación {0} de HostedService {1} no está preparada para la migración. | Ejecute la preparación en el servicio en la nube antes de ejecutar la operación de confirmación. | 
| Error UnknownExceptionInEndExecute: Contract.Assert: rgName es NULL o está vacío: Excepción recibida en EndExecute que no es RdfeException. |   Use [Microsoft Q&A](https://docs.microsoft.com/answers/topics/azure-cloud-services-extended-support.html) o póngase en contacto con el soporte técnico. | 
| UnknownExceptionInEndExecute: se canceló una tarea: Excepción recibida en EndExecute que no es RdfeException. | Use [Microsoft Q&A](https://docs.microsoft.com/answers/topics/azure-cloud-services-extended-support.html) o póngase en contacto con el soporte técnico. | 
| XrpVirtualNetworkMigrationError: Error de migración de la red virtual. | Use [Microsoft Q&A](https://docs.microsoft.com/answers/topics/azure-cloud-services-extended-support.html) o póngase en contacto con el soporte técnico. | 
| La implementación {0} de HostedService {1} pertenece a la red virtual {2}. Migre la red virtual {2} para migrar este HostedService {1}. | Consulte [Migración de redes virtuales](in-place-migration-technical-details.md#virtual-network-migration). | 
| La cuota actual para el nombre de recurso en Azure Resource Manager es insuficiente para completar la migración. La cuota actual es de {0}, el adicional necesario es de {1}. Genere una solicitud de soporte técnico para aumentar la cuota y vuelva a intentar la migración una vez que se haya aumentado la cuota.    | Siga los canales adecuados para solicitar un aumento de la cuota: <br>[Aumento de la cuota de recursos de red](../azure-portal/supportability/networking-quota-requests.md) <br>[Aumento de cuota de recursos de proceso](../azure-portal/supportability/per-vm-quota-requests.md) | 

## <a name="next-steps"></a>Pasos siguientes
Para obtener más información sobre los requisitos de la migración, consulte [Detalles técnicos de la migración a Azure Cloud Services (soporte extendido)](in-place-migration-technical-details.md).
