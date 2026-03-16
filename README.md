# HDFS + Spark en macOS (curso)

## Objetivo
Este documento deja un setup compatible de **HDFS + Spark + PySpark** en macOS para ejecutar el notebook `ecommerce_spark_hadoop_exercise.ipynb` usando **HDFS real** (sin fallback a filesystem local).

## Compatibilidad
Spark debe ser build para la misma familia mayor de Hadoop:
- Hadoop 3.x ↔ Spark build `hadoop3`
- Hadoop 2.x ↔ Spark build `hadoop2.7`

Este setup usa:
- Hadoop 3.4.2 (Homebrew)
- Spark 3.5.8 build `hadoop3`
- PySpark 3.5.8
- Java 11

## 1) Instalar Hadoop (HDFS) y Java 11
```bash
brew install hadoop
```

## 2) Variables de entorno
Agrega al final de `~/.zshrc`:
```bash
export JAVA_HOME=$(/usr/libexec/java_home -v 11)
export HADOOP_HOME="$(brew --prefix hadoop)/libexec"
export PATH="$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH"
```
Aplica cambios:
```bash
source ~/.zshrc
```

## 3) Configurar HDFS en pseudo‑distribuido
Crea carpetas de datos:
```bash
mkdir -p /tmp/hadoop-$USER/dfs/name
mkdir -p /tmp/hadoop-$USER/dfs/data
```

Edita los archivos en `"$HADOOP_HOME/etc/hadoop"`.

`core-site.xml`:
```xml
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://localhost:9000</value>
  </property>
</configuration>
```

`hdfs-site.xml`:
```xml
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:/tmp/hadoop-$USER/dfs/name</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:/tmp/hadoop-$USER/dfs/data</value>
  </property>
</configuration>
```

En `hadoop-env.sh` agrega o reemplaza:
```bash
export JAVA_HOME=$(/usr/libexec/java_home -v 11)
```

## 4) Formatear y levantar HDFS
```bash
hdfs namenode -format
hdfs --daemon start namenode
hdfs --daemon start datanode
jps
```
Verifica:
```bash
hdfs dfs -ls /
```

## 5) Instalar Spark compatible con Hadoop 3
Descarga e instala Spark 3.5.8 build Hadoop 3:
```bash
cd ~/Downloads
curl -O https://downloads.apache.org/spark/spark-3.5.8/spark-3.5.8-bin-hadoop3.tgz
tar xzvf spark-3.5.8-bin-hadoop3.tgz
mv spark-3.5.8-bin-hadoop3 ~/opt/spark-3.5.8
```

Agrega a `~/.zshrc`:
```bash
export SPARK_HOME=~/opt/spark-3.5.8
export PATH="$SPARK_HOME/bin:$PATH"
```
Aplica cambios:
```bash
source ~/.zshrc
```

Nota: `brew install spark` no instala Apache Spark (es otro paquete), por eso se usa el tarball oficial.

## 6) Instalar PySpark para el notebook
En el mismo entorno de Python que usa Jupyter:
```bash
PYSPARK_HADOOP_VERSION=3 pip install pyspark==3.5.8
```

## 7) Validaciones rápidas
```bash
python3 -c "import pyspark; print(pyspark.__version__)"
hdfs dfs -mkdir -p /tmp/ecommerce_bigdata/raw/customers
hdfs dfs -ls /tmp/ecommerce_bigdata/raw
```

## 8) Tips de ejecución en curso
- Asegura que el kernel de Jupyter use el mismo Python donde instalaste PySpark.
- Si `hdfs` no aparece, revisa que `HADOOP_HOME/bin` esté en `PATH`.
- Si Spark no arranca, revisa `JAVA_HOME` y la versión de Java.
