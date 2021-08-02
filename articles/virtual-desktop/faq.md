---
title: 'Preguntas frecuentes de Azure Virtual Desktop: Azure'
description: Preguntas frecuentes y procedimientos recomendados para Azure Virtual Desktop.
author: Heidilohr
ms.topic: conceptual
ms.date: 03/09/2021
ms.author: helohr
ms.custom: devx-track-azurepowershell
manager: femila
ms.openlocfilehash: 604f7ac3d06ceb9ea981deb6aba4e9f208f06e20
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111757722"
---
# <a name="azure-virtual-desktop-faq"></a>Preguntas frecuentes de Azure Virtual Desktop

En este artículo se responden preguntas frecuentes y se explican procedimientos recomendados para Azure Virtual Desktop.

## <a name="what-are-the-minimum-admin-permissions-i-need-to-manage-objects"></a>¿Cuáles son los permisos de administrador mínimos necesarios para administrar objetos?

Si desea crear grupos de hosts y otros objetos, debe tener asignado el rol Colaborador en la suscripción o grupo de recursos con el que está trabajando.

Debe tener asignado el rol Administrador de acceso de usuarios en un grupo de aplicaciones para publicar grupos de aplicaciones para usuarios o grupos de usuarios.

Para impedir que un administrador solo administre las sesiones de usuario, como el envío de mensajes a los usuarios, el cierre de la sesión de los usuarios, etc., puede crear roles personalizados. Por ejemplo:

```powershell
"actions": [
"Microsoft.Resources/deployments/operations/read",
"Microsoft.Resources/tags/read",
"Microsoft.Authorization/roleAssignments/read",
"Microsoft.DesktopVirtualization/hostpools/sessionhosts/usersessions/*",
"Microsoft.DesktopVirtualization/hostpools/sessionhosts/read",
"Microsoft.DesktopVirtualization/hostpools/sessionhosts/write"
],
"notActions": [],
"dataActions": [],
"notDataActions": []
}
```

## <a name="does-azure-virtual-desktop-support-split-azure-active-directory-models"></a>¿Admite Azure Virtual Desktop los modelos divididos de Azure Active Directory?

Cuando se asigna un usuario a un grupo de aplicaciones, el servicio realiza una simple asignación de roles de Azure. Como consecuencia, la instancia de Azure Active Directory (AD) del usuario y la instancia de Azure AD del grupo de aplicaciones deben estar en la misma ubicación. Todos los objetos de servicio, como los grupos de hosts, los grupos de aplicaciones y las áreas de trabajo, también deben estar en el mismo Azure AD que el usuario.

Puede crear máquinas virtuales (VM) en otra instancia de Azure AD siempre que sincronice la instancia de Active Directory con la instancia de Azure AD del usuario en la misma red virtual (VNET).

## <a name="what-are-location-restrictions"></a>¿Cuáles son las restricciones de ubicación?

Todos los recursos de servicio tienen una ubicación asociada a ellos. La ubicación de un grupo de hosts determina en qué área geográfica se almacenan los metadatos del servicio para el grupo de hosts. Un grupo de aplicaciones no puede existir sin un grupo de hosts. Si agrega aplicaciones a un grupo de aplicaciones de RemoteApp, también necesitará un host de sesión para establecer las aplicaciones del menú Inicio. Para cualquier acción del grupo de aplicaciones, también necesitará un acceso a datos relacionado en el grupo de hosts. Para asegurarse de que los datos no se transfieran entre varias ubicaciones, la ubicación del grupo de aplicaciones debe ser la misma que la del grupo de hosts.

Las áreas de trabajo también deben estar en la misma ubicación que sus grupos de aplicaciones. Siempre que el área de trabajo se actualiza, el grupo de aplicaciones relacionado se actualiza junto con él. Al igual que con los grupos de aplicaciones, el servicio requiere que todas las áreas de trabajo estén asociadas a grupos de aplicaciones creados en la misma ubicación.

## <a name="how-do-you-expand-an-objects-properties-in-powershell"></a>¿Cómo se expanden las propiedades de un objeto en PowerShell?

Al ejecutar un cmdlet de PowerShell, solo ve el nombre y la ubicación del recurso.

Por ejemplo:

```powershell
Get-AzWvdHostPool -Name 0224hp -ResourceGroupName 0224rg

Location Name   Type
-------- ----   ----
westus   0224hp Microsoft.DesktopVirtualization/hostpools
```

Para ver todas las propiedades de un recurso, agregue `format-list` o `fl` al final del cmdlet.

Por ejemplo:

```powershell
Get-AzWvdHostPool -Name 0224hp -ResourceGroupName 0224rg |fl
```

Para ver propiedades específicas, agregue los nombres de propiedad específicos después `format-list` o `fl`.

Por ejemplo:

```powershell
Get-AzWvdHostPool -Name demohp -ResourceGroupName 0414rg |fl CustomRdpProperty

CustomRdpProperty : audiocapturemode:i:0;audiomode:i:0;drivestoredirect:s:;redirectclipboard:i:1;redirectcomports:i:0;redirectprinters:i:1;redirectsmartcards:i:1;screen modeid:i:2;
```

## <a name="does-azure-virtual-desktop-support-guest-users"></a>¿Admite Azure Virtual Desktop los usuarios invitados?

Azure Virtual Desktop no admite cuentas de usuario invitado de Azure AD. Por ejemplo, supongamos que un grupo de usuarios invitados tiene licencias de Microsoft 365 E3 por usuario, Windows E3 por usuario o WIN VDA en su propia empresa, pero son usuarios invitados en la instancia de Azure AD de otra empresa. La otra empresa administraría los objetos de usuario de los usuarios invitados tanto en Azure AD como en Active Directory como cuentas locales.

No puede usar sus propias licencias en beneficio de un tercero. Además, Azure Virtual Desktop no es compatible actualmente con las cuentas de Microsoft (MSA).

## <a name="why-dont-i-see-the-client-ip-address-in-the-wvdconnections-table"></a>¿Por qué no se ve la dirección IP del cliente en la tabla WVDConnections?

Actualmente no hay una manera confiable de recopilar las direcciones IP del cliente web, por lo que no se incluye ese valor en la tabla.

## <a name="how-does-azure-virtual-desktop-handle-backups"></a>¿Cómo controla Azure Virtual Desktop las copias de seguridad?

Hay varias opciones en Azure para controlar las copias de seguridad. Puede usar Azure Backup, Site Recovery e instantáneas.

## <a name="does-azure-virtual-desktop-support-third-party-collaboration-apps"></a>¿Admite Azure Virtual Desktop aplicaciones de colaboración de terceros?

Actualmente, Azure Virtual Desktop está optimizado para Teams. En este momento, Microsoft no admite aplicaciones de colaboración de terceros, como Zoom. Las organizaciones externas son responsables de proporcionar las instrucciones de compatibilidad a sus clientes. Además, Azure Virtual Desktop no admite Skype Empresarial.

## <a name="can-i-change-from-pooled-to-personal-host-pools"></a>¿Se puede cambiar de grupos de hosts agrupados a personales?

Una vez que se crea un grupo de hosts, no se puede cambiar el tipo. Sin embargo, las VM que registre en un grupo de hosts pueden moverse a otro tipo de grupo de hosts.

## <a name="whats-the-largest-profile-size-fslogix-can-handle"></a>¿Cuál es el tamaño de perfil más grande que FSLogix puede controlar?

Las limitaciones o cuotas de FSLogix dependen del tejido de almacenamiento usado para almacenar los archivos VHD(X) de perfil de usuario.

En la tabla siguiente se muestra un ejemplo de las operaciones de E/S que necesita un perfil de FSLogix para admitir cada usuario. Los requisitos pueden variar enormemente en función del usuario, las aplicaciones y la actividad de cada perfil.

| Resource | Requisito |
|---|---|
| IOPS de estado estable | 10 |
| IOPS de inicio de sesión y cierre de sesión | 50 |

El ejemplo de esta tabla corresponde a un solo usuario, pero se puede usar para calcular los requisitos del número total de usuarios en su entorno. Por ejemplo, necesitaría aproximadamente 1000 IOPS para 100 usuarios, y alrededor de 5000 IOPS durante el inicio y cierre de sesión.

## <a name="is-there-a-scale-limit-for-host-pools-created-in-the-azure-portal"></a>¿Hay un límite de escala para los grupos de hosts creados en Azure Portal?

Los factores siguientes pueden afectar al límite de escala de los grupos de hosts:

- La plantilla de Azure está limitada a 800 objetos. Para obtener más información, consulte [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure](../azure-resource-manager/management/azure-subscription-service-limits.md#template-limits). Cada VM crea también aproximadamente seis objetos; es decir, que puede crear alrededor de 132 VM cada vez que ejecuta la plantilla.

- Existen restricciones en cuanto a la cantidad de núcleos que se pueden crear por región y por suscripción. Por ejemplo, si tiene una suscripción de Contrato Enterprise, puede crear 350 núcleos. Tendrá que dividir 350 por el número predeterminado de núcleos por VM o por su propio límite de núcleos para conocer el número de VM que puede crear cada vez que ejecuta la plantilla. Más información sobre los [Límites de Virtual Machines: Azure Resource Manager](../azure-resource-manager/management/azure-subscription-service-limits.md#virtual-machines-limits---azure-resource-manager).

- El nombre del prefijo de la VM y el número de VM tiene menos de 15 caracteres. Para obtener más información, consulte [Reglas y restricciones de nomenclatura para los recursos de Azure](../azure-resource-manager/management/resource-name-rules.md#microsoftcompute).

## <a name="can-i-manage-azure-virtual-desktop-environments-with-azure-lighthouse"></a>¿Puedo administrar entornos de Azure Virtual Desktop con Azure Lighthouse?

Azure Lighthouse no es totalmente compatible con la administración de entornos de Azure Virtual Desktop. Dado que Lighthouse actualmente no admite la administración de usuarios inquilinos entre instancias de Azure AD, los clientes de Lighthouse aún tienen que iniciar sesión en la instancia de Azure AD que los clientes usan para administrar usuarios.

Tampoco puede usar las suscripciones de espacio aislado de CSP con el servicio Azure Virtual Desktop. Para obtener más información, consulte [Cuenta de espacio aislado de integración](/partner-center/develop/set-up-api-access-in-partner-center#integration-sandbox-account).

Por último, si ha habilitado el proveedor de recursos de la cuenta de propietario de CSP, las cuentas de cliente de CSP no podrán modificar el proveedor de recursos.

## <a name="how-often-should-i-turn-my-vms-on-to-prevent-registration-issues"></a>¿Con qué frecuencia debo activar mis máquinas virtuales para evitar problemas de registro?

Después de registrar una máquina virtual en un grupo de hosts en el servicio Azure Virtual Desktop, el agente actualiza periódicamente el token de la máquina virtual cada vez que la máquina virtual está activa. El certificado para el token de registro es válido durante 90 días. Debido a este límite de 90 días, se recomienda que las máquinas virtuales estén en línea durante 20 minutos cada 90 días, a fin de que puedan pueda actualizar sus tokens y actualizar el agente y los componentes de la pila en paralelo. La activación de la máquina virtual en este límite de tiempo impedirá que su token de registro expire o deje de ser válido. Si ha iniciado la máquina virtual después de 90 días y está experimentando problemas de registro, siga las instrucciones de la [guía de solución de problemas del agente de Azure Virtual Desktop](troubleshoot-agent.md#your-issue-isnt-listed-here-or-wasnt-resolved) para quitar la máquina virtual del grupo de hosts, reinstalar el agente y volver a registrarlo en el grupo.

## <a name="can-i-set-availability-options-when-creating-host-pools"></a>¿Se pueden establecer las opciones de disponibilidad al crear grupos de hosts?

Sí. Los grupos de hosts de Azure Virtual Desktop tienen una opción para seleccionar el conjunto de disponibilidad o las zonas de disponibilidad cuando se crea una máquina virtual. Estas opciones de disponibilidad son las mismas que usa Azure Compute. Si selecciona una zona para la máquina virtual que crea en un grupo de hosts, la configuración se aplica automáticamente a todas las máquinas virtuales que cree en esa zona. Si prefiere distribuir las máquinas virtuales del grupo de hosts en varias zonas, deberá seguir las instrucciones de [Agregar máquinas virtuales con el Azure Portal](expand-existing-host-pool.md#add-virtual-machines-with-the-azure-portal) para seleccionar manualmente una zona nueva para cada nueva máquina virtual que cree.

## <a name="which-availability-option-is-best-for-me"></a>¿Qué opción de disponibilidad es la más adecuada para mí?

La opción de disponibilidad que debe usar para las máquinas virtuales depende de la ubicación de la imagen y de sus campos de disco administrado. En la tabla siguiente se explica la relación que tiene cada valor de configuración con estas variables para ayudarle a averiguar qué opción es la mejor para su implementación. 

| Opción de disponibilidad | Ubicación de las imágenes | Botón de opción usar disco administrado (botón de radio) |
|---|---|---|
| None | Galería | Deshabilitado con "Sí" como predeterminado |
| None | Blob Storage | Habilitado con "No" como predeterminado |
| Zona de disponibilidad | Galería (opción de almacenamiento de blob deshabilitada) | Deshabilitado con "Sí" como predeterminado |
| Conjunto de disponibilidad con SKU administrada (disco administrado) | Galería | Deshabilitado con "Sí" como predeterminado |
| Conjunto de disponibilidad con SKU administrada (disco administrado) | Blob Storage | Habilitado con "No" como predeterminado |
| Conjunto de disponibilidad con SKU administrada (disco administrado) | Almacenamiento Blob (opción de la Galería deshabilitada) | Deshabilitado con "No" como valor predeterminado |
| Conjunto de disponibilidad (creado recientemente por el usuario) | Galería | Deshabilitado con "Sí" como predeterminado |
| Conjunto de disponibilidad (creado recientemente por el usuario) | Blob Storage | Habilitado con "No" como predeterminado |
