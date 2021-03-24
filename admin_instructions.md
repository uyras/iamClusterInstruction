# WIKI для админов (подсказки)

## Скрипт для инициализации centos8-stream через wwmkchroot

Файл: `/usr/libexec/warewulf/wwmkchroot/centos-8-stream.tmpl`

Содержание:
```bash
#DESC: Red Hat Enterprise Linux 8-stream

# The general RHEL include has all of the necessary functions, but requires
# some basic variables specific to each chroot type to be defined.

# Use DNF as the package manager
PKG_MGR=dnf
EXTRA_ARGS="--releasever=8"
PLATFORMID="platform:el8"

# Uncomment to disable GPG checks on added repos
# REPO_NOGPGCHECK=1

. include-rhel

# Define the location of the YUM repository
if [ -z "$YUM_MIRROR" ]; then
    if [ -z "$YUM_MIRROR_BASE" ]; then
        YUM_MIRROR_BASE="http://mirror.centos.org/centos"
    fi
    YUM_MIRROR="${YUM_MIRROR_BASE}/8-stream/BaseOS/\$basearch/os","${YUM_MIRROR_BASE}/8-stream/AppStream/\$basearch/os","${YUM_MIRROR_BASE}/8-stream/PowerTools/\$basearch/os"
fi

# Install only what is necessary/specific for this distribution
PKGLIST="basesystem bash chkconfig coreutils e2fsprogs ethtool
    filesystem findutils gawk grep initscripts iproute iputils
    net-tools nfs-utils pam psmisc rsync sed setup
    shadow-utils rsyslog tzdata util-linux words
    zlib tar less gzip which util-linux openssh-clients
    openssh-server dhclient pciutils vim-minimal shadow-utils
    strace cronie crontabs cpio wget redhat-release hostname grub2-common glibc-langpack-en yum"

# vim:filetype=sh:syntax=sh:expandtab:ts=4:sw=4:
```

## Настройка openmpi

Чтобы работал srun без доп запуска mpiexec, надо в файле /etc/slurm/slurm.com поменять строку `MpiDefault=none` на `MpiDefault=pmix_v3`.

По умолчанию на кластер ставится рабочая версия openmpi. Но для более плотной интеграции cuda и mpi потребуется пересобрать весь slurm с правильными параметрами.


## Сборка openmpi

Зависимости: libevent libevent-devel pmix libfabric ucx hwloc

```bash
export OMPI_VERSION=4.1.0
wget https://download.open-mpi.org/release/open-mpi/v4.1/openmpi-$OMPI_VERSION.tar.gz
tar -zxf openmpi-$OMPI_VERSION.tar.gz
cd openmpi-$OMPI_VERSION.tar.gz

module load gnu9 cuda pmix ucx libfabric autotools

./configure --prefix=/opt/soft/openmpi/$OMPI_VERSION --with-hwloc=/opt/ohpc/pub/libs/hwloc/2.1.0  --with-ucx=/opt/ohpc/pub/mpi/ucx-ohpc/1.8.0 --with-pmix=/opt/ohpc/admin/pmix --with-libevent=/usr --with-ofi=/opt/ohpc/pub/mpi/libfabric/1.10.1 --with-cuda=/opt/soft/cuda-11.2 --with-slurm --without-psm2

make -j 52
make install
```

Создайте файл `/opt/ohpc/pub/moduledeps/gnu9/openmpi4/4.1.0` с содержанием:

```perl
#%Module1.0#####################################################################

proc ModulesHelp { } {

puts stderr " "
puts stderr "This module loads the openmpi library built with the gnu9 toolchain."
puts stderr "\nVersion 4.1.0\n"

}
module-whatis "Name: openmpi4 built with gnu9 toolchain"
module-whatis "Version: 4.1.0"
module-whatis "Category: runtime library"
module-whatis "Description: openmpi MPI implementation"
module-whatis "URL: https://www.open-mpi.org/"

set     version                     4.1.0

setenv          MPI_DIR             /opt/soft/openmpi/4.1.0
prepend-path    PATH                /opt/soft/openmpi/4.1.0/bin
prepend-path    MANPATH             /opt/soft/openmpi/4.1.0/share/man
prepend-path    LD_LIBRARY_PATH     /opt/soft/openmpi/4.1.0/lib
prepend-path    MODULEPATH          /opt/ohpc/pub/moduledeps/gnu9-openmpi4
prepend-path    PKG_CONFIG_PATH     /opt/soft/openmpi/4.1.0/lib/pkgconfig

depends-on ucx cuda pmix libfabric hwloc
family "MPI"
```

## Как конвертировать публичный ключ Putty в OpenSSH

Если прислали публичный ключ Putty, его сперва нужно подправить чтобы он был принят в OpenSSH.

Putty при генерации ключа выдает на экран корректный ключ OpenSSH, но при сохранении в файл его изменяет. Вот пример файла:
```
---- BEGIN SSH2 PUBLIC KEY ----
Comment: "rsa-key-20210315"
AAAAB3NzaC1yc2EAAAABJQAAAQEA7rjNFU5cytvNg1GHf7cp/GQohE2l71Lyjy3k
HEMjhPN4Jy7E8e7oUdfpYrzHr5zj79zUOScbYbpQaKio6sM8VrtMutLBs6qNbuXz
eZwVHqEOon4vZIj42woyPTc0vjGEXzi+11qGldjPByw4amNDDN61biX2fyUkz05n
0SaRiKiyu82Ye60VrcEF0GfaZeT5W2y5rCBBBRPLkKzgPsc4Y3EWVAhH9sS5HZBh
RQ35BZFSlu6dkLP95NvhrE7ZZG+R5ULX5nvLjLucVIoMRTrjeIfCHmgh/GhhIvBq
1OnBOf3Hs7kDp0e2EbAitWZYBPGMrnGLR2DewG96qf1lMXzhGw==
---- END SSH2 PUBLIC KEY ----
```

Это тот же ключ, только разбитый на строки по 64 символа и добавлены коментарии.
Для записи в формате OpenSSH надо указать тип ключа (обычно ssh-rsa), затем через пробел сам ключ (без пробелов и переносов строк), затем через пробел коментарий к ключу (не обязательно). Пример:
```
ssh-rsa AAAAB3NzaC1yc2EAAAABJQAAAQEA7rjNFU5cytvNg1GHf7cp/GQohE2l71Lyjy3kHEMjhPN4Jy7E8e7oUdfpYrzHr5zj79zUOScbYbpQaKio6sM8VrtMutLBs6qNbuXzeZwVHqEOon4vZIj42woyPTc0vjGEXzi+11qGldjPByw4amNDDN61biX2fyUkz05n0SaRiKiyu82Ye60VrcEF0GfaZeT5W2y5rCBBBRPLkKzgPsc4Y3EWVAhH9sS5HZBhRQ35BZFSlu6dkLP95NvhrE7ZZG+R5ULX5nvLjLucVIoMRTrjeIfCHmgh/GhhIvBq1OnBOf3Hs7kDp0e2EbAitWZYBPGMrnGLR2DewG96qf1lMXzhGw== rsa-key-20210315
```