# ansible-cap

Ansible rol para desplegar el entorno necesario para las prácticas del módulo HPC de la asignatura Computación de Altas Prestaciones,
perteneciente al Máster en Ingeniería Informática de la Universidad de Pablo de Olavide.

## Dependencias
Este rol hace uso a su vez de los roles:
  - ansible-spark
  - ansible-hdfs
  - ansible-mesos

## Variables
Las siguientes variables deberán ser asignadas en los ficheros de configuración....

**spark_master_ip**: "192.168.130.148"
**spark_worker_ip**: "192.168.130.150"
**spark_master_user**: "sparkuser"  (no testeado aún con usuarios diferentes)
**spark_worker_user**: "sparkuser"
**spark_master_user_pwd**: "sparkuser1"
**spark_worker_user_pwd**: "sparkuser1"
**spark_worker_cores**: "1"

Importante la siguiente variable hay que especificar si son Megas o Gigas, si ponemos sólo 1 pensará que es 1M
(más info https://stackoverflow.com/questions/58206006/whats-the-reason-on-memory-is-below-1mb-or-missing-a-m-g-at-the-end-of-the-me)

**spark_worker_memory**: "512M"
**spark_slaves**: ["worker"]
**spark_user_groups**: [] Grupos de usuarios a los cuales pertenece el usuario del servicio spark
**spark_test_app** Indica si ejecutar el test de Spark, ya que no es posible con Ansible, sólo mostrará un mensaje

**hadoop_master_ip**: "192.168.130.148"
**hadoop_worker_ip**: "192.168.130.150"
**hadoop_user**: "usuario" (mismo usuario para ambos servidores)
**hadoop_user_pwd**: "usuario1"
**hadoop_slaves**: ["worker"]
**hadoop_user_groups**: [] Grupos de usuarios a los cuales pertenece el usuario del servicio hadoop


## Molecule tests

En este proyecto es posible realizar pruebas con Molecule, mediante las cuales se crearán dos máquinas virtuales, una que será el máster
y otra que será el worker.

Para que se creen las máquinas y sean configuradas bastará con ejecutar lo siguiente:


Si queremos sólo instalar Spark:
```
molecule converge --scenario-name default
```

Si queremos instalar Spark y HDFS:
```
molecule converge --scenario-name hdfs
```

Si queremos instalar Spark, HDFS y Mesos:
```
molecule converge --scenario-name mesos
```


Para acceder individualmente a cada una de las máquinas una vez estén creadas hacer:

```
molecule login -h master --scenario-name=[default,hdfs,mesos]
molecule login -h worker --scenario-name=[default,hdfs,mesos]
```
