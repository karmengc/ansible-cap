# ansible-cap

Ansible rol para desplegar el entorno necesario para las prácticas del módulo HPC de la asignatura Computación de Altas Prestaciones,
perteneciente al Máster en Ingeniería Informática de la Universidad de Pablo de Olavide.


## Requisitos
Es necesario tener instalado en el host:
  - ansible
  - vagrant
  - molecule-vagrant (mediante pip)
  - molecule (mediante pip)
  - virtualbox

Añadir la imagen de Centos7 de virtual box a vagrant de la siguiente manera:
```
vagrant box add centos77.vbox --name centos77
```

También es ideal añadir los plugins de Vagrant:
```
vagrant plugin install [vagrant-share,vagrant-vbguest]
```

## Dependencias
Este rol hace uso a su vez de los roles:
  - ansible-spark
  - ansible-hdfs
  - ansible-ssh-copy-id (Fuente: https://github.com/ryankwilliams/ansible-ssh-copy-id)
  - ansible-mesos-carmen

Ya que el proyecto desarrollado para la instalación de Mesos presentaba problemas
con la configuración de Zookeeper, se ha optado finalmente por incluir los proyectos:

  - ansible-mesos-master

(Fuente: https://github.com/AnsibleShipyard/ansible-mesos)

  - ansible-zookeeper-master

(Fuente: https://github.com/AnsibleShipyard/ansible-zookeeper)

## Variables
Las siguientes variables deberán ser asignadas en los ficheros de configuración....

### SPARK
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

### HDFS

**hadoop_master_ip**: "192.168.130.148"
**hadoop_worker_ip**: "192.168.130.150"
**hadoop_user**: "usuario" (mismo usuario para ambos servidores)
**hadoop_user_pwd**: "usuario1"
**hadoop_slaves**: ["worker"]
**hadoop_user_groups**: [] Grupos de usuarios a los cuales pertenece el usuario del servicio hadoop
**hadoop_test_app**: true


### MESOS
```
#mesos_install_mode: "master" # {master|slave|master-slave}
mesos_version: "1.0.1"

# Debian
mesos_package_version: "2.0.93"
mesos_os_distribution: "{{ ansible_distribution | lower }}"
mesos_os_version: "{{ ansible_distribution_version.split('.') | join('') }}"
mesos_apt_url: "http://{{ mesos_repo_host }}/{{ ansible_distribution | lower }}"
mesos_package_full_version: "{{ mesos_version }}-{{ mesos_package_version }}.{{ mesos_os_distribution }}{{ mesos_os_version }}"
mesos_apt_package: "mesos={{ mesos_package_full_version }}"

# RedHat: EPEL and Mesosphere yum repositories URL
epel_repo: "https://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/e/epel-release-7-12.noarch.rpm"
mesosphere_yum_repo: "http://repos.mesosphere.io/el/7/noarch/RPMS/mesosphere-el-repo-7-1.noarch.rpm"

# conf file settings
mesos_cluster_name: "mesos_cluster"
mesos_ip: "{{ ansible_all_ipv4_addresses[1] }}" 
mesos_hostname: "{{ ansible_hostname }}"
mesos_master_port: "5050"
mesos_slave_port: "5051"
mesos_log_location: "/var/log/mesos"
mesos_ulimit: "-n 8192"
mesos_work_dir: "/var/mesos"
mesos_quorum: "1"
zookeeper_client_port: "2181"
zookeeper_hostnames: "{{ mesos_hostname }}:{{ zookeeper_client_port }}"
mesos_zookeeper_masters: "zk://{{ zookeeper_hostnames }}/mesos"
mesos_owner: root
mesos_group: root

# Containerizer
mesos_containerizers: "docker,mesos"
mesos_executor_timeout: "5mins"

# SSL
mesos_ssl_enabled: false
mesos_ssl_support_downgrade: false
mesos_ssl_key_file: # When SSL is enabled this must be used to point to the SSL key file
mesos_ssl_cert_file: # When SSL is enabled this must be used to point to the SSL certificate file

mesos_option_prefix: "MESOS_"
```
### ZOOKEEPER
```
zookeeper_version: "3.4.14"
zookeeper_url: "http://www.us.apache.org/dist/zookeeper/zookeeper-{{zookeeper_version}}/zookeeper-{{zookeeper_version}}.tar.gz"

# Flag that selects if systemd or upstart will be used for the init service:
# Note: by default Ubuntu 15.04 and later use systemd (but support switch to upstart)
zookeeper_debian_systemd_enabled: "{{ ansible_distribution_version|version_compare(15.04, '>=') }}"
zookeeper_debian_apt_install: false
# (Optional:) add custom 'ppa' repositories depending on the distro version (only with debian_apt_install=true)
# Example: to use a community zookeeper v3.4.8 deb pkg for Ubuntu 14.04 (where latest official is v3.4.5)
zookeeper_debian_apt_repositories:
  - repository_url: "ppa:ufscar/zookeeper"
    distro_version: "14.04"

apt_cache_timeout: 3600
zookeeper_register_path_env: false

client_port: 2181
init_limit: 5
sync_limit: 2
tick_time: 2000
zookeeper_autopurge_purgeInterval: 0
zookeeper_autopurge_snapRetainCount: 10
zookeeper_cluster_ports: "2888:3888"
zookeeper_max_client_connections: 60

data_dir: "/var/lib/zookeeper"
log_dir: "/var/log/zookeeper"
zookeeper_dir: "/opt/zookeeper-{{ zookeeper_version }}" # or /usr/share/zookeeper when zookeeper_debian_apt_install is true
zookeeper_conf_dir: "{{ zookeeper_dir }}" # or /etc/zookeeper when zookeeper_debian_apt_install is true
zookeeper_tarball_dir: /opt/src

zookeeper_hosts_hostname: "{{ inventory_hostname }}"
# List of dict (i.e. {zookeeper_hosts:[{host:,id:},{host:,id:},...]})
zookeeper_hosts:
  - host: "192.168.130.148" # the machine running
    id: 1
  - host: "192.168.130.150" # the machine running
    id: 2

# Dict of ENV settings to be written into the (optional) conf/zookeeper-env.sh
zookeeper_env: {}

# Controls Zookeeper myid generation
zookeeper_force_myid: yes
```

## Molecule tests

En este proyecto es posible realizar pruebas con Molecule, mediante las cuales se crearán dos máquinas virtuales, una que será el máster
y otra que será el worker.

Si se desea modificar los recursos asignados a las máquinas habrá que modificar el fichero molecule/escenario/molecule.yml de cada
uno de los escenarios (spark,hdfs y mesos).

Para que se creen las máquinas y sean configuradas bastará con ejecutar lo siguiente:


Si queremos sólo instalar Spark:
```
molecule converge --scenario-name spark
```

Si queremos instalar Spark(Standalone) y HDFS:
```
molecule converge --scenario-name hdfs
```

Si queremos instalar Spark, HDFS y Mesos:
```
molecule converge --scenario-name mesos
```


Para acceder individualmente a cada una de las máquinas una vez estén creadas hacer:

```
molecule login -h master --scenario-name=[spark,hdfs,mesos]
molecule login -h worker --scenario-name=[spark,hdfs,mesos]
```

## Importante
Este rol ha sido creado exclusivamente para la realización de esta práctica, con lo cual, no es válido para instalaciones en otros entornos de los paquetes
indicados, aunque podría servir como base para ese fin realizando las modificaciones oportunas.

La librería wordCount\_2.0.2.jar debe descargarse y ubicarse en roles/ansible-hdfs/files para que el proyecto funcione correctamente o si no se desea
entonces establecer la variable hadoop\_test\_app a false.

Es ideal ubicar todos los roles de ansible en /etc/ansible (o enlaces simbólicos a los originales) para que sean encontrados cuando se referencien 
en playbooks como estos. Es decir, los que se encuentran en la carpeta "roles".
