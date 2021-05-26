---
title: Introducción a la seguridad
titleSuffix: Azure Cognitive Search
description: Más información sobre las características de seguridad de Azure Cognitive Search para proteger los puntos de conexión, el contenido y las operaciones.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 02/04/2021
ms.custom: references_regions
ms.openlocfilehash: d4310c724707548cd55df5591504736d8f3c3857
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110460058"
---
# <a name="security-overview-for-azure-cognitive-search"></a>Información general de seguridad para Azure Cognitive Search

En este artículo se describen las características de seguridad de Azure Cognitive Search que protegen el contenido y las operaciones.

En el caso de las solicitudes de entrada realizadas a un servicio de búsqueda, existe una progresión de las medidas de seguridad que protegen el punto de conexión del servicio de búsqueda: desde las claves de API de la solicitud hasta las reglas de entrada del firewall o los puntos de conexión privados que protegen totalmente el servicio de la red pública de Internet.

En el caso de las solicitudes salientes realizadas a otros servicios, la solicitud predominante la realizan los indizadores que leen el contenido de orígenes externos. Puede proporcionar credenciales en la cadena de conexión. O bien, puede configurar una identidad administrada para que busque en un servicio de confianza al acceder a los datos de Azure Storage, Azure SQL, Cosmos DB u otros orígenes de datos de Azure. Una identidad administrada es un sustituto de las credenciales o las claves de acceso en la conexión. Para obtener más información acerca de esta capacidad, vea [Configuración de una conexión de indexador a un origen de datos mediante una identidad administrada](search-howto-managed-identities-data-sources.md).

Las operaciones de escritura en servicios externos son pocas: un servicio de búsqueda escribe en los archivos de registro y escribirá en Azure Storage al crear almacenes de conocimiento, conservar los enriquecimientos almacenados en caché y conservar las sesiones de depuración. Otras llamadas entre servicios, como Cognitive Services, se realizan en la red interna.

Vea este vídeo de resumen para obtener información general sobre la arquitectura de seguridad y cada categoría de características.

> [!VIDEO https://channel9.msdn.com/Shows/AI-Show/Azure-Cognitive-Search-Whats-new-in-security/player]

## <a name="network-security"></a>Seguridad de red

<a name="service-access-and-authentication"></a>

Las características de seguridad de entrada protegen el punto de conexión del servicio de búsqueda a través de niveles crecientes de seguridad y complejidad. En primer lugar, todas las solicitudes requieren una clave de API para el acceso autenticado. En segundo lugar, puede establecer opcionalmente las reglas de firewall que limitan el acceso a direcciones IP específicas. Para la protección avanzada, una tercera opción consiste en habilitar Azure Private Link para proteger el punto de conexión de servicio de todo el tráfico de Internet.

### <a name="public-access-using-api-keys"></a>Acceso público mediante claves de API

De forma predeterminada, se tiene acceso a un servicio de búsqueda a través de la nube pública mediante la autenticación basada en claves para el acceso de administrador o de consulta al punto de conexión del servicio de búsqueda. El envío de una clave válida se considera una prueba de que la solicitud se origina desde una entidad de confianza. La autenticación basada en claves se explica en la sección siguiente.

### <a name="configure-ip-firewalls"></a>Configuración de firewalls de IP

Para controlar aún más el acceso al servicio de búsqueda, puede crear reglas de firewall de entrada que permitan el acceso a una dirección IP específica o a un intervalo de direcciones IP. Todas las conexiones de cliente deben realizarse a través de una dirección IP permitida, o la conexión se denegará.

:::image type="content" source="media/search-security-overview/inbound-firewall-ip-restrictions.png" alt-text="Ejemplo de diagrama de arquitectura para el acceso restringido de IP":::

Puede usar el portal para [configurar el acceso de entrada](service-configure-firewall.md).

Como alternativa, puede usar las API REST de administración. Desde la versión de API 2020-03-13, con el parámetro [IpRule](/rest/api/searchmanagement/services/createorupdate#iprule), puede restringir el acceso a su servicio mediante la identificación (de forma individual o en un intervalo) de las direcciones IP a las que quiera otorgar acceso a su servicio de búsqueda.

### <a name="network-isolation-through-a-private-endpoint-no-internet-traffic"></a>Aislamiento de red a través de un punto de conexión privado (sin tráfico de Internet)

Puede establecer un [punto de conexión privado](../private-link/private-endpoint-overview.md) para Azure Cognitive Search, lo que permite a un cliente de una [red virtual](../virtual-network/virtual-networks-overview.md) obtener acceso de forma segura a los datos de un índice de búsqueda a través de una instancia de [Private Link](../private-link/private-link-overview.md).

El punto de conexión privado usa una dirección IP del espacio de direcciones de la red virtual para las conexiones al servicio de búsqueda. El tráfico de red entre el cliente y el servicio de búsqueda atraviesa la red virtual y un vínculo privado de la red troncal de Microsoft, lo que elimina la exposición a la red pública de Internet. Una red virtual permite establecer una comunicación segura entre recursos, con su red local y con Internet.

:::image type="content" source="media/search-security-overview/inbound-private-link-azure-cog-search.png" alt-text="Diagrama de la arquitectura de ejemplo para el acceso de punto de conexión privado":::

Aunque esta solución es la más segura, el uso de servicios adicionales supone un costo adicional, por lo que debe comprender claramente las ventajas antes de profundizar en ella. Para más información sobre los costos, consulte la [página de precios](https://azure.microsoft.com/pricing/details/private-link/). Para obtener más información sobre cómo funcionan conjuntamente estos componentes, vea el vídeo que se encuentra en la parte superior de este artículo. La opción de punto de conexión privado se trata a partir del minuto 5:48 del vídeo. Para obtener instrucciones sobre cómo configurar el punto de conexión, consulte [Creación de un punto de conexión privado para una conexión segura a Azure Cognitive Search](service-create-private-endpoint.md).

## <a name="authentication"></a>Authentication

En el caso de las solicitudes de entrada al servicio de búsqueda, la autenticación se realiza a través de un [clave de API obligatoria](search-security-api-keys.md) (una cadena formada por números y letras generados aleatoriamente) que demuestra que la solicitud proviene de un origen de confianza. Cognitive Search no admite actualmente la autenticación de Azure Active Directory para las solicitudes de entrada.

Las solicitudes de salida realizadas por un indizador están sujetas a la autenticación por parte del servicio externo. El subservicio del indizador de Cognitive Search se puede convertir en un servicio de confianza en Azure y conectarse a otros servicios mediante una identidad administrada. Para obtener más información, consulte [Configuración de una conexión de indexador a un origen de datos mediante una identidad administrada](search-howto-managed-identities-data-sources.md).

## <a name="authorization"></a>Autorización

Cognitive Search proporciona diferentes modelos de autorización para la administración de contenido y la administración de servicios. 

### <a name="authorization-for-content-management"></a>Autorización para la administración de contenido

La autorización para el contenido y las operaciones relacionadas con el contenido son de acceso de escritura, tal y como se confiere a través de la [clave de API](search-security-api-keys.md) proporcionada en la solicitud. La clave de API es un mecanismo de autenticación, pero también autoriza el acceso según el tipo de clave de API.

+ Clave de administración (permite el acceso de lectura y escritura para operaciones de creación, lectura, actualización y eliminación en el servicio de búsqueda) creada al aprovisionar el servicio.

+ Clave de consulta (permite el acceso de solo lectura a la colección de documentos de un índice) creada según la necesidad y diseñada para las aplicaciones cliente que emiten consultas.

En el código de la aplicación, especifique el punto de conexión y una clave de API para permitir el acceso al contenido y las opciones. Un punto de conexión puede ser el propio servicio, la colección de índices, un índice específico, una colección de documentos o un documento específico. Cuando se encadenan juntos, el punto de conexión, la operación (por ejemplo, una solicitud de creación o actualización) y el nivel de permiso (derechos completos o de solo lectura basados en la clave) constituyen la fórmula de seguridad que protege el contenido y las operaciones.

### <a name="controlling-access-to-indexes"></a>Control del acceso a los índices

En Azure Cognitive Search, un índice individual no es un objeto protegible. En su lugar, el acceso a un índice se determina en la capa de servicio (acceso de lectura o escritura según la clave de API que proporcione), junto con el contexto de una operación.

Para el acceso de solo lectura, puede estructurar las solicitudes de consulta para conectarse con una [clave de consulta](search-security-rbac.md) e incluir el índice específico utilizado por la aplicación. En una solicitud de consulta, no existe el concepto de combinación de índices ni de acceso simultáneo a varios índices para que todas las solicitudes tengan un único índice de destino por definición. Por lo tanto, la construcción de la solicitud de consulta misma (una clave y un índice único de destino) define el límite de seguridad.

Los accesos de administrador y de desarrollador a los índices no se diferencian: ambos necesitan tener acceso de escritura para crear, eliminar y actualizar objetos administrados por el servicio. Cualquier persona con una [clave de administración](search-security-rbac.md) del servicio puede leer, modificar o eliminar cualquier índice en el mismo servicio. Para la protección contra la eliminación accidental o malintencionada de índices, su control de código fuente interno para los recursos de código es la solución para revertir una eliminación o modificación de índices no deseada. Azure Cognitive Search incluye conmutación por error dentro del clúster para garantizar la disponibilidad, pero no almacena ni ejecuta el código propietario utilizado para crear o cargar los índices.

Para soluciones multiinquilino que requieren límites de seguridad en el nivel de índice, estas soluciones suelen incluir un nivel intermedio que los clientes utilizan para controlar el aislamiento de índices. Para más información sobre los casos de uso de varios inquilinos, consulte [Modelos de diseño de aplicaciones SaaS para varios inquilinos y Azure Cognitive Search](search-modeling-multitenant-saas-applications.md).

### <a name="controlling-access-to-documents"></a>Control del acceso a los documentos

Si necesita tener un control por usuario pormenorizado de los resultados de la búsqueda, puede crear filtros de seguridad en las consultas, que devuelven documentos asociados con una identidad de seguridad determinada. 

Conceptualmente equivalente a "seguridad de nivel de fila", la autorización de contenido en el índice no se admite de forma nativa mediante roles predefinidos o asignaciones de roles que se asignan a entidades en Azure Active Directory. Los permisos de usuario para datos de sistemas externos, como Cosmos DB, no se transfieren con esos datos cuando Cognitive Search los indexa.

Las alternativas para las soluciones que requieren "seguridad de nivel de fila" incluyen la creación de un campo en el origen de datos que representa un grupo de seguridad o una identidad de usuario y, a continuación, el uso de filtros en Cognitive Search para recortar de forma selectiva los resultados de la búsqueda de documentos y contenido en función de las identidades. La tabla siguiente describe dos enfoques para recortar el contenido no autorizado de los resultados de la búsqueda.

| Enfoque | Descripción |
|----------|-------------|
|[Recorte de seguridad basado en filtros de identidad](search-security-trimming-for-azure-search.md)  | Documenta el flujo de trabajo básico para implementar el control de acceso de identidades de usuario. Trata la incorporación de identificadores de seguridad a un índice y, a continuación, se explica el filtrado por ese campo para no incluir el contenido prohibido en los resultados. |
|[Recorte de seguridad basado en identidades de Azure Active Directory](search-security-trimming-for-azure-search-with-aad.md)  | En este artículo se amplía el artículo anterior y se proporcionan los pasos necesarios para recuperar identidades de Azure Active Directory (Azure AD), uno de los [servicios gratuitos](https://azure.microsoft.com/free/) de la plataforma de nube de Azure. |

### <a name="authorization-for-service-management"></a>Autorización para la administración de servicios

Las operaciones de administración de servicios se autorizan mediante el [control de acceso basado en rol de Azure (RBAC de Azure)](../role-based-access-control/overview.md). RBAC de Azure es un sistema de autorización basado en [Azure Resource Manager](../azure-resource-manager/management/overview.md) para el aprovisionamiento de recursos de Azure. 

En Azure Cognitive Search, Resource Manager se usa para crear o eliminar el servicio, administrar las claves de API y escalar el servicio. Como tal, las asignaciones de roles de Azure determinan quién puede realizar esas tareas, independientemente de si se usa el [portal](search-manage.md), [PowerShell](search-manage-powershell.md) o las [API de REST de administración](/rest/api/searchmanagement/search-howto-management-rest-api).

Se han definido [tres roles básicos](search-security-rbac.md) para la administración de servicios de búsqueda. Las asignaciones de roles se pueden realizar con cualquier metodología compatible (portal, PowerShell, etc.) y se respetan en todo el servicio. Los roles Propietario y Colaborador pueden realizar diversas funciones de administración. Puede asignar el rol Lector a los usuarios que solo ven información esencial.

> [!Note]
> Mediante el uso de mecanismos de aplicación en todo el sistema de Azure, puede bloquear una suscripción o un recurso para evitar la eliminación accidental o no autorizada del servicio de búsqueda por parte de usuarios con derechos de administrador. Para más información, consulte [Bloqueo de recursos para impedir eliminación inesperada](../azure-resource-manager/management/lock-resources.md).

<a name="encryption"></a>

## <a name="data-protection"></a>Protección de datos

En la capa de almacenamiento, el cifrado de datos está integrado en todo el contenido administrado por el servicio guardado en el disco, incluidos los índices, los mapas de sinónimos y las definiciones de indizadores, orígenes de datos y conjuntos de aptitudes. Opcionalmente, puede agregar claves administradas por el cliente (CMK) para el cifrado complementario del contenido indexado. En el caso de los servicios creados después del 1 de agosto de 2020, el cifrado de CMK se extiende a los datos de los discos temporales, para el cifrado doble completo del contenido indexado.

### <a name="data-in-transit"></a>Datos en tránsito

En Azure Cognitive Search, el cifrado empieza por las conexiones y las transmisiones y se extiende al contenido almacenado en el disco. En el caso de los servicios de búsqueda en la red pública de Internet, Azure Cognitive Search escucha en el puerto HTTPS 443. Todas las conexiones de cliente a servicio usan el cifrado TLS 1.2. Las versiones anteriores (1.0 o 1.1) no se admiten.

### <a name="encrypted-data-at-rest"></a>Datos cifrados en reposo

En el caso de los datos que administra internamente el servicio de búsqueda, en la tabla siguiente se describen los [modelos de cifrado de datos](../security/fundamentals/encryption-models.md). Algunas características, como el almacén de conocimiento, el enriquecimiento incremental y la indización basada en indizador, leen o escriben en estructuras de datos de otros servicios de Azure. Esos servicios tienen sus propios niveles de compatibilidad de cifrado independientes de Azure Cognitive Search.

| Modelo | Claves&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | Requisitos&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | Restricciones | Se aplica a |
|------------------|-------|-------------|--------------|------------|
| Cifrado del servidor | Claves administradas por Microsoft | Ninguno (integrados) | Ninguna, disponible en todos los niveles, en todas las regiones, para el contenido creado después del 24 de enero de 2018. | Contenido (índices y mapas de sinónimos) y definiciones (indizadores, orígenes de datos, conjuntos de aptitudes) |
| Cifrado del servidor | Claves administradas por el cliente | Azure Key Vault | Disponible en niveles facturables, en todas las regiones, para el contenido creado después de enero de 2019. | Contenido (índices y mapas de sinónimos) en discos de datos |
| Cifrado doble del servidor | Claves administradas por el cliente | Azure Key Vault | Disponible en niveles facturables, en regiones seleccionadas, en servicios de búsqueda después del 1 de agosto de 2020. | Contenido (índices y mapas de sinónimos) en discos de datos y discos temporales |

### <a name="service-managed-keys"></a>Claves administradas por el servicio

El cifrado administrado por el servicio es una operación interna de Microsoft que se basa en [Azure Storage Service Encryption](../storage/common/storage-service-encryption.md) con [cifrado AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) de 256 bits. Se produce automáticamente en todas las indizaciones, incluidas las actualizaciones incrementales a índices que no están totalmente cifrados (creados antes de enero de 2018).

### <a name="customer-managed-keys-cmk"></a>Claves administradas por el cliente (CMK)

Las claves administradas por el cliente requieren un servicio facturable adicional, Azure Key Vault, que puede estar en otra región, pero en la misma suscripción que Azure Cognitive Search. Al habilitar el cifrado de CMK aumenta el tamaño de los índices y empeora el rendimiento de las consultas. Según las observaciones hechas hasta la fecha, puede esperar un aumento del 30 al 60% en los tiempos de consultas, aunque el rendimiento real variará según la definición del índice y los tipos de consultas. Debido a este impacto en el rendimiento, se recomienda habilitar esta característica solo en los índices que realmente la necesitan. Para obtener más información, vea [Configuración de claves administradas por el cliente para el cifrado de datos en Azure Cognitive Search](search-security-manage-encryption-keys.md).

<a name="double-encryption"></a>

### <a name="double-encryption"></a>Cifrado doble

En Azure Cognitive Search, el cifrado doble es una extensión de CMK. Se entiende que se trata de un cifrado doble (una vez mediante CMK y otra mediante claves administradas por el servicio) y de ámbito general, que engloba el almacenamiento a largo plazo que se escribe en un disco de datos y el almacenamiento a corto plazo escrito en discos temporales. La diferencia entre CMK antes del 1 de agosto de 2020 y después, y lo que convierte a CMK en una característica de cifrado doble en Azure Cognitive Search, es el cifrado adicional de datos en reposo en discos temporales.

El cifrado doble está disponible actualmente en los nuevos servicios creados en estas regiones después del 1 de agosto:

+ Oeste de EE. UU. 2
+ Este de EE. UU.
+ Centro-sur de EE. UU.
+ US Gov - Virginia
+ US Gov: Arizona

## <a name="security-management"></a>Administración de la seguridad

### <a name="api-keys"></a>claves de API

La dependencia de la autenticación basada en claves de API significa que debe tener un plan para volver a generar la clave de administración a intervalos regulares, según los procedimientos recomendados de seguridad de Azure. Hay un máximo de dos claves de administrador por servicio de búsqueda. Para más información acerca de protección y administración de claves de API, vea [Creación y administración de claves de API](search-security-api-keys.md).

#### <a name="activity-and-diagnostic-logs"></a>Registros de actividad y diagnóstico

Cognitive Search no registra las identidades de usuario, por lo que no puede hacer referencia a los registros para obtener información sobre un usuario específico. Sin embargo, el servicio registra las operaciones de creación, lectura, actualización y eliminación, que es posible que pueda correlacionar con otros registros para comprender la intervención de acciones específicas.

Por medio de alertas y de la infraestructura de registro de Azure, puede seleccionar picos de volumen de consultas u otras acciones que se desvían de las cargas de trabajo previstas. Para obtener más información sobre la configuración de registros, vea [Recopilación y análisis de datos de registro](search-monitor-logs.md) y [Supervisión de solicitudes de consulta](search-monitor-queries.md).

### <a name="certifications-and-compliance"></a>Certificaciones y cumplimiento

Azure Cognitive Search participa en auditorías regulares, y tiene la certificación de varios estándares globales, regionales y específicos del sector para la nube pública y Azure Government. Para obtener la lista completa, descargue las notas del producto de las [**ofertas de cumplimiento de** Microsoft Azure](https://azure.microsoft.com/resources/microsoft-azure-compliance-offerings/).

En el caso del cumplimiento, puede usar [Azure Policy](../governance/policy/overview.md) para implementar los procedimientos recomendados de alta seguridad de [Azure Security Benchmark](../security/benchmarks/introduction.md). Azure Security Benchmark es una colección de recomendaciones de seguridad codificadas en controles de seguridad que se asignan a acciones clave que se deben realizar para mitigar las amenazas a los servicios y los datos. Actualmente hay 11 controles de seguridad, entre los que se incluyen [Seguridad de red](../security/benchmarks/security-control-network-security.md), [Registro y supervisión](../security/benchmarks/security-control-logging-monitoring.md) y [Protección de datos](../security/benchmarks/security-control-data-protection.md), por nombrar algunos.

Azure Policy es una capacidad integrada en Azure que ayuda a administrar el cumplimiento de varios estándares, incluidos los de Azure Security Benchmark. En el caso de los bancos de pruebas conocidos, Azure Policy proporciona definiciones integradas que ofrecen tanto criterios como una respuesta accionable que aborda el no cumplimiento.

En Azure Cognitive Search, actualmente hay una definición integrada. Es para el registro de diagnóstico. Con ella integrada, puede asignar una directiva que identifique cualquier servicio de búsqueda que no tenga registro de diagnóstico y lo active. Para obtener más información, vea [Controles de Cumplimiento normativo de Azure Policy para Azure Cognitive Search](security-controls-policy.md).

## <a name="see-also"></a>Consulte también

+ [Aspectos básicos de la seguridad de Azure](../security/fundamentals/index.yml)
+ [Azure Security](https://azure.microsoft.com/overview/security)
+ [Azure Security Center](../security-center/index.yml)