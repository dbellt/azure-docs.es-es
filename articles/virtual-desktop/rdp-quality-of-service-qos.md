---
title: Implementación de calidad de servicio (QoS) para Azure Virtual Desktop (versión preliminar)
titleSuffix: Azure
description: Configuración de QoS (versión preliminar) para Azure Virtual Desktop.
author: gundarev
ms.topic: conceptual
ms.date: 11/16/2020
ms.author: denisgun
ms.openlocfilehash: c90811009a38db0874589dc828059277b9ae285c
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111753043"
---
# <a name="implement-quality-of-service-qos-for-azure-virtual-desktop-preview"></a>Implementación de calidad de servicio (QoS) para Azure Virtual Desktop (versión preliminar)

> [!IMPORTANT]
> La compatibilidad con la directiva de calidad de servicio (QoS) para Azure Virtual Desktop se encuentra actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

[RDP Shortpath](./shortpath.md) proporciona un transporte directo basado en UDP entre el cliente de Escritorio remoto y el host de la sesión. RDP Shortpath permite la configuración de las directivas de calidad de servicio (QoS) para los datos del RDP.
QoS en Azure Virtual Desktop permite que el tráfico del RDP en tiempo real que es sensible a los retrasos en la red "se cuele" en el tráfico menos sensible. Un ejemplo de este tipo de tráfico menos sensible sería la descarga de una nueva aplicación ya que, en una descarga, un segundo más o menos no es un gran problema. QoS usa objetos de la directiva de grupo de Windows para identificar y marcar todos los paquetes de las secuencias en tiempo real, y ayuda a la red a proporcionar al tráfico RDP una parte dedicada del ancho de banda.

Si proporciona soporte técnico a un grupo grande de usuarios que sufren cualquiera de los problemas que se describen en este artículo, es probable que tenga que implementar QoS. Es posible que una empresa pequeña con pocos usuarios no necesite QoS, pero debería ser útil incluso en ese caso.

Sin algún tipo de QoS, podrían surgir los siguientes problemas:

* Inestabilidad: paquetes del RDP que llegan a velocidades diferentes, lo que puede generar problemas de audio y visuales.
* Pérdida de paquetes: paquetes descartados, lo que produce una retransmisión que requiere más tiempo.
* Tiempo de ida y vuelta (RTT) con retraso: paquetes del RDP que tardan mucho tiempo en llegar a sus destinos, lo que provoca retrasos perceptibles entre la entrada y la reacción de la aplicación remota.

La forma menos complicada de solucionar estos problemas es aumentar el tamaño de las conexiones de datos, tanto internas como de salida por Internet. Dado que a menudo el costo de esto es prohibitivo, QoS proporciona una manera de administrar los recursos de los que dispone, en lugar de agregar ancho de banda de forma más eficaz. Para solucionar los problemas de calidad, se recomienda usar QoS en primer lugar y agregar ancho de banda solo cuando sea necesario.

Para que QoS sea eficaz, es preciso aplicar una configuración de QoS coherente en toda la organización. Cualquier parte de la ruta de acceso que no admita las prioridades de QoS puede degradar la calidad de la sesión de RDP.

## <a name="introduction-to-qos-queues"></a>Introducción a las colas de QoS

Para proporcionar QoS, los dispositivos de red deben tener una manera de clasificar el tráfico y deben poder distinguir RDP de otro tráfico de red.

Cuando el tráfico de red entra en un enrutador, el tráfico se coloca en una cola. Si no se configura una directiva de QoS, hay solo una cola y todos los datos se tratan con el método "el primero en llegar es el primero en salir" con la misma prioridad. Esto significa que el tráfico del RDP puede quedar atascado detrás de tráfico en el que unos pocos milisegundos adicionales no serían un problema.

Al implementar QoS, se definen varias colas y para ello se usa una de las distintas características de administración de la congestión, como la puesta en cola de prioridades y la [espera equitativa ponderada basada en clases (CBWFQ)](https://www.cisco.com/en/US/docs/ios/12_0t/12_0t5/feature/guide/cbwfq.html#wp17641) de Cisco, y las características de prevención de congestiones, como la [detección anticipada aleatoria ponderada (WRED)](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/qos_conavd/configuration/15-mt/qos-conavd-15-mt-book/qos-conavd-cfg-wred.html).

Una analogía sencilla es que QoS crea "carriles para vehículos compartidos" virtuales en la red de datos. Por eso, algunos tipos de datos nunca, o casi nunca, sufren ningún retraso. Una vez que se crean esos carriles, se puede ajustar su tamaño relativo y administrar de forma mucho más eficaz el ancho de banda de la conexión, y todo ello sin dejar de proporcionar experiencias de nivel empresarial a los usuarios de una organización.

## <a name="qos-implementation-checklist"></a>Lista de comprobación de la implementación de QoS

A nivel general, siga estos pasos para implementar QoS:

1. [Asegúrese de que la red está preparada](#make-sure-your-network-is-ready).
2. [Asegúrese de que RDP Shortpath está habilitado](./shortpath.md) (las directivas de QoS no se admiten para el transporte de conexión inversa).
3. [Implemente la inserción de marcadores de DSCP](#insert-dscp-markers) en los hosts de sesión.

Cuando se prepare para la implementación de QoS, tenga en cuenta las siguientes directrices:

* La ruta de acceso más corta al host de sesión es la mejor.
* No se recomienda que haya ningún obstáculo, como servidores proxy o dispositivos de inspección de paquetes.

## <a name="make-sure-your-network-is-ready"></a>Asegúrese de que la red está preparada

Si se plantea la posibilidad de realizar una implementación de QoS, ya debe haber determinado los requisitos de ancho de banda y los demás [requisitos de red](/windows-server/remote/remote-desktop-services/network-guidance?context=/azure/virtual-desktop/context/context).
  
La congestión del tráfico en una red afectará significativamente a la calidad de los medios. La falta de ancho de banda conduce a la degradación del rendimiento y a una mala experiencia del usuario. A medida que crece el uso y la adopción de Azure Virtual Desktop, use [Log Analytics](./diagnostics-log-analytics.md) para identificar los posibles problemas y, después, realice los ajustes necesarios mediante QoS y las adiciones de ancho de banda selectivo.

### <a name="vpn-considerations"></a>Consideraciones al respecto de la red privada virtual

QoS solo funciona según lo esperado cuando se implementa en todos los vínculos entre los clientes y los hosts de la sesión. Si QoS se usa en una red interna y un usuario inicia sesión desde una ubicación remota, solo se pueden asignar prioridades dentro de la red interna administrada. Aunque las ubicaciones remotas pueden recibir una conexión administrada mediante la implementación de una red privada virtual, una VPN implica intrínsecamente una sobrecarga de paquetes y crea retrasos en el tráfico en tiempo real.

En una organización global con vínculos administrados que abarcan continentes, se recomienda encarecidamente usar QoS, ya que el ancho de banda para esos vínculos está limitado, en comparación con la LAN.

## <a name="insert-dscp-markers"></a>Inserción de marcadores de DSCP

Para implementar QoS se puede usar un objeto de directiva de grupo (GPO) para indicar a los hosts de sesión que inserten un marcador de DSCP en los encabezados de paquetes IP que lo identifiquen como un tipo determinado de tráfico. Tanto los enrutadores como otros dispositivos de red se pueden configurar para que reconozcan estas marcas y coloquen el tráfico en una cola independiente de mayor prioridad.

Las marcas de DSCP se pueden comparar con los sellos de franqueo que indican a los trabajadores de Correos el grado de urgencia de la entrega y cual es la mejor forma de ordenar los paquetes para entregarlos lo más rápidamente posible. Una vez que se haya configurado la red para dar prioridad a las secuencias de RDP, tanto los paquetes perdidos como los que van con retraso deberían disminuir considerablemente.

Una vez que todos los dispositivos de red usan las mismas clasificaciones, marcas y prioridades, es posible reducir o eliminar retrasos, paquetes descartados e inestabilidad. Desde la perspectiva de RDP, el paso de configuración esencial es la clasificación y el marcado de paquetes. Sin embargo, para que la QoS completa se realice correctamente, también es preciso alinear meticulosamente la configuración de RDP con la configuración de red subyacente.
El valor de DSCP indica a una red configurada correspondiente qué prioridad se debe proporcionar a un paquete o secuencia.

Se recomienda usar el valor 46 de DSCP que se asigna a la clase de DSCP **Expedited Forwarding (EF)** .

### <a name="implement-qos-on-session-host-using-group-policy"></a>Implementación de QoS en un host de sesión mediante una directiva de grupo

Puede usar la calidad de servicio (QoS) basada en directivas en la directiva de grupo para establecer el valor de DSCP predefinido.

Para crear una directiva de QoS para hosts de sesión unidos a un dominio, lo primero que debe hacer es iniciar sesión en un equipo en el que se haya instalado la Administración de directivas de grupo. Abra la Administración de directivas de grupo (seleccione Inicio, Herramientas administrativas y seleccione Administración de directivas de grupo) y, después, siga estos pasos:

1. En Administración de directivas de grupo, busque el contenedor en el que se debe crear la nueva directiva. Por ejemplo, si todos los equipos host de sesión se encuentran en una unidad organizativa denominada **"hosts de sesión"** , la nueva directiva se debe crear en la unidad organizativa Hosts de sesión.

2. Haga clic con el botón derecho en el contenedor adecuado y, después, seleccione **Crear un GPO en este dominio y vincularlo aquí**.

3. En el cuadro de diálogo **Nuevo GPO**, escriba el nombre del nuevo objeto de directiva del grupo en el cuadro **Nombre** y seleccione **Aceptar**.

4. Haga clic con el botón derecho en la directiva recién creada y seleccione **Editar**.

5. En el Editor de administración de directivas de grupo, expanda **Configuración del equipo** y **Configuración de Windows**, haga clic con el botón derecho en **QoS basada en directiva** y, después, seleccione **Crear nueva directiva**.

6. En el cuadro de diálogo **QoS basada en directiva**, en la página de apertura, escriba el nombre de la nueva directiva en el cuadro **Nombre**. Seleccione **Specify DSCP Value** (Especificar valor de DSCP) y establezca el valor en **46**. Deje **Specify Outbound Throttle Rate** (Especificar velocidad de regulador de carga de trabajo de salida) sin seleccionar y, después, seleccione **Next** (Siguiente).

7. En la página siguiente, seleccione **Only applications with this executable name** (Solo aplicaciones con su nombre ejecutable), escriba el nombre **svchost.exe** y, después, seleccione **Next** (Siguiente). Esta configuración indica a la directiva que solo dé prioridad al tráfico coincidente del servicio de Escritorio remoto.

8. En la tercera página, asegúrese de que tanto **Any source IP address** (Cualquier dirección IP de origen) como **Any destination IP address** (Cualquier dirección IP de destino) estén seleccionadas y, después, seleccione **Next** (Siguiente). Estos dos valores garantizan que los paquetes se administrarán independientemente de cuál sea el equipo (dirección IP) que envíe los paquetes y cuál (dirección IP) el que los va a recibir.

9. En la página cuatro, seleccione **UDP** en la lista desplegable **Select the protocol this QoS policy applies to** (Seleccionar el protocolo al que se aplica esta directiva de QoS).

10. En el título **Specify the source port number** (Especificar el número de puerto de origen), seleccione **From this source port or range** (Desde este intervalo o número de puerto de origen). En el cuadro de texto que lo acompaña, escriba **3390**. Seleccione **Finalizar**.

Las nuevas directivas que ha creado no surtirán efecto hasta que la directiva de grupo se haya actualizado en los equipos host de sesión. Aunque la directiva de grupo se actualiza sola periódicamente, siga estos pasos para forzar una actualización inmediata:

1. En cada host de sesión para el que desee actualizar directiva de grupo, abra un símbolo del sistema como administrador (*Ejecutar como administrador*).

1. En el símbolo del sistema, escriba:

   ```console
   gpupdate /force
   ```

### <a name="implement-qos-on-session-host-using-powershell"></a>Implementación de QoS en un host de sesión mediante PowerShell

Puede establecer QoS para RDP Shortpath mediante el siguiente cmdlet de PowerShell:

```powershell
New-NetQosPolicy -Name "RDP Shortpath" -AppPathNameMatchCondition "svchost.exe" -IPProtocolMatchCondition UDP -IPSrcPortStartMatchCondition 3390 -IPSrcPortEndMatchCondition 3390 -DSCPAction 46 -NetworkProfile All
```

## <a name="related-articles"></a>Artículos relacionados

* [Directiva de calidad de servicio (QoS)](/windows-server/networking/technologies/qos/qos-policy-top)

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre los requisitos de ancho de banda de Azure Virtual Desktop, consulte la [descripción de los requisitos del protocolo de Escritorio remoto (RDP) para Azure Virtual Desktop](rdp-bandwidth.md).
* Para más información sobre la conectividad de red de Azure Virtual Desktop, consulte la [descripción de la conectividad de red de Azure Virtual Desktop](network-connectivity.md).
