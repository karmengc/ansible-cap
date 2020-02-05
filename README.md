# ansible-cap

Ansible rol para desplegar el entorno necesario para las prácticas del módulo HPC de la asignatura Computación de Altas Prestaciones,
perteneciente al Máster en Ingeniería Informática de la Universidad de Pablo de Olavide.

## Dependencias
Este rol hace uso a su vez de los roles:
  - ansible-hadoop
  - ansible-spark
  - ansible-mesos

## Variables
Las siguientes variables deberán ser asignadas en los ficheros de configuración....

**mode**: fjbeofbeuf

**fap\_entity**: oabwdwpiubdpw


(EJEMPLO)

\- { 'name':'redcidec201601', 'war\_version':'20160405', 'war\_tag':'146cab8b', 'path':'aciisi/redcide/c201601' }

## archive-logs

## Molecule tests

En este proyecto es posible realizar pruebas con Molecule, mediante las cuales se crearán dos máquinas virtuales, una que será el máster
y otra que será el worker.

Para que se creen las máquinas y sean configuradas bastará con ejecutar:

```
molecule converge
```

Para acceder individualmente a cada una de las máquinas una vez estén creadas hacer:

```
molecule login -h master
molecule login -h worker1
```
