# Cloud-init konfiguracija

Sa Arch Linux stranice preuzimamo: 

```
Arch-Linux-x86_64-cloudimg-20230623.159987.qcow2
```

nakon toga generiramo šifru u hashiranom obliku koja će se koristiti za prijavu korisnika:

```
openssl passwd -6 -salt 0123456789abcdef password   
$6$0123456789abcdef$xDR267KnEdmU47Tv58n3gNdiOagKeAfIkcMyR6onqwUh3VWY6KqolWZcf8h/.iTNDA0
```
Zatim generiramo rsa ključ:

```
ssh-keygen   
Generating public/private rsa key pair.
Enter file in which to save the key (/home/mbakk/.ssh/id_rsa): 
/home/mbakk/.ssh/id_rsa already exists.
Overwrite (y/n)? y
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/mbakk/.ssh/id_rsa
Your public key has been saved in /home/mbakk/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:gRGJSMfl+r4iztq2dh4gFLevMjSmlIt6TcQfu+FdM6k mbakk@mbakk
The key's randomart image is:
+---[RSA 3072]----+
| o.+.o+o         |
|  +.+..o         |
| . o  o .        |
|. . +..  .       |
|.*...o oS  .     |
|*.o.o.+   =      |
|o+ +...+ o o     |
|.oB +oo E        |
|o*==..o.         |
+----[SHA256]-----+
```

Sada stvarano user-data datoteku u kojoj definiramo korisnika "arch" i postavljamo mu lozinku "password":

```
#cloud-config
users:
  - name: arch
    gecos: korisnik
    groups: wheel, adm
    sudo: ALL=(ALL) NOPASSWD:ALL
    lock_passwd: false
    passwd: $6$0123456789abcdef$xDR267KnEdmU47Tv58n3gNdiOagKeAfIkcMyR6onqwUh3VWY6KqolWZcf8h/S6vI3O1EuVVODhFyX8F.iTNDA0 
    ssh_authorized_keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC7WGeGBYaLDXSCqu6GXKnu9F4/XcqQ22BwjSyruP0K5rifIY7GLqUjbPCvUvu8tHXym9HYS0r3bIuzYKLTliN/OpN//30LOyvG2S2eh846LdzKAxUYQ7g+0Raa74PIr300VC5SBCW6JxEO47gymt7Zx1Iv2mx9xboIG0MLvmHAMK+a/sgC2wCqQrGI2aN6eBdMI+4xVsPGwKMpIOLHeokwkV/X0pLOxMlLOziBo5rGOpF/WUwpGjg5N6YjAejLzuKvnm9T88gzS1p8T8WL4ouWb2lslwhGlb107bRfRqk3gctJXsebwm1Ti6+I/dEjbX3jpsIsCph1pqqQwxxtXbQMWRnY4HRAqybcgw/oZKkybik11aU34f3MykIUM+ShnuCjE3PXQi0BT97r2xySCSD1kqnQJINy+m2Wg4+4alhfxTaTObsB4FsTadP5MB29gOed+4DmHFTvWBz9+LNANaQ8IQGeFtCss8qBAPwbqhCJGB4h8ZRo2Xg9QghQKoahGTU= mbakk@mbakk
```


Pomoću alata cloud-localds stvaramo sliku diska:


```
cloud-localds user-data.img user-data
```

Tijekom instalacije virtualne mašine dodajemo disk user-data.img i nakon toga je prijava na virtualnu mašinu moguća kao korisnik "arch".

```
ssh arch@192.168.122.74 
The authenticity of host '192.168.122.74 (192.168.122.74)' can't be established.
ED25519 key fingerprint is SHA256:nagOBJ5tWeAGxS8dl5DbGeSLrCJyqZHgaq/X/rYv99M.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.122.74' (ED25519) to the list of known hosts.
arch@192.168.122.74's password: 
Last login: Mon Jun 26 03:51:34 2023
```










# ZPOOL

Ažuriranje baze podataka paketa i nadogradnja (opcionalno):
```
sudo pacman -Syu
```

Instalacija paketa za kompajliranje i izradu softverskih paketa:
```
sudo pacman -S base-devel
```

Instalacija git paketa:

```
sudo pacman -S git
```

Instalacija yay upravitelja paketa:
```
cd /opt
sudo git clone https://aur.archlinux.org/yay.git

sudo chown -R debugpoint:users ./yay
cd yay

makepkg -si
```
Promjeniti debugpoint:users ovisno o nazivu korisnika i grupi

Instalacija paketa 'linux-headers' koji sadrži datoteke zaglavlja i skripte za izgradnju povezane s jezgrom Linuxa:

```
sudo pacman -S linux-headers
```

Instalacija 'zfs-dkms' i 'zfs-utils' paketa za implementaciju i upravljanje ZFS datotečnim sustavom putem yay upravitelja podataka:

```
yay -S zfs-dkms zfs-utils
```

Provjera instaliranih paketa:

```
$ yay -Qi zfs-dkms
Name            : zfs-dkms
Version         : 2.1.12-1
Description     : Kernel modules for the Zettabyte File System.
Architecture    : any
URL             : https://zfsonlinux.org/
Licenses        : CDDL
Groups          : None
Provides        : ZFS-MODULE=2.1.12  SPL-MODULE=2.1.12  spl-dkms  zfs
Depends On      : zfs-utils=2.1.12  dkms
Optional Deps   : None
Required By     : None
Optional For    : None
Conflicts With  : spl-dkms
Replaces        : spl-dkms
Installed Size  : 17.31 MiB
Packager        : Unknown Packager
Build Date      : Tue Jul 4 14:57:54 2023
Install Date    : Tue Jul 4 14:58:05 2023
Install Reason  : Explicitly installed
Install Script  : No
Validated By    : None

$ yay -Qi zfs-utils
Name            : zfs-utils
Version         : 2.1.12-1
Description     : Userspace utilities for the Zettabyte File System.
Architecture    : x86_64
URL             : https://zfsonlinux.org/
Licenses        : CDDL
Groups          : None
Provides        : None
Depends On      : None
Optional Deps   : python: for arcstat/arc_summary/dbufstat
Required By     : zfs-dkms
Optional For    : None
Conflicts With  : None
Replaces        : None
Installed Size  : 8.91 MiB
Packager        : Unknown Packager
Build Date      : Tue Jul 4 14:13:23 2023
Install Date    : Tue Jul 4 14:57:08 2023
Install Reason  : Explicitly installed
Install Script  : No
Validated By    : None
```

Učitavanje ZFS modula u Linux kernel:
```
sudo modprobe zfs
```

Ako se prilikom pokretanja naredbe pojavi sljedeći problem, potrebno je ponovo pokrenuti virtualnu mašinu:
```
$ sudo modprobe zfs
modprobe: FATAL: Module zfs not found in directory /lib/modules/6.3.8-arch1-1
```

Nakon učitavanja može se omogućiti da se ZFS usluge automatski pokreću prilikom pokretanja sustava:
```
sudo systemctl enable zfs.target
sudo systemctl enable zfs-import-cache
sudo systemctl enable zfs-mount
sudo systemctl enable zfs-import.target
```


Nakon toga trebali bi smo imati dostupnu naredbu 'zpool' za upravljanje ZFS volumenima.
```
$ zpool --version
zfs-2.1.12-1
zfs-kmod-2.1.12-1

```

Za izradu ZFS bazena, potrebno je prvo stvoriti 2 diska virtualne mašine. To radimo tako da na virtualnoj mašini otidemo u View->Details->Add Hardware->Finish


Nakon toga naredbom 'lsblk' možemo provjeriti dostupne diskove i njihove oznake:

```
$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
vda    254:0    0   40G  0 disk 
├─vda1 254:1    0    1M  0 part 
├─vda2 254:2    0  300M  0 part 
└─vda3 254:3    0 39.7G  0 part /
vdb    254:16   0   20G  0 disk 
vdc    254:32   0   20G  0 disk 
```

Diskovi koje smo dodali virtualnoj mašini su 'vdb' i 'vdc'. Sada možemo stvoriti ZSF bazen 'myzpool' sa diskovima '/dev/vdb' i '/dev/vdc' u mirroru koristeći naredbu 'zpool create':
```
sudo zpool create myzpool mirror /dev/vdb /dev/vdc
```

Status bazena mozemo provjeriti naredbom 'zpool status':
```
$ sudo zpool status myzpool
  pool: myzpool
 state: ONLINE
config:

        NAME        STATE     READ WRITE CKSUM
        myzpool     ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            vdb     ONLINE       0     0     0
            vdc     ONLINE       0     0     0

errors: No known data errors
```

Sada unutar kreiranog ZFS bazena možemo stvoriti jedan ili vise ZFS datotečnih sustava za upravljanje i organiziranje podataka, tako da koristimo naredbu 'zfs create':
```
$ sudo zfs create myzpool/storage
```

ZFS datotečni sustavi se automatski montiraju pri izradi, ali mogu se i ručno montirati i demontirati naredbom 'zfs mount' i 'zfs unmount'

```
sudo zfs mount myzpool/storage
sudo zfs unmount myzpool/storage
```

Datotečni sustav biti će montiran na /myzpool/storage te mu je moguće pristupiti i koristiti ga kao pohranu podataka.


Sljedećom naredbom možemo vidjeti svojstva stvorenog datotečnog sustava:
```
$ zfs get all myzpool/storage
NAME             PROPERTY              VALUE                  SOURCE
myzpool/storage  type                  filesystem             -
myzpool/storage  creation              Tue Jul  4 15:40 2023  -
myzpool/storage  used                  24K                    -
myzpool/storage  available             18.9G                  -
myzpool/storage  referenced            24K                    -
myzpool/storage  compressratio         1.00x                  -
myzpool/storage  mounted               yes                    -
myzpool/storage  quota                 none                   default
myzpool/storage  reservation           none                   default
myzpool/storage  recordsize            128K                   default
myzpool/storage  mountpoint            /myzpool/storage       default
myzpool/storage  sharenfs              off                    default
myzpool/storage  checksum              on                     default
myzpool/storage  compression           off                    default
myzpool/storage  atime                 on                     default
myzpool/storage  devices               on                     default
myzpool/storage  exec                  on                     default
myzpool/storage  setuid                on                     default
myzpool/storage  readonly              off                    default
myzpool/storage  zoned                 off                    default
myzpool/storage  snapdir               hidden                 default
myzpool/storage  aclmode               discard                default
myzpool/storage  aclinherit            restricted             default
myzpool/storage  createtxg             114                    -
myzpool/storage  canmount              on                     default
myzpool/storage  xattr                 on                     default
myzpool/storage  copies                1                      default
myzpool/storage  version               5                      -
myzpool/storage  utf8only              off                    -
myzpool/storage  normalization         none                   -
myzpool/storage  casesensitivity       sensitive              -
myzpool/storage  vscan                 off                    default
myzpool/storage  nbmand                off                    default
myzpool/storage  sharesmb              off                    default
myzpool/storage  refquota              none                   default
myzpool/storage  refreservation        none                   default
myzpool/storage  guid                  9074445908743719072    -
myzpool/storage  primarycache          all                    default
myzpool/storage  secondarycache        all                    default
myzpool/storage  usedbysnapshots       0B                     -
myzpool/storage  usedbydataset         24K                    -
myzpool/storage  usedbychildren        0B                     -
myzpool/storage  usedbyrefreservation  0B                     -
myzpool/storage  logbias               latency                default
myzpool/storage  objsetid              260                    -
myzpool/storage  dedup                 off                    default
myzpool/storage  mlslabel              none                   default
myzpool/storage  sync                  standard               default
myzpool/storage  dnodesize             legacy                 default
myzpool/storage  refcompressratio      1.00x                  -
myzpool/storage  written               24K                    -
myzpool/storage  logicalused           12K                    -
myzpool/storage  logicalreferenced     12K                    -
myzpool/storage  volmode               default                default
myzpool/storage  filesystem_limit      none                   default
myzpool/storage  snapshot_limit        none                   default
myzpool/storage  filesystem_count      none                   default
myzpool/storage  snapshot_count        none                   default
myzpool/storage  snapdev               hidden                 default
myzpool/storage  acltype               off                    default
myzpool/storage  context               none                   default
myzpool/storage  fscontext             none                   default
myzpool/storage  defcontext            none                   default
myzpool/storage  rootcontext           none                   default
myzpool/storage  relatime              off                    default
myzpool/storage  redundant_metadata    all                    default
myzpool/storage  overlay               on                     default
myzpool/storage  encryption            off                    default
myzpool/storage  keylocation           none                   default
myzpool/storage  keyformat             none                   default
myzpool/storage  pbkdf2iters           0                      default
myzpool/storage  special_small_blocks  0                      default
```


# Automatizacija postavljanja Funkwhale web aplikacije 


Na virtualnoj mašini potrebno je instalirati pakete: 'base-level', 'git', 'python', 'ansible', 'yay' i 'openssh'.
```
sudo pacman -S base-devel
sudo pacman -S git
sudo pacman -S python
sudo pacman -S ansible

cd /opt
sudo git clone https://aur.archlinux.org/yay.git

# sudo chown -R debugpoint:users ./yay
sudo chown -R arch:arch ./yay

cd yay
makepkg -si

sudo pacman -S openssh
```

Nakon instalacije potrebnih paketa za rad anisble-a u direktoriju /etc/ansible stvaramo konfiguracijsku datoteku 'inventory.ini' koja se koristi za definiranje hostova kojima Ansible upravlja.
```yaml
[web]
arch@192.168.122.74
```
Nakon definiranja hostova generiramo SSH ključ naredbom:
```
$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/arch/.ssh/id_rsa): 
Created directory '/home/arch/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/arch/.ssh/id_rsa
Your public key has been saved in /home/arch/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:Gl4A62i/K2lETG/uKwDQPIBnABR1Zxgu45DIgCBkA70 arch@archlinux
The key's randomart image is:
+---[RSA 3072]----+
|^%o..ooo         |
|X @ o+o          |
|oO *...          |
|. Eo=  .         |
|..o+. . S        |
|...... +         |
| o o. o          |
|  = ..           |
| . o+o           |
+----[SHA256]-----+

```
te ga kopiramo na virtualnu mašinu:
```
$ ssh-copy-id arch@192.168.122.74
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/arch/.ssh/id_rsa.pub"
The authenticity of host '192.168.122.74 (192.168.122.74)' can't be established.
ED25519 key fingerprint is SHA256:TM4DLCHt8ENxM1x5mTNvO4/wbLIiydQcHD26LpHIpY8.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
arch@192.168.122.74's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'arch@192.168.122.74'"
and check to make sure that only the key(s) you wanted were added.
```

Zatim u istom direktoriju '/etc/ansible' stvaramo 'funhwhale.yaml' datoteku:

```yaml
---
- name: Install and configure Funkwhale on Arch Linux
  hosts: web
  become: true
  gather_facts: false

  tasks:
    - name: Update package cache
      pacman:
        update_cache: yes

    - name: Install Apache HTTP Server
      pacman:
        name: apache
        state: present

    - name: Install PostgreSQL
      pacman:
        name: postgresql
        state: present

    - name: Install Redis
      pacman:
        name: redis
        state: present

    - name: Install yay AUR helper
      pacman:
        name: yay
        state: present

    - name: Install Funkwhale from AUR
      become_user: arch
      shell: "yay -S --noconfirm funkwhale-venv"

    - name: Configure /etc/hosts
      lineinfile:
        path: /etc/hosts
        line: "{{ item }}"
        state: present
      with_items:
        - "127.0.0.1       localhost"
        - "::1             localhost"
        - "127.0.0.1       funkwhale.local"

    - name: Copy Apache configuration
      copy:
        src: /etc/webapps/funkwhale/apache-funkwhale.conf
        dest: /etc/httpd/conf/extra/funkwhale.conf

    - name: Add Funkwhale configuration to Apache
      lineinfile:
        path: /etc/httpd/conf/httpd.conf
        line: "Include conf/extra/funkwhale.conf"
        state: present

    - name: Restart Apache service
      service:
        name: httpd
        state: restarted

    - name: Create Funkwhale database
      become_user: postgres
      command: psql -c "CREATE DATABASE funkwhale WITH ENCODING 'utf8';"
      ignore_errors: true

    - name: Create Funkwhale database user
      become_user: postgres
      command: psql -c "CREATE USER funkwhale;"
      ignore_errors: true

    - name: Set Funkwhale database owner
      become_user: postgres
      command: psql -c "ALTER DATABASE funkwhale OWNER TO funkwhale;"
      ignore_errors: true

    - name: Set ownership and permissions for Funkwhale user
      file:
        path: "/srv/funkwhale"
        owner: arch
        group: arch
        mode: "0755"

    - name: Install Funkwhale as funkwhale user
      become_user: funkwhale
      command: funkwhale_manage migrate
      args:
        chdir: /usr/share/webapps/funkwhale/api/

    - name: Create Funkwhale superuser
      become_user: funkwhale
      command: funkwhale_manage createsuperuser
      args:
        chdir: /usr/share/webapps/funkwhale/api/

    - name: Collect static files for Funkwhale
      become_user: funkwhale
      command: funkwhale_manage collectstatic
      args:
        chdir: /usr/share/webapps/funkwhale/api/
```


Za pokretanje anisble playbook-a pokrenuti sljedeću naredbu:
```bash
ansible-playbook -i inventory.ini funkwhale.yaml
```

Ako prilikom pokretanja playbook-a dođe do sljedećeg problema sa postgreSQL uslugom:
```bash
TASK [Create Funkwhale database] ******************************************************************************************
[WARNING]: Unable to use /var/lib/postgres/.ansible/tmp as temporary directory, failing back to system: [Errno 13]
Permission denied: '/var/lib/postgres/.ansible'
fatal: [arch@192.168.122.74]: FAILED! => {"changed": true, "cmd": ["psql", "-c", "CREATE DATABASE funkwhale WITH ENCODING 'utf8';"], "delta": "0:00:00.007172", "end": "2023-07-04 18:54:47.659600", "msg": "non-zero return code", "rc": 2, "start": "2023-07-04 18:54:47.652428", "stderr": "psql: error: connection to server on socket \"/run/postgresql/.s.PGSQL.5432\" failed: No such file or directory\n\tIs the server running locally and accepting connections on that socket?", "stderr_lines": ["psql: error: connection to server on socket \"/run/postgresql/.s.PGSQL.5432\" failed: No such file or directory", "\tIs the server running locally and accepting connections on that socket?"], "stdout": "", "stdout_lines": []}
...ignoring
```

I uslugu postgreSQL nije moguće pokrenuti ručno:
```bash
$ sudo systemctl status postgresql
○ postgresql.service - PostgreSQL database server
     Loaded: loaded (/usr/lib/systemd/system/postgresql.service; disabled; preset: disabled)
     Active: inactive (dead)

$ sudo systemctl start postgresql
Job for postgresql.service failed because the control process exited with error code.
See "systemctl status postgresql.service" and "journalctl -xeu postgresql.service" for details.
```

Potrebno je inicijalizirati novi klaster baze podataka naredbom:

```bash
$ sudo -u postgres initdb --locale C.UTF-8 -E UTF8 -D '/var/lib/postgres/data'
The files belonging to this database system will be owned by user "postgres".
This user must also own the server process.

The database cluster will be initialized with locale "C.UTF-8".
The default text search configuration will be set to "english".

Data page checksums are disabled.

fixing permissions on existing directory /var/lib/postgres/data ... ok
creating subdirectories ... ok
selecting dynamic shared memory implementation ... posix
selecting default max_connections ... 100
selecting default shared_buffers ... 128MB
selecting default time zone ... UTC
creating configuration files ... ok
running bootstrap script ... ok
performing post-bootstrap initialization ... ok
syncing data to disk ... ok

initdb: warning: enabling "trust" authentication for local connections
initdb: hint: You can change this by editing pg_hba.conf or using the option -A, or --auth-local and --auth-host, the next time you run initdb.

Success. You can now start the database server using:

    pg_ctl -D /var/lib/postgres/data -l logfile start
```

Nakon inicijalizacije moguće je ručno pokretanje postgreSQL usluge:
```bash

$ sudo systemctl start postgresql

$ sudo systemctl status postgresql
● postgresql.service - PostgreSQL database server
     Loaded: loaded (/usr/lib/systemd/system/postgresql.service; disabled; preset: disabled)
     Active: active (running) since Tue 2023-07-04 19:03:05 UTC; 14s ago
    Process: 7227 ExecStartPre=/usr/bin/postgresql-check-db-dir ${PGROOT}/data (code=exited, status=0/SUCCESS)
   Main PID: 7230 (postgres)
      Tasks: 6 (limit: 9287)
     Memory: 17.2M
        CPU: 159ms
     CGroup: /system.slice/postgresql.service
             ├─7230 /usr/bin/postgres -D /var/lib/postgres/data
             ├─7232 "postgres: checkpointer "
             ├─7233 "postgres: background writer "
             ├─7235 "postgres: walwriter "
             ├─7236 "postgres: autovacuum launcher "
             └─7237 "postgres: logical replication launcher "

Jul 04 19:03:05 archlinux systemd[1]: Starting PostgreSQL database server...
Jul 04 19:03:05 archlinux postgres[7230]: 2023-07-04 19:03:05.804 UTC [7230] LOG:  starting PostgreSQL 15.3 on x86_64-pc-l>
Jul 04 19:03:05 archlinux postgres[7230]: 2023-07-04 19:03:05.804 UTC [7230] LOG:  listening on IPv6 address "::1", port 5>
Jul 04 19:03:05 archlinux postgres[7230]: 2023-07-04 19:03:05.804 UTC [7230] LOG:  listening on IPv4 address "127.0.0.1", >
Jul 04 19:03:05 archlinux postgres[7230]: 2023-07-04 19:03:05.807 UTC [7230] LOG:  listening on Unix socket "/run/postgres>
Jul 04 19:03:05 archlinux postgres[7234]: 2023-07-04 19:03:05.813 UTC [7234] LOG:  database system was shut down at 2023-0>
Jul 04 19:03:05 archlinux postgres[7230]: 2023-07-04 19:03:05.817 UTC [7230] LOG:  database system is ready to accept conn>
Jul 04 19:03:05 archlinux systemd[1]: Started PostgreSQL database server.

```


Zatim ponovo pokrenimo playbook:

```
$ ansible-playbook -i inventory.ini funkwhale.yaml

PLAY [Install and configure Funkwhale on Arch Linux] **********************************************************************

TASK [Update package cache] ***********************************************************************************************
[WARNING]: Platform linux on host arch@192.168.122.74 is using the discovered Python interpreter at /usr/bin/python3.11,
but future installation of another Python interpreter could change the meaning of that path. See
https://docs.ansible.com/ansible-core/2.15/reference_appendices/interpreter_discovery.html for more information.
changed: [arch@192.168.122.74]

TASK [Install Apache HTTP Server] *****************************************************************************************
ok: [arch@192.168.122.74]

TASK [Install PostgreSQL] *************************************************************************************************
ok: [arch@192.168.122.74]

TASK [Install Redis] ******************************************************************************************************
ok: [arch@192.168.122.74]

TASK [Install yay AUR helper] *********************************************************************************************
ok: [arch@192.168.122.74]

TASK [Install Funkwhale from AUR] *****************************************************************************************
changed: [arch@192.168.122.74]

TASK [Configure /etc/hosts] ***********************************************************************************************
ok: [arch@192.168.122.74] => (item=127.0.0.1       localhost)
ok: [arch@192.168.122.74] => (item=::1             localhost)
ok: [arch@192.168.122.74] => (item=127.0.0.1       funkwhale.local)

TASK [Copy Apache configuration] ******************************************************************************************
ok: [arch@192.168.122.74]

TASK [Add Funkwhale configuration to Apache] ******************************************************************************
ok: [arch@192.168.122.74]

TASK [Restart Apache service] *********************************************************************************************
changed: [arch@192.168.122.74]

TASK [Create Funkwhale database] ******************************************************************************************
[WARNING]: Unable to use /var/lib/postgres/.ansible/tmp as temporary directory, failing back to system: [Errno 13]
Permission denied: '/var/lib/postgres/.ansible'
changed: [arch@192.168.122.74]

TASK [Create Funkwhale database user] *************************************************************************************
changed: [arch@192.168.122.74]

TASK [Set Funkwhale database owner] ***************************************************************************************
changed: [arch@192.168.122.74]

TASK [Set ownership and permissions for Funkwhale user] *******************************************************************
changed: [arch@192.168.122.74]

TASK [Install Funkwhale as funkwhale user] ********************************************************************************
[WARNING]: Unable to use /srv/funkwhale/.ansible/tmp as temporary directory, failing back to system: [Errno 13] Permission
denied: '/srv/funkwhale/.ansible'
changed: [arch@192.168.122.74]

TASK [Create Funkwhale superuser] *****************************************************************************************
changed: [arch@192.168.122.74]

TASK [Collect static files for Funkwhale] *********************************************************************************
changed: [arch@192.168.122.74]

PLAY RECAP ****************************************************************************************************************
arch@192.168.122.74        : ok=17   changed=10   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```


Unutar konfiguracijske datoteke za Apache HTTP poslužitelj 'etc/httpd/conf/httpd.conf' potrebno je odkomentirati liniju:
```bash
#LoadModule rewrite_module modules/mod_rewrite.so
```
i na kraj datoteke dodati:
```bash
<Directory /usr/share/webapps/funkwhale/front/dist>
    Require all granted
</Directory>
```

Zatim ponovo pokrenuti Apache HTTP posluzitelj naredbom:

```bash
sudo systemctl restart httpd
```

Sada se aplikaciji može pristupiti preko host mašine:

```bash
$ curl -v http://192.168.122.74/funkwhale/

*   Trying 192.168.122.74:80...
* Connected to 192.168.122.74 (192.168.122.74) port 80 (#0)
> GET /funkwhale/ HTTP/1.1
> Host: 192.168.122.74
> User-Agent: curl/8.1.2
> Accept: */*
> 
< HTTP/1.1 200 OK
< Date: Wed, 05 Jul 2023 12:20:47 GMT
< Server: Apache/2.4.57 (Unix)
< Last-Modified: Tue, 04 Jul 2023 16:26:23 GMT
< ETag: "cb9-5ffabbe6f29c0"
< Accept-Ranges: bytes
< Content-Length: 3257
< Content-Type: text/html
< 
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="utf-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width,initial-scale=1.0">
  <meta name="generator" content="Funkwhale">
  <title>Funkwhale</title>
  <meta name="description" content="Your free and federated audio platform">
  <link rel="apple-touch-icon" sizes="180x180" href="/apple-touch-icon.png?v=1">
  <link rel="icon" type="image/png" sizes="32x32" href="/favicon-32x32.png?v=1">
  <link rel="icon" type="image/png" sizes="16x16" href="/favicon-16x16.png?v=1">
  <link rel="mask-icon" href="/safari-pinned-tab.svg?v=1" color="#009fe3">
  <link rel="shortcut icon" href="/favicon.ico?v=1">
  <meta name="apple-mobile-web-app-title" content="Funkwhale">
  <meta name="application-name" content="Funkwhale">
  <meta name="msapplication-TileColor" content="#009fe3">
  <meta name="theme-color" content="#ffffff">
  <style>
    #fake-app {
      width: 100vw;
      height: 100vh;
      z-index: -1;
      position: fixed;
      top: 0;
      left: 0;
      display: flex;
      font-family: sans-serif;
    }
    #fake-sidebar {
      width: 275px;
      height: 100vh;
      background-color: #2D2F33;
    }
    #fake-sidebar.loaded,  #fake-content.loaded {
      display: none;
    }
    #orange-square {
      width: 56px;
      height: 56px;
      background-color: #f2711c;
    }
    #fake-content {
      height: 100vh;
      flex-grow: 1;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
    }
    #fake-content h1 {
      margin-bottom: 2em;
    }
    #fake-content .placeholder {
      width: 20em;
      max-width: 95%;
    }
    @media only screen and (max-width: 768px) {
      #fake-app {
        flex-direction: column;
      }
      #fake-sidebar {
        width: 100%;
        height: 56px;
      }
    }
  </style>
  <script type="module" crossorigin src="/assets/index-d8582d40.js"></script>
  <link rel="modulepreload" crossorigin href="/assets/index-6c6b9641.js">
  <link rel="modulepreload" crossorigin href="/assets/useLogger-964d3ffc.js">
  <link rel="modulepreload" crossorigin href="/assets/index-0ab871f8.js">
  <link rel="modulepreload" crossorigin href="/assets/en_US-055cb820.js">
  <link rel="modulepreload" crossorigin href="/assets/locale-73627d48.js">
<link rel="manifest" href="/manifest.json"></head>

<body id="body">
  <div id="fake-app">
    <div id="fake-sidebar">
      <div id="orange-square"></div>
    </div>
    <div id="fake-content">
      <noscript>
        <strong>We're sorry but Funkwhale doesn't work properly without JavaScript enabled. Please enable it to continue.</strong>
      </noscript>
      <h1>Loading Funkwhale…</h1>
      <div class="ui placeholder">
        <div class="image header">
          <div class="full line"></div>
          <div class="line"></div>
        </div>
        <div class="image header">
          <div class="line"></div>
          <div class="full line"></div>
        </div>
        <div class="image header">
          <div class="medium line"></div>
          <div class="full line"></div>
        </div>
      </div>
    </div>
  </div>
  <div id="app"></div>
  
</body>

</html>

```


# Periodički backup podataka

Za izradu periodičkog backup-a podataka web aplikacije prvo je potrebno napraviti skriptu 'backup_funkwhale.sh' koja izrađuje sigurnosnu kopiju media i music direktorija koristeci naredbu 'rsync' za prijenos datoteka na različite lokacije:

```bash
#!/bin/bash


source_media="/srv/funkwhale/data/media"
source_music="/srv/funkwhale/data/music"
destination="/myzpool/storage"


rsync -avzhP "$source_media" "$destination/media"
rsync -avzhP "$source_music" "$destination/music"

```

Skripti dodajemo dozvolu za izvršavanje:
```bash
sudo chmod +x /home/arch/backup_funkwhale.sh
```

Nakon toga u direktoriju '/etc/systemd/system' stvaramo systemd.service datoteku 'backup_funkwhale.service' koja će izvršiti skriptu 'backup_funkwhale.sh'.

```bash 


[Unit]
Description=Backup Funkwhale Data
After=network.target

[Service]
ExecStart=/home/arch/backup_funkwhale.sh

[Install]
WantedBy=multi-user.target

```

Nakon toga u direktoriju /etc/systemd/system kreiramo systemd.timer datoteku 'backup_funkwhale.timer':

```bash

[Unit]
Description=Periodic Backup for Funkwhale Data

[Timer]
OnCalendar=hourly
Persistent=true
Unit=backup_funkwhale.service

[Install]
WantedBy=timers.target

```

Usluga se moze i rucno pokrenuti naredbom :
```bash
sudo systemctl start backup_funkwhale.service
```
Nakon cega se izvrsava 'backup_funkwhale.sh' skripta
```
[arch@archlinux system]$ sudo systemctl status backup_funkwhale.service
○ backup_funkwhale.service - Backup Funkwhale Data
     Loaded: loaded (/etc/systemd/system/backup_funkwhale.service; disabled; preset: disabled)
     Active: inactive (dead) since Tue 2023-07-04 21:25:28 UTC; 1s ago
   Duration: 120ms
TriggeredBy: ● backup_funkwhale.timer
    Process: 1295 ExecStart=/home/arch/backup_funkwhale.sh (code=exited, status=0/SUCCESS)
   Main PID: 1295 (code=exited, status=0/SUCCESS)
        CPU: 41ms

Jul 04 21:25:28 archlinux systemd[1]: Started Backup Funkwhale Data.
Jul 04 21:25:28 archlinux backup_funkwhale.sh[1296]: sending incremental file list
Jul 04 21:25:28 archlinux backup_funkwhale.sh[1296]: media/
Jul 04 21:25:28 archlinux backup_funkwhale.sh[1296]: sent 79 bytes  received 20 bytes  198.00 bytes/sec
Jul 04 21:25:28 archlinux backup_funkwhale.sh[1296]: total size is 0  speedup is 0.00
Jul 04 21:25:28 archlinux backup_funkwhale.sh[1299]: sending incremental file list
Jul 04 21:25:28 archlinux backup_funkwhale.sh[1299]: music/
Jul 04 21:25:28 archlinux backup_funkwhale.sh[1299]: sent 79 bytes  received 20 bytes  198.00 bytes/sec
Jul 04 21:25:28 archlinux backup_funkwhale.sh[1299]: total size is 0  speedup is 0.00
Jul 04 21:25:28 archlinux systemd[1]: backup_funkwhale.service: Deactivated successfully.
```
