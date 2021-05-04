---
title: 'Tutorial: Incorporación de Azure Load Balancer a un conjunto de escalado existente de máquinas virtuales mediante Azure Portal'
description: En este tutorial, aprenderá a agregar un equilibrador de carga a un conjunto de escalado existente de máquinas virtuales mediante el Azure Portal.
author: asudbring
ms.author: allensu
ms.service: load-balancer
ms.topic: tutorial
ms.date: 4/21/2021
ms.custom: template-tutorial
ms.openlocfilehash: cb87efe8402d49463d10e25aa79c50fc3b9b4c6e
ms.sourcegitcommit: bd1a4e4df613ff24e954eb3876aebff533b317ae
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2021
ms.locfileid: "107931046"
---
# <a name="tutorial-add-azure-load-balancer-to-an-existing-virtual-machine-scale-set-using-the-azure-portal"></a>Tutorial: Incorporación de Azure Load Balancer a un conjunto de escalado existente de máquinas virtuales mediante Azure Portal

La necesidad puede surgir cuando una instancia de Azure Load Balancer no está asociada a un conjunto de escalado de máquinas virtuales. 

Es posible que tenga un conjunto de escalado de máquinas virtuales implementado con una instancia de Azure Load Balancer que requiera actualización.

Azure Portal se puede usar para agregar o actualizar una instancia de Azure Load Balancer asociada a un conjunto de escalado de máquinas virtuales.  

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Creación de una red virtual
> * Crear una puerta de enlace NAT para la conectividad saliente
> * Creación de una instancia de Azure Load Balancer con SKU estándar
> * Creación de un conjunto de escalado de máquinas virtuales sin un equilibrador de carga
> * Incorporación de Azure Load Balancer al conjunto de escalado de máquinas virtuales

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree su cuenta de forma gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="create-a-virtual-network"></a>Creación de una red virtual

En esta sección creará una red virtual para el conjunto de escalado y otros recursos que se usan en el tutorial.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

2. En el cuadro de búsqueda de la parte superior del portal, escriba **Red virtual**.

3. En los resultados de la búsqueda, seleccione **Redes virtuales**.

4. Seleccione **+ Create** (+ Crear).

5. En la pestaña **Conceptos básicos** de **Crear una red virtual**, escriba o seleccione la siguiente información:

    | Configuración | Value |
    | ------- | ------|
    | **Detalles del proyecto** |   |
    | Subscription | Seleccione su suscripción. |
    | Resource group | Seleccione **Crear nuevo**. </br> Escriba **TutorLBVMSS-rg** en **Nombre**. </br> Seleccione **Aceptar**. |
    | **Detalles de instancia** |   |
    | Nombre | Escriba **myVNet**. |
    | Region | Seleccione **(EE. UU.) Oeste de EE. UU. 2**. |

6. Seleccione la pestaña **Direcciones IP** o el botón **Next: IP Addresses** (Siguiente: direcciones IP) situado en la parte inferior de la página.

7. En la pestaña **Direcciones IP,** en **Nombre de subred**, seleccione **predeterminado**.

8. En el panel **Editar subred**, en **Nombre de subred**, escriba **myBackendSubnet**.

9. Seleccione **Guardar**.

10. Seleccione la pestaña **Seguridad** o el botón **Next: Security** (Siguiente: seguridad) situado en la parte inferior de la página.

11. En la pestaña **Seguridad**, en **BastionHost**, seleccione **Habilitar**.

12. Escriba o seleccione la siguiente información:

    | Configuración | Value |
    | ------- | ----- |
    | Nombre del bastión | Escriba **MyBastionHost**. |
    | Espacio de direcciones de AzureBastionSubnet | Escriba **10.1.1.0/27**. |
    | Dirección IP pública | Seleccione **Crear nuevo**. </br> Escriba **myBastionIP** en **Nombre**. |

13. Seleccione la pestaña **Revisar y crear** o el botón azul **Revisar y crear** de la parte inferior de la página.

14. Seleccione **Crear**.

## <a name="create-nat-gateway"></a>Creación de una instancia de NAT Gateway 

En esta sección creará una puerta de enlace NAT para la conectividad saliente de las máquinas virtuales.

1. En el cuadro de búsqueda que aparece en la parte superior del portal, escriba **Puerta de enlace NAT**.

2. Seleccione **Puertas de enlace NAT** en los resultados de búsqueda.

3. Seleccione **+ Create** (+ Crear).

4. En la pestaña **Conceptos básicos** de **Crear puerta de enlace de traducción de direcciones de red (NAT)** , escriba o seleccione la información siguiente:

    | Configuración | Value |
    | ------- | ----- |
    | **Detalles del proyecto** |   |
    | Subscription | Seleccione su suscripción. |
    | Resource group | Seleccione **TutorLBVMSS-rg**. |
    | **Detalles de instancia** |   |
    | Nombre de NAT Gateway | Escriba **myNATgateway**. |
    | Region | Seleccione **(EE. UU.) Oeste de EE. UU. 2**. |
    | Zona de disponibilidad | Seleccione **Ninguno**. |
    | Tiempo de espera de inactividad (minutos) | Escriba **15**. |

5. Seleccione la pestaña **Dirección IP de salida** o elija el botón **Next: Outbound IP** (Siguiente: Dirección IP de salida) situado en la parte inferior de la página.

6. Seleccione **Crear una dirección IP pública** junto a **Direcciones IP públicas** en la pestaña **Outbound IP** (Dirección IP de salida).

7. En **Nombre**, escriba **myPublicIP-nat**.

8. Seleccione **Aceptar**.

9. Seleccione la pestaña **Subred** o elija el botón **Next: Subnet** (Siguiente: Subred) situado en la parte inferior de la página.

10. Seleccione **myVNet** en el menú desplegable de **Red virtual**.

11. Active la casilla situada junto a **myBackendSubnet**.

12. Seleccione la pestaña **Revisar y crear** o el botón azul **Revisar y crear** en la parte inferior de la página.

13. Seleccione **Crear**.

## <a name="create-load-balancer"></a>Creación de un equilibrador de carga

En esta sección va a crear un equilibrador de carga para las máquinas virtuales.

1. En el cuadro de búsqueda que aparece en la parte superior del portal, escriba **Load Balancer**.

2. Seleccione **Equilibradores de carga** en los resultados de la búsqueda.

3. Seleccione **+ Create** (+ Crear).

4. En la pestaña **Conceptos básicos** de **Crear equilibrador de carga**, escriba o seleccione la siguiente información:

    | Configuración | Value |
    | ------- | ----- |
    | **Detalles del proyecto** |   |
    | Subscription | Seleccione su suscripción. |
    | Resource group | Seleccione **TutorLBVMSS-rg**. |
    | **Detalles de instancia** |   |
    | Nombre | Escriba **myLoadBalancer**. |
    | Region | Seleccione **(EE. UU.) Oeste de EE. UU. 2**. |
    | Tipo | Deje el valor predeterminado, **Públicas**. |
    | SKU | Deje el valor predeterminado **Estándar**. |
    | Nivel | Deje el valor predeterminado, **Regional**. |
    | **Dirección IP pública** |   |
    | Dirección IP pública | Deje el valor **Crear nuevo** predeterminado. |
    | Nombre de la dirección IP pública | Escriba **myPublicIP-lb**. |
    | Zona de disponibilidad | seleccione **Con redundancia de zona**. |
    | Adición de una dirección IPv6 pública | Deje el valor predeterminado de **No**. |
    | Preferencia de enrutamiento | Deje el valor predeterminado **Microsoft Network**. |

5. Seleccione la pestaña **Revisar y crear** o el botón azul **Revisar y crear** en la parte inferior de la página.

6. Seleccione **Crear**.

### <a name="configure-load-balancer-settings"></a>Configuración del equilibrador de carga

En esta sección creará un grupo de servidores back-end para **myLoadBalancer**.

Creará un sondeo de estado para supervisar **HTTP** y el **puerto 80**. El sondeo de estado supervisará el estado de las máquinas virtuales del grupo de servidores back-end. 

Creará una regla de equilibrio de carga para el **puerto 80** con SNAT de salida deshabilitado. La puerta de enlace NAT que creó anteriormente controlará la conectividad de salida de las máquinas virtuales.

1. En el cuadro de búsqueda que aparece en la parte superior del portal, escriba **Load Balancer**.

2. Seleccione **Equilibradores de carga** en los resultados de la búsqueda.

3. Seleccione **myLoadBalancer**.

4. En **myLoadBalancer**, seleccione **Grupos de back-end** en **Configuración**.

5. Seleccione **+ Agregar** en **Grupos de back-end**.

6. En **Agregar grupo back-end**, escriba o seleccione la siguiente información:

    | Configuración | Valor |
    | ------- | ----- |
    | Nombre | Escriba **myBackendPool**. |
    | Virtual network | Seleccione **myVNet**. |
    | Configuración del grupo de back-end | Deje el valor predeterminado, **NIC**. |
    | Versión de la dirección IP | Deje el valor predeterminado, **IPv4**. |

7. Seleccione **Agregar**.

8. Seleccione **Sondeos de estado**.

9. Seleccione **+Agregar**.

10. En **Agregar sondeo de estado**, escriba o seleccione la siguiente información:

    | Configuración | Valor |
    | ------- | ----- |
    | Nombre | Escriba **myHTTPProbe**. |
    | Protocolo | Seleccione **HTTP**. |
    | Port | Deje el valor predeterminado, **80**. |
    | Ruta de acceso | Deje el valor predeterminado, **/** . |
    | Intervalo | Deje el valor predeterminado, **5** segundos. |
    | Umbral incorrecto | Deje el valor predeterminado, **2** errores consecutivos. |

11. Seleccione **Agregar**.

12. Seleccione **Reglas de equilibrio de carga**. 

13. Seleccione **+Agregar**.

14. Escriba o seleccione la siguiente información para **Agregar rega de equilibrio de carga**:

    | Configuración | Valor |
    | ------- | ----- |
    | Nombre | Escriba **myHTTPRule**. |
    | Versión de la dirección IP | Deje el valor predeterminado, **IPv4**. |
    | Dirección IP del front-end | Seleccione **LoadBalancerFrontEnd**. |
    | Protocolo | Seleccione el valor predeterminado, **TCP**. |
    | Port | Escriba **80**. |
    | Puerto back-end | Escriba **80**. |
    | Grupo back-end | Seleccione **MyBackendPool**. |
    | Sondeo de mantenimiento | Seleccione **myHTTPProbe**. |
    | Persistencia de la sesión | Deje el valor predeterminado de **No**. |
    | Tiempo de espera de inactividad (minutos) | Cambie el control deslizante a **15**. |
    | Restablecimiento de TCP | Seleccione **Habilitado**. |
    | Dirección IP flotante | Deje el valor predeterminado, **Deshabilitado**. |
    | Traducción de direcciones de red de origen (SNAT) de salida | Deje el valor predeterminado, **(Recommended) Use outbound rules to provide backend pool members access to the internet** ([Recomendado] Usar reglas de salida para que los miembros del grupo de servidores de back-end puedan acceder a Internet). |

5. Seleccione **Agregar**.

## <a name="create-virtual-machine-scale-set"></a>Creación de un conjunto de escalado de máquinas virtuales

En esta sección creará un conjunto de escalado de máquinas virtuales sin un equilibrador de carga. Más adelante, agregará un equilibrador de carga a este conjunto de escalado en Azure Portal.

1. En el cuadro de búsqueda que aparece en la parte superior del portal, escriba **conjunto de escalado de máquinas virtuales**.

2. En los resultados de la búsqueda, seleccione **Conjunto de escalado de máquinas virtuales**.

3. Seleccione **+Agregar**.

4. En la pestaña **Conceptos básicos** de **Crear un conjunto de escalado de máquinas virtuales**, escriba o seleccione la siguiente información:

    | Parámetro | Value |
    | ------- | ----- |
    | **Detalles del proyecto** |   |
    | Subscription | Seleccione su suscripción. |
    | Resource group | Seleccione **TutorLBVMSS-rg**. |
    | **Detalles del conjunto de escalado** |   |
    | Nombre del conjunto de escalado de máquinas virtuales | Escriba **myVMScaleSet**. |
    | Region | Seleccione **(EE. UU.) Oeste de EE. UU. 2**. |
    | Zona de disponibilidad | Deje el valor predeterminado de **No**. |
    | **Orquestación** |   |
    | Modo de orquestación | Deje el valor predeterminado de **Uniform: optimized for large-scale stateless workloads with identical instances** (Uniforme: optimizado para cargas de trabajo sin estado a gran escala con instancias idénticas). |
    | **Detalles de instancia** |   |
    | Imagen | Seleccione **Windows Server 2019 Datacenter - Gen1**. |
    | Instancia de Azure Spot | Deje desactivado el valor predeterminado de esta casilla. |
    | Size | Seleccione un tamaño. |
    | **Cuenta de administrador** |
    | Nombre de usuario | Especifique un nombre de usuario. |
    | Contraseña | Escriba una contraseña. |
    | Confirmación de la contraseña | Confirme la contraseña. |

5. Seleccione la pestaña **Redes**.

6. En la pestaña **Redes**, escriba o seleccione la siguiente información:

    | Parámetro | Valor |
    | ------- | ----- |
    | **Configuración de redes virtuales** |   |
    | Virtual network | Seleccione **myVNet**. |

7. Seleccione la pestaña **Revisar y crear** o el botón azul **Revisar y crear** en la parte inferior de la página.

8. Seleccione **Crear**.

## <a name="add-load-balancer-to-scale-set"></a>Incorporación de un equilibrador de carga al conjunto de escalado

En esta sección, accederá al conjunto de escalado de la Azure Portal para agregarle un equilibrador de carga.

1. En el cuadro de búsqueda que aparece en la parte superior del portal, escriba **conjunto de escalado de máquinas virtuales**.

2. En los resultados de la búsqueda, seleccione **Conjunto de escalado de máquinas virtuales**.

3. Seleccione **myVMScaleSet**.

4. En la sección **Configuración** de **myVMScaleSet**, seleccione **Redes**.

5. Seleccione la pestaña **Equilibrio de carga** en la página **Información general** de la configuración de **redes** en **myVMScaleSet**.

    :::image type="content" source="./media/tutorial-add-lb-existing-scale-set-portal/load-balancing-tab.png" alt-text="Seleccione la pestaña Equilibrio de carga en Redes" border="true":::.

6. Seleccione el botón azul **Agregar el equilibrio de carga**.

7. En **Agregar equilibrio de carga**, escriba o seleccione la siguiente información:

    | Parámetro | Valor |
    | ------- | ----- |
    | Opciones de equilibrio de carga | Seleccione **Azure Load Balancer**. |
    | Seleccionar un equilibrador de carga | Seleccione **myLoadBalancer**. |
    | Grupo back-end | Seleccione **Usar existente**. |
    | Seleccionar un grupo de back-end | Seleccione **MyBackendPool**. |

8. Seleccione **Guardar**.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a seguir usando esta aplicación, elimine el equilibrador de carga y los recursos auxiliares mediante los siguientes pasos:

1. Escriba **Grupo de recursos** en el cuadro de búsqueda que se encuentra en la parte superior del portal.

2. Seleccione **Grupos de recursos** en los resultados de la búsqueda.

3. Seleccione **TutorLBVMSS-rg**.

4. En la página de información general de **TutorLBVMSS-rg**, seleccione **Eliminar grupo de recursos**.

5. Escriba **TutorLBVMSS-rg** en **TYPE THE RESOURCE GROUP NAME** (ESCRIBIR EL NOMBRE DEL GRUPO DE RECURSOS).

6. Seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha:

* Creado una red virtual y un host de Azure Bastion.
* Creado una instancia de Azure Standard Load Balancer.
* Creado un conjunto de escalado de máquinas virtuales.
* Agregado el equilibrador de carga al conjunto de escalado de máquinas virtuales.

Pase al siguiente artículo para aprender a crear una instancia de Azure Load Balancer entre regiones:
> [!div class="nextstepaction"]
> [Creación de un equilibrador de carga entre regiones](tutorial-cross-region-portal.md)