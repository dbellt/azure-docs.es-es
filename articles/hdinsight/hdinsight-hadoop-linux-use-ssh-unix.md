---
title: Uso de SSH con Hadoop en Azure HDInsight
description: Puede acceder a HDInsight mediante Secure Shell (SSH). Este documento proporciona información sobre cómo conectarse a HDInsight utilizando los comandos ssh desde clientes Windows, Linux, Unix o macOS.
ms.service: hdinsight
ms.topic: how-to
ms.custom: H1Hack27Feb2017,hdinsightactive,hdiseo17may2017,seoapr2020
ms.date: 02/28/2020
ms.openlocfilehash: 7d7ba260177cb27ac181f081335ffb1e19aa5c33
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110062801"
---
# <a name="connect-to-hdinsight-apache-hadoop-using-ssh"></a>Conexión a HDInsight (Apache Hadoop) mediante SSH

Aprenda a usar [Secure Shell (SSH)](https://en.wikipedia.org/wiki/Secure_Shell) para conectarse de forma segura a Apache Hadoop en Azure HDInsight. Para obtener información acerca de cómo conectarse a través de una red virtual, consulte [Arquitectura de red virtual de Azure HDInsight](./hdinsight-virtual-network-architecture.md). Vea también [Planeamiento de la implementación de una red virtual para clústeres de Azure HDInsight](./hdinsight-plan-virtual-network-deployment.md).

La tabla siguiente contiene la información de dirección y puerto necesaria para conectarse a HDInsight a través de un cliente SSH:

| Dirección | Port | Se conecta a... |
| ----- | ----- | ----- |
| `<clustername>-ssh.azurehdinsight.net` | 22 | Nodo principal primario |
| `<clustername>-ssh.azurehdinsight.net` | 23 | Nodo principal secundario |
| `<clustername>-ed-ssh.azurehdinsight.net` | 22 | Nodo perimetral (Machine Learning Services en HDInsight) |
| `<edgenodename>.<clustername>-ssh.azurehdinsight.net` | 22 | Nodo perimetral (cualquier otro tipo de clúster, si existe un nodo perimetral) |

Reemplace `<clustername>` por el nombre del clúster. Reemplace `<edgenodename>` con el nombre del nodo perimetral.

Si el clúster contiene un nodo perimetral, se recomienda que __siempre se conecte al nodo perimetral__ mediante SSH. Los nodos principales hospedan servicios que son críticos para el estado de Hadoop. El nodo perimetral ejecuta solo lo que incluya en él. Para más información, consulte [Uso de nodos perimetrales en HDInsight](hdinsight-apps-use-edge-node.md#access-an-edge-node).

> [!TIP]
> Cuando se conecte por primera vez a HDInsight, el cliente SSH puede mostrar una advertencia de que no se puede establecer la autenticidad del host. Cuando se le pida, seleccione "sí" para agregar el host a la lista de servidores de confianza de su cliente SSH.
>
> Si se ha conectado anteriormente a un servidor con el mismo nombre, puede recibir una advertencia de que la clave del host almacenada no coincide con la del servidor. Consulte la documentación del cliente SSH sobre cómo el nombre del servidor existente.

## <a name="ssh-clients"></a>Clientes SSH

Los sistemas Linux, Unix y macOS proporcionan los comandos `ssh` y `scp`. El cliente `ssh` se usa normalmente para crear una sesión de línea de comandos remota con un sistema basado en Unix o Linux. El cliente `scp` se usa para copiar archivos entre el cliente y el sistema remoto de forma segura.

Microsoft Windows no instala los clientes SSH de forma predeterminada. Los clientes `ssh` y `scp` están disponibles para Windows a través de los siguientes paquetes:

* [Cliente OpenSSH](/windows-server/administration/openssh/openssh_install_firstuse): Este cliente es una característica opcional que se introdujo en Windows 10 Fall Creators Update.

* [Bash en Ubuntu en Windows 10](/windows/wsl/about).

* [Azure Cloud Shell](../cloud-shell/quickstart.md). Cloud Shell proporciona un entorno de Bash en el explorador.

* [Git](https://git-scm.com/).

También hay varios clientes SSH gráficos, como [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/) y [MobaXterm](https://mobaxterm.mobatek.net/). Aunque estos clientes se pueden usar para conectar con HDInsight, el proceso de conexión es diferente a usar la utilidad `ssh`. Para más información, consulte la documentación del cliente gráfico que esté usando.

## <a name="authentication-ssh-keys"></a><a id="sshkey"></a>Autenticación: claves SSH

Las claves SSH utilizan [criptografía de clave pública](https://en.wikipedia.org/wiki/Public-key_cryptography) para autenticar las sesiones de SSH. Las claves SSH son más seguras que las contraseñas y proporcionan una manera fácil para proteger el acceso al clúster de Hadoop.

Si su cuenta SSH se protege utilizando una clave, el cliente tiene que proporcionar la clave privada correspondiente al conectarse:

* La mayoría de los clientes pueden configurarse para utilizar una __clave predeterminada__. Por ejemplo, el cliente `ssh` busca una clave privada en `~/.ssh/id_rsa` en entornos Linux y Unix.

* Puede especificar la __ruta de acceso a una clave privada__. Con el cliente `ssh`, el parámetro `-i` se utiliza para especificar la ruta de acceso a la clave privada. Por ejemplo, `ssh -i ~/.ssh/id_rsa sshuser@myedge.mycluster-ssh.azurehdinsight.net`.

* Si tiene __varias claves privadas__ para su uso con distintos servidores, considere la posibilidad de usar una utilidad como [ssh-agent (https://en.wikipedia.org/wiki/Ssh-agent)](https://en.wikipedia.org/wiki/Ssh-agent). La utilidad `ssh-agent` se puede usar para seleccionar automáticamente la clave que se usará al establecer una sesión de SSH.

> [!IMPORTANT]
> Si se protege la clave privada con una frase de contraseña, tiene que escribir la frase de contraseña cuando se use la clave. Utilidades como `ssh-agent` puede almacenar en caché la contraseña para su comodidad.

### <a name="create-an-ssh-key-pair"></a>Creación de un par de claves SSH

Use el comando `ssh-keygen` para crear archivos de clave pública y privada. El comando siguiente genera un par de claves de RSA de 2048 bits que puede utilizarse con HDInsight:

```azurepowershell-interactive
ssh-keygen -t rsa -b 2048
```

Se le pedirá información durante el proceso de creación de claves. Por ejemplo, dónde se almacenan las claves o si se debe usar una frase de contraseña. Una vez completado el proceso, se crean dos archivos: una clave pública y una clave privada.

* La __clave pública__ se utiliza para crear un clúster de HDInsight. La clave pública tiene una extensión `.pub`.

* La __clave privada__ se utiliza para autenticar el cliente al clúster de HDInsight.

> [!IMPORTANT]
> Puede proteger las claves con una frase de contraseña. Una frase de contraseña es efectivamente una contraseña de la clave privada. Incluso si alguien obtiene la clave privada, tienen que tener la frase de contraseña para poder usar la clave.

### <a name="create-hdinsight-using-the-public-key"></a>Creación de HDInsight usando la clave pública

| Método de creación | Cómo usar la clave pública |
| ------- | ------- |
| Azure Portal | Desactive la opción __Usar contraseña de inicio de sesión de clúster para SSH__ y, a continuación, seleccione __Clave pública__ como el tipo de autenticación de SSH. Por último, seleccione el archivo de clave pública o pegue el contenido de texto del archivo en el campo __Clave pública SSH__.</br>:::image type="content" source="./media/hdinsight-hadoop-linux-use-ssh-unix/create-hdinsight-ssh-public-key.png" alt-text="Cuadro de diálogo de clave pública SSH en la creación de clústeres de HDInsight"::: |
| Azure PowerShell | Use el parámetro `-SshPublicKey` del cmdlet [New-AzHdinsightCluster](/powershell/module/az.hdinsight/new-azhdinsightcluster) y pase el contenido de la clave pública como una cadena.|
| Azure CLI | Use el parámetro `--sshPublicKey` del comando [`az hdinsight create`](/cli/azure/hdinsight#az_hdinsight_create) y pase el contenido de la clave pública como una cadena. |
| Plantilla de Resource Manager | Para obtener un ejemplo del uso de claves SSH con una plantilla, consulte la [implementación de HDInsight en Linux con una clave SSH](https://azure.microsoft.com/resources/templates/quickstarts/microsoft.hdinsight/hdinsight-linux-ssh-publickey/). El elemento `publicKeys` en el archivo [azuredeploy.json](https://github.com/Azure/azure-quickstart-templates/blob/master/quickstarts/microsoft.hdinsight/hdinsight-linux-ssh-publickey/azuredeploy.json) se usa para pasar las claves a Azure al crear el clúster. |

## <a name="authentication-password"></a>Autenticación: contraseña

Las cuentas SSH se pueden proteger mediante una contraseña. Cuando se conecte a HDInsight usando SSH, se le pedirá que escriba la contraseña.

> [!WARNING]
> Microsoft no recomienda usar la autenticación de contraseña para SSH. Las contraseñas se pueden adivinar y son vulnerables a ataques por fuerza bruta. En su lugar, recomendamos que use [claves SSH para la autenticación](#sshkey).

> [!IMPORTANT]
> La contraseña de la cuenta SSH expira 70 días después de crear el clúster de HDInsight. Si la contraseña expira, se puede cambiar con la información del documento sobre la [administración de HDInsight](hdinsight-administer-use-portal-linux.md#change-passwords).

### <a name="create-hdinsight-using-a-password"></a>Creación de HDInsight usando una contraseña

| Método de creación | Cómo especificar la contraseña |
| --------------- | ---------------- |
| Azure Portal | De forma predeterminada, la cuenta de usuario SSH tiene la misma contraseña que la cuenta de inicio de sesión de clúster. Para usar una contraseña diferente, desactive la opción __Usar contraseña de inicio de sesión de clúster para SSH__ y, a continuación, escriba la contraseña en el campo __Contraseña SSH__.</br>:::image type="content" source="./media/hdinsight-hadoop-linux-use-ssh-unix/create-hdinsight-ssh-password.png" alt-text="Cuadro de diálogo de contraseña SSH en la creación de clústeres de HDInsight":::|
| Azure PowerShell | Use el parámetro `--SshCredential` del cmdlet [New-AzHdinsightCluster](/powershell/module/az.hdinsight/new-azhdinsightcluster) y pase un objeto `PSCredential` que contenga el nombre de cuenta de usuario SSH y la contraseña. |
| Azure CLI | Use el parámetro `--ssh-password` del comando [`az hdinsight create`](/cli/azure/hdinsight#az_hdinsight_create) y proporcione el valor de contraseña. |
| Plantilla de Resource Manager | Para obtener un ejemplo del uso de contraseña con una plantilla, consulte la [implementación de HDInsight en Linux con una contraseña SSH](https://azure.microsoft.com/resources/templates/101-hdinsight-linux-ssh-password/). El elemento `linuxOperatingSystemProfile` en el archivo [azuredeploy.json](https://github.com/Azure/azure-quickstart-templates/blob/master/quickstarts/microsoft.hdinsight/hdinsight-linux-ssh-password/azuredeploy.json) se usa para pasar el nombre de cuenta de SSH y la contraseña a Azure al crear el clúster.|

### <a name="change-the-ssh-password"></a>Cambio de contraseña de SSH

Para información acerca de cómo cambiar la contraseña de cuenta de usuario SSH, consulte la sección __Cambio de contraseñas__ del documento sobre [administración de HDInsight](hdinsight-administer-use-portal-linux.md#change-passwords).

## <a name="authentication-domain-joined-hdinsight"></a>Autenticación de HDInsight unido a un dominio

Si está usando un __clúster de HDInsight unido a un dominio__, tiene que utilizar el comando `kinit` después de conectarse con el usuario local de SSH. Este comando le pide un usuario de dominio y una contraseña y autentica la sesión con el dominio de Azure Active Directory asociado con el clúster.

También puede habilitar la autenticación Kerberos en cada nodo unido a un dominio (por ejemplo, nodo principal, nodo perimetral) para realizar un ssh mediante la cuenta de dominio. Para ello, edite el archivo de configuración de sshd:

```bash
sudo vi /etc/ssh/sshd_config
```

Quite los comentarios y cambie `KerberosAuthentication` a `yes`

```bash
sudo service sshd restart
```

Utilice el comando `klist` para verificar si la autenticación Kerberos se realizó correctamente.

Para más información, consulte [Configuración de clústeres de HDInsight unidos a un dominio](./domain-joined/apache-domain-joined-configure-using-azure-adds.md).

## <a name="connect-to-nodes"></a>Conexión a nodos

Los nodos principales y el nodo perimetral (si hay alguno) son accesibles a través de Internet en los puertos 22 y 23.

* Cuando se conecte a los __nodos principales__, use el puerto __22__ para conectarse al nodo principal primario y el puerto __23__ para conectarse al nodo principal secundario. El nombre de dominio completo que se debe usar es `clustername-ssh.azurehdinsight.net`, donde `clustername` es el nombre del clúster.

    ```bash
    # Connect to primary head node
    # port not specified since 22 is the default
    ssh sshuser@clustername-ssh.azurehdinsight.net

    # Connect to secondary head node
    ssh -p 23 sshuser@clustername-ssh.azurehdinsight.net
    ```

* Al conectarse al __nodo perimetral__, utilice el puerto 22. El nombre de dominio completo es `edgenodename.clustername-ssh.azurehdinsight.net`, donde `edgenodename` es el nombre que proporcionó al crear el nodo perimetral. `clustername` es el nombre del clúster.

    ```bash
    # Connect to edge node
    ssh sshuser@edgnodename.clustername-ssh.azurehdinsight.net
    ```

> [!IMPORTANT]
> En los ejemplos anteriores se supone que está usando la autenticación con contraseña, o que la autenticación de certificados se está produciendo automáticamente. Si usa un par de claves SSH para la autenticación y el certificado no se utiliza automáticamente, use el parámetro `-i` para especificar la clave privada. Por ejemplo, `ssh -i ~/.ssh/mykey sshuser@clustername-ssh.azurehdinsight.net`.

Una vez conectado, el símbolo del sistema cambia para indicar el nombre de usuario SSH y el nodo al que está conectado. Por ejemplo, cuando se conecta al nodo principal primario como `sshuser`, el símbolo del sistema es `sshuser@<active-headnode-name>:~$`.

### <a name="connect-to-worker-and-apache-zookeeper-nodes"></a>Conexión a los nodos de Apache Zookeeper y de trabajo

Los nodos de trabajo y los nodos de Zookeeper no son directamente accesibles desde Internet. Solo se puede acceder a ellos desde los nodos principales y perimetrales del clúster. Estos son los pasos generales para conectarse a otros nodos:

1. Use SSH para conectarse al nodo principal o perimetral:

    ```bash
    ssh sshuser@myedge.mycluster-ssh.azurehdinsight.net
    ```

2. En la conexión SSH con el nodo principal o perimetral, use el comando `ssh` para conectarse a un nodo de trabajador en el clúster:

    ```bash
    ssh sshuser@wn0-myhdi
    ```

    Para recuperar una lista de los nombres de nodos, consulte el documento [Administración de HDInsight con la API REST de Apache Ambari](hdinsight-hadoop-manage-ambari-rest-api.md#get-the-fqdn-of-cluster-nodes).

Si la cuenta SSH se protege utilizando un __contraseña__, escriba la contraseña al conectarse.

Si la cuenta SSH se protege utilizando __claves SSH__, asegúrese de que el reenvío de SSH está habilitado en el cliente.

> [!NOTE]
> Otra manera de obtener acceso directo a todos los nodos del clúster es instalar HDInsight en una instancia de Azure Virtual Network. A continuación, puede unir la máquina remota a la misma red virtual y tener acceso directamente a todos los nodos del clúster.
>
> Para más información, consulte [Planeamiento de una red virtual para HDInsight](hdinsight-plan-virtual-network-deployment.md).

### <a name="configure-ssh-agent-forwarding"></a>Configuración del reenvío del agente SSH

> [!IMPORTANT]
> Los pasos siguientes presuponen un sistema basado en Linux o UNIX y funcionan con Bash en Windows 10. Si estos pasos no funcionan en su sistema, es posible que deba consultar la documentación para el cliente SSH.

1. Abra `~/.ssh/config`con un editor de texto. Si este archivo no existe, puede crearlo si escribe `touch ~/.ssh/config` en la línea de comandos.

2. Agregue al archivo `config` el texto siguiente.

    ```
    Host <edgenodename>.<clustername>-ssh.azurehdinsight.net
        ForwardAgent yes
    ```

    Reemplace la información de __Host__ con la dirección del nodo al que se conecta a través de SSH. El ejemplo anterior utiliza el nodo perimetral. Esta entrada configura el reenvío del agente SSH para el nodo especificado.

3. Use el siguiente comando desde el terminal para probar el desvío del agente SSH:

    ```bash
    echo "$SSH_AUTH_SOCK"
    ```

    Este comando devuelve información similar al texto siguiente:

    ```output
    /tmp/ssh-rfSUL1ldCldQ/agent.1792
    ```

    Si no se devuelve nada, `ssh-agent` no se está ejecutando. Consulte la información de los scripts de inicio del agente en [Using ssh-agent with ssh http://mah.everybody.org/docs/ssh)](http://mah.everybody.org/docs/ssh) (Uso de ssh-agent con ssh) o la documentación del cliente SSH para más información.

4. Una vez que haya comprobado que **ssh-agent** está en ejecución, use lo siguiente para agregar la clave privada SSH al agente:

    ```bash
    ssh-add ~/.ssh/id_rsa
    ```

    Si la clave privada está almacenada en un archivo distinto, reemplace `~/.ssh/id_rsa` por la ruta de acceso al archivo.

5. Conecte con el nodo perimetral o nodos principales del clúster usando SSH. A continuación, use el comando SSH para conectarse a un nodo de trabajo o de zookeeper. La conexión se establece mediante la clave reenviada.

## <a name="next-steps"></a>Pasos siguientes

* [Uso de la tunelización de SSH con HDInsight](hdinsight-linux-ambari-ssh-tunnel.md)
* [Uso de nodos perimetrales en HDInsight](hdinsight-apps-use-edge-node.md#access-an-edge-node)
* [Uso de SCP con HDInsight](./use-scp.md)