# HDFS + Spark + PySpark en Ubuntu

## Objetivo

Configurar un entorno de curso en **Ubuntu** para trabajar con:

- Java 17
- Hadoop 3.x / HDFS
- Spark 4.1.2
- PySpark 4.1.2
- Python 3.12.11
- Jupyter Notebook / JupyterLab

Este setup permite ejecutar notebooks de PySpark usando un entorno aislado llamado:

```text
spark-course
```

---

## Stack recomendado

```text
Ubuntu
Python 3.12.11
Java 17
Hadoop 3.x
Spark 4.1.2
PySpark 4.1.2
Jupyter
ipykernel
```

---

# 1. Instalar dependencias base

```bash
sudo apt update

sudo apt install -y \
build-essential \
curl \
git \
wget \
tar \
libssl-dev \
zlib1g-dev \
libbz2-dev \
libreadline-dev \
libsqlite3-dev \
libncursesw5-dev \
xz-utils \
tk-dev \
libxml2-dev \
libxmlsec1-dev \
libffi-dev \
liblzma-dev \
ca-certificates
```

---

# 2. Instalar Java 17

```bash
sudo apt install -y openjdk-17-jdk
```

Validar:

```bash
java -version
```

La salida debe mostrar Java 17.

Ejemplo:

```text
openjdk version "17..."
```

---

# 3. Configurar `JAVA_HOME`

Buscar la ruta de Java:

```bash
readlink -f $(which java)
```

Normalmente en Ubuntu será algo como:

```text
/usr/lib/jvm/java-17-openjdk-amd64/bin/java
```

Agregar al archivo `~/.zshrc`:

```bash
cat >> ~/.zshrc <<'EOF'

export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export PATH="$JAVA_HOME/bin:$PATH"

EOF
```



Aplicar cambios:

```bash
source ~/.zshrc
```

Validar:

```bash
echo $JAVA_HOME
java -version
```

---

# 4. Instalar `pyenv`

```bash
curl https://pyenv.run | bash
```

Agregar al final de `~/.zshrc`:

```bash
cat >> ~/.zshrc <<'EOF'
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"

eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
EOF
```

Aplicar cambios:

```bash
source ~/.zshrc
```

Validar:

```bash
pyenv --version
```

---

# 5. Instalar Python 3.12.11

```bash
pyenv install 3.12.11
```

Validar:

```bash
pyenv versions
```

Debe aparecer:

```text
3.12.11
```

---

# 6. Crear entorno virtual para el curso

Crear el entorno:

```bash
pyenv virtualenv 3.12.11 spark-course
```

Activarlo:

```bash
pyenv activate spark-course
```

Validar:

```bash
which python
python --version
```

La salida esperada debe parecerse a:

```text
/home/ubuntu/.pyenv/versions/spark-course/bin/python
Python 3.12.11
```

---

# 7. Instalar Jupyter e ipykernel

Con el entorno `spark-course` activado:

```bash
python -m pip install --upgrade pip
python -m pip install jupyter ipykernel
```

---

# 8. Instalar PySpark 4.1.2

Crear carpetas temporales dentro del home:

```bash
mkdir -p ~/tmp
mkdir -p ~/.cache/pip
```

Instalar PySpark usando `~/tmp` como directorio temporal:

```bash
TMPDIR=$HOME/tmp \
PIP_CACHE_DIR=$HOME/.cache/pip \
python -m pip install --no-cache-dir pyspark==4.1.2
```

Validar:

```bash
python -c "import pyspark; print(pyspark.__version__)"
```

La salida esperada es:

```text
4.1.2
```

---

# 9. Registrar kernel de Jupyter

Con el entorno `spark-course` activado:

```bash
python -m ipykernel install \
--user \
--name spark-course \
--display-name "Python 3.12 (Spark)"
```

Validar kernels disponibles:

```bash
jupyter kernelspec list
```

Debe aparecer:

```text
spark-course
```

En Jupyter seleccionar el kernel:

```text
Python 3.12 (Spark)
```

---

# 10. Instalar Hadoop 3.x

Crear carpeta de trabajo:

```bash
mkdir -p ~/opt
cd ~/opt
```

Descargar Hadoop 3.4.2:

```bash
wget https://downloads.apache.org/hadoop/common/hadoop-3.4.2/hadoop-3.4.2.tar.gz
```

Descomprimir:

```bash
tar -xzf hadoop-3.4.2.tar.gz
```

Crear alias de carpeta:

```bash
ln -sfn ~/opt/hadoop-3.4.2 ~/opt/hadoop
```

Agregar variables al archivo `~/.zshrc`:

```bash
cat >> ~/.zshrc <<'EOF'
export HADOOP_HOME=$HOME/opt/hadoop
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export PATH="$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH"
EOF
```

Aplicar cambios:

```bash
source ~/.zshrc
```

Validar:

```bash
hadoop version
```

---

# 11. Configurar Hadoop para HDFS local

Editar:

```bash
nano $HADOOP_HOME/etc/hadoop/hadoop-env.sh
```

Agregar o reemplazar:

```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
```

---

## `core-site.xml`

Editar:

```bash
nano $HADOOP_HOME/etc/hadoop/core-site.xml
```

Contenido:

```xml
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://localhost:9000</value>
  </property>
</configuration>
```

---

## `hdfs-site.xml`

Crear carpetas para NameNode y DataNode:

```bash
mkdir -p /tmp/hadoop-$USER/dfs/name
mkdir -p /tmp/hadoop-$USER/dfs/data
```

Editar:

```bash
nano $HADOOP_HOME/etc/hadoop/hdfs-site.xml
```

Contenido:

```xml
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>

  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:/tmp/hadoop-${user.name}/dfs/name</value>
  </property>

  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:/tmp/hadoop-${user.name}/dfs/data</value>
  </property>
</configuration>
```

---

# 12. Formatear HDFS

Solo se hace la primera vez:

```bash
hdfs namenode -format
```

---

# 13. Levantar HDFS

```bash
hdfs --daemon start namenode
hdfs --daemon start datanode
```

Validar procesos:

```bash
jps
```

Debe aparecer algo parecido a:

```text
NameNode
DataNode
Jps
```

Validar HDFS:

```bash
hdfs dfs -ls /
```

Crear carpeta de prueba:

```bash
hdfs dfs -mkdir -p /tmp/ecommerce_bigdata/raw/customers
hdfs dfs -ls /tmp/ecommerce_bigdata/raw
```

---

# 14. Instalar Spark 4.1.2 compatible con Hadoop 3

Ir a la carpeta de trabajo:

```bash
mkdir -p ~/opt
cd ~/opt
```

Descargar Spark 4.1.2 para Hadoop 3:

```bash
wget https://downloads.apache.org/spark/spark-4.1.2/spark-4.1.2-bin-hadoop3.tgz
```

Descomprimir:

```bash
tar -xzf spark-4.1.2-bin-hadoop3.tgz
```

Crear alias de carpeta:

```bash
ln -sfn ~/opt/spark-4.1.2-bin-hadoop3 ~/opt/spark-4.1.2
```

Agregar variables al archivo `~/.zshrc`:

```bash
cat >> ~/.zshrc <<'EOF'
export SPARK_HOME=$HOME/opt/spark-4.1.2
export PATH="$SPARK_HOME/bin:$SPARK_HOME/sbin:$PATH"

export PYSPARK_PYTHON=$HOME/.pyenv/versions/spark-course/bin/python
export PYSPARK_DRIVER_PYTHON=$HOME/.pyenv/versions/spark-course/bin/python
EOF
```

Aplicar cambios:

```bash
source ~/.zshrc
```

Validar Spark:

```bash
spark-submit --version
```

Debe mostrar Spark 4.1.2.

---

# 15. Validaciones finales

Activar entorno:

```bash
pyenv activate spark-course
```

Validar Python:

```bash
which python
python --version
```

Validar PySpark:

```bash
python -c "import pyspark; print(pyspark.__version__)"
```

Validar Java:

```bash
java -version
```

Validar Hadoop:

```bash
hadoop version
```

Validar HDFS:

```bash
hdfs dfs -ls /
```

Validar Spark:

```bash
spark-submit --version
```

Validar kernel de Jupyter:

```bash
jupyter kernelspec list
```

---

# 16. Probar PySpark básico

Ejecutar:

```bash
python
```

Dentro de Python:

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("SparkCourseTest") \
    .master("local[*]") \
    .getOrCreate()

data = [("Ana", 10), ("Luis", 20), ("Marta", 30)]
df = spark.createDataFrame(data, ["name", "value"])

df.show()

spark.stop()
```

Salida esperada:

```text
+-----+-----+
| name|value|
+-----+-----+
|  Ana|   10|
| Luis|   20|
|Marta|   30|
+-----+-----+
```

---

# 17. Probar escritura en HDFS desde Spark

Crear carpeta en HDFS:

```bash
hdfs dfs -mkdir -p /tmp/spark_test
```

Ejecutar Python:

```bash
python
```

Dentro de Python:

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("SparkHDFSTest") \
    .master("local[*]") \
    .getOrCreate()

data = [("A", 1), ("B", 2), ("C", 3)]
df = spark.createDataFrame(data, ["letter", "number"])

df.write.mode("overwrite").parquet("hdfs://localhost:9000/tmp/spark_test/output_parquet")

spark.stop()
```

Validar en HDFS:

```bash
hdfs dfs -ls /tmp/spark_test/output_parquet
```

---

# 18. Usar Jupyter

Activar entorno:

```bash
pyenv activate spark-course
```

Abrir Jupyter:

```bash
jupyter notebook
```

o:

```bash
jupyter lab
```

Abrir el notebook del curso:

```text
ecommerce_spark_hadoop_exercise.ipynb
```

Seleccionar el kernel:

```text
Python 3.12 (Spark)
```

---

# 19. Solución a errores comunes

## Error: `externally-managed-environment`

Si aparece:

```text
error: externally-managed-environment
```

significa que se está usando el Python del sistema, no el entorno `spark-course`.

Solución:

```bash
pyenv activate spark-course

which python
python --version

python -m pip install --upgrade pip
```

No usar:

```bash
pip install --break-system-packages
```

---

## Error: `Disk quota exceeded`

Si aparece:

```text
error: [Errno 122] Disk quota exceeded
```

durante la instalación de PySpark, instalar usando un directorio temporal en el home:

```bash
mkdir -p ~/tmp
mkdir -p ~/.cache/pip

TMPDIR=$HOME/tmp \
PIP_CACHE_DIR=$HOME/.cache/pip \
python -m pip install --no-cache-dir pyspark==4.1.2
```

---

## Error: `No matching distribution found for pyspark`

No usar:

```bash
python -m pip install --only-binary=:all: pyspark
```

Usar:

```bash
TMPDIR=$HOME/tmp \
PIP_CACHE_DIR=$HOME/.cache/pip \
python -m pip install --no-cache-dir pyspark==4.1.2
```

---

## Error: `ModuleNotFoundError: No module named 'pyspark'`

Validar que el entorno correcto está activo:

```bash
pyenv activate spark-course
which python
python --version
```

Luego validar instalación:

```bash
python -m pip show pyspark
python -c "import pyspark; print(pyspark.__version__)"
```

Si no aparece instalado:

```bash
TMPDIR=$HOME/tmp \
PIP_CACHE_DIR=$HOME/.cache/pip \
python -m pip install --no-cache-dir pyspark==4.1.2
```

---

# 20. Comandos finales resumidos

```bash
pyenv activate spark-course

which python
python --version

python -m pip install --upgrade pip
python -m pip install jupyter ipykernel

mkdir -p ~/tmp
mkdir -p ~/.cache/pip

TMPDIR=$HOME/tmp \
PIP_CACHE_DIR=$HOME/.cache/pip \
python -m pip install --no-cache-dir pyspark==4.1.2

python -c "import pyspark; print(pyspark.__version__)"

python -m ipykernel install \
--user \
--name spark-course \
--display-name "Python 3.12 (Spark)"

jupyter kernelspec list
```

---

# Estado esperado final

Al terminar, el entorno debe tener:

```text
Ubuntu
Java 17
Hadoop 3.x
HDFS funcionando en hdfs://localhost:9000
Spark 4.1.2
PySpark 4.1.2
Python 3.12.11
Jupyter
Kernel Python 3.12 (Spark)
```

Validación mínima final:

```bash
pyenv activate spark-course

python --version
python -c "import pyspark; print(pyspark.__version__)"
java -version
hadoop version
spark-submit --version
hdfs dfs -ls /
jupyter kernelspec list
```
