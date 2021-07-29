---
title: 'Guía para la migración de MySQL en el entorno local a Azure Database for MySQL: seguridad'
description: Cambiarse a un servicio basado en la nube no significa que todo el mundo en Internet tenga acceso siempre a él.
ms.service: mysql
ms.subservice: migration-guide
ms.topic: how-to
author: arunkumarthiags
ms.author: arthiaga
ms.reviewer: maghan
ms.custom: ''
ms.date: 06/11/2021
ms.openlocfilehash: 1d48bce85e7ab618b510165db9347592ba345b87
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112083015"
---
# <a name="mysql-on-premises-to-azure-database-for-mysql-migration-guide-security"></a>Guía para la migración de MySQL en el entorno local a Azure Database for MySQL: seguridad

## <a name="prerequisites"></a>Prerrequisitos

[Continuidad empresarial y recuperación ante desastres](12-business-continuity-and-disaster-recovery.md)

## <a name="overview"></a>Información general

Cambiarse a un servicio basado en la nube no significa que todo el mundo en Internet tenga acceso siempre a él. Azure proporciona una excelente seguridad que garantiza que las cargas de trabajo de datos estén protegidas continuamente de actores no deseados y programas fuera de servicio.

## <a name="authentication"></a>Authentication

Azure Database for MySQL admite los mecanismos de autenticación básicos para la conectividad de usuarios de MySQL, pero también admite la [integración con Azure Active Directory](../../concepts-azure-ad-authentication.md). Esta integración de seguridad funciona mediante la emisión de tokens que actúan como contraseñas durante el proceso de inicio de sesión de MySQL. La [configuración de la integración de Active Directory](../../howto-configure-sign-in-azure-ad-authentication.md) es increíblemente fácil de establecer y admite los grupos de AAD, además de los usuarios.

Esta estrecha integración permite a los administradores y las aplicaciones aprovechar las características de seguridad mejoradas de [Azure Identity Protection](/azure/active-directory/identity-protection/overview-identity-protection) para detectar cualquier problema de identidad.

> [!NOTE] 
> MySQL 5.7 y las versiones posteriores admiten esta característica de seguridad. La mayoría de los [controladores de aplicaciones](../../howto-configure-sign-in-azure-ad-authentication.md) se admiten siempre que se proporcione la opción `clear-text`.

## <a name="threat-protection"></a>Protección contra amenazas

Si las credenciales de usuario o aplicación están en peligro, es probable que los registros no reflejen los intentos de inicio de sesión con errores. Las credenciales en peligro pueden permitir que los actores no autorizados accedan a los datos y los descarguen. La [Protección contra amenazas de Microsoft](/azure/mysql/concepts-data-access-and-security-threat-protection) puede observar si hay anomalías en los inicios de sesión (como ubicaciones inusuales, usuarios poco frecuentes o ataques por fuerza bruta) y otras actividades sospechosas. Se puede enviar notificaciones a los administradores en caso de que algo no tenga un buen aspecto (`look`).

## <a name="audit-logging"></a>Registro de auditoría

MySQL tiene una sólida característica de registro de auditoría integrada. De forma predeterminada, esta [característica de registro de auditoría está deshabilitada](../../concepts-audit-logs.md) en Azure Database for MySQL. El registro a nivel de servidor se puede habilitar al cambiar el parámetro de servidor `audit\_log\_enabled`. Una vez habilitados, se puede acceder a los registros de [Azure Monitor](../../../azure-monitor/overview.md) y [Log Analytics](../../../azure-monitor/logs/design-logs-deployment.md) si se activa el [registro de diagnóstico](../../howto-configure-audit-logs-portal.md#set-up-diagnostic-logs).

Para consultar eventos relacionados con la conexión del usuario, ejecute la siguiente consulta KQL:

```
AzureDiagnostics  
| where ResourceProvider =="MICROSOFT.DBFORMYSQL" 
| where Category == 'MySqlAuditLogs' and event\_class\_s == "connection\_log"  
| project TimeGenerated, LogicalServerName\_s, event\_class\_s, event\_subclass\_s  
, event\_time\_t, user\_s , ip\_s , sql\_text\_s  
| order by TimeGenerated asc
```

## <a name="encryption"></a>Cifrado

Los datos de la instancia de MySQL se cifran en reposo de forma predeterminada. Las copias de seguridad automatizadas también se cifran para evitar posibles filtraciones de datos a partes no autorizadas. Normalmente, este cifrado se realiza con una clave que se crea cuando se crea la instancia. Además de esta clave de cifrado predeterminada, los administradores tienen la opción de contar con el servicio [Bring Your Own Key (BYOK)](../../concepts-data-encryption-mysql.md).

Cuando se usa una estrategia de clave administrada por el cliente, es fundamental comprender las responsabilidades en torno a la administración del ciclo de vida de las claves. Las claves de cliente se almacenan en una instancia de [Azure Key Vault](/azure/key-vault/general/basic-concepts) y, a continuación, se accede a ellas a través de directivas. Es fundamental seguir todas las recomendaciones para la administración de claves, ya que la pérdida de la clave de cifrado equivale a la pérdida de acceso a los datos.

Además de una clave administrada por el cliente, use claves de nivel de servicio para [agregar el cifrado doble.](/azure/mysql/concepts-infrastructure-double-encryption) La implementación de esta característica puede proporcionar datos altamente cifrados en reposo, pero incluye penalizaciones en el rendimiento del cifrado. Se deben realizar pruebas.

Los datos se pueden cifrar durante el tránsito mediante SSL/TLS. Como se ha comentado anteriormente, puede ser necesario [modificar las aplicaciones](../../howto-configure-ssl.md) para admitir este cambio y también configurar las opciones de validación de TLS adecuadas.

## <a name="firewall"></a>Firewall

Una vez que los usuarios están configurados y los datos se cifran en reposo, el equipo de migración debe revisar los flujos de datos de red. Azure Database for MySQL proporciona varios mecanismos para proteger las capas de la red. Para ello, limita el acceso solo a los usuarios, las aplicaciones y los dispositivos autorizados.

La primera línea de defensa para proteger la instancia de MySQL es implementar [reglas de firewall](../../concepts-firewall-rules.md). Las direcciones IP solo se pueden limitar a ubicaciones válidas al acceder a la instancia a través de direcciones IP internas o externas. Si la instancia de MySQL está destinada solo a atender aplicaciones internas, [restrinja el acceso público](../../howto-deny-public-network-access.md).

Al mover una aplicación a Azure junto con la carga de trabajo de MySQL, es probable que haya varias redes virtuales configuradas en un patrón de tipo hub-and-spoke que requiere que se configure el [emparejamiento de red virtual](/azure/virtual-network/virtual-network-peering-overview).

## <a name="private-link"></a>Private Link

Para limitar el acceso de Azure Database for MySQL a los recursos internos de Azure, habilite [Private Link](/azure/mysql/concepts-data-access-security-private-link). Private Link garantiza que se asigne una dirección IP privada a la instancia de MySQL en lugar de una dirección IP pública.

> [!NOTE]
> Hay muchas otras [consideraciones de Redes de Azure básicas](../../concepts-data-access-and-security-vnet.md) que deben tenerse en cuenta y que no se tratan en esta guía.

Revise un conjunto de posibles tareas de [base de referencia de seguridad](/azure/mysql/security-baseline) que se pueden implementar en todos los recursos de Azure. No todos los elementos descritos en el vínculo de referencia se aplican a las cargas de trabajo de datos específicas o a los recursos de Azure.

## <a name="security-checklist"></a>Lista de comprobación de seguridad

  - Use la Autenticación de Azure AD cuando sea posible.

  - Habilite la Protección contra amenazas avanzada.

  - Habilite todas las características de auditoría.

  - Considere la posibilidad de usar una estrategia Bring-Your-Own-Key (BYOK).

  - Implemente reglas de firewall.

  - Use puntos de conexión privados para cargas de trabajo que no se distribuyan por Internet.  


> [!div class="nextstepaction"]
> [Resumen](./14-summary.md)