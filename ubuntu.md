# HDFS + Spark en Ubuntu (curso)

## Objetivo
Este documento deja un setup compatible de **HDFS + Spark + PySpark** en Ubuntu para ejecutar el notebook `ecommerce_spark_hadoop_exercise.ipynb` usando **HDFS real** (sin fallback a filesystem local).

## Compatibilidad
Spark debe ser build para la misma familia mayor de Hadoop:
- Hadoop 3.x ↔ Spark build `hadoop3`
- Hadoop 2.x ↔ Spark build `hadoop2.7`

Este setup usa:
- Hadoop 3.4.2
- Spark 3.5.8 build `hadoop3`
- PySpark 3.5.8
- Java 11

## 1) Instalar Java 11 y descargar Hadoop
```bash
sudo apt update
sudo apt install openjdk-11-jdk
```

Descarga Hadoop 3.4.2:
```bash
cd ~/Downloads
wget https://downloads.apache.org/hadoop/common/hadoop-3.4.2/hadoop-3.4.2.tar.gz
tar xzvf hadoop-3.4.2.tar.gz
sudo mv hadoop-3.4.2 /opt/hadoop
```

## 2) Variables de entorno
Agrega al final de `~/.bashrc`:
```bash
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export HADOOP_HOME=/opt/hadoop
export PATH="$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH"
```
Aplica cambios:
```bash
source ~/.bashrc
```

## 2.1) ZSH 

```bash
echo 'export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export HADOOP_HOME=/opt/hadoop
export PATH="$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH"' >> ~/.zshrc && source ~/.zshrc
```

## 3) Configurar HDFS en pseudo‑distribuido
```bash
mkdir -p /tmp/hadoop-$USER/dfs/name /tmp/hadoop-$USER/dfs/data && printf '<configuration>\n  <property>\n    <name>fs.defaultFS</name>\n    <value>hdfs://localhost:9000</value>\n  </property>\n</configuration>\n' > $HADOOP_HOME/etc/hadoop/core-site.xml && printf '<configuration>\n  <property>\n    <name>dfs.replication</name>\n    <value>1</value>\n  </property>\n  <property>\n    <name>dfs.namenode.name.dir</name>\n    <value>file:/tmp/hadoop-$USER/dfs/name</value>\n  </property>\n  <property>\n    <name>dfs.datanode.data.dir</name>\n    <value>file:/tmp/hadoop-$USER/dfs/data</value>\n  </property>\n</configuration>\n' > $HADOOP_HOME/etc/hadoop/hdfs-site.xml && sed -i 's|^# export JAVA_HOME.*|export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64|' $HADOOP_HOME/etc/hadoop/hadoop-env.sh
```



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
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
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
wget https://downloads.apache.org/spark/spark-3.5.8/spark-3.5.8-bin-hadoop3.tgz
tar xzvf spark-3.5.8-bin-hadoop3.tgz
sudo mv spark-3.5.8-bin-hadoop3 /opt/spark-3.5.8
```

Agrega a `~/.bashrc`:
```bash
export SPARK_HOME=/opt/spark-3.5.8
export PATH="$SPARK_HOME/bin:$PATH"
```
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
