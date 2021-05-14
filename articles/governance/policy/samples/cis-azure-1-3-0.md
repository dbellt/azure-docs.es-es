---
title: Detalles del cumplimiento normativo de CIS Microsoft Azure Foundations Benchmark 1.3.0
description: Detalles de la iniciativa integrada de cumplimiento normativo de CIS Microsoft Azure Foundations Benchmark 1.3.0. Cada control se corresponde a una o varias definiciones de Azure Policy que ayudan en la evaluación.
ms.date: 05/04/2021
ms.topic: sample
ms.custom: generated
ms.openlocfilehash: 065a768fb3b040991cc7257b651f9f1e93268080
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108761082"
---
# <a name="details-of-the-cis-microsoft-azure-foundations-benchmark-130-regulatory-compliance-built-in-initiative"></a>Detalles de la iniciativa integrada de cumplimiento normativo de CIS Microsoft Azure Foundations Benchmark 1.3.0.

En el siguiente artículo se detalla la correspondencia entre los **dominios de cumplimiento** y los **controles** de la definición de la iniciativa integrada del cumplimiento normativo de Azure Policy en CIS Microsoft Azure Foundations Benchmark 1.3.0.
Para más información sobre este estándar de cumplimiento, consulte [CIS Microsoft Azure Foundations Benchmark 1.3.0](https://www.cisecurity.org/benchmark/azure/). Para entender el concepto de _propiedad_, consulte [Definición de directivas de Azure Policy](../concepts/definition-structure.md#type) y [Responsabilidad compartida en la nube](../../../security/fundamentals/shared-responsibility.md).

Las siguientes asignaciones son para los controles de **CIS Microsoft Azure Foundations Benchmark 1.3.0**. Use el panel de navegación de la derecha para ir directamente a un **dominio de cumplimiento** específico. Muchos de los controles se implementan con una definición de iniciativa de [Azure Policy](../overview.md). Para revisar la definición de iniciativa completa, abra **Policy** en Azure Portal y seleccione la página **Definiciones**.
Busque y seleccione la definición de la iniciativa integrada de cumplimiento normativo de **CIS Microsoft Azure Foundations Benchmark 3.0**.

> [!IMPORTANT]
> Cada control que se muestra a continuación está asociado a una o varias definiciones de [Azure Policy](../overview.md).
> Estas directivas pueden ayudarle a [evaluar el cumplimiento](../how-to/get-compliance-data.md) mediante el control. Sin embargo, con frecuencia no hay una correspondencia completa o exacta entre un control y una o varias directivas. Como tal, el **cumplimiento** en Azure Policy solo se refiere a las propias definiciones de directiva; esto no garantiza que se cumpla totalmente con todos los requisitos de un control. Además, el estándar de cumplimiento incluye controles que no se abordan con las definiciones de Azure Policy en este momento. Por lo tanto, el cumplimiento en Azure Policy es solo una vista parcial del estado general de cumplimiento. Las asociaciones entre los dominios de cumplimiento, los controles y las definiciones de Azure Policy para este estándar de cumplimiento pueden cambiar con el tiempo. Para ver el historial de cambios, consulte el [historial de confirmación de GitHub](https://github.com/Azure/azure-policy/commits/master/built-in-policies/policySetDefinitions/Regulatory%20Compliance/CISv1_3_0.json).

## <a name="identity-and-access-management"></a>Administración de identidades y acceso

### <a name="ensure-that-multi-factor-authentication-is-enabled-for-all-privileged-users"></a>Asegúrese de que la autenticación multifactor esté habilitada para todos los usuarios con privilegios.

**Identificador**: CIS Azure 1.1 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[MFA debe estar habilitado en las cuentas con permisos de escritura de la suscripción.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9297c21d-2ed6-4474-b48f-163f75654ce3) |Multi-Factor Authentication (MFA) debe estar habilitada para todas las cuentas de la suscripción que tengan permisos de escritura, a fin de evitar una brecha de seguridad en las cuentas o los recursos. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableMFAForWritePermissions_Audit.json) |
|[MFA debe estar habilitada en las cuentas con permisos de propietario en la suscripción](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Faa633080-8b72-40c4-a2d7-d00c03e80bed) |Multi-Factor Authentication (MFA) debe estar habilitada para todas las cuentas de la suscripción que tengan permisos de propietario, a fin de evitar una brecha de seguridad en las cuentas o los recursos. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableMFAForOwnerPermissions_Audit.json) |

### <a name="ensure-that-multi-factor-authentication-is-enabled-for-all-non-privileged-users"></a>Asegúrese de que la autenticación multifactor esté habilitada para todos los usuarios sin privilegios.

**Identificador**: CIS Azure 1.2 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[MFA debe estar habilitada en las cuentas con permisos de lectura en la suscripción](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe3576e28-8b17-4677-84c3-db2990658d64) |Multi-Factor Authentication (MFA) debe estar habilitada para todas las cuentas de la suscripción que tengan permisos de lectura, a fin de evitar una brecha de seguridad en las cuentas o los recursos. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableMFAForReadPermissions_Audit.json) |

### <a name="ensure-guest-users-are-reviewed-on-a-monthly-basis"></a>Asegúrese de que los usuarios invitados se revisen mensualmente.

**Identificador**: CIS Azure 1.3 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Las cuentas externas con permisos de propietario deben quitarse de la suscripción](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff8456c1c-aa66-4dfb-861a-25d127b775c9) |Las cuentas externas con permisos de propietario deben quitarse de la suscripción a fin de evitar el acceso no supervisado. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_RemoveExternalAccountsWithOwnerPermissions_Audit.json) |
|[Las cuentas externas con permisos de lectura deben quitarse de la suscripción](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5f76cf89-fbf2-47fd-a3f4-b891fa780b60) |Las cuentas externas con privilegios de lectura deben quitarse de la suscripción a fin de evitar el acceso no supervisado. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_RemoveExternalAccountsReadPermissions_Audit.json) |
|[Las cuentas externas con permisos de escritura deben quitarse de la suscripción](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5c607a2e-c700-4744-8254-d77e7c9eb5e4) |Las cuentas externas con privilegios de escritura deben quitarse de la suscripción a fin de evitar el acceso no supervisado. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_RemoveExternalAccountsWritePermissions_Audit.json) |

### <a name="ensure-that-no-custom-subscription-owner-roles-are-created"></a>Asegúrese de que no existe ningún rol de propietario de suscripción personalizado.

**Identificador**: CIS Azure 1.21 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[No deben existir roles de propietario de suscripción personalizados](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F10ee2ea2-fb4d-45b8-a7e9-a2e770044cd9). |Esta directiva garantiza que no existe ningún rol de propietario de suscripción personalizado. |Audit, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/General/CustomSubscription_OwnerRole_Audit.json) |

## <a name="security-center"></a>Security Center

### <a name="ensure-that-azure-defender-is-set-to-on-for-servers"></a>Asegúrese de que Azure Defender esté establecido en Activado para Servidores.

**Identificador**: CIS Azure 2.1 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Se debe habilitar Azure Defender para servidores](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4da35fc9-c9e7-4960-aec9-797fe7d9051d) |Azure Defender para servidores proporciona protección en tiempo real contra amenazas para las cargas de trabajo del servidor y genera recomendaciones de protección, así como alertas sobre la actividad sospechosa. |AuditIfNotExists, Disabled |[1.0.3](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableAdvancedThreatProtectionOnVM_Audit.json) |

### <a name="ensure-that-azure-defender-is-set-to-on-for-app-service"></a>Asegúrese de que Azure Defender esté establecido en Activado para App Service.

**Identificador**: CIS Azure 2.2 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Se debe habilitar Azure Defender para App Service](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2913021d-f2fd-4f3d-b958-22354e2bdbcb) |Azure Defender para App Service aprovecha la escalabilidad de la nube, y la visibilidad que ofrece Azure como proveedor de servicios en la nube, para supervisar si se producen ataques comunes a aplicaciones web. |AuditIfNotExists, Disabled |[1.0.3](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableAdvancedThreatProtectionOnAppServices_Audit.json) |

### <a name="ensure-that-azure-defender-is-set-to-on-for-azure-sql-database-servers"></a>Asegúrese de que Azure Defender esté establecido en Activado para servidores de bases de datos de Azure SQL.

**Identificador**: CIS Azure 2.3 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Se debe habilitar Azure Defender para servidores de Azure SQL Database](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7fe3b40f-802b-4cdd-8bd4-fd799c948cc2) |Azure Defender para SQL proporciona funcionalidad para mostrar y mitigar posibles vulnerabilidades de base de datos, detectar actividades anómalas que podrían indicar amenazas para bases de datos SQL, y detectar y clasificar datos confidenciales. |AuditIfNotExists, Disabled |[1.0.2](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableAdvancedDataSecurityOnSqlServers_Audit.json) |

### <a name="ensure-that-azure-defender-is-set-to-on-for-sql-servers-on-machines"></a>Asegúrese de que Azure Defender esté establecido en Activado para servidores SQL Server en máquinas.

**Identificador**: CIS Azure 2.4 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Se debe habilitar Azure Defender para servidores SQL Server en las máquinas](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6581d072-105e-4418-827f-bd446d56421b) |Azure Defender para SQL proporciona funcionalidad para mostrar y mitigar posibles vulnerabilidades de base de datos, detectar actividades anómalas que podrían indicar amenazas para bases de datos SQL, y detectar y clasificar datos confidenciales. |AuditIfNotExists, Disabled |[1.0.2](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableAdvancedDataSecurityOnSqlServerVirtualMachines_Audit.json) |

### <a name="ensure-that-azure-defender-is-set-to-on-for-storage"></a>Asegúrese de que Azure Defender esté establecido en Activado para Storage.

**Identificador**: CIS Azure 2.5 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Se debe habilitar Azure Defender para Storage](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F308fbb08-4ab8-4e67-9b29-592e93fb94fa) |Azure Defender para Storage detecta intentos inusuales y potencialmente perjudiciales de acceder a las cuentas de almacenamiento o de vulnerarlas. |AuditIfNotExists, Disabled |[1.0.3](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableAdvancedThreatProtectionOnStorageAccounts_Audit.json) |

### <a name="ensure-that-azure-defender-is-set-to-on-for-kubernetes"></a>Asegúrese de que Azure Defender esté establecido en Activado para Kubernetes.

**Identificador**: CIS Azure 2.6 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Se debe habilitar Azure Defender para Kubernetes](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F523b5cd1-3e23-492f-a539-13118b6d1e3a) |Azure Defender para Kubernetes proporciona protección en tiempo real contra amenazas para entornos en contenedores y genera alertas en caso de actividad sospechosa. |AuditIfNotExists, Disabled |[1.0.3](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableAdvancedThreatProtectionOnKubernetesService_Audit.json) |

### <a name="ensure-that-azure-defender-is-set-to-on-for-container-registries"></a>Asegúrese de que Azure Defender esté establecido en Activado para registros de contenedor.

**Identificador**: CIS Azure 2.7 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Se debe habilitar Azure Defender para registros de contenedor](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc25d9a16-bc35-4e15-a7e5-9db606bf9ed4) |Azure Defender para registros de contenedor proporciona análisis de vulnerabilidades de las imágenes extraídas en los últimos 30 días, insertadas en el registro o importadas, y expone los hallazgos detallados por imagen. |AuditIfNotExists, Disabled |[1.0.3](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableAdvancedThreatProtectionOnContainerRegistry_Audit.json) |

### <a name="ensure-that-azure-defender-is-set-to-on-for-key-vault"></a>Asegúrese de que Azure Defender esté establecido en Activado para Key Vault.

**Identificador**: CIS Azure 2.8 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Se debe habilitar Azure Defender para Key Vault](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0e6763cc-5078-4e64-889d-ff4d9a839047) |Azure Defender para Key Vault proporciona un nivel de protección adicional de inteligencia de seguridad, ya que detecta intentos inusuales y potencialmente dañinos de obtener acceso a las cuentas de Key Vault o aprovechar sus vulnerabilidades de seguridad. |AuditIfNotExists, Disabled |[1.0.3](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableAdvancedThreatProtectionOnKeyVaults_Audit.json) |

### <a name="ensure-that-automatic-provisioning-of-monitoring-agent-is-set-to-on"></a>Asegúrese de que "Aprovisionamiento automático del agente de supervisión" esté establecido en "Activado".

**Identificador**: CIS Azure 2.11 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[El aprovisionamiento automático del agente de Log Analytics debe estar habilitado en la suscripción](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F475aae12-b88a-4572-8b36-9b712b2b3a17) |A fin de supervisar las amenazas y vulnerabilidades de seguridad, Azure Security Center recopila datos de las máquinas virtuales de Azure. El agente de Log Analytics, anteriormente conocido como Microsoft Monitoring Agent (MMA), recopila los datos al leer distintas configuraciones relacionadas con la seguridad y distintos registros de eventos de la máquina y copiar los datos en el área de trabajo de Log Analytics para analizarlos. Se recomienda habilitar el aprovisionamiento automático para implementar automáticamente el agente en todas las máquinas virtuales de Azure admitidas y en las nuevas que se creen. |AuditIfNotExists, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_Automatic_provisioning_log_analytics_monitoring_agent.json) |

### <a name="ensure-additional-email-addresses-is-configured-with-a-security-contact-email"></a>Asegúrese de que "Additional email addresses" (Direcciones de correo electrónico adicionales) esté configurado con una dirección de correo electrónico de contacto de seguridad.

**Identificador**: CIS Azure 2.13 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Las suscripciones deben tener una dirección de correo electrónico de contacto para los problemas de seguridad](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4f4f78b8-e367-4b10-a341-d9a4ad5cf1c7) |Para asegurarse de que las personas pertinentes de la organización reciban una notificación cuando se produzca una vulneración de seguridad potencial en una de las suscripciones, establezca un contacto de seguridad para la recepción de notificaciones por correo electrónico de Security Center. |AuditIfNotExists, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_Security_contact_email.json) |

### <a name="ensure-that-notify-about-alerts-with-the-following-severity-is-set-to-high"></a>Asegúrese de que la opción "Notify about alerts with the following severity" (Notificar alertas con la gravedad siguiente) esté establecida en "Alto".

**Identificador**: CIS Azure 2.14 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[La opción para enviar notificaciones por correo electrónico para alertas de gravedad alta debe estar habilitada.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6e2593d9-add6-4083-9c9b-4b7d2188c899) |Para asegurarse de que las personas pertinentes de la organización reciban una notificación cuando se produzca una vulneración de seguridad potencial en una de las suscripciones, habilite las notificaciones por correo electrónico de alertas de gravedad alta en Security Center. |AuditIfNotExists, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_Email_notification.json) |

## <a name="storage-accounts"></a>Cuentas de almacenamiento

### <a name="ensure-that-secure-transfer-required-is-set-to-enabled"></a>Asegúrese de la opción "Se requiere transferencia segura" esté establecida en "Habilitada".

**Identificador**: CIS Azure 3.1 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Se debe habilitar la transferencia segura a las cuentas de almacenamiento](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F404c3081-a854-4457-ae30-26a93ef643f9) |Permite auditar el requisito de transferencia segura en la cuenta de almacenamiento. La transferencia segura es una opción que obliga a la cuenta de almacenamiento a aceptar solamente solicitudes de conexiones seguras (HTTPS). El uso de HTTPS garantiza la autenticación entre el servidor y el servicio, y protege los datos en tránsito de ataques de nivel de red, como los de tipo "Man in the middle", interceptación y secuestro de sesión |Audit, Deny, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/Storage_AuditForHTTPSEnabled_Audit.json) |

### <a name="ensure-that-public-access-level-is-set-to-private-for-blob-containers"></a>Asegúrese de que "Nivel de acceso público" se haya establecido en Privado para los contenedores de blobs

**Identificador**: CIS Azure 3.5 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[No se debe permitir el acceso público a la cuenta de almacenamiento](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4fa4b6c0-31ca-4c0d-b10d-24b96f62a751) |El acceso de lectura público anónimo a contenedores y blobs de Azure Storage es una manera cómoda de compartir datos, pero también puede plantear riesgos para la seguridad. Para evitar las infracciones de datos producidas por el acceso anónimo no deseado, Microsoft recomienda impedir el acceso público a una cuenta de almacenamiento a menos que su escenario lo requiera. |audit, deny, disabled |[2.0.1-preview](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/ASC_Storage_DisallowPublicBlobAccess_Audit.json) |

### <a name="ensure-default-network-access-rule-for-storage-accounts-is-set-to-deny"></a>Asegúrese de que la regla de acceso de red predeterminada para las cuentas de almacenamiento esté configurada para denegar.

**Identificador**: CIS Azure 3.6 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Se debe restringir el acceso de red a las cuentas de almacenamiento](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F34c877ad-507e-4c82-993e-3452a6e0ad3c) |El acceso de red a las cuentas de almacenamiento debe estar restringido. Configure reglas de red, solo las aplicaciones de redes permitidas pueden acceder a la cuenta de almacenamiento. Para permitir conexiones desde clientes específicos locales o de Internet, se puede conceder acceso al tráfico procedente de redes virtuales de Azure específicas o a intervalos de direcciones IP de Internet públicas. |Audit, Deny, Disabled |[1.1.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/Storage_NetworkAcls_Audit.json) |
|[Las cuentas de almacenamiento deben restringir el acceso a la red mediante el uso de reglas de red virtual](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2a1a9cdf-e04d-429a-8416-3bfb72a1b26f) |Proteja las cuentas de almacenamiento frente a amenazas potenciales mediante reglas de red virtual como método preferente en lugar de filtrado basado en IP. La deshabilitación del filtrado basado en IP evita que las direcciones IP públicas accedan a las cuentas de almacenamiento. |Audit, Deny, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/StorageAccountOnlyVnetRulesEnabled_Audit.json) |

### <a name="ensure-trusted-microsoft-services-is-enabled-for-storage-account-access"></a>Asegúrese de que "Servicios de Microsoft de confianza" está habilitado para el acceso a la cuenta de almacenamiento.

**Identificador**: CIS Azure 3.7 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Las cuentas de almacenamiento deben permitir el acceso desde los servicios de Microsoft de confianza](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc9d007d0-c057-4772-b18c-01e546713bcd) |Algunos servicios de Microsoft que interactúan con las cuentas de almacenamiento funcionan desde redes a las que no se puede conceder acceso a través de reglas de red. Para ayudar a que este tipo de servicio funcione según lo previsto, permita que el conjunto de servicios de Microsoft de confianza omita las reglas de red. Estos servicios usarán luego una autenticación sólida para acceder a la cuenta de almacenamiento. |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/StorageAccess_TrustedMicrosoftServices_Audit.json) |

### <a name="ensure-storage-for-critical-data-are-encrypted-with-customer-managed-key"></a>Asegúrese de que el almacenamiento de datos críticos esté cifrado con la clave administrada por el cliente.

**Identificador**: CIS Azure 3.9 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Las cuentas de almacenamiento deben utilizar una clave administrada por el cliente para el cifrado](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6fac406b-40ca-413b-bf8e-0bf964659c25). |Proteja su cuenta de almacenamiento con mayor flexibilidad mediante el uso de claves administradas por el cliente. Cuando se especifica una clave administrada por el cliente, esa clave se usa para proteger y controlar el acceso a la clave que cifra los datos. El uso de claves administradas por el cliente proporciona funcionalidades adicionales para controlar la rotación de la clave de cifrado de claves o para borrar datos mediante criptografía. |Audit, Disabled |[1.0.2](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/StorageAccountCustomerManagedKeyEnabled_Audit.json) |

## <a name="database-services"></a>Servicios de base de datos

### <a name="ensure-that-auditing-is-set-to-on"></a>Asegúrese de que la opción "Auditando" esté establecida en "Activada".

**Identificador**: CIS Azure 4.1.1 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[La auditoría de SQL Server debe estar habilitada](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa6fb4358-5bf4-4ad7-ba82-2cd2f41ce5e9) |La auditoría debe estar habilitada en SQL Server para realizar un seguimiento de las actividades de todas las bases de datos del servidor y guardarlas en un registro de auditoría. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlServerAuditing_Audit.json) |

### <a name="ensure-that-data-encryption-is-set-to-on-on-a-sql-database"></a>Asegúrese de que la opción "Cifrado de datos" esté establecida en "Activado" en una base de datos SQL Database.

**Identificador**: CIS Azure 4.1.2 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[El cifrado de datos transparente en bases de datos SQL debe estar habilitado](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F17k78e20-9358-41c9-923c-fb736d382a12) |El cifrado de datos transparente debe estar habilitado para proteger los datos en reposo y satisfacer los requisitos de cumplimiento. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlDBEncryption_Audit.json) |

### <a name="ensure-that-auditing-retention-is-greater-than-90-days"></a>Asegúrese de que la retención de "Auditoría" sea "más de 90 días".

**Identificador**: CIS Azure 4.1.3 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Los servidores SQL Server con auditoría en el destino de la cuenta de almacenamiento se deben configurar con una retención de 90 días o superior.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F89099bee-89e0-4b26-a5f4-165451757743) |Con fines de investigación de incidentes, se recomienda establecer la retención de datos de auditoría de las instancias de SQL Server en el destino de la cuenta de almacenamiento en al menos 90 días. Confirme que cumple las reglas de retención necesarias para las regiones en las que trabaja. A veces, es necesario para cumplir con los estándares normativos. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlServerAuditingRetentionDays_Audit.json) |

### <a name="ensure-that-advanced-threat-protection-atp-on-a-sql-server-is-set-to-enabled"></a>Asegúrese de que la protección contra amenazas avanzada (ATP) en un servidor SQL Server esté establecida en "Habilitado".

**Identificador**: CIS Azure 4.2.1 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Se debe habilitar Advanced Data Security en la Instancia administrada de SQL](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fabfb7388-5bf4-4ad7-ba99-2cd2f41cebb9) |Permite auditr cada servicio SQL Managed Instance sin Advanced Data Security. |AuditIfNotExists, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlManagedInstance_AdvancedDataSecurity_Audit.json) |
|[La opción Advanced Data Security debe estar habilitada en los servidores SQL Server](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fabfb4388-5bf4-4ad7-ba82-2cd2f41ceae9) |Auditoría de los servidores de SQL sin Advanced Data Security |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlServer_AdvancedDataSecurity_Audit.json) |

### <a name="ensure-that-vulnerability-assessment-va-is-enabled-on-a-sql-server-by-setting-a-storage-account"></a>Asegúrese de que la evaluación de vulnerabilidades (VA) esté habilitada en un servidor SQL Server mediante la configuración de una cuenta de almacenamiento

**Identificador**: CIS Azure 4.2.2 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[La evaluación de vulnerabilidades debe estar habilitada en Instancia administrada de SQL](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1b7aa243-30e4-4c9e-bca8-d0d3022b634a) |Audita cada servicio SQL Managed Instance que no tiene habilitado los exámenes de evaluación de vulnerabilidades periódicos. La evaluación de vulnerabilidades permite detectar las vulnerabilidades potenciales de la base de datos, así como hacer un seguimiento y ayudar a corregirlas. |AuditIfNotExists, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/VulnerabilityAssessmentOnManagedInstance_Audit.json) |
|[La evaluación de vulnerabilidades debe estar activada en sus servidores de SQL Server](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fef2a8f2a-b3d9-49cd-a8a8-9a3aaaf647d9) |Audita los servidores Azure SQL Server que no tienen habilitados los exámenes de evaluación de vulnerabilidades periódicos. La evaluación de vulnerabilidades permite detectar las vulnerabilidades potenciales de la base de datos, así como hacer un seguimiento y ayudar a corregirlas. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/VulnerabilityAssessmentOnServer_Audit.json) |

### <a name="ensure-that-va-setting-send-scan-reports-to-is-configured-for-a-sql-server"></a>Asegúrese de que la configuración de la evaluación de vulnerabilidades de "Enviar informes de examen a" esté establecida para un servidor SQL Server

**Identificador**: CIS Azure 4.2.4 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[La configuración de Evaluación de vulnerabilidad para SQL Server debe contener una dirección de correo electrónico para recibir los informes de los exámenes](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F057d6cfe-9c4f-4a6d-bc60-14420ea1f1a9) |Asegúrese de que se proporcione una dirección de correo electrónico para el campo "Enviar informes de examen a" en la opción Evaluación de vulnerabilidad. Esta dirección de correo electrónico recibe un resumen del resultado del análisis después de ejecutar un examen periódico en los servidores SQL Server. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlServer_VulnerabilityAssessmentEmails_Audit.json) |

### <a name="ensure-enforce-ssl-connection-is-set-to-enabled-for-postgresql-database-server"></a>Asegúrese de que "Aplicar conexión SSL" esté establecido en "HABILITADO" para el servidor de bases de datos PostgreSQL.

**Identificador**: CIS Azure 4.3.1 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Exigir una conexión SSL debe estar habilitado en los servidores de bases de datos MySQL](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe802a67a-daf5-4436-9ea6-f6d821dd0c5d) |Azure Database for MySQL permite conectar el servidor de Azure Database for MySQL con aplicaciones cliente mediante Capa de sockets seguros (SSL). La aplicación de conexiones SSL entre el servidor de bases de datos y las aplicaciones cliente facilita la protección frente a ataques de tipo "Man in the middle" al cifrar el flujo de datos entre el servidor y la aplicación. Esta configuración exige que SSL esté siempre habilitado para el acceso al servidor de bases de datos. |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/MySQL_EnableSSL_Audit.json) |

### <a name="ensure-enforce-ssl-connection-is-set-to-enabled-for-mysql-database-server"></a>Asegúrese de que "Aplicar conexión SSL" esté establecido en "HABILITADO" para el servidor de bases de datos MySQL.

**Identificador**: CIS Azure 4.3.2 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[La aplicación de la conexión SSL debe estar habilitada para los servidores de base de datos PostgreSQL](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd158790f-bfb0-486c-8631-2dc6b4e8e6af) |Azure Database for PostgreSQL permite conectar el servidor de Azure Database for PostgreSQL a las aplicaciones cliente mediante la Capa de sockets seguros (SSL). La aplicación de conexiones SSL entre el servidor de bases de datos y las aplicaciones cliente facilita la protección frente a ataques de tipo "Man in the middle" al cifrar el flujo de datos entre el servidor y la aplicación. Esta configuración exige que SSL esté siempre habilitado para el acceso al servidor de bases de datos. |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/PostgreSQL_EnableSSL_Audit.json) |

### <a name="ensure-server-parameter-log_checkpoints-is-set-to-on-for-postgresql-database-server"></a>Asegúrese de que "log_checkpoints" del parámetro del servidor está establecido en "ACTIVADO" para el servidor de bases de datos PostgreSQL.

**Identificador**: CIS Azure 4.3.3 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Los puntos de control del registro se deben habilitar para los servidores de base de datos de PostgreSQL](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Feb6f77b9-bd53-4e35-a23d-7f65d5f0e43d) |Esta directiva ayuda a realizar una auditoría de las bases de datos de PostgreSQL del entorno sin habilitar la opción log_checkpoints. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/PostgreSQL_EnableLogCheckpoint_Audit.json) |

### <a name="ensure-server-parameter-log_connections-is-set-to-on-for-postgresql-database-server"></a>Asegúrese de que "log_connections" del parámetro del servidor está establecido en "ACTIVADO" para el servidor de bases de datos PostgreSQL.

**Identificador**: CIS Azure 4.3.4 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Las conexiones del registro deben estar habilitadas para los servidores de bases de datos de PostgreSQL](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Feb6f77b9-bd53-4e35-a23d-7f65d5f0e442) |Esta directiva ayuda a realizar una auditoría de las bases de datos de PostgreSQL del entorno sin habilitar la opción log_connections. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/PostgreSQL_EnableLogConnections_Audit.json) |

### <a name="ensure-server-parameter-log_disconnections-is-set-to-on-for-postgresql-database-server"></a>Asegúrese de que "log_disconnections" del parámetro del servidor está establecido en "ACTIVADO" para el servidor de bases de datos PostgreSQL.

**Identificador**: CIS Azure 4.3.5 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Las desconexiones se deben registrar para los servidores de base de datos de PostgreSQL.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Feb6f77b9-bd53-4e35-a23d-7f65d5f0e446) |Esta directiva ayuda a realizar una auditoría de las bases de datos de PostgreSQL del entorno sin habilitar la opción log_disconnections. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/PostgreSQL_EnableLogDisconnections_Audit.json) |

### <a name="ensure-server-parameter-connection_throttling-is-set-to-on-for-postgresql-database-server"></a>Asegúrese de que "connection_throttling" del parámetro del servidor está establecido en "ACTIVADO" para el servidor de bases de datos PostgreSQL.

**Identificador**: CIS Azure 4.3.6 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[La limitación de conexiones debe estar habilitada para los servidores de bases de datos PostgreSQL](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5345bb39-67dc-4960-a1bf-427e16b9a0bd) |Esta directiva permite realizar una auditoría de las bases de datos de PostgreSQL del entorno sin la limitación de conexiones habilitada. Esta configuración habilita la limitación de conexiones temporales por IP si hay demasiados errores de inicio de sesión con una contraseña no válida. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/PostgreSQL_ConnectionThrottling_Enabled_Audit.json) |

### <a name="ensure-that-azure-active-directory-admin-is-configured"></a>Asegúrese de que el administrador de Azure Active Directory está configurado.

**Identificador**: CIS Azure 4.4 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[El administrador de Azure Active Directory debe aprovisionarse para servidores SQL Server](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1f314764-cb73-4fc9-b863-8eca98ac36e9) |Permite aprovisionar un administrador de Azure Active Directory para SQL Server a fin de habilitar la autenticación de Azure AD. La autenticación de Azure AD permite la administración simplificada de permisos y la administración centralizada de identidades de usuarios de base de datos y otros servicios de Microsoft |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SQL_DB_AuditServerADAdmins_Audit.json) |

### <a name="ensure-sql-servers-tde-protector-is-encrypted-with-customer-managed-key"></a>Asegúrese de que el protector de TDE de SQL Server esté cifrado con una clave administrada por el cliente.

**Identificador**: CIS Azure 4.5 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Las instancias administradas de SQL deben usar claves administradas por el cliente para cifrar los datos en reposo](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F048248b0-55cd-46da-b1ff-39efd52db260) |La implementación de Cifrado de datos transparente (TDE) con una clave propia proporciona una mayor transparencia y control sobre el protector de TDE, ofrece mayor seguridad con un servicio externo respaldado con HSM y permite la separación de tareas. Esta recomendación se aplica a las organizaciones con un requisito de cumplimiento relacionado. |AuditIfNotExists, Disabled |[1.0.2](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlManagedInstance_EnsureServerTDEisEncryptedWithYourOwnKey_Audit.json) |
|[Los servidores SQL deben usar claves administradas por el cliente para cifrar los datos en reposo](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0d134df8-db83-46fb-ad72-fe0c9428c8dd) |La implementación de Cifrado de datos transparente (TDE) con una clave propia proporciona una mayor transparencia y control sobre el protector de TDE, ofrece mayor seguridad con un servicio externo respaldado con HSM y permite la separación de tareas. Esta recomendación se aplica a las organizaciones con un requisito de cumplimiento relacionado. |AuditIfNotExists, Disabled |[2.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlServer_EnsureServerTDEisEncryptedWithYourOwnKey_Audit.json) |

## <a name="logging-and-monitoring"></a>Registro y supervisión

### <a name="ensure-the-storage-container-storing-the-activity-logs-is-not-publicly-accessible"></a>Asegúrese de que el contenedor de almacenamiento donde se almacenan los registros de actividad no sea accesible públicamente

**Identificador**: CIS Azure 5.1.3 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[No se debe permitir el acceso público a la cuenta de almacenamiento](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4fa4b6c0-31ca-4c0d-b10d-24b96f62a751) |El acceso de lectura público anónimo a contenedores y blobs de Azure Storage es una manera cómoda de compartir datos, pero también puede plantear riesgos para la seguridad. Para evitar las infracciones de datos producidas por el acceso anónimo no deseado, Microsoft recomienda impedir el acceso público a una cuenta de almacenamiento a menos que su escenario lo requiera. |audit, deny, disabled |[2.0.1-preview](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/ASC_Storage_DisallowPublicBlobAccess_Audit.json) |

### <a name="ensure-the-storage-account-containing-the-container-with-activity-logs-is-encrypted-with-byok-use-your-own-key"></a>Asegúrese de que la cuenta de almacenamiento que contiene el contenedor con los registros de actividad está cifrada con BYOK (use su propia clave).

**Identificador**: CIS Azure 5.1.4 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[La cuenta de almacenamiento que contiene el contenedor con los registros de actividad debe estar cifrada con BYOK](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ffbb99e8e-e444-4da0-9ff1-75c92f5a85b2) |Esta directiva audita si la cuenta de almacenamiento que contiene el contenedor con los registros de actividad está cifrada con BYOK. Esta directiva solo funciona si la cuenta de almacenamiento se encuentra en la misma suscripción que los registros de actividad por diseño. Se puede encontrar más información sobre el cifrado de Azure Storage en reposo en [https://aka.ms/azurestoragebyok](https://aka.ms/azurestoragebyok).  |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_StorageAccountBYOK_Audit.json) |

### <a name="ensure-that-logging-for-azure-keyvault-is-enabled"></a>Asegúrese de que el registro de Azure KeyVault esté habilitado.

**Identificador**: CIS Azure 5.1.5 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Los registros de recursos de Key Vault deben estar habilitados](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fcf820ca0-f99e-4f3e-84fb-66e913812d21) |Habilitación de la auditoría de los registros de recursos. De esta forma, puede volver a crear seguimientos de actividad con fines de investigación en caso de incidentes de seguridad o riesgos para la red. |AuditIfNotExists, Disabled |[4.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Key%20Vault/KeyVault_AuditDiagnosticLog_Audit.json) |

### <a name="ensure-that-activity-log-alert-exists-for-create-policy-assignment"></a>Asegúrese de que existe una alerta de registro de actividad para la creación de una asignación de directiva.

**Identificador**: CIS Azure 5.2.1 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Debe existir una alerta de registro de actividad para las operaciones específicas de la directiva](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc5447c04-a4d7-4ba8-a263-c9ee321a6858) |Esta directiva audita las operaciones específicas de la directiva sin alertas de registro de actividad configuradas. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_PolicyOperations_Audit.json) |

### <a name="ensure-that-activity-log-alert-exists-for-delete-policy-assignment"></a>Asegúrese de que exista una alerta de registro de actividad para la eliminación de una asignación de directiva.

**Identificador**: CIS Azure 5.2.2 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Debe existir una alerta de registro de actividad para las operaciones específicas de la directiva](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc5447c04-a4d7-4ba8-a263-c9ee321a6858) |Esta directiva audita las operaciones específicas de la directiva sin alertas de registro de actividad configuradas. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_PolicyOperations_Audit.json) |

### <a name="ensure-that-activity-log-alert-exists-for-create-or-update-network-security-group"></a>Asegúrese de que existe una alerta de registro de actividad para crear o actualizar el grupo de seguridad de red.

**Identificador**: CIS Azure 5.2.3 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Debe existir una alerta de registro de actividad para operaciones administrativas específicas](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb954148f-4c11-4c38-8221-be76711e194a) |Esta directiva audita operaciones administrativas específicas sin alertas de registro de actividad configuradas. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_AdministrativeOperations_Audit.json) |

### <a name="ensure-that-activity-log-alert-exists-for-delete-network-security-group"></a>Asegúrese de que existe una alerta de registro de actividad para eliminar el grupo de seguridad de red.

**Identificador**: CIS Azure 5.2.4 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Debe existir una alerta de registro de actividad para operaciones administrativas específicas](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb954148f-4c11-4c38-8221-be76711e194a) |Esta directiva audita operaciones administrativas específicas sin alertas de registro de actividad configuradas. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_AdministrativeOperations_Audit.json) |

### <a name="ensure-that-activity-log-alert-exists-for-create-or-update-network-security-group-rule"></a>Asegúrese de que existe una alerta de registro de actividad para la regla Crear o actualizar grupo de seguridad de red.

**Identificador**: CIS Azure 5.2.5 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Debe existir una alerta de registro de actividad para operaciones administrativas específicas](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb954148f-4c11-4c38-8221-be76711e194a) |Esta directiva audita operaciones administrativas específicas sin alertas de registro de actividad configuradas. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_AdministrativeOperations_Audit.json) |

### <a name="ensure-that-activity-log-alert-exists-for-the-delete-network-security-group-rule"></a>Asegúrese de que existe una alerta de registro de actividad para la regla Eliminar grupo de seguridad de red.

**Identificador**: CIS Azure 5.2.6 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Debe existir una alerta de registro de actividad para operaciones administrativas específicas](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb954148f-4c11-4c38-8221-be76711e194a) |Esta directiva audita operaciones administrativas específicas sin alertas de registro de actividad configuradas. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_AdministrativeOperations_Audit.json) |

### <a name="ensure-that-activity-log-alert-exists-for-create-or-update-security-solution"></a>Asegúrese de que existe una alerta de registro de actividad para la solución de seguridad de creación o actualización.

**Identificador**: CIS Azure 5.2.7 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Debe existir una alerta de registro de actividad para las operaciones específicas de seguridad](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3b980d31-7904-4bb7-8575-5665739a8052) |Esta directiva audita las operaciones específicas de seguridad sin alertas de registro de actividad configuradas. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_SecurityOperations_Audit.json) |

### <a name="ensure-that-activity-log-alert-exists-for-delete-security-solution"></a>Asegúrese de que existe una alerta de registro de actividad para la solución de seguridad de eliminación.

**Identificador**: CIS Azure 5.2.8 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Debe existir una alerta de registro de actividad para las operaciones específicas de seguridad](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3b980d31-7904-4bb7-8575-5665739a8052) |Esta directiva audita las operaciones específicas de seguridad sin alertas de registro de actividad configuradas. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_SecurityOperations_Audit.json) |

### <a name="ensure-that-activity-log-alert-exists-for-create-or-update-or-delete-sql-server-firewall-rule"></a>Asegúrese de que existe una alerta de registro de actividad para la regla de firewall Crear, actualizar o eliminar SQL Server.

**Identificador**: CIS Azure 5.2.9 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Debe existir una alerta de registro de actividad para operaciones administrativas específicas](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb954148f-4c11-4c38-8221-be76711e194a) |Esta directiva audita operaciones administrativas específicas sin alertas de registro de actividad configuradas. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/ActivityLog_AdministrativeOperations_Audit.json) |

### <a name="ensure-that-diagnostic-logs-are-enabled-for-all-services-which-support-it"></a>Asegúrese de que los registros de diagnóstico estén habilitados para todos los servicios que los admitan.

**Identificador**: CIS Azure 5.3 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Los registros de diagnóstico de App Services deben estar habilitados](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb607c5de-e7d9-4eee-9e5c-83f1bcee4fa0) |Permite auditar la habilitación de registros de diagnóstico en la aplicación. Esto le permite volver a crear seguimientos de actividad con fines de investigación si se produce un incidente de seguridad o se pone en peligro la red. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_AuditLoggingMonitoring_Audit.json) |
|[Los registros de recursos de Azure Data Lake Store deben estar habilitados](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F057ef27e-665e-4328-8ea3-04b3122bd9fb). |Habilitación de la auditoría de los registros de recursos. Esto le permite volver a crear seguimientos de actividad con fines de investigación en caso de incidente de seguridad o riesgo para la red. |AuditIfNotExists, Disabled |[4.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Data%20Lake/DataLakeStore_AuditDiagnosticLog_Audit.json) |
|[Los registros de recursos de Azure Stream Analytics deben estar habilitados](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff9be5368-9bf5-4b84-9e0a-7850da98bb46). |Habilitación de la auditoría de los registros de recursos. Esto le permite volver a crear seguimientos de actividad con fines de investigación en caso de incidente de seguridad o riesgo para la red. |AuditIfNotExists, Disabled |[4.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Stream%20Analytics/StreamAnalytics_AuditDiagnosticLog_Audit.json) |
|[Los registros de recursos de las cuentas de Batch deben estar habilitados](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F428256e6-1fac-4f48-a757-df34c2b3336d). |Habilitación de la auditoría de los registros de recursos. Esto le permite volver a crear seguimientos de actividad con fines de investigación en caso de incidente de seguridad o riesgo para la red. |AuditIfNotExists, Disabled |[4.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Batch/Batch_AuditDiagnosticLog_Audit.json) |
|[Los registros de recursos de Data Lake Analytics deben estar habilitados](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc95c74d9-38fe-4f0d-af86-0c7d626a315c) |Habilitación de la auditoría de los registros de recursos. Esto le permite volver a crear seguimientos de actividad con fines de investigación en caso de incidente de seguridad o riesgo para la red. |AuditIfNotExists, Disabled |[4.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Data%20Lake/DataLakeAnalytics_AuditDiagnosticLog_Audit.json) |
|[Los registros de recursos de la instancia de Event Hubs deben estar habilitados](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F83a214f7-d01a-484b-91a9-ed54470c9a6a). |Habilitación de la auditoría de los registros de recursos. Esto le permite volver a crear seguimientos de actividad con fines de investigación en caso de incidente de seguridad o riesgo para la red. |AuditIfNotExists, Disabled |[4.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Event%20Hub/EventHub_AuditDiagnosticLog_Audit.json) |
|[Los registros de recursos de IoT Hub deben estar habilitados](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F383856f8-de7f-44a2-81fc-e5135b5c2aa4). |Habilitación de la auditoría de los registros de recursos. Esto le permite volver a crear seguimientos de actividad con fines de investigación en caso de incidente de seguridad o riesgo para la red. |AuditIfNotExists, Disabled |[3.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Internet%20of%20Things/IoTHub_AuditDiagnosticLog_Audit.json) |
|[Los registros de recursos de Key Vault deben estar habilitados](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fcf820ca0-f99e-4f3e-84fb-66e913812d21) |Habilitación de la auditoría de los registros de recursos. De esta forma, puede volver a crear seguimientos de actividad con fines de investigación en caso de incidentes de seguridad o riesgos para la red. |AuditIfNotExists, Disabled |[4.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Key%20Vault/KeyVault_AuditDiagnosticLog_Audit.json) |
|[Los registros de recursos de Logic Apps deben estar habilitados](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F34f95f76-5386-4de7-b824-0d8478470c9d) |Habilitación de la auditoría de los registros de recursos. Esto le permite volver a crear seguimientos de actividad con fines de investigación en caso de incidente de seguridad o riesgo para la red. |AuditIfNotExists, Disabled |[4.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Logic%20Apps/LogicApps_AuditDiagnosticLog_Audit.json) |
|[Los registros de recursos de los servicios Search deben estar habilitados](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb4330a05-a843-4bc8-bf9a-cacce50c67f4). |Habilitación de la auditoría de los registros de recursos. Esto le permite volver a crear seguimientos de actividad con fines de investigación en caso de incidente de seguridad o riesgo para la red. |AuditIfNotExists, Disabled |[4.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Search/Search_AuditDiagnosticLog_Audit.json) |
|[Los registros de recursos de Service Bus deben estar habilitados](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff8d36e2f-389b-4ee4-898d-21aeb69a0f45) |Habilitación de la auditoría de los registros de recursos. Esto le permite volver a crear seguimientos de actividad con fines de investigación en caso de incidente de seguridad o riesgo para la red. |AuditIfNotExists, Disabled |[4.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Service%20Bus/ServiceBus_AuditDiagnosticLog_Audit.json) |
|[Los registros de recursos de Virtual Machine Scale Sets deben estar habilitados](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7c1b1214-f927-48bf-8882-84f0af6588b1) |Se recomienda habilitar los registros para que ese seguimiento de actividad se pueda volver a crear cuando se necesiten investigaciones en caso de incidente o riesgo. |AuditIfNotExists, Disabled |[2.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Compute/ServiceFabric_and_VMSS_AuditVMSSDiagnostics.json) |

## <a name="networking"></a>Redes

### <a name="ensure-that-rdp-access-is-restricted-from-the-internet"></a>Asegúrese de que el acceso RDP está restringido desde Internet.

**Identificador**: CIS Azure 6.1 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[El acceso RDP desde Internet debe estar bloqueado](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe372f825-a257-4fb8-9175-797a8a8627d6) |Esta directiva audita cualquier regla de seguridad de red que permita el acceso RDP desde Internet. |Audit, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/NetworkSecurityGroup_RDPAccess_Audit.json) |

### <a name="ensure-that-ssh-access-is-restricted-from-the-internet"></a>Asegúrese de que el acceso SSH está restringido desde Internet.

**Identificador**: CIS Azure 6.2 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[El acceso SSH desde Internet debe estar bloqueado](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2c89a2e5-7285-40fe-afe0-ae8654b92fab) |Esta directiva audita cualquier regla de seguridad de red que permita el acceso SSH desde Internet. |Audit, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/NetworkSecurityGroup_SSHAccess_Audit.json) |

### <a name="ensure-that-network-watcher-is-enabled"></a>Asegúrese de que Network Watcher está "Habilitado".

**Identificador**: CIS Azure 6.5 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Network Watcher debe estar habilitado](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb6e2945c-0b7b-40f5-9233-7a5323b5cdc6) |Network Watcher es un servicio regional que permite supervisar y diagnosticar problemas en un nivel de escenario de red mediante Azure. La supervisión del nivel de escenario permite diagnosticar problemas en una vista de nivel de red de un extremo a otro. Las herramientas de visualización y diagnóstico de red que incluye Network Watcher le ayudan a conocer, diagnosticar y obtener información acerca de cualquier red de Azure. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/NetworkWatcher_Enabled_Audit.json) |

## <a name="virtual-machines"></a>Virtual Machines

### <a name="ensure-virtual-machines-are-utilizing-managed-disks"></a>Asegúrese de que Virtual Machines esté utilizando Managed Disks.

**Identificador**: CIS Azure 7.1 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Auditar las máquinas virtuales que no utilizan discos administrados](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F06a78e20-9358-41c9-923c-fb736d382a4d) |Esta directiva audita las máquinas virtuales que no utilizan discos administrados. |auditoría |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Compute/VMRequireManagedDisk_Audit.json) |

### <a name="ensure-that-os-and-data-disks-are-encrypted-with-cmk"></a>Asegúrese de que la opción "OS and Data disks" (Discos de SO y de datos) estén cifrada con CMK.

**Identificador**: CIS Azure 7.2 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[El cifrado de discos debe aplicarse en máquinas virtuales](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0961003e-5a0a-4549-abde-af6a37f2724d) |Azure Security Center supervisará las máquinas virtuales sin el cifrado de disco habilitado como recomendaciones. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_UnencryptedVMDisks_Audit.json) |

### <a name="ensure-that-unattached-disks-are-encrypted-with-cmk"></a>Asegúrese de que la opción "Unattached disks" (Discos no asignados) esté cifrada con CMK.

**Identificador**: CIS Azure 7.3 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Los discos sin asignar deben cifrarse](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2c89a2e5-7285-40fe-afe0-ae8654b92fb2) |Esta directiva audita cualquier disco sin adjuntar y que no tenga el cifrado habilitado. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Compute/UnattachedDisk_Encryption_Audit.json) |

### <a name="ensure-that-only-approved-extensions-are-installed"></a>Asegúrese de que solo están instaladas las extensiones aprobadas.

**Identificador**: CIS Azure 7.4 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Solo deben instalarse las extensiones de máquina virtual aprobadas](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc0e996f8-39cf-4af9-9f45-83fbde810432) |Esta directiva rige las extensiones de máquina virtual que no están aprobadas. |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Compute/VirtualMachines_ApprovedExtensions_Audit.json) |

### <a name="ensure-that-the-latest-os-patches-for-all-virtual-machines-are-applied"></a>Asegúrese de que se aplican las revisiones del sistema operativo más recientes para todas las máquinas virtuales.

**Identificador**: CIS Azure 7.5 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Se deben instalar actualizaciones del sistema en las máquinas](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F86b3d65f-7626-441e-b690-81a8b71cff60) |Azure Security Center supervisará las actualizaciones del sistema de seguridad que faltan en los servidores como recomendaciones. |AuditIfNotExists, Disabled |[4.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_MissingSystemUpdates_Audit.json) |

### <a name="ensure-that-the-endpoint-protection-for-all-virtual-machines-is-installed"></a>Asegúrese de que Endpoint Protection esté instalado para todas las máquinas virtuales.

**Identificador**: CIS Azure 7.6 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Supervisar la falta de Endpoint Protection en Azure Security Center](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Faf6cd1bd-1635-48cb-bde7-5b15693900b9) |Azure Security Center supervisará los servidores sin un agente de Endpoint Protection instalado como recomendaciones. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_MissingEndpointProtection_Audit.json) |

## <a name="other-security-considerations"></a>Otras consideraciones de seguridad

### <a name="ensure-that-the-expiration-date-is-set-on-all-keys"></a>Asegúrese de que la fecha de expiración se haya establecida en todas las claves

**Identificador**: CIS Azure 8.1 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Las claves de Key Vault deben tener una fecha de expiración](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F152b15f7-8e1f-4c1f-ab71-8c010ba5dbc0) |Las claves criptográficas deben tener una fecha de expiración definida y no ser permanentes. Las claves que no expiran proporcionan a los posibles atacantes más tiempo para hacerse con ellas. Por ello, se recomienda como práctica de seguridad establecer fechas de expiración en las claves criptográficas. |Audit, Deny, Disabled |[1.0.1-preview](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Key%20Vault/Keys_ExpirationSet.json) |

### <a name="ensure-that-the-expiration-date-is-set-on-all-secrets"></a>Asegúrese de que la fecha de expiración se haya establecido en todos los secretos

**Identificador**: CIS Azure 8.2 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Los secretos de Key Vault deben tener una fecha de expiración](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F98728c90-32c7-4049-8429-847dc0f4fe37) |Los secretos deben tener una fecha de expiración definida y no ser permanentes. Los secretos que no expiran proporcionan a un posible atacante más tiempo para ponerlos en peligro. Por ello, se recomienda como práctica de seguridad establecer fechas de expiración en los secretos. |Audit, Deny, Disabled |[1.0.1-preview](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Key%20Vault/Secrets_ExpirationSet.json) |

### <a name="ensure-the-key-vault-is-recoverable"></a>Asegúrese de que el almacén de claves se puede recuperar.

**Identificador**: CIS Azure 8.4 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Los almacenes de claves deben tener habilitada la protección contra operaciones de purga](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0b60c0b2-2dc2-4e1c-b5c9-abbed971de53) |La eliminación malintencionada de un almacén de claves puede provocar una pérdida de datos permanente. Un usuario malintencionado de la organización puede eliminar y purgar los almacenes de claves. La protección contra purgas le protege frente a ataques internos mediante la aplicación de un período de retención obligatorio para almacenes de claves eliminados temporalmente. Ningún usuario de su organización o Microsoft podrá purgar los almacenes de claves durante el período de retención de eliminación temporal. |Audit, Deny, Disabled |[1.1.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Key%20Vault/KeyVault_Recoverable_Audit.json) |

### <a name="enable-role-based-access-control-rbac-within-azure-kubernetes-services"></a>Habilite el control de acceso basado en rol (RBAC) en Azure Kubernetes Services.

**Identificador**: CIS Azure 8.5 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Se debe usar el control de acceso basado en rol (RBAC) en los servicios de Kubernetes](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fac4a19c2-fa67-49b4-8ae5-0b2e78c49457) |Para proporcionar un filtrado detallado de las acciones que los usuarios pueden realizar, use el control de acceso basado en rol (RBAC) para administrar los permisos en los clústeres de Kubernetes Service y configurar las directivas de autorización correspondientes. |Audit, Disabled |[1.0.2](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableRBAC_KubernetesService_Audit.json) |

## <a name="app-service"></a>App Service

### <a name="ensure-app-service-authentication-is-set-on-azure-app-service"></a>Asegúrese de que la autenticación de App Service está establecida en Azure App Service.

**Identificador**: CIS Azure 9.1 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[La autenticación debe estar habilitada en la aplicación de API](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc4ebc54a-46e1-481a-bee2-d4411e95d828) |La autenticación de Azure App Service es una característica que puede impedir que solicitudes HTTP anónimas lleguen a la aplicación de API o autenticar aquellas que tienen tokens antes de que lleguen a la aplicación de API. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_Authentication_ApiApp_Audit.json) |
|[La autenticación debe estar habilitada en la aplicación de funciones](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc75248c1-ea1d-4a9c-8fc9-29a6aabd5da8) |La autenticación de Azure App Service es una característica que puede impedir que solicitudes HTTP anónimas lleguen a la aplicación de funciones o autenticar aquellas que tienen tokens antes de que lleguen a la aplicación de funciones. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_Authentication_functionapp_Audit.json) |
|[La autenticación debe estar habilitada en la aplicación web](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F95bccee9-a7f8-4bec-9ee9-62c3473701fc) |La autenticación de Azure App Service es una característica que puede impedir que solicitudes HTTP anónimas lleguen a la aplicación web o autenticar aquellas que tienen tokens antes de que lleguen a la aplicación web. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_Authentication_WebApp_Audit.json) |

### <a name="ensure-web-app-redirects-all-http-traffic-to-https-in-azure-app-service"></a>Asegúrese de que la aplicación web redirige todo el tráfico HTTP a HTTPS en Azure App Service.

**Identificador**: CIS Azure 9.2 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Acceso a la aplicación web solo a través de HTTPS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa4af4a39-4135-47fb-b175-47fbdf85311d) |El uso de HTTPS garantiza la autenticación del servicio y el servidor, y protege los datos en tránsito frente a ataques de intercepción de nivel de red. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppServiceWebapp_AuditHTTP_Audit.json) |

### <a name="ensure-web-app-is-using-the-latest-version-of-tls-encryption"></a>Asegúrese de que la aplicación web usa la versión más reciente del cifrado TLS.

**Identificador**: CIS Azure 9.3 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Debe usarse la versión más reciente de TLS en la aplicación de API](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8cb6aa8b-9e41-4f4e-aa25-089a7ac2581e) |Actualiza a la versión más reciente de TLS. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_RequireLatestTls_ApiApp_Audit.json) |
|[Debe usarse la versión más reciente de TLS en la aplicación de funciones](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff9d614c5-c173-4d56-95a7-b4437057d193) |Actualiza a la versión más reciente de TLS. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_RequireLatestTls_FunctionApp_Audit.json) |
|[Debe usarse la versión más reciente de TLS en la aplicación web](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff0e6e85b-9b9f-4a4b-b67b-f730d42f1b0b) |Actualiza a la versión más reciente de TLS. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_RequireLatestTls_WebApp_Audit.json) |

### <a name="ensure-the-web-app-has-client-certificates-incoming-client-certificates-set-to-on"></a>Asegúrese de que la aplicación web tenga la opción "Client Certificates (Incoming client certificates)" activada.

**Identificador**: CIS Azure 9.4 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Asegúrese de que la aplicación de API tenga la opción "Client Certificates (Incoming client certificates)" activada](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0c192fe8-9cbb-4516-85b3-0ade8bd03886) |Los certificados de cliente permiten que la aplicación solicite un certificado para las solicitudes entrantes. Solo los clientes que tienen un certificado válido podrán acceder a la aplicación. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_ApiApp_Audit_ClientCert.json) |
|[Asegúrese de que la aplicación web tenga la opción "Client Certificates (Incoming client certificates)" activada](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5bb220d9-2698-4ee4-8404-b9c30c9df609) |Los certificados de cliente permiten que la aplicación solicite un certificado para las solicitudes entrantes. Solo los clientes que tienen un certificado válido podrán acceder a la aplicación. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_Webapp_Audit_ClientCert.json) |
|[Las aplicaciones de funciones deben tener la opción "Certificados de cliente (certificados de cliente entrantes)" habilitada](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Feaebaea7-8013-4ceb-9d14-7eb32271373c) |Los certificados de cliente permiten que la aplicación solicite un certificado para las solicitudes entrantes. Solo los clientes con certificados válidos pueden acceder a la aplicación. |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_FunctionApp_Audit_ClientCert.json) |

### <a name="ensure-that-register-with-azure-active-directory-is-enabled-on-app-service"></a>Asegúrese de que el registro con Azure Active Directory esté habilitado en App Service.

**Identificador**: CIS Azure 9.5 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[La identidad administrada debe usarse en la aplicación de API](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc4d441f8-f9d9-4a9e-9cef-e82117cb3eef) |Usa una identidad administrada para la seguridad de autenticación mejorada. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_UseManagedIdentity_ApiApp_Audit.json) |
|[La identidad administrada debe usarse en la aplicación de funciones](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0da106f2-4ca3-48e8-bc85-c638fe6aea8f) |Usa una identidad administrada para la seguridad de autenticación mejorada. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_UseManagedIdentity_FunctionApp_Audit.json) |
|[La identidad administrada debe usarse en la aplicación web](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2b9ad585-36bc-4615-b300-fd4435808332) |Usa una identidad administrada para la seguridad de autenticación mejorada. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_UseManagedIdentity_WebApp_Audit.json) |

### <a name="ensure-that-php-version-is-the-latest-if-used-to-run-the-web-app"></a>Asegúrese de que la "Versión de PHP" es la más reciente si se usa para ejecutar la aplicación web.

**Identificador**: CIS Azure 9.6 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Asegurarse de que la "Versión de PHP" es la más reciente, si se usa como parte de la aplicación de API](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1bc1795e-d44a-4d48-9b3b-6fff0fd5f9ba) |A causa de errores de seguridad o para incluir funcionalidades, se publican de forma periódica versiones más recientes del software de PHP. Para las aplicaciones de API, se recomienda usar la versión más reciente de PHP con el fin de aprovechar las correcciones de seguridad, de haberlas, o las nuevas funcionalidades de la versión más reciente. Actualmente, esta directiva solo se aplica a las aplicaciones web de Linux. |AuditIfNotExists, Disabled |[2.1.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_ApiApp_Audit_PHP_Latest.json) |
|[Asegúrese de que la "Versión de PHP" es la más reciente, si se usa como parte de la aplicación web](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7261b898-8a84-4db8-9e04-18527132abb3) |A causa de errores de seguridad o para incluir funcionalidades, se publican de forma periódica versiones más recientes del software de PHP. Para las aplicaciones web, se recomienda usar la versión más reciente de PHP con el fin de aprovechar las correcciones de seguridad, de haberlas, o las nuevas funcionalidades de la versión más reciente. Actualmente, esta directiva solo se aplica a las aplicaciones web de Linux. |AuditIfNotExists, Disabled |[2.1.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_Webapp_Audit_PHP_Latest.json) |

### <a name="ensure-that-python-version-is-the-latest-if-used-to-run-the-web-app"></a>Asegúrese de que la "Versión de Python" es la más reciente si se usa para ejecutar la aplicación web.

**Identificador**: CIS Azure 9.7 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Asegurarse de que la "Versión de Python" es la más reciente, si se usa como parte de la aplicación de API](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F74c3584d-afae-46f7-a20a-6f8adba71a16) |A causa de errores de seguridad o para incluir funcionalidades, se publican de forma periódica versiones más recientes del software de Python. Para las aplicaciones de API, se recomienda usar la versión más reciente de Python con el fin de aprovechar las correcciones de seguridad, de haberlas, o las nuevas funcionalidades de la versión más reciente. Actualmente, esta directiva solo se aplica a las aplicaciones web de Linux. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_ApiApp_Audit_python_Latest.json) |
|[Asegúrese de que la "Versión de Python" es la más reciente, si se usa como parte de la aplicación de funciones](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7238174a-fd10-4ef0-817e-fc820a951d73) |A causa de errores de seguridad o para incluir funcionalidades, se publican de forma periódica versiones más recientes del software de Python. Para las aplicaciones de funciones, se recomienda usar la versión más reciente de Python con el fin de aprovechar las correcciones de seguridad, de haberlas, o las nuevas funcionalidades de la versión más reciente. Actualmente, esta directiva solo se aplica a las aplicaciones web de Linux. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_FunctionApp_Audit_python_Latest.json) |
|[Asegúrese de que la "Versión de Python" es la más reciente, si se usa como parte de la aplicación web](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F7008174a-fd10-4ef0-817e-fc820a951d73) |A causa de errores de seguridad o para incluir funcionalidades, se publican de forma periódica versiones más recientes del software de Python. Para las aplicaciones web, se recomienda usar la versión más reciente de Python con el fin de aprovechar las correcciones de seguridad, de haberlas, o las nuevas funcionalidades de la versión más reciente. Actualmente, esta directiva solo se aplica a las aplicaciones web de Linux. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_WebApp_Audit_python_Latest.json) |

### <a name="ensure-that-java-version-is-the-latest-if-used-to-run-the-web-app"></a>Asegúrese de que la "Versión de Java" es la más reciente si se usa para ejecutar la aplicación web.

**Identificador**: CIS Azure 9.8 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Asegurarse de que la "Versión de Java" es la más reciente, si se usa como parte de la aplicación de API](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F88999f4c-376a-45c8-bcb3-4058f713cf39) |A causa de errores de seguridad o para incluir funcionalidades, se publican de forma periódica versiones más recientes de Java. Para las aplicaciones de API, se recomienda usar la versión más reciente de Python con el fin de aprovechar las correcciones de seguridad, de haberlas, o las nuevas funcionalidades de la versión más reciente. Actualmente, esta directiva solo se aplica a las aplicaciones web de Linux. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_ApiApp_Audit_java_Latest.json) |
|[Asegúrese de que la versión de Java sea la más reciente, si se usa como parte de la aplicación de funciones](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9d0b6ea4-93e2-4578-bf2f-6bb17d22b4bc) |A causa de errores de seguridad o para incluir funcionalidades, se publican de forma periódica versiones más recientes del software de Java. Para las aplicaciones de funciones, se recomienda usar la versión más reciente de Java con el fin de aprovechar las correcciones de seguridad, de haberlas, o las nuevas funcionalidades de la versión más reciente. Actualmente, esta directiva solo se aplica a las aplicaciones web de Linux. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_FunctionApp_Audit_java_Latest.json) |
|[Asegúrese de que la "Versión de Java" es la más reciente, si se usa como parte de la aplicación web](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F496223c3-ad65-4ecd-878a-bae78737e9ed) |A causa de errores de seguridad o para incluir funcionalidades, se publican de forma periódica versiones más recientes del software de Java. Para las aplicaciones web, se recomienda usar la versión más reciente de Java con el fin de aprovechar las correcciones de seguridad, de haberlas, o las nuevas funcionalidades de la versión más reciente. Actualmente, esta directiva solo se aplica a las aplicaciones web de Linux. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_WebApp_Audit_java_Latest.json) |

### <a name="ensure-that-http-version-is-the-latest-if-used-to-run-the-web-app"></a>Asegúrese de que la "Versión de HTTP" es la más reciente, si se usa para ejecutar la aplicación web.

**Identificador**: CIS Azure 9.9 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Asegurarse de que la "Versión de HTTP" es la más reciente, si se usa para ejecutar la aplicación de API](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F991310cd-e9f3-47bc-b7b6-f57b557d07db) |A causa de errores de seguridad o para incluir funcionalidades, se publican de forma periódica versiones más recientes de HTTP. Para las aplicaciones web, use la versión más reciente de HTTP con el fin de aprovechar las correcciones de seguridad, de haberlas, o las nuevas funcionalidades de la versión más reciente. Actualmente, esta directiva solo se aplica a las aplicaciones web de Linux. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_ApiApp_Audit_HTTP_Latest.json) |
|[Asegúrese de que la "Versión de HTTP" es la más reciente, si se usa para ejecutar la aplicación de funciones](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe2c1c086-2d84-4019-bff3-c44ccd95113c) |A causa de errores de seguridad o para incluir funcionalidades, se publican de forma periódica versiones más recientes de HTTP. Para las aplicaciones web, use la versión más reciente de HTTP con el fin de aprovechar las correcciones de seguridad, de haberlas, o las nuevas funcionalidades de la versión más reciente. Actualmente, esta directiva solo se aplica a las aplicaciones web de Linux. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_FunctionApp_Audit_HTTP_Latest.json) |
|[Asegúrese de que la "Versión de HTTP" es la más reciente, si se usa para ejecutar la aplicación web](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8c122334-9d20-4eb8-89ea-ac9a705b74ae) |A causa de errores de seguridad o para incluir funcionalidades, se publican de forma periódica versiones más recientes de HTTP. Para las aplicaciones web, use la versión más reciente de HTTP con el fin de aprovechar las correcciones de seguridad, de haberlas, o las nuevas funcionalidades de la versión más reciente. Actualmente, esta directiva solo se aplica a las aplicaciones web de Linux. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_WebApp_Audit_HTTP_Latest.json) |

### <a name="ensure-ftp-deployments-are-disabled"></a>Asegúrese de que las implementaciones de FTP estén deshabilitadas.

**Identificador**: CIS Azure 9.10 **Propiedad**: Customer

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Es necesario exigir FTPS en la aplicación de API](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9a1b8c48-453a-4044-86c3-d8bfd823e4f5) |Habilite el cumplimiento con FTPS para mejorar la seguridad. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_AuditFTPS_ApiApp_Audit.json) |
|[Es necesario exigir FTPS en la aplicación de funciones](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F399b2637-a50f-4f95-96f8-3a145476eb15) |Habilite el cumplimiento con FTPS para mejorar la seguridad. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_AuditFTPS_FunctionApp_Audit.json) |
|[Es necesario exigir FTPS en la aplicación web](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4d24b6d4-5e53-4a4f-a7f4-618fa573ee4b) |Habilite el cumplimiento con FTPS para mejorar la seguridad. |AuditIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/App%20Service/AppService_AuditFTPS_WebApp_Audit.json) |

> [!NOTE]
> La disponibilidad de definiciones específicas de Azure Policy puede variar tanto en Azure Government como en otras nubes nacionales.

## <a name="next-steps"></a>Pasos siguientes

Artículos adicionales sobre Azure Policy:

- Introducción al [Cumplimiento normativo](../concepts/regulatory-compliance.md).
- Consulte la [estructura de definición de la iniciativa](../concepts/initiative-definition-structure.md).
- Consulte más ejemplos en [Ejemplos de Azure Policy](./index.md).
- Vea la [Descripción de los efectos de directivas](../concepts/effects.md).
- Obtenga información sobre cómo [corregir recursos no compatibles](../how-to/remediate-resources.md).
