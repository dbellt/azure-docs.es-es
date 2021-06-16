---
title: Ejecución de Deployment Planner para la recuperación ante desastres de VMware con Azure Site Recovery
description: En este artículo se describe cómo ejecutar Azure Site Recovery Deployment Planner para la recuperación ante desastres de VMware en Azure.
author: Sharmistha-Rai
manager: gaggupta
ms.service: site-recovery
ms.topic: conceptual
ms.author: sharrai
ms.date: 05/27/2021
ms.openlocfilehash: cfd9dcab6fb04c877db0160b9f9e9768b2fe91f0
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110579350"
---
# <a name="run-the-deployment-planner-for-vmware-disaster-recovery"></a>Ejecución de Deployment Planner para la recuperación ante desastres de VMware
Este artículo es la guía del usuario de Azure Site Recovery Deployment Planner para implementaciones de producción de VMware en Azure.


## <a name="modes-of-running-deployment-planner"></a>Modos de ejecutar Deployment Planner
La herramienta de línea de comandos (ASRDeploymentPlanner.exe) se puede ejecutar en cualquiera de los tres modos siguientes:

1.  [Generación de perfiles](#profile-vmware-vms)
2.  [Generación de informes](#generate-report)
3.  [Obtención de rendimiento](#get-throughput)

En primer lugar, ejecute la herramienta en modo de generación de perfiles de modo para recopilar la actividad de datos y las IOPS de la máquina virtual. A continuación, ejecute la herramienta para generar el informe para encontrar los requisitos de ancho de banda de red y de almacenamiento, y el costo de la recuperación ante desastres.

## <a name="profile-vmware-vms"></a>Generación de perfiles de máquina virtual de VMware
En el modo de generación de perfiles, la herramienta Deployment Planner se conecta al servidor vCenter/host de vSphere ESXi para recopilar datos relativos al rendimiento de la máquina virtual.

* La generación de perfiles no afecta al rendimiento de las máquinas virtuales de producción, porque no se realiza ninguna conexión directa con ellas. Todos los datos de rendimiento se recopilan del servidor vCenter/host de vSphere ESXi.
* Para asegurarse de que solo haya un efecto insignificante en el servidor debido a la generación de perfiles, la herramienta consulta el servidor vCenter/host vSphere ESXi una vez cada 15 minutos. Este intervalo de consulta no afecta a la precisión de la generación de perfiles, ya que la herramienta almacena los datos de los contadores de rendimiento cada minuto.

### <a name="create-a-list-of-vms-to-profile"></a>Creación de una lista de máquinas virtuales cuyo perfil se va a generar
En primer lugar, se necesita una lista de las máquinas virtuales cuyo perfil se va a generar. Para obtener todos los nombres de las máquinas virtuales de un servidor vCenter/host de vSphere ESXi, utilice los comandos de VMware vSphere PowerCLI en el siguiente procedimiento. Como alternativa, puede enumerar en un archivo los nombres descriptivos o las direcciones IP de las máquinas virtuales cuyos perfiles desea generar de forma manual.

1. Inicie sesión en la máquina virtual en la que está instalado VMware vSphere PowerCLI.
2. Abra la consola de VMware vSphere PowerCLI.
3. Asegúrese de que la directiva de ejecución está deshabilitada en el script. Si está deshabilitada, inicie la consola de VMware vSphere PowerCLI en modo de administrador y habilítela, para lo que debe ejecutar el siguiente comando:

    ```powershell
    Set-ExecutionPolicy –ExecutionPolicy AllSigned
    ```

4. Opcionalmente, es posible que necesite ejecutar el siguiente comando si no se reconoce al servidor de Connect-VI como nombre del cmdlet.

    ```powershell
    Add-PSSnapin VMware.VimAutomation.Core
    ```

5. Para obtener todos los nombres de las máquinas virtuales en un servidor vCenter/host de vSphere ESXi y almacenar la lista en un archivo .txt, ejecute los dos comandos que se enumeran aquí.
Reemplace &lsaquo;server name&rsaquo; (nombre del servidor), &lsaquo;user name&rsaquo; (nombre de usuario), &lsaquo;password&rsaquo; (contraseña), &lsaquo;outputfile.txt&rsaquo;; (archivo de salida.txt) por sus entradas.

    ```powershell
    Connect-VIServer -Server <server name> -User <user name> -Password <password>

    Get-VM |  Select Name | Sort-Object -Property Name >  <outputfile.txt>
    ```

6. Abra el archivo de salida en el Bloc de notas y, después, copie en otro archivo los nombres de todas las máquinas virtuales cuyo perfil desea generar (por ejemplo, ProfileVMList.txt), un nombre de máquina virtual por línea. Este archivo se usa como entrada para el parámetro *-VMListFile* de la herramienta de línea de comandos.

    ![Lista de nombres de máquinas virtuales en Deployment Planner
](media/site-recovery-vmware-deployment-planner-run/profile-vm-list-v2a.png)

### <a name="start-profiling"></a>Inicio de la generación de perfiles
Una vez que tenga la lista de máquinas virtuales cuyo perfil se va a generar, puede ejecutar la herramienta en modo de generación de perfiles. Esta es la lista de parámetros obligatorios y opcionales de la herramienta para que se ejecute en modo de generación de perfiles.

```
ASRDeploymentPlanner.exe -Operation StartProfiling /?
```

| Nombre de parámetro | Descripción |
|---|---|
| -Operation | Inicio de la generación de perfiles |
| -Server | El nombre de dominio completo o la dirección IP del servidor vCenter/host de vSphere ESXi de cuyas máquinas virtuales se va a generar el perfil.|
| -User | El nombre de usuario para conectarse al servidor de vCenter/host de vSphere ESXi. El usuario debe tener, como mínimo, acceso de solo lectura.|
| -VMListFile | El archivo que contiene la lista de máquinas virtuales cuyos perfiles se van a generar. La ruta de acceso del archivo puede ser absoluta o relativa. El archivo debe contener un nombre de máquina virtual o una dirección IP en cada línea. El nombre de la máquina virtual especificado en el archivo debe ser el mismo que el nombre de la máquina virtual del servidor vCenter/host de vSphere ESXi.<br>Por ejemplo, el archivo VMList.txt contiene las siguientes máquinas virtuales:<ul><li>virtual_machine_A</li><li>10.150.29.110</li><li>virtual_machine_B</li><ul> |
|-NoOfMinutesToProfile|Número de minutos durante los cuales se va a ejecutar la generación de perfiles. El mínimo es 30 minutos.|
|-NoOfHoursToProfile|Número de horas durante las cuales se va a ejecutar la generación de perfiles.|
| -NoOfDaysToProfile | El número de días durante el que se va a ejecutar la generación de perfiles. Se recomienda ejecutar la generación de perfiles durante más de 7 días para asegurarse de que el patrón de carga de trabajo en su entorno durante el período especificado se sigue y se usa para proporcionar una recomendación adecuada. |
|-Virtualization|Especifique el tipo de virtualización (Hyper-V o VMware).|
| -Directory | (Opcional) La convención de nomenclatura universal (UNC) o ruta de acceso del directorio local para almacenar los datos de la generación de perfiles que se han producido durante esta. Si no se especifica un nombre de directorio, se usará "ProfiledData" de la ruta de acceso actual como directorio predeterminado. |
| -Password | (Opcional) La contraseña que se usa para conectarse al servidor vCenter/host de vSphere ESXi. Si no se especifica ahora, se le pedirá que lo haga cuando se ejecute el comando.|
|-Port|(Opcional) Número de puerto para conectarse al host de vCenter/ESXi. El puerto predeterminado es 443.|
|-Protocol| (Opcional) Especifica el protocolo 'http' o 'https' para conectarse a vCenter. El protocolo predeterminado es https.|
| -StorageAccountName | (Opcional) El nombre de la cuenta de almacenamiento que se usa para ver el rendimiento que se puede obtener en la replicación de datos desde una ubicación local a Azure. La herramienta carga los datos de prueba en esta cuenta de almacenamiento para calcular el rendimiento. La cuenta de almacenamiento debe ser de tipo de uso general v1 (GPv1). |
| -StorageAccountKey | (Opcional) La clave de la cuenta de almacenamiento utilizada para acceder a dicha cuenta. Vaya a Azure Portal > Cuentas de almacenamiento > <*nombre de la cuenta de almacenamiento*> > Configuración > Claves de acceso > Key1. |
| -Environment | (Opcional) Se trata del entorno de la cuenta de Azure Storage de destino. Puede ser uno de estos tres valores: AzureCloud, AzureUSGovernment y AzureChinaCloud. El valor predeterminado es AzureCloud. Use el parámetro si la región de destino de Azure se corresponde con Azure US Government o Azure China 21Vianet. |


Se recomienda generar perfiles de las máquinas virtuales durante más de 7 días. Si el patrón de actividad varía en un mes, se recomienda realizar la generación de perfiles durante la semana en que contemple la actividad máxima. La mejor manera es generar perfiles durante 31 días para obtener la mejor recomendación. Durante el período de generación de perfiles ASRDeploymentPlanner.exe sigue ejecutándose. La herramienta admite la entrada de tiempo de generación de perfiles en días. Para realizar una prueba rápida de la herramienta o una prueba de concepto, puede generar perfiles durante algunas horas o minutos. El tiempo mínimo de generación de perfiles permitido es de 30 minutos.

Durante la generación de perfiles, también se pueden pasar un nombre y una clave de cuenta de almacenamiento para ver el rendimiento que Site Recovery puede lograr en el momento de la replicación desde el servidor de configuración o de procesos a Azure. Si el nombre y la clave de la cuenta de almacenamiento no se pasan durante la generación de perfiles, la herramienta no calculará el rendimiento que se puede obtener.

Puede ejecutar varias instancias de la herramienta para varios conjuntos de máquinas virtuales. Asegúrese de que los nombres de máquina virtual no se repiten en ninguno de los conjuntos de generación de perfiles. Por ejemplo, si ha generado el perfil de diez máquinas virtuales (de VM1 a VM10) y unos días después desea generar el de otras cinco (de VM11 a VM15), puede ejecutar la herramienta desde otra consola de línea de comandos para el segundo conjunto de máquinas virtuales (de VM11 a VM15). Asegúrese de que el segundo conjunto de máquinas virtuales no tiene ningún nombre de máquina virtual de la primera instancia de generación de perfiles o utilice un directorio de salida diferente para la segunda ejecución. Si se usan dos instancias de la herramienta para generar los perfiles de las mismas máquinas virtuales y se usa el mismo directorio de salida, el informe generado no será correcto.

De forma predeterminada, la herramienta está configurada tanto para generar perfiles como para generar informes de hasta 1000 VM. Para cambiar el límite, cambie el valor de la clave MaxVMsSupported en el archivo *ASRDeploymentPlanner.exe.config*.
```
<!-- Maximum number of vms supported-->
<add key="MaxVmsSupported" value="1000"/>
```
Con la configuración predeterminada, para generar perfiles de 1500 máquinas virtuales, cree dos archivos VMList.txt. Uno con 1000 máquinas virtuales y otro con una lista de 500 máquinas virtuales. Ejecute las dos instancias de Azure Site Recovery Deployment Planner, una con VMList1.txt y la otra con VMList2.txt. Puede usar la misma ruta de acceso de directorio para almacenar los datos de los perfiles de las máquinas virtuales de ambos VMList.

Hemos visto que en función de la configuración de hardware, sobre todo el tamaño de la RAM del servidor desde el que se ejecuta la herramienta para generar el informe, la operación puede no completarse debido a que la memoria no es suficiente. Si el hardware es bueno, puede asignar cualquier valor MaxVMsSupported, por grande que sea.  

Si tiene varios servidores de vCenter, para generar los perfiles debe ejecutar una instancia de ASRDeploymentPlanner por cada uno de ellos.

Las configuraciones de las máquinas virtuales se capturan una vez al principio de la operación de generación de perfiles y se almacena en un archivo denominado VMDetailList.xml. Esta información se usa cuando se genera el informe. Los cambios en la configuración de una máquina virtual (por ejemplo, un mayor número de núcleos, discos o NIC) que se realizan desde el principio hasta el final de la generación de perfiles no se capturan. Esta es la solución alternativa para obtener la información más reciente de la máquina virtual más reciente al generar un informe en caso de que la configuración de una máquina virtual haya cambiado durante el transcurso de la generación de perfiles en la versión preliminar pública:

* Realice una copia de seguridad de VMdetailList.xml y elimine el archivo de su ubicación actual.
* Pase los argumentos -User y -Password en el momento de la generación de informes.

El comando de generación de perfiles genera varios archivos en el directorio de generación de perfiles. No elimine ninguno de los archivos, ya que la generación de informes puede resultar afectada.

#### <a name="example-1-profile-vms-for-30-days-and-find-the-throughput-from-on-premises-to-azure"></a>Ejemplo 1: Generación de perfiles de máquinas virtuales durante 30 días y búsqueda del rendimiento desde una ubicación local en Azure
```
ASRDeploymentPlanner.exe -Operation StartProfiling -Virtualization VMware -Directory “E:\vCenter1_ProfiledData” -Server vCenter1.contoso.com -VMListFile “E:\vCenter1_ProfiledData\ProfileVMList1.txt”  -NoOfDaysToProfile  30  -User vCenterUser1 -StorageAccountName  asrspfarm1 -StorageAccountKey Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==
```

#### <a name="example-2-profile-vms-for-15-days"></a>Ejemplo 2: Generación de perfiles de máquinas virtuales durante 15 días

```
ASRDeploymentPlanner.exe -Operation StartProfiling -Virtualization VMware -Directory “E:\vCenter1_ProfiledData” -Server vCenter1.contoso.com -VMListFile “E:\vCenter1_ProfiledData\ProfileVMList1.txt” -NoOfDaysToProfile  15  -User vCenterUser1
```

#### <a name="example-3-profile-vms-for-60-minutes-for-a-quick-test-of-the-tool"></a>Ejemplo 3: Generación de perfiles de máquinas virtuales durante 60 minutos para realizar una prueba rápida de la herramienta
```
ASRDeploymentPlanner.exe -Operation StartProfiling -Virtualization VMware -Directory “E:\vCenter1_ProfiledData” -Server vCenter1.contoso.com -VMListFile “E:\vCenter1_ProfiledData\ProfileVMList1.txt”  -NoOfMinutesToProfile 60  -User vCenterUser1
```

#### <a name="example-4-profile-vms-for-2-hours-for-a-proof-of-concept"></a>Ejemplo 4: Generación de perfiles de máquinas virtuales durante 2 horas para una prueba de concepto
```
ASRDeploymentPlanner.exe -Operation StartProfiling -Virtualization VMware -Directory “E:\vCenter1_ProfiledData” -Server vCenter1.contoso.com -VMListFile “E:\vCenter1_ProfiledData\ProfileVMList1.txt” -NoOfHoursToProfile 2 -User vCenterUser1
```

>[!NOTE]
>
>* Si el servidor en el que se ejecuta la herramienta se reinicia o se ha bloqueado, o si cierra la herramienta con Ctrl + C, se conservan los datos de los perfiles generados. Sin embargo, es posible que falten los últimos 15 minutos de los datos de los perfiles generados. En ese caso, vuelva a ejecutar la herramienta en modo de generación de perfiles después de que se reinicie el servidor.
>* Cuando se pasan el nombre y la clave de la cuenta de almacenamiento, la herramienta mide el rendimiento en el último paso de la generación de perfiles. Si la herramienta se cierra antes de que se complete la generación de perfiles, no se calcula el rendimiento. Para hallar el rendimiento antes de generar el informe, puede ejecutar la operación de GetThroughput desde la consola de línea de comandos. Si no lo hace, el informe generado no contendrá la información de rendimiento.


## <a name="generate-report"></a>Generación de informes
La herramienta genera un archivo de Microsoft Excel habilitado para macros (archivo XLSM) como la salida del informe, que resume todas las recomendaciones de implementación. El informe se llama `DeploymentPlannerReport_<unique numeric identifier>.xlsm` y está ubicado en el directorio especificado.

>[!NOTE]
>La generación de informes requiere un equipo con Windows o Windows Server con Excel 2013, o cualquier versión posterior. El símbolo decimal de esta máquina debe configurarse como "." para generar las estimaciones de costos. En caso de que haya configurado "," como símbolo decimal, vaya a "Cambiar los formatos de fecha, hora o número" en el Panel de control y vaya a "Configuración adicional" para cambiar el símbolo decimal a ".".

Una vez que se completa la generación de perfiles, se puede ejecutar la herramienta en modo de generación de informes. La siguiente tabla contiene una lista de los parámetros obligatorios y opcionales de la herramienta que se ejecutan en modo de generación de informes.

`ASRDeploymentPlanner.exe -Operation GenerateReport /?`

|Nombre de parámetro | Descripción |
|-|-|
| -Operation | GenerateReport |
| -Server |  El nombre de dominio completo o la dirección IP del servidor vCenter o vSphere (use el mismo nombre o dirección IP que utilizó en la generación de perfiles) en el que se encuentran las máquinas virtuales con perfiles cuyo informe va a generar. Tenga en cuenta que si usó un servidor vCenter en el momento de la generación de perfiles, no puede usar un servidor de vSphere para la generación de informes, y viceversa.|
| -VMListFile | El archivo que contiene la lista de máquinas virtuales con perfiles para las que se va a generar el informe. La ruta de acceso del archivo puede ser absoluta o relativa. El archivo debe contener un nombre de máquina virtual o una dirección IP en cada línea. Los nombres de máquina virtual que se especifican en el archivo deben ser los mismos que los del servidor vCenter/host de vSphere ESXi y coincidir con los que se usaron en la generación de perfiles.|
|-Virtualization|Especifique el tipo de virtualización (Hyper-V o VMware).|
| -Directory | (Opcional) El UNC o la ruta de acceso del directorio local en que se almacenan los datos de la generación de perfiles (los archivos que se crean en la generación de perfiles). Estos datos son necesarios para generar el informe. Si no se especifica ningún nombre, se usará el directorio 'ProfiledData'. |
| -GoalToCompleteIR | (Opcional) El número de horas en que se debe completar la replicación inicial de las máquinas virtuales cuyo perfil se va a generar. El informe generado proporciona el número de máquinas virtuales en las que se puede completar la replicación inicial en el tiempo especificado. El valor predeterminado es 72 horas. |
| -User | (Opcional) El nombre de usuario que se utiliza para conectarse al servidor vCenter o vSphere. Se utiliza para capturar la información de configuración más reciente de las máquinas virtuales, como el número de discos, de núcleos y de NIC que se usan en el informe. Si no se especifica el nombre, se usa la información de configuración recopilada al principio de la generación de perfiles. |
| -Password | (Opcional) La contraseña que se usa para conectarse al servidor vCenter/host de vSphere ESXi. Si la contraseña no se especifica como parámetro, se le pedirá más tarde, cuando se ejecute el comando. |
|-Port|(Opcional) Número de puerto para conectarse al host de vCenter/ESXi. El puerto predeterminado es 443.|
|-Protocol|(Opcional) Especifica el protocolo 'http' o 'https' para conectarse a vCenter. El protocolo predeterminado es https.|
| -DesiredRPO | (Opcional) El objetivo del punto de recuperación deseado, en minutos. El valor predeterminado es 15 minutos.|
| -Bandwidth | Ancho de banda en Mbps. El parámetro se usa para calcular el RPO que se puede lograr para el ancho de banda especificado. |
| -StartDate | (Opcional) La fecha y hora de inicio en DD-MM-AAAA:HH:MM (formato de 24 horas). *StartDate* se debe especificar junto con *EndDate*. Cuando se especifica StartDate, se genera el informe de los datos de generación de perfiles que se recopilan entre StartDate y EndDate. |
| -EndDate | (Opcional) La fecha y hora de finalización en DD-MM-AAAA:HH:MM (formato de 24 horas). *EndDate* se debe especificar junto con *StartDate*. Cuando se especifica EndDate, se genera el informe para los datos de la generación de perfiles recopilados entre StartDate y EndDate. |
| -GrowthFactor | (Opcional) El factor de crecimiento, expresado en forma de porcentaje. El valor predeterminado es 30 %. |
| -UseManagedDisks | (Opcional) UseManagedDisks - Yes/No. El valor predeterminado es Yes. El cálculo del número de máquinas virtuales que se pueden colocar en una única cuenta de almacenamiento se realiza teniendo en cuenta si se ha realizado la conmutación por error o la conmutación por error de prueba en discos administrados y no en discos sin administrar. |
|-SubscriptionId |(Opcional) El GUID de la suscripción. Tenga en cuenta que este parámetro es necesario para generar el informe de estimación de costos con el precio más reciente según su suscripción, la oferta asociada a la suscripción y la región de Azure de destino concreta en la **moneda especificada**.|
|-TargetRegion|(Opcional) La región de Azure que es el destino de la replicación. Como Azure tiene diferentes costos por región, para generar un informe con la región de Azure de destino específica, use este parámetro.<br>El valor predeterminado es WestUS2 o la región de destino usada por última vez.<br>Consulte la lista de [regiones de destino admitidas](site-recovery-vmware-deployment-planner-cost-estimation.md#supported-target-regions).|
|-OfferId|(Opcional) La oferta asociada a la suscripción especificada. El valor predeterminado es MS-AZR-0003P (pago por uso).|
|-Currency|(Opcional) La moneda en la que se muestra el costo en el informe generado. El valor predeterminado es el dólar estadounidense ($) o la moneda usada por última vez.<br>Consulte la lista de [monedas admitidas](site-recovery-vmware-deployment-planner-cost-estimation.md#supported-currencies).|

De forma predeterminada, la herramienta está configurada tanto para generar perfiles como para generar informes de hasta 1000 VM. Para cambiar el límite, cambie el valor de la clave MaxVMsSupported en el archivo *ASRDeploymentPlanner.exe.config*.
```xml
<!-- Maximum number of vms supported-->
<add key="MaxVmsSupported" value="1000"/>
```

#### <a name="example-1-generate-a-report-with-default-values-when-the-profiled-data-is-on-the-local-drive"></a>Ejemplo 1: Generación de un informe con los valores predeterminados cuando los datos de generación de perfiles están en la unidad local
```
ASRDeploymentPlanner.exe -Operation GenerateReport -Virtualization VMware -Server vCenter1.contoso.com -Directory “E:\vCenter1_ProfiledData” -VMListFile “E:\vCenter1_ProfiledData\ProfileVMList1.txt”
```

#### <a name="example-2-generate-a-report-when-the-profiled-data-is-on-a-remote-server"></a>Ejemplo 2: Generación de un informe cuando los datos de generación de perfiles están en un servidor remoto
El usuario debe tener acceso de lectura y escritura en el directorio remoto.
```
ASRDeploymentPlanner.exe -Operation GenerateReport -Virtualization VMware -Server vCenter1.contoso.com -Directory “\\PS1-W2K12R2\vCenter1_ProfiledData” -VMListFile “\\PS1-W2K12R2\vCenter1_ProfiledData\ProfileVMList1.txt”
```

#### <a name="example-3-generate-a-report-with-a-specific-bandwidth-and-goal-to-complete-ir-within-specified-time"></a>Ejemplo 3: Generación de un informe con un ancho de banda y objetivo específicos para finalizar IR en el tiempo especificado
```
ASRDeploymentPlanner.exe -Operation GenerateReport -Virtualization VMware -Server vCenter1.contoso.com -Directory “E:\vCenter1_ProfiledData” -VMListFile “E:\vCenter1_ProfiledData\ProfileVMList1.txt” -Bandwidth 100 -GoalToCompleteIR 24
```

#### <a name="example-4-generate-a-report-with-a-5-percent-growth-factor-instead-of-the-default-30-percent"></a>Ejemplo 4: Generación de un informe con un factor de crecimiento del 5 %, en lugar del valor predeterminado del 30 %
```
ASRDeploymentPlanner.exe -Operation GenerateReport -Virtualization VMware -Server vCenter1.contoso.com -Directory “E:\vCenter1_ProfiledData” -VMListFile “E:\vCenter1_ProfiledData\ProfileVMList1.txt” -GrowthFactor 5
```

#### <a name="example-5-generate-a-report-with-a-subset-of-profiled-data"></a>Ejemplo 5: Generación de un informe con un subconjunto de datos de la generación de perfiles
Por ejemplo, tiene 30 días de datos de generación de perfiles y desea generar un informe de solo 20 días.
```
ASRDeploymentPlanner.exe -Operation GenerateReport -Virtualization VMware -Server vCenter1.contoso.com -Directory “E:\vCenter1_ProfiledData” -VMListFile “E:\vCenter1_ProfiledData\ProfileVMList1.txt” -StartDate  01-10-2017:12:30 -EndDate 01-19-2017:12:30
```

#### <a name="example-6-generate-a-report-for-5-minute-rpo"></a>Ejemplo 6: Generación de un informes para un RPO de 5 minutos
```
ASRDeploymentPlanner.exe -Operation GenerateReport -Virtualization VMware -Server vCenter1.contoso.com -Directory “E:\vCenter1_ProfiledData” -VMListFile “E:\vCenter1_ProfiledData\ProfileVMList1.txt”  -DesiredRPO 5
```

#### <a name="example-7-generate-a-report-for-south-india-azure-region-with-indian-rupee-and-specific-offer-id"></a>Ejemplo 7: Generación de un informe para la región de Azure de Sur de la India con la rupia india y el identificador de oferta específico

Tenga en cuenta que el identificador de suscripción es necesario para generar el informe de costos en una moneda específica.
```
ASRDeploymentPlanner.exe -Operation GenerateReport -Virtualization VMware  -Directory “E:\vCenter1_ProfiledData” -VMListFile “E:\vCenter1_ProfiledData\ProfileVMList1.txt”  -SubscriptionID 4d19f16b-3e00-4b89-a2ba-8645edf42fe5 -OfferID MS-AZR-0148P -TargetRegion southindia -Currency INR
```

## <a name="percentile-value-used-for-the-calculation"></a>Valor del percentil usado para el cálculo
**¿Qué valor del percentil predeterminado de las métricas de rendimiento recopiladas durante la generación de perfiles utiliza la herramienta al generar informes?**

La herramienta usa los valores del percentil 95 como valor predeterminado de las IOPS de lectura/escritura, las IOPS de escritura y la actividad de datos que se recopilan en la generación de perfiles de todas las máquinas virtuales. Esta métrica garantiza que el pico del percentil 100 que las máquinas virtuales pueden ver debido a eventos temporales no se utiliza para determinar los requisitos de ancho de banda del origen y de la cuenta de almacenamiento de destino. Por ejemplo, un evento temporal podría ser un trabajo de copia de seguridad que se ejecuta una vez al día, una actividad periódica de indexación de base de datos o de generación de informes de análisis, u otros eventos similares de corta duración que se producen en un momento dado.

El uso de valores del percentil 95 ofrece una imagen real de las verdaderas características de las cargas de trabajo y ofrece el mejor rendimiento cuando dichas cargas se ejecutan en Azure. No se espera que haga falta cambiar este número. Si cambia el valor (por ejemplo, al percentil 90), puede actualizar el archivo de configuración *ASRDeploymentPlanner.exe.config* en la carpeta predeterminada y guardarlo para generar un nuevo informe de los datos de generación de perfiles existentes.
```xml
<add key="WriteIOPSPercentile" value="95" />      
<add key="ReadWriteIOPSPercentile" value="95" />      
<add key="DataChurnPercentile" value="95" />
```

## <a name="growth-factor-considerations"></a>Consideraciones acerca del factor de crecimiento
**¿Por qué hay que tener en cuenta el factor de crecimiento al planear implementaciones?**

Es fundamental tener en cuenta el crecimiento en las características de la carga de trabajo y suponer que el uso puede aumentar con el tiempo. Una vez instaurada la protección, si las características de la carga de trabajo cambian, no podrá cambiar a otra cuenta de almacenamiento sin tener que deshabilitar y volver a habilitar la protección.

Por ejemplo, supongamos que en la actualidad la máquina virtual se ajusta a una cuenta de replicación de almacenamiento estándar. En los tres próximos meses, es probable que se produzcan varios cambios:

* Aumentará el número de usuarios de la aplicación que se ejecuta en la máquina virtual.
* El aumento de actividad resultante en la máquina virtual requerirá que la máquina virtual use Premium Storage para que la replicación de Site Recovery pueda mantener el ritmo.
* En consecuencia, tendrá que deshabilitar y volver a habilitar la protección en una cuenta de Premium Storage.

Se recomienda encarecidamente que planee el crecimiento durante el planeamiento de la implementación y mientras el valor predeterminado sea el 30 por ciento. El usuario es quien mejor conoce el patrón de uso de sus aplicaciones y las proyecciones de crecimiento, y este número se puede cambiar según sea necesario durante la generación de un informe. Además, puede generar varios informes con distintos factores de crecimiento con los mismos datos de generación de perfiles y determinar qué recomendaciones de ancho de banda de origen y de almacenamiento de destino son las más apropiadas en su caso.

El informe de Microsoft Excel generado contiene la siguiente información:

* [Resumen local](site-recovery-vmware-deployment-planner-analyze-report.md#on-premises-summary)
* [Recomendaciones](site-recovery-vmware-deployment-planner-analyze-report.md#recommendations)
* [VM&lt;-&gt;Selección de ubicación de almacenamiento](site-recovery-vmware-deployment-planner-analyze-report.md#vm-storage-placement)
* [VM compatibles](site-recovery-vmware-deployment-planner-analyze-report.md#compatible-vms)
* [VM incompatibles](site-recovery-vmware-deployment-planner-analyze-report.md#incompatible-vms)
* [Estimación de costos](site-recovery-vmware-deployment-planner-cost-estimation.md)

![Deployment Planner](media/site-recovery-vmware-deployment-planner-analyze-report/Recommendations-v2a.png)

## <a name="get-throughput"></a>Obtención de rendimiento

Para calcular el rendimiento que Site Recovery puede lograr durante la replicación desde una ubicación local a Azure, ejecute la herramienta en modo GetThroughput. La herramienta calcula el rendimiento desde el servidor en el que se ejecuta la herramienta. Idealmente, este servidor se basa en la guía de ajuste de tamaño del servidor de configuración. Si ya ha implementado los componentes de la infraestructura de Site Recovery de forma local, ejecute la herramienta en el servidor de configuración.

Abra una consola de línea de comandos y vaya a la carpeta de la herramienta de planeamiento de la implementación de Site Recovery. Ejecute ASRDeploymentPlanner.exe con los siguientes parámetros.

`ASRDeploymentPlanner.exe -Operation GetThroughput /?`

|Nombre de parámetro | Descripción |
|-|-|
| -Operation | GetThroughput |
|-Virtualization|Especifique el tipo de virtualización (Hyper-V o VMware).|
| -Directory | (Opcional) El UNC o la ruta de acceso del directorio local en que se almacenan los datos de la generación de perfiles (los archivos que se crean en la generación de perfiles). Estos datos son necesarios para generar el informe. Si no se especifica un nombre de directorio, se utiliza el directorio 'ProfiledData'. |
| -StorageAccountName | El nombre de la cuenta de almacenamiento que se usa para hallar el ancho de banda consumido durante la replicación de datos desde una ubicación local a Azure. La herramienta carga los datos de prueba en esta cuenta de almacenamiento para calcular el ancho de banda consumido. La cuenta de almacenamiento debe ser de tipo de uso general v1 (GPv1).|
| -StorageAccountKey | La clave de la cuenta de almacenamiento utilizada para acceder a dicha cuenta. Vaya a Azure Portal > Cuentas de almacenamiento > <*Nombre de cuenta de almacenamiento*> > Configuración > Claves de acceso > Key1 (o una clave de acceso principal para una cuenta de almacenamiento clásico). |
| -VMListFile | El archivo que contiene la lista de máquinas virtuales de las que se va a generar el perfil para calcular el ancho de banda consumido. La ruta de acceso del archivo puede ser absoluta o relativa. El archivo debe contener un nombre de máquina virtual o una dirección IP en cada línea. Los nombres de máquina virtual especificados en el archivo debe ser los mismo que los nombres de máquina virtual del servidor vCenter/host de vSphere ESXi.<br>Por ejemplo, el archivo VMList.txt contiene las siguientes máquinas virtuales:<ul><li>VM_A</li><li>10.150.29.110</li><li>VM_B</li></ul>|
| -Environment | (Opcional) Se trata del entorno de la cuenta de Azure Storage de destino. Puede ser uno de estos tres valores: AzureCloud, AzureUSGovernment y AzureChinaCloud. El valor predeterminado es AzureCloud. Use el parámetro si la región de destino de Azure se corresponde con Azure US Government o Azure China 21Vianet. |

La herramienta crea varios archivos llamados asrvhdfile<#>.vhd de 64 MB (donde "#" es el número de archivos) en el directorio especificado. La herramienta carga los archivos en la cuenta de almacenamiento para hallar el rendimiento. Después de medir el rendimiento, la herramienta elimina todos estos archivos de la cuenta de almacenamiento y del servidor local. Si la herramienta se termina por cualquier motivo mientras calcula el rendimiento, no elimina los archivos del almacenamiento ni del servidor local. Será preciso eliminarlos manualmente.

El rendimiento se mide en un momento especificado y es el rendimiento máximo que Site Recovery puede lograr durante la replicación, siempre que los restantes factores no varíen. Por ejemplo, si alguna aplicación empieza a consumir más ancho de banda en la misma red, el rendimiento real variará durante la replicación. Si se ejecuta el comando GetThroughput desde un servidor de configuración, la herramienta no detecta las máquinas virtuales protegidas ni la replicación en curso. El resultado de rendimiento medido es distinto si la operación de GetThroughput se ejecuta cuando las máquinas virtuales protegidas tienen una actividad de datos alta. Se recomienda ejecutar la herramienta en varios momentos de la generación de perfiles para saber qué niveles de rendimiento se pueden lograr en cada uno de ellos. En el informe, la herramienta muestra el último rendimiento medido.

### <a name="example"></a>Ejemplo
```
ASRDeploymentPlanner.exe -Operation GetThroughput -Directory  E:\vCenter1_ProfiledData -Virtualization VMware -VMListFile E:\vCenter1_ProfiledData\ProfileVMList1.txt  -StorageAccountName  asrspfarm1 -StorageAccountKey by8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==
```

>[!NOTE]
>
> Ejecute la herramienta en un servidor que tenga las mismas características de almacenamiento y de CPU que el servidor de configuración.
>
> En la replicación, establezca el ancho de banda recomendado de forma que cubra el RPO el 100 % del tiempo. Después de establecer el ancho de banda correcto, si no percibe un aumento en el rendimiento obtenido que notifica la herramienta, siga estos pasos:
>
>  1. Realice las comprobaciones necesarias para determinar si hay algún requisito de calidad de servicio (QoS) de la red que limite el rendimiento de Site Recovery.
>
>  2. Realice las comprobaciones necesarias para determinar si el almacén de Site Recovery está en la región física de Microsoft Azure más cercana admitida para minimizar la latencia de red.
>
>  3. Compruebe las características del almacenamiento local para determinar si puede mejorar el hardware (por ejemplo, pasar de HDD a SSD).
>
>  4. Cambie la configuración de Site Recovery en el servidor de procesos para [aumentar la cantidad de ancho de banda de red que se usa para la replicación](./site-recovery-plan-capacity-vmware.md#control-network-bandwidth).

## <a name="next-steps"></a>Pasos siguientes
* [Análisis del informe generado](site-recovery-vmware-deployment-planner-analyze-report.md).
