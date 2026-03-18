# 🧩 Configuración del Driver JDBC de MySQL para Spark

Este documento explica cómo descargar y configurar el driver JDBC necesario para conectar **Apache Spark** con **MySQL**.

---

## 📥 Paso 1: Descargar el JAR

```bash
cd ~/Downloads
wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-j-8.4.0.tar.gz
tar xzvf mysql-connector-j-8.4.0.tar.gz
```

---

## 📦 Paso 2: Moverlo a una ruta limpia (recomendado)

```bash
sudo mkdir -p /opt/jars
sudo cp mysql-connector-j-8.4.0/mysql-connector-j-8.4.0.jar /opt/jars/
```

---

## ⚙️ Paso 3: Definir la ruta en tu código

```python
MYSQL_JDBC_JAR = "/opt/jars/mysql-connector-j-8.4.0.jar"
```

---

## 🚨 Notas importantes

* Este archivo es **obligatorio** para que Spark pueda conectarse a MySQL vía JDBC.
* Si la ruta es incorrecta, verás errores como:

```text
java.lang.ClassNotFoundException: com.mysql.cj.jdbc.Driver
```

* Verifica que el archivo `.jar` exista exactamente en la ruta indicada:

```bash
ls /opt/jars/
```

---

## 💡 Tip profesional

Puedes evitar hardcodear la ruta usando una variable de entorno:

```bash
export MYSQL_JDBC_JAR=/opt/jars/mysql-connector-j-8.4.0.jar
```

Y en Python:

```python
import os

MYSQL_JDBC_JAR = os.getenv("MYSQL_JDBC_JAR")
```

---

## 🧠 Contexto

* **Hadoop** → almacenamiento (HDFS)
* **Spark** → procesamiento distribuido
* **JDBC Driver** → conexión entre Spark y MySQL

Sin este driver, tu pipeline Spark → MySQL no funcionará.

---
