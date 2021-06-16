---
title: Opciones para servidores de Oracle BareMetal Infrastructure
description: Obtenga información sobre las opciones y consideraciones para los servidores de Oracle BareMetal Infrastructure.
ms.topic: reference
ms.subservice: baremetal-oracle
ms.date: 04/15/2021
ms.openlocfilehash: eb77731ad018817cf6d868cefd6a4d84b8f6d330
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110578660"
---
# <a name="options-for-oracle-baremetal-infrastructure-servers"></a>Opciones para servidores de Oracle BareMetal Infrastructure

En este artículo, consideraremos opciones y recomendaciones para obtener el máximo nivel de protección y rendimiento al ejecutar Oracle en servidores BareMetal Infrastructure. 

## <a name="data-guard-protection-modes"></a>Modos de protección de Data Guard

Data Guard ofrece protección que no disponible únicamente a través de Oracle Real Applications Cluster (RAC), replicación lógica (como GoldenGate) y replicación basada en almacenamiento. 

| Modo de protección | Descripción |
| --- | --- |
| **Máximo rendimiento** | El nivel de protección predeterminado. Proporciona el mayor nivel de protección sin afectar al rendimiento de la base de datos principal. Los datos se consideran confirmados en cuanto se han escrito en el flujo de fase de puesta al día de la base de datos principal. A continuación, se replican en la base de datos en espera de forma asincrónica. Por lo general, la base de datos en espera los recibe en cuestión de segundos, pero no se ofrece ninguna garantía a tales efectos. En general, este modo cumple los requisitos empresariales (junto con la supervisión de retrasos) sin necesidad de conectividad de red de baja latencia entre los sitios primario y en espera.<br /><br />Ofrece la mejor persistencia operativa; sin embargo, no garantiza que no haya pérdida de datos.   |
| **Máxima disponibilidad** | Proporciona el mayor nivel de protección sin afectar a la disponibilidad de la base de datos principal. Los datos nunca se consideran confirmados en la base de datos principal hasta que también se hayan confirmado en al menos una base de datos en espera. Si la base de datos principal no puede escribir los cambios de la fase de puesta al día en al menos una base de datos en espera, vuelve al modo de máximo rendimiento en lugar de dejar de estar disponible. <br /><br />Permite que el servicio continúe si el sitio en espera no está disponible. Si solo funciona un sitio, solo se mantendrá una copia de los datos hasta que el segundo sitio esté en línea y se vuelva a establecer la sincronización. |
| **Máxima protección** | Proporciona un nivel de protección similar a la máxima disponibilidad. La base de datos principal se apaga con la característica agregada si no se pueden escribir los cambios de la fase de puesta al día en al menos una base de datos en espera. Esto garantiza que no se pueda producir una pérdida de datos, pero a costa de una disponibilidad más frágil. |

>[!IMPORTANT]
>Si necesita un objetivo de punto de recuperación (RPO) de cero, se recomienda la configuración de máxima disponibilidad. A continuación, se puede garantizar el RPO incluso cuando se produzcan varios errores. Por ejemplo, incluso en el caso de una interrupción de la red de la base de datos principal, seguida de la pérdida de la base de datos principal en algún momento posterior mientras la interrupción de la red sigue en vigor.

### <a name="data-guard-deployment-patterns"></a>Patrones de implementación de Data Guard

Oracle permite configurar varios destinos para la generación de la fase de puesta al día, lo que permite varias bases de datos en espera. La configuración más común se muestra en la ilustración siguiente, una única base de datos en espera en una región diferente.

:::image type="content" source="media/oracle-high-availability/default-data-guard-deployment.png" alt-text="Diagrama que muestra la implementación predeterminada de Data Guard de Oracle.":::

Data Guard se configura en modo de máximo rendimiento para una implementación predeterminada. Esta configuración ofrece replicación de datos casi en tiempo real a través del transporte asincrónico de la fase de puesta al día. No es necesario que la base de datos en espera se ejecute dentro de una implementación de RAC, pero se recomienda que cumpla las exigencias de rendimiento del sitio primario.

Se recomienda una implementación como la que se muestra en la ilustración siguiente para entornos que requieren un tiempo de actividad estricto o un RPO de cero. La configuración de máxima disponibilidad consta de una base de datos en espera local que aplica la fase de puesta al día en modo sincrónico y una segunda base de datos en espera que se ejecuta en una región remota.

:::image type="content" source="media/oracle-high-availability/max-availability-data-guard-deployment.png" alt-text="Diagrama que muestra la implementación de Data Guard de máxima disponibilidad.":::

Puede crear una base de datos en espera local cuando el rendimiento de la aplicación se vaya a ver afectado por ejecutar la base de datos y los servidores de aplicaciones en regiones independientes. En esta configuración, se usa una base de datos en espera local cuando se necesita un mantenimiento planeado o no planeado en el clúster principal. Puede ejecutar estas bases de datos con replicación sincrónica, ya que están en la misma región, lo que garantiza que no se pierdan datos entre ellas.

### <a name="data-guard-configuration-considerations"></a>Consideraciones sobre la configuración de Data Guard

Debe implementarse Data Guard Broker, ya que simplifica la implementación de una configuración de Data Guard y garantiza el cumplimiento de los procedimientos recomendados. Proporciona la funcionalidad de supervisión del rendimiento y simplifica enormemente los procedimientos de intercambio, conmutación por error y recreación de instancias.

Data Guard permite ejecutar un proceso de observador, que supervisa todas las bases de datos de una configuración de Data Guard para determinar la disponibilidad de las bases de datos. Si se produce un error en una base de datos principal, el observador de Data Guard puede iniciar automáticamente una conmutación por error a una base de datos en espera en la configuración. Puede implementar el observador de Data Guard con varios observadores en función del número de sitios físicos (hasta tres). 

Este observador debe estar ubicado en la infraestructura que admitirá la capa de aplicación. El observador principal debe existir siempre en el sitio físico donde no se encuentra la base de datos principal. Se recomienda tener cuidado al automatizar las operaciones de conmutación por error desencadenadas por un observador de Data Guard. En primer lugar, asegúrese de que las aplicaciones estén diseñadas y probadas para brindar un servicio aceptable cuando la base de datos se ejecute en una ubicación independiente.

Si la aplicación solo puede funcionar localmente, la conmutación por error al sitio secundario debe ser manual. Los entornos que requieren niveles de alta disponibilidad (tiempo de actividad del 99,99 % o 99,999 %) deben usar una base de datos en espera local y remota, como se muestra en la figura anterior. En estos casos, el parámetro FastStartFailoverTarget solo se establecerá en la base de datos en espera local.

Para todas las aplicaciones que admiten el acceso a aplicaciones o bases de datos entre sitios, FastStartFailoverTarget se establece en todas las bases de datos en espera en la configuración de Data Guard.

### <a name="active-data-guard"></a>Active Data Guard

Oracle Active Data Guard (ADG) es un superconjunto de funcionalidades básicas de Data Guard que se incluyen con Oracle Database Enterprise Edition. Proporciona las siguientes características agregadas, que se usarán en toda la implementación de Oracle Exadata:

- Características exclusivas de detección y reparación automática de daños.
- Conmutación por error rápida a la réplica sincronizada de producción: manual o automática.
- Descarga de la carga de trabajo de producción en una base de datos de espera sincronizada, de solo lectura, abierta.
- Actualizaciones graduales de base de datos y en espera. Primera aplicación de revisiones mediante base de datos física en espera.
- Descarga de copias de seguridad incrementales a base de datos en espera.
- Protección de recuperación de datos sin pérdida de datos a cualquier distancia sin afectar al rendimiento.

Las notas del producto disponibles en [https://www.oracle.com/technetwork/database/availability/dg-adg-technical-overview-wp-5347548.pdf](https://www.oracle.com/technetwork/database/availability/dg-adg-technical-overview-wp-5347548.pdf) brindan una buena visión general de las características anteriores, como se muestra en la figura siguiente.

:::image type="content" source="media/oracle-high-availability/active-data-guard-features.png" alt-text="Diagrama que muestra información general de las características de Oracle Active Data Guard.":::

## <a name="backup-recommendations"></a>Recomendaciones de copia de seguridad

Asegúrese de hacer una copia de seguridad de las bases de datos. Use las características de restauración y recuperación para restaurar una base de datos en el mismo sistema o en otro, o para recuperar archivos de base de datos.

Es importante establecer una estrategia de recuperación de copia de seguridad para proteger las bases de Oracle Database Appliance frente a la pérdida de datos. Esta pérdida podría deberse a un problema físico en un disco, que provoque un error de lectura o escritura en un archivo del disco, necesario para ejecutar la base de datos. Un error de un usuario también puede provocar la pérdida de datos. La característica de copia de seguridad ofrece la capacidad de **restauración a un momento dado (PITR) de la base de datos, la recuperación del número de cambio del sistema (SCN) y la recuperación más reciente**. Puede crear una directiva de copia de seguridad en la interfaz de usuario del explorador o desde la interfaz de la línea de comandos.

Están disponibles las siguientes opciones de copia de seguridad:

- Copia de seguridad en un volumen de almacenamiento NFS (área de recuperación rápida: FRA- /u98).
- Mediante instantánea de SnapCenter de Azure NetApp Files.

Proceso que se debe tener en cuenta:

- Copias de seguridad manuales o automáticas.
- Las copias de seguridad automáticas se escriben en volúmenes de almacenamiento NFS (por ejemplo, /u98).
- Las copias de seguridad se ejecutan entre las 12:00 a. m. y las 6:00 a. m. en la zona horaria del sistema de base de datos.
- Períodos de retención presentes: 7, 15, 30, 45 y 60 días.

- Recupere la base de datos de una copia de seguridad almacenada en el almacenamiento de objetos:
  - Hasta el último estado correcto conocido con la menor pérdida de datos posible.
  - Con la marca de tiempo especificada.
  - Con el SCN especificado.
  - BackupReport: _usa el SCN del informe de copia de seguridad en lugar del SCN especificado_.

:::image type="content" source="media/oracle-high-availability/customer-db-backup-to-fra.png" alt-text="Diagrama que muestra la copia de seguridad de la base de datos del cliente en FRA (/u98) y no FRA (/u95).":::

### <a name="backup-policy"></a>Directiva de copia de seguridad

La directiva de copia de seguridad define los detalles de la copia de seguridad. Al crear una directiva de copia de seguridad, se define el destino de la FRA de las copias de seguridad de base de datos (ubicación NFS) y se define la ventana de recuperación.

De manera predeterminada, se usa el algoritmo de compresión BASIC. Al usar algoritmos de compresión LOW, MEDIUM o HIGH para la directiva de copia de seguridad de disco o NFS, se deben tener en cuenta las licencias.

### <a name="backup-levels"></a>Niveles de copia de seguridad

Especifique el nivel de copia de seguridad al hacer una copia de seguridad.

- Nivel 0: completo
- Nivel 1: incremental
- LongTerm/Archievelog: excepto para la directiva de retención de copia de seguridad, use una ubicación que no sea FRA (por ejemplo, /u95).

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo recuperar la base de datos de Oracle cuando se produce un error:

> [!div class="nextstepaction"]
> [Recuperación de la base de datos de Oracle en la Infraestructura de Azure sin sistema operativo](oracle-high-availability-recovery.md)
