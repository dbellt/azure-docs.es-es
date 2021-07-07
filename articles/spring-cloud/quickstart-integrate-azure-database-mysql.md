---
title: 'Inicio rápido: Integración con Azure Database for MySQL'
description: Se explica cómo aprovisionar y preparar una instancia de Azure Database for MySQL y, luego, configurar Pet Clinic en Azure Spring Cloud para usarla como una base de datos persistente con un solo comando.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: quickstart
ms.date: 05/13/2021
ms.custom: devx-track-java, devx-track-azurecli
ms.openlocfilehash: e4a9dc4d01cb2ea0aea42f00e6ece436194ce498
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111965009"
---
# <a name="quickstart-integrate-azure-spring-cloud-with-azure-database-for-mysql"></a>Inicio rápido: Integración de Azure Spring Cloud con Azure Database for MySQL

Pet Clinic, como se implementa en la configuración predeterminada del [Inicio rápido: Compilación e implementación de aplicaciones en Azure Spring Cloud](quickstart-deploy-apps.md), usa una base de datos en memoria (HSQLDB) que se rellena con datos al inicio. En este inicio rápido se explica cómo aprovisionar y preparar una instancia de Azure Database for MySQL y, luego, configurar Pet Clinic en Azure Spring Cloud para usarla como una base de datos persistente con un solo comando.

## <a name="variables-preparation"></a>Preparación de las variables

Se usarán los valores siguientes. Para evitar errores, guárdelos en un archivo de texto o en variables de entorno.

```bash
export RESOURCE_GROUP=<resource-group-name> # customize this
export MYSQL_SERVER_NAME=<mysql-server-name> # customize this
export MYSQL_SERVER_FULL_NAME=${MYSQL_SERVER_NAME}.mysql.database.azure.com
export MYSQL_SERVER_ADMIN_NAME=<admin-name> # customize this
export MYSQL_SERVER_ADMIN_LOGIN_NAME=${MYSQL_SERVER_ADMIN_NAME}\@${MYSQL_SERVER_NAME}
export MYSQL_SERVER_ADMIN_PASSWORD=<password> # customize this
export MYSQL_DATABASE_NAME=petclinic
```

## <a name="prepare-an-azure-database-for-mysql-instance"></a>Preparación de una instancia de Azure Database for MySQL

1. Cree de un servidor de Azure Database for MySQL.

    ```azcli
    az mysql server create --resource-group ${RESOURCE_GROUP} \
        --name ${MYSQL_SERVER_NAME} \
        --admin-user ${MYSQL_SERVER_ADMIN_NAME} \
        --admin-password ${MYSQL_SERVER_ADMIN_PASSWORD} \
        --sku-name GP_Gen5_2 \
        --ssl-enforcement Disabled \
        --version 5.7
    ```

1. Permita el acceso desde recursos de Azure.

    ```azcli
    az mysql server firewall-rule create --name allAzureIPs \
     --server ${MYSQL_SERVER_NAME} \
     --resource-group ${RESOURCE_GROUP} \
     --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
    ```

1. Permita el acceso desde la máquina de desarrollo para las pruebas.

    ```azcli
    az mysql server firewall-rule create --name devMachine \
     --server ${MYSQL_SERVER_NAME} \
     --resource-group ${RESOURCE_GROUP} \
     --start-ip-address <ip-address-of-your-dev-machine> \
     --end-ip-address <ip-address-of-your-dev-machine>
    ```

1. Aumente el tiempo de expiración de la conexión.

    ```azcli
    az mysql server configuration set --name wait_timeout \
     --resource-group ${RESOURCE_GROUP} \
     --server ${MYSQL_SERVER_NAME} --value 2147483
    ```

1. Cree una base de datos en el servidor MySQL y establezca la configuración correspondiente.

    ```sql
    // SUBSTITUTE values
    mysql -u ${MYSQL_SERVER_ADMIN_LOGIN_NAME} \
     -h ${MYSQL_SERVER_FULL_NAME} -P 3306 -p
    
    Enter password:
    Welcome to the MySQL monitor.  Commands end with ; or \g.
    Your MySQL connection id is 64379
    Server version: 5.6.39.0 MySQL Community Server (GPL)
    
    Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.
    
    Oracle is a registered trademark of Oracle Corporation and/or its
    affiliates. Other names may be trademarks of their respective
    owners.
    
    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
    
    mysql> CREATE DATABASE petclinic;
    Query OK, 1 row affected (0.10 sec)
    
    mysql> CREATE USER 'root' IDENTIFIED BY 'petclinic';
    Query OK, 0 rows affected (0.11 sec)
    
    mysql> GRANT ALL PRIVILEGES ON petclinic.* TO 'root';
    Query OK, 0 rows affected (1.29 sec)
    
    mysql> CALL mysql.az_load_timezone();
    Query OK, 3179 rows affected, 1 warning (6.34 sec)
    
    mysql> SELECT name FROM mysql.time_zone_name;
    ...
    
    mysql> quit
    Bye
    ```

1. Establezca la zona horaria.

    ```azcli
    az mysql server configuration set --name time_zone \
     --resource-group ${RESOURCE_GROUP} \
     --server ${MYSQL_SERVER_NAME} --value "US/Pacific"
    ```

## <a name="update-apps-to-use-mysql-database"></a>Actualización de las aplicaciones para usar la base de datos MySQL

Para habilitar MySQL como base de datos para la aplicación de ejemplo, basta con actualizar la aplicación *customer-service* con el perfil activo MySQL y las credenciales de base de datos como variables de entorno.

```azcli
az spring-cloud app update --name customers-service \
    --jvm-options="-Xms2048m -Xmx2048m -Dspring.profiles.active=mysql" \
    --env MYSQL_SERVER_FULL_NAME=${MYSQL_SERVER_FULL_NAME} \
        MYSQL_DATABASE_NAME=${MYSQL_DATABASE_NAME} \
        MYSQL_SERVER_ADMIN_LOGIN_NAME=${MYSQL_SERVER_ADMIN_LOGIN_NAME} \
        MYSQL_SERVER_ADMIN_PASSWORD=${MYSQL_SERVER_ADMIN_PASSWORD}
```

## <a name="update-extra-apps"></a>Actualización de aplicaciones adicionales

```azcli
az spring-cloud app update --name api-gateway \
    --jvm-options="-Xms2048m -Xmx2048m -Dspring.profiles.active=mysql"
az spring-cloud app update --name admin-server \
    --jvm-options="-Xms2048m -Xmx2048m -Dspring.profiles.active=mysql"
az spring-cloud app update --name customers-service \
    --jvm-options="-Xms2048m -Xmx2048m -Dspring.profiles.active=mysql" \
    --env MYSQL_SERVER_FULL_NAME=${MYSQL_SERVER_FULL_NAME} \
        MYSQL_DATABASE_NAME=${MYSQL_DATABASE_NAME} \
        MYSQL_SERVER_ADMIN_LOGIN_NAME=${MYSQL_SERVER_ADMIN_LOGIN_NAME} \
        MYSQL_SERVER_ADMIN_PASSWORD=${MYSQL_SERVER_ADMIN_PASSWORD}
az spring-cloud app update --name vets-service \
    --jvm-options="-Xms2048m -Xmx2048m -Dspring.profiles.active=mysql" \
    --env MYSQL_SERVER_FULL_NAME=${MYSQL_SERVER_FULL_NAME} \
        MYSQL_DATABASE_NAME=${MYSQL_DATABASE_NAME} \
        MYSQL_SERVER_ADMIN_LOGIN_NAME=${MYSQL_SERVER_ADMIN_LOGIN_NAME} \
        MYSQL_SERVER_ADMIN_PASSWORD=${MYSQL_SERVER_ADMIN_PASSWORD}
az spring-cloud app update --name visits-service \
    --jvm-options="-Xms2048m -Xmx2048m -Dspring.profiles.active=mysql" \
    --env MYSQL_SERVER_FULL_NAME=${MYSQL_SERVER_FULL_NAME} \
        MYSQL_DATABASE_NAME=${MYSQL_DATABASE_NAME} \
        MYSQL_SERVER_ADMIN_LOGIN_NAME=${MYSQL_SERVER_ADMIN_LOGIN_NAME} \
        MYSQL_SERVER_ADMIN_PASSWORD=${MYSQL_SERVER_ADMIN_PASSWORD}
```

## <a name="next-steps"></a>Pasos siguientes
* [Enlace de una instancia de Azure Database for MySQL con una aplicación de Azure Spring Cloud](how-to-bind-mysql.md)
* [Uso de una identidad administrada para conectar Azure SQL Database a una aplicación de Azure Spring Cloud](./connect-managed-identity-to-azure-sql.md)