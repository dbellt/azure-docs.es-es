---
title: Planeamiento de una aplicación administrada de Azure para una oferta de Aplicación de Azure
description: Aprenda lo que es necesario para crear un plan de aplicación administrada para una nueva oferta de Aplicación de Azure usando el portal del marketplace comercial en el Centro de partners de Microsoft.
author: aarathin
ms.author: aarathin
ms.reviewer: dannyevers
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 11/06/2020
ms.openlocfilehash: a34bdfe55d2200176d6f6c7fde389c19dc26cfa5
ms.sourcegitcommit: bd65925eb409d0c516c48494c5b97960949aee05
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/06/2021
ms.locfileid: "111536474"
---
# <a name="plan-an-azure-managed-application-for-an-azure-application-offer"></a>Planeamiento de una aplicación administrada de Azure para una oferta de Aplicación de Azure

Un plan de _aplicación administrada_ de Azure es una manera de publicar una oferta de Aplicación de Azure en Azure Marketplace. Si aún no lo ha hecho, lea la sección [Planeamiento de una oferta de Aplicación de Azure para el marketplace comercial](plan-azure-application-offer.md).

Las aplicaciones administradas son ofertas de transacciones que se implementan y facturan mediante Azure Marketplace. La opción de publicación que el usuario ve es Obtener ahora.

Uso de una Aplicación de Azure: Plan de aplicación administrada en las siguientes circunstancias:

- Va a implementar una solución basada en suscripciones para su cliente mediante una máquina virtual (VM) o una solución completa basada en infraestructura como servicio (IaaS).
- Usted o su cliente requieren que un asociado administre la solución. Por ejemplo, un asociado puede ser un integrador de sistemas o un proveedor de servicios administrados (MSP).

## <a name="managed-application-offer-requirements"></a>Requisitos de las ofertas de aplicaciones administradas

| Requisitos | Detalles |
| ------------ | ------------- |
| Una suscripción de Azure | Las aplicaciones administradas se deben implementar en la suscripción del cliente, pero las puede administrar un tercero. |
| Facturación y medición | Los recursos se proporcionan en la suscripción de Azure de un cliente. Las transacciones de VM que usan el modelo de pago por uso se realizan con el cliente mediante Microsoft y se facturan con la suscripción de Azure del cliente. <br><br> En el caso de las VM con el modelo "traiga su propia licencia", Microsoft factura los costos de infraestructura derivados en la suscripción del cliente, mientras que usted realizará la transacción de las tarifas de licencia de software directamente con el cliente. |
| Disco duro virtual (VHD) compatible con Azure | Las máquinas virtuales deben estar basadas en Windows o Linux. Para más información, consulte:<br> • [Creación de un recurso técnico de máquina virtual de Azure](./azure-vm-create-certification-faq.md#address-a-vulnerability-or-an-exploit-in-a-vm-offer) (discos duros virtuales de Windows).<br> • [Distribuciones de Linux aprobadas en Azure](../virtual-machines/linux/endorsed-distros.md) (para discos duros virtuales de Linux). |
| Atribución de uso del cliente | Todas las nuevas ofertas de la aplicación de Azure deben incluir un GUID de [atribución de uso del cliente para asociados de Azure](azure-partner-customer-usage-attribution.md). Para obtener más información sobre la atribución de uso del cliente y cómo habilitarla, consulte [Atribución de uso del cliente para asociados de Azure](azure-partner-customer-usage-attribution.md). |
| Paquete de implementación | Necesitará un paquete de implementación que permita a los clientes implementar el plan. Si crea varios planes que requieren la misma configuración técnica, puede usar el mismo paquete. Para más información, consulte la siguiente sección: Paquete de implementación. |
|||

> [!NOTE]
> Las aplicaciones administradas se deben poder implementar desde Azure Marketplace. Si la comunicación con el cliente es una preocupación, póngase en contacto con los clientes interesados después de habilitar el uso compartido de clientes potenciales.

## <a name="deployment-package"></a>Paquete de implementación

El paquete de implementación contiene todos los archivos de plantilla necesarios para este plan, así como recursos adicionales, empaquetados en un archivo .zip.

Todas las aplicaciones de Azure deben incluir estos dos archivos en la carpeta raíz de un archivo .zip:

- Un archivo de plantilla de Resource Manager llamado [mainTemplate.json](../azure-resource-manager/managed-applications/publish-service-catalog-app.md?tabs=azure-powershell#create-the-arm-template). Esta plantilla define los recursos que se van a implementar en la suscripción de Azure del cliente. Para ver ejemplos de plantillas de Resource Manager, consulte la [galería de plantillas de inicio rápido de Azure](https://azure.microsoft.com/documentation/templates/) o el repositorio de [GitHub correspondiente: Plantillas de inicio rápido de Azure Resource Manager](https://github.com/azure/azure-quickstart-templates) correspondiente.
- Una definición de interfaz de usuario para la experiencia de creación de aplicaciones de Azure llamada [createUiDefinition.json](../azure-resource-manager/managed-applications/create-uidefinition-overview.md). En la interfaz de usuario, puede especificar los elementos que permiten a los consumidores proporcionar los valores de los parámetros.

Los tamaños de archivo máximos admitidos son:

- Hasta 1 GB para el tamaño de archivo .zip comprimido total.
- Hasta 1 GB para cada archivo individual descomprimido dentro del archivo .zip.

Todas las nuevas ofertas de la aplicación de Azure deben incluir un GUID de [atribución de uso del cliente para asociados de Azure](azure-partner-customer-usage-attribution.md).

## <a name="azure-regions"></a>Regiones de Azure

Puede publicar su plan en la región pública de Azure, en la región de Azure Government, o en ambas. Antes de publicar en [Azure Government](../azure-government/documentation-government-manage-marketplace-partners.md), pruebe y valide el plan en el entorno, ya que algunos puntos de conexión pueden ser diferentes. Para configurar y probar el plan, solicite una cuenta de prueba desde [Prueba de Microsoft Azure Government](https://azure.microsoft.com/global-infrastructure/government/request/).

Como publicador, es responsable de los controles de cumplimiento, las medidas de seguridad y los procedimientos recomendados. Azure Government usa redes y centros de datos aislados físicamente (ubicados solo en Estados Unidos).

Para ver una lista de países y regiones que se admiten en el marketplace comercial, consulte [Disponibilidad geográfica y compatibilidad con monedas](marketplace-geo-availability-currencies.md).

Los servicios de Azure Government controlan datos que están sujetos a determinados reglamentos y requisitos gubernamentales. Por ejemplo, FedRAMP, NIST 800.171 (DIB), ITAR, IRS 1075, DoD L4 y CJIS. Para dar a conocer sus certificaciones de estos programas, puede proporcionar hasta 100 vínculos que las describan. Pueden ser vínculos a su anuncio en el programa directamente o a descripciones de su cumplimiento de estos en sus propios sitios web. Estos vínculos solo son visibles para los clientes de Azure Government.

## <a name="choose-who-can-see-your-plan"></a>Selección de quién puede ver el plan

Puede configurar cada plan para que sea visible para todo el mundo o solo para una audiencia concreta (privada). Puede crear hasta 100 planes, y 45 de ellos pueden ser privados. Puede que quiera crear un plan privado para ofrecer diferentes opciones de precios o configuraciones técnicas a clientes específicos.

Puede conceder acceso a un plan privado con los identificadores de la suscripción de Azure con la opción de incluir una descripción de cada identificador de suscripción que asigne. Puede agregar hasta 10 identificadores de suscripción manualmente o hasta 10 000 mediante un archivo .csv. Los identificadores de suscripción de Azure se representan como GUID y las letras deben estar en minúsculas.

Los planes privados no son compatibles con las suscripciones de Azure establecidas mediante un revendedor del programa de proveedores de soluciones en la nube (CSP). Para más información, consulte [Ofertas privadas en el marketplace comercial de Azure](private-offers.md).

> [!NOTE]
> Si publica un plan privado, puede cambiar su visibilidad a público más adelante. Sin embargo, una vez que se publica un plan público, no se puede cambiar su visibilidad a privado.

## <a name="define-pricing"></a>Definición de los precios

Debe proporcionar el precio mensual de cada plan. El precio se suma a cualquier infraestructura de Azure o costos de software de pago por uso en los que incurren los recursos implementados por esta solución.

Además del precio por mes, también puede establecer precios por el consumo de unidades no estándar mediante la [facturación de uso medido](marketplace-metering-service-apis.md). Si lo desea, puede establecer el precio por mes en cero y cobrar exclusivamente con la facturación de uso medido.

Los precios establecidos en USD (USD = dólares de Estados Unidos) se convierten a la moneda local de todos los mercados seleccionados con los tipos de cambio actuales en el momento de guardarlos. Sin embargo, puede elegir establecer los precios de los clientes para cada mercado.

## <a name="just-in-time-jit-access"></a>Acceso Just-In-Time (JIT)

El acceso JIT permite solicitar acceso con privilegios elevados a recursos de la aplicación administrada con fines de solución de problemas o mantenimiento. Siempre tendrá acceso de solo lectura a los recursos, pero puede tener mayor acceso durante un período de tiempo específico. Para obtener más información, consulte [Habilitación y solicitud de acceso Just-In-Time para Azure Managed Applications](../azure-resource-manager/managed-applications/request-just-in-time-access.md).

> [!NOTE]
> Asegúrese de actualizar su archivo `createUiDefinition.json` para admitir esta característica.

## <a name="deployment-mode"></a>Modo de implementación

Puede configurar un plan de aplicación administrada para usar el modo de implementación **Completa** o **Incremental**. En el modo completo, una reimplementación de la aplicación por parte del cliente provocará la eliminación de recursos del grupo de recursos administrado si los recursos no están definidos en [mainTemplate.json](../azure-resource-manager/managed-applications/publish-service-catalog-app.md?tabs=azure-powershell#create-the-arm-template). En el modo incremental, una reimplementación de la aplicación deja los recursos existentes sin cambios. Para más información, consulte [Modos de implementación de Azure Resource Manager](../azure-resource-manager/templates/deployment-modes.md).

## <a name="notification-endpoint-url"></a>Dirección URL del punto de conexión de notificaciones

Opcionalmente, puede proporcionar un punto de conexión de webhook HTTPS para recibir notificaciones sobre todas las operaciones CRUD que están en las instancias de aplicación administrada de un plan.

## <a name="customize-allowed-customer-actions-optional"></a>Personalización de las acciones de cliente permitidas (opcional)

Opcionalmente, puede especificar qué acciones pueden realizar los clientes en los recursos administrados, además de las acciones "`*/read`" que están disponibles de forma predeterminada.

Si elige esta opción, debe proporcionar las acciones de control, las acciones de datos permitidas, o ambas. Para más información, consulte [Descripción de las asignaciones de denegación para recursos de Azure](../role-based-access-control/deny-assignments.md). Puede consultar las acciones disponibles en [Operaciones del proveedor de recursos de Azure Resource Manager](../role-based-access-control/resource-provider-operations.md). Por ejemplo, para permitir que los consumidores reinicien las máquinas virtuales, agregue `Microsoft.Compute/virtualMachines/restart/action` a las acciones permitidas.

## <a name="choose-who-can-manage-the-application"></a>Selección de quién puede administrar la aplicación

Debe indicar quién puede administrar una aplicación administrada en cada una de las nubes seleccionadas: _Azure pública_ y _nube de Azure Government_. Recopile la información siguiente:

- **Id. de inquilino de Azure Active Directory**: el identificador de inquilino de Azure AD (también conocido como identificador de directorio) que contiene las identidades de los usuarios, grupos o aplicaciones a los que quiere conceder permisos. Encontrará su identificador de inquilino de Azure AD en la página [Propiedades de Azure Active Directory](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Properties), en Azure Portal.
- **Autorizaciones**: agregue el identificador de objeto de Azure Active Directory del usuario, grupo o aplicación a los que quiere conceder permiso para el grupo de recursos administrado. Identifique al usuario por su identificador de entidad de seguridad, que encontrará en la [hoja Usuarios de Azure Active Directory, en Azure Portal](https://portal.azure.com/#blade/Microsoft_AAD_IAM/UsersManagementMenuBlade/AllUsers).

A cada identificador de entidad de seguridad le asociará uno de los roles integrados de Azure AD (propietario o colaborador). El rol que seleccione describirá los permisos que tendrá la entidad de seguridad en los recursos de la suscripción del cliente. Para más información, consulte [Roles integrados en Azure](../role-based-access-control/built-in-roles.md). Para obtener más información sobre el control de acceso basado en rol (RBAC), consulte [Introducción al control de acceso basado en rol en Azure Portal](../role-based-access-control/overview.md).

> [!NOTE]
> Aunque puede agregar hasta 100 autorizaciones por región de Azure, suele ser más fácil crear un grupo de usuarios de Active Directory y especificar su identificador en "Id. de entidad de seguridad". Esta opción le permite agregar más usuarios al grupo de administración una vez implementado el plan y reducir la necesidad de actualizar el plan solo para agregar más autorizaciones.

## <a name="policy-settings"></a>Configuración de directiva

Puede aplicar [directivas de Azure](../governance/policy/index.yml) a la aplicación administrada para especificar los requisitos de cumplimiento de la solución implementada. Para las definiciones de directiva y el formato de los valores de parámetro, consulte [ejemplos de Azure Policy](../governance/policy/samples/index.md).

Puede configurar hasta cinco directivas y solo una instancia de cada tipo de directiva. Algunos tipos de directiva requieren parámetros adicionales.

| Tipo de directiva | Parámetros de directiva requeridos |
| ------------ | ------------- |
| Cifrado de Azure SQL Database | No |
| Configuración de auditoría de Azure SQL Server | Sí |
| Cifrado de Azure Data Lake Store | No |
| Auditar la configuración de diagnóstico | Sí |
| Auditar el cumplimiento de la ubicación de recursos | No |
|||

Para cada tipo de directiva que agregue, debe asociar la SKU de directiva estándar o gratuita. La SKU Estándar es obligatoria para las directivas de auditoría. Los nombres de directiva están limitados a 50 caracteres.

## <a name="next-steps"></a>Pasos siguientes

- [Creación de una oferta de aplicación de Azure](azure-app-offer-setup.md)
