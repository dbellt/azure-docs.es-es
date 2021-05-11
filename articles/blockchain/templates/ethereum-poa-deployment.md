---
title: Implementación de la solución Consorcio de prueba de autoridad de Ethereum en Azure
description: Usar la solución del consorcio de prueba de autoridad de Ethereum para implementar y configurar una red con varios miembros del consorcio Ethereum en Azure
ms.date: 03/01/2021
ms.topic: how-to
ms.reviewer: ravastra
ms.custom: contperf-fy21q3
ms.openlocfilehash: 0ced84afc47c188dc8f5d911794a90cdbeeebd81
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108164928"
---
# <a name="deploy-ethereum-proof-of-authority-consortium-solution-template-on-azure"></a>Implementación de la solución Consorcio de prueba de autoridad de Ethereum en Azure

Puede utilizar [la plantilla de solución de Azure Consorcio de prueba de autoridad de Ethereum (versión preliminar)](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-azure-blockchain.azure-blockchain-ethereum) para fácil implementar, configurar y gobernar una red con varios miembros del consorcio de prueba de autoridad de Ethereum con un conocimiento mínimo de Azure y Ethereum.

Todos los miembros del consorcio pueden usar la plantilla de solución para aprovisionar una superficie de red de cadena de bloques mediante los servicios de proceso, redes y almacenamiento de Microsoft Azure. La superficie de red de cada miembro del consorcio se compone de un conjunto de nodos de validador de carga equilibrada con el que una aplicación o un usuario pueden interactuar para enviar las transacciones de Ethereum.

[!INCLUDE [Preview note](./includes/preview.md)]

## <a name="choose-an-azure-blockchain-solution"></a>Elegir una solución de Azure Blockchain

Antes de optar por usar la plantilla de solución Consorcio de prueba de autoridad de Ethereum, compare su escenario con los casos de uso comunes de las opciones de Azure Blockchain disponibles.

> [!IMPORTANT]
> Considere la posibilidad de usar [Azure Blockchain Service](../service/overview.md) en lugar de Ethereum en la plantilla de solución de Azure. Azure Blockchain Service es un servicio de Azure administrado compatible. Parity Ethereum realizó la transición a mantenimiento y desarrollo controlados por la comunidad. Para obtener más información, vea [Transición de Parity Ethereum a OpenEthereum DAO](https://www.parity.io/parity-ethereum-openethereum-dao/).

Opción | Modelo de servicio | Caso de uso común
-------|---------------|-----------------
Plantillas de solución | IaaS | Las plantillas de solución son plantillas de Azure Resource Manager que puede usar para aprovisionar una topología de red de cadena de bloques totalmente configurada. Las plantillas implementan y configuran servicios de proceso, redes y almacenamiento de Microsoft Azure para un tipo de red de cadena de bloques determinado. Las plantillas de solución se proporcionan sin un contrato de nivel de servicio. Use la [página de preguntas y respuestas de Microsoft](/answers/topics/azure-blockchain-workbench.html) para obtener soporte técnico.
[Azure Blockchain Service](../service/overview.md) | PaaS | Azure Blockchain Service Preview simplifica la formación, administración y regulación de las redes de cadena de bloques del consorcio. Use Azure Blockchain Service para soluciones que requieran PaaS, administración de consorcios o privacidad de contratos y transacciones.
[Azure Blockchain Workbench](../workbench/overview.md) | IaaS y PaaS | La versión preliminar de Azure Blockchain Workbench es una colección de servicios y funcionalidades de Azure diseñada para ayudarle a crear e implementar aplicaciones de cadena de bloques para compartir datos y procesos empresariales con otras organizaciones. Use Azure Blockchain Workbench para crear un prototipo de una solución de cadena de bloques o una prueba de concepto de la aplicación de cadena de bloques. Azure Blockchain Workbench se proporciona sin un acuerdo de nivel de servicio. Use la [página de preguntas y respuestas de Microsoft](/answers/topics/azure-blockchain-workbench.html) para obtener soporte técnico.

## <a name="solution-architecture"></a>Arquitectura de la solución

La plantilla de solución de Ethereum permite implementar una red con varios miembros del consorcio de prueba de autoridad de Ethereum basada en una o en varias regiones.

![arquitectura de implementación](./media/ethereum-poa-deployment/deployment-architecture.png)

Cada implementación de los miembros del consorcio incluye:

* Máquinas virtuales para ejecutar los validadores de prueba de autoridad
* Azure Load Balancer para distribuir solicitudes de RPC, de emparejamiento y de aplicaciones descentralizadas de gobernanza
* Azure Key Vault para proteger las identidades de validador
* Azure Storage para hospedar la información de red persistente y coordinar la concesión
* Azure Monitor para agregar registros y estadísticas de rendimiento
* Puerta de enlace de red virtual (opcional) para permitir conexiones VPN entre redes virtuales privadas

De forma predeterminada, se puede acceder a los puntos de conexión RPC y de emparejamiento a través de una dirección IP pública para habilitar la conectividad simplificada entre suscripciones y nubes. Puede utilizar los [contratos de permisos de Parity](https://openethereum.github.io/Permissioning.html) para controles de acceso de nivel de aplicación. Se admiten las redes que se ha implementado detrás de VPN, que aprovechan las puertas de enlace de red virtual para conectividad entre suscripciones. Dado que las implementaciones de VPN y de red virtual son más complejas, puede que quiera empezar con un modelo de dirección IP pública al crear un prototipo de una solución.

Los contenedores de Docker se usan por su confiabilidad y modularidad. Azure Container Registry se usa para hospedar y proporcionar imágenes con versiones como parte de cada implementación. Las imágenes de contenedor constan de:

* Orquestador: genera identidades y contratos de gobernanza. Almacena identidades en un almacén de identidades.
* Cliente de paridad: concede la identidad del almacén de identidades. Detecta los pares y se conecta a ellos.
* Agente EthStats: recopila estadísticas y registros locales a través de RPC y envía la información a Azure Monitor.
* Aplicación descentralizada de gobernanza: interfaz web para interactuar con los contratos de gobernanza.

### <a name="validator-nodes"></a>Nodos de validación

En el protocolo de prueba de autoridad, los nodos de validador ocupan el lugar de los nodos de minero tradicionales. Cada validador tiene una identidad de Ethereum única que le permite participar en el proceso de creación de bloques. Para redundancia geográfica, cada miembro del consorcio puede aprovisionar dos o más nodos de validador en cinco regiones. Los nodos de validador se comunican con otros nodos de validador para llegar a un consenso sobre el estado del libro de contabilidad distribuida subyacente. Para garantizar la participación razonable en la red, se prohíbe a cada miembro del consorcio el uso de más validadores que el primer miembro de la red. Por ejemplo, si el primer miembro implementa tres validadores, cada miembro solo puede tener un máximo de tres validadores.

### <a name="identity-store"></a>Almacén de identidades

En la suscripción de cada miembro se implementa un almacén de identidades que contiene de forma segura las identidades de Ethereum generadas. Para cada validador, el contenedor de orquestación genera una clave privada de Ethereum y la almacena en Azure Key Vault.

## <a name="deploy-ethereum-consortium-network"></a>Implementación de la red del consorcio de Ethereum

En este tutorial, se supone que está creando una red del consorcio de Ethereum de varios participantes. El flujo siguiente es un ejemplo de implementación de varios participantes:

1. Tres miembros generan una cuenta de Ethereum mediante MetaMask
1. El *miembro A* implementa la prueba de autoridad de Ethereum y proporciona su dirección pública de Ethereum
1. El *miembro A* proporciona la dirección URL del consorcio al *miembro B* y al *miembro C*
1. El *miembro B* y el *miembro C* implementan la prueba de autoridad de Ethereum, proporcionando sus direcciones públicas de Ethereum y la dirección URL del consorcio del *miembro A*
1. El *miembro A* vota al *miembro B* como administrador
1. El *miembro A* y el *miembro B* votan al *miembro C* como administrador

En las secciones siguientes se muestra cómo configurar la superficie del primer miembro de la red.

### <a name="create-resource"></a>Crear el recurso

En la esquina superior derecha de [Azure Portal](https://portal.azure.com), seleccione **Crear un recurso**.

Seleccione **Cadena de bloques** > **Consorcio de prueba de autoridad de Ethereum (versión preliminar)** .

### <a name="basics"></a>Aspectos básicos

En **Aspectos básicos**, especifique los valores para los parámetros estándar de cualquier implementación.

![Aspectos básicos](./media/ethereum-poa-deployment/basic-blade.png)

Parámetro | Descripción | Valor de ejemplo
----------|-------------|--------------
Crear una nueva red o unirse a una existente | Puede crear una nueva red del consorcio o unirse a una red existente del consorcio. La unión a una red existente requiere parámetros adicionales. | Crear nuevo
Dirección de correo electrónico | Recibirá una notificación por correo electrónico con información sobre la implementación cuando esta se complete. | Una dirección de correo electrónico válida
Nombre de usuario de máquina virtual | Nombre de usuario del administrador de cada una de las máquinas virtuales que se ha implementado | 1-64 caracteres alfanuméricos
Tipo de autenticación | El método de autenticación en la máquina virtual. | Contraseña
Contraseña | La contraseña de la cuenta de administrador para cada una de las máquinas virtuales implementadas. Inicialmente, todas las VM tienen la misma contraseña. Puede cambiar la contraseña después del aprovisionamiento. | De 12 a 72 caracteres 
Suscripción | La suscripción en la que se va a implementar la red del consorcio. |
Grupo de recursos| El grupo de recursos en el que se va a implementar la red del consorcio. | myResourceGroup
Location | La región de Azure para el grupo de recursos. | Oeste de EE. UU. 2

Seleccione **Aceptar**.

### <a name="deployment-regions"></a>Regiones de implementación

En *Regiones de implementación*, especifique el número de regiones y las ubicaciones de cada una de ellas. Puede realizar la implementación en un máximo de cinco regiones. La primera región debe coincidir con la ubicación del grupo de recursos de la sección *Aspectos básicos*. Para las redes de desarrollo o prueba, puede usar una sola región por miembro. Para entornos de producción, realice la implementación en dos o más regiones para lograr una alta disponibilidad.

![Regiones de implementación](./media/ethereum-poa-deployment/deployment-regions.png)

Parámetro | Descripción | Valor de ejemplo
----------|-------------|--------------
Número de regiones|Número de regiones para implementar la red del consorcio| 2
Primera región | Primera región para implementar la red del consorcio | Oeste de EE. UU. 2
Segunda región | Segunda región para implementar la red del consorcio. Las regiones adicionales son visibles cuando el número de regiones es de dos o más. | Este de EE. UU. 2

Seleccione **Aceptar**.

### <a name="network-size-and-performance"></a>Tamaño de red y rendimiento

En *Tamaño de red y rendimiento*, especifique las entradas para el tamaño de la red del consorcio. El tamaño de almacenamiento del nodo de validador determina el tamaño potencial de la cadena de bloques. El tamaño se puede cambiar después de la implementación.

![Tamaño de red y rendimiento](./media/ethereum-poa-deployment/network-size-and-performance.png)

Parámetro | Descripción | Valor de ejemplo
----------|-------------|--------------
Number of load balanced validator nodes (Número de nodos de validador con equilibrio de carga) | Número de nodos de validación que se aprovisionarán como parte de la red. | 2
Validator node storage performance (Rendimiento del almacenamiento de los nodos de validador) | Tipo de disco administrado para cada uno de los nodos de validación implementados. Para obtener más información sobre precios, consulte el artículo sobre los [precios de almacenamiento](https://azure.microsoft.com/pricing/details/managed-disks/). | SSD estándar
Validator node virtual machine size (Tamaño de la máquina virtual del nodo de validador) | Tamaño de la máquina virtual usada para los nodos de validador. Para obtener más información sobre precios, consulte el artículo sobre los [precios de máquinas virtuales](https://azure.microsoft.com/pricing/details/virtual-machines/windows/). | Standard D2 v3

La máquina virtual y la capa de almacenamiento afectan al rendimiento de la red.  Utilice la tabla siguiente como ayuda para elegir la opción más rentable:

SKU de máquina virtual|Capa de almacenamiento|Price|Throughput|Latencia
---|---|---|---|---
F1|SSD estándar|low|low|high
D2_v3|SSD estándar|medio|medio|medio
F16s|SSD Premium|high|high|low

Seleccione **Aceptar**.

### <a name="ethereum-settings"></a>Configuración de Ethereum

En *Configuración de Ethereum*, especifique las opciones de configuración relacionadas con Ethereum.

![Configuración de Ethereum](./media/ethereum-poa-deployment/ethereum-settings.png)

Parámetro | Descripción | Valor de ejemplo
----------|-------------|--------------
Consortium Member ID (Identificador de miembro del consorcio) | Identificador asociado con cada miembro que participa en la red del consorcio. Se usa para configurar espacios de direcciones IP para evitar colisiones. En el caso de una red privada, el identificador de miembro debe ser único en las diferentes organizaciones de la misma red.  Es necesario un identificador de miembro único incluso si la misma organización se implementa en varias regiones. Tome nota del valor de este parámetro, ya que debe compartirlo con otros miembros que se unan para garantizar que no haya colisiones. El intervalo válido es de 0 a 255. | 0
Id. de red | El identificador de red para la red del consorcio de Ethereum que se está implementando. Cada red de Ethereum tiene su propio identificador de red, siendo 1 el identificador de la red pública. El intervalo válido es de 5 a 999 999 999. | 10101010
Admin Ethereum Address (Dirección de Ethereum del administrador) | Dirección de cuenta de Ethereum que se usa para participar en la gobernanza de prueba de autoridad. Puede usar MetaMask para generar una dirección de Ethereum. |
Opciones avanzadas | Opciones avanzadas de configuración de Ethereum | Habilitar
Implementación mediante la dirección IP pública | Si se selecciona Red virtual privada, la red se implementa detrás de una puerta de enlace de red virtual y se quita el acceso de emparejamiento. Para la red virtual privada, todos los miembros deben usar una puerta de enlace de red virtual para que la conexión sea compatible. | Dirección IP pública
Límite de gas del bloque | Límite de gas del bloque inicial de la red. | 50000000
Período de resellado del bloque (s) | La frecuencia con que se crearán bloques vacíos cuando no hay ninguna transacción en la red. Una frecuencia mayor finalizará más rápido, pero los costos de almacenamiento serán superiores. | 15
Contrato de permisos de la transacción | Código de bytes para el contrato de permisos de la transacción. Restringe la implementación de contrato inteligente y la ejecución a una lista permitida de cuentas de Ethereum. |

Seleccione **Aceptar**.

### <a name="monitoring"></a>Supervisión

La supervisión le permite configurar un recurso de registro para la red. El agente de supervisión recopila y expone métricas y registros útiles de la red, lo que proporciona la capacidad de comprobar rápidamente el estado de la red o los problemas de depuración.

![Azure Monitor](./media/ethereum-poa-deployment/azure-monitor.png)

Parámetro | Descripción | Valor de ejemplo
----------|-------------|--------------
Supervisión | Opción para habilitar la supervisión | Habilitar
Conectarse a registros de Azure Monitor existente | Opción para crear una instancia de registros de Azure Monitor o unirse a una instancia existente | Crear nuevo
Location | Región donde se implementa la instancia nueva | Este de EE. UU.
Identificador de área de trabajo de Log Analytics existente (Conectarse a registros de Azure Monitor existente = Unirse a existente)|Identificador del área de trabajo de la instancia de registros de Azure Monitor existente|N/D
Clave principal de Log Analytics existente (Conectarse a registros de Azure Monitor existente = Unirse a existente)|Clave principal que se usa para conectarse a la instancia de registros de Azure Monitor existente|N/D

Seleccione **Aceptar**.

### <a name="summary"></a>Resumen

Haga clic en el resumen para revisar las entradas especificadas y ejecutar una validación básica anterior a la implementación. Antes de realizar la implementación, puede descargar la plantilla y los parámetros.

Seleccione **Crear** para realizar la implementación.

Si la implementación incluye puertas de enlace de red virtual, puede tardar entre 45 y 50 minutos.

## <a name="deployment-output"></a>Salida de la implementación

Al finalizar la implementación, puedes acceder a los parámetros necesarios a través de Azure Portal.

### <a name="confirmation-email"></a>Correo electrónico de confirmación

Si proporciona una dirección de correo electrónico ([sección Aspectos básicos](#basics)), se envía un correo electrónico que incluye la información de implementación y vínculos a esta documentación.

![correo electrónico de implementación](./media/ethereum-poa-deployment/deployment-email.png)

### <a name="portal"></a>Portal

Una vez que la implementación finaliza correctamente y todos los recursos se han aprovisionado, puede ver los parámetros de salida en el grupo de recursos.

1. Vaya al grupo de recursos en el portal.
1. Seleccione **Información general > Implementaciones**.

    ![Información general del grupo de recursos](./media/ethereum-poa-deployment/resource-group-overview.png)

1. Seleccione la implementación **microsoft-azure-blockchain.azure-blockchain-ether-...** .
1. Seleccione la sección **Salidas**.

    ![Salidas de implementación](./media/ethereum-poa-deployment/deployment-outputs.png)

## <a name="growing-the-consortium"></a>Crecimiento del consorcio

Para expandir el consorcio, primero debe conectar la red física. Si realiza la implementación detrás de una VPN, vea la sección [Conexión de puertas de enlace de red virtual](#connecting-vnet-gateways) para configurar la conexión de red como parte de la implementación del nuevo miembro. Una vez finalizada la implementación, use la [aplicación descentralizada de gobernanza](#governance-dapp) para convertirse en administrador de red.

### <a name="new-member-deployment"></a>Implementación de un nuevo miembro

Comparta la siguiente información con el miembro que se une. Esta información se encuentra en el correo electrónico posterior a la implementación o en la salida de implementación del portal.

* Dirección URL de datos del consorcio
* Número de nodos que ha implementado
* Identificador del recurso de puerta de enlace de red virtual (si se usa VPN)

El miembro que realiza la implementación debe usar la misma plantilla de solución del consorcio de prueba de autoridad de Ethereum al implementar su presencia de red mediante la siguiente guía:

* Seleccione **Unirse a existente**
* Elija el mismo número de nodos de validación que el resto de los miembros de la red para garantizar una representación razonable.
* Utilice la misma dirección de Ethereum de administrador.
* Use la *dirección URL de datos del consorcio* proporcionada en *Configuración de Ethereum*.
* Si el resto de la red está detrás de una VPN, seleccione **Red virtual privada** bajo la sección avanzada.

### <a name="connecting-vnet-gateways"></a>Conexión de puertas de enlace de red virtual

Esta sección solo es necesaria si se ha realizado la implementación con una red virtual privada. Puede omitir esta sección si usa direcciones IP públicas.

En el caso de una red privada, los diferentes miembros se conectan a través de conexiones de puerta de enlace de red virtual. Antes de que un miembro pueda unirse a la red y ver el tráfico de transacción, un miembro existente debe realizar una configuración final en su puerta de enlace de VPN para aceptar la conexión. Los nodos de Ethereum del miembro que se está uniendo no se ejecutarán hasta que se establezca una conexión. Cree conexiones de red redundantes en el consorcio para reducir las posibilidades de un único punto de error.

Después de que el nuevo miembro se implemente, el miembro existente debe completar la conexión bidireccional configurando una conexión de puerta de enlace de red virtual con el nuevo miembro. El miembro existente necesita:

* El identificador del recurso de la puerta de enlace de red virtual del miembro que se conecta. Consulte [Salida de la implementación](#deployment-output).
* La clave de conexión compartida.

El miembro existente debe ejecutar el siguiente script de PowerShell para completar la conexión. Puede usar Azure Cloud Shell, que se encuentra en la barra de navegación superior derecha del portal.

![cloud shell](./media/ethereum-poa-deployment/cloud-shell.png)

```powershell
$MyGatewayResourceId = "<EXISTING_MEMBER_RESOURCEID>"
$OtherGatewayResourceId = "<NEW_MEMBER_RESOURCEID]"
$ConnectionName = "Leader2Member"
$SharedKey = "<NEW_MEMBER_KEY>"

## $myGatewayResourceId tells me what subscription I am in, what ResourceGroup and the VNetGatewayName
$splitValue = $MyGatewayResourceId.Split('/')
$MySubscriptionid = $splitValue[2]
$MyResourceGroup = $splitValue[4]
$MyGatewayName = $splitValue[8]

## $otherGatewayResourceid tells me what the subscription and VNet GatewayName are
$OtherGatewayName = $OtherGatewayResourceId.Split('/')[8]
$Subscription=Select-AzSubscription -SubscriptionId $MySubscriptionid

## create a PSVirtualNetworkGateway instance for the gateway I want to connect to
$OtherGateway=New-Object Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
$OtherGateway.Name = $OtherGatewayName
$OtherGateway.Id = $OtherGatewayResourceId
$OtherGateway.GatewayType = "Vpn"
$OtherGateway.VpnType = "RouteBased"

## get a PSVirtualNetworkGateway instance for my gateway
$MyGateway = Get-AzVirtualNetworkGateway -Name $MyGatewayName -ResourceGroupName $MyResourceGroup

## create the connection
New-AzVirtualNetworkGatewayConnection -Name $ConnectionName -ResourceGroupName $MyResourceGroup -VirtualNetworkGateway1 $MyGateway -VirtualNetworkGateway2 $OtherGateway -Location $MyGateway.Location -ConnectionType Vnet2Vnet -SharedKey $SharedKey -EnableBgp $True
```

## <a name="governance-dapp"></a>Aplicación descentralizada de gobernanza

En el corazón de la prueba de autoridad está la gobernanza descentralizado. Puesto que la prueba de autoridad se basa en una lista permitida de entidades de red para que la red se mantenga en estado correcto, es importante proporcionar un mecanismo razonable para realizar modificaciones en esta lista de permisos. Cada implementación incluye un conjunto de contratos inteligentes y un portal para la gobernanza en cadena de esta lista permitida. Una vez que un cambio propuesto alcanza el voto mayoritario de los miembros del consorcio, se realiza dicho cambio. Votar permite agregar nuevos participantes de consenso o quitar participantes comprometidos de una forma transparente que fomenta una red honesta.

La aplicación descentralizada de gobernanza es un conjunto de [contratos inteligentes](https://github.com/Azure-Samples/blockchain/tree/master/ledger/template/ethereum-on-azure/permissioning-contracts) implementados previamente y una aplicación web que se usan para gobernar las autoridades en la red. Las autoridades se dividen en identidades de administrador y nodos de validación.
Los administradores tienen la capacidad de delegar la participación de consenso en un conjunto de nodos de validación. Los administradores también pueden votar otros administradores dentro o fuera de la red.

![Aplicación descentralizada de gobernanza](./media/ethereum-poa-deployment/governance-dapp.png)

* **Gobernanza descentralizada:** los cambios en las autoridades de la red se administran mediante la votación en cadena por parte de administradores seleccionados.
* **Delegación de validador:** las autoridades pueden administrar sus nodos de validación que se configuran en cada implementación de prueba de autoridad.
* **Historial de cambios auditable:** cada cambio se registra en la cadena de bloques, lo que proporciona transparencia y capacidad de auditoría.

### <a name="getting-started-with-governance"></a>Introducción a la gobernanza

Para realizar cualquier tipo de transacción con la aplicación descentralizada de gobernanza, debe usar una cartera de Ethereum. La estrategia más sencilla es usar una cartera en el explorador, como [MetaMask](https://metamask.io); pero dado que estos contratos inteligentes están implementados en la red, también puede automatizar sus interacciones con el contrato de gobernanza.

Después de instalar MetaMask, vaya a la aplicación descentralizada de gobernanza en el explorador.  Puede buscar la dirección URL a través de Azure Portal en la salida de implementación.  Si no tiene ninguna cartera instalada en el explorador, no podrá realizar ninguna acción, pero podrá ver el estado del administrador.

### <a name="becoming-an-admin"></a>Convertirse en administrador

Si es el primer miembro que se implementa en la red, se convertirá automáticamente en administrador y sus nodos de paridad se mostrarán como validadores. Si se une a la red, necesita el voto de una mayoría del conjunto de administración existente (superior al 50 %) para ser administrador. Si opta por no ser administrador, sus nodos se siguen sincronizando y validan la cadena de bloques, pero no participan en el proceso de creación de bloques. Para iniciar el proceso de votación para convertirse en administrador, haga clic en **Designar** y escriba su alias y dirección de Ethereum.

![Designar](./media/ethereum-poa-deployment/governance-dapp-nominate.png)

### <a name="candidates"></a>Candidatos

Al seleccionar la pestaña **Candidatos**, se le muestra el conjunto actual de los candidatos a administrador.  Cuando un candidato alcanza una mayoría de los votos de los administradores actuales, pasa a ser administrador.  Para votar un candidato, seleccione la fila y elija **Votar**. Si cambia de opinión sobre un voto, seleccione el candidato y elija **Rescindir voto**.

![Candidatos](./media/ethereum-poa-deployment/governance-dapp-candidates.png)

### <a name="admins"></a>Administradores

En la pestaña **Administradores** puede ver el conjunto actual de administradores y votar en contra.  Si un administrador pierde más del 50 % de los respaldos, deja de ser administrador de la red. Los nodos de validación que posee este administrador pierden el estado de validador y se convierten en nodos de transacción en la red. Un administrador puede eliminarse por diferentes motivos, pero el consorcio debe tener una directiva acordada de antemano.

![Administradores](./media/ethereum-poa-deployment/governance-dapp-admins.png)

### <a name="validators"></a>Validadores

Al seleccionar la pestaña **Validadores** se muestran los nodos de paridad implementados actualmente para esta instancia y su estado actual (tipo de nodo). Cada miembro del consorcio tiene un conjunto diferente de validadores en esta lista, puesto que esta vista representa al miembro del consorcio implementado actualmente. Si se trata de una instancia recién implementada y aún no ha agregado los validadores, se muestra la opción para **agregar validadores**. Al agregar validadores, se elige automáticamente un conjunto equilibrado por regiones de nodos de paridad y se asignan a su conjunto de validadores. Si ha implementado más nodos de la capacidad permitida, los nodos restantes se convierten en nodos de transacción en la red.

La dirección de cada validador se asigna automáticamente a través del [almacén de identidades](#identity-store) de Azure.  Si un nodo deja de funcionar, renuncia a su identidad, lo que permite que otro nodo de la implementación ocupe su lugar. Este proceso garantiza que su participación consensuada tenga alta disponibilidad.

![Validadores](./media/ethereum-poa-deployment/governance-dapp-validators.png)

### <a name="consortium-name"></a>Nombre del consorcio

Cualquier administrador puede actualizar el nombre del consorcio.  Seleccione el icono de engranaje situado en la esquina superior izquierda para modificar el nombre del consorcio.

### <a name="account-menu"></a>Menú de la cuenta

En la parte superior derecha se encuentra el icono de identidad y el alias de la cuenta de Ethereum.  Los administradores pueden modificar su alias.

![Cuenta](./media/ethereum-poa-deployment/governance-dapp-account.png)

## <a name="support-and-feedback"></a>Soporte y comentarios<a id="tutorials"></a>

Para leer noticias acerca de Azure Blockchain, visite el [blog de Azure Blockchain](https://azure.microsoft.com/blog/topics/blockchain/), que le permitirá mantenerse al día sobre las ofertas de servicio de Blockchain y le proporcionará información del equipo de ingeniería de Azure Blockchain.

Para proporcionar comentarios sobre el producto o solicitar nuevas características, publique o vote una idea a través del [Foro de comentarios de Azure para Blockchain](https://aka.ms/blockchainuservoice).

### <a name="community-support"></a>Soporte técnico de la comunidad

Interactúe con los ingenieros de Microsoft y con expertos de la comunidad de Azure Blockchain.

* [Página de preguntas y respuestas de Microsoft](/answers/topics/azure-blockchain-workbench.html). El soporte técnico para plantillas con cadena de bloques se limita a los problemas de implementación.
* [Comunidad tecnológica de Microsoft](https://techcommunity.microsoft.com/t5/Blockchain/bd-p/AzureBlockchain)
* [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-blockchain-workbench)

## <a name="next-steps"></a>Pasos siguientes

Para obtener más soluciones de Azure Blockchain, consulte la [documentación de Azure Blockchain](../index.yml).
