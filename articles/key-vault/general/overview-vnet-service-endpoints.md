---
title: Puntos de conexión de servicio de red virtual para Azure Key Vault
description: Obtenga información sobre el modo en que los puntos de conexión de servicio de red virtual para Azure Key Vault permiten restringir el acceso a una red virtual especificada, incluidos escenarios de uso.
services: key-vault
author: amitbapat
ms.author: ambapat
ms.date: 01/02/2019
ms.service: key-vault
ms.subservice: general
ms.topic: conceptual
ms.openlocfilehash: 1bb2b8bc7e55dde7c6fa9098640109f85bebce23
ms.sourcegitcommit: 5f785599310d77a4edcf653d7d3d22466f7e05e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108065428"
---
# <a name="virtual-network-service-endpoints-for-azure-key-vault"></a>Puntos de conexión de servicio de red virtual para Azure Key Vault

Los puntos de conexión de servicio de red virtual para Azure Key Vault permiten restringir el acceso a una red virtual especificada. También permiten restringir el acceso a una lista de intervalos de direcciones IPv4 (protocolo de Internet, versión 4). A todos los usuarios que se conecten a su almacén de claves desde fuera de esos orígenes se les negará el acceso.

Hay una excepción importante a esta restricción. Si un usuario ha decidido permitir los servicios de Microsoft de confianza, se habilitan las conexiones de esos servicios a través del firewall. Por ejemplo, estos servicios incluyen Office 365 Exchange Online, Office 365 SharePoint Online, la instancia de proceso de Azure, Azure Resource Manager y Azure Backup. Tales usuarios deben presentar un token de Azure Active Directory válido y necesitan tener permisos (configurados como directivas de acceso) para realizar la operación solicitada. Para obtener más información, consulte [puntos de conexión de servicio de red virtual](../../virtual-network/virtual-network-service-endpoints-overview.md).

## <a name="usage-scenarios"></a>Escenarios de uso

Puede configurar los [firewalls y redes virtuales de Key Vault](network-security.md) para denegar el acceso al tráfico de todas las redes (incluido el tráfico de Internet) de forma predeterminada. Puede conceder acceso al tráfico desde redes virtuales específicas de Azure y rangos de direcciones IP públicas de Internet, lo que le permite generar un límite de red seguro para las aplicaciones.

> [!NOTE]
> Los firewall de Key Vault y las reglas de red virtual solo se aplican al [plan de datos](security-features.md#privileged-access) de Key Vault. Las operaciones del plano de control de Key Vault (como crear, eliminar, modificar operaciones, configurar directivas de acceso, configurar firewalls y reglas de red virtual, e implementar secretos o claves mediante plantillas de ARM) no se ven afectadas por los firewalls ni las reglas de red virtual.

Estos son algunos ejemplos de cómo puede usar los puntos de conexión de servicio:

* Si utiliza Key Vault para almacenar claves de cifrado, secretos de aplicación o certificados y quiere bloquear el acceso al almacén de claves de Internet pública.
* Si quiere bloquear el acceso al almacén de claves para que solo la aplicación o una lista breve de los hosts designados puedan conectarse al almacén de claves.
* Tiene una aplicación que se ejecuta en su red virtual de Azure y esta red virtual está bloqueada para todo el tráfico entrante y saliente. Su aplicación aún necesita conectarse a Key Vault para capturar secretos o certificados, o usar claves criptográficas.

## <a name="trusted-services"></a>Servicios de confianza

Esta es una lista de servicios de confianza que tienen permiso para acceder a un almacén de claves si está habilitada la opción **Permitir servicios de confianza**.

|Servicio de confianza|Escenarios de uso admitidos|
| --- | --- |
|Servicio de implementación de Azure Virtual Machines|[Implementar certificados en máquinas virtuales desde el almacén de claves administrado por el cliente](/archive/blogs/kv/updated-deploy-certificates-to-vms-from-customer-managed-key-vault).|
|Servicio de implementación de plantillas de Azure Resource Manager|[Pasar valores seguros durante la implementación](../../azure-resource-manager/templates/key-vault-parameter.md).|
|Servicio de cifrado de volúmenes de Azure Disk Encryption|Permitir el acceso a BitLocker Key (máquina virtual Windows) o a la frase de contraseña de DM (máquina virtual Linux) y a la clave de cifrado durante la implementación de máquinas virtuales. Esto habilita [Azure Disk Encryption](../../security/fundamentals/encryption-overview.md).|
|Azure Backup|Permitir la opción de copia de seguridad y restauración de claves y secretos pertinentes durante la copia de seguridad de máquinas virtuales de Azure mediante [Azure Backup](../../backup/backup-overview.md).|
|Exchange Online y SharePoint Online|Permitir el acceso a la clave de cliente para Azure Storage Service Encryption con la [clave de cliente](/microsoft-365/compliance/customer-key-overview).|
|Azure Information Protection|Permitir el acceso a la clave de inquilino para [Azure Information Protection.](/azure/information-protection/what-is-information-protection)|
|Azure App Service|[Implementar un certificado de Azure Web App mediante Key Vault](https://azure.github.io/AppService/2016/05/24/Deploying-Azure-Web-App-Certificate-through-Key-Vault.html).|
|Azure SQL Database|[Cifrado de datos transparente con compatibilidad con Bring Your Own Key para Azure SQL Database y Azure Synapse Analytics](../../azure-sql/database/transparent-data-encryption-byok-overview.md).|
|Azure Storage|[Storage Service Encryption mediante claves administradas por el cliente en Azure Key Vault](../../storage/common/customer-managed-keys-configure-key-vault.md).|
|Azure Data Lake Store|[Cifrado de datos en Azure Data Lake Store](../../data-lake-store/data-lake-store-encryption.md) con una clave administrada de cliente.|
|Azure Synapse Analytics|[Cifrado de datos mediante claves administradas por el cliente en Azure Key Vault](../../synapse-analytics/security/workspaces-encryption.md)|
|Azure Databricks|[Servicio de análisis rápido, sencillo y de colaboración basado en Apache Spark](/azure/databricks/scenarios/what-is-azure-databricks)|
|Azure API Management|[Implementación de certificados para el dominio personalizado de Key Vault mediante MSI](../../api-management/api-management-howto-use-managed-service-identity.md#use-ssl-tls-certificate-from-azure-key-vault)|
|Azure Data Factory|[Captura de credenciales del almacén de datos en Key Vault desde Data Factory](https://go.microsoft.com/fwlink/?linkid=2109491)|
|Azure Event Hubs|[Permitir el acceso a un almacén de claves para un escenario de claves administradas por el cliente](../../event-hubs/configure-customer-managed-key.md)|
|Azure Service Bus|[Permitir el acceso a un almacén de claves para un escenario de claves administradas por el cliente](../../service-bus-messaging/configure-customer-managed-key.md)|
|Azure Import/Export| [Uso de claves administradas por el cliente en Azure Key Vault para el servicio de importación y exportación](../../import-export/storage-import-export-encryption-key-portal.md)
|Azure Container Registry|[Cifrado del registro con claves administradas por el cliente](../../container-registry/container-registry-customer-managed-keys.md)
|Azure Application Gateway |[Usar certificados de Key Vault para los clientes de escucha habilitados para HTTPS](../../application-gateway/key-vault-certs.md)
|Azure Front Door|[Uso de certificados de Key Vault para HTTPS](../../frontdoor/front-door-custom-domain-https.md#prepare-your-azure-key-vault-account-and-certificate)

> [!NOTE]
> Debe configurar las directivas de acceso pertinentes de Key Vault para permitir que los servicios correspondientes obtengan acceso a Key Vault.

## <a name="next-steps"></a>Pasos siguientes

- Para ver las instrucciones detalladas paso a paso, consulte [Configuración de firewalls y redes virtuales de Azure Key Vault](network-security.md).
- Consulte [Introducción a la seguridad de Azure Key Vault](security-features.md).
