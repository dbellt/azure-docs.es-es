---
title: Uso de grupos extendidos en Azure HPC Cache
description: Cómo configurar servicios de directorio para el acceso de cliente a destinos de almacenamiento en Azure HPC Cache
author: ekpgh
ms.service: hpc-cache
ms.topic: how-to
ms.date: 03/15/2021
ms.author: v-erkel
ms.openlocfilehash: 9db7576cb2278cc2ef0d8b93ef04bb633962cd0e
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107255756"
---
# <a name="configure-directory-services"></a>Configuración de servicios de directorio

La configuración de **Servicios de directorio** permite a Azure HPC Cache usar un origen externo a fin de autenticar a los usuarios para obtener acceso al almacenamiento de back-end.

Puede que tenga que habilitar **Grupos extendidos** si el flujo de trabajo incluye destinos de almacenamiento de NFS y clientes que son miembros de más de 16 grupos.

Después de hacer clic en el botón para habilitar los grupos extendidos, debe elegir el origen que usará Azure HPC Cache para obtener las credenciales de usuario y de grupo.

* [Active Directory](#configure-active-directory): obtenga credenciales de un servidor de Active Directory externo. No puede usar Azure Active Directory para esta tarea.
* [Archivo plano](#configure-file-download): descargue los archivos `/etc/group` y `/etc/passwd` desde una ubicación de red.
* [LDAP](#configure-ldap): obtenga credenciales de un origen compatible con el Protocolo ligero de acceso a directorios (LDAP).

> [!NOTE]
> Asegúrese de que la memoria caché puede tener acceso a su origen de información de grupo desde dentro de su subred segura.<!-- + details/examples -->

En el campo **Nombre de usuario descargado** se muestra el estado de la descarga de información de grupo más reciente.

![captura de pantalla de la página de configuración de la página de servicios de directorio del portal, con la opción Sí seleccionada para grupos extendidos y el origen de descarga etiquetado del menú desplegable abierto](media/directory-services-select-group-source.png)

## <a name="configure-active-directory"></a>Configurar Active Directory

En esta sección se explica cómo configurar la memoria caché para obtener las credenciales de usuario y de grupo de un servidor de Active Directory (AD) externo.

En **Detalles de Active Directory**, proporcione estos valores:

* **DNS principal**: especifique la dirección IP de un servidor de nombres de dominio que la caché puede usar para resolver el nombre de dominio de AD.

* **DNS secundario** (opcional): escriba la dirección de un servidor de nombres que se usará si el servidor principal no está disponible.

* **Nombre de dominio DNS de AD**: proporcione el nombre de dominio completo del servidor de AD al que se unirá la caché para obtener las credenciales.

* **Nombre del servidor de caché (cuenta de equipo)** : establezca el nombre que se asignará a esta instancia de HPC Cache cuando se una al dominio de AD. Especifique un nombre que sea fácil de reconocer como esta caché. Dicho nombre puede tener hasta 15 caracteres y puede incluir letras mayúsculas o minúsculas, números y guiones (-).

En la sección **Credenciales**, proporcione un nombre de usuario y una contraseña de administrador de AD que pueda usar Azure HPC Cache para obtener acceso al servidor de AD. Esta información se cifra cuando se almacena y no se puede consultar.

Guarde la configuración haciendo clic en el botón situado en la parte superior de la página.

![captura de pantalla de la sección Detalles de la descarga con los valores de Active Directory rellenados](media/group-download-details-ad.png)

## <a name="configure-file-download"></a>Configuración de la descarga de archivos

Estos valores son necesarios si desea descargar archivos con la información de usuario y de grupo. Los archivos deben estar en el formato `/etc/group` y`/etc/passwrd` de Linux/UNIX estándar.

* **URI del archivo de usuario**: escriba el URI completo del archivo `/etc/passwrd`.
* **URI del archivo de grupo**: escriba el URI completo del archivo `/etc/group`.

![captura de pantalla de la sección Detalles de la descarga para la descarga de un archivo plano](media/group-download-details-file.png)

## <a name="configure-ldap"></a>Configuración de LDAP

Rellene estos valores si desea utilizar un origen LDAP que no sea de AD para obtener credenciales de usuario y de grupo. Consulte con el administrador de LDAP si necesita ayuda con estos valores.

* **Servidor LDAP**: escriba el nombre de dominio completo o la dirección IP del servidor LDAP que se usará. <!-- only one, not up to 3 -->

* **LDAP base DN** (DN base de LDAP): especifique el nombre distintivo base del dominio de LDAP, en formato DN. Si no conoce el DN base, pregunte al administrador de LDAP.

El servidor y el DN base son la única configuración necesaria para el funcionamiento de LDAP, pero las opciones adicionales hacen que la conexión sea más segura.

![captura del área de configuración de LDAP de la página de configuración de la página de servicios de directorio](media/group-download-details-ldap.png)

En la sección **Acceso seguro**, puede habilitar el cifrado y la validación de certificados para la conexión LDAP. Después de hacer clic en **Sí** para habilitar el cifrado, tiene estas opciones:

* **Validate certificate** (Validar certificado): al establecerse este, el certificado del servidor LDAP se comprueba con la entidad de certificación en el campo de URI siguiente.

* **CA certificate URI** (URI de certificado de CA): especifique la ruta de acceso al certificado autoritativo. Puede tratarse de un vínculo a un certificado validado por CA o a un certificado autofirmado. Este campo es necesario para usar la configuración de certificados validados externamente.

* **Auto-download certificate** (Certificado de descarga automática): elija **Sí** si desea intentar descargar un certificado nada más enviar esta configuración.

Rellene la sección **Credenciales** si desea usar credenciales estáticas para la seguridad de LDAP. Esta información se cifra cuando se almacena y no se puede consultar.

* **DN de enlace**: escriba el nombre distintivo de enlace que se va a usar para autenticarse en el servidor LDAP (use el formato DN).
* **Contraseña de enlace**: proporcione la contraseña del DN de enlace.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre el acceso de los clientes en [Montaje de la instancia de Azure HPC Cache](hpc-cache-mount.md).
* Si sus credenciales no se descargan correctamente, consulte al administrador el origen de las credenciales. Abra una [incidencia de soporte técnico](hpc-cache-support-ticket.md) si es necesario.
