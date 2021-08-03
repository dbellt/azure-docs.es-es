---
title: Información general de hosts dedicados de Azure para máquinas virtuales
description: Obtenga más información sobre cómo se pueden usar los hosts dedicados de Azure para implementar máquinas virtuales.
author: cynthn
ms.service: virtual-machines
ms.subservice: dedicated-hosts
ms.topic: conceptual
ms.workload: infrastructure
ms.date: 12/07/2020
ms.author: cynthn
ms.reviewer: zivr
ms.openlocfilehash: d1bb111bbaa909c52261a0dbafe6d6e0a24390ac
ms.sourcegitcommit: 190658142b592db528c631a672fdde4692872fd8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112007269"
---
# <a name="azure-dedicated-hosts"></a>Hosts dedicados de Azure

Azure Dedicated Host es un servicio que proporciona servidores físicos (capaces de hospedar una o varias máquinas virtuales) dedicados a una suscripción a Azure. Los hosts dedicados son los mismos servidores físicos que se usan en nuestros centros de datos y se proporcionan como un recurso. Puede aprovisionar hosts dedicados dentro de una región, zona de disponibilidad y dominio de error. Después, puede colocar las máquinas virtuales directamente en los hosts aprovisionados, en la configuración que más se ajuste a sus necesidades.


## <a name="benefits"></a>Ventajas

La reserva de todo el host proporciona las siguientes ventajas:

-   Aislamiento del hardware a nivel de servidor físico. No se colocarán otras máquinas virtuales en los hosts. Los hosts dedicados se implementan en los mismos centros de datos y comparten la misma red y la misma infraestructura de almacenamiento subyacente que otros hosts no aislados.
-   Control sobre los eventos de mantenimiento iniciados por la plataforma Azure. Aunque la mayoría de los eventos de mantenimiento tienen poco o ningún impacto en las máquinas virtuales, hay algunas cargas de trabajo confidenciales en las que cada segundo de pausa puede tener un impacto. Con los hosts dedicados, puede participar en una ventana de mantenimiento para reducir el impacto en el servicio.
-   Con la ventaja híbrida de Azure, puede traer sus propias licencias para Windows y SQL a Azure. El uso de las ventajas híbridas proporciona ventajas adicionales. Para más información, consulte [Ventaja híbrida de Azure](https://azure.microsoft.com/pricing/hybrid-benefit/).


## <a name="groups-hosts-and-vms"></a>Grupos, hosts y máquinas virtuales

![Vista de los nuevos recursos para hosts dedicados.](./media/virtual-machines-common-dedicated-hosts/dedicated-hosts2.png)

Un **grupo host** es un recurso que representa una colección de hosts dedicados. Puede crear un grupo host en una región y una zona de disponibilidad, y agregarle hosts.

Un **host** es un recurso que se asigna a un servidor físico en un centro de datos de Azure. El servidor físico se asigna cuando se crea el host. Un host se crea dentro de un grupo host. Un host tiene una SKU que describe qué tamaños de máquina virtual se pueden crear. Cada host puede hospedar varias máquinas virtuales, de diferentes tamaños, siempre y cuando sean de la misma serie de tamaño.


## <a name="high-availability-considerations"></a>Consideraciones acerca de la alta disponibilidad

Para lograr una alta disponibilidad, debe implementar varias máquinas virtuales, que se distribuyen entre varios hosts (un mínimo de 2). Con Azure Dedicated Host, tiene varias opciones para aprovisionar su infraestructura para dar forma a sus límites de aislamiento de errores.

### <a name="use-availability-zones-for-fault-isolation"></a>Uso de las zonas de disponibilidad para el aislamiento de errores

Las zonas de disponibilidad son ubicaciones físicas exclusivas dentro de una región de Azure. Cada zona de disponibilidad consta de uno o varios centros de datos equipados con alimentación, refrigeración y redes independientes. Un grupo host se crea en una zona de disponibilidad individual. Una vez creada, todos los hosts se colocarán dentro de esa zona. Para lograr una alta disponibilidad entre zonas, es preciso crear varios grupos host (uno por zona) y distribuir los hosts en consecuencia.

Si asigna un grupo host a una zona de disponibilidad, todas las máquinas virtuales creadas en ese host deben crearse en la misma zona.

### <a name="use-fault-domains-for-fault-isolation"></a>Uso de dominios de error para el aislamiento de errores

Se puede crear un host en un dominio de error específico. Al igual que las máquinas virtuales de un conjunto de escalado o de un conjunto de disponibilidad, los hosts de distintos dominios de error se colocarán en distintos bastidores físicos del centro de datos. Al crear un grupo host, es necesario especificar el número de dominios de error. Al crear hosts en el grupo host, se asigna un dominio de error a cada host. Las máquinas virtuales no requieren ninguna asignación de dominio de error.

Los dominios de error no son lo mismo que la coubicación. Tener el mismo dominio de error para dos hosts no significa que estén próximos entre sí.

El ámbito de los dominios de error es el grupo host. No debe realizar ninguna suposición sobre la antiafinidad entre dos grupos host (a menos que se encuentren en zonas de disponibilidad diferentes).

Las máquinas virtuales implementadas en hosts con distintos dominios de error, tendrán sus servicios subyacentes de Managed Disks en varias marcas de almacenamiento, con el fin de aumentar la protección del aislamiento de errores.

### <a name="using-availability-zones-and-fault-domains"></a>Uso de zonas de disponibilidad y dominios de error

Puede usar ambas funcionalidades juntas para lograr un mayor aislamiento de los errores. En este caso, especificará la zona de disponibilidad y el número de dominios de error en cada grupo de hosts, asignará un dominio de error a cada uno de los hosts del grupo y asignará una zona de disponibilidad a cada una de las máquinas virtuales

La [plantilla de ejemplo de Resource Manager](https://github.com/Azure/azure-quickstart-templates/blob/master/quickstarts/microsoft.compute/vm-dedicated-hosts/README.md) usa zonas y dominios de error para distribuir los hosts a fin de conseguir la máxima resistencia en una región.


## <a name="manual-vs-automatic-placement"></a>Selección de ubicación manual frente a automático

Al crear una máquina virtual en Azure, puede seleccionar el host dedicado que se va a usar. También puede usar la opción para ubicar automáticamente las máquinas virtuales en los hosts existentes, dentro de un grupo host.

Al crear un grupo host, asegúrese de que esté seleccionada la opción de selección automática de ubicación de máquinas virtuales. Al crear la máquina virtual, seleccione el grupo host y deje que Azure elija el mejor host para la máquina virtual.

Los grupos host que están habilitados para la selección de ubicación automática no requieren que todas las máquinas virtuales se ubiquen de forma automática. Todavía podrá elegir de forma explícita un host, incluso cuando esté seleccionada la opción de selección de ubicación automática para el grupo host.

### <a name="limitations"></a>Limitaciones

Incidencias y limitaciones conocidas cuando se usa la selección automática de ubicación de máquinas virtuales:

- No se podrán aplicar las Ventajas híbridas de Azure en los hosts dedicados.
- No se podrá reimplementar la máquina virtual.
- No se podrán usar máquinas virtuales de las series Lsv2, NVasv4, NVsv3, Msv2 o M con hosts dedicados.


## <a name="virtual-machine-scale-set-support"></a>Compatibilidad con los conjuntos de escalado de máquinas virtuales

Los conjuntos de escalado de máquinas virtuales permiten tratar un grupo de máquinas virtuales como un único recurso y aplicar las directivas de disponibilidad, administración, escalado y orquestación como un grupo. Los hosts dedicados existentes también se pueden usar para conjuntos de escalado de máquinas virtuales.

Al crear un conjunto de escalado de máquinas virtuales, puede especificar un grupo host existente para que tenga todas las instancias de máquina virtual creadas en hosts dedicados.

Al crear un conjunto de escalado de máquinas virtuales en un grupo host dedicado, se aplican los siguientes requisitos:

- Es necesario habilitar la selección automática de ubicación de máquinas virtuales.
- La configuración de disponibilidad del grupo host debe coincidir con el conjunto de escalado.
    - Se debe usar un grupo host regional (creado sin especificar una zona de disponibilidad) para los conjuntos de escalado regionales.
    - El grupo host y el conjunto de escalado deben usar la misma zona de disponibilidad.
    - El recuento de dominios de error para el nivel de grupo host debe coincidir con el del conjunto de escalado. Azure Portal permite especificar la *propagación máxima* del conjunto de escalado, lo que establece el recuento de dominios de error de 1.
- Los hosts dedicados deben crearse en primer lugar, con capacidad suficiente y con la misma configuración para las zonas de conjunto de escalado y los dominios de error.
- Los tamaños de máquina virtual admitidos para los hosts dedicados deben coincidir con el del conjunto de escalado.

No todas las configuraciones de orquestación y optimización de conjunto de escalado son compatibles con los hosts dedicados. Aplique los valores siguientes en el conjunto de escalado:
- No se recomienda el sobreaprovisionamiento y está deshabilitado de forma predeterminada. Puede habilitar el sobreaprovisionamiento, pero se producirá un error en la asignación del conjunto de escalado si el grupo host no tiene capacidad para todas las máquinas virtuales, incluidas las instancias que se han sobreaprovisionado.
- Use el modo de orquestación ScaleSetVM.
- No use los grupo con ubicación por proximidad para la coubicación.



## <a name="maintenance-control"></a>Control de mantenimiento

En ocasiones, es posible que la infraestructura que da soporte a las máquinas virtuales se actualice para mejorar la confiabilidad, el rendimiento, la seguridad y para iniciar nuevas características. La plataforma Azure intenta minimizar el impacto del mantenimiento de la plataforma siempre que sea posible, pero los clientes con cargas de trabajo *sensibles al mantenimiento* no pueden tolerar los pocos segundos en los que la máquina virtual debe estar sin funcionamiento o congelada para realizar el mantenimiento.

El **control del mantenimiento** proporciona a los clientes una opción para omitir las actualizaciones de plataforma normales programadas en sus hosts dedicados y, después, aplicarlas en el momento que prefieran en un periodo acumulado de 35 días. Dentro de la ventana de mantenimiento, puede aplicar el mantenimiento directamente en el nivel de host, en cualquier orden. Una vez finalizada la ventana de mantenimiento, Microsoft avanzará y aplicará el mantenimiento pendiente a los hosts en un orden que puede no seguir los dominios de error definidos por el usuario.

Para obtener más información, consulte [Administración de las actualizaciones de la plataforma con el control de mantenimiento](./maintenance-control.md).

## <a name="capacity-considerations"></a>Consideraciones de capacidad

Una vez que se aprovisiona un host dedicado, Azure lo asigna a un servidor físico. De esta forma se garantiza la disponibilidad de la capacidad cuando se necesita aprovisionar la máquina virtual. Azure usa toda la capacidad de la región (o zona) para elegir un servidor físico para el host. También significa que los clientes pueden esperar poder aumentar la superficie de su host dedicado sin tener que preocuparse de quedarse sin espacio en el clúster.

## <a name="quotas"></a>Cuotas

Hay dos tipos de cuota que se consumen al implementar un host dedicado.

1. Cuota de vCPU de host dedicado. El límite de cuota predeterminado es 3000 vCPU por región.
1. Cuota de la familia de tamaños de máquina virtual. Por ejemplo, una suscripción de **pago por uso** solo puede tener una cuota de 10 vCPU disponibles para la serie de tamaños Dsv3 en la región Este de EE. UU. Para implementar un host dedicado Dsv3, tendría que solicitar un aumento de la cuota de al menos 64 vCPU antes de poder implementar el host dedicado.

Para solicitar un aumento de la cuota, cree una solicitud de soporte técnico en [Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

El aprovisionamiento de un host dedicado consumirá la cuota de vCPU de host dedicado y la cuota de vCPU de familia de VM, pero no consumirá la vCPU regional.


![Captura de pantalla de la página de uso y cuotas del portal](./media/virtual-machines-common-dedicated-hosts/quotas.png)

Para más información, consulte [Cuotas de vCPU de máquinas virtuales](./windows/quotas.md).

La evaluación gratuita y las suscripciones a MSDN no tienen cuota para los hosts dedicados de Azure.

## <a name="pricing"></a>Precios

A los usuarios se les cobra por host dedicado, independientemente del número de máquinas virtuales que se implementen. En el extracto mensual, verá un nuevo tipo de recurso facturable de hosts. Las máquinas virtuales de un host dedicado se seguirán mostrando en la instrucción, pero su precio será 0.

El precio del host se establece en función de la familia de máquinas virtuales, del tipo (tamaño de hardware) y de la región. El precio de un host tiene que ver con el tamaño de máquina virtual mayor que admita el host.

Las licencias de software, el almacenamiento y el uso de la red se facturan por separado del host y las máquinas virtuales. No hay ningún cambio en esos elementos facturables.

Para más información, consulte [Precios de Azure Dedicated Host](https://aka.ms/ADHPricing).

También puede ahorrar costos con una [instancia reservada de Azure Dedicated Host](prepay-dedicated-hosts-reserved-instances.md).

## <a name="sizes-and-hardware-generations"></a>Tamaños y generaciones de hardware

Se define una SKU para un host y representa el tipo y la serie de tamaño de máquina virtual. Puede mezclar varias máquinas virtuales de distintos tamaños en un solo host, siempre que no sean de la misma serie de tamaño.

El *tipo* es la generación de hardware. Los distintos tipos de hardware de la misma serie de máquinas virtuales proceden de distintos proveedores de CPU y tienen diferentes generaciones de CPU y número de núcleos.

Los tamaños y tipos de hardware varían según la región. Para más información, consulte la [página de precios](https://aka.ms/ADHPricing) de hosts.

> [!NOTE]
> Una vez que se aprovisiona un host dedicado, no se puede cambiar su tipo o tamaño. Si necesita otro tamaño o tipo, tendrá que crear uno nuevo.

## <a name="host-life-cycle"></a>Ciclo de vida del host


Azure supervisa y administra el estado de mantenimiento de los hosts. Al consultar el host se devolverán los siguientes estados:

| Estado de mantenimiento   | Descripción       |
|----------|----------------|
| Host disponible     | No hay ningún problema conocido en el host.   |
| Host bajo investigación  | Hay algunos problemas en el host y se están examinando. Se trata de un estado transitorio necesario para que Azure intente identificar el ámbito y la causa principal del problema identificado. Es posible que se vean afectadas las máquinas virtuales que se ejecutan en el host. |
| Host pendiente de desasignación   | Azure no puede devolver el host a un estado correcto y le solicita que vuelva a implementar las máquinas virtuales fuera de este host. Si `autoReplaceOnFailure` está habilitado, las máquinas virtuales *se recuperan* con un hardware que funcione correctamente. De lo contrario, es posible que la máquina virtual se esté ejecutando en un host que esté a punto de producir un error.|
| Host desasignado  | Todas las máquinas virtuales se han quitado del host. Ya no se le cobrará por este host, ya que el hardware se eliminado de la rotación.   |


## <a name="next-steps"></a>Pasos siguientes

- Puede implementar un host dedicado mediante [Azure PowerShell](./windows/dedicated-hosts-powershell.md), el [portal](./dedicated-hosts-portal.md) y la [CLI de Azure](./linux/dedicated-hosts-cli.md).

- Hay una [plantilla de ejemplo](https://github.com/Azure/azure-quickstart-templates/blob/master/quickstarts/microsoft.compute/vm-dedicated-hosts/README.md) en la que se usan zonas y dominios de error para obtener la máxima resistencia en una región.

- También puede ahorrar costos con una [instancia reservada de Azure Dedicated Host](prepay-dedicated-hosts-reserved-instances.md).
