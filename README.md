# HDFS + Spark en macOS / Linux Ubuntu

## Objetivo

Este documento deja un setup compatible de **HDFS + Spark + PySpark** para ejecutar el notebook:

```text
ecommerce_spark_hadoop_exercise.ipynb
```

usando **HDFS real**, sin fallback a filesystem local.

El objetivo del entorno es tener una combinación estable para curso de Big Data:

- Java 17
- Hadoop 3.x
- Spark 4.1.2
- PySpark 4.1.2
- Python 3.12

---

## Problema detectado

Inicialmente se intentó instalar:

```bash
pip install --no-cache-dir jupyter pyspark==4.1.2
```

pero fallaba con:

```text
error: [Errno 122] Disk quota exceeded
```

Aunque el sistema todavía tenía espacio disponible:

```text
/dev/root  61G  8.9G  53G free
```

Después de revisar:

```bash
df -h
du -sh ~
quota
python --version
```

se detectó que el problema no era el disco principal.

Había dos factores importantes:

1. El entorno estaba usando **Python 3.14**, lo cual complicaba la instalación de PySpark en ese ambiente.
2. El directorio temporal `/tmp` estaba montado como `tmpfs` con poco espacio disponible, por ejemplo:

```text
tmpfs  2.0G  /tmp
```

PySpark es un paquete grande porque incluye muchos archivos JAR de Spark. Durante la instalación, `pip` usa directorios temporales para desempaquetar y preparar el paquete. Por eso, aunque el disco principal tuviera espacio libre, la instalación podía fallar por el tamaño limitado de `/tmp`.

La solución fue usar una combinación más estable:

```text
Python 3.12.11
Java 17
Hadoop 3.x
Spark 4.1.2
PySpark 4.1.2
```

y forzar a `pip` a usar un directorio temporal dentro del home:

```bash
TMPDIR=$HOME/tmp \
PIP_CACHE_DIR=$HOME/.cache/pip \
python -m pip install --no-cache-dir pyspark==4.1.2
```

---

## Compatibilidad

Spark debe estar compilado para la misma familia mayor de Hadoop:

```text
Hadoop 3.x ↔ Spark build hadoop3
Hadoop 2.x ↔ Spark build hadoop2.7
```

Este setup usa:

```text
Hadoop 3.x
Spark 4.1.2 build hadoop3
PySpark 4.1.2
Java 17
Python 3.12.11
```

La versión de PySpark instalada con `pip` debe coincidir con la versión de Spark que se usará en el curso.

Ejemplo:

```text
Spark 4.1.2
PySpark 4.1.2
```

---

# Parte A: Setup en Ubuntu / Linux

## 1) Instalar dependencias del sistema

```bash
sudo apt update

sudo apt install -y \
build-essential \
curl \
git \
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

## 2) Instalar pyenv

```bash
curl https://pyenv.run | bash
```

Agregar al archivo `~/.zshrc`:

```bash
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"

eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
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

## 3) Instalar Python 3.12

```bash
pyenv install 3.12.11
```

Esto instala Python en una ruta similar a:

```text
/home/ubuntu/.pyenv/versions/3.12.11
```

Validar:

```bash
pyenv versions
```

---

## 4) Crear entorno aislado para Spark

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

La salida esperada debe parecerse a esto:

```text
/home/ubuntu/.pyenv/versions/spark-course/bin/python
Python 3.12.11
```

---

## 5) Instalar Jupyter, ipykernel y PySpark

Con el entorno `spark-course` activado:

```bash
pyenv activate spark-course
```

Validar de nuevo que se está usando el Python correcto:

```bash
which python
python --version
```

Actualizar `pip`:

```bash
python -m pip install --upgrade pip
```

Instalar Jupyter e ipykernel:

```bash
python -m pip install jupyter ipykernel
```

Crear carpetas temporales para evitar errores de espacio o cuota durante la instalación de PySpark:

```bash
mkdir -p ~/tmp
mkdir -p ~/.cache/pip
```

Instalar PySpark usando `~/tmp` como directorio temporal, en lugar de `/tmp`:

```bash
TMPDIR=$HOME/tmp \
PIP_CACHE_DIR=$HOME/.cache/pip \
python -m pip install --no-cache-dir pyspark==4.1.2
```

Validar que PySpark quedó instalado correctamente:

```bash
python -c "import pyspark; print(pyspark.__version__)"
```

La salida esperada es:

```text
4.1.2
```

---

## 6) Registrar kernel en Jupyter

```bash
python -m ipykernel install \
--user \
--name spark-course \
--display-name "Python 3.12 (Spark)"
```

Después, en Jupyter, seleccionar el kernel:

```text
Python 3.12 (Spark)
```

---

## 7) Validaciones rápidas del entorno Python

```bash
pyenv activate spark-course

which python
python --version

python -m pip --version
python -c "import pyspark; print(pyspark.__version__)"
jupyter kernelspec list
```

Salidas esperadas:

```text
Python 3.12.11
4.1.2
spark-course
```

---

# Parte B: Setup en macOS

## 1) Instalar Java 17 y Hadoop

```bash
brew install openjdk@17 hadoop
```

---

## 2) Variables de entorno

Agregar al final de `~/.zshrc`:

```bash
export JAVA_HOME=$(/usr/libexec/java_home -v 17)
export HADOOP_HOME="$(brew --prefix hadoop)/libexec"
export PATH="$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH"
```

Aplicar cambios:

```bash
source ~/.zshrc
```

Validar:

```bash
java -version
hadoop version
```

---

## 3) Configurar HDFS en pseudo-distribuido

Crear carpetas de datos:

```bash
mkdir -p /tmp/hadoop-$USER/dfs/name
mkdir -p /tmp/hadoop-$USER/dfs/data
```

Editar los archivos en:

```bash
"$HADOOP_HOME/etc/hadoop"
```

---

### `core-site.xml`

```xml
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://localhost:9000</value>
  </property>
</configuration>
```

---

### `hdfs-site.xml`

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

### `hadoop-env.sh`

Agregar o reemplazar:

```bash
export JAVA_HOME=$(/usr/libexec/java_home -v 17)
```

---

## 4) Formatear y levantar HDFS

Formatear el NameNode:

```bash
hdfs namenode -format
```

Levantar NameNode y DataNode:

```bash
hdfs --daemon start namenode
hdfs --daemon start datanode
```

Validar procesos:

```bash
jps
```

Deberías ver algo parecido a:

```text
NameNode
DataNode
Jps
```

Validar HDFS:

```bash
hdfs dfs -ls /
```

Si no existe nada todavía, puedes crear una carpeta de prueba:

```bash
hdfs dfs -mkdir -p /tmp/ecommerce_bigdata/raw/customers
hdfs dfs -ls /tmp/ecommerce_bigdata/raw
```

---

## 5) Instalar Spark compatible con Hadoop 3

Descargar Spark 4.1.2 build Hadoop 3:

```bash
cd ~/Downloads
curl -O https://downloads.apache.org/spark/spark-4.1.2/spark-4.1.2-bin-hadoop3.tgz
tar xzvf spark-4.1.2-bin-hadoop3.tgz
mkdir -p ~/opt
mv spark-4.1.2-bin-hadoop3 ~/opt/spark-4.1.2
```

Agregar a `~/.zshrc`:

```bash
export SPARK_HOME=~/opt/spark-4.1.2
export PATH="$SPARK_HOME/bin:$PATH"
```

Aplicar cambios:

```bash
source ~/.zshrc
```

Validar:

```bash
spark-submit --version
```

Nota:

```bash
brew install spark
```

no instala Apache Spark como se necesita para este setup. Por eso se usa el tarball oficial.

---

## 6) Instalar Python 3.12 con pyenv en macOS

Instalar `pyenv` y `pyenv-virtualenv`:

```bash
brew install pyenv pyenv-virtualenv
```

Agregar al final de `~/.zshrc`:

```bash
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"

eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

Aplicar cambios:

```bash
source ~/.zshrc
```

Instalar Python 3.12:

```bash
pyenv install 3.12.11
```

Crear entorno:

```bash
pyenv virtualenv 3.12.11 spark-course
```

Activar entorno:

```bash
pyenv activate spark-course
```

Validar:

```bash
which python
python --version
```

---

## 7) Instalar Jupyter, ipykernel y PySpark en macOS

Con el entorno `spark-course` activado:

```bash
python -m pip install --upgrade pip
python -m pip install jupyter ipykernel
```

Crear carpetas temporales:

```bash
mkdir -p ~/tmp
mkdir -p ~/.cache/pip
```

Instalar PySpark:

```bash
TMPDIR=$HOME/tmp \
PIP_CACHE_DIR=$HOME/.cache/pip \
python -m pip install --no-cache-dir pyspark==4.1.2
```

Validar:

```bash
python -c "import pyspark; print(pyspark.__version__)"
```

Registrar kernel:

```bash
python -m ipykernel install \
--user \
--name spark-course \
--display-name "Python 3.12 (Spark)"
```

---

# Parte C: Configuración de HDFS en Linux / Ubuntu

Si se usará Hadoop en Linux, configurar variables de entorno en `~/.zshrc`.

Ejemplo:

```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export HADOOP_HOME=$HOME/hadoop
export PATH="$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH"
```

Si Hadoop se instaló en otra ruta, ajustar `HADOOP_HOME`.

Aplicar cambios:

```bash
source ~/.zshrc
```

Validar:

```bash
java -version
hadoop version
```

---

## Configurar HDFS en pseudo-distribuido en Linux

Crear carpetas:

```bash
mkdir -p /tmp/hadoop-$USER/dfs/name
mkdir -p /tmp/hadoop-$USER/dfs/data
```

Editar:

```bash
$HADOOP_HOME/etc/hadoop/core-site.xml
$HADOOP_HOME/etc/hadoop/hdfs-site.xml
$HADOOP_HOME/etc/hadoop/hadoop-env.sh
```

### `core-site.xml`

```xml
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://localhost:9000</value>
  </property>
</configuration>
```

### `hdfs-site.xml`

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

### `hadoop-env.sh`

Ejemplo en Ubuntu:

```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
```

---

## Formatear y levantar HDFS

```bash
hdfs namenode -format
hdfs --daemon start namenode
hdfs --daemon start datanode
jps
```

Verificar:

```bash
hdfs dfs -ls /
```

Crear carpeta de prueba:

```bash
hdfs dfs -mkdir -p /tmp/ecommerce_bigdata/raw/customers
hdfs dfs -ls /tmp/ecommerce_bigdata/raw
```

---

# Parte D: Variables de entorno recomendadas

Agregar al final de `~/.zshrc`:

```bash
export SPARK_HOME=$HOME/opt/spark-4.1.2
export PATH="$SPARK_HOME/bin:$PATH"

export PYSPARK_PYTHON=$HOME/.pyenv/versions/spark-course/bin/python
export PYSPARK_DRIVER_PYTHON=$HOME/.pyenv/versions/spark-course/bin/python
```

Aplicar:

```bash
source ~/.zshrc
```

Validar:

```bash
echo $SPARK_HOME
echo $PYSPARK_PYTHON
echo $PYSPARK_DRIVER_PYTHON
```

---

# Parte E: Uso en Jupyter

Levantar Jupyter:

```bash
jupyter notebook
```

o:

```bash
jupyter lab
```

Abrir el notebook:

```text
ecommerce_spark_hadoop_exercise.ipynb
```

Seleccionar el kernel:

```text
Python 3.12 (Spark)
```

---

# Parte F: Validaciones completas

Ejecutar:

```bash
pyenv activate spark-course

which python
python --version

java -version
hadoop version
spark-submit --version

python -c "import pyspark; print(pyspark.__version__)"

hdfs dfs -mkdir -p /tmp/ecommerce_bigdata/raw/customers
hdfs dfs -ls /tmp/ecommerce_bigdata/raw
```

Versiones esperadas:

```text
Python 3.12.11
Java 17
Hadoop 3.x
Spark 4.1.2
PySpark 4.1.2
```

---

# Solución al error `externally-managed-environment`

Si aparece:

```text
error: externally-managed-environment
```

durante una instalación con `pip`, significa que se está usando el Python del sistema administrado por Ubuntu/Debian, no el Python del entorno `spark-course`.

No usar:

```bash
pip install --break-system-packages ...
```

La solución correcta es activar el entorno y usar `python -m pip`:

```bash
pyenv activate spark-course

which python
python --version

python -m pip install --upgrade pip
python -m pip install jupyter ipykernel
```

La ruta correcta debe parecerse a:

```text
/home/ubuntu/.pyenv/versions/spark-course/bin/python
```

---

# Solución al error `No matching distribution found for pyspark`

Si se ejecuta:

```bash
python -m pip install --only-binary=:all: pyspark
```

puede aparecer:

```text
ERROR: No matching distribution found for pyspark
```

Esto pasa porque PySpark no siempre se distribuye como wheel/binario compatible para este entorno.

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

# Solución al error `Disk quota exceeded`

Si aparece:

```text
error: [Errno 122] Disk quota exceeded
```

durante:

```bash
python -m pip install pyspark==4.1.2
```

no asumir inmediatamente que falta disco.

Revisar primero:

```bash
df -h
df -ih
echo $TMPDIR
ulimit -a
python --version
which python
```

Si `/tmp` aparece como `tmpfs` con poco espacio, por ejemplo:

```text
tmpfs  2.0G  /tmp
```

crear un temporal dentro del home:

```bash
mkdir -p ~/tmp
mkdir -p ~/.cache/pip
```

e instalar así:

```bash
TMPDIR=$HOME/tmp \
PIP_CACHE_DIR=$HOME/.cache/pip \
python -m pip install --no-cache-dir pyspark==4.1.2
```

---

# Comandos finales recomendados

Esta fue la secuencia que resolvió el problema:

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
```

---

# Estado esperado final

Al terminar, debes tener:

```text
Python 3.12.11 activo en el entorno spark-course
Jupyter instalado
ipykernel instalado
PySpark 4.1.2 instalado
Kernel de Jupyter registrado como Python 3.12 (Spark)
Java 17 funcionando
Hadoop 3.x funcionando
HDFS levantado en localhost:9000
Spark 4.1.2 build hadoop3 instalado
```

Validaciones finales:

```bash
pyenv activate spark-course

which python
python --version

python -c "import pyspark; print(pyspark.__version__)"
jupyter kernelspec list
```

Salidas esperadas:

```text
Python 3.12.11
4.1.2
spark-course
```

Después de esto ya se puede continuar con:

```text
HDFS + Spark + notebook ecommerce_spark_hadoop_exercise.ipynb
```
