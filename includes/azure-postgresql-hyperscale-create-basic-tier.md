---
title: archivo de inclusión
description: archivo de inclusión
author: jonels-msft
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: include
ms.date: 04/07/2021
ms.author: jonels
ms.custom: include file
ms.openlocfilehash: 09eb6d9483268314febd9478a551595a059d973b
ms.sourcegitcommit: 6ed3928efe4734513bad388737dd6d27c4c602fd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107023913"
---
Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en [Azure Portal](https://portal.azure.com).


Para crear un servidor de Azure Database for PostgreSQL, siga estos pasos:
1. Haga clic en **Crear un recurso** de la esquina superior izquierda de Azure Portal.
2. En la página **Nuevo**, seleccione **Bases de datos** y, en la página **Bases de datos**, seleccione **Azure Database for PostgreSQL**.
3. En la opción de implementación, haga clic en el botón **Crear** en el grupo de servidores de **Hiperescala (Citus).**
4. Rellene el formulario del nuevo servidor con la siguiente información:
   - Grupo de recursos: haga clic en el vínculo **Crear nuevo** que está debajo de este cuadro de texto para este campo. Escriba un nombre como **myresourcegroup**.
   - Nombre del grupo de servidores: escriba un nombre exclusivo para el nuevo grupo de servidores, que también se usará para un subdominio de servidor.
   - Active la casilla **Habilitar características de vista previa**.
   - Nombre de usuario de administrador: actualmente, se necesita el valor **citus** y no se puede cambiar.
   - Contraseña: tiene que tener al menos ocho caracteres de las tres categorías siguientes: letras mayúsculas del alfabeto inglés, letras minúsculas del alfabeto inglés, números (0-9) y caracteres no alfanuméricos (!, $, # o %, entre otros)
   - Ubicación: use la ubicación más cercana a los usuarios para que puedan acceder de la forma más rápida posible a los datos.

   > [!IMPORTANT]
   > Se precisa la contraseña de administrador del servidor que especifique aquí para iniciar sesión en el servidor y sus bases de datos. Recuerde o grabe esta información para su uso posterior.

5. Haga clic en **Configurar grupo de servidores**.
   - Seleccione el botón de radio **Básico** para el campo **Nivel**.
   - Haga clic en **Save**(Guardar).
6. Haga clic en **Siguiente: Redes >** en la parte inferior de la pantalla.

7. En la pestaña **Redes**, haga clic en el botón de radio **Punto de conexión público**.
   ![Punto de conexión público seleccionado](./media/azure-postgresql-hyperscale-create-db/network-public-endpoint.png)
8. Haga clic en el vínculo **+ Agregar dirección IP del cliente actual**.
   ![Dirección IP del cliente agregada](./media/azure-postgresql-hyperscale-create-db/network-add-client-ip.png)

   > [!NOTE]
   > El servidor Azure PostgreSQL se comunica a través de puerto 5432. Si intenta conectarse desde una red corporativa, es posible que el firewall de la red no permita el tráfico saliente a través del puerto 5432. En este caso, no se podrá conectar al clúster de Hiperscala (Citus), salvo que el departamento de TI abra el puerto 5432.
   >

9. Haga clic en **Revisar y crear** y luego en **Crear** para aprovisionar el servidor. El aprovisionamiento tarda unos minutos.
10. La página irá a la supervisión de la implementación. Cuando el estado activo cambia de **La implementación está en curso** a **Se completó la implementación**, haga clic en el elemento de menú **Salidas** que se encuentra a la izquierda de la página.
11. La página de resultados incluirá un nombre de host de coordinación junto a un botón para copiar el valor en el Portapapeles. Anote esta información para usarla más adelante.

## <a name="connect-to-the-database-using-psql"></a>Conexión a la base de datos mediante psql

Al crear el servidor de Azure Database for PostgreSQL, también se crea la base de datos predeterminada denominada **citus**. Para conectarse al servidor de bases de datos, necesita una cadena de conexión y la contraseña de administrador.

1. Obtenga la cadena de conexión. En la página del grupo de servidores, haga clic en el elemento de menú **Cadenas de conexión**. (Se encuentra en **Configuración**). Busque la cadena marcada como **psql**. Tendrá el formato:

   ```
   psql "host=hostname.postgres.database.azure.com port=5432 dbname=citus user=citus password={your_password} sslmode=require"
   ```

   Copie la cadena. Deberá reemplazar "{su\_contraseña}" por la contraseña administrativa que eligió anteriormente. El sistema no almacena la contraseña de texto no cifrado y, por lo tanto, no puede mostrarla en la cadena de conexión.

2. Abra una ventana del terminal en el equipo local.

3. En el símbolo de sistema, conéctese al servidor de Azure Database for PostgreSQL con la utilidad [psql](https://www.postgresql.org/docs/current/app-psql.html). Pase la cadena de conexión entre comillas, asegurándose de que contiene la contraseña:
   ```bash
   psql "host=..."
   ```

   Por ejemplo, el siguiente comando se conecta al nodo de coordinación del grupo de servidores **mydemoserver**:

   ```bash
   psql "host=mydemoserver-c.postgres.database.azure.com port=5432 dbname=citus user=citus password={your_password} sslmode=require"
   ```
