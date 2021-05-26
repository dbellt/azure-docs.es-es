---
title: 'Replicación de clústeres de HBase en redes virtuales: Azure HDInsight'
description: Aprenda a configurar la replicación de HBase de una versión de HDInsight a otra para conseguir equilibrio de carga, alta disponibilidad, migración y actualizaciones sin tiempo de inactividad y recuperación ante desastres.
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: how-to
ms.date: 12/06/2019
ms.openlocfilehash: 69c1acff7c03ce44ff70b4feb99105a7fbd8ee30
ms.sourcegitcommit: a9f131fb59ac8dc2f7b5774de7aae9279d960d74
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110190245"
---
# <a name="set-up-apache-hbase-cluster-replication-in-azure-virtual-networks"></a>Configuración de la replicación de clústeres de Apache HBase en redes virtuales de Azure

Aprenda a configurar la replicación de [Apache HBase](https://hbase.apache.org/) en una red virtual o entre dos redes virtuales en Azure.

La replicación de clúster usa una metodología de inserción de origen. Un clúster de HBase puede ser un origen, un destino o cumplir ambos roles a la vez. La replicación es asincrónica. El objetivo de la replicación es la coherencia final. Cuando el origen recibe una modificación en una familia de columnas cuando la replicación está habilitada, la modificación se propaga a todos los clústeres de destino. Cuando se replican datos de un clúster a otro, se realiza un seguimiento del clúster de origen y de todos los clústeres que ya han consumido los datos para evitar bucles de replicación.

En este artículo, va a configurar una replicación de origen y destino. Para otras topologías de clúster, consulte la [Guía de referencia de HBase Apache](https://hbase.apache.org/book.html#_cluster_replication).

Los siguientes son casos de uso de replicación de HBase para una única red virtual:

* Equilibrio de carga. Por ejemplo, puede ejecutar exámenes o trabajos MapReduce en el clúster de destino e ingerir datos en el clúster de origen.
* Agregar alta disponibilidad.
* Migrar datos de un clúster de HBase a otro.
* Actualizar un clúster de Azure HDInsight de una versión a otra.

Los siguientes son casos de uso de replicación de HBase para dos redes virtuales:

* Configuración de la recuperación ante desastres.
* Equilibrio de carga y creación de particiones de la aplicación.
* Agregar alta disponibilidad.

Puede replicar clústeres mediante scripts de [acción de script](../hdinsight-hadoop-customize-cluster-linux.md) disponibles en [GitHub](https://github.com/Azure/hbase-utils/tree/master/replication).

## <a name="prerequisites"></a>Requisitos previos
Para comenzar a trabajar con este artículo, es preciso tener una suscripción a Azure. Consulte cómo [obtener una evaluación gratuita de Azure](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/).

## <a name="set-up-the-environments"></a>Configuración de los entornos

Existen tres opciones de configuración:

- Dos clústeres de Apache HBase en una única red virtual de Azure.
- Dos clústeres de Apache HBase en dos redes virtuales diferentes de la misma región.
- Dos clústeres de Apache HBase en dos redes virtuales diferentes de dos regiones diferentes (replicación geográfica).

Este artículo trata sobre el escenario de replicación geográfica.

Para ayudar a configurar los entornos, hemos creado algunas [plantillas de Azure Resource Manager](../../azure-resource-manager/management/overview.md). Si prefiere configurar los entornos mediante otros métodos, consulte:

- [Creación de clústeres de Apache Hadoop en HDInsight](../hdinsight-hadoop-provision-linux-clusters.md)
- [Creación de clústeres de Apache HBase en Azure Virtual Network](apache-hbase-provision-vnet.md)

### <a name="set-up-two-virtual-networks-in-two-different-regions"></a>Configuración de dos redes virtuales en dos regiones distintas

Para usar una plantilla que crea dos redes virtuales en dos regiones distintas y la conexión VPN entre ellas, seleccione el botón **Implementar en Azure** siguiente. La definición de la plantilla se almacena en un [almacenamiento de blobs público](https://hditutorialdata.blob.core.windows.net/hbaseha/azuredeploy.json).

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fhditutorialdata.blob.core.windows.net%2Fhbaseha%2Fazuredeploy.json" target="_blank"><img src="./media/apache-hbase-replication/hdi-deploy-to-azure1.png" alt="Deploy to Azure button for new cluster"></a>

Algunos de los valores de la plantilla están codificados de forma rígida:

**Red virtual 1**

| Propiedad | Value |
|----------|-------|
| Location | Oeste de EE. UU. |
| Nombre de red virtual | &lt;ClusterNamePrevix>-vnet1 |
| Prefijo del espacio de direcciones | 10.1.0.0/16 |
| Nombre de subred | subred 1 |
| Prefijo de la subred | 10.1.0.0/24 |
| Nombre de subred (puerta de enlace) | GatewaySubnet (no se puede cambiar) |
| Prefijo de la subred (puerta de enlace) | 10.1.255.0/27 |
| Nombre de puerta de enlace | vnet1gw |
| Tipo de puerta de enlace | VPN |
| Tipo de VPN de la puerta de enlace | RouteBased |
| SKU de puerta de enlace | Básico |
| Dirección IP de puerta de enlace | vnet1gwip |

**Red virtual 2**

| Propiedad | Value |
|----------|-------|
| Location | Este de EE. UU. |
| Nombre de red virtual | &lt;ClusterNamePrevix>-vnet2 |
| Prefijo del espacio de direcciones | 10.2.0.0/16 |
| Nombre de subred | subred 1 |
| Prefijo de la subred | 10.2.0.0/24 |
| Nombre de subred (puerta de enlace) | GatewaySubnet (no se puede cambiar) |
| Prefijo de la subred (puerta de enlace) | 10.2.255.0/27 |
| Nombre de puerta de enlace | vnet2gw |
| Tipo de puerta de enlace | VPN |
| Tipo de VPN de la puerta de enlace | RouteBased |
| SKU de puerta de enlace | Básico |
| Dirección IP de puerta de enlace | vnet1gwip |

## <a name="setup-dns"></a>Configuración de DNS

En la última sección, la plantilla crea una máquina virtual de Ubuntu en cada una de las dos redes virtuales.  En esta sección, puede instalar Bind en las dos máquinas virtuales del DNS y, después, configurar el reenvío de DNS en ambas.

Para instalar Bind, debe encontrar la dirección IP pública de las dos máquinas virtuales del DNS.

1. Abra [Azure Portal](https://portal.azure.com).
2. Abra la máquina virtual del DNS seleccionando **Grupos de recursos > [nombre del grupo de recursos] > [vnet1DNS]**.  El nombre del grupo de recursos es el que creó en el último procedimiento. Los nombres predeterminados de las máquinas virtuales del DNS son *vnet1DNS* y *vnet2NDS*.
3. Seleccione **Propiedades** para abrir la página de propiedades de la red virtual.
4. Anote la **dirección IP pública** y compruebe también la **dirección IP privada**.  La dirección IP privada debe ser **10.1.0.4** para vnet1DNS y **10.2.0.4** para vnet2DNS.  
5. Cambie los servidores DNS de ambas redes virtuales para usar servidores DNS predeterminados (proporcionados por Azure) para permitir el acceso de entrada y de salida para descargar los paquetes para instalar Bind en los pasos siguientes.

Para instalar Bind, use el procedimiento siguiente:

1. Use SSH para conectarse a la __dirección IP pública__ de la máquina virtual del DNS. En el ejemplo siguiente, se realiza una conexión a una máquina virtual en 40.68.254.142:

    ```bash
    ssh sshuser@40.68.254.142
    ```

    Reemplace `sshuser` por la cuenta de usuario SSH que especificó cuando creó la máquina virtual de DNS.

    > [!NOTE]  
    > Existen diversas formas de obtener la utilidad `ssh`. En Linux, Unix y macOS, se proporciona como parte del sistema operativo. Si usa Windows, considere una de las opciones siguientes:
    >
    > * [Azure Cloud Shell](../../cloud-shell/quickstart.md)
    > * [Bash en Ubuntu en Windows 10](/windows/wsl/about)
    > * [Git (https://git-scm.com/)](https://git-scm.com/)
    > * [OpenSSH (https://github.com/PowerShell/Win32-OpenSSH/wiki/Install-Win32-OpenSSH)](https://github.com/PowerShell/Win32-OpenSSH/wiki/Install-Win32-OpenSSH)

2. Para instalar Bind, use los comandos siguientes en la sesión de SSH:

    ```bash
    sudo apt-get update -y
    sudo apt-get install bind9 -y
    ```

3. Configure Bind para reenviar las solicitudes de resolución de nombre al servidor DNS local. Para ello, use el texto siguiente como contenido del archivo `/etc/bind/named.conf.options`:

    ```
    acl goodclients {
        10.1.0.0/16; # Replace with the IP address range of the virtual network 1
        10.2.0.0/16; # Replace with the IP address range of the virtual network 2
        localhost;
        localhost;
    };
    
    options {
        directory "/var/cache/bind";
        recursion yes;
        allow-query { goodclients; };

        forwarders {
            168.63.129.16; #This is the Azure DNS server
        };

        dnssec-validation auto;

        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
    };
    ```
    
    > [!IMPORTANT]  
    > Reemplace los valores de la sección `goodclients` por el intervalo de direcciones IP de las dos redes virtuales. En esta sección se definen las direcciones desde las cuales el servidor DNS acepta solicitudes.

    Para editar el archivo, use el comando siguiente:

    ```bash
    sudo nano /etc/bind/named.conf.options
    ```

    Para guardar el archivo, use __Ctrl+X__, __Y__ y, luego, __Entrar__.

4. En la sesión de SSH, use el comando siguiente:

    ```bash
    hostname -f
    ```

    Este comando devuelve un valor similar al siguiente texto:

    ```output
    vnet1DNS.icb0d0thtw0ebifqt0g1jycdxd.ex.internal.cloudapp.net
    ```

    El texto `icb0d0thtw0ebifqt0g1jycdxd.ex.internal.cloudapp.net` está en el __sufijo DNS__ de esta red virtual. Guarde este valor, porque se usará más adelante.

    También deberá averiguar el sufijo DNS del otro servidor DNS. La necesitará en el paso siguiente.

5. Para configurar Bind a fin de resolver nombres DNS para recursos dentro de la red virtual, use el texto siguiente como contenido del archivo `/etc/bind/named.conf.local`:

    ```
    // Replace the following with the DNS suffix for your virtual network
    zone "v5ant3az2hbe1edzthhvwwkcse.bx.internal.cloudapp.net" {
            type forward;
            forwarders {10.2.0.4;}; # The Azure recursive resolver
    };
    ```

    > [!IMPORTANT]  
    > Debe reemplazar el valor `v5ant3az2hbe1edzthhvwwkcse.bx.internal.cloudapp.net` por el sufijo DNS de la otra red virtual. Y la dirección IP del reenviador es la dirección IP privada del servidor DNS de la otra red virtual.

    Para editar el archivo, use el comando siguiente:

    ```bash
    sudo nano /etc/bind/named.conf.local
    ```

    Para guardar el archivo, use __Ctrl+X__, __Y__ y, luego, __Entrar__.

6. Para iniciar Bind, use el comando siguiente:

    ```bash
    sudo service bind9 restart
    ```

7. Para comprobar que Bind puede resolver los nombres de recursos en la otra red virtual, use los comandos siguientes:

    ```bash
    sudo apt install dnsutils
    nslookup vnet2dns.v5ant3az2hbe1edzthhvwwkcse.bx.internal.cloudapp.net
    ```

    > [!IMPORTANT]  
    > Reemplace `vnet2dns.v5ant3az2hbe1edzthhvwwkcse.bx.internal.cloudapp.net` por el nombre de dominio completo (FQDN) de la máquina virtual de DNS de la otra red.
    >
    > Reemplace `10.2.0.4` por la __dirección IP interna__ del servidor DNS personalizado de la otra red virtual.

    La respuesta es similar al texto siguiente:

    ```output
    Server:         10.2.0.4
    Address:        10.2.0.4#53
    
    Non-authoritative answer:
    Name:   vnet2dns.v5ant3az2hbe1edzthhvwwkcse.bx.internal.cloudapp.net
    Address: 10.2.0.4
    ```

    Hasta ahora, no podía buscar la dirección IP de la otra red sin especificar la dirección IP del servidor DNS.

### <a name="configure-the-virtual-network-to-use-the-custom-dns-server"></a>Configuración de la red virtual para usar el servidor DNS personalizado

Para configurar la red virtual para que use el servidor DNS personalizado en lugar de la resolución recursiva de Azure, use los pasos siguientes:

1. En [Azure Portal](https://portal.azure.com), seleccione la red virtual y, luego, seleccione __Servidores DNS__.

2. Seleccione __Personalizar__ y escriba la __dirección IP interna__ del servidor DNS personalizado. Por último, seleccione __Guardar__.

6. Abra la máquina virtual del servidor DNS en vnet1 y haga clic en **Reiniciar**.  Debe reiniciar todas las máquinas virtuales de la red virtual para que la configuración de DNS surta efecto.
7. Repita los pasos para configurar el servidor DNS personalizado para vnet2.

Para probar la configuración de DNS, puede conectarse a las dos máquinas virtuales de DNS mediante SSH y hacer ping en el servidor DNS de la otra red virtual mediante su nombre de host. Si no funciona, utilice el siguiente comando para comprobar el estado del DNS:

```bash
sudo service bind9 status
```

## <a name="create-apache-hbase-clusters"></a>Creación de clústeres de Apache HBase

Cree un clúster de [Apache HBase](https://hbase.apache.org/) en cada una de las dos redes virtuales con la siguiente configuración:

- **Nombre del grupo de recursos**: use el mismo nombre del grupo de recursos que cuando creó las redes virtuales.
- **Tipo de clúster**: HBase
- **Versión**: HBase 1.1.2 (HDI 3.6)
- **Ubicación**: seleccione la misma ubicación que la red virtual.  De forma predeterminada, para vnet1 es *Oeste de EE. UU.* y para vnet2 es *Este de EE. UU*.
- **Almacenamiento**: cree una nueva cuenta de almacenamiento para el clúster.
- **Red virtual** (en Configuración avanzada en el portal): seleccione vnet1, la máquina virtual que creó en el último procedimiento.
- **Subred**: el nombre predeterminado que se utiliza en la plantilla es **subnet1**.

Para asegurarse de que el entorno está configurado correctamente, debe poder hacer ping en el FQDN del nodo principal entre los dos clústeres.

## <a name="load-test-data"></a>Carga de datos de prueba

Al replicar un clúster, debe especificar las tablas que quere replicar. En esta sección, va a cargar algunos datos en el clúster de origen. En la siguiente sección, habilitará la replicación entre los dos clústeres.

Para crear una tabla **Contacts** e insertar algunos datos en ella, siga las instrucciones que se indican en el [tutorial de HBase sobre la introducción al uso de Apache HBase en HDInsight](apache-hbase-tutorial-get-started-linux.md).

> [!NOTE]
> Si desea replicar tablas desde un espacio de nombres personalizado, debe asegurarse de que también se definen los espacios de nombres personalizados adecuados en el clúster de destino.
>

## <a name="enable-replication"></a>Habilitar replicación

En los pasos siguientes se describe cómo llamar al script de acción de script desde Azure Portal. Para información sobre la ejecución de una acción de script mediante Azure PowerShell y la CLI clásica de Azure, consulte [Personalización de clústeres de HDInsight mediante la acción de scripts](../hdinsight-hadoop-customize-cluster-linux.md).

**Para habilitar la replicación de HBase desde Azure Portal**

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Abra el clúster de HBase de origen.
3. En el menú de clúster, seleccione **Acciones de script**.
4. En la parte superior de la página, seleccione **Enviar nuevo**.
5. Seleccione o escriba la siguiente información:

   1. **Nombre** especifique **Enable replication** (Habilitar replicación).
   2. **URL de script de Bash**: escriba **https://raw.githubusercontent.com/Azure/hbase-utils/master/replication/hdi_enable_replication.sh**.
   3. **Principal**: asegúrese de que esta opción está seleccionada. Borre los demás tipos de nodo.
   4. **Parámetros**: los siguientes parámetros de ejemplo permiten la replicación en todas las tablas existentes y copian todos los datos del clúster de origen al clúster de destino:

    `-m hn1 -s <source hbase cluster name> -d <destination hbase cluster name> -sp <source cluster Ambari password> -dp <destination cluster Ambari password> -copydata`
    
      > [!NOTE]
      > Use el nombre de host en lugar de FQDN para el nombre DNS del clúster de origen y de destino.
      >
      > En este tutorial se da por supuesto que hn1 es el nodo principal activo. Compruebe el clúster para identificar el nodo principal activo.

6. Seleccione **Crear**. El script puede tardar un poco en ejecutarse, especialmente cuando se usa el argumento **-copydata**.

Argumentos necesarios:

|Nombre|Descripción|
|----|-----------|
|-s, --src-cluster | Especifica el nombre DNS del clúster de HBase de origen. Por ejemplo: -s hbsrccluster, --src-cluster=hbsrccluster |
|-d, --dst-cluster | Especifica el nombre DNS del clúster de HBase de destino (réplica). Por ejemplo: -s dsthbcluster, --src-cluster=dsthbcluster |
|-sp, --src-ambari-password | Especifica la contraseña de administrador de Ambari en el clúster de HBase de origen. |
|-dp, --dst-ambari-password | Especifica la contraseña de administrador de Ambari en el clúster de HBase de destino.|

Argumentos opcionales:

|Nombre|Descripción|
|----|-----------|
|-su, --src-ambari-user | Especifica el nombre de usuario de administrador de Ambari en el clúster de HBase de origen. El valor predeterminado es **admin**. |
|-du, --dst-ambari-user | Especifica el nombre de usuario de administrador de Ambari en el clúster de HBase de destino. El valor predeterminado es **admin**. |
|-t, --table-list | Especifica las tablas que se van a replicar. Por ejemplo: --table-list="table1;table2;table3". Si no se especifica unas tablas determinadas, se replican todas las tablas de HBase existentes.|
|-m, --machine | Especifica el nodo principal en el que se ejecuta la acción de script. El valor debe elegirse en función del nodo principal activo. Use esta opción si se está ejecutando el script $0 como acción de script desde el portal de HDInsight o Azure PowerShell.|
|-cp, -copydata | Habilita la migración de datos existentes en las tablas en las que está habilitada la replicación. |
|-rpm, -replicate-phoenix-meta | Habilita la replicación en las tablas del sistema Phoenix. <br><br>*Esta opción se debe utilizar con precaución.* Se recomienda volver a crear tablas de Phoenix en clústeres de réplica antes de utilizar este script. |
|-h, --help | Muestra información de uso. |

La sección `print_usage()` del [script](https://github.com/Azure/hbase-utils/blob/master/replication/hdi_enable_replication.sh) contiene una explicación detallada de los parámetros.

Una vez implementada correctamente la acción de script, puede usar SSH para conectarse al clúster de HBase de destino y comprobar que los datos se hayan replicado.

### <a name="replication-scenarios"></a>Escenarios de replicación

En la lista siguiente se muestran algunos casos de uso general y la configuración de parámetros:

- **Habilitar la replicación en todas las tablas entre los dos clústeres**. En este escenario no es necesario copiar o migrar datos existentes de las tablas y no se usan tablas de Phoenix. Use los parámetros siguientes:

  `-m hn1 -s <source hbase cluster name> -d <destination hbase cluster name> -sp <source cluster Ambari password> -dp <destination cluster Ambari password>`

- **Habilitar la replicación en tablas específicas**. Para habilitar la replicación en table1, table2 y table3, use los parámetros siguientes:

  `-m hn1 -s <source hbase cluster name> -d <destination hbase cluster name> -sp <source cluster Ambari password> -dp <destination cluster Ambari password> -t "table1;table2;table3"`

- **Habilitar la replicación en tablas específicas y copiar los datos existentes**. Para habilitar la replicación en table1, table2 y table3, use los parámetros siguientes:

  `-m hn1 -s <source hbase cluster name> -d <destination hbase cluster name> -sp <source cluster Ambari password> -dp <destination cluster Ambari password> -t "table1;table2;table3" -copydata`

- **Habilitar la replicación en todas las tablas y replicar metadatos de Phoenix del origen al destino**. La replicación de metadatos de Phoenix no es perfecta. Úsela con precaución. Use los parámetros siguientes:

  `-m hn1 -s <source hbase cluster name> -d <destination hbase cluster name> -sp <source cluster Ambari password> -dp <destination cluster Ambari password> -t "table1;table2;table3" -replicate-phoenix-meta`

## <a name="copy-and-migrate-data"></a>Copia y migración de datos

Hay dos scripts de acción de script independientes para copiar o migrar datos después de habilitar la replicación:

- [Script para tablas pequeñas](https://raw.githubusercontent.com/Azure/hbase-utils/master/replication/hdi_copy_table.sh) (tablas que tienen unos cuantos gigabytes de tamaño y se espera que una copia general finalice en menos de una hora)

- [Script para tablas grandes](https://raw.githubusercontent.com/Azure/hbase-utils/master/replication/nohup_hdi_copy_table.sh) (tablas que se espera que tarden más de una hora en copiarse)

Puede seguir el mismo procedimiento que se describe en [Habilitar replicación](#enable-replication) para llamar a la acción de script. Use los parámetros siguientes:

`-m hn1 -t <table1:start_timestamp:end_timestamp;table2:start_timestamp:end_timestamp;...> -p <replication_peer> [-everythingTillNow]`

La sección `print_usage()` del [script](https://github.com/Azure/hbase-utils/blob/master/replication/hdi_copy_table.sh) contiene una descripción detallada de los parámetros.

### <a name="scenarios"></a>Escenarios

- **Copiar tablas específicas (test1, test2 y test3) con todas las filas editadas hasta ahora (marca de tiempo actual)**:

  `-m hn1 -t "test1::;test2::;test3::" -p "<zookeepername1>;<zookeepername2>;<zookeepername3>:2181:/hbase-unsecure" -everythingTillNow`

  O:

  `-m hn1 -t "test1::;test2::;test3::" --replication-peer="<zookeepername1>;<zookeepername2>;<zookeepername3>:2181:/hbase-unsecure" -everythingTillNow`

- **Copiar tablas específicas con un intervalo de tiempo especificado**:

  `-m hn1 -t "table1:0:452256397;table2:14141444:452256397" -p "<zookeepername1>;<zookeepername2>;<zookeepername3>:2181:/hbase-unsecure"`

## <a name="disable-replication"></a>Deshabilitar replicación

Para deshabilitar la replicación, use otro script de acción de script de [GitHub](https://raw.githubusercontent.com/Azure/hbase-utils/master/replication/hdi_disable_replication.sh). Puede seguir el mismo procedimiento que se describe en [Habilitar replicación](#enable-replication) para llamar a la acción de script. Use los parámetros siguientes:

`-m hn1 -s <source hbase cluster name> -sp <source cluster Ambari password> <-all|-t "table1;table2;...">`

La sección `print_usage()` del [script](https://raw.githubusercontent.com/Azure/hbase-utils/master/replication/hdi_disable_replication.sh) contiene una explicación detallada de los parámetros.

### <a name="scenarios"></a>Escenarios

- **Deshabilitar la replicación en todas las tablas**:

  `-m hn1 -s <source hbase cluster name> -sp Mypassword\!789 -all`

  or

  `--src-cluster=<source hbase cluster name> --dst-cluster=<destination hbase cluster name> --src-ambari-user=<source cluster Ambari user name> --src-ambari-password=<source cluster Ambari password>`

- **Deshabilitar la replicación en las tablas especificadas (table1, table2 y table3)**:

  `-m hn1 -s <source hbase cluster name> -sp <source cluster Ambari password> -t "table1;table2;table3"`

> [!NOTE]
> Si piensa eliminar el clúster de destino, asegúrese de quitarlo de la lista del mismo nivel del clúster de origen. Esto puede ejecutando el comando remove_peer '1' en el shell de HBase en el clúster de origen. Si se produce un error, es posible que el clúster de origen no funcione correctamente.
>

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a configurar la replicación de Apache HBase en una red virtual, o entre dos redes virtuales. Para más información sobre HDInsight y Apache HBase, consulte estos artículos:

* [Introducción a HBase Apache en HDInsight](./apache-hbase-tutorial-get-started-linux.md)
* [Introducción a Apache HBase de HDInsight](./apache-hbase-overview.md)
* [Creación de clústeres de Apache HBase en Azure Virtual Network](./apache-hbase-provision-vnet.md)