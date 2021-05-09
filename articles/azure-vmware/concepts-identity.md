---
title: Conceptos sobre identidad y acceso
description: Obtenga información sobre los conceptos de identidad y acceso de Azure VMware Solution
ms.topic: conceptual
ms.date: 03/22/2021
ms.openlocfilehash: 352daaec87c2df7a0bc062abd38a96ad0285180a
ms.sourcegitcommit: 2e123f00b9bbfebe1a3f6e42196f328b50233fc5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108069838"
---
# <a name="azure-vmware-solution-identity-concepts"></a>Conceptos de identidad en Azure VMware Solution

Las nubes privadas de Azure VMware Solution se aprovisionan con vCenter Server y NSX-T Manager. Puede usar vCenter para administrar cargas de trabajo de máquinas virtuales (VM), y NSX-T Manager para administrar y ampliar la nube privada. En la administración de identidades y acceso se usa el rol CloudAdmin para vCenter y derechos de administrador restringidos para NSX-T Manager. 

## <a name="vcenter-access-and-identity"></a>Acceso e identidad de vCenter

En Azure VMware Solution, vCenter tiene un usuario local integrado denominado cloudadmin que está asignado al rol CloudAdmin. El usuario cloudadmin local se usa para configurar usuarios en Active Directory (AD). En general, el rol CloudAdmin crea y administra las cargas de trabajo en la nube privada. En cambio, en Azure VMware Solution, el rol CloudAdmin tiene privilegios de vCenter que son distintos de otras soluciones en la nube de VMware.     

- En una implementación local de vCenter y ESXi, el administrador tiene acceso a la cuenta de administrador de vCenter\@vsphere.local. También se pueden tener más usuarios y grupos de AD asignados. 

- En una implementación de Azure VMware Solution, el administrador no tiene acceso a la cuenta de usuario de administrador. Sin embargo, puede asignar usuarios y grupos de AD al rol CloudAdmin en vCenter.  

El usuario de la nube privada no tiene acceso a componentes de administración específicos admitidos y administrados por Microsoft ni puede configurarlos. Por ejemplo, clústeres, hosts, almacenes de datos y conmutadores virtuales distribuidos.

> [!IMPORTANT]
> Azure VMware Solution ofrece roles personalizados en vCenter, pero actualmente no los ofrece en el portal de Azure VMware Solution. Para obtener más información, consulte la sección [Creación de roles personalizados en vCenter](#create-custom-roles-on-vcenter) más adelante en este artículo. 

### <a name="view-the-vcenter-privileges"></a>Ver los privilegios de vCenter

Puede ver los privilegios concedidos al rol CloudAdmin de Azure VMware Solution en su instancia de vCenter de la nube privada de Azure VMware Solution.

1. Inicie sesión en el cliente de SDDC vSphere y vaya a **Menú** > **Administración**.
1. Mire en **Control de acceso**, seleccione **Roles**.
1. En la lista de roles, seleccione **CloudAdmin** y luego, seleccione **Privilegios**. 

   :::image type="content" source="media/role-based-access-control-cloudadmin-privileges.png" alt-text="Cómo ver los privilegios de rol de CloudAdmin en el cliente de vSphere":::

El rol CloudAdmin de Azure VMware Solution tiene los siguientes privilegios en vCenter. Para más información, consulte la [documentación del producto de VMware](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.security.doc/GUID-ED56F3C4-77D0-49E3-88B6-B99B8B437B62.html).

| Privilegio | Descripción |
| --------- | ----------- |
| **Alarmas** | Confirmar alarma<br />Crear alarma<br />Deshabilitar acción de alarma<br />Modificar alarma<br />Eliminar alarma<br />Establecer estado de la alarma |
| **Biblioteca de contenido** | Agregar elemento de biblioteca<br />Crear una suscripción para una biblioteca publicada<br />Crear biblioteca local<br />Crear biblioteca suscrita<br />Eliminar elemento de biblioteca<br />Eliminar biblioteca local<br />Eliminar biblioteca suscrita<br />Eliminar la suscripción de una biblioteca publicada<br />Descarga de archivos<br />Expulsar elementos de biblioteca<br />Desalojar biblioteca suscrita<br />Importar almacenamiento<br />Sondear información de suscripción<br />Publicar un elemento de biblioteca en sus suscriptores<br />Publicar una biblioteca en sus suscriptores<br />Leer en el almacenamiento<br />Sincronizar elemento de biblioteca<br />Sincronizar biblioteca suscrita<br />Escribir introspección<br />Actualizar valores de configuración<br />Actualizar archivos<br />Actualizar biblioteca<br />Actualizar elemento de biblioteca<br />Actualizar biblioteca local<br />Actualizar biblioteca suscrita<br />Actualizar la suscripción de una biblioteca publicada<br />Ver valores de configuración |
| **Operaciones criptográficas** | Acceso directo |
| **Almacén de datos** | Asignar espacio<br />Examen de un almacén de datos<br />Configurar almacén de datos<br />Operaciones de archivo de bajo nivel<br />Quitar archivos<br />Actualizar metadatos de máquina virtual |
| **Carpeta** | Crear carpeta<br />Eliminar carpeta<br />Mover carpeta<br />Cambiar nombre de la carpeta |
| **Global** | Cancelar tarea<br />Etiqueta global<br />Health<br />Registrar eventos<br />Administrar atributos personalizados<br />Administradores de servicio<br />Establecer atributo personalizado<br />Etiqueta del sistema |
| **Host** | Replica de vSphere<br />&#160;&#160;&#160;&#160;Administrar la replicación |
| **Network** | Asignar red |
| **Permisos** | Modificar permisos<br />Modificar rol |
| **Perfil** | Vista de almacenamiento controlado por perfiles |
| **Recurso** | Aplicar recomendaciones<br />Asignar vApp a un grupo de recursos<br />Asignar máquina virtual a grupo de recursos<br />Crear grupo de recursos<br />Migrar máquina virtual apagada<br />Migrar máquina virtual encendida<br />Modificar grupo de recursos<br />Mover grupo de recursos<br />Consultar vMotion<br />Eliminar grupo de recursos<br />Cambiar nombre del grupo de recursos |
| **Tarea programada** | Crear la tarea<br />Modificar tarea<br />Quitar tarea<br />Ejecutar tarea |
| **Sesiones** | Message<br />Validar sesión |
| **Vista de almacenamiento** | Ver |
| **vApp** | Agregar máquina virtual<br />Asignar grupo de recursos<br />Asignar vApp<br />Clonar<br />Crear<br />Eliminar<br />Exportación<br />Importar<br />Move<br />Apagado<br />Encendido<br />Cambiar nombre<br />Suspender<br />Unregister<br />Ver el entorno de OVF<br />Configuración de aplicaciones de vApp<br />Configuración de instancias de vApp<br />Configuración de managedBy de vApp<br />Configuración de recursos de vApp |
| **Máquina virtual** | Cambiar configuración<br />&#160;&#160;&#160;&#160;Adquirir concesión de disco<br />&#160;&#160;&#160;&#160;Agregar disco existente<br />&#160;&#160;&#160;&#160;Agregar nuevo disco<br />&#160;&#160;&#160;&#160;Agregar o quitar dispositivo<br />&#160;&#160;&#160;&#160;Configuración avanzada<br />&#160;&#160;&#160;&#160;Cambiar cantidad de CPU<br />&#160;&#160;&#160;&#160;Cambiar memoria<br />&#160;&#160;&#160;&#160;Cambiar configuración<br />&#160;&#160;&#160;&#160;Cambiar la ubicación del archivo de intercambio<br />&#160;&#160;&#160;&#160;Cambiar recurso<br />&#160;&#160;&#160;&#160;Configurar dispositivo USB de host<br />&#160;&#160;&#160;&#160;Configurar dispositivo sin formato<br />&#160;&#160;&#160;&#160;Configurar managedBy<br />&#160;&#160;&#160;&#160;Mostrar la configuración de la conexión<br />&#160;&#160;&#160;&#160;Extender disco virtual<br />&#160;&#160;&#160;&#160;Modificar la configuración del dispositivo<br />&#160;&#160;&#160;&#160;Compatibilidad de tolerancia a errores de consulta<br />&#160;&#160;&#160;&#160;Consultar archivos sin propietario<br />&#160;&#160;&#160;&#160;Recarga desde rutas de acceso<br />&#160;&#160;&#160;&#160;Quitar disco<br />&#160;&#160;&#160;&#160;Cambiar el nombre<br />&#160;&#160;&#160;&#160;Restablecer información de invitado<br />&#160;&#160;&#160;&#160;Establecer anotación<br />&#160;&#160;&#160;&#160;Alternancia del seguimiento de cambios de disco<br />&#160;&#160;&#160;&#160;Alternar la bifurcación primaria<br />&#160;&#160;&#160;&#160;Actualización de la compatibilidad de máquinas virtuales<br />Editar inventario<br />&#160;&#160;&#160;&#160;Crear a partir de existente<br />&#160;&#160;&#160;&#160;Crear nuevo<br />&#160;&#160;&#160;&#160;Mover<br />&#160;&#160;&#160;&#160;Registrar<br />&#160;&#160;&#160;&#160;Quitar<br />&#160;&#160;&#160;&#160;Anular el registro<br />Operaciones de invitado<br />&#160;&#160;&#160;&#160;Modificación de alias de operación de invitado<br />&#160;&#160;&#160;&#160;Consulta de alias de operación de invitado<br />&#160;&#160;&#160;&#160;Modificaciones de operaciones de invitado<br />&#160;&#160;&#160;&#160;Ejecución del programa de operaciones de invitado<br />&#160;&#160;&#160;&#160;Consultas de operaciones de invitado<br />Interacción<br />&#160;&#160;&#160;&#160;Responder pregunta<br />&#160;&#160;&#160;&#160;Operación de copia de seguridad en la máquina virtual<br />&#160;&#160;&#160;&#160;Configurar elementos multimedia de CD<br />&#160;&#160;&#160;&#160;Configurar soporte de disquete<br />&#160;&#160;&#160;&#160;Conexión de dispositivos<br />&#160;&#160;&#160;&#160;Interacción de la consola<br />&#160;&#160;&#160;&#160;Crear captura de pantalla<br />&#160;&#160;&#160;&#160;Desfragmentar todos los discos<br />&#160;&#160;&#160;&#160;Arrastrar y colocar<br />&#160;&#160;&#160;&#160;Administración de sistemas operativos invitados por VIX API<br />&#160;&#160;&#160;&#160;Inyectar códigos de digitalización de USB HID<br />&#160;&#160;&#160;&#160;Instalación de herramientas de VMware<br />&#160;&#160;&#160;&#160;Pausar o desactivar la pausa<br />&#160;&#160;&#160;&#160;Realizar operaciones de borrado o reducción<br />&#160;&#160;&#160;&#160;Apagar<br />&#160;&#160;&#160;&#160;Encender<br />&#160;&#160;&#160;&#160;Registrar sesión en la máquina virtual<br />&#160;&#160;&#160;&#160;Sesión de reproducción en máquina virtual<br />&#160;&#160;&#160;&#160;Suspender<br />&#160;&#160;&#160;&#160;Suspender tolerancia a errores<br />&#160;&#160;&#160;&#160;Probar la conmutación por error<br />&#160;&#160;&#160;&#160;Reinicio de prueba de máquina virtual secundaria<br />&#160;&#160;&#160;&#160;Desactivar la tolerancia a errores<br />&#160;&#160;&#160;&#160;Activar la tolerancia a errores<br />Aprovisionamiento<br />&#160;&#160;&#160;&#160;Permitir acceso al disco<br />&#160;&#160;&#160;&#160;Permitir el acceso a archivos<br />&#160;&#160;&#160;&#160;Permitir acceso a disco de solo lectura<br />&#160;&#160;&#160;&#160;Permitir descarga de máquina virtual<br />&#160;&#160;&#160;&#160;Clonar plantilla<br />&#160;&#160;&#160;&#160;Clonar máquina virtual<br />&#160;&#160;&#160;&#160;Crear plantilla a partir de la máquina virtual<br />&#160;&#160;&#160;&#160;Personalizar invitado<br />&#160;&#160;&#160;&#160;Implementar plantilla<br />&#160;&#160;&#160;&#160;Marcar como plantilla<br />&#160;&#160;&#160;&#160;Modificar especificación de personalización<br />&#160;&#160;&#160;&#160;Promover discos<br />&#160;&#160;&#160;&#160;Leer especificaciones de personalización<br />Configuración del servicio<br />&#160;&#160;&#160;&#160;Permitir notificaciones<br />&#160;&#160;&#160;&#160;Permitir sondeo de notificaciones de eventos globales<br />&#160;&#160;&#160;&#160;Administrar la configuración del servicio<br />&#160;&#160;&#160;&#160;Modificar la configuración del servicio<br />&#160;&#160;&#160;&#160;Configuraciones del servicio de consulta<br />&#160;&#160;&#160;&#160;Leer configuración de servicio<br />Administración de instantáneas<br />&#160;&#160;&#160;&#160;Crear instantánea<br />&#160;&#160;&#160;&#160;Quitar instantánea<br />&#160;&#160;&#160;&#160;Cambiar nombre de instantánea<br />&#160;&#160;&#160;&#160;Revertir instantánea<br />Replica de vSphere<br />&#160;&#160;&#160;&#160;Configurar la replicación<br />&#160;&#160;&#160;&#160;Administrar la replicación<br />&#160;&#160;&#160;&#160;Supervisión de la replicación |
| **vService** | Crear dependencia<br />Destruir dependencia<br />Volver a configurar la configuración de dependencia<br />Actualizar dependencia |
| **Etiquetas de vSphere** | Asignar y anular la asignación de la etiqueta vSphere<br />Crear etiqueta vSphere<br />Crear categoría de etiqueta vSphere<br />Eliminar etiqueta vSphere<br />Eliminar categoría de etiqueta vSphere<br />Editar etiqueta vSphere<br />Editar categoría de etiqueta vSphere<br />Modificar el campo UsedBy de la categoría<br />Modificar el campo UsedBy de la etiqueta |

### <a name="create-custom-roles-on-vcenter"></a>Creación de roles personalizados en vCenter

Azure VMware Solution admite el uso de roles personalizados con privilegios iguales o menores que el rol CloudAdmin. 

El rol CloudAdmin puede crear, modificar o eliminar roles personalizados que tengan privilegios iguales o menores que su rol actual. Es posible que pueda crear roles con privilegios mayores que CloudAdmin, pero no podrá asignar el rol a ningún usuario o grupo ni eliminar el rol.

Para evitar la creación de roles que no se pueden asignar o eliminar, Azure VMware Solution recomienda clonar el rol CloudAdmin como base para crear nuevos roles personalizados.

#### <a name="create-a-custom-role"></a>Crear un rol personalizado
1. Inicie sesión en vCenter con cloudadmin\@vsphere.local o un usuario con el rol CloudAdmin.
2. Navegue a la sección configuración **Roles** y seleccione **Menú** > **Administración** > **Access Control** > **Roles**.
3. Seleccione el rol **CloudAdmin** y seleccione el icono de la acción **Clonación de roles**.

   > [!NOTE] 
   > No clone el rol **Administrador**. Este rol no se puede usar y cloudadmin\@vsphere.local no puede eliminar el rol personalizado creado.

4. Proporcione el nombre que desee para el rol clonado.
5. Agregue o quite privilegios para el rol y seleccione **Aceptar**. El rol clonado debe estar ahora visible en la lista de **Roles**.


#### <a name="use-a-custom-role"></a>Uso de un rol personalizado

1. Navegue hasta el objeto que requiere el permiso agregado. Por ejemplo, para aplicar el permiso a una carpeta, vaya a **Menú** > **VM y plantillas** > **Nombre de carpeta**.
1. Haga clic con el botón derecho en el objeto y seleccione **Agregar permiso**.
1. En la ventana **Agregar permiso**, seleccione el origen de identidad en el menú desplegable **Usuario**, donde se puede encontrar el grupo o el usuario.
1. Busque el usuario o el grupo después de seleccionar el origen de identidad en la sección **Usuario**. 
1. Seleccione el rol que se aplicará al usuario o grupo.
1. Marque la casilla **Propagate to children** (Propagar a elementos secundarios) si es necesario y seleccione **Aceptar**.
   El permiso agregado se muestra en la sección **Permisos** del objeto.

## <a name="nsx-t-manager-access-and-identity"></a>Acceso e identidad de NSX-T Manager

>[!NOTE]
>NSX-T 2.5 se admite actualmente en todas las nuevas nubes privadas.

Use la cuenta de *administrador* para acceder a NSX-T Manager. Tiene privilegios completos y le permite crear y administrar puertas de enlace, segmentos (conmutadores lógicos) y todos los servicios de nivel 1 (T1). Los privilegios proporcionan acceso a la puerta de enlace de nivel 0 (T0) de NSX-T. Un cambio en la puerta de enlace T0 podría provocar una disminución en el rendimiento de la red o una pérdida de acceso a la nube privada. Abra una solicitud de soporte técnico en Azure Portal para solicitar cambios en la puerta de enlace NSX-T T0.

>[!TIP]
>Como alternativa al uso de NSX-T Manager, puede usar la [característica NSX simplificada de Azure Portal](configure-nsx-network-components-azure-portal.md).

 
## <a name="next-steps"></a>Pasos siguientes

Ahora que ha visto los conceptos de identidad y acceso de Azure VMware Solution, puede que quiera obtener información sobre:

- [Habilitación del recurso de Azure VMware Solution](enable-azure-vmware-solution.md)
- [Detalles de cada privilegio](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.security.doc/GUID-ED56F3C4-77D0-49E3-88B6-B99B8B437B62.html)
- [Supervisión y reparación de nubes privadas de Azure VMware Solution](/azure/azure-vmware/concepts-private-clouds-clusters#host-monitoring-and-remediation)
- [Habilitación del recurso de Azure VMware Solution](enable-azure-vmware-solution.md)


<!-- LINKS - external-->
[VMware product documentation]: https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.security.doc/GUID-ED56F3C4-77D0-49E3-88B6-B99B8B437B62.html

<!-- LINKS - internal -->
[concepts-upgrades]: ./concepts-private-clouds-clusters#host-maintenance-and-lifecycle-management
