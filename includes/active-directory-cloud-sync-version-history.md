---
ms.openlocfilehash: 1beac7a75bc8ddfbdec046d381ee95b23b1575ca
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108792331"
---
En este artículo se enumeran las versiones y las características del agente de aprovisionamiento de Azure Active Directory Connect que se han publicado. El equipo de Azure AD actualiza periódicamente el agente de aprovisionamiento con nuevas características y funciones. El agente de aprovisionamiento se actualiza de forma automática cuando se publica una nueva versión. 

Microsoft proporciona soporte técnico directo para la versión más reciente del agente y una versión anterior.

## <a name="113590"></a>1.1.359.0

### <a name="new-features-and-improvements"></a>Nuevas características y mejoras
- Cmdlets de GMSA para establecer o restablecer el permiso

### <a name="fixed-issues"></a>Problemas corregidos
- Corrección de errores de permisos de carpeta de GMSA (originalmente, el problema provocó problemas de arranque)
- Corrección de errores para controlar varios cambios en un atributo de referencia de valor único (por ejemplo, administrador)
- Corrección de errores en la enumeración inicial, además del seguimiento mejorado del error
- Optimización de las actualizaciones de pertenencia a grupos a un grupo de ámbito. Con esto, los clientes ahora pueden sincronizar un grupo de hasta 50 000 miembros mediante el filtro de ámbito de grupo. 
- Compatibilidad con la recuperación de un solo objeto por DN con ámbito usado por el aprovisionamiento a petición para cumplir la lógica de ámbito





## <a name="113540"></a>1.1.354.0

20 de enero de 2021: publicado para descarga.

### <a name="new-features-and-improvements"></a>Nuevas características y mejoras
- Mejora de la experiencia de GMSA que incluye compatibilidad con cuentas de GMSA personalizadas creadas previamente
- Compatibilidad de los [cmdlets de PowerShell](../articles/active-directory/cloud-sync/how-to-gmsa-cmdlets.md) con la configuración de GMSA
- [Compatibilidad de la CLI](../articles/active-directory/cloud-sync/how-to-install-pshell.md) con la instalación del agente (instalación silenciosa)
- Diagnósticos adicionales para problemas de cuarentena del origen del agente
- Correcciones de errores que incluyen la reducción del uso de memoria de los filtros de ámbito de UO, la ejecución de PHS solo para usuarios del ámbito, el control de objetos anidados en una unidad organizativa al usar el ámbito de UO, etc. 


### <a name="fixed-issues"></a>Problemas corregidos
-    Se evita la cuarentena cuando el grupo de ámbito está fuera de ámbito
-   Si los filtros de ámbito están configurados, el trabajo de PHS ahora solo funciona para los usuarios del ámbito
-   El agente se bloquearía en algún momento durante la actualización
-   Sincronización inicial de objetos en unidades organizativas anidadas al usar el ámbito de UO
-   Repair-AADCloudSyncToolsAccount es más sólido
-   Se ha reducido el uso de memoria de los filtros de ámbito de UO
-   Se produce un error en la comprobación del rol de administrador si los miembros del rol contienen un grupo de seguridad
-   Se ha corregido el problema de permisos de la carpeta de GMSA que evita la renovación de certificados del agente







## <a name="112810"></a>1.1.281.0

### <a name="release-status"></a>Estado de la versión

23 de noviembre de 2020: publicado para descarga.

### <a name="new-features-and-improvements"></a>Nuevas características y mejoras

* Compatibilidad con [gMSA](../articles/active-directory/cloud-sync/how-to-prerequisites.md#group-managed-service-accounts).
* Compatibilidad con grupos de hasta 1500 miembros durante el ciclo de sincronización incremental o diferencial. Esto se puede aplicar cuando se usa el filtro de ámbito de grupo
* Compatibilidad con grupos grandes, de hasta 15 000 miembros
* Mejoras en la sincronización inicial
* Registro detallado avanzado
* Introducción del [módulo AADCloudSyncTools de PowerShell](../articles/active-directory/cloud-sync/reference-powershell.md)
* Limitaciones fijas para que el agente se pueda instalar en un servidor que no esté en inglés
* Compatibilidad con el filtrado PHS solo para los objetos en el ámbito (originalmente, se sincronizaban los hash de contraseña de todos los objetos)
* Se ha corregido el problema de fuga de memoria en el agente
* Registros de aprovisionamiento mejorados
* Compatibilidad con la configuración del [tiempo de espera de la conexión LDAP](../articles/active-directory/cloud-sync/how-to-manage-registry-options.md#configure-ldap-connection-timeout) 
* Compatibilidad con la configuración del [seguimiento de referencias](../articles/active-directory/cloud-sync/how-to-manage-registry-options.md#configure-referral-chasing) 


## <a name="11960"></a>1.1.96.0

### <a name="release-status"></a>Estado de la versión

4 de diciembre de 2019: publicado para descarga.

### <a name="new-features-and-improvements"></a>Nuevas características y mejoras

* Incluye compatibilidad con la [sincronización en la nube de Azure AD Connect](../articles/active-directory/cloud-sync/what-is-cloud-sync.md) para sincronizar datos de usuarios, contactos y grupos de la instancia de Active Directory local en Azure AD


## <a name="11670"></a>1.1.67.0

### <a name="release-status"></a>Estado de la versión

9 de septiembre de 2019: publicada para actualización automática

### <a name="new-features-and-improvements"></a>Nuevas características y mejoras

* Capacidad de configurar el seguimiento y el registro adicionales para depurar problemas del agente de aprovisionamiento
* Capacidad de capturar solo los atributos de Azure AD configurados en la asignación para mejorar el rendimiento de la sincronización

### <a name="fixed-issues"></a>Problemas corregidos

* Se corrigió un error que hacía que el agente dejara de responder si había problemas de conexión con Azure AD.
* Se corrigió un error que causaba problemas cuando se leían datos binarios de Azure Active Directory.
* Se corrigió un error por el que el agente no podía renovar la confianza con el servicio de identidad híbrida en la nube.

## <a name="11300"></a>1.1.30.0

### <a name="release-status"></a>Estado de la versión

23 de enero de 2019: publicado para descarga.

### <a name="new-features-and-improvements"></a>Nuevas características y mejoras

* Se ha renovado la arquitectura del conector y el agente de aprovisionamiento para mejorar el rendimiento, la estabilidad y la confiabilidad. 
* Se ha simplificado la configuración del agente de aprovisionamiento simplificado mediante el Asistente para la instalación basada en la interfaz de usuario 
* Se ha agregado la compatibilidad para las actualizaciones automáticas del agente.