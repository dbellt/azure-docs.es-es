---
title: 'Azure VMware Solution by CloudSimple: modelo de permisos para la nube privada'
description: Describe el modelo de permisos, los grupos y las categorías de la nube privada de CloudSimple
author: shortpatti
ms.author: v-patsho
ms.date: 08/16/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: be77be14acf05aa543c1494f16e499f00598e2b4
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108184521"
---
# <a name="cloudsimple-private-cloud-permission-model-of-vmware-vcenter"></a>Modelo de permisos de la nube privada de VMware vCenter de CloudSimple

CloudSimple conserva el acceso administrativo completo al entorno de nube privada. A cada cliente de CloudSimple se le conceden suficientes privilegios administrativos para poder implementar y administrar las máquinas virtuales en su entorno.  Si es necesario, puede escalar temporalmente los privilegios para realizar funciones administrativas.

## <a name="cloud-owner"></a>Usuario CloudOwner

Cuando crea una nube privada, se crea un usuario **CloudOwner** en el dominio de inicio de sesión único de vCenter, con acceso **Cloud-Owner-Role** para administrar objetos en la nube privada. Este usuario también puede configurar [orígenes de identidad de vCenter](set-vcenter-identity.md) adicionales y otros usuarios en la instancia de vCenter de la nube privada.

> [!NOTE]
> El usuario predeterminado para el vCenter de la nube privada de CloudSimple es cloudowner@cloudsimple.local cuando se crea una nube privada.

## <a name="user-groups"></a>Grupos de usuarios

Durante la implementación de una nube privada se crea un grupo denominado **Cloud-Owner-Group**. Los usuarios de este grupo pueden administrar diversas partes del entorno de vSphere en la nube privada. A este grupo se le conceden automáticamente privilegios de **Cloud-Owner-Role** y el usuario **CloudOwner** se agrega como miembro de este grupo.  CloudSimple crea grupos adicionales con privilegios limitados para facilitar la administración.  Puede agregar cualquier usuario a estos grupos creados previamente y los privilegios definidos a continuación se asignarán automáticamente a los usuarios de esos grupos.

### <a name="pre-created-groups"></a>Grupos creados previamente

| Nombre del grupo | Finalidad | Role |
| -------- | ------- | ------ |
| Cloud-Owner-Group | Los miembros de este grupo tienen privilegios administrativos para el vCenter de la nube privada | [Cloud-Owner-Role](#cloud-owner-role) |
| Cloud-Global-Cluster-Admin-Group | Los miembros de este grupo tienen privilegios administrativos para el clúster de vCenter de la nube privada | [Cloud-Cluster-Admin-Role](#cloud-cluster-admin-role) |
| Cloud-Global-Storage-Admin-Group | Los miembros de este grupo pueden administrar el almacenamiento en el vCenter de la nube privada | [Cloud-Storage-Admin-Role](#cloud-storage-admin-role) |
| Cloud-Global-Network-Admin-Group | Los miembros de este grupo pueden administrar grupos de puertos distribuidos y de red en el vCenter de la nube privada | [Cloud-Network-Admin-Role](#cloud-network-admin-role) |
| Cloud-Global-VM-Admin-Group | Los miembros de este grupo pueden administrar máquinas virtuales en el vCenter de la nube privada | [Cloud-VM-Admin-Role](#cloud-vm-admin-role) |

Para conceder permisos a usuarios individuales para que administren la nube privada, cree cuentas de usuario que se agreguen a los grupos adecuados.

> [!CAUTION]
> Los nuevos usuarios solo se deben agregar a *Cloud-Owner-Group*, *Cloud-Global-Cluster-Admin-Group*, *Cloud-Global-Storage-Admin-Group*, *Cloud-Global-Network-Admin-Group* o *Cloud-Global-VM-Admin-Group*.  Los usuarios agregados al grupo *Administradores* se quitarán automáticamente.  Solo se deben agregar cuentas de servicio al grupo *Administradores* y estas cuentas no se deben usar para iniciar sesión en la interfaz de usuario web de vSphere.

## <a name="list-of-vcenter-privileges-for-default-roles"></a>Lista de privilegios de vCenter para roles predeterminados

### <a name="cloud-owner-role"></a>Cloud-Owner-Role

| **Categoría** | **Privilegio** |
|----------|-----------|
| **Alarmas** | Confirmar alarma <br> Crear alarma <br> Deshabilitar acción de alarma <br> Modificar alarma <br> Eliminar alarma <br> Establecer estado de la alarma |
| **Permisos** | Permiso de modificación |
| **Biblioteca de contenido** | Agregar elemento de biblioteca <br> Crear biblioteca local <br> Crear biblioteca suscrita <br> Eliminar elemento de biblioteca <br> Eliminar biblioteca local <br> Eliminar biblioteca suscrita <br> Descarga de archivos <br> Desalojar elemento de biblioteca <br> Desalojar biblioteca suscrita <br> Importar almacenamiento <br> Sondear información de suscripción <br> Leer en el almacenamiento <br> Sincronizar elemento de biblioteca <br> Sincronizar biblioteca suscrita <br> Escribir introspección <br> Actualizar valores de configuración <br> Actualizar archivos <br> Actualizar biblioteca <br> Actualizar elemento de biblioteca <br> Actualizar biblioteca local <br> Actualizar biblioteca suscrita <br> Ver valores de configuración |
| **Operaciones criptográficas** | Agregar disco <br> Clonar <br> Descifrado <br> Direct Access <br> Cifrado <br> Descifrar nuevo <br> Administrar KMS <br> Administración de directivas de cifrado <br> Administración de claves <br> Migrar <br> Volver a cifrar <br> Registrar máquina virtual <br> Registrar host |
| **Grupo dvPort** | Crear <br> Eliminar <br> Modificar <br> Operación de directiva <br> Operación de ámbito |
| **Almacén de datos** | Asignar espacio <br> Examen de un almacén de datos <br> Configurar almacén de datos <br> Operaciones de archivo de bajo nivel <br> Mover almacén de datos <br> Eliminar almacén de datos <br> Quitar archivo <br> Cambiar nombre del almacén de datos <br> Actualizar archivos de máquina virtual <br> Actualizar metadatos de máquina virtual |
| **ESX Agent Manager** | Config <br> Modificar <br> Ver |
| **Extensión** | Registrar extensión <br> Anular registro de la extensión <br> Actualizar extensión |
| **Proveedor de estadísticas externo**| Register <br> Unregister <br> Actualizar |
| **Carpeta** | Crear carpeta <br> Eliminar carpeta <br> Mover carpeta <br> Cambiar nombre de la carpeta |
| **Global** | Cancelar tarea <br> Planificación de capacidad <br> Diagnóstico <br> Deshabilitar métodos <br> Habilitar métodos <br> Etiqueta global <br> Health <br> Licencias <br> Registrar eventos <br> Administrar atributos personalizados <br> Proxy <br> Acción de script <br> Administradores de servicio <br> Establecer atributo personalizado <br> Etiqueta del sistema |
| **Proveedor de actualización de estado** | Register <br> Unregister <br> Actualizar |
| **Host > Configuración** | Configuración de la partición de almacenamiento |
| **Host > Inventario** | Modificar clúster |
| **Etiquetas de vSphere** | Asignar o cancelar la asignación de la etiqueta vSphere <br> Crear etiqueta vSphere <br> Crear categoría de etiqueta vSphere <br> Eliminar etiqueta vSphere <br> Eliminar categoría de etiqueta vSphere <br> Editar etiqueta vSphere <br> Editar categoría de etiqueta vSphere <br> Modificar el campo UsedBy de la categoría <br> Modificar el campo UsedBy de la etiqueta |
| **Network** | Asignar red <br> Configuración <br> Mover red <br> Remove |
| **Rendimiento** | Modificar intervalos |
| **Hospedar perfil** | Ver |
| **Recurso** | Aplicar recomendaciones <br> Asignar vApp a un grupo de recursos <br> Asignar máquina virtual a grupo de recursos <br> Crear grupo de recursos <br> Migrar máquina virtual apagada <br> Migrar máquina virtual encendida <br> Modificar grupo de recursos <br> Mover grupo de recursos <br> Consultar vMotion <br> Eliminar grupo de recursos <br> Cambiar nombre del grupo de recursos |
| **Tarea programada** | Creación de tareas <br> Modificar tarea <br> Quitar tarea <br> Ejecutar tarea |
| **Sesiones** | Suplantar usuario <br> Message <br> Validar sesión <br> Ver y detener sesiones |
| **Clúster de almacén de datos** | Configurar un clúster de almacén de datos |
| **Almacenamiento controlado por perfiles** | Actualización del almacenamiento controlado por perfiles <br> Vista del almacenamiento controlado por perfiles |
| **Vistas de almacenamiento** | Configurar servicio <br> Ver |
| **Tareas** | Crear la tarea <br> Actualizar tarea |
| **Servicio de transferencia**| Administración <br> Supervisión |
| **vApp** | Agregar máquina virtual <br> Asignar grupo de recursos <br> Asignar vApp <br> Clonar <br> Crear <br> Eliminar <br> Exportación <br> Importar <br> Move <br> Apagado <br> Encendido <br> Cambiar nombre <br> Suspender <br> Unregister <br> Ver el entorno de OVF <br> Configuración de aplicaciones de vApp <br> Configuración de instancias de vApp <br> Configuración de managedBy de vApp <br> Configuración de recursos de vApp |
| **VRMPolicy** | Consultar VRMPolicy <br> Actualizar VRMPolicy |
| **Máquina virtual > Configuración** | Agregar disco existente <br> Agregar disco nuevo <br> Agregar o eliminar dispositivo <br> Avanzado <br> Cambiar cantidad de CPU <br> Cambiar recurso <br> Configurar managedBy <br> Seguimiento de cambios de disco <br> Concesión de disco <br> Mostrar configuración de conexión <br> Extender disco virtual <br> Hospedar dispositivo USB <br> Memoria <br> Modificar la configuración del dispositivo <br> Consultar compatibilidad con tolerancia a errores <br> Consultar archivos sin propietario <br> Dispositivo sin formato <br> Volver a cargar desde la ruta de acceso <br> Quitar disco <br> Cambiar nombre <br> Restablecer información de invitados <br> Establecer anotación <br> Configuración <br> Ubicación del archivo de intercambio <br> Alternar la bifurcación principal <br> Desbloquear máquina virtual <br> Actualizar compatibilidad con máquina virtual |
| **Máquina virtual > Operaciones de invitado** | Modificación de alias de operación de invitado <br> Consulta de alias de operación de invitado <br> Modificaciones de operación de invitado <br> Ejecución del programa de operaciones de invitado <br> Consultas de operaciones de invitado |
| **Máquina virtual > Interacción** | Responder pregunta <br> Operación de copia de seguridad en máquina virtual <br> Configurar soporte de CD <br> Configurar soporte de disquete <br> Interacción de la consola <br> Crear captura de pantalla <br> Desfragmentar todos los discos <br> Conexión del dispositivo <br> Arrastrar y colocar <br> Administración del sistema operativo invitado de VIX API <br> Inyectar códigos de digitalización de USB HID <br> Pausar o desactivar la pausa <br> Realizar operaciones de borrado o reducción <br> Apagado <br> Encendido <br> Registrar sesión en la máquina virtual <br> Volver a reproducir sesión en la máquina virtual <br> Reset <br> Reanudar tolerancia a errores <br> Suspender <br> Suspender tolerancia a errores <br> Conmutación por error de prueba <br> Reinicio de prueba de máquina virtual secundaria <br> Desactivar tolerancia a errores <br> Activar tolerancia a errores <br> Instalación de herramientas de VMware |
| **Máquina virtual > Inventario** | Crear a partir de un elemento existente <br> Crear nuevo <br> Move <br> Register <br> Remove <br> Unregister |
| **Máquina virtual > Aprovisionamiento** | Permitir acceso al disco <br> Permitir acceso al archivo <br> Permitir acceso a disco de solo lectura <br> Permitir descarga de máquina virtual <br> Permitir carga de archivos de máquina virtual <br> Clonar plantilla <br> Clonar máquina virtual <br> Crear plantilla a partir de máquina virtual <br> Personalizar <br> Implementar plantilla <br> Marcar como plantilla <br> Marcar como máquina virtual <br> Modificar especificación de personalización <br> Promocionar discos <br> Leer especificaciones de personalización |
| **Máquina virtual > Configuración de servicio** | Permitir notificaciones <br> Permitir sondeo de notificaciones de eventos globales <br> Administrar configuraciones de servicio <br> Modificar configuraciones de servicio <br> Consultar configuraciones de servicio <br> Leer configuraciones de servicio |
| **Máquina virtual > Administración de instantáneas** | Creación de instantáneas <br> Quitar instantánea <br> Cambiar nombre de instantánea <br> Revertir a instantánea |
| **Máquina virtual > Replicación de vSphere** | Configuración de la replicación <br> Administrar replicación <br> Supervisión de la replicación |
| **vService** | Crear dependencia <br> Destruir dependencia <br> Volver a configurar la configuración de dependencia <br> Actualizar dependencia |

### <a name="cloud-cluster-admin-role"></a>Cloud-Cluster-Admin-Role

| **Categoría** | **Privilegio** |
|----------|-----------|
| **Almacén de datos** | Asignar espacio <br> Examen de un almacén de datos <br> Configurar almacén de datos <br> Operaciones de archivo de bajo nivel <br> Eliminar almacén de datos <br> Cambiar nombre del almacén de datos <br> Actualizar archivos de máquina virtual <br> Actualizar metadatos de máquina virtual |
| **Carpeta** | Crear carpeta <br> Eliminar carpeta <br> Mover carpeta <br> Cambiar nombre de la carpeta |
| **Host > Configuración**  | Configuración de la partición de almacenamiento |
| **Etiquetas de vSphere** | Asignar o cancelar la asignación de la etiqueta vSphere <br> Crear etiqueta vSphere <br> Crear categoría de etiqueta vSphere <br> Eliminar etiqueta vSphere <br> Eliminar categoría de etiqueta vSphere <br> Editar etiqueta vSphere <br> Editar categoría de etiqueta vSphere <br> Modificar el campo UsedBy de la categoría <br> Modificar el campo UsedBy de la etiqueta |
| **Network** | Asignar red |
| **Recurso** | Aplicar recomendaciones <br> Asignar vApp a un grupo de recursos <br> Asignar máquina virtual a grupo de recursos <br> Crear grupo de recursos <br> Migrar máquina virtual apagada <br> Migrar máquina virtual encendida <br> Modificar grupo de recursos <br> Mover grupo de recursos <br> Consultar vMotion <br> Eliminar grupo de recursos <br> Cambiar nombre del grupo de recursos |
| **vApp** | Agregar máquina virtual <br> Asignar grupo de recursos <br> Asignar vApp <br> Clonar <br> Crear <br> Eliminar <br> Exportación <br> Importar <br> Move <br> Apagado <br> Encendido <br> Cambiar nombre <br> Suspender <br> Unregister <br> Ver el entorno de OVF <br> Configuración de aplicaciones de vApp <br> Configuración de instancias de vApp <br> Configuración de managedBy de vApp <br> Configuración de recursos de vApp |
| **VRMPolicy** | Consultar VRMPolicy <br> Actualizar VRMPolicy |
| **Máquina virtual > Configuración** | Agregar disco existente <br> Agregar disco nuevo <br> Agregar o eliminar dispositivo <br> Avanzado <br> Cambiar cantidad de CPU <br> Cambiar recurso <br> Configurar managedBy <br> Seguimiento de cambios de disco <br> Concesión de disco <br> Mostrar configuración de conexión <br> Extender disco virtual <br> Hospedar dispositivo USB <br> Memoria <br> Modificar la configuración del dispositivo <br> Consultar compatibilidad con tolerancia a errores <br> Consultar archivos sin propietario <br> Dispositivo sin formato <br> Volver a cargar desde la ruta de acceso <br> Quitar disco <br> Cambiar nombre <br> Restablecer información de invitados <br> Establecer anotación <br> Configuración <br> Ubicación del archivo de intercambio <br> Alternar la bifurcación principal <br> Desbloquear máquina virtual <br> Actualizar compatibilidad con máquina virtual |
| **Máquina virtual > Operaciones de invitado** | Modificación de alias de operación de invitado <br> Consulta de alias de operación de invitado <br> Modificaciones de operación de invitado <br> Ejecución del programa de operaciones de invitado <br> Consultas de operaciones de invitado |
| **Máquina virtual > Interacción** | Responder pregunta <br> Operación de copia de seguridad en máquina virtual <br> Configurar soporte de CD <br> Configurar soporte de disquete <br> Interacción de la consola <br> Crear captura de pantalla <br> Desfragmentar todos los discos <br> Conexión del dispositivo <br> Arrastrar y colocar <br> Administración del sistema operativo invitado de VIX API <br> Inyectar códigos de digitalización de USB HID <br> Pausar o desactivar la pausa <br> Realizar operaciones de borrado o reducción <br> Apagado <br> Encendido <br> Registrar sesión en la máquina virtual <br> Volver a reproducir sesión en la máquina virtual <br> Reset <br> Reanudar tolerancia a errores <br> Suspender <br> Suspender tolerancia a errores <br> Conmutación por error de prueba <br> Reinicio de prueba de máquina virtual secundaria <br> Desactivar tolerancia a errores <br> Activar tolerancia a errores <br> Instalación de herramientas de VMware
| **Máquina virtual > Inventario** | Crear a partir de un elemento existente <br> Crear nuevo <br> Move <br> Register <br> Remove <br> Unregister |
| **Máquina virtual > Aprovisionamiento** | Permitir acceso al disco <br> Permitir acceso al archivo <br> Permitir acceso a disco de solo lectura <br> Permitir descarga de máquina virtual <br> Permitir carga de archivos de máquina virtual <br> Clonar plantilla <br> Clonar máquina virtual <br> Crear plantilla a partir de máquina virtual <br> Personalizar <br> Implementar plantilla <br> Marcar como plantilla <br> Marcar como máquina virtual <br> Modificar especificación de personalización <br> Promocionar discos  <br> Leer especificaciones de personalización |
| **Máquina virtual > Configuración de servicio** | Permitir notificaciones <br> Permitir sondeo de notificaciones de eventos globales <br> Administrar configuraciones de servicio <br> Modificar configuraciones de servicio <br> Consultar configuraciones de servicio <br> Leer configuraciones de servicio
| **Máquina virtual > Administración de instantáneas** | Creación de instantáneas <br> Quitar instantánea <br> Cambiar nombre de instantánea <br> Revertir a instantánea |
| **Máquina virtual > Replicación de vSphere** | Configuración de la replicación <br> Administrar replicación <br> Supervisión de la replicación |
| **vService** | Crear dependencia <br> Destruir dependencia <br> Volver a configurar la configuración de dependencia <br> Actualizar dependencia |

### <a name="cloud-storage-admin-role"></a>Cloud-Storage-Admin-Role

| **Categoría** | **Privilegio** |
|----------|-----------|
| **Almacén de datos** | Asignar espacio <br> Examen de un almacén de datos <br> Configurar almacén de datos <br> Operaciones de archivo de bajo nivel <br> Eliminar almacén de datos <br> Cambiar nombre del almacén de datos <br> Actualizar archivos de máquina virtual <br> Actualizar metadatos de máquina virtual |
| **Host > Configuración** | Configuración de la partición de almacenamiento |
| **Clúster de almacén de datos** | Configurar un clúster de almacén de datos |
| **Almacenamiento controlado por perfiles** | Actualización del almacenamiento controlado por perfiles <br> Vista del almacenamiento controlado por perfiles |
| **Vistas de almacenamiento** | Configurar servicio <br> Ver |

### <a name="cloud-network-admin-role"></a>Cloud-Network-Admin-Role

| **Categoría** | **Privilegio** |
|----------|-----------|
| **Grupo dvPort** | Crear <br> Eliminar <br> Modificar <br> Operación de directiva <br> Operación de ámbito |
| **Network** | Asignar red <br> Configuración <br> Mover red <br> Remove |
| **Máquina virtual > Configuración** | Modificar la configuración del dispositivo |

### <a name="cloud-vm-admin-role"></a>Cloud-VM-Admin-Role

| **Categoría** | **Privilegio** |
|----------|-----------|
| **Almacén de datos** | Asignar espacio <br> Examen de un almacén de datos |
| **Network** | Asignar red |
| **Recurso** | Asignar máquina virtual a grupo de recursos <br> Migrar máquina virtual apagada <br> Migrar máquina virtual encendida
| **vApp** | Exportación <br> Importar |
| **Máquina virtual > Configuración** | Agregar disco existente <br> Agregar disco nuevo <br> Agregar o eliminar dispositivo <br> Avanzado <br> Cambiar cantidad de CPU <br> Cambiar recurso <br> Configurar managedBy <br> Seguimiento de cambios de disco <br> Concesión de disco <br> Mostrar configuración de conexión <br> Extender disco virtual <br> Hospedar dispositivo USB <br> Memoria <br> Modificar la configuración del dispositivo <br> Consultar compatibilidad con tolerancia a errores <br> Consultar archivos sin propietario <br> Dispositivo sin formato <br> Volver a cargar desde la ruta de acceso <br> Quitar disco <br> Cambiar nombre <br> Restablecer información de invitados <br> Establecer anotación <br> Configuración <br> Ubicación del archivo de intercambio <br> Alternar la bifurcación principal <br> Desbloquear máquina virtual <br> Actualizar compatibilidad con máquina virtual |
| **Máquina virtual > Operaciones de invitado** | Modificación de alias de operación de invitado <br> Consulta de alias de operación de invitado <br> Modificaciones de operación de invitado <br> Ejecución del programa de operaciones de invitado <br> Consultas de operaciones de invitado    |
| **Máquina virtual > Interacción** | Responder pregunta <br> Operación de copia de seguridad en máquina virtual <br> Configurar soporte de CD <br> Configurar soporte de disquete <br> Interacción de la consola <br> Crear captura de pantalla <br> Desfragmentar todos los discos <br> Conexión del dispositivo <br> Arrastrar y colocar <br> Administración del sistema operativo invitado de VIX API <br> Inyectar códigos de digitalización de USB HID <br> Pausar o desactivar la pausa <br> Realizar operaciones de borrado o reducción <br> Apagado <br> Encendido <br> Registrar sesión en la máquina virtual <br> Volver a reproducir sesión en la máquina virtual <br> Reset <br> Reanudar tolerancia a errores <br> Suspender <br> Suspender tolerancia a errores <br> Conmutación por error de prueba <br> Reinicio de prueba de máquina virtual secundaria <br> Desactivar tolerancia a errores <br> Activar tolerancia a errores <br> Instalación de herramientas de VMware |
| **Máquina virtual > Inventario** | Crear a partir de un elemento existente <br> Crear nuevo <br> Move <br> Register <br> Remove <br> Unregister |
| **Máquina virtual > Aprovisionamiento** | Permitir acceso al disco <br> Permitir acceso al archivo <br> Permitir acceso a disco de solo lectura <br> Permitir descarga de máquina virtual <br> Permitir carga de archivos de máquina virtual <br> Clonar plantilla <br> Clonar máquina virtual <br> Crear plantilla a partir de máquina virtual <br> Personalizar <br> Implementar plantilla <br> Marcar como plantilla <br> Marcar como máquina virtual <br> Modificar especificación de personalización <br> Promocionar discos <br> Leer especificaciones de personalización |
| **Máquina virtual > Configuración de servicio** | Permitir notificaciones <br> Permitir sondeo de notificaciones de eventos globales <br> Administrar configuraciones de servicio <br> Modificar configuraciones de servicio <br> Consultar configuraciones de servicio <br> Leer configuraciones de servicio
| **Máquina virtual > Administración de instantáneas** | Creación de instantáneas <br> Quitar instantánea <br> Cambiar nombre de instantánea <br> Revertir a instantánea |
| **Máquina virtual > Replicación de vSphere** | Configuración de la replicación <br> Administrar replicación <br> Supervisión de la replicación |
| **vService** | Crear dependencia <br> Destruir dependencia <br> Volver a configurar la configuración de dependencia <br> Actualizar dependencia |
