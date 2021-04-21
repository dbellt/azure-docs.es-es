---
title: Ciclo de vida del soporte técnico de Red Hat OpenShift en Azure
description: Descripción del ciclo de vida de soporte técnico y las versiones admitidas de Red Hat OpenShift en Azure
author: sakthi-vetrivel
ms.author: suvetriv
ms.service: azure-redhat-openshift
ms.topic: conceptual
ms.date: 03/06/2021
ms.openlocfilehash: 35cbaf2b6d2618d116d2dd322ae5ac28f578cecc
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107303549"
---
# <a name="support-lifecycle-for-azure-red-hat-openshift-4"></a>Ciclo de vida del soporte técnico de Red Hat OpenShift en Azure 4

Red Hat publica versiones secundarias de Red Hat OpenShift Container Platform (OCP) aproximadamente cada tres meses. Estas versiones incluyen nuevas características y mejoras. Las versiones de revisión son más frecuentes (normalmente semanales) y solo están previstas para correcciones de errores críticos en una versión secundaria. Estas versiones de revisión pueden incluir correcciones para vulnerabilidades de seguridad o errores importantes.

Red Hat OpenShift en Azure se crea a partir de versiones específicas de OCP. En este artículo se describen las versiones de OCP que se admiten en Red Hat OpenShift en Azure y se detallan las actualizaciones, las caídas en desuso y la directiva de soporte técnico.

## <a name="red-hat-openshift-versions"></a>Versiones de Red Hat OpenShift

Red Hat OpenShift Container Platform usa el control de versiones semántico. El control de versiones semántico utiliza diferentes niveles de número de versión para especificar diferentes niveles de control de versiones. En la tabla siguiente se muestran las distintas partes de un número de versión semántico, en este caso, con el número de versión de ejemplo 4.4.11.

|Versión principal (x)|Versión secundaria (y)|Revisión (z)|
|-|-|-|
|4|4|11|

Cada número en la versión indica compatibilidad general con la versión anterior:

* **Versión principal**: no se han planeado versiones principales en este momento. Las versiones principales cambian cuando cambia una API incompatible o la compatibilidad con versiones anteriores deja de funcionar.
* **Versión secundaria**: publicada aproximadamente cada tres meses. Las actualizaciones de versiones secundarias pueden incluir adiciones de características, mejoras, caídas en desuso, eliminaciones, correcciones de errores, mejoras de seguridad y otras actualizaciones.
* **Revisiones**: normalmente se publican cada semana, o según sea necesario. Las actualizaciones de versiones de revisión pueden incluir correcciones de errores, mejoras de seguridad y otras actualizaciones.

Los clientes deben tratar de ejecutar la última versión secundaria de su versión principal. Por ejemplo, si el clúster de producción ejecuta la versión 4.4 y 4.5 es la última versión secundaria disponible con carácter general para la serie 4, debe actualizar a 4.5 tan pronto como pueda. 

### <a name="upgrade-channels"></a>Canales de actualización

Los canales de actualización están vinculados a una versión secundaria de Red Hat OpenShift Container Platform (OCP). Por ejemplo, los canales de actualización de OCP 4.4 nunca incluirán una actualización a una versión 4.5. Los canales de actualización controlan solo la selección de versión y no afectan a la versión del clúster.

Red Hat OpenShift 4 en Azure admite solo canales estables. Por ejemplo: estable-4.4.

Puede usar el canal estable-4.5 para actualizar desde una versión secundaria anterior de Red Hat OpenShift en Azure. No se admitirán los clústeres actualizados con canales rápidos, de versión preliminar y candidatos.

Si cambia a un canal que no incluye la versión actual, se muestra una alerta y no se recomienda ninguna actualización, pero puede volver a cambiar de forma segura al canal original en cualquier momento.

## <a name="red-hat-openshift-container-platform-version-support-policy"></a>Directiva de compatibilidad de la versión de Red Hat OpenShift Container Platform

Red Hat OpenShift en Azure admite dos versiones secundarias de disponibilidad general (GA) de Red Hat OpenShift Container Platform:
* La versión más reciente de GA, publicada en Red Hat OpenShift en Azure (a la que nos referiremos como N)
* Una versión secundaria anterior (N-1)

Si está disponible en un canal de actualización estable, también se pueden admitir las versiones secundarias más recientes (N+1, N+2) disponibles en las versiones de OCP anteriores.

Las actualizaciones de revisión críticas se aplican automáticamente a los clústeres por parte de los ingenieros de confiabilidad de sitios de Red Hat OpenShift en Azure. Los clientes que deseen instalar las actualizaciones de revisión con anterioridad pueden hacerlo libremente.

Por ejemplo, si Red Hat OpenShift en Azure publica 4.5.z hoy, se proporciona soporte técnico para las siguientes versiones:

|Nueva versión secundaria|Lista de versiones admitidas|
|-|-|
|4.5.z|4.5.z, 4.4.z|

"z" representa las versiones de revisión. Si está disponible en un canal de actualización estable, los clientes también pueden actualizar a 4.6.z.

Cuando se publica una nueva versión secundaria, la versión secundaria más antigua cae en desuso y se quita. Por ejemplo, pongamos que la lista de versiones admitidas actual es 4.5.z y 4.4.z. Cuando Red Hat OpenShift en Azure publique 4.6.z, se quitará la versión 4.4.z y dejará de ser compatible en un plazo de 30 días.

> [!NOTE]
> Tenga en cuenta que si los clientes ejecutan una versión incompatible de Red Hat OpenShift, se les podría pedir que actualicen al solicitar soporte técnico para el clúster. Los clústeres que ejecutan versiones de Red Hat OpenShift no compatibles no están incluidos en el contrato de nivel de servicio de Red Hat OpenShift en Azure.

## <a name="release-and-deprecation-process"></a>Proceso de publicación y desuso

Puede consultar las próximas publicaciones de versiones y las que se quedarán obsoletas en el calendario publicación de Red Hat OpenShift en Azure.

Para las nuevas versiones secundarias de Red Hat OpenShift Container Platform:
* El equipo de ingenieros de confiabilidad de sitios de Red Hat OpenShift en Azure publica un anuncio previo con la fecha planificada de la nueva publicación de una versión y el respectivo desuso de una versión antigua en las [notas de la versión de Red Hat OpenShift en Azure](https://github.com/Azure/OpenShift/releases) al menos 30 días antes de la eliminación.
* Este equipo publica una notificación de estado del servicio disponible para todos los clientes con Red Hat OpenShift en Azure y acceso al portal, y envía un correo electrónico a los administradores de suscripciones con las fechas de eliminación de versión planeadas.
* Los clientes tienen 30 días a partir de la eliminación de una versión para actualizar a una versión secundaria compatible para seguir recibiendo soporte técnico.

Para las nuevas versiones de revisión de Red Hat OpenShift Container Platform:
* Debido a la naturaleza urgente de las versiones de revisión, se pueden introducir en el servicio por parte del equipo de ingenieros de confiabilidad de sitios de Red Hat OpenShift en Azure a medida que estén disponibles.
* En general, este equipo no realiza grandes comunicaciones para la instalación de nuevas versiones de revisión. Sin embargo, supervisa y valida constantemente las revisiones de CVE disponibles para admitirlas de manera puntual. Si se requiere una acción del cliente, el equipo enviará una notificación sobre la actualización.

## <a name="supported-versions-policy-exceptions"></a>Excepciones de directiva de versiones admitidas

El equipo de ingenieros de confiabilidad de sitios de Red Hat OpenShift en Azure se reserva el derecho de agregar o eliminar versiones nuevas/existentes, o aplazar próximas versiones secundarias, donde se hayan identificado uno o varios problemas de seguridad o errores críticos que afecten a la producción sin previo aviso.

Las versiones de revisión específicas se pueden omitir, o su implantación puede verse acelerada en función de la gravedad del error o el problema de seguridad.

## <a name="azure-portal-and-cli-versions"></a>Versiones de Azure Portal y CLI

Al implementar un clúster de Red Hat OpenShift en Azure en el portal o con la CLI de Azure, el valor predeterminado del clúster es la última versión secundaria (N) y la revisión crítica más reciente. Por ejemplo, si Red Hat OpenShift en Azure admite 4.5.z y 4.4.z, la versión predeterminada para las nuevas instalaciones es 4.5.z. Los clientes que quieran usar la última versión secundaria de OCP anterior (N+1, N+2) pueden actualizar su clúster en cualquier momento a cualquier versión disponible en los canales de actualización estables.

## <a name="azure-red-hat-openshift-release-calendar"></a>Calendario de versiones de Red Hat OpenShift en Azure

Consulte la siguiente guía para ver el [historial anterior de versiones (precedentes) de OpenShift Container Platform](https://access.redhat.com/support/policy/updates/openshift/#dates).

|Versión de OCP|Versión anterior|Disponibilidad general de Red Hat OpenShift en Azure|Fin de la vida útil|
|-|-|-|-|
|4.3|Enero de 2020|Abril de 2020| Agosto de 2020|
|4.4.|Mayo de 2020|Julio de 2020|4.6 Disponibilidad general|
|4,5|Julio de 2020| Noviembre de 2020|4.7 GA
|4,6|Octubre de 2020| Febrero de 2021|4.8 GA|

## <a name="faq"></a>Preguntas más frecuentes

**¿Qué ocurre cuando un cliente actualiza un clúster de OpenShift con una versión secundaria que no es compatible?**

Si está en la versión N–2 u otra anterior, quedará fuera del soporte técnico y se le pedirá que actualice la versión para seguir recibiéndolo. Si la versión N–2 se actualiza correctamente con la N–1, volverá a cumplir nuestras directivas para recibir soporte técnico. Puede que resulte difícil y, en algunos casos, imposible actualizar versiones N–3 o anteriores con otra compatible. Se recomienda mantener el clúster con la versión más reciente de OpenShift para evitar posibles problemas de actualización. Por ejemplo:
* Si la versión de Red Hat OpenShift en Azure admitida más antigua es la 4.4.z y la suya es la 4.3.z o anterior, está fuera del soporte técnico.
* Si la actualización de 4.3.z a 4.4.z se realiza correctamente, estará de nuevo dentro de nuestras directivas de soporte técnico. 

No se admite la reversión del clúster a una versión anterior. Solo se admite la actualización a una versión más reciente.

**¿Qué significa "fuera de soporte técnico"?**

Si el clúster de ARO ejecuta una versión de OpenShift que no está en la lista de versiones admitidas o usa una [configuración de clúster no admitida](https://docs.microsoft.com/azure/openshift/support-policies-v4), dicho clúster queda “fuera del soporte técnico”. Como resultado:
- Al abrir una incidencia de soporte técnico para el clúster, se le pedirá que lo actualice con una versión compatible antes de recibir el soporte, a menos que esté en el período de gracia de 30 días desde la finalización del soporte técnico de la versión. 
- Las garantías de tiempo de ejecución o contrato de nivel de servicio para clústeres fuera del soporte técnico quedan anuladas.
- Los clústeres fuera del soporte técnico se revisarán dentro de lo posible.
- Estos clústeres no se supervisarán.
