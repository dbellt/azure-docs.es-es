---
title: Uso de la tunelización de SSH para acceder a Azure HDInsight
description: Obtenga información acerca de cómo usar un túnel SSH para ir con seguridad a los recursos web alojados en los nodos de HDInsight basados en Linux.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 04/14/2020
ms.openlocfilehash: 60e1e06dfdc904683f2d65e984d3c752e1e6a055
ms.sourcegitcommit: 5f785599310d77a4edcf653d7d3d22466f7e05e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108064690"
---
# <a name="use-ssh-tunneling-to-access-apache-ambari-web-ui-jobhistory-namenode-apache-oozie-and-other-uis"></a>Use la tunelización SSH para tener acceso a la interfaz de usuario web de Apache Ambari, JobHistory, NameNode, Apache Oozie y otras interfaces de usuario.

Los clústeres de HDInsight proporcionan acceso a la interfaz de usuario web de Apache Ambari en Internet. Algunas características requieren un túnel SSH. Por ejemplo, la interfaz de usuario web de Apache Oozie no es accesible a través de Internet sin un túnel SSH.

## <a name="why-use-an-ssh-tunnel"></a>¿Por qué usar un túnel SSH?

Algunos de los menús de Ambari solo funcionan a través de un túnel SSH. Estos menús se basan en sitios web y servicios que se ejecutan en otros tipos de nodo, como nodos de trabajo.

Las siguientes interfaces de usuario web requieren un túnel SSH:

* Historial de trabajos
* NameNode
* Pilas de subprocesos
* Interfaz de usuario web de Oozie
* Interfaz de usuario de registros y maestro de HBase

Los servicios instalados con acciones de script que exponen un servicio web requerirán un túnel SSH. Hue instalado con la acción de script requiere un túnel SSH para acceder a la interfaz de usuario web.

> [!IMPORTANT]  
> Si tiene acceso directo a HDInsight a través de una red virtual, no es necesario usar túneles SSH. Para ver un ejemplo de acceso directo a HDInsight a través de una red virtual, consulte el documento [Conexión de HDInsight a la red local](connect-on-premises-network.md).

## <a name="what-is-an-ssh-tunnel"></a>¿Qué es un túnel SSH?

La [tunelización Secure Shell (SSH) ](https://en.wikipedia.org/wiki/Tunneling_protocol#Secure_Shell_tunneling) conecta un puerto del equipo local a un nodo principal en HDInsight. El tráfico enviado al puerto local se enruta a través de una conexión SSH en el nodo principal. La solicitud se resuelve como si se originara en el nodo principal. A continuación, la respuesta se enruta a través del túnel a la estación de trabajo.

## <a name="prerequisites"></a>Prerrequisitos

* Un cliente SSH. Para más información, consulte [Conexión a través de SSH con HDInsight (Apache Hadoop)](hdinsight-hadoop-linux-use-ssh-unix.md).

* Un explorador web que se puede configurar para usar un proxy SOCKS5.

    > [!WARNING]  
    > La compatibilidad con el proxy SOCKS integrada en la configuración de Internet de Windows no es compatible con SOCKS5 y no funciona con los pasos descritos en este documento. Los siguientes exploradores se basan en la configuración de proxy de Windows y actualmente no funcionan con los pasos descritos en este documento:
    >
    > * Microsoft Edge
    > * Microsoft Internet Explorer
    >
    > Google Chrome también se basa en la configuración de proxy de Windows. Sin embargo, se pueden instalar extensiones que admiten SOCKS5. Se recomienda [FoxyProxy Standard](https://chrome.google.com/webstore/detail/foxyproxy-standard/gcknhkkoolaabfmlnjonogaaifnjlfnp).

## <a name="create-a-tunnel-using-the-ssh-command"></a><a name="usessh"></a>Creación de un túnel con el comando SSH

Use el siguiente comando para crear un túnel SSH con el comando `ssh` . Reemplace `sshuser` por un usuario SSH para su clúster de HDInsight y sustituya `CLUSTERNAME` por el nombre de su clúster de HDInsight.

```cmd
ssh -C2qTnNf -D 9876 sshuser@CLUSTERNAME-ssh.azurehdinsight.net
```

Este comando crea una conexión que enruta el tráfico al puerto local 9876 al clúster a través de SSH. Las opciones son:

|Opción |Descripción |
|---|---|
|D 9876|Puerto local que enruta el tráfico a través del túnel.|
|C|Comprimir todos los datos, debido a que el tráfico web es principalmente texto.|
|2|Forzar a SSH a que intente solo la versión 2 del protocolo.|
|q|Modo silencioso.|
|T|Deshabilitar la asignación seudotty, ya que solo está desviando un puerto.|
|n|Evitar la lectura de STDIN, ya que solo está desviando un puerto.|
|N|No ejecutar un comando remoto, ya que solo está desviando un puerto.|
|f|Ejecutar en segundo plano.|

Una vez que se completa el comando, el tráfico enviado al puerto 9876 de la máquina local se enruta al nodo principal del cluster.

## <a name="create-a-tunnel-using-putty"></a><a name="useputty"></a>Creación de un túnel mediante PuTTY

[PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty) es un cliente SSH gráfico para Windows. Si no está familiarizado con PuTTY, consulte la [documentación de PuTTY ](https://www.chiark.greenend.org.uk/~sgtatham/putty/docs.html). Use los siguientes pasos para crear un túnel SSH con PuTTY:

### <a name="create-or-load-a-session"></a>Creación o carga de una sesión

1. Abra PuTTY y asegúrese de que la opción **Sesión** está seleccionada en el menú izquierdo. Si ya ha guardado una sesión, seleccione el nombre de la sesión en la lista **Sesiones guardadas** y seleccione **Cargar**.

1. Si aún no tiene una sesión guardada, escriba su información de conexión:

    |Propiedad |Value |
    |---|---|
    |Nombre de host (o dirección IP)|La dirección SSH del clúster de HDInsight. Por ejemplo, **mycluster-ssh.azurehdinsight.net**.|
    |Port|22|
    |Tipo de conexión.|SSH|

1. Seleccione **Guardar**.

    :::image type="content" source="./media/hdinsight-linux-ambari-ssh-tunnel/hdinsight-create-putty-session.png" alt-text="Creación de sesión de PuTTY de HDInsight":::

1. En la sección **Category** (Categoría) a la izquierda del cuadro de diálogo, expanda **Connection** (Conexión), **SSH** y, a continuación, seleccione **Tunnels** (Túneles).

1. Proporcione la siguiente información en el formulario **Options controlling SSH port forwarding** (Opciones que controlan el desvío de puertos SSH):

    |Propiedad |Value |
    |---|---|
    |Puerto de origen|El puerto en el cliente que desea desviar. Por ejemplo, **9876**.|
    |Destination|La dirección SSH del clúster de HDInsight. Por ejemplo, **mycluster-ssh.azurehdinsight.net**.|
    |Dinámica|Habilita el enrutamiento dinámico del proxy SOCKS.|

    :::image type="content" source="./media/hdinsight-linux-ambari-ssh-tunnel/hdinsight-putty-tunnel.png" alt-text="Opciones de tunelización de configuración de PuTTY":::

1. Seleccione **Agregar** para agregar la configuración y, a continuación, seleccione **Abrir** para abrir una conexión SSH.

1. Cuando se le solicite, inicie sesión en el servidor.

## <a name="use-the-tunnel-from-your-browser"></a>Uso del túnel desde el explorador

> [!IMPORTANT]  
> Los pasos de esta sección usan el explorador Mozilla FireFox, ya que proporciona la misma configuración de proxy para todas las plataformas. Otros exploradores modernos, como Google Chrome, pueden requerir una extensión como FoxyProxy para funcionar con el túnel.

1. Configure el explorador para usar **localhost** y el puerto que utilizó al crear el túnel como un proxy **SOCKS v5**. La configuración de Firefox se verá de la siguiente manera. Si usa un puerto que no es 9876, cambie el puerto al que usa:

    :::image type="content" source="./media/hdinsight-linux-ambari-ssh-tunnel/firefox-proxy-settings.png" alt-text="configuración de proxy del explorador Firefox":::

   > [!NOTE]  
   > La selección de **DNS remoto** resuelve las solicitudes del sistema de nombres de dominio (DNS) mediante el uso del clúster de HDInsight. Esta configuración resuelve el DNS con el nodo principal del clúster.

2. Compruebe que el túnel funciona; para ello, visite un sitio como [https://www.whatismyip.com/](https://www.whatismyip.com/). La dirección IP devuelta debe ser una que use el centro de datos de Microsoft Azure.

## <a name="verify-with-ambari-web-ui"></a>Compruebe con la interfaz de usuario web de Ambari

Una vez que se ha establecido el clúster, siga estos pasos para comprobar que puede tener acceso a las interfaces de usuario web del servicio desde la web de Ambari:

1. En el explorador, vaya a `http://headnodehost:8080`. La dirección `headnodehost` se envía al clúster a través del túnel y se resuelve en el nodo principal en el que se ejecuta Ambari. Cuando se le solicite, escriba el nombre de usuario administrador (admin) y la contraseña del clúster. Es posible que se le pida una segunda vez mediante la interfaz de usuario web de Ambari. Si es así, vuelva a escribir la información.

   > [!NOTE]  
   > Si utiliza la dirección `http://headnodehost:8080` para conectarse al clúster, se estará conectando a través del túnel. La comunicación se protege si usa el túnel SSH en lugar de HTTPS. Para conectarse a Internet mediante HTTPS, use `https://clustername.azurehdinsight.net`, donde `clustername` es el nombre del clúster.

2. En la interfaz de usuario de Ambari Web, seleccione HDFS en la lista de la izquierda de la página.

    :::image type="content" source="./media/hdinsight-linux-ambari-ssh-tunnel/hdfs-service-selected.png" alt-text="Servicio HDFS de Apache Ambari seleccionado":::

3. Cuando se muestra la información del servicio HDFS, seleccione **Vínculos rápidos**. Aparece una lista de los nodos del clúster principal. Seleccione uno de los nodos principales y luego seleccione **IU de NameNode**.

    :::image type="content" source="./media/hdinsight-linux-ambari-ssh-tunnel/namenode-drop-down-menu.png" alt-text="Imagen con el menú Vínculos rápidos expandido":::

    > [!NOTE]  
    > Al seleccionar __Vínculos rápidos__, es posible que obtenga un indicador de espera. Esta condición puede ocurrir si tiene una conexión lenta a Internet. Espere un minuto o dos hasta que se reciban los datos del servidor e intente de nuevo la lista.
    >
    > Algunas entradas en el menú **Vínculos rápidos** pueden quedar cortadas en el lado derecho de la pantalla. Si es así, expanda el menú con el mouse y use la tecla de dirección derecha para desplazarse por la pantalla hacia la derecha para ver el resto del menú.

4. Aparece una página similar a la siguiente imagen:

    :::image type="content" source="./media/hdinsight-linux-ambari-ssh-tunnel/hdinsight-namenode-ui.png" alt-text="Imagen de la interfaz de usuario de NameNode de Hadoop":::

    > [!NOTE]  
    > Observe la dirección URL de esta página, que debe ser similar a `http://hn*.randomcharacters.cx.internal.cloudapp.net:8088/cluster`. Este identificador URI usa el nombre de dominio completo interno (FQDN) del nodo y solo es accesible si se usa un túnel SSH.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a crear y usar un túnel SSH, consulte el documento siguiente para conocer otras formas de usar Ambari:

* [Administración de clústeres de HDInsight con Apache Ambari](hdinsight-hadoop-manage-ambari.md)
