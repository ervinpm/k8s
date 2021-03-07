# Setting up NFS Persistent volume

## Enable NTP
First check if your NTP is enabled by doing a `timedatectl` command:

```bash
$ timedatectl
               Local time: Sat 2021-02-13 11:16:22 CST
           Universal time: Sat 2021-02-13 17:16:22 UTC
                 RTC time: Sat 2021-02-13 17:16:20
                Time zone: America/Chicago (CST, -0600)
System clock synchronized: no
              NTP service: inactive
          RTC in local TZ: no
```

In the example above you'll see that system clock is not synchronized and NTP service is disabled. You can enable it by this command.

```bash
$ sudo timedatectl set-ntp true
$ sudo hwclock --systohc
```

After enabling, your `timedatectl` should look something like this:

```bash
$ timedatectl
               Local time: Sat 2021-02-13 11:26:11 CST
           Universal time: Sat 2021-02-13 17:26:11 UTC
                 RTC time: Sat 2021-02-13 17:26:11
                Time zone: America/Chicago (CST, -0600)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

Make sure to enable this to the rest of the clients who will connect to the NFS server.

## Install NFS Utils

In Arch Linux, you will need to install `nfs-utils`. This is done via pacman using the following command:

```bash
$ sudo pacman -S nfs-utils
```

## Update exports file

NFS is configured by a file called `/etc/exports`. You can use any text editor you like.

```bash
$ sudo vim /etc/exports
```

To share a file, the syntax is `<folder name> <ip>(<options>)`. Here is an example that will share `myawesomefolder` to the entire network. Relace `*` to a specific IP address if you want:

```bash
/home/myuser/myawesomefolder    *(rw,sync)
```

Apply the changes by doing a `exportfs` command. 

```bash
$ sudo exportfs -rav
```


Enable the NFS service

```bash
$ sudo systemctl enable --now nfs-server
```

## Verify shared exports

In the client, you can verify the shared folders via `showmount` command.

```bash
$ showmount -e 192.168.1.101
Export list for 192.168.1.101:
/home/myuser/myawesomefolder *
```

## Create the mounting directory

The shared folder will need a target directory. Create the new directory via `mkdir` command:

```bash
$ mkdir myawesomefolder
$ ls
myawsomefolder
$
```

## Mount the shared folder

```bash
$ sudo mount -t nfs 192.168.1.101:/home/myuser/myawesomefolder myawesomefolder
Created symlink /run/systemd/system/remote-fs.target.wants/rpc-statd.service → /usr/lib/systemd/system/rpc-statd.service.
```

## Clone the NFS Provisioner
NFS Provisioner `YAML` files are found in Kubernetes SIGS repository. This assumes that you already have git installed

```bash
$ git clone git@github.com:kubernetes-sigs/nfs-subdir-external-provisioner.git
```

## Update NFS Details
After cloning the repository, udpate `deployment.yaml` and change the following to the nfs details that you have noted earlier

```yaml
          env:
            - name: PROVISIONER_NAME
              value: k8s-sigs.io/nfs-subdir-external-provisioner
            - name: NFS_SERVER
              value: <replace with NFS IP>
            - name: NFS_PATH
              value: <replace with NFS Folder>
      volumes:
        - name: nfs-client-root
          nfs:
            server: <replace with NFS IP>
            path: <replace with NFS Folder>
```

## Create Role Based Access Control (RBAC)
This is needed to allow a pod to automatically create kubernetes objects that will provision Persistent Volumes

```bash
$ kubectl create -f deploy/rbac.yaml
```

## Update class storage file configuration
You may want to update the default configuration to not delete the entire folder when a persistent volume is destroyed.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: managed-nfs-storage
provisioner: k8s-sigs.io/nfs-subdir-external-provisioner # or choose another name, must match deployment's env PROVISIONER_NAME'
parameters:
  pathPattern: "${.PVC.namespace}/${.PVC.annotations.nfs.io/storage-path}" # waits for nfs.io/storage-path annotation, if not specified will accept as empty string.
  onDelete: delete
```

Create the class storage object

```bash
$ kubectl create -f deploy/class.yaml
```

## Testing
This is optional but recommended specially if this is the first time that you are doing this. The repository has test deployments.

```bash
$ kubectl create -f deploy/test-claim.yaml -f deploy/test-pod.yaml
```

Check the NFS Server and see if there is a folder with a `SUCCESS` file name in it. If you see it, then all is set, you may delete the created test objects in your kubernetes installation.

```bash
$ kubectl delete -f deploy/test-pod.yaml -f deploy/test-claim.yaml
```