技术002CRunc

Creating an OCI Bundle
======================

-  create the top most bundle directory

``mkdir /mycontainer cd /mycontainer``

-  create the rootfs directory

``mkdir rootfs``

-  export busybox via Docker into the rootfs directory

``docker export $(docker create busybox) | tar -C rootfs -xvf -``
命令\ ``runc spec``\ 会生成基本的模版文件config.json

Running Containers
==================

1. 以root用户运行

-  run as root

``cd /mycontainer runc run mycontainerid``

If you used the unmodified runc spec template this should give you a sh
session inside the container.

The second way to start a container is using the specs lifecycle
operations. This gives you more power over how the container is created
and managed while it is running. This will also launch the container in
the background so you will have to edit the config.json to remove the
terminal setting for the simple examples here. Your process field in the
config.json should look like this below with “terminal”: false and
\`“args”: [“sleep”, “5”].

“process”: { “terminal”: false, “user”: { “uid”: 0, “gid”: 0 }, “args”:
[ “sleep”, “5”], “env”: [
“PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin”,
“TERM=xterm”], “cwd”: “/”, “capabilities”: { “bounding”: [
“CAP_AUDIT_WRITE”, “CAP_KILL”, “CAP_NET_BIND_SERVICE”], “effective”: [
“CAP_AUDIT_WRITE”, “CAP_KILL”, “CAP_NET_BIND_SERVICE”], “inheritable”: [
“CAP_AUDIT_WRITE”, “CAP_KILL”, “CAP_NET_BIND_SERVICE”], “permitted”: [
“CAP_AUDIT_WRITE”, “CAP_KILL”, “CAP_NET_BIND_SERVICE”], “ambient”: [
“CAP_AUDIT_WRITE”, “CAP_KILL”, “CAP_NET_BIND_SERVICE”] }, “rlimits”: [ {
“type”: “RLIMIT_NOFILE”, “hard”: 1024, “soft”: 1024 }],
“noNewPrivileges”: true },\`

-  run as root

cd /mycontainer runc create mycontainerid

-  view the container is created and in the “created” state

runc list

-  start the process inside the container

runc start mycontainerid

-  after 5 seconds view that the container has exited and is now in the
   stopped state

runc list

-  now delete the container

runc delete mycontainerid 1. 以普通用户运行

-  Same as the first example

   mkdir ~/mycontainer cd ~/mycontainer mkdir rootfs docker export
   $(docker create busybox) \| tar -C rootfs -xvf -

-  The –rootless parameter instructs runc spec to generate a
   configuration for a rootless container, which will allow you to run
   the container as a non-root user.

``runc spec --rootless``

-  The –root parameter tells runc where to store the container state. It
   must be writable by the user.

``runc --root /tmp/runc run mycontainerid``

通过Supervisor运行runc
======================

Supervisors \`[Unit] Description=Start My Container [Service]
Type=forking

ExecStart=/usr/local/sbin/runc run -d –pid-file /run/mycontainerid.pid
mycontainerid

ExecStopPost=/usr/local/sbin/runc delete mycontainerid
WorkingDirectory=/mycontainer PIDFile=/run/mycontainerid.pid [Install]
WantedBy=multi-user.target\`

%23%23%20Creating%20an%20OCI%20Bundle%0A-%20create%20the%20top%20most%20bundle%20directory%0A%60mkdir%20%2Fmycontainer%0Acd%20%2Fmycontainer%60%0A%0A-%20create%20the%20rootfs%20directory%0A%60mkdir%20rootfs%60%0A%0A-%20export%20busybox%20via%20Docker%20into%20the%20rootfs%20directory%0A%60docker%20export%20%24(docker%20create%20busybox)%20%7C%20tar%20-C%20rootfs%20-xvf%20-%60%0A%0A%E5%91%BD%E4%BB%A4%60runc%20spec%60%E4%BC%9A%E7%94%9F%E6%88%90%E5%9F%BA%E6%9C%AC%E7%9A%84%E6%A8%A1%E7%89%88%E6%96%87%E4%BB%B6config.json%0A%0A%23%23%20Running%20Containers%0A1.%20%E4%BB%A5root%E7%94%A8%E6%88%B7%E8%BF%90%E8%A1%8C%0A-%20run%20as%20root%0A%60cd%20%2Fmycontainer%0Arunc%20run%20mycontainerid%60%0AIf%20you%20used%20the%20unmodified%20runc%20spec%20template%20this%20should%20give%20you%20a%20sh%20session%20inside%20the%20container.%0A%0AThe%20second%20way%20to%20start%20a%20container%20is%20using%20the%20specs%20lifecycle%20operations.%20This%20gives%20you%20more%20power%20over%20how%20the%20container%20is%20created%20and%20managed%20while%20it%20is%20running.%20This%20will%20also%20launch%20the%20container%20in%20the%20background%20so%20you%20will%20have%20to%20edit%20the%20config.json%20to%20remove%20the%20terminal%20setting%20for%20the%20simple%20examples%20here.%20Your%20process%20field%20in%20the%20config.json%20should%20look%20like%20this%20below%20with%20%22terminal%22%3A%20false%20and%20%60%22args%22%3A%20%5B%22sleep%22%2C%20%225%22%5D.%0A%0A%22process%22%3A%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22terminal%22%3A%20false%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22user%22%3A%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22uid%22%3A%200%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22gid%22%3A%200%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22args%22%3A%20%5B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22sleep%22%2C%20%225%22%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%5D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22env%22%3A%20%5B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22PATH%3D%2Fusr%2Flocal%2Fsbin%3A%2Fusr%2Flocal%2Fbin%3A%2Fusr%2Fsbin%3A%2Fusr%2Fbin%3A%2Fsbin%3A%2Fbin%22%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22TERM%3Dxterm%22%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%5D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22cwd%22%3A%20%22%2F%22%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22capabilities%22%3A%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22bounding%22%3A%20%5B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22CAP_AUDIT_WRITE%22%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22CAP_KILL%22%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22CAP_NET_BIND_SERVICE%22%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%5D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22effective%22%3A%20%5B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22CAP_AUDIT_WRITE%22%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22CAP_KILL%22%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22CAP_NET_BIND_SERVICE%22%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%5D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22inheritable%22%3A%20%5B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22CAP_AUDIT_WRITE%22%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22CAP_KILL%22%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22CAP_NET_BIND_SERVICE%22%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%5D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22permitted%22%3A%20%5B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22CAP_AUDIT_WRITE%22%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22CAP_KILL%22%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22CAP_NET_BIND_SERVICE%22%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%5D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22ambient%22%3A%20%5B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22CAP_AUDIT_WRITE%22%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22CAP_KILL%22%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22CAP_NET_BIND_SERVICE%22%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%5D%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22rlimits%22%3A%20%5B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22type%22%3A%20%22RLIMIT_NOFILE%22%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22hard%22%3A%201024%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22soft%22%3A%201024%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%5D%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22noNewPrivileges%22%3A%20true%0A%20%20%20%20%20%20%20%20%7D%2C%60%0A-%20run%20as%20root%0Acd%20%2Fmycontainer%0Arunc%20create%20mycontainerid%0A%0A-%20view%20the%20container%20is%20created%20and%20in%20the%20%22created%22%20state%0Arunc%20list%0A%0A-%20start%20the%20process%20inside%20the%20container%0Arunc%20start%20mycontainerid%0A%0A-%20after%205%20seconds%20view%20that%20the%20container%20has%20exited%20and%20is%20now%20in%20the%20stopped%20state%0Arunc%20list%0A%0A-%20now%20delete%20the%20container%0Arunc%20delete%20mycontainerid%0A%0A%0A2.%20%E4%BB%A5%E6%99%AE%E9%80%9A%E7%94%A8%E6%88%B7%E8%BF%90%E8%A1%8C%0A-%20Same%20as%20the%20first%20example%0A%60%60%60%0Amkdir%20\ :sub:`%2Fmycontainer%0Acd%20`\ %2Fmycontainer%0Amkdir%20rootfs%0Adocker%20export%20%24(docker%20create%20busybox)%20%7C%20tar%20-C%20rootfs%20-xvf%20-%0A%60%60%60%0A%0A-%20The%20–rootless%20parameter%20instructs%20runc%20spec%20to%20generate%20a%20configuration%20for%20a%20rootless%20container%2C%20which%20will%20allow%20you%20to%20run%20the%20container%20as%20a%20non-root%20user.%0A%60runc%20spec%20–rootless%60%0A%0A-%20The%20–root%20parameter%20tells%20runc%20where%20to%20store%20the%20container%20state.%20It%20must%20be%20writable%20by%20the%20user.%0A%60runc%20–root%20%2Ftmp%2Frunc%20run%20mycontainerid%60%0A%0A%23%23%20%E9%80%9A%E8%BF%87Supervisor%E8%BF%90%E8%A1%8Crunc%0ASupervisors%0A%60%5BUnit%5D%0ADescription%3DStart%20My%20Container%0A%0A%5BService%5D%0AType%3Dforking%0AExecStart%3D%2Fusr%2Flocal%2Fsbin%2Frunc%20run%20-d%20–pid-file%20%2Frun%2Fmycontainerid.pid%20mycontainerid%0AExecStopPost%3D%2Fusr%2Flocal%2Fsbin%2Frunc%20delete%20mycontainerid%0AWorkingDirectory%3D%2Fmycontainer%0APIDFile%3D%2Frun%2Fmycontainerid.pid%0A%0A%5BInstall%5D%0AWantedBy%3Dmulti-user.target%60
