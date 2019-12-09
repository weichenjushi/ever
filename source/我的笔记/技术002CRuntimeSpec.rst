技术002CRuntimeSpec

OCI制定了操作系统和应用容器之间交互的规范。

该文档主要概述配置、可执行环境和容器的生命周期。

-  配置存放在文件config.json中，该文件中包含容器支持的平台和创建容器所需字段的详细信息。
-  可执行环境。可执行环境确保容器中的应用有一致性环境在容器的整个生命周期中。

Platforms 各平台的规范如下 Platforms defined by this specification are:

-  linux: runtime.md, config.md, config-linux.md, and runtime-linux.md.
-  solaris: runtime.md, config.md, and config-solaris.md.
-  windows: runtime.md, config.md, and config-windows.md.
-  vm: runtime.md, config.md, and config-vm.md.

Runtime.md
