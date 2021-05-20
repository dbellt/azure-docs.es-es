---
title: Seguridad para las soluciones de Azure Digital Twins
titleSuffix: Azure Digital Twins
description: Procedimientos recomendados de seguridad con Azure Digital Twins
author: baanders
ms.author: baanders
ms.date: 3/18/2020
ms.topic: conceptual
ms.service: digital-twins
ms.openlocfilehash: 5224804b1e0d306bfca8e9e42511ad908a3bb5c1
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109787750"
---
# <a name="secure-azure-digital-twins"></a>Protección de Azure Digital Twins

Por seguridad, Azure Digital Twins permite el control de acceso preciso sobre datos, recursos y acciones específicos en su implementación. Para ello, se usa una estrategia pormenorizada de roles y permisos que se conoce como **Control de acceso basado en roles de Azure (Azure RBAC)** . Puede leer sobre los principios generales de Azure RBAC [aquí](../role-based-access-control/overview.md).

Azure Digital Twins también admite el cifrado de datos en reposo.

## <a name="roles-and-permissions-with-azure-rbac"></a>Roles y permisos con RBAC de Azure

Azure RBAC se proporciona en Azure Digital Twins mediante la integración con [Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md) (Azure AD).

Puede usar Azure RBAC para conceder permisos a una *entidad de seguridad*, que puede ser un usuario, un grupo o una entidad de servicio de aplicación. Azure AD autentica la entidad de seguridad y recibe un token de OAuth 2.0 a cambio. Este token se puede usar para autorizar una solicitud de acceso a una instancia de Azure Digital Twins.

### <a name="authentication-and-authorization"></a>Autenticación y autorización

Con Azure AD, el acceso es un proceso de dos pasos. Cuando una entidad de seguridad (un usuario, un grupo o una aplicación) intenta acceder a Azure Digital Twins, la solicitud se debe *autenticar* y *autorizar*. 

1. En primer lugar, se *autentica* la identidad de la entidad de seguridad y se devuelve un token de OAuth 2.0.
2. Luego, el token se pasa como parte de una solicitud al servicio Azure Digital Twins para *autorizar* el acceso al recurso especificado.

El paso de autenticación exige que cualquier solicitud de aplicación contenga un token de acceso de OAuth 2.0 en tiempo de ejecución. Si una aplicación se ejecuta dentro de una entidad de Azure, como una aplicación de [Azure Functions](../azure-functions/functions-overview.md), puede usar una **identidad administrada** para acceder a los recursos. Para obtener más información sobre las identidades administradas, consulte la siguiente sección.

El paso de autorización requiere que se asigne un rol de Azure a la entidad de seguridad. Los roles que se asignan a una entidad de seguridad determinan los permisos que tiene esa entidad de seguridad. Azure Digital Twins proporciona roles de Azure que abarcan conjuntos de permisos para los recursos de Azure Digital Twins. Estos roles se describen más adelante en este artículo.

Para más información sobre los roles y las asignaciones de roles que se admiten en Azure, consulte la [Descripción de los distintos roles](../role-based-access-control/rbac-and-directory-admin-roles.md) en la documentación de RBAC de Azure.

#### <a name="authentication-with-managed-identities"></a>Autenticación con identidades administradas

[Identidades administradas para recursos de Azure](../active-directory/managed-identities-azure-resources/overview.md) es una característica de Azure que permite crear una identidad segura asociada con la implementación en la que se ejecuta el código de la aplicación. A continuación, puede asociar esa identidad con los roles de control de acceso para conceder permisos personalizados de acceso a recursos específicos de Azure que la aplicación necesita.

Con las identidades administradas, la plataforma Azure administra esta identidad en tiempo de ejecución. No es necesario almacenar y proteger las claves de acceso en la configuración o el código de la aplicación, ya sea para la propia identidad o para los recursos a los que necesita acceder. Una aplicación cliente de Azure Digital Twins que se ejecuta dentro de una aplicación de Azure App Service no necesita controlar las reglas y claves SAS ni cualquier otro token de acceso. La aplicación cliente solo necesita la dirección del punto de conexión del espacio de nombres de Azure Digital Twins. Cuando se conecta la aplicación, Azure Digital Twins enlaza el contexto de la entidad administrada con el cliente. Una vez creada la asociación con una identidad administrada, el cliente de Azure Digital Twins puede realizar todas las operaciones autorizadas. Para conceder la autorización, se asociará una entidad administrada con un rol de Azure Digital Twins (los roles se describen a continuación).

#### <a name="authorization-azure-roles-for-azure-digital-twins"></a>Autorización: Roles de Azure para Azure Digital Twins

Azure proporciona **dos roles integrados de Azure** para autorizar el acceso a las [API del plano de datos](concepts-apis-sdks.md#overview-data-plane-apis) de Azure Digital Twins. Puede hacer referencia a los roles por nombre o por identificador:

| Rol integrado | Descripción | ID | 
| --- | --- | --- |
| Propietario de datos de Azure Digital Twins | Proporciona acceso completo a los recursos de Azure Digital Twins. | bcd981a7-7f74-457b-83e1-cceb9e632ffe |
| Lector de datos de Azure Digital Twins | Proporciona acceso de solo lectura a los recursos de Azure Digital Twins. | d57506d4-4c8d-48b1-8587-93c323f6a5a3 |

Puede asignar roles de dos maneras:
* mediante el panel de control de acceso (IAM) de Azure Digital Twins en Azure Portal (consulte [Asignación de roles de Azure mediante Azure Portal](../role-based-access-control/role-assignments-portal.md))
* mediante comandos de la CLI para agregar o quitar un rol

Para obtener pasos más detallados sobre cómo hacerlo, pruébelo en el [Tutorial de Azure Digital Twins: Conexión de una solución de un extremo a otro](tutorial-end-to-end.md).

Para más información sobre cómo se definen los roles integrados, consulte [Descripción de definiciones de roles](../role-based-access-control/role-definitions.md) en la documentación de RBAC de Azure. Para más información acerca de la creación de roles personalizados de Azure, consulte [Roles personalizados de Azure](../role-based-access-control/custom-roles.md).

##### <a name="automating-roles"></a>Automatización de roles

Al hacer referencia a los roles en escenarios automatizados, se recomienda hacer referencia a ellos por sus **identificadores** en lugar de sus nombres. Los nombres pueden cambiar entre versiones, pero los identificadores no, lo que los convierte en una referencia más estable en la automatización.

> [!TIP]
> Si se asignan roles con un cmdlet, como `New-AzRoleAssignment` ([referencia](/powershell/module/az.resources/new-azroleassignment)), puede usar el parámetro `-RoleDefinitionId` en lugar de `-RoleDefinitionName` para pasar un identificador en lugar de un nombre para el rol.

### <a name="permission-scopes"></a>Ámbitos de permiso

Antes de asignar un rol de Azure a una entidad de seguridad, determine el ámbito de acceso que debería tener la entidad de seguridad. Los procedimientos recomendados dictan que siempre es mejor conceder únicamente el ámbito más restringido posible.

En la lista siguiente se describen los niveles en los que puede definir el ámbito de acceso a recursos de Azure Digital Twins.
* Modelos: Las acciones para este recurso dictan el control sobre los [modelos](concepts-models.md) cargados en Azure Digital Twins.
* Consulta del grafo de Digital Twins: Las acciones para este recurso determinan la capacidad de ejecutar [operaciones de consulta](concepts-query-language.md) en gemelos digitales en el grafo de Azure Digital Twins.
* Gemelo digital: Las acciones de este recurso proporcionan control sobre las operaciones de CRUD en [gemelos digitales](concepts-twins-graph.md) en el grafo gemelo.
* Relación de los gemelos digitales: Las acciones de este recurso definen el control sobre las operaciones de CRUD en las [relaciones](concepts-twins-graph.md) entre gemelos digitales en el grafo gemelo.
* Ruta del evento: Las acciones de este recurso determinan los permisos para [redirigir eventos](concepts-route-events.md) de Azure Digital Twins a un servicio de punto de conexión como [Event Hub](../event-hubs/event-hubs-about.md), [Event Grid](../event-grid/overview.md) o [Service Bus](../service-bus-messaging/service-bus-messaging-overview.md).

### <a name="troubleshooting-permissions"></a>Solución de problemas de permisos

Si un usuario intenta realizar una acción no permitida por su rol, es posible que reciba el siguiente error de la solicitud de servicio: `403 (Forbidden)`. Para obtener más información y pasos para solucionar problemas, vea [Solución de problemas: Error en la solicitud de Azure Digital Twins con el estado: 403 (Prohibido)](troubleshoot-error-403.md) .

## <a name="managed-identity-for-accessing-other-resources-preview"></a>Identidad administrada para acceder a otros recursos (versión preliminar)

La configuración de una **identidad administrada** de [Azure Active Directory (Azure AD)](../active-directory/fundamentals/active-directory-whatis.md) para una instancia de Azure Digital Twins puede permitir que la instancia acceda fácilmente a otros recursos protegidos de Azure AD, como [Azure Key Vault](../key-vault/general/overview.md). La identidad está administrada por la plataforma Azure y no es necesario que aprovisione o rote ningún secreto. Para obtener más información sobre las identidades administradas en Azure AD, vea [Identidades administradas para recursos de Azure](../active-directory/managed-identities-azure-resources/overview.md). 

Azure admite dos tipos de identidades administradas: asignadas por el sistema y asignadas por el usuario. Actualmente, Azure Digital Twins solo admite las **identidades asignadas por el sistema**. 

Puede usar una identidad administrada asignada por el sistema para que la instancia de Azure Digital Twins se autentique en un [punto de conexión definido por el usuario](concepts-route-events.md#create-an-endpoint). Azure Digital Twins admite la autenticación basada en identidades asignadas por el sistema en los puntos de conexión de destinos de [centro de eventos](../event-hubs/event-hubs-about.md) y  [Service Bus](../service-bus-messaging/service-bus-messaging-overview.md) , y en un punto de conexión de [Azure Storage Container](../storage/blobs/storage-blobs-introduction.md)  para los [eventos de mensajes con problemas de entrega](concepts-route-events.md#dead-letter-events). Actualmente, los puntos de conexión de [Event Grid](../event-grid/overview.md) no se admiten para las identidades administradas.

Para obtener instrucciones sobre cómo habilitar una identidad administrada por el sistema para Azure Digital Twins y usarla para enrutar eventos, vea [Procedimiento para habilitar una identidad administrada para el enrutamiento de eventos (versión preliminar)](./how-to-enable-managed-identities-portal.md) .

## <a name="private-network-access-with-azure-private-link-preview"></a>Acceso a redes privadas con Azure Private Link (versión preliminar)

[Azure Private Link](../private-link/private-link-overview.md) es un servicio que le permite acceder a recursos de Azure (como [Azure Event Hubs](../event-hubs/event-hubs-about.md), [Azure Storage](../storage/common/storage-introduction.md) y [Azure Cosmos DB](../cosmos-db/introduction.md)) y a los servicios de asociados o clientes hospedados por Azure mediante un punto de conexión privado en la instancia de [Azure Virtual Network (VNet)](../virtual-network/virtual-networks-overview.md). 

Del mismo modo, puede usar puntos de conexión privados para la instancia de Azure Digital Twins a fin de permitir que los clientes ubicados en la red virtual accedan de forma segura a la instancia mediante Private Link. 

El punto de conexión privado usa una dirección IP del espacio de direcciones de la red virtual de Azure. El tráfico de red entre un cliente en la red privada y la instancia de Azure Digital Twins atraviesa la red virtual y un servicio Private Link en la red troncal de Microsoft, lo que elimina la exposición a la red pública de Internet. Esta es una representación visual del sistema:

:::image type="content" source="media/concepts-security/private-link.png" alt-text="Un diagrama en el que se muestra una red para una empresa PowerGrid que es una red virtual protegida sin acceso a Internet ni a la nube pública, que se conecta a través de Private Link a una instancia de Azure Digital Twins llamada CityOfTwins.":::

La configuración de un punto de conexión privado para la instancia de Azure Digital Twins le permite protegerla y eliminar la exposición pública, además de evitar la filtración de datos desde la red virtual.

Para obtener instrucciones sobre cómo configurar Private Link para Azure Digital Twins, vea [Procedimiento para habilitar el acceso privado con Private Link (versión preliminar)](./how-to-enable-private-link-portal.md) .

### <a name="design-considerations"></a>Consideraciones de diseño 

Al trabajar con Private Link para Azure Digital Twins, estos son algunos de los factores que debería tener en cuenta:
* **Precios**: para obtener más información sobre los precios, vea  [Precios de Azure Private Link](https://azure.microsoft.com/pricing/details/private-link). 
* **Disponibilidad regional**: en el caso de Azure Digital Twins, esta característica está disponible en todas las regiones de Azure donde está disponible Azure Digital Twins. 
* **Número máximo de puntos de conexión privados por instancia de Azure Digital Twins**: 10

Para obtener información sobre los límites de Azure Private Link, vea  [Documentación de Azure Private Link: Limitaciones](../private-link/private-link-service-overview.md#limitations).

## <a name="service-tags"></a>Etiquetas de servicio

Una **etiqueta de servicio** representa un grupo de prefijos de direcciones IP de un servicio de Azure determinado. Microsoft administra los prefijos de direcciones que la etiqueta de servicio incluye y actualiza automáticamente dicha etiqueta a medida que las direcciones cambian, lo que minimiza la complejidad de las actualizaciones frecuentes en las reglas de seguridad de red. Para más información sobre las etiquetas de servicio, consulte  [Etiquetas de red virtuales](../virtual-network/service-tags-overview.md). 

Puede usar etiquetas de servicio para definir controles de acceso de red en  [grupos de seguridad de red](../virtual-network/network-security-groups-overview.md#security-rules) o [Azure Firewall](../firewall/service-tags.md), usando etiquetas de servicio en lugar de direcciones IP específicas al crear reglas de seguridad. Al especificar el nombre de la etiqueta de servicio (en este caso,  **AzureDigitalTwins**) en el campo de  *origen* o *destino* apropiado de una regla, puede permitir o denegar el tráfico para el servicio correspondiente. 

A continuación, se muestran los detalles de la etiqueta de servicio **AzureDigitalTwins**.

| Etiqueta | Propósito | ¿Se puede usar para tráfico entrante o saliente? | ¿Puede ser regional? | ¿Se puede usar con Azure Firewall? |
| --- | --- | --- | --- | --- |
| AzureDigitalTwins | Azure Digital Twins<br>Nota: Esta etiqueta o las direcciones IP que abarca se pueden utilizar para restringir el acceso a los puntos de conexión configurados para [rutas de eventos](concepts-route-events.md). | Entrada | No | Sí |

### <a name="using-service-tags-for-accessing-event-route-endpoints"></a>Uso de etiquetas de servicio para acceder a puntos de conexión de ruta de eventos 

Estos son los pasos para acceder a puntos de conexión de [ruta de eventos](concepts-route-events.md) usando etiquetas de servicio con Azure Digital Twins.

1. En primer lugar, descargue esta referencia de archivo JSON que muestra los intervalos IP y las etiquetas de servicio de Azure: [Intervalos IP y etiquetas de servicio de Azure](https://www.microsoft.com/download/details.aspx?id=56519). 

2. Busque los intervalos IP de "AzureDigitalTwins" en el archivo JSON.  

3. Consulte la documentación del recurso externo conectado al punto de conexión (por ejemplo [Event Grid](../event-grid/overview.md), [Event Hub](../event-hubs/event-hubs-about.md), [Service Bus](../service-bus-messaging/service-bus-messaging-overview.md) o [Azure Storage](../storage/blobs/storage-blobs-overview.md) en busca de los [eventos con problemas de entrega](concepts-route-events.md#dead-letter-events)) para ver cómo definir filtros IP para ese recurso.

4. Establezca filtros IP en los recursos externos mediante los intervalos IP del *paso 2*.  

5. Actualice los intervalos IP periódicamente según sea necesario. Los rangos pueden cambiar con el tiempo, por lo que es aconsejable comprobarlos periódicamente y actualizarlos cuando sea necesario. Aunque la frecuencia de estas actualizaciones puede variar, es una buena idea comprobarlos una vez por semana.

## <a name="encryption-of-data-at-rest"></a>Cifrado de datos en reposo

Azure Digital Twins permite el cifrado de los datos en reposo y en tránsito a medida que se escriben en nuestros centros de datos y los descifra automáticamente mientras accede a ellos. Este cifrado se produce mediante una clave de cifrado administrada de Microsoft.

## <a name="cross-origin-resource-sharing-cors"></a>Uso compartido de recursos entre orígenes

Azure Digital Twins no admite actualmente el **uso compartido de recursos entre orígenes (CORS)** . Como resultado, si está llamando a una API REST desde una aplicación de explorador, una interfaz de [API Management (APIM)](../api-management/api-management-key-concepts.md) o un conector de [Power Apps](/powerapps/powerapps-overview), es posible que vea un error de directiva.

Para resolver este error, puede realizar una de las acciones siguientes:
* Quite el encabezado CORS `Access-Control-Allow-Origin` del mensaje. Este encabezado indica si la respuesta se puede compartir. 
* Como alternativa, cree un proxy CORS y solicite la API REST de Azure Digital Twins través de él. 

## <a name="next-steps"></a>Pasos siguientes

* Vea estos conceptos en acción en [Procedimiento: Configuración de una instancia y autenticación](how-to-set-up-instance-portal.md).

* Vea cómo interactuar con estos conceptos del código de la aplicación cliente en [Procedimiento: Escritura de código de autenticación de aplicación](how-to-authenticate-client.md).

* Obtenga más información sobre [Azure RBAC](../role-based-access-control/overview.md).