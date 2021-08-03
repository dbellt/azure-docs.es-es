---
title: Actualizaciones de la plataforma para Azure VMware Solution
description: Más información sobre las actualizaciones de la plataforma en Azure VMware Solution.
ms.topic: reference
ms.date: 05/26/2021
ms.openlocfilehash: 38cd183b783989f6a682b68e92e490354b032750
ms.sourcegitcommit: 9ad20581c9fe2c35339acc34d74d0d9cb38eb9aa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110538070"
---
# <a name="platform-updates-for-azure-vmware-solution"></a>Actualizaciones de la plataforma para Azure VMware Solution

Azure VMware Solution va a aplicar actualizaciones importantes a partir de marzo de 2021. Va a recibir una notificación por medio de Azure Service Health con la escala de tiempo del mantenimiento. Para más información, consulte [Mantenimiento del host y administración del ciclo de vida](concepts-private-clouds-clusters.md#host-maintenance-and-lifecycle-management).

## <a name="may-25-2021"></a>25 de mayo de 2021
Según el aviso de seguridad de VMware [VMSA-2021-0010](https://www.vmware.com/security/advisories/VMSA-2021-0010.html), se ha informado a VMware de múltiples vulnerabilidades en VMware ESXi y vSphere Client (HTML5). 

Para hacer frente a las vulnerabilidades ([CVE-2021-21985](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-21985) y [CVE-2021-21986](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-21986)) notificadas en el aviso de seguridad de VMware [VMSA-2021-0010](https://www.vmware.com/security/advisories/VMSA-2021-0010.html), se ha actualizado vCenter Server en todas las nubes privadas de Azure VMware Solution.

No es necesario hacer nada.

## <a name="may-21-2021"></a>21 de mayo de 2021
 
El servicio Azure VMware Solution realizará tareas de mantenimiento hasta el 23 de mayo de 2021 para aplicar actualizaciones importantes a vCenter Server en su nube privada.  Va a recibir una notificación por medio de Azure Service Health con la escala de tiempo del mantenimiento de su nube privada.
 
Durante este tiempo, VMware vCenter no estará disponible y no podrá administrar las máquinas virtuales (detener, iniciar, crear o eliminar). Se recomienda que, durante este tiempo, no planee ninguna otra actividad, como el escalado vertical de la nube privada, la creación de nuevas redes, entre otras, en la nube privada.
 
No hay ningún impacto en las cargas de trabajo que se ejecutan en la nube privada.


## <a name="april-26-2021"></a>26 de abril de 2021
Todas las nuevas nubes privadas de Azure VMware Solution se implementan ahora con VMware vCenter, versión 6.7U3l y NSX-T, versión 2.5.2. No se usa NSX-T 3.1.1 en este escenario debido a un problema identificado en este entorno que afecta a la conectividad de las máquinas virtuales del cliente. 

La mitigación recomendada de VMware se aplicó a todas las nubes privadas existentes que actualmente ejecutan NSX-T 3.1.1 en Azure VMware Solution. Se ha confirmado que la solución alternativa no tiene afecto alguno sobre la conectividad de la máquina virtual del cliente.

## <a name="march-24-2021"></a>24 de marzo de 2021
Todas las nubes privadas de Azure VMware Solution se implementan con VMware vCenter, versión 6.7U3l y NSX-T, versión 3.1.1. Todas las nubes privadas existentes se actualizarán a las versiones mencionadas anteriormente **hasta junio de 2021**.

Va a recibir un correo electrónico con la fecha y hora de mantenimiento planeado. Puede volver a programar una actualización. El correo electrónico también proporciona detalles sobre el componente actualizado, su efecto en las cargas de trabajo, el acceso a la nube privada y otros servicios de Azure.  Una hora antes de la actualización va a recibir una notificación y luego otra cuando termine.

## <a name="march-15-2021"></a>15 de marzo de 2021 

- El servicio Azure VMware Solution va a efectuar tareas de mantenimiento **hasta el 19 de marzo de 2021** para actualizar la instancia de vCenter Server de la nube privada a la versión vCenter Server 6.7 Update 3l.

- VMware vCenter no estará disponible durante este tiempo, por lo que no podrá administrar las máquinas virtuales (detener, iniciar, crear, eliminar) ni el escalado de la nube privada (agregar o quitar servidores y clústeres). Pero la alta disponibilidad de VMware va a seguir funcionando para proteger las máquinas virtuales existentes. 
 
Para obtener más información sobre esta versión de vCenter, vea [Notas de la versión de VMware vCenter Server 6.7, actualización 3l](https://docs.vmware.com/en/VMware-vSphere/6.7/rn/vsphere-vcenter-server-67u3l-release-notes.html).

## <a name="march-4-2021"></a>4 de marzo de 2021

- Azure VMware Solution va a aplicar [VMware ESXi 6.7, Patch Release ESXi670-202011002](https://docs.vmware.com/en/VMware-vSphere/6.7/rn/esxi670-202011002.html) a las nubes privadas existentes **hasta el 15 de marzo de 2021**.

- Las soluciones alternativas documentadas para la pila de vSphere, según [VMSA-2021-0002](https://www.vmware.com/security/advisories/VMSA-2021-0002.html), también se van a aplicar **hasta el 15 de marzo de 2021**.

>[!NOTE]
>Esto no provoca interrupciones y no debe afectar a los servicios o las cargas de trabajo de Azure VMware. Durante el mantenimiento, varias alertas de VMware, como _Lost network connectivity on DVPorts_ (Conectividad de red perdida en DVPorts) y _Lost uplink redundancy on DVPorts_ (Redundancia de vínculo superior perdido en DVPorts), aparecen en vCenter y se van borrando automáticamente a medida que progresa el mantenimiento.

## <a name="post-update"></a>Después de la actualización
Una vez finalizada, aparecen las versiones más recientes de los componentes de VMware. Si nota algún problema o tiene alguna pregunta, póngase en contacto con nuestro equipo de soporte técnico al abrir una incidencia de soporte técnico.
