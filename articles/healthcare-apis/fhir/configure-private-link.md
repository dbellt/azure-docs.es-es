---
title: Vínculo privado para Azure API for FHIR
description: En este artículo se describe cómo configurar un punto de conexión privado para los servicios Azure API for FHIR
services: healthcare-apis
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: reference
ms.date: 05/27/2021
ms.author: zxue
ms.openlocfilehash: 6b025c35c5faa64a2333710693c992a410cafecc
ms.sourcegitcommit: 1b698fb8ceb46e75c2ef9ef8fece697852c0356c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110652730"
---
# <a name="configure-private-link"></a>Configuración del vínculo privado

Private Link le permite acceder a Azure API for FHIR a través de un punto de conexión privado, que es una interfaz de red que se conecta de forma privada y segura mediante una dirección IP privada desde la red virtual. Con private link, puede acceder a nuestros servicios de forma segura desde la red virtual como un servicio propio sin tener que pasar por un sistema de nombres de dominio (DNS) público. En este artículo se describe cómo crear, probar y administrar el punto de conexión privado para Azure API for FHIR.

>[!Note]
>Ninguna de las instancias de Private Link ni de la API de Azure para FHIR se pueden migrar de un grupo de recursos a otro o de una suscripción a otra, una vez que la instancia de Private Link está habilitada. Para realizar un movimiento, elimine primero el Private Link y, a continuación, Azure API for FHIR. Cree un nuevo Private Link una vez completado el traslado. Evalúe las posibles ramificaciones de seguridad antes de eliminar la instancia de Private Link.
>
>Si la exportación de registros de auditoría y métricas está habilitada para Azure API for FHIR, actualice la configuración de exportación a través de **Configuración** de diagnóstico desde el portal.

## <a name="prerequisites"></a>Requisitos previos

Antes de crear un punto de conexión privado, hay algunos recursos de Azure que deberá crear primero:

- Grupo de recursos: el grupo de recursos de Azure que contendrá la red virtual y el punto de conexión privado.
- Azure API for FHIR: el recurso FHIR que quiere colocar detrás de un punto de conexión privado.
- Virtual Network: la red virtual a la que se conectarán los servicios de cliente y el punto de conexión privado.

Para obtener más información, [vea Private Link Documentation](../../private-link/index.yml).

## <a name="create-private-endpoint"></a>Creación de un punto de conexión privado

Para crear un punto de conexión privado, un desarrollador con permisos de control de acceso basado en rol (RBAC) en el recurso de FHIR puede usar Azure Portal, [Azure PowerShell](../../private-link/create-private-endpoint-powershell.md)o [CLI de Azure](../../private-link/create-private-endpoint-cli.md). Este artículo le guiará a través de los pasos para usar Azure Portal. Se recomienda usar Azure Portal, ya que automatiza la creación y configuración de la DNS privado zona. Para obtener más información, [vea Private Link Inicio rápido guías](../../private-link/create-private-endpoint-portal.md).

Hay dos formas de crear un punto de conexión privado. El flujo de aprobación automática permite a un usuario con permisos de RBAC en el recurso FHIR crear un punto de conexión privado sin necesidad de aprobación. El flujo de aprobación manual permite a un usuario sin permisos en el recurso FHIR solicitar un punto de conexión privado para que los propietarios del recurso FHIR lo aprueben.

> [!NOTE]
> Cuando se crea un punto de conexión privado aprobado Azure API for FHIR, el tráfico público a él se deshabilita automáticamente. 

### <a name="auto-approval"></a>Aprobación automática

Asegúrese de que la región del nuevo punto de conexión privado es la misma que la región de la red virtual. La región del recurso FHIR puede ser diferente.

![Pestaña Aspectos básicos de Azure Portal](media/private-link/private-link-portal2.png)

Para el tipo de recurso, busque y **seleccione Microsoft.HealthcareApis/services.** Para el recurso, seleccione el recurso FHIR. Para el recurso de destino, seleccione **FHIR.**

![Pestaña Recursos de Azure Portal](media/private-link/private-link-portal1.png)

Si no tiene una zona de DNS privado configurada, seleccione **(Nuevo)privatelink.azurehealthcareapis.com**. Si ya ha configurado la zona de DNS privado, puede seleccionarla en la lista. Debe tener el formato de **privatelink.azurehealthcareapis.com**.

![Pestaña Configuración de Azure Portal](media/private-link/private-link-portal3.png)

Una vez completada la implementación, puede volver a **la** pestaña Conexiones de punto de conexión privado de la que observará **Aprobado** como estado de conexión.

### <a name="manual-approval"></a>Aprobación manual

Para la aprobación manual, seleccione la segunda opción en Recurso, "Conéctese a un recurso de Azure por identificador de recurso o alias". En Subrecurso de destino, escriba "fhir" en Aprobación automática.

![Aprobación manual](media/private-link/private-link-manual.png)

Una vez que se ha completado la implementación, puede volver a la pestaña "Conexiones de punto de conexión privado", en la que puede Aprobar, Rechazar o Quitar la conexión.

![Opciones](media/private-link/private-link-options.png)

## <a name="test-private-endpoint"></a>Prueba del punto de conexión privado

Para asegurarse de que el servidor de FHIR no recibe tráfico público después de deshabilitar el acceso a la red pública, seleccione el punto de conexión /metadata para el servidor desde el equipo. Debería recibir un mensaje 403 Prohibido. 


> [!NOTE]
> Puede tardar hasta 5 minutos después de actualizar la marca de acceso a la red pública antes de que se bloquee el tráfico público.

Para asegurarse de que el punto de conexión privado puede enviar tráfico al servidor:

1. Cree una máquina virtual (VM) que esté conectada a la red virtual y a la subred en la que está configurado el punto de conexión privado. Para asegurarse de que el tráfico de la máquina virtual solo usa la red privada, deshabilite el tráfico saliente de Internet mediante la regla del grupo de seguridad de red (NSG).
2. Use RDP en la máquina virtual.
3. Acceda al punto de conexión /metadata del servidor de FHIR desde la máquina virtual. Debe recibir la instrucción de funcionalidad como respuesta.

## <a name="manage-private-endpoint"></a>Administración de un punto de conexión privado

### <a name="view"></a>Ver

Los puntos de conexión privados y el controlador de interfaz de red (NIC) asociado son visibles en Azure Portal del grupo de recursos en el que se crearon.

![Visualización de recursos](media/private-link/private-link-view.png)

### <a name="delete"></a>Eliminar

Los puntos de conexión privados solo se pueden  eliminar del Azure Portal  de la hoja Información general o seleccionando la opción Quitar en la pestaña Redes Conexiones de punto de **conexión** privado. Al seleccionar **Quitar,** se eliminarán el punto de conexión privado y la NIC asociada. Si elimina todos los puntos de conexión privados al recurso de FHIR y a la red pública, el acceso está deshabilitado y no se realizará ninguna solicitud al servidor de FHIR.

![Eliminación del punto de conexión privado](media/private-link/private-link-delete.png)
