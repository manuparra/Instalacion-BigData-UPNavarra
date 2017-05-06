# Instalación y despliegue para BigData  UPNavarra


# Arquitectura del sistema de ejemplo

- 4 Nodos 
- Nodo Master/Cabecera:  192.168.1.10 -> lola01
- Nodo Esclavo: 192.168.1.11 -> lola02
- Nodo Esclavo: 192.168.1.12 -> lola03
- Nodo Esclavo: 192.168.1.13 -> lola04


# Previo a la instalación

- Tener instalado JAVA. JAVA debe estar instalado en todos los nodos en el mismo directorio. En caso contrario instalar siguiendo los pasos de [#instalando-java].
- Deshabilitar SELinux.
- Firewall IPTABLES habilitado con los puertos abiertos adicionales para Hadoop/Spark  en todos los nodos:
```
-A INPUT -m state --state NEW -m tcp -p tcp --dport 50090 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 50105 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 50075 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 50070 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 50475 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 50470 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 8032 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 8030 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 8088 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 8090 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 8031 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 8033 -j ACCEPT
```
- 

# Instalación de Hadoop y HDFS

## Configuración del fichero de /etc/hosts

Para todos los nodos  editamos el fichero ```/etc/hosts``` y añadimos lo siguiente:

```
[todos_nodos]$  vi /etc/hosts
```

y añadimos (en caso de que no estén):

```
192.168.1.10 lola01
192.168.1.11 lola02
192.168.1.12 lola03
192.168.1.13 lola04
```

Es importante indicar los nombres de los ``host`` correctamente.


## Creación de usuario hadoop

Para todos los nodos hay que crear el usuario ``hadoop``:

```
[todos_nodos]$  useradd hadoop
```
y luego:

```
[todos_nodos]$  passwd hadoop
```

Utiliza un password robusto para este usuario.

Realiza estos dos pasos en todos los nodos.

## Configuración de SSH keyless

Usamos el usuario que acabamos de crear:

```
[root@lola01]$  su hadoop
```

Generamos las claves:
```
[hadoop@lola01]$  ssh-keygen -t rsa
```

Pulsa INTRO siempre que lo pregunte; deja todas las opciones por defecto y en la clave que pide, igualmente pulsa INTRO.

Ahora copiamos la clave publica al usuario hadoop en todos los nodos:

```
[hadoop@lola01]$  ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop@lola01
[hadoop@lola01]$  ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop@lola03	
[hadoop@lola01]$  ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop@lola03
[hadoop@lola01]$  ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop@lola04
```

Nos pedirá la clave de usuario hadoop que hemos creado previamente para el usuario ``hadoop``.

```
[hadoop@lola01]$  chmod 0600 ~/.ssh/authorized_keys
```

Realizar este proceso para cada uno de los nodos. De modo que todos pueda conectar a los otros sin tener que usar clave.


## Instalando JAVA

Haz este paso sólo si no tienes instalado JAVA o bien quieres actualizar la versión de JAVA:

Descargar el JDK:  http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.tar.gz?AuthParam=1494069154_427166ee96b26530d5f6ddb2a6dc76ca

En este caso usamos la ultima version disponible del JDK de la página de ORACLE:


Vamos al nodo Master ``lola01``, como root:

```
cd /opt
```
Descargamos ahí el paquete del JDK JAVA:
```
curl -O http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.tar.gz?AuthParam=1494069154_427166ee96b26530d5f6ddb2a6dc76ca
```
Descomprimir el fichero:

```
tar -zxf <file downloaded>
```

En este caso se ha descargado la versión jdk1.8.0_131.

Realizamos una copia del JDK JAVA en cada nodo:

```	
[root@lola01]$  scp -r jdk1.8.0_131 lola02:/opt
[root@lola01]$  scp -r jdk1.8.0_131 lola03:/opt
[root@lola01]$  scp -r jdk1.8.0_131 lola04:/opt
```


Ahora configuramos la versión de JAVA que vamos a usar. Hacemos lo siguiente en cada nodo:
```
[root@todos_nodos]$  alternatives --install /usr/bin/java java /opt/jdk1.8.0_131/bin/java 2
[root@todos_nodos]$  alternatives --config java
[root@todos_nodos]$  alternatives --install /usr/bin/jar jar /opt/jdk1.8.0_131/bin/jar 2
[root@todos_nodos]$  alternatives --install /usr/bin/javac javac /opt/jdk1.8.0_131/bin/javac 2
[root@todos_nodos]$  alternatives --set jar /opt/jdk1.8.0_131/bin/jar
[root@todos_nodos]$  alternatives --set javac /opt/jdk1.8.0_131/bin/javac 
```

Ahora creamos las variables de entorno en todos los nodos:

```
[root@todos_nodos]$  vi  /etc/bashrc
```
Añadimos al final:
```
[root@todos_nodos]$  export JAVA_HOME=/opt/jdk1.8.0_131
[root@todos_nodos]$  export JRE_HOME=/opt/jdk1.8.0_131/jre
[root@todos_nodos]$  export PATH=$PATH:/opt/jdk1.8.0_131/bin:/opt/jdk1.8.0_131/jre/bin
```
La ruta de JAVA debe ser la misma donde se ha instalado JAVA en el paso previo.

Fijamos los exports:
```
[root@todos_nodos]$  source /etc/bashrc
```
y validamos que tenemos el HOME de JAVA:

```
echo $JAVA_HOME
```




## Instalando Hadoop

Accedemos al nodo de cabecera ```lola01```:


```
[root@lola01]$  cd /opt
[root@lola01]$  curl -O http://www.eu.apache.org/dist/hadoop/common/hadoop-2.6.5/hadoop-2.6.5.tar.gz
[root@lola01]$  tar -zxf hadoop-2.6.5.tar.gz
[root@lola01]$  rm hadoop-2.6.5.tar.gz
[root@lola01]$  mv hadoop-2.6.5 hadoop
```

Copiamos a los demas nodos:
```
[root@lola01]$   scp -r hadoop lola02:/opt
[root@lola01]$   scp -r hadoop lola03:/opt
[root@lola01]$   scp -r hadoop lola04:/opt
```

Ahora realizamos las siguientes operaciones en todos los nodos, usando el usuario ```hadoop```:

```
[root@todos_nodos]$  su hadoop
```

```
[hadoop@todos_nodos]$  vi /home/hadoop/.bashrc 
```
Añadimos al final del fichero:

```
export HADOOP_PREFIX=/opt/hadoop
export HADOOP_HOME=$HADOOP_PREFIX
export HADOOP_COMMON_HOME=$HADOOP_PREFIX
export HADOOP_CONF_DIR=$HADOOP_PREFIX/etc/hadoop
export HADOOP_HDFS_HOME=$HADOOP_PREFIX
export HADOOP_MAPRED_HOME=$HADOOP_PREFIX
export HADOOP_YARN_HOME=$HADOOP_PREFIX
export PATH=$PATH:$HADOOP_PREFIX/sbin:$HADOOP_PREFIX/bin
```

Fijamos los export:

```
[hadoop@todos_nodos]$  source .bashrc
```

## Configuración de Hadoop

Ahora hacemos los siguientes cambios en todo los nodos:

```
[hadoop@todos_nodos]$  vi /opt/hadoop/etc/hadoop/core-site.xml
```

Añadir (ten en cuenta el nombre del servidor ```lola01```):

```
<configuration>
<property>
    <name>fs.defaultFS</name>
    <value>hdfs://lola01:9000/</value>
</property>
</configuration>
```

Ahora realizar lo siguiente:

```
[root@todos_nodos]$  chown hadoop /opt/hadoop/ -R
[root@todos_nodos]$  chgrp hadoop /opt/hadoop/ -R
[root@todos_nodos]$  mkdir /home/hadoop/datanode
[root@todos_nodos]$  chown hadoop /home/hadoop/datanode/
[root@todos_nodos]$  chgrp hadoop /home/hadoop/datanode/
```

Luego configurar el fichero: ```/opt/hadoop/etc/hadoop/hdfs-site.xml```  en todos los nodos como usuario hadoop:


```
[hadoop@todos_nodos]$  vi /opt/hadoop/etc/hadoop/hdfs-site.xml
```
y añadimos:

```
<configuration>
<property>
  <name>dfs.replication</name>
  <value>3</value>
</property>
<property>
  <name>dfs.permissions</name>
  <value>false</value>
</property>
<property>
   <name>dfs.datanode.data.dir</name>
   <value>/home/hadoop/datanode</value>
</property>
</configuration>
```


Ahora sólo en el master ```lola01```:
 
```
[hadoop@lola01]$   mkdir /home/hadoop/namenode
[hadoop@lola01]$   chown hadoop /home/hadoop/namenode/
[hadoop@lola01]$   chgrp hadoop /home/hadoop/namenode/    
```

Editamos /opt/hadoop/etc/hadoop/hdfs-site.xml en ```lola01```. Añadimos lo siguiente, aunque ya debería estar:
```
<property>
        <name>dfs.namenode.data.dir</name>
        <value>/home/hadoop/namenode</value>
</property>
```

Edita en ```lola01``` /opt/hadoop/etc/hadoop/mapred-site.xml. No exitirá el fichero, así que 
debes copiar un fichero de template a este fichero:

```
[hadoop@lola01]$   cp /opt/hadoop/etc/hadoop/mapred-site.xml.template /opt/hadoop/etc/hadoop/mapred-site.xml
```

```
[hadoop@lola01]$   vi /opt/hadoop/etc/hadoop/mapred-site.xml
```

```
<configuration>
 <property>
  <name>mapreduce.framework.name</name>
   <value>yarn</value> 
 </property>
</configuration>
```
 
Edita en ```lola01``` /opt/hadoop/etc/hadoop/yarn-site.xml  para preparar el ResourceManager y NodeManagers:



```
<property>
        <name>yarn.resourcemanager.hostname</name>
        <value>hmaster</value>
</property>
<property>
        <name>yarn.nodemanager.hostname</name>
        <value>hmaster</value> <!-- or hslave1, hslave2, hslave3 -->
</property>
<property>
  <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
</property>
```

Luego únicamente en ```lola01``` :

```
[hadoop@lola01]$   vi /opt/hadoop/etc/hadoop/slaves
```

Añadir los nodos
````
lola01
lola02
lola03
lola04
````

Finalmente añadir las siguiente líneas a /etc/sysctl.conf en todos los nodos:

```
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
```


## Despliegue de Hadoop y HDFS


Entramos en el MASTER ```lola01``` y hacemos lo siguiente para primero formatear hdfs y ejecutar el servicio HDFS:

```
[root@lola01]$     su hadoop
[hadoop@lola01]$   hdfs namenode -format
```

Arrancamos HDFS, siempre desde el nodo Cabecera:

```
[hadoop@lola01]$   cd /opt/hadoop/sbin/
[hadoop@lola01]$   ./start-dfs.sh
```

Comprobamos que todo es correcto:
```
[hadoop@lola01]$   jps
```

o bien intentado acceder a la URL: http://lola01:50070/

Por ultimo arrancamos YARN:

```
[hadoop@lola01]$  cd /opt/hadoop/sbin/
[hadoop@lola01]$  ./start-yarn.sh 
```

## Validación de la instalación

Comprobamos HDFS como el usario ``hadoop``. Para ello creamos la carpeta /user que permitirá que cada usuario del sistema pueda utilizar HDFS desde su cuenta de usuario. De modo que si creamo otro usuario p.e. manuparra, y luego creamos la carpeta en HDFS, podrá acceder a su directorio en HDFS de forma directa.

```
hdfs dfsadmin -safemode leave
hdfs dfs -mkdir /user
hdfs dfs -mkdir /user/hadoop/
```
Creamos un fichero de prueba y probamos copiarlo:

```
[hadoop@lola01]$  vi test.file
```

añadimos contenido al fichero y luego lo movemos a la carpeta en HDFS del usuario:

```
[hadoop@lola01]$  hdfs dfs -copyFromLocal test.file /user/hadoop/
```

Validamos que está el fichero copiado al espacio HDFS:

```
[hadoop@lola01]$  hdfs dfs -ls /user/hadoop/
```


# Instalación de Spark


Para todos los nodos usando el usuario root

- Descargamos Spark 2.0.1

```
[root@todos_nodos]$  curl -O http://apache.rediris.es/spark/spark-2.0.1/spark-2.0.1-bin-hadoop2.7.tgz
```

- Descomprimimos el fichero de Spark:

```
[root@todos_nodos]$  tar xfvz spark-2.0.1-bin-hadoop2.7.tgz
```

Copiamos Spark a su caperta:

```
[root@todos_nodos]$  mv spark-2.0.1-bin-hadoop2.7 /usr/local/spark
```

Descargamos Scala

```
[root@todos_nodos]$  curl -O http://downloads.lightbend.com/scala/2.11.8/scala-2.11.8.tgz
```

Descomprimimos Scala y lo movemos a su caperta
```
[root@todos_nodos]$  tar xfvz scala-2.11.8.tgz 
[root@todos_nodos]$  mv scala-2.11.8 /usr/local/scala
```

Establecemos los PATH para scala y spark:

```
echo 'export PATH=$PATH:/usr/local/spark/bin/' >> $HOME/.bash_profile 
echo 'export PATH=$PATH:/usr/local/scala/bin/' >> $HOME/.bash_profile 
echo 'export SPARK_HOME=/usr/local/spark/' >> $HOME/.bash_profile
```

Ejecutamos para setear los exports:
```
[root@todos_nodos]$  source .bash_profile
```

```
[root@todos_nodos]$  cp $SPARK_HOME/conf/spark-env.sh.template $SPARK_HOME/conf/spark-env.sh 
```

Añadimos al final del fichero $SPARK_HOME/conf/spark-env.sh estás variables :
```
[root@todos_nodos]$  vi $SPARK_HOME/conf/spark-env.sh
```
Hay que modificar en estos tres parámetros lo siguiente:

- El PATH al JAVA_HOME
- La IP del nodo concreto;  es decir la ip local del nodo por ejemplo si estoy en lola02, pondría : 192.168.1.11
- El número de cores máximos del nodo local
```
export JAVA_HOME=/opt/jdk1.8.0_131  
export SPARK_PUBLIC_DNS="<IP LOCAL>"  
export SPARK_WORKER_CORES=<CORES>
```

Ahora sólo para el nodo Cabecera ```lola01```

```
[root@lola01]$  vi $SPARK_HOME/conf/slaves
```
Añadimos los nodos esclavos de Spark:

```
lola02
lola03
lola04
lola05
```

Por último, como root desde ```lola01```, copiar claves píblicas a los otros nodos:
```
ssh-copy-id -i ~/.ssh/id_rsa.pub root@lola01
ssh-copy-id -i ~/.ssh/id_rsa.pub root@lola02
ssh-copy-id -i ~/.ssh/id_rsa.pub root@lola03
ssh-copy-id -i ~/.ssh/id_rsa.pub root@lola04
ssh-copy-id -i ~/.ssh/id_rsa.pub root@lola05
```

Finalmente para arrancar el cluster de Spark, ejecutar desde el nodo de cabecera ```lola01```:
```
$SPARK_HOME/sbin/start-all.sh
```

Chequear que todo es correcto accediendo a:

http://localhost:8080

# Comenzar con Spark+R

Necesitamos tener instalado R. En caso de no tener instalado R, usamos:

```
yum install epel-release
yum install R
```

Ahora probamos que podemos aprovechar Spark+R. Entramos en R


```
R
```

y dentro del entorno de R usamos lo siguiente:

```
library(SparkR, lib.loc = c(file.path(Sys.getenv("SPARK_HOME"), "R", "lib")))
sparkR.session(master = "local[*]", sparkConfig = list(spark.driver.memory = "2g"))

```

Ya tenemos disponible Spark+R para poder utilizarlo.












