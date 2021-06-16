---
title: Panel Clientes de los análisis del marketplace comercial de Microsoft del Centro de partners, Azure Marketplace y Microsoft AppSource
description: Obtenga detalles sobre cómo acceder a la información de los clientes, incluidas las tendencias de crecimiento, mediante el panel Clientes de los análisis de marketplace comercial.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
ms.date: 11/09/2020
author: sayantanroy83
ms.author: sroy
ms.openlocfilehash: 3d5016e1671bb52d4b819c10e7c52c12e5046b7e
ms.sourcegitcommit: bd65925eb409d0c516c48494c5b97960949aee05
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/06/2021
ms.locfileid: "111541167"
---
# <a name="customers-dashboard-in-commercial-marketplace-analytics"></a>Panel Clientes de los análisis de marketplace comercial

En este artículo se proporciona información sobre el panel Clientes del Centro de partners. En este panel se muestra información sobre los clientes, incluidas las tendencias de crecimiento, presentada en un formato gráfico y descargable

Para obtener acceso al panel Clientes en el Centro de partners, vaya a **Marketplace comercial** y seleccione **[Analizar](https://partner.microsoft.com/dashboard/commercial-marketplace/analytics/summary)**  > **Clientes**.

>[!NOTE]
> Para obtener definiciones detalladas de la terminología de análisis, consulte [Terminología de análisis y preguntas comunes de Marketplace comercial](./analytics-faq.md).

## <a name="customers-dashboard"></a>Panel Clientes

En el panel Clientes se muestran los datos de los clientes que han adquirido sus ofertas. Puede ver representaciones gráficas de los elementos siguientes:

- Tendencia de los clientes activos y perdidos
- Tendencia de crecimiento de los clientes, incluidos los clientes existentes, nuevos y perdidos
- Clientes por pedidos o uso
- Percentil de clientes 
- Tipo de cliente por pedidos y uso
- Clientes por geografía
- Tabla de detalles de clientes
- Filtros de la página de clientes

> [!NOTE]
> La latencia máxima entre la adquisición de clientes y la generación de informes en el Centro de partners es de 48 horas.

## <a name="elements-of-the-customers-dashboard"></a>Elementos del panel Clientes

En las secciones siguientes se describe cómo usar el panel Clientes y cómo leer los datos.

### <a name="month-range"></a>Intervalo mensual

Puede encontrar una selección del intervalo mensual en la esquina superior derecha de cada página. Puede personalizar la salida de los gráficos de la página **Clientes** seleccionando un intervalo mensual basado en los últimos 6 o 12 meses, o seleccionando un intervalo mensual personalizado con una duración máxima de 12 meses. El intervalo mensual predeterminado (período de cálculo) es de seis meses.

:::image type="content" source="./media/customer-dashboard/month-range-filters.png" alt-text="Muestra los filtros del mes en la página Clientes.":::

> [!NOTE]
> Todas las métricas de los widgets de visualización y los informes de exportación respetan el período de cálculo seleccionado por el usuario.

### <a name="active-and-churned-customers-trend"></a>Tendencia de los clientes activos y perdidos

En esta sección, encontrará la tendencia de crecimiento de los clientes durante el período de cálculo seleccionado. Las métricas y las tendencias de crecimiento se representan mediante un gráfico de líneas y muestran el valor de cada mes al mantener el puntero sobre la línea del gráfico. El valor de porcentaje que aparece debajo de **Clientes activos** del widget representa la cantidad de crecimiento durante el período de cálculo seleccionado. Por ejemplo, en la siguiente captura de pantalla se muestra un crecimiento del 0,92 % durante el período de cálculo seleccionado.

[![Muestra el widget Clientes en la página Clientes.](./media/customer-dashboard/customers-widget.png)](./media/customer-dashboard/customers-widget.png#lightbox)

Hay tres _tipos de clientes_: nuevos, existentes y perdidos.

- Un nuevo cliente ha adquirido una o varias de sus ofertas por primera vez en el mes seleccionado.
- Un cliente existente ha adquirido una o varias de sus ofertas antes del mes seleccionado.
- Un cliente perdido ha cancelado todas las ofertas previamente adquiridas. Los clientes perdidos se representan en el eje negativo del widget Tendencia.

### <a name="customer-growth-trend-including-existing-new-and-churned-customers"></a>Tendencia de crecimiento de los clientes, incluidos los clientes existentes, nuevos y perdidos

En esta sección, encontrará la tendencia y el recuento de todos los clientes, incluidos nuevos, existentes y perdidos, con una tendencia de crecimiento mes a mes.

- El gráfico de líneas representa los porcentajes de crecimiento globales del cliente.
- La columna de mes representa el número de clientes apilados por clientes nuevos, existentes y perdidos.
- El recuento de clientes perdidos se muestra en la dirección negativa del eje X.
- Puede seleccionar elementos específicos de la leyenda para mostrar vistas más detalladas. Por ejemplo, seleccione los nuevos clientes en la leyenda para mostrar solo los nuevos clientes.
- Al mantener el puntero sobre una columna del gráfico solo se muestran detalles de ese mes.

[![Muestra el widget Tendencia de los clientes en la página Clientes.](./media/customer-dashboard/customers-trend.png)](./media/customer-dashboard/customers-trend.png#lightbox)

### <a name="customers-by-ordersusage&quot;></a>Clientes por pedidos o uso

El gráfico **Clientes por pedidos o uso** tiene tres pestañas: Pedidos, Uso normalizado y Uso sin procesar. Seleccione la pestaña **Pedidos** para mostrar detalles de pedido.

:::image type=&quot;content&quot; source=&quot;./media/customer-dashboard/customers-by-orders-usage.png&quot; alt-text=&quot;Muestra la pestaña Pedidos del widget Clientes por pedidos y uso en la página Clientes.&quot;:::

Tenga en cuenta lo siguiente:

- En la tabla de clasificación se presentan los detalles de los clientes clasificados por recuento de pedidos. Después de seleccionar un cliente, los detalles del cliente se presentan en las secciones “Detalles”, “Pedidos por SKU” y “SKU por puestos” adyacentes.
- Los detalles del perfil Cliente se muestran en este espacio cuando los editores inician sesión con un rol de propietario. Si los editores inician sesión con un rol de colaborador, los detalles de esta sección no estarán disponibles.
- En el gráfico de anillos **Pedidos por SKU** se muestra el desglose de los pedidos realizados de planes. Se muestran los cinco planes principales con el número de pedidos más alto, mientras que el resto de los pedidos se agrupan en **Rest all** (&quot;Todo el resto").
- En el gráfico de anillos **SKU por puestos** se muestra el desglose de los puestos pedidos de planes. Se muestran los cinco planes principales con el número de puestos más alto, mientras que el resto de los pedidos se agrupan en **Rest All** ("Todo el resto").

También puede seleccionar la tabla **Uso normalizado** o **Uso sin procesar** para ver los detalles de uso.

- En la tabla de clasificación se presentan los detalles de los clientes clasificados por recuento de horas de uso. Después de seleccionar un cliente, los detalles del cliente se presentan en las secciones “Detalles”, “Uso normalizado en función de las ofertas” y “Uso normalizado por tamaños de VM” adyacentes.
- Los detalles del perfil Cliente se muestran en este espacio cuando los editores inician sesión con un rol de propietario. Si los editores inician sesión con un rol de colaborador, los detalles de esta sección no estarán disponibles.
- En el gráfico de anillos **Uso normalizado en función de las ofertas** se muestra el desglose del uso consumido por las ofertas. Se muestran los cinco planes principales con el recuento de uso más alto, mientras que el resto de las ofertas se agrupan en **Rest All** ("Todo el resto").
- En el gráfico de anillos **Uso normalizado por tamaños de VM** se muestra el desglose del uso consumido por los distintos tamaños de VM. Se muestran los cinco tamaños de VM principales con el uso normalizado más alto, mientras que el resto del uso se agrupa en **Rest All** ("Todo el resto").

### <a name="top-customers-percentile"></a>Percentil de principales clientes

El gráfico **Percentil de principales clientes** tiene las tres pestañas siguientes: "Pedidos", "Uso normalizado" y "Uso sin procesar". El _percentil de principales clientes_ muestra el eje x, según lo determina el número de pedidos. El eje y muestra el número de pedidos del cliente. En el eje z secundario (gráfico de líneas) se muestra el porcentaje acumulado del número total de pedidos. Puede mostrar detalles manteniendo el puntero sobre los puntos del gráfico de líneas.

[![Muestra la pestaña Pedidos del widget Percentil de principales clientes en la página Clientes.](./media/customer-dashboard/top-customer-percentile.png)](./media/customer-dashboard/top-customer-percentile.png#lightbox)

### <a name="customer-type-by-orders-and-usage"></a>Tipo de cliente por pedidos y uso

En el gráfico **Orders/usage by customer type** (Pedidos/uso por tipo de cliente) se muestra el número de pedidos, el uso normalizado y las horas de uso sin procesar divididas entre nuevos clientes y clientes existentes. Estos se muestran respectivamente en dicho gráfico en función de la selección de pedidos y el uso normalizado y sin procesar.

Este gráfico muestra lo siguiente:

- Un nuevo cliente ha adquirido una o varias de sus ofertas o ha notificado el uso por primera vez en el mismo mes natural (eje y).
- Un cliente existente ha adquirido una de sus ofertas con anterioridad o ha notificado el uso antes del mes natural especificado (en el eje y).

[![Muestra la pestaña Pedidos del widget Pedidos por tipo de cliente en la página Clientes.](./media/customer-dashboard/orders-by-customer-type.png)](./media/customer-dashboard/orders-by-customer-type.png#lightbox)

### <a name="customers-by-geography"></a>Clientes por geografía

En el período de cálculo seleccionado, el mapa térmico muestra el número total de clientes y el porcentaje de los clientes recién agregados en una dimensión de geografía. El tono del color de claro a oscuro del mapa representa el número de clientes de bajo a alto. Seleccione un registro de la tabla para acercar un país o región.

[![Muestra la pestaña Pedidos del widget Pedidos por geografía en la página Clientes.](./media/customer-dashboard/customers-by-geography.png)](./media/customer-dashboard/customers-by-geography.png#lightbox)

Tenga en cuenta lo siguiente:

- Puede mover el mapa para ver la ubicación exacta.
- Puede hacer zoom en una ubicación específica.
- El mapa térmico tiene una cuadrícula complementaria para ver los detalles del recuento de clientes, recuento de pedidos y horas de uso normalizado en la ubicación específica.
- Puede buscar y seleccionar un país o región en la cuadrícula para ampliar la ubicación en el mapa. Vuelva a la vista original presionando el botón **Inicio** en el mapa.

### <a name="customer-details-table"></a>Tabla de detalles de clientes

En la tabla **Detalles del cliente** se muestra una lista numerada de los primeros 1000 clientes ordenados por la fecha en que adquirieron una de sus ofertas por primera vez. Puede expandir una sección seleccionando el icono de expansión en la cinta de detalles.

Tenga en cuenta lo siguiente:

- La información personal del cliente estará disponible solo si el cliente ha dado su consentimiento. Solo puede ver esta información si ha iniciado sesión con un nivel de rol de propietario de permisos.
- Cada columna de la cuadrícula se puede ordenar.
- Los datos se pueden extraer a un archivo .CSV o .TSV si el recuento de los registros es inferior a 1000.
- Si el número de registros es superior a 1000, los datos exportados se colocarán de forma asincrónica en una página de descargas durante los próximos 30 días.
- Aplique filtros a la tabla para mostrar solo los datos que le interesen. Filtre los datos por nombre de empresa, identificador de cliente, identificador de suscripción de Marketplace, tipo de licencia de Azure, fecha de adquisición, fecha de pérdida, correo electrónico del cliente, país/estado/ciudad/código postal del cliente, idioma del cliente, etc.
- Cuando un cliente protegido adquiere una oferta, la información de **Customer Detailed Data** (Datos detallados del cliente) se enmascara (************).
- Los detalles de la dimensión del cliente, como el nombre de la empresa, el nombre del cliente y el correo electrónico del cliente, están en el nivel de identificador de la organización, no en el nivel de transacción de Azure Marketplace o Microsoft AppSource.

_**Tabla 1: diccionario de términos de datos**_

| Nombre de la columna en la<br>interfaz de usuario | Nombre del atributo | Definición | Nombre de la columna en informes<br>de acceso mediante programación |
| ------------ | ------------- | ------------- | ------------- |
| Identificador de la suscripción a Marketplace | Identificador de la suscripción a Marketplace | Identificador único asociado a la suscripción a Azure que el cliente usó para comprar la oferta de Marketplace comercial. En el caso de las ofertas de infraestructura, este es el GUID de la suscripción de Azure del cliente. En el caso de las ofertas de SaaS, se muestra como ceros, ya que las compras de SaaS no requieren una suscripción a Azure. | MarketplaceSubscriptionId |
| DateAcquired | Date Acquired (Fecha de adquisición) | Primera fecha en que el cliente adquirió cualquier oferta publicada por usted. | DateAcquired |
| DateLost | Date Lost (Fecha de pérdida) | Última fecha en que el cliente canceló la última de todas las ofertas que había comprado anteriormente. | DateLost |
| Nombre del proveedor | Nombre del proveedor | Nombre del proveedor que participa en la relación entre Microsoft y el cliente. Si el cliente es una empresa a través de un revendedor, será el revendedor. Si participa un proveedor de soluciones en la nube (CSP), será el CSP. | ProviderName |
| Correo electrónico del proveedor | Correo electrónico del proveedor | Dirección de correo electrónico del proveedor que participa en la relación entre Microsoft y el cliente. Si el cliente es una empresa a través de un revendedor, será el revendedor. Si participa un proveedor de soluciones en la nube (CSP), será el CSP. | ProviderEmail |
| Nombre | Customer First Name (Nombre del cliente) | Nombre proporcionado por el cliente. El nombre podría ser diferente del nombre proporcionado en la suscripción a Azure del cliente. | Nombre |
| Apellidos | Customer Last Name (Apellidos del cliente) | Apellidos proporcionados por el cliente. El nombre podría ser diferente del nombre proporcionado en la suscripción a Azure del cliente. | Apellidos |
| Correo electrónico | Customer Email (Correo electrónico del cliente) | Dirección de correo electrónico proporcionada por el cliente final. El correo electrónico podría ser diferente de la dirección de correo electrónico de la suscripción a Azure del cliente. | Correo electrónico |
| Customer Company Name (Nombre de la empresa del cliente) | Customer Company Name (Nombre de la empresa del cliente) | Nombre de empresa proporcionado por el cliente. El nombre podría ser diferente de la ciudad de la suscripción a Azure del cliente. | CustomerCompany Name |
| Ciudad del cliente | Ciudad del cliente | Nombre de ciudad proporcionado por el cliente. La ciudad podría ser diferente de la ciudad de la suscripción a Azure del cliente. | Ciudad del cliente |
| Customer Postal Code (Código postal del cliente) | Customer Postal Code (Código postal del cliente) | Código postal proporcionado por el cliente. El código podría ser diferente del código postal proporcionado en la suscripción a Azure del cliente. | CustomerPostal Code |
| Cultura de comunicación del cliente | Customer Communication Language (Idioma de comunicación del cliente) | Idioma preferido por el cliente para la comunicación. | Cultura de comunicación del cliente |
| Región del país del cliente | País o región del cliente. | Nombre del país o región proporcionado por el cliente. Podría ser diferente del país o región de la suscripción a Azure del cliente. | Región del país del cliente |
| AzureLicenseType | Azure License Type (Tipo de licencia de Azure) | Tipo de contrato de licencia que los clientes utilizan para comprar Azure. También se conoce como _canal_. Los valores posibles son:<ul><li>Proveedor de soluciones en la nube</li><li>Enterprise</li><li>Enterprise a través de revendedor</li><li>Pago por uso</li></ul> | AzureLicenseType |
| PromotionalCustomers | Participación de contacto promocional | El valor le permitirá saber si el cliente ha optado de manera proactiva por el contacto promocional de los anunciantes. En este momento, no presentamos la opción a los clientes, por lo que hemos indicado "No" de forma general. Una vez implementada esta característica, comenzaremos a actualizarla en consecuencia. | PromotionalCustomers |
| CustomerState | Customer State (Estado del cliente) | Estado de residencia proporcionado por el cliente. El estado podría ser diferente del estado proporcionado en la suscripción a Azure del cliente. | CustomerState |
| CommerceRootCustomer | Cliente raíz comercial | Un identificador de cuenta de facturación se puede asociar a varios identificadores de cliente.<br>Una combinación de un identificador de cuenta de facturación y un identificador de cliente se puede asociar a varias suscripciones a marketplace comercial.<br>El cliente raíz comercial indica el nombre del cliente de la suscripción. | CommerceRootCustomer |
| Customer ID | Customer ID | Identificador único asignado a un cliente. Un cliente puede no tener ninguna suscripción a Azure Marketplace o bien tener varias. | CustomerId |
| Identificador de la cuenta de facturación | Identificador de la cuenta de facturación | Identificador de la cuenta en la que se genera la facturación. Asigne el **identificador de la cuenta de facturación** a **customerID** para conectar el informe de transacciones de pago con los informes de cliente, pedido y uso. | BillingAccountId |
| Tipo de cliente | Tipo de cliente | El valor de este campo indica el tipo de cliente. Los valores posibles son:<ul><li>individuo</li> <li>organization</li></ul> | CustomerType |
|||||

### <a name="customers-page-filters"></a>Filtros de la página de clientes

Los filtros de la página Clientes se aplican a nivel de la página Clientes. Puede seleccionar varios filtros para representar el gráfico para los criterios que quiera ver, así como los datos que quiera que se muestren en la cuadrícula o exportación "Datos detallados de los pedidos". Los filtros se aplican a los datos extraídos del intervalo mensual que haya seleccionado en la esquina superior derecha de la página Clientes.

> [!TIP]
> Puede usar el icono de descarga que está en la esquina superior derecha de cualquier widget para descargar los datos. Igualmente, para proporcionar comentarios sobre cada uno de los widgets, haga clic en el icono "pulgar hacia arriba" o "pulgar hacia abajo".

## <a name="next-steps"></a>Pasos siguientes

- Para obtener más información sobre los informes de análisis disponibles en marketplace comercial, consulte [Acceso a los informes de análisis de marketplace comercial en el Centro de partners](analytics.md).
- Para ver los gráficos, las tendencias y los valores de datos agregados que resumen la actividad de la oferta en el marketplace, consulte [Panel Resumen de los análisis de marketplace comercial](./summary-dashboard.md).
- Para información sobre los pedidos en un formato gráfico que se pueda descargar, consulte [Panel Pedidos de los análisis de marketplace comercial](./orders-dashboard.md).
- Para información sobre las métricas de uso y facturación de las ofertas de máquina virtual, consulte [Panel Uso de los análisis de marketplace comercial](./usage-dashboard.md).
- Para obtener una lista de las solicitudes de descarga de los últimos 30 días, consulte [Panel Descargas de los análisis de marketplace comercial](downloads-dashboard.md).
- Para obtener una vista consolidada con los comentarios de los clientes sobre las ofertas de Azure Marketplace y Microsoft AppSource, consulte [Panel de análisis de calificaciones y opiniones en el Centro de partners](ratings-reviews.md).
- Para ver las preguntas más frecuentes sobre los análisis de marketplace comercial y un diccionario completo de términos de datos, consulte [Preguntas más frecuentes y terminología del análisis de marketplace comercial](./analytics-faq.md).
