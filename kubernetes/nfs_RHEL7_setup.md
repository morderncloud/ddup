## Install NFS Server
* Install NFS packages on NFS server

    [root@server ~]# yum install nfs-utils libnfsidmap

Once the packages are installed, enable and start NFS services.

    systemctl enable rpcbind

    systemctl enable nfs-server

    systemctl start rpcbind

    systemctl start nfs-server

    systemctl start rpc-statd

    systemctl start nfs-idmapd

## Create NFS Share
* Create a directory to share with client servers.

    [root@server ~]# mkdir /nfs

* Allow client servers to read and write to the created directory.

    [root@server ~]# chmod 777 /nfs/

* Modify /etc/exports to make an entry of directory /nfs

    [root@server ~]# vi /etc/exports

    /nfs 9.45.221.116(rw,sync,no_root_squash)

* /nfs : shared directory
* 9.45.221.116 :  IP address of client machine. We can also use the hostname instead of an IP address. It is also possible to define the range of clients with subnet like 192.168.12.0/24.
* rw : Writable permission to shared folder
* sync :  all changes to the according filesystem are immediately flushed to disk; the respective write operations are being waited for.
* no_root_squash : By default, any file request made by user root on the client machine is treated as by user nobody on the server. (Exactly which UID the request is mapped to depends on the UID of user “nobody” on the server, not the client.) If no_root_squash is selected, then root on the client machine will have the same level of access to the files on the system as root on the server.
* You can get to know all the option in the man page (man exports) or here.

## Export the shared directories using the following command.

    [root@server ~]# exportfs -r

Extras:

    exportfs -v : Displays a list of shares files and export options on a server

    exportfs -a : Exports all directories listed in /etc/exports

    exportfs -u : Unexport one or more directories

    exportfs -r : Reexport all directories after modifying /etc/exports

