---
title: Introducción al ejemplo de plano técnico de Azure Security Benchmark Foundation
description: Introducción y arquitectura del ejemplo de plano técnico de Azure Security Benchmark Foundation.
ms.date: 03/12/2021
ms.topic: sample
ms.openlocfilehash: a60f70b1f3e9d810ccded8bbeee726ea950524e9
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108739266"
---
# <a name="overview-of-the-azure-security-benchmark-foundation-blueprint-sample"></a>Introducción al ejemplo de plano técnico de Azure Security Benchmark Foundation

El ejemplo de plano técnico de Azure Security Benchmark Foundation proporciona un conjunto de patrones de infraestructura de la linea base que le ayuda a crear un entorno seguro y compatible con Azure. El plano técnico le ayuda a implementar una arquitectura basada en la nube que ofrece soluciones para escenarios que tienen requisitos de acreditación o de cumplimiento. Este ejemplo de plano técnico fundamental es una extensión del [plano técnico de ejemplo de Azure Security Benchmark](../azure-security-benchmark.md). Implementa y configura los límites de la red, la supervisión y otros recurso en consonancia con las directivas y los límites de protección definidos en [Azure Security Benchmark](../../../../security/benchmarks/index.yml).

## <a name="architecture"></a>Architecture

El entorno fundamental creado por este ejemplo de plano técnico se basa en los principios arquitectónicos de un [modelo en estrella tipo hub-and-spoke](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke).
El plano técnico implementa una red virtual de centro de conectividad que contiene recursos comunes y compartidos, servicios y artefactos como Azure Bastion, puerta de enlace y firewall para ofrecer conectividad y administración, y subredes de jumpbox para hospedar una infraestructura adicional/opcional de conectividad, administración y mantenimiento. Se implementan una o varias redes virtuales de radio para hospedar cargas de trabajo de aplicaciones, como servicios web y de base de datos. Las redes virtuales de radio se conectan a la red virtual del centro de conectividad mediante el emparejamiento de red virtual de Azure para lograr una conectividad segura y sin problemas. Se pueden agregar radios adicionales mediante la reasignación del plano técnico de ejemplo o la creación manual de una red virtual de Azure y su emparejamiento con la red virtual del centro de conectividad. Toda la conectividad externa a las redes y subredes virtuales de radio está configurada para enrutarse a través de la red virtual del centro de conectividad, así como del firewall, la puerta de enlace y los jumpboxes de administración.

:::image type="content" source="../../media/azure-security-benchmark-foundation/architecture.png" alt-text="Diagrama de la arquitectura del ejemplo de plano técnico de Azure Security Benchmark Foundation" border="false":::

Este plano técnico implementa varios servicios de Azure para proporcionar una base segura, administrada y lista para la empresa. Este entorno se compone de:

- [Registros de Azure Monitor](../../../../azure-monitor/logs/data-platform-logs.md) y una cuenta de Azure Monitor para asegurarse de que los registros de recursos, los registros de actividad, las métricas y los flujos del tráfico de redes se almacenan en una ubicación central para facilitar la realización de consultas, análisis, archivado y generación de alertas.
- [Azure Security Center](../../../../security-center/security-center-introduction.md) (versión estándar) para proporcionar protección contra amenazas a los recursos de Azure.
- [Azure Virtual Network](../../../../virtual-network/virtual-networks-overview.md) en el centro de conectividad que da soporte a las subredes para devolver la conectividad a una red local, una pila de entrada y salida para la conectividad a Internet y subredes opcionales para la implementación de servicios de administración. Microsoft Azure Virtual Network en el radio contiene subredes que hospedan las cargas de trabajo de las aplicaciones. Después de la implementación se pueden crear tantas subredes como sea necesario para dar soporte a los escenarios aplicables.
- [Azure Firewall](../../../../firewall/overview.md) para enrutar todo el tráfico de Internet saliente y para habilitar el tráfico de Internet entrante a través de jumpbox (las reglas de firewall predeterminadas bloquean todo el tráfico de entrada y salida de Internet, y se deben configurar reglas después de la implementación como corresponda).
- [Grupos de seguridad de red](../../../../virtual-network/network-security-group-how-it-works.md) asignados a todas las subredes (excepto las subredes que sean propiedad del servicio, como Azure Bastion, Gateway y Azure Firewall) configurados para bloquear todo el tráfico de entrada y salida de Internet.
- [Grupo de seguridad de aplicaciones](../../../../virtual-network/application-security-groups.md) que permiten la agrupación de máquinas virtuales de Azure para aplicar directivas de seguridad de red comunes.
- [Tablas de enrutamiento](../../../../virtual-network/manage-route-table.md) para enrutar todo el tráfico de Internet saliente de las subredes a través del firewall (después de la implementación será preciso configurar Azure Firewall y las reglas de NSG para abrir la conectividad).
- [Azure Network Watcher](../../../../network-watcher/network-watcher-monitoring-overview.md) para supervisar, diagnosticar y ver las métricas de los recursos de una red virtual de Azure.
- [Azure DDoS Protection Standard](../../../../ddos-protection/ddos-protection-overview.md) para proteger los recursos de Azure contra ataques de DDoS.
- [Azure Bastion](../../../../bastion/bastion-overview.md) para proporcionar una conectividad segura y sin problemas a una máquina virtual que no requiere una dirección IP pública, un agente o un software cliente especial.
- [Azure VPN Gateway](../../../../vpn-gateway/vpn-gateway-about-vpngateways.md) para habilitar el tráfico cifrado entre una red virtual de Azure y una ubicación local a través de la red pública de Internet.

> [!NOTE]
> Azure Security Benchmark Foundation diseña una arquitectura básica para las cargas de trabajo. El diagrama arquitectónico anterior incluye varios recursos nocionales para demostrar el uso de subredes. Deberá implementar cargas de trabajo adicionales en esta arquitectura básica.

## <a name="next-steps"></a>Pasos siguientes

Ha examinado la introducción y arquitectura del ejemplo de plano técnico de Azure Security Benchmark Foundation.

> [!div class="nextstepaction"]
> [Plano técnico de Azure Security Benchmark Foundation: pasos de implementación](./deploy.md)

Artículos adicionales sobre planos técnicos y cómo utilizarlos:

- Información acerca del [ciclo de vida del plano técnico](../../concepts/lifecycle.md).
- Descubra cómo utilizar [parámetros estáticos y dinámicos](../../concepts/parameters.md).
- Aprenda a personalizar el [orden de secuenciación de planos técnicos](../../concepts/sequencing-order.md).
- Averigüe cómo usar el [bloqueo de recursos de planos técnicos](../../concepts/resource-locking.md).
- Aprenda a [actualizar las asignaciones existentes](../../how-to/update-existing-assignments.md).
