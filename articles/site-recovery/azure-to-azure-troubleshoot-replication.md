---
title: Solución de problemas de replicación para máquinas virtuales de Azure con Azure Site Recovery
description: Solución de problemas de replicación en recuperación ante desastres de VM de Azure con Azure Site Recovery
author: sideeksh
manager: rochakm
ms.topic: troubleshooting
ms.date: 04/03/2020
ms.openlocfilehash: e7f46a73b16a2af7085aa7d6312fa62f5bfd9ef8
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111960929"
---
# <a name="troubleshoot-replication-in-azure-vm-disaster-recovery"></a>Solución de problemas de replicación en recuperación ante desastres de VM de Azure

En este artículo se describen problemas comunes en Azure Site Recovery cuando se replican y recuperan máquinas virtuales (VM) de Azure de una región a otra. También se explica cómo solucionar los problemas habituales. Para más información sobre las configuraciones admitidas, consulte la [matriz de compatibilidad para la replicación de máquinas virtuales de Azure](./azure-to-azure-support-matrix.md).

Azure Site Recovery replica los datos de manera coherente de la región de origen a la región de recuperación ante desastres. También crea un punto de recuperación coherente con los bloqueos cada 5 minutos. Si Site Recovery no se puede crear puntos de recuperación durante 60 minutos, le avisa con esta información:

```plaintext
Error message: "No crash consistent recovery point available for the VM in the last 60 minutes."

Error ID: 153007
```

En las secciones siguientes se describen las causas y las soluciones.

## <a name="high-data-change-rate-on-the-source-virtual-machine"></a>Velocidad de cambio elevada en los datos de la máquina virtual de origen

Azure Site Recovery crea un evento si la frecuencia de cambio de datos en la máquina virtual de origen supera los límites admitidos. Para comprobar si el problema se debe una elevada renovación, vaya a **Elementos replicados** > **Máquina virtual** > **Events -last 72 hours** (Eventos: últimas 72 horas).
Debería ver el evento **Frecuencia de cambio de datos superior a los límites admitidos**:

:::image type="content" source="./media/site-recovery-azure-to-azure-troubleshoot/data_change_event.png" alt-text="Página de Azure Site Recovery que muestra una frecuencia de cambio de datos que es demasiado alta.":::

Si selecciona el evento, debería ver la información exacta del disco:

:::image type="content" source="./media/site-recovery-azure-to-azure-troubleshoot/data_change_event2.png" alt-text="Página que muestra los detalles del evento de la frecuencia de cambio de datos.":::

### <a name="azure-site-recovery-limits"></a>Límites de Azure Site Recovery

En la tabla siguiente se proporcionan los límites de Azure Site Recovery. Estos límites se basan en nuestras pruebas, pero no pueden cubrir todas las combinaciones de E/S posibles de la aplicación. Los resultados reales pueden variar en función de la combinación de E/S de la aplicación.

Hay dos límites que se deben tener en cuenta: la frecuencia de modificación de datos por disco y la frecuencia de modificación de datos por máquina virtual. Por ejemplo, veamos el disco Premium P20 en la tabla siguiente. En el caso de una sola máquina virtual, Site Recovery puede controlar 5 MB/s de renovación por disco con un máximo de cinco discos de este tipo. Site Recovery tiene un límite de 54 MB/s de renovación total por máquina virtual.

**Destino de almacenamiento de la replicación** | **Tamaño medio de E/S de disco de origen** |**Actividad de datos media de disco de origen** | **Actividad de datos total por día de disco de datos de origen**
---|---|---|---
Standard Storage | 8 KB    | 2 MB/s | 168 GB por disco
Disco Premium P10 o P15 | 8 KB    | 2 MB/s | 168 GB por disco
Disco Premium P10 o P15 | 16 KB | 4 MB/s |    336 GB por disco
Disco Premium P10 o P15 | 32 KB, o más | 8 MB/s | 672 GB por disco
Disco Premium P20, P30, P40 o P50 | 8 KB    | 5 MB/s | 421 GB por disco
Disco Premium P20, P30, P40 o P50 | 16 KB, o más |20 MB/s | 1684 GB por disco

### <a name="solution"></a>Solución

Azure Site Recovery tiene límites en la frecuencia de cambio de datos según el tipo de disco. Para ver si este problema es periódico o temporal, busque la frecuencia de cambio en la máquina virtual afectada. Vaya a la máquina virtual de origen. busque las métricas en **Supervisión** y agregue las métricas como se muestra en esta captura de pantalla:

:::image type="content" source="./media/site-recovery-azure-to-azure-troubleshoot/churn.png" alt-text="Página que muestra el proceso de tres pasos para buscar la frecuencia de cambio de datos.":::

1. Seleccione **Agregar métrica** y agregue **Bytes de escritura de discos del SO/s** y **Bytes de escritura de discos de datos/s**.
1. Supervise el aumento como se muestra en la captura de pantalla.
1. Vea el total de todas las operaciones de escritura que se producen en el disco del sistema operativo y en todos los discos de datos. Estas métricas podrían no proporcionar información en el nivel de disco, pero indican el patrón de la actividad de datos.

Un pico en la frecuencia de cambio de los datos puede proceder de una ráfaga de datos ocasional. Si la frecuencia de cambio de datos supera los 10 MB/s (para Premium) o 2 MB/s (para Estándar) y desciende, la replicación mantendrá el ritmo. Sin embargo, si la renovación supera con creces y de manera constante el límite admitido, considere una de las siguientes opciones:

- Excluya el disco que provoca la alta frecuencia de cambio en los datos: En primer lugar, deshabilite la replicación. A continuación, puede excluir el disco mediante el uso de [PowerShell](azure-to-azure-exclude-disks.md).
- Cambie el nivel del disco de almacenamiento de la recuperación ante desastres: Esta opción solo es posible si la renovación de datos de disco tiene una velocidad inferior a 20 MB/s. Por ejemplo, una VM con un disco P10 tiene una frecuencia de modificación de datos superior a 8 MB/s pero inferior a 10 MB/s. Si el cliente puede usar un disco P30 de almacenamiento de destino durante la protección, el problema se puede resolver. Esta solución solo es posible para las máquinas que usan Managed Disks Premium. Siga estos pasos:

  1. Vaya a **Discos** de la máquina replicada afectada y copie el nombre del disco de réplica.
  1. Vaya a esta réplica del disco administrado.
  1. Es posible que vea un mensaje emergente en **Información general** que indica que se ha generado una dirección URL de SAS. Seleccione este mensaje emergente y cancele la exportación. Omita este paso si no aparece el mensaje emergente.
  1. En cuanto se revoque la dirección URL de SAS, vaya a **Configuración** en el disco administrado. Aumente el tamaño para que Site Recovery admita la frecuencia de renovación observada en el disco de origen.

## <a name="network-connectivity-problems"></a>Problemas de conectividad de red

### <a name="network-latency-to-a-cache-storage-account"></a>Latencia de red en la cuenta de almacenamiento en caché

Site Recovery envía los datos replicados a la cuenta de almacenamiento en caché. Es posible que experimente latencia de red si la carga de los datos de una máquina virtual a la cuenta de almacenamiento en caché es inferior a 4 MB en 3 segundos.

Para comprobar si hay un problema relacionado con la latencia, use [AzCopy](../storage/common/storage-use-azcopy-v10.md). Puede usar esta utilidad de línea de comandos para cargar datos desde la máquina virtual a la cuenta de almacenamiento en caché. Si la latencia es alta, compruebe si está utilizando alguna aplicación virtual de red (NVA) para controlar el tráfico que sale de las máquinas virtuales. Es posible que se genere un cuello de botella si todo el tráfico de replicación pasa por la aplicación virtual de red.

Se recomienda crear un punto de conexión de servicio de red en la red virtual de "Storage" para que el tráfico de replicación no se dirija a la NVA. Para obtener más información, consulte [Configuración de la aplicación virtual de red](azure-to-azure-about-networking.md#network-virtual-appliance-configuration).

### <a name="network-connectivity"></a>Conectividad de red

Para que la replicación de Site Recovery funcione, es necesario que la máquina virtual proporcione conectividad saliente a direcciones URL o intervalos IP específicos. Puede que la máquina virtual esté detrás de un firewall o que use reglas de grupo de seguridad de red (NSG) para controlar la conectividad saliente. Si es así, puede experimentar problemas. Consulte [Conectividad de salida para las direcciones URL](azure-to-azure-about-networking.md#outbound-connectivity-for-urls) para comprobar que todas las direcciones URL están conectadas.

## <a name="error-id-153006---no-app-consistent-recovery-point-available-for-the-vm-in-the-past-x-minutes"></a>Id. del error: 153006. No hay ningún punto de recuperación coherente con la aplicación disponible para la máquina virtual en los "X" últimos minutos

A continuación se muestran algunos de los problemas más comunes.

### <a name="known-issue-in-sql-server-20082008-r2"></a>Incidencia conocida en SQL Server 2008/2008 R2

**Solución:** Existe una incidencia conocida en SQL Server 2008/2008 R2. Consulte el artículo [Copia de seguridad de ASR Agent u otros VSS no componente produce un error en un servidor que aloja SQL Server 2008 R2](https://support.microsoft.com/help/4504103/non-component-vss-backup-fails-for-server-hosting-sql-server-2008-r2).

### <a name="azure-site-recovery-jobs-fail-on-servers-hosting-any-version-of-sql-server-instances-with-auto_close-dbs"></a>Se producen errores en los trabajos de Azure Site Recovery en los servidores que alojan cualquier versión de las instancias de SQL Server con bases de datos AUTO_CLOSE

**Solución:** Consulte el artículo [Las copias de seguridad de VSS que no son componentes, como los trabajos de Azure Site Recovery, producen errores en los servidores que hospedan instancias de SQL Server con bases de datos AUTO_CLOSE](https://support.microsoft.com/help/4504104/non-component-vss-backups-such-as-azure-site-recovery-jobs-fail-on-ser).

### <a name="known-issue-in-sql-server-2016-and-2017"></a>Incidencia conocida en SQL Server 2016 y 2017

**Solución:** consulte el artículo [Actualización acumulativa 16 de SQL Server 2017](https://support.microsoft.com/help/4508218/cumulative-update-16-for-sql-server-2017).

### <a name="youre-using-azure-storage-spaces-direct-configuration"></a>Está usando la configuración de Espacios de almacenamiento directo de Azure

**Solución:** Azure Site Recovery no puede crear un punto de recuperación coherente con la aplicación para la configuración de Espacios de almacenamiento directo. [Configure la directiva de replicación](azure-to-azure-how-to-enable-replication-s2d-vms.md).

### <a name="app-consistency-not-enabled-on-linux-servers"></a>La coherencia de aplicaciones no está habilitada en servidores Linux

**Solución:** Azure Site Recovery para el sistema operativo Linux admite scripts personalizados de aplicaciones para la coherencia de aplicaciones. El agente de movilidad de Azure Site Recovery usará el script personalizado con opciones previas y posteriores para la coherencia de aplicaciones. [Aquí](/azure/site-recovery/site-recovery-faq.yml#replication) se indican los pasos necesarios para habilitarlo.

### <a name="more-causes-because-of-vss-related-issues"></a>Más causas debidas a problemas relacionados con VSS:

Para seguir solucionando el problema, compruebe los archivos en la máquina de origen para obtener el código de error exacto:

`C:\Program Files (x86)\Microsoft Azure Site Recovery\agent\Application Data\ApplicationPolicyLogs\vacp.log`

Para encontrar los errores, abra el archivo _vacp.log_ en un editor de texto y busque la cadena **vacpError**.

```plaintext
Ex: vacpError:220#Following disks are in FilteringStopped state [\\.\PHYSICALDRIVE1=5, ]#220|^|224#FAILED: CheckWriterStatus().#2147754994|^|226#FAILED to revoke tags.FAILED: CheckWriterStatus().#2147754994|^|
```

En el ejemplo anterior, **2147754994** es el código de error que le informa sobre el error después de esta frase.

#### <a name="vss-writer-is-not-installed---error-2147221164"></a>VSS Writer no está instalado - Error 2147221164

**Solución:** Para generar la etiqueta de coherencia de la aplicación, Azure Site Recovery usa el Servicio de instantáneas de volumen (VSS). Site Recovery instala un proveedor de VSS para su funcionamiento y para realizar instantáneas de la coherencia de las aplicaciones. Azure Site Recovery instala este proveedor VSS como servicio. Si el proveedor de VSS no está instalado, se produce un error en la creación de instantáneas de la coherencia de aplicaciones. Aparece el **identificador de error 0x80040154 Clase no registrada**. Consulte el artículo para [solucionar problemas con la instalación de VSS Writer](vmware-azure-troubleshoot-push-install.md#vss-installation-failures).

#### <a name="vss-writer-is-disabled---error-2147943458"></a>VSS Writer está deshabilitado - Error 2147943458

**Solución:** Para generar la etiqueta de coherencia de la aplicación, Azure Site Recovery usa VSS. Site Recovery instala un proveedor de VSS para su funcionamiento y para realizar instantáneas de la coherencia de las aplicaciones. Este proveedor de VSS se instala como un servicio. Si el proveedor de VSS no está habilitado, se produce un error en la creación de instantáneas de la coherencia de aplicaciones. Muestra el error: **El servicio especificado está desactivado y no se puede iniciar (0x80070422)** .

Si VSS está deshabilitado:

- Compruebe que el tipo de inicio del servicio de proveedor de VSS está establecido en **Automático**.
- Reinicie los servicios siguientes:
  - Servicio VSS.
  - Proveedor VSS de Azure Site Recovery.
  - Servicio VDS.

#### <a name="vss-provider-not_registered---error-2147754756"></a>VSS PROVIDER NOT_REGISTERED - Error 2147754756

**Solución:** Para generar la etiqueta de coherencia de la aplicación, Azure Site Recovery usa el Servicio de instantáneas de volumen. Compruebe si el servicio de proveedor de VSS de Azure Site Recovery está instalado.

Use los comandos siguientes para volver a instalar el proveedor de VSS:

1. Desinstale el proveedor existente:

   `"C:\Program Files (x86)\Microsoft Azure Site Recovery\agent\InMageVSSProvider_Uninstall.cmd"`

1. Vuelva a instalar el proveedor de VSS:

   `"C:\Program Files (x86)\Microsoft Azure Site Recovery\agent\InMageVSSProvider_Install.cmd"`

Compruebe que el tipo de inicio del servicio de proveedor de VSS está establecido en **Automático**.

Reinicie los servicios siguientes:

- Servicio VSS.
- Proveedor VSS de Azure Site Recovery.
- Servicio VDS.