---
title: Matriz de asignación de responsabilidades de Red Hat OpenShift en Azure
description: Obtenga información acerca de la propiedad de responsabilidades para el funcionamiento de un clúster de Red Hat OpenShift en Azure
ms.service: azure-redhat-openshift
ms.topic: article
ms.date: 4/12/2021
author: sakthi-vetrivel
ms.author: suvetriv
keywords: aro, openshift, az aro, red hat, cli, RACI, soporte
ms.openlocfilehash: 477fe40dc565bf5e20627b4f4cd0d53273240ec3
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108319618"
---
# <a name="overview-of-responsibilities-for-azure-red-hat-openshift"></a>Introducción a las responsabilidades de Red Hat OpenShift en Azure

En este documento se describen las responsabilidades de Microsoft, Red Hat y los clientes para los clústeres de Red Hat OpenShift en Azure. Para obtener más información acerca de Red Hat OpenShift en Azure y sus componentes, consulte la definición del servicio Red Hat OpenShift en Azure.

Mientras Microsoft y Red Hat administran el servicio Red Hat OpenShift en Azure, el cliente comparte la responsabilidad de la funcionalidad de su clúster. Aunque los clústeres de Red Hat OpenShift en Azure se hospedan en recursos de Azure de las suscripciones a Azure del cliente, se accede a ellos de manera remota. Microsoft y Red Hat son los propietarios de la plataforma subyacente y la seguridad de datos.

## <a name="overview"></a>Información general
<table>
  <tr>
   <td><strong>Recurso</strong>
   </td>
   <td><strong><a href="#incident-and-operations-management">Administración de incidentes y operaciones</a></strong>
   </td>
   <td><strong><a href="#change-management">Administración de cambios</a></strong>
   </td>
   <td><strong><a href="#identity-and-access-management">Administración de identidades y acceso</a></strong>
   </td>
   <td><strong><a href="#security-and-regulation-compliance">Seguridad y cumplimiento normativo</a></strong>
   </td>
  </tr>
  <tr>
   <td><a href="#customer-data-and-applications">Datos del cliente</a>
   </td>
   <td>Customer </td>
   <td>Customer </td>
   <td>Customer </td>
   <td>Customer </td>
  </tr>
  <tr>
   <td><a href="#customer-data-and-applications">Aplicaciones de cliente</a>
   </td>
   <td>Customer </td>
   <td>Customer </td>
   <td>Customer </td>
   <td>Customer </td>
  </tr>
  <tr>
   <td><a href="#customer-data-and-applications">Servicios de desarrollador</a>
   </td>
   <td>Customer </td>
   <td>Customer </td>
   <td>Customer </td>
   <td>Customer </td>
  </tr>
  <tr>
   <td>Supervisión de la plataforma </td>
   <td>Microsoft y Red Hat </td>
   <td>Microsoft y Red Hat </td>
   <td>Microsoft y Red Hat </td>
   <td>Microsoft y Red Hat </td>
  </tr>
  <tr>
   <td>Registro </td>
   <td>Microsoft y Red Hat </td>
   <td>Compartido </td>
   <td>Compartido </td>
   <td>Compartido </td>
  </tr>
  <tr>
   <td>Redes de aplicación </td>
   <td>Compartido </td>
   <td>Compartido </td>
   <td>Compartido </td>
   <td>Microsoft y Red Hat </td>
  </tr>
  <tr>
   <td>Redes de clúster </td>
   <td>Microsoft y Red Hat </td>
   <td>Compartido </td>
   <td>Compartido </td>
   <td>Microsoft y Red Hat </td>
  </tr>
  <tr>
   <td>Redes virtuales </td>
   <td>Compartido </td>
   <td>Compartido </td>
   <td>Compartido </td>
   <td>Compartido </td>
  </tr>
  <tr>
   <td>Nodos del plano de control </td>
   <td>Microsoft y Red Hat </td>
   <td>Microsoft y Red Hat </td>
   <td>Microsoft y Red Hat </td>
   <td>Microsoft y Red Hat </td>
  </tr>
  <tr>
   <td>Nodos de trabajo </td>
   <td>Microsoft y Red Hat </td>
   <td>Microsoft y Red Hat </td>
   <td>Microsoft y Red Hat </td>
   <td>Microsoft y Red Hat </td>
  </tr>
  <tr>
   <td>Versión del clúster </td>
   <td>Microsoft y Red Hat </td>
   <td>Compartido </td>
   <td>Microsoft y Red Hat </td>
   <td>Microsoft y Red Hat </td>
  </tr>
  <tr>
   <td>Administración de la capacidad </td>
   <td>Microsoft y Red Hat </td>
   <td>Compartido </td>
   <td>Microsoft y Red Hat </td>
   <td>Microsoft y Red Hat </td>
  </tr>
  <tr>
   <td>Almacenamiento virtual </td>
   <td>Microsoft y Red Hat </td>
   <td>Microsoft y Red Hat </td>
   <td>Microsoft y Red Hat </td>
   <td>Microsoft y Red Hat </td>
  </tr>
  <tr>
   <td>Seguridad e infraestructura físicas </td>
   <td>Microsoft y Red Hat </td>
   <td>Microsoft y Red Hat </td>
   <td>Microsoft y Red Hat </td>
   <td>Microsoft y Red Hat </td>
  </tr>
</table>


Tabla 1. Responsabilidades por recurso


## <a name="tasks-for-shared-responsibilities-by-area"></a>Tareas de las responsabilidades compartidas por área 

### <a name="incident-and-operations-management"></a>Administración de incidentes y operaciones 

El cliente, Microsoft y Red Hat comparten la responsabilidad de la supervisión y mantenimiento de un clúster de Red Hat OpenShift en Azure. El cliente tiene la responsabilidad de llevar a cabo la administración de incidentes y operaciones de los [datos de la aplicación del cliente](#customer-data-and-applications) y de todas las redes personalizadas que el cliente pueda haber configurado.

<table>
  <tr>
   <td><strong>Recurso</strong>
   </td>
   <td><strong>Responsabilidades de Microsoft y Red Hat</strong>
   </td>
   <td><strong>Responsabilidades del cliente</strong>
   </td>
  </tr>
  <tr>
   <td>Redes de aplicación </td>
   <td>
<ul>

<li>Supervisar los equilibradores de carga en la nube y el servicio de enrutador nativo de OpenShift y responder a las alertas.
</li>
</ul>
   </td>
   <td>
<ul>

<li>Supervisar el mantenimiento de los puntos de conexión del equilibrador de carga del servicio.

<li>Supervisar el mantenimiento de las rutas de aplicación y los puntos de conexión subyacentes.

<li>Notificar interrupciones a Microsoft y Red Hat.
</li>
</ul>
   </td>
  </tr>
  <tr>
   <td>Redes virtuales
   </td>
   <td>
<ul>

<li>Supervisar los equilibradores de carga en la nube, las subredes y los componentes de nube de Azure necesarios para las redes predeterminadas de plataforma y responder a las alertas.
</li>
</ul>
   </td>
   <td>
<ul>

<li>Supervisar el tráfico de red que se configura de manera opcional a través de la conexión de red virtual a red virtual, la conexión VPN o la conexión Private Link para detectar posibles problemas o amenazas de seguridad.
</li>
</ul>
   </td>
  </tr>
</table>


Tabla 2. Responsabilidades compartidas para la administración de incidentes y operaciones


### <a name="change-management"></a>Administración de cambios

Microsoft y Red Hat tienen la responsabilidad de habilitar los cambios en la infraestructura y los servicios del clúster que controlará el cliente, así como de mantener las versiones disponibles para los nodos maestros, los servicios de infraestructura y los nodos de trabajo. El cliente tiene la responsabilidad de iniciar los cambios de infraestructura, así como de instalar y mantener servicios opcionales y configuraciones de red en el clúster, además de todos los cambios en los datos y las aplicaciones del cliente.


<table>
  <tr>
   <td><strong>Recurso</strong>
   </td>
   <td><strong>Responsabilidades de Microsoft y Red Hat</strong>
   </td>
   <td><strong>Responsabilidades del cliente</strong>
   </td>
  </tr>
  <tr>
   <td>Registro </td>
   <td>
<ul>

<li>Agregar y supervisar de manera centralizada los registros de auditoría de la plataforma.

<li>Proporcionar documentación para que el cliente habilite el registro de aplicaciones con Log Analytics a través Azure Monitor para contenedores.

<li>Proporcionar registros de auditoría cuando el cliente los solicite.
</li>
</ul>
   </td>
   <td>
<ul>

<li>Instalar el operador de registro de aplicaciones predeterminado opcional en el clúster.

<li>Instalar, configurar y mantener cualquier solución de registro de aplicaciones opcional, como el registro de contenedores sidecar o las aplicaciones de registro de terceros.

<li>Ajustar el tamaño y la frecuencia de los registros de aplicaciones que generan las aplicaciones cliente si afectan a la estabilidad del clúster.

<li>Solicitar registros de auditoría de plataforma a través de un caso de soporte técnico para investigar incidentes específicos.
</li>
</ul>
   </td>
  </tr>
  <tr>
   <td>Redes de aplicación
   </td>
   <td>
<ul>

<li>Configurar equilibradores de carga en la nube pública.

<li>Configurar el servicio de enrutador nativo de OpenShift. Proporcionar la capacidad de establecer el enrutador como privado y agregar hasta una partición de enrutador adicional.

<li>Instalar, configurar y mantener los componentes de SDN de OpenShift para el tráfico interno de pods predeterminado.
</li>
</ul>
   </td>
   <td>
<ul>

<li>Configurar los permisos no predeterminados de redes de pods para las redes de proyecto y pod, la entrada de pods y la salida de pods mediante objetos NetworkPolicy.

<li>Solicitar y configurar los equilibradores de carga de servicio adicionales para servicios específicos.
</li>
</ul>
   </td>
  </tr>
  <tr>
   <td>Redes de clúster
   </td>
   <td>
<ul>

<li>Configurar los componentes de administración de clústeres, como los puntos de conexión de servicio públicos o privados, y la integración necesaria con los componentes de la red virtual.

<li>Configurar los componentes de red internos necesarios para la comunicación interna del clúster entre los nodos maestro y de trabajo.
</li>
</ul>
   </td>
   <td>
<ul>

<li>Proporcionar intervalos de direcciones IP opcionales que no son predeterminados para CIDR de máquinas, CIDR de servicios y CIDR de pods, si es necesario, a través del administrador de clústeres de OpenShift cuando se aprovisiona el clúster.

<li>Solicitar que el punto de conexión de servicio de la API se pueda hacer público o privado durante o después de la creación del clúster mediante la CLI de Azure.
</li>
</ul>
   </td>
  </tr>
  <tr>
   <td>Redes virtuales
   </td>
   <td>
<ul>

<li>Instalar y configurar los componentes de red virtual necesarios para aprovisionar el clúster, incluidas la nube privada virtual, las subredes, los equilibradores de carga, las puertas de enlace de Internet, las puertas de enlace NAT, etc.

<li>Proporcionar al cliente la capacidad de administrar la conectividad VPN con recursos locales, la conectividad de red virtual a red virtual y la conectividad Private Link según sea necesario a través del administrador de clústeres de OpenShift.

<li>Permitir a los clientes la creación e implementación de equilibradores de carga en la nube pública para su uso con equilibradores de carga de servicio.
</li>
</ul>
   </td>
   <td>
<ul>

<li>Configurar y mantener componentes de red de nube pública opcionales, como la conexión de red virtual a red virtual, la conexión VPN o la conexión Private Link.

<li>Solicitar y configurar los equilibradores de carga de servicio adicionales para servicios específicos.
</li>
</ul>
   </td>
  </tr>
  <tr>
   <td>Versión del clúster
   </td>
   <td>
<ul>

<li>Comunicar la programación y el estado de las actualizaciones para las versiones secundarias y de mantenimiento.

<li>Publicar los registros de cambios y las notas de la versión para las actualizaciones secundarias y de mantenimiento.
</li>
</ul>
   </td>
   <td>
<ul>

<li>Iniciar la actualización del clúster.

<li>Probar las aplicaciones de cliente en las versiones secundarias y de mantenimiento para garantizar la compatibilidad.
</li>
</ul>
   </td>
  </tr>
  <tr>
   <td>Administración de la capacidad
   </td>
   <td>
<ul>

<li>Supervisar el uso del plano de control (nodos maestros).

<li>Escalar o cambiar el tamaño de los nodos del plano de control para mantener la calidad del servicio.

<li>Supervisar el uso de los recursos del cliente, incluida la capacidad de red, almacenamiento y proceso. Cuando las características de escalabilidad automática no están habilitadas, se alerta al cliente de los cambios necesarios en los recursos del clúster (por ejemplo, nuevos nodos de proceso para escalar, almacenamiento adicional, etc.).
</li>
</ul>
   </td>
   <td>
<ul>

<li>Usar los controles proporcionados del administrador de clústeres de OpenShift para agregar o quitar nodos de trabajo adicionales según sea necesario.

<li>Responder a las notificaciones de Microsoft y Red Hat relativas a los requisitos de recursos del clúster.
</li>
</ul>
   </td>
  </tr>
</table>


Tabla 3. Responsabilidades compartidas de la administración de cambios


### <a name="identity-and-access-management"></a>Administración de identidades y acceso 

La administración de identidades y acceso incluye todas las responsabilidades para garantizar que solo las personas adecuadas accedan a los recursos de clústeres, aplicaciones e infraestructura. Esto incluye tareas como proporcionar mecanismos de control de acceso, autenticación, autorización y administración del acceso a los recursos.


<table>
  <tr>
   <td><strong>Recurso</strong>
   </td>
   <td><strong>Responsabilidades de Microsoft y Red Hat</strong>
   </td>
   <td><strong>Responsabilidades del cliente</strong>
   </td>
  </tr>
  <tr>
   <td>Registro </td>
   <td>
<ul>

<li>Seguir un proceso de acceso interno en capas basado en estándares del sector para los registros de auditoría de la plataforma.

<li>Proporcionar funcionalidades nativas de OpenShift RBAC.
</li>
</ul>
   </td>
   <td>
<ul>

<li>Configurar OpenShift RBAC para controlar el acceso a los proyectos y, por extensión, a los registros de aplicación de un proyecto.

<li>En el caso de las soluciones de registro de aplicaciones personalizadas o de terceros, el cliente se encarga de la administración del acceso.
</li>
</ul>
   </td>
  </tr>
  <tr>
   <td>Redes de aplicación
   </td>
   <td>
<ul>

<li>Proporcionar funcionalidades nativas de OpenShift RBAC.
</li>
</ul>
   </td>
   <td>
<ul>

<li>Configurar OpenShift RBAC para controlar el acceso a la configuración de enrutamiento según sea necesario.
</li>
</ul>
   </td>
  </tr>
  <tr>
   <td>Redes de clúster
   </td>
   <td>
<ul>

<li>Proporcionar funcionalidades nativas de OpenShift RBAC.
</li>
</ul>
   </td>
   <td>
<ul>

<li>Administrar la pertenencia a la organización de Red Hat de las cuentas de Red Hat.

<li>Administrar a los administradores de la organización para la organización de Red Hat a fin de conceder acceso al administrador de clústeres de OpenShift.

<li>Configurar OpenShift RBAC para controlar el acceso a la configuración de enrutamiento según sea necesario.
</li>
</ul>
   </td>
  </tr>
  <tr>
   <td>Redes virtuales
   </td>
   <td>
<ul>

<li>Proporcionar controles de acceso al cliente a través del administrador de clústeres de OpenShift.
</li>
</ul>
   </td>
   <td>
<ul>

<li>Administrar el acceso de usuario opcional a los componentes de la nube pública mediante el administrador de clústeres de OpenShift.
</li>
</ul>
   </td>
  </tr>
</table>


Tabla 4. Responsabilidades compartidas para la administración de identidades y acceso


### <a name="security-and-regulation-compliance"></a>Seguridad y cumplimiento normativo 

La seguridad y el cumplimiento incluyen todas las responsabilidades y controles que garantizan el cumplimiento de las legislaciones, directivas y regulaciones pertinentes.


<table>
  <tr>
   <td><strong>Recurso</strong>
   </td>
   <td><strong>Responsabilidades de Microsoft y Red Hat</strong>
   </td>
   <td><strong>Responsabilidades del cliente</strong>
   </td>
  </tr>
  <tr>
   <td>Registro </td>
   <td>
<ul>

<li>Enviar registros de auditoría de clúster a una instancia de SIEM de Microsoft y Red Hat para analizar los eventos de seguridad. Conservar los registros de auditoría durante un período de tiempo definido para ayudar a los análisis forenses.
</li>
</ul>
   </td>
   <td>
<ul>

<li>Analizar los registros de aplicaciones en busca de eventos de seguridad. Enviar registros de aplicación a un punto de conexión externo a través de contenedores sidecar de registro o aplicaciones de registro de terceros si se requiere una retención más prolongada que la que ofrece la pila de registro predeterminada.
</li>
</ul>
   </td>
  </tr>
  <tr>
   <td>Redes virtuales
   </td>
   <td>
<ul>

<li>Supervise los componentes de red virtual en busca de posibles problemas y amenazas de seguridad.

<li>Usar herramientas públicas de Azure de Microsoft y Red Hat adicionales para una supervisión y protección adicionales.
</li>
</ul>
   </td>
   <td>
<ul>

<li>Supervisar los componentes de red virtual configurados de manera opcional en busca de posibles problemas y amenazas de seguridad.

<li>Configurar las reglas de firewall necesarias o las protecciones del centro de datos según sea necesario.
</li>
</ul>
   </td>
  </tr>
</table>


Tabla 5. Responsabilidades compartidas de seguridad y cumplimiento normativo


## <a name="customer-responsibilities-when-using-azure-red-hat-openshift"></a>Responsabilidades del cliente al usar Red Hat OpenShift en Azure 


### <a name="customer-data-and-applications"></a>Aplicaciones y datos de clientes

El cliente se encarga de las aplicaciones, cargas de trabajo y datos que implementa en Red Hat OpenShift en Azure. Sin embargo, Microsoft y Red Hat proporcionan diversas herramientas para ayudar al cliente a administrar los datos y aplicaciones en la plataforma.


<table>
  <tr>
   <td><strong>Recurso</strong>
   </td>
   <td><strong>Cómo ayudan Microsoft y Red Hat</strong>
   </td>
   <td><strong>Responsabilidades del cliente</strong>
   </td>
  </tr>
  <tr>
   <td>Datos del cliente </td>
   <td>
<ul>

<li>Mantenga los estándares de nivel de plataforma para el cifrado de datos tal y como se definen en los estándares de seguridad y cumplimiento del sector. 

<li>Proporcionar componentes de OpenShift para ayudar a administrar los datos de la aplicación, como los secretos.

<li>Habilitar la integración con servicios de datos de terceros (como Azure SQL) para almacenar y administrar datos fuera del clúster o de Microsoft y Red Hat Azure.
</li>
</ul>
   </td>
   <td>
<ul>

<li>Conservar la responsabilidad de todos los datos de cliente almacenados en la plataforma y de cómo las aplicaciones de cliente consumen y exponen estos datos.

<li>Cifrado etcd
</li>
</ul>
   </td>
  </tr>
  <tr>
   <td>Aplicaciones de cliente
   </td>
   <td>
<ul>

<li>Aprovisionar clústeres que tienen componentes de OpenShift instalados para que los clientes puedan acceder a las API de OpenShift y Kubernetes a fin de implementar y administrar aplicaciones contenedorizadas.

<li>Proporcionar acceso a las API de OpenShift que un cliente puede usar para configurar operadores a fin de agregar servicios de Red Hat, de comunidades, de terceros y de Microsoft y Red Hat al clúster. 

<li>Proporcionar clases de almacenamiento y complementos para admitir volúmenes persistentes para usarlos con aplicaciones de cliente.
</li>
</ul>
   </td>
   <td>
<ul>

<li>Conservar la responsabilidad de las aplicaciones de terceros y de clientes, así como sus datos y ciclo de vida completo.

<li>Si un cliente agrega servicios de Red Hat, de comunidades, de terceros, propios o de otro tipo al clúster mediante operadores o imágenes externas, el cliente es responsable de estos servicios y de trabajar con el proveedor adecuado (incluido Red Hat) para solucionar cualquier problema.

<li>Usar las herramientas y características proporcionadas para <a href="https://docs.openshift.com/aro/4/architecture/understanding-development.html#application-types">configurar e implementar aplicaciones</a>; <a href="https://docs.openshift.com/aro/4/applications/deployments/deployment-strategies.html">mantenerlas actualizadas</a>; <a href="https://docs.openshift.com/dedicated/4/applications/working-with-quotas.html">configurar las solicitudes de recursos y límites</a>; <a href="https://docs.openshift.com/dedicated/4/getting_started/scaling-your-cluster.html">cambiar el tamaño del clúster a fin de tener suficientes recursos para ejecutar las aplicaciones</a>; <a href="https://docs.openshift.com/dedicated/4/administering_a_cluster/cluster-admin-role.html">configurar los permisos</a>; integrar las aplicaciones con otros servicios; <a href="https://docs.openshift.com/aro/4/openshift_images/images-understand.html">administrar cualquier flujo de imagen o plantillas que el cliente implemente</a>; <a href="https://docs.openshift.com/dedicated/4/cloud_infrastructure_access/dedicated-understanding-aws.html">servir externamente</a>; guardar, hacer copias de seguridad y restaurar datos; y administrar de cualquier otro modo las cargas de trabajo resistentes y de alta disponibilidad de las aplicaciones.

<li>Conservar la responsabilidad de supervisar las aplicaciones que se ejecutan en Red Hat OpenShift en Azure; incluida la instalación y el funcionamiento de software para recopilar métricas y crear alertas.
</li>
</ul>
   </td>
  </tr>
</table>


Tabla 7. Responsabilidades del cliente con los datos de los clientes, las aplicaciones de cliente y los servicios
