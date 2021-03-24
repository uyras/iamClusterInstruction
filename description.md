# Описание вычислительного кластера

## Общие характеристики

- Адрес: 188.170.233.178
- Порт: 22
- Протокол: SSH
- Операционная система: CentOS Stream
- Система очередей: Slurm

## Аппаратное обеспечение

- 8 серверов HPE ProLiant XL170r Gen10, в каждом 2 шт. Intel Xeon-Gold 6230R, 2.1GHz/26-core/150W, 192GB DDR4. 
- 4 сервера HPE ProLiant XL225n Gen10, в каждом 2 шт AMD EPYC 7452, 2.3GHz/32-core/155W, 256GB DDR4. 
- 1 (управляющий) сервер HPE ProLiant ML350 Gen10 с 2-мя Xeon Gold 6230R (2.1GHz, 26C), 256GB DDR4, 8x8TB SATA HDD, 2шт Nvidia RTX 2060. Сеть Infiniband 100G (EDR) single link и Ethernet 10G dual link.

## Системное программное обеспечение

- Операционная система: CentOS 8 Stream
- Система модулей ПО: [LMod](https://lmod.readthedocs.io/)
- Поставщик программного обеспечения: [OpenHPC](https://openhpc.community/)
- Система очередей: [Slurm](https://slurm.schedmd.com/)
- Реализация MPI: [OpenMPI](https://www.open-mpi.org/)
- Сетевая файловая система: NFS
- Комплект компиляторов: GNU9