# MySQL básico para el curso (Spark + JDBC)

## Objetivo
Este documento instala un **MySQL local básico** para conectar Spark con una base relacional usando el driver JDBC de `sql.md`.

## Qué necesitas
- **MySQL Server** (este documento)
- **Driver JDBC** (`mysql-connector-j`) → ver [`sql.md`](sql.md)
- **Spark 4.1.2 + PySpark 4.1.2 + Java 17 + Python 3.14** → ver [`README.md`](README.md) o [`ubuntu.md`](ubuntu.md)

## Credenciales de ejemplo (solo desarrollo local)
Para el curso usaremos valores simples:

| Variable | Valor |
|---|---|
| Host | `localhost` |
| Puerto | `3306` |
| Base de datos | `ecommerce` |
| Usuario | `spark` |
| Contraseña | `spark123` |

> Cambia la contraseña si expones MySQL fuera de tu máquina.

---

## macOS (Homebrew)

### 1) Instalar y arrancar MySQL
```bash
brew install mysql
brew services start mysql
```

### 2) Entrar a MySQL
```bash
mysql -u root
```

Si pide contraseña y es instalación nueva, prueba sin contraseña o la que configuraste en la instalación.

---

## Ubuntu

### 1) Instalar y arrancar MySQL
```bash
sudo apt update
sudo apt install -y mysql-server
sudo systemctl start mysql
sudo systemctl enable mysql
```

### 2) Entrar a MySQL
```bash
sudo mysql
```

---

## Configuración básica (macOS y Ubuntu)

Dentro del cliente MySQL (`mysql>`), ejecuta:

```sql
CREATE DATABASE IF NOT EXISTS ecommerce;

CREATE USER IF NOT EXISTS 'spark'@'localhost' IDENTIFIED BY 'spark123';
GRANT ALL PRIVILEGES ON ecommerce.* TO 'spark'@'localhost';
FLUSH PRIVILEGES;

USE ecommerce;

CREATE TABLE IF NOT EXISTS customers (
  customer_id INT PRIMARY KEY,
  name VARCHAR(100),
  email VARCHAR(100),
  city VARCHAR(80)
);

INSERT INTO customers (customer_id, name, email, city) VALUES
  (1, 'Ana López', 'ana@example.com', 'CDMX'),
  (2, 'Luis Pérez', 'luis@example.com', 'Guadalajara'),
  (3, 'María Ruiz', 'maria@example.com', 'Monterrey');
```

Sal del cliente:
```sql
EXIT;
```

---

## Validaciones rápidas

### Probar login del usuario del curso
```bash
mysql -u spark -pspark123 -h localhost -P 3306 -e "SHOW DATABASES;"
```

### Ver tabla de prueba
```bash
mysql -u spark -pspark123 -D ecommerce -e "SELECT * FROM customers;"
```

### Verificar que MySQL está activo
```bash
# macOS
brew services list | grep mysql

# Ubuntu
sudo systemctl status mysql
```

---

## Conectar desde Spark (PySpark)

Primero configura el JAR JDBC según [`sql.md`](sql.md).

Ejemplo mínimo en Python:

```python
import os
from pyspark.sql import SparkSession

MYSQL_JDBC_JAR = os.getenv("MYSQL_JDBC_JAR", "/opt/jars/mysql-connector-j-8.4.0.jar")

spark = (
    SparkSession.builder
    .appName("mysql-test")
    .config("spark.jars", MYSQL_JDBC_JAR)
    .getOrCreate()
)

jdbc_url = "jdbc:mysql://localhost:3306/ecommerce"
props = {
    "user": "spark",
    "password": "spark123",
    "driver": "com.mysql.cj.jdbc.Driver",
}

df = spark.read.jdbc(url=jdbc_url, table="customers", properties=props)
df.show()
```

---

## Variables de entorno útiles (opcional)

Agrega a `~/.zshrc` o `~/.bashrc`:

```bash
export MYSQL_HOST=localhost
export MYSQL_PORT=3306
export MYSQL_DB=ecommerce
export MYSQL_USER=spark
export MYSQL_PASSWORD=spark123
export MYSQL_JDBC_JAR=/opt/jars/mysql-connector-j-8.4.0.jar
```

Aplica cambios:
```bash
source ~/.zshrc   # macOS
# o
source ~/.bashrc  # Ubuntu
```

---

## Errores comunes

### `ERROR 2002 (HY000): Can't connect to local MySQL server`
MySQL no está corriendo.

```bash
# macOS
brew services start mysql

# Ubuntu
sudo systemctl start mysql
```

### `Access denied for user 'spark'@'localhost'`
Repite la creación de usuario y permisos en la sección de configuración básica.

### `java.lang.ClassNotFoundException: com.mysql.cj.jdbc.Driver`
Falta el driver JDBC o la ruta es incorrecta. Revisa [`sql.md`](sql.md).

### `Public Key Retrieval is not allowed` (a veces en MySQL 8)
Agrega esto a la URL JDBC:

```text
jdbc:mysql://localhost:3306/ecommerce?allowPublicKeyRetrieval=true&useSSL=false
```

---

## Tips de ejecución en curso
- Usa el mismo entorno Python 3.14 donde instalaste PySpark.
- El JAR JDBC y MySQL Server son cosas distintas: uno conecta, el otro almacena datos.
- Si solo trabajas con HDFS en un ejercicio, puedes omitir MySQL y el JDBC.
- Spark 4.x requiere Java 17; ver [`README.md`](README.md) o [`ubuntu.md`](ubuntu.md).
