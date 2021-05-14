---
title: archivo de inclusión
description: archivo de inclusión
services: bastion
author: cherylmc
ms.service: bastion
ms.topic: include
ms.date: 02/05/2021
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: c0ca74b6cf63e3c81d830eca76eea6815548faa1
ms.sourcegitcommit: 6f1aa680588f5db41ed7fc78c934452d468ddb84
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107732551"
---
### <a name="do-i-need-a-public-ip-on-my-virtual-machine-to-connect-via-azure-bastion"></a><a name="publicip"></a>¿Necesito una dirección IP pública en mi máquina virtual para conectarme mediante Azure Bastion?

No. Si va a conectarse a una máquina virtual mediante Azure Bastion, no necesita una dirección IP pública en la máquina virtual de Azure a la que se va a conectar. El servicio Bastion abrirá la sesión o conexión RDP/SSH a la máquina virtual a través de la dirección IP privada de su máquina virtual dentro de su red virtual.

### <a name="is-ipv6-supported"></a>¿Se admite IPv6?

Actualmente, IPv6 no se admite. Azure Bastion solo admite IPv4.

### <a name="can-i-use-azure-bastion-with-azure-private-dns-zones"></a>¿Puedo usar Azure Bastion con zonas de DNS privado de Azure?

En este momento no se admite el uso de Azure Bastion con las zonas de DNS privado de Azure. Antes de implementar el recurso de Azure Bastion, asegúrese de que la red virtual del host no esté vinculada a una zona de DNS privado.

### <a name="do-i-need-an-rdp-or-ssh-client"></a><a name="rdpssh"></a>¿Necesito un cliente RDP o SSH?

No. No es necesario un cliente RDP o SSH para acceder mediante el protocolo de escritorio remoto o Secure Shell a su máquina virtual de Azure en Azure Portal. Use [Azure Portal](https://portal.azure.com) para que RDP/SSH acceda a su máquina virtual directamente en el explorador.

### <a name="do-i-need-an-agent-running-in-the-azure-virtual-machine"></a><a name="agent"></a>¿Es necesario que un agente se ejecute en la máquina virtual de Azure?

No. No es preciso instalar un agente ni ningún otro software en el explorador ni en la máquina virtual de Azure. El servicio Bastion no utiliza agentes y no requiere software adicional para RDP y SSH.

### <a name="how-many-concurrent-rdp-and-ssh-sessions-does-each-azure-bastion-support"></a><a name="limits"></a>¿Cuántas sesiones RDP y SSH simultáneas admite cada instancia de Azure Bastion?

Tanto RDP como SSH son protocolos basados en el uso. El uso intensivo de las sesiones hará que el host bastión admita un número total de sesiones inferior. Las siguientes cifras presuponen unos flujos de trabajo normales rutinarios.

[!INCLUDE [limits](bastion-limits.md)]

### <a name="what-features-are-supported-in-an-rdp-session"></a><a name="rdpfeaturesupport"></a>¿Qué características se admiten en una sesión de RDP?

En este momento, solo se admiten las operaciones de copiado y pegado de texto. No se admiten características como la copia de archivos. No dude en compartir sus comentarios sobre nuevas características en la [página de comentarios de Azure Bastion](https://feedback.azure.com/forums/217313-networking?category_id=367303).

### <a name="does-bastion-hardening-work-with-aadj-vm-extension-joined-vms"></a><a name="aadj"></a>¿Funciona la protección de Bastion con las máquinas virtuales unidas mediante extensión a una máquina virtual AADJ?

Esta característica no funciona con las máquinas unidas mediante extensión que utilizan usuarios de Azure AD. Para más información, consulte la sección sobre [máquinas virtuales Azure con Windows y Azure AD](../articles/active-directory/devices/howto-vm-sign-in-azure-ad-windows.md#requirements).

### <a name="which-browsers-are-supported"></a><a name="browsers"></a>¿Qué exploradores se admiten?

El explorador debe admitir HTML 5. Utilice el explorador Microsoft Edge o Google Chrome en Windows. Para Apple Mac, use Google Chrome. Microsoft Edge Chromium también es compatible tanto con Windows como con Mac.

### <a name="where-does-azure-bastion-store-customer-data"></a><a name="data"></a>¿Dónde se almacenan los datos de los clientes en Azure Bastion?

Azure Bastion no mueve ni almacena los datos de los clientes fuera de la región en la que se implementa.

### <a name="are-any-roles-required-to-access-a-virtual-machine"></a><a name="roles"></a>¿Se necesitan roles para acceder a una máquina virtual?

Para crear una conexión, se requieren los siguientes roles:

* Rol de lector en la máquina virtual
* Rol de lector en la tarjeta de interfaz de red con la dirección IP privada de la máquina virtual
* Rol de lector en el recurso de Azure Bastion

### <a name="what-is-the-pricing"></a><a name="pricingpage"></a>¿Cuáles son los precios?

Consulte la [página de precios](https://aka.ms/BastionHostPricing)para obtener más información.

### <a name="does-azure-bastion-require-an-rds-cal-for-administrative-purposes-on-azure-hosted-vms"></a><a name="rdscal"></a>¿Requiere Azure Bastion una CAL de RDS con fines administrativos en máquinas virtuales hospedadas en Azure?

No, el acceso a las máquinas virtuales Windows Server con Azure Bastion no requiere una [CAL de RDS](https://www.microsoft.com/p/windows-server-remote-desktop-services-cal/dg7gmgf0dvsv?activetab=pivot:overviewtab) cuando se utiliza únicamente con fines administrativos.

### <a name="which-keyboard-layouts-are-supported-during-the-bastion-remote-session"></a><a name="keyboard"></a>¿Qué distribuciones de teclado se admiten durante la sesión remota de Bastion?

Actualmente, Azure Bastion admite la distribución de teclado en-US-QWERTY dentro de la máquina virtual.  La compatibilidad con otras configuraciones regionales para la distribución del teclado está en curso.

### <a name="is-user-defined-routing-udr-supported-on-an-azure-bastion-subnet"></a><a name="udr"></a>¿Se admite el enrutamiento definido por el usuario (UDR) en las subredes de Azure Bastion?

No. UDR no es compatible con las subredes de Azure Bastion.

En los escenarios que incluyen Azure Bastion y Azure Firewall/Aplicación virtual de red (NVA) en la misma red virtual, no es preciso forzar el tráfico de una subred de Azure Bastion a Azure Firewall, ya que la comunicación entre Azure Bastion y las máquinas virtuales es privada. Para más información, consulte [Acceso a las máquinas virtuales que están detrás de Azure Firewall con Bastion](https://azure.microsoft.com/blog/accessing-virtual-machines-behind-azure-firewall-with-azure-bastion/).

### <a name="why-do-i-get-your-session-has-expired-error-message-before-the-bastion-session-starts"></a><a name="session"></a>¿Por qué obtengo el error "Su sesión ha expirado" antes de iniciar la sesión de Bastion?

Las sesiones deben iniciarse solo en Azure Portal. Inicie sesión en Azure Portal y vuelva a iniciar una sesión. Si va a la dirección URL directamente desde otra sesión del explorador o pestaña, este error es previsible. Ayuda a garantizar que la sesión sea más segura y que se solo se pueda acceder a la sesión desde Azure Portal.

### <a name="how-do-i-handle-deployment-failures"></a><a name="udr"></a>¿Cómo controlo los errores de implementación?

Examine los mensajes de error y [cree una solicitud de soporte técnico en Azure Portal](../articles/azure-portal/supportability/how-to-create-azure-support-request.md) si fuera necesario. Los errores de implementación pueden estar motivados por [límites, cuotas y restricciones de la suscripción de Azure](../articles/azure-resource-manager/management/azure-subscription-service-limits.md). Más concretamente, los clientes pueden encontrar un límite en cuanto al número de direcciones IP públicas que se permiten por suscripción y que provoca un error en la implementación de Azure Bastion.

### <a name="how-do-i-incorporate-azure-bastion-in-my-disaster-recovery-plan"></a><a name="dr"></a>¿Cómo puedo incorporar Azure Bastion en mi plan de recuperación ante desastres?

Azure Bastion se implementa en redes virtuales o redes virtuales emparejadas y está asociado a una región de Azure. Usted es responsable de implementar Azure Bastion en una red virtual del sitio de recuperación ante desastres (DR). En caso de que se produzca un error en la región de Azure, realice una operación de conmutación por error de las máquinas virtuales a la región de DR. A continuación, use el host de Azure Bastion implementado en la región de DR para conectarse a las máquinas virtuales que ahora están implementadas en ella.
