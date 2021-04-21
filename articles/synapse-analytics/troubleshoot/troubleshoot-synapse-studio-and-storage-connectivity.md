---
title: Solución de problemas de conectividad entre Synapse Studio y el almacenamiento
description: Solución de problemas de conectividad entre Synapse Studio y el almacenamiento
author: saveenr
ms.service: synapse-analytics
ms.subservice: workspace
ms.topic: conceptual
ms.date: 11/11/2020
ms.author: xujiang1
ms.reviewer: jrasnick
ms.openlocfilehash: d5f79131608315f7e1c05cbfc0117300eea6c511
ms.sourcegitcommit: 590f14d35e831a2dbb803fc12ebbd3ed2046abff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107566280"
---
# <a name="troubleshoot-connectivity-between-azure-synapse-analytics-synapse-studio-and-storage"></a>Solución de problemas de conectividad entre Azure Synapse Analytics Synapse Studio y el almacenamiento

En Synapse Studio puede explorar los recursos de datos ubicados en el almacenamiento vinculado. Esta guía le ayudará a solucionar los problemas de conectividad que surjan al intentar acceder a los recursos de datos. 

## <a name="case-1-storage-account-lacks-proper-permissions"></a>Caso 1: faltan los permisos adecuados en la cuenta de almacenamiento

Si la cuenta de almacenamiento carece de los permisos adecuados, no podrá expandir la estructura de almacenamiento a través de "Datos" --> "Vinculados" en Synapse Studio. Vea la captura de pantalla del síntoma del problema. 

El mensaje de error detallado puede variar, pero su significado general es: "Esta solicitud no está autorizado para realizar esta operación."

En el nodo almacenamiento vinculado:  
![Problema de conectividad de Storage 1](media/troubleshoot-synapse-studio-and-storage-connectivity/storage-connectivity-issue-1.png)

En el nodo del contenedor de almacenamiento:  
![Problema de conectividad de Storage 1a](media/troubleshoot-synapse-studio-and-storage-connectivity/storage-connectivity-issue-1a.png)

**SOLUCIÓN**: Para asignar el rol adecuado a una cuenta, consulte [Uso de Azure Portal para asignar un rol de Azure para el acceso a datos de blobs y colas](../../storage/common/storage-auth-aad-rbac-portal.md).


## <a name="case-2-failed-to-send-the-request-to-storage-server"></a>Caso 2: no se pudo enviar la solicitud al servidor de almacenamiento

Al seleccionar la flecha para expandir la estructura de almacenamiento en "Datos" --> "Vinculado" en Synapse Studio, es posible que vea el problema "REQUEST_SEND_ERROR" en el panel izquierdo. Vea la captura de pantalla del síntoma del problema:

En el nodo almacenamiento vinculado:  
![Problema de conectividad de Storage 2](media/troubleshoot-synapse-studio-and-storage-connectivity/storage-connectivity-issue-2.png)

En el nodo del contenedor de almacenamiento:  
![Problema de conectividad de Storage 2a](media/troubleshoot-synapse-studio-and-storage-connectivity/storage-connectivity-issue-2a.png)

Tras este problema puede haber varias razones posibles:

### <a name="the-storage-resource-is-behind-a-vnet-and-a-storage-private-endpoint-needs-to-configure"></a>El recurso de almacenamiento está detrás de una red virtual y es preciso configurar un punto de conexión privado de almacenamiento

**SOLUCIÓN**: en este caso, debe configurar el punto de conexión privado de almacenamiento para la cuenta de almacenamiento. Para aprender a configurar el punto de conexión privado de almacenamiento para una red virtual, consulte [Uso de Azure Portal para asignar un rol de Azure para el acceso a datos de blobs y colas](../security/how-to-connect-to-workspace-from-restricted-network.md).

Puede usar el comando "nslookup \<storage-account-name\>.dfs.core.windows.net" para comprobar la conectividad después de configurar el punto de conexión privado de almacenamiento. Debe devolver una cadena similar a: "\<storage-account-name\>.privatelink.dfs.core.windows.net".

### <a name="the-storage-resource-is-not-behind-a-vnet-but-the-blob-service-azure-ad-endpoint-is-not-accessible-due-to-firewall-configured"></a>El recurso de almacenamiento no está detrás de una red virtual, pero no se puede acceder al punto de conexión de Blob service (Azure AD) debido al firewall configurado

**SOLUCIÓN**: en este caso, debe abrir la cuenta de almacenamiento en Azure Portal. En el panel de navegación izquierdo, desplácese hacia abajo hasta **Soporte técnico y solución de problemas** y seleccione **Comprobación de conectividad** para comprobar el estado de conectividad de **Blob service (Azure AD)** . Si no puede acceder a él, siga la guía para comprobar la configuración de **Firewalls y redes virtuales** en su página de la cuenta de almacenamiento. Para más información sobre los firewalls de almacenamiento, consulte [Configuración de redes virtuales y firewalls de Azure Storage](../../storage/common/storage-network-security.md).

### <a name="other-issues-to-check"></a>Otros problemas que se deben comprobar primero son 

* El recurso de almacenamiento al que accede es Azure Data Lake Storage Gen2 y está detrás de un firewall y una red virtual (con el punto de conexión privado de almacenamiento configurado) al mismo tiempo.
* El recurso de contenedor al que accede se ha eliminado o no existe.
* Entre inquilinos: el inquilino de área de trabajo con el que el usuario inició sesión no coincide con el inquilino de la cuenta de almacenamiento. 


## <a name="next-steps"></a>Pasos siguientes
Si los pasos anteriores no ayudan a resolver el problema, [cree una incidencia de soporte técnico](../sql-data-warehouse/sql-data-warehouse-get-started-create-support-ticket.md).