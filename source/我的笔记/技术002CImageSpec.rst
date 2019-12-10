技术002CImageSpec

技术002CImageSpec
=================

[]

Image Manifest - a document describing the components that make up a
container image 描述了容器镜像的组件

-  mediaType

application/vnd.oci.image.layer.v1.tar
application/vnd.oci.image.layer.v1.tar+gzip
application/vnd.oci.image.layer.nondistributable.v1.tar
application/vnd.oci.image.layer.nondistributable.v1.tar+gzip Image Index
- an annotated index of image manifests Image Layout - a filesystem
layout representing the contents of an image

-  镜像每层的内容可能类似tar,zip的压缩文件，nfs的共享文件或者http/ftp/rsync的网络文件
-  +gzip Media Types

The media type application/vnd.oci.image.layer.v1.tar+gzip represents an
application/vnd.oci.image.layer.v1.tar payload which has been compressed
with gzip.

The media type
application/vnd.oci.image.layer.nondistributable.v1.tar+gzip represents
an application/vnd.oci.image.layer.nondistributable.v1.tar payload which
has been compressed with gzip.

-  +zstd Media Types

The media type application/vnd.oci.image.layer.v1.tar+zstd represents an
application/vnd.oci.image.layer.v1.tar payload which has been compressed
with zstd.

The media type
application/vnd.oci.image.layer.nondistributable.v1.tar+zstd represents
an application/vnd.oci.image.layer.nondistributable.v1.tar payload which
has been compressed with zstd.

-  Distributable Format

Layer Changesets for the media type
application/vnd.oci.image.layer.v1.tar MUST be packaged in tar archive.

Layer Changesets for the media type
application/vnd.oci.image.layer.v1.tar MUST NOT include duplicate
entries for file paths in the resulting tar archive.

-  File Types支持的文件类型

regular files directories sockets symbolic links block devices character
devices FIFOs

-  Non-Distributable Layers

Due to legal requirements, certain layers may not be regularly
distributable. Such “non-distributable” layers are typically downloaded
directly from a distributor but never uploaded.

Non-distributable layers SHOULD be tagged with an alternative mediatype
of application/vnd.oci.image.layer.nondistributable.v1.tar.
Implementations SHOULD NOT upload layers tagged with this media type;
however, such a media type SHOULD NOT affect whether an implementation
downloads the layer.

Descriptors referencing non-distributable layers MAY include urls for
downloading these layers directly; however, the presence of the urls
field SHOULD NOT be used to determine whether or not a layer is
non-distributable.

Filesystem Layer - a changeset that describes a container’s filesystem
该文档描述了如何序列化文件和文件的变化比如移除文件放到一个blob称layer。

Image Configuration - a document determining layer ordering and
configuration of the image suitable for translation into a runtime
bundle 定义了layer的顺序和将image转换成运行的bundle

-  OCI
   镜像是一个有序的文件层的集合和在容器运行环境中根据执行参数而改变。
-  Layer

Image filesystems are composed of layers.

Each layer represents a set of filesystem changes in a tar-based layer
format, recording files to be added, changed, or deleted relative to its
parent layer. 每层是根据父层记录文件的增加修改和删除。

Layers do not have configuration metadata such as environment variables
or default arguments - these are properties of the image as a whole
rather than any particular layer.
层没有配置环境变量等元数据，环境变量是针对整个镜像而不是单个层的。

Using a layer-based or union filesystem such as AUFS, or by computing
the diff from filesystem snapshots, the filesystem changeset can be used
to present a series of image layers as if they were one cohesive
filesystem.
通过计算文件系统快照的不同，文件系统改变集合代表一系列的镜像层。

-  Image JSON

Each image has an associated JSON structure which describes some basic
information about the image such as date created, author, as well as
execution/runtime configuration like its entrypoint, default arguments,
networking, and volumes.
每个镜像文件有一个JSON结构的文件，信息包括创建时间，作者，以及entrypoint和默认的参数、网络、volume等。

The JSON structure also references a cryptographic hash of each layer
used by the image, and provides history information for those layers.

This JSON is considered to be immutable, because changing it would
change the computed ImageID.

Changing it means creating a new derived image, instead of changing the
existing image.

-  Layer DiffID
-  Layer ChainID 根据Ln层生成Ln+1层的chainID

ChainID(L₀) = DiffID(L₀) ChainID(L₀|…|Lₙ₋₁|Lₙ) =
Digest(ChainID(L₀|…|Lₙ₋₁) + " " + DiffID(Lₙ))

Let’s expand the definition of ChainID(A|B|C) to explore its internal
structure:

ChainID(A) = DiffID(A) ChainID(A|B) = Digest(ChainID(A) + " " +
DiffID(B)) ChainID(A|B|C) = Digest(ChainID(A|B) + " " + DiffID(C))

-  ImageID 每个镜像ID是它的配置文件的SHA256的hash值。
-  Properties

Note: Any OPTIONAL field MAY also be set to null, which is equivalent to
being absent.

created string, OPTIONAL

An combined date and time at which the image was created, formatted as
defined by RFC 3339, section 5.6.

author string, OPTIONAL

Gives the name and/or email address of the person or entity which
created and is responsible for maintaining the image.

architecture string, REQUIRED

The CPU architecture which the binaries in this image are built to run
on. Configurations SHOULD use, and implementations SHOULD understand,
values listed in the Go Language document for GOARCH.

os string, REQUIRED

The name of the operating system which the image is built to run on.
Configurations SHOULD use, and implementations SHOULD understand, values
listed in the Go Language document for GOOS.

config object, OPTIONAL

The execution parameters which SHOULD be used as a base when running a
container using the image. This field can be null, in which case any
execution parameters should be specified at creation of the container.

User string, OPTIONAL

The username or UID which is a platform-specific structure that allows
specific control over which user the process run as. This acts as a
default value to use when the value is not specified when creating a
container. For Linux based systems, all of the following are valid:
user, uid, user:group, uid:gid, uid:group, user:gid. If group/gid is not
specified, the default group and supplementary groups of the given
user/uid in /etc/passwd from the container are applied.

ExposedPorts object, OPTIONAL

A set of ports to expose from a container running this image. Its keys
can be in the format of: port/tcp, port/udp, port with the default
protocol being tcp if not specified. These values act as defaults and
are merged with any specified when creating a container. NOTE: This JSON
structure value is unusual because it is a direct JSON serialization of
the Go type map[string]struct{} and is represented in JSON as an object
mapping its keys to an empty object.

Env array of strings, OPTIONAL

Entries are in the format of VARNAME=VARVALUE. These values act as
defaults and are merged with any specified when creating a container.

Entrypoint array of strings, OPTIONAL

A list of arguments to use as the command to execute when the container
starts. These values act as defaults and may be replaced by an
entrypoint specified when creating a container.

Cmd array of strings, OPTIONAL

Default arguments to the entrypoint of the container. These values act
as defaults and may be replaced by any specified when creating a
container. If an Entrypoint value is not specified, then the first entry
of the Cmd array SHOULD be interpreted as the executable to run.

Volumes object, OPTIONAL

A set of directories describing where the process is likely write data
specific to a container instance. NOTE: This JSON structure value is
unusual because it is a direct JSON serialization of the Go type
map[string]struct{} and is represented in JSON as an object mapping its
keys to an empty object.

WorkingDir string, OPTIONAL

Sets the current working directory of the entrypoint process in the
container. This value acts as a default and may be replaced by a working
directory specified when creating a container.

Labels object, OPTIONAL

The field contains arbitrary metadata for the container. This property
MUST use the annotation rules.

StopSignal string, OPTIONAL

The field contains the system call signal that will be sent to the
container to exit. The signal can be a signal name in the format
SIGNAME, for instance SIGKILL or SIGRTMIN+3.

rootfs object, REQUIRED

The rootfs key references the layer content addresses used by the image.
This makes the image config hash depend on the filesystem hash.

type string, REQUIRED

MUST be set to layers. Implementations MUST generate an error if they
encounter a unknown value while verifying or unpacking an image.

diff_ids array of strings, REQUIRED An array of layer content hashes
(DiffIDs), in order from first to last. history array of objects,
OPTIONAL

Describes the history of each layer. The array is ordered from first to
last. The object has the following fields:

created string, OPTIONAL

A combined date and time at which the layer was created, formatted as
defined by RFC 3339, section 5.6.

author string, OPTIONAL The author of the build point. created_by
string, OPTIONAL The command which created the layer. comment string,
OPTIONAL A custom message set when creating the layer. empty_layer
boolean, OPTIONAL

This field is used to mark if the history item created a filesystem
diff. It is set to true if this history item doesn’t correspond to an
actual layer in the rootfs section (for example, Dockerfile’s ENV
command results in no change to the filesystem).

Any extra fields in the Image JSON struct are considered implementation
specific and MUST be ignored by any implementations which are unable to
interpret them.

Whitespace is OPTIONAL and implementations MAY have compact JSON with no
whitespace.

Conversion - a document describing how this translation should occur

Descriptor - a reference that describes the type, metadata and content
address of referenced content

-  OCI镜像的层是通过Merkle树有向无循环调度来组织的。

-  

%23%20%E6%8A%80%E6%9C%AF002CImageSpec%0A%5BTOC%5D%0A%0AImage%20Manifest%20-%20a%20document%20describing%20the%20components%20that%20make%20up%20a%20container%20image%20%E6%8F%8F%E8%BF%B0%E4%BA%86%E5%AE%B9%E5%99%A8%E9%95%9C%E5%83%8F%E7%9A%84%E7%BB%84%E4%BB%B6%0A\ *%20mediaType%0Aapplication%2Fvnd.oci.image.layer.v1.tar%0Aapplication%2Fvnd.oci.image.layer.v1.tar%2Bgzip%0Aapplication%2Fvnd.oci.image.layer.nondistributable.v1.tar%0Aapplication%2Fvnd.oci.image.layer.nondistributable.v1.tar%2Bgzip%0AImage%20Index%20-%20an%20annotated%20index%20of%20image%20manifests%20%0AImage%20Layout%20-%20a%20filesystem%20layout%20representing%20the%20contents%20of%20an%20image%0A*\ %20%E9%95%9C%E5%83%8F%E6%AF%8F%E5%B1%82%E7%9A%84%E5%86%85%E5%AE%B9%E5%8F%AF%E8%83%BD%E7%B1%BB%E4%BC%BCtar%2Czip%E7%9A%84%E5%8E%8B%E7%BC%A9%E6%96%87%E4%BB%B6%EF%BC%8Cnfs%E7%9A%84%E5%85%B1%E4%BA%AB%E6%96%87%E4%BB%B6%E6%88%96%E8%80%85http%2Fftp%2Frsync%E7%9A%84%E7%BD%91%E7%BB%9C%E6%96%87%E4%BB%B6%0A\ *%20%2Bgzip%20Media%20Types%0AThe%20media%20type%20application%2Fvnd.oci.image.layer.v1.tar%2Bgzip%20represents%20an%20application%2Fvnd.oci.image.layer.v1.tar%20payload%20which%20has%20been%20compressed%20with%20gzip.%0AThe%20media%20type%20application%2Fvnd.oci.image.layer.nondistributable.v1.tar%2Bgzip%20represents%20an%20application%2Fvnd.oci.image.layer.nondistributable.v1.tar%20payload%20which%20has%20been%20compressed%20with%20gzip.%0A*\ %20%2Bzstd%20Media%20Types%0AThe%20media%20type%20application%2Fvnd.oci.image.layer.v1.tar%2Bzstd%20represents%20an%20application%2Fvnd.oci.image.layer.v1.tar%20payload%20which%20has%20been%20compressed%20with%20zstd.%0AThe%20media%20type%20application%2Fvnd.oci.image.layer.nondistributable.v1.tar%2Bzstd%20represents%20an%20application%2Fvnd.oci.image.layer.nondistributable.v1.tar%20payload%20which%20has%20been%20compressed%20with%20zstd.%0A\ *%20Distributable%20Format%0ALayer%20Changesets%20for%20the%20media%20type%20application%2Fvnd.oci.image.layer.v1.tar%20MUST%20be%20packaged%20in%20tar%20archive.%0ALayer%20Changesets%20for%20the%20media%20type%20application%2Fvnd.oci.image.layer.v1.tar%20MUST%20NOT%20include%20duplicate%20entries%20for%20file%20paths%20in%20the%20resulting%20tar%20archive.%0A*\ %20File%20Types%E6%94%AF%E6%8C%81%E7%9A%84%E6%96%87%E4%BB%B6%E7%B1%BB%E5%9E%8B%0Aregular%20files%0Adirectories%0Asockets%0Asymbolic%20links%0Ablock%20devices%0Acharacter%20devices%0AFIFOs%0A\ *%20Non-Distributable%20Layers%0ADue%20to%20legal%20requirements%2C%20certain%20layers%20may%20not%20be%20regularly%20distributable.%20Such%20%22non-distributable%22%20layers%20are%20typically%20downloaded%20directly%20from%20a%20distributor%20but%20never%20uploaded.%0ANon-distributable%20layers%20SHOULD%20be%20tagged%20with%20an%20alternative%20mediatype%20of%20application%2Fvnd.oci.image.layer.nondistributable.v1.tar.%20Implementations%20SHOULD%20NOT%20upload%20layers%20tagged%20with%20this%20media%20type%3B%20however%2C%20such%20a%20media%20type%20SHOULD%20NOT%20affect%20whether%20an%20implementation%20downloads%20the%20layer.%0ADescriptors%20referencing%20non-distributable%20layers%20MAY%20include%20urls%20for%20downloading%20these%20layers%20directly%3B%20however%2C%20the%20presence%20of%20the%20urls%20field%20SHOULD%20NOT%20be%20used%20to%20determine%20whether%20or%20not%20a%20layer%20is%20non-distributable.%0AFilesystem%20Layer%20-%20a%20changeset%20that%20describes%20a%20container’s%20filesystem%20%E8%AF%A5%E6%96%87%E6%A1%A3%E6%8F%8F%E8%BF%B0%E4%BA%86%E5%A6%82%E4%BD%95%E5%BA%8F%E5%88%97%E5%8C%96%E6%96%87%E4%BB%B6%E5%92%8C%E6%96%87%E4%BB%B6%E7%9A%84%E5%8F%98%E5%8C%96%E6%AF%94%E5%A6%82%E7%A7%BB%E9%99%A4%E6%96%87%E4%BB%B6%E6%94%BE%E5%88%B0%E4%B8%80%E4%B8%AAblob%E7%A7%B0layer%E3%80%82%0AImage%20Configuration%20-%20a%20document%20determining%20layer%20ordering%20and%20configuration%20of%20the%20image%20suitable%20for%20translation%20into%20a%20runtime%20bundle%20%E5%AE%9A%E4%B9%89%E4%BA%86layer%E7%9A%84%E9%A1%BA%E5%BA%8F%E5%92%8C%E5%B0%86image%E8%BD%AC%E6%8D%A2%E6%88%90%E8%BF%90%E8%A1%8C%E7%9A%84bundle%0A*\ %20OCI%20%E9%95%9C%E5%83%8F%E6%98%AF%E4%B8%80%E4%B8%AA%E6%9C%89%E5%BA%8F%E7%9A%84%E6%96%87%E4%BB%B6%E5%B1%82%E7%9A%84%E9%9B%86%E5%90%88%E5%92%8C%E5%9C%A8%E5%AE%B9%E5%99%A8%E8%BF%90%E8%A1%8C%E7%8E%AF%E5%A2%83%E4%B8%AD%E6%A0%B9%E6%8D%AE%E6%89%A7%E8%A1%8C%E5%8F%82%E6%95%B0%E8%80%8C%E6%94%B9%E5%8F%98%E3%80%82%0A\ *%20Layer%0AImage%20filesystems%20are%20composed%20of%20layers.%0AEach%20layer%20represents%20a%20set%20of%20filesystem%20changes%20in%20a%20tar-based%20layer%20format%2C%20recording%20files%20to%20be%20added%2C%20changed%2C%20or%20deleted%20relative%20to%20its%20parent%20layer.%20%E6%AF%8F%E5%B1%82%E6%98%AF%E6%A0%B9%E6%8D%AE%E7%88%B6%E5%B1%82%E8%AE%B0%E5%BD%95%E6%96%87%E4%BB%B6%E7%9A%84%E5%A2%9E%E5%8A%A0%E4%BF%AE%E6%94%B9%E5%92%8C%E5%88%A0%E9%99%A4%E3%80%82%0ALayers%20do%20not%20have%20configuration%20metadata%20such%20as%20environment%20variables%20or%20default%20arguments%20-%20these%20are%20properties%20of%20the%20image%20as%20a%20whole%20rather%20than%20any%20particular%20layer.%20%E5%B1%82%E6%B2%A1%E6%9C%89%E9%85%8D%E7%BD%AE%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F%E7%AD%89%E5%85%83%E6%95%B0%E6%8D%AE%EF%BC%8C%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F%E6%98%AF%E9%92%88%E5%AF%B9%E6%95%B4%E4%B8%AA%E9%95%9C%E5%83%8F%E8%80%8C%E4%B8%8D%E6%98%AF%E5%8D%95%E4%B8%AA%E5%B1%82%E7%9A%84%E3%80%82%0AUsing%20a%20layer-based%20or%20union%20filesystem%20such%20as%20AUFS%2C%20or%20by%20computing%20the%20diff%20from%20filesystem%20snapshots%2C%20the%20filesystem%20changeset%20can%20be%20used%20to%20present%20a%20series%20of%20image%20layers%20as%20if%20they%20were%20one%20cohesive%20filesystem.%20%E9%80%9A%E8%BF%87%E8%AE%A1%E7%AE%97%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F%E5%BF%AB%E7%85%A7%E7%9A%84%E4%B8%8D%E5%90%8C%EF%BC%8C%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F%E6%94%B9%E5%8F%98%E9%9B%86%E5%90%88%E4%BB%A3%E8%A1%A8%E4%B8%80%E7%B3%BB%E5%88%97%E7%9A%84%E9%95%9C%E5%83%8F%E5%B1%82%E3%80%82%0A*\ %20Image%20JSON%0AEach%20image%20has%20an%20associated%20JSON%20structure%20which%20describes%20some%20basic%20information%20about%20the%20image%20such%20as%20date%20created%2C%20author%2C%20as%20well%20as%20execution%2Fruntime%20configuration%20like%20its%20entrypoint%2C%20default%20arguments%2C%20networking%2C%20and%20volumes.%20%E6%AF%8F%E4%B8%AA%E9%95%9C%E5%83%8F%E6%96%87%E4%BB%B6%E6%9C%89%E4%B8%80%E4%B8%AAJSON%E7%BB%93%E6%9E%84%E7%9A%84%E6%96%87%E4%BB%B6%EF%BC%8C%E4%BF%A1%E6%81%AF%E5%8C%85%E6%8B%AC%E5%88%9B%E5%BB%BA%E6%97%B6%E9%97%B4%EF%BC%8C%E4%BD%9C%E8%80%85%EF%BC%8C%E4%BB%A5%E5%8F%8Aentrypoint%E5%92%8C%E9%BB%98%E8%AE%A4%E7%9A%84%E5%8F%82%E6%95%B0%E3%80%81%E7%BD%91%E7%BB%9C%E3%80%81volume%E7%AD%89%E3%80%82%0AThe%20JSON%20structure%20also%20references%20a%20cryptographic%20hash%20of%20each%20layer%20used%20by%20the%20image%2C%20and%20provides%20history%20information%20for%20those%20layers.%0AThis%20JSON%20is%20considered%20to%20be%20immutable%2C%20because%20changing%20it%20would%20change%20the%20computed%20ImageID.%0AChanging%20it%20means%20creating%20a%20new%20derived%20image%2C%20instead%20of%20changing%20the%20existing%20image.%0A\ *%20Layer%20DiffID%0A*\ %20Layer%20ChainID%20%E6%A0%B9%E6%8D%AELn%E5%B1%82%E7%94%9F%E6%88%90Ln%2B1%E5%B1%82%E7%9A%84chainID%0AChainID(L%E2%82%80)%20%3D%20%20DiffID(L%E2%82%80)%0AChainID(L%E2%82%80%7C…%7CL%E2%82%99%E2%82%8B%E2%82%81%7CL%E2%82%99)%20%3D%20Digest(ChainID(L%E2%82%80%7C…%7CL%E2%82%99%E2%82%8B%E2%82%81)%20%2B%20%22%20%22%20%2B%20DiffID(L%E2%82%99))%0ALet’s%20expand%20the%20definition%20of%20ChainID(A%7CB%7CC)%20to%20explore%20its%20internal%20structure%3A%0AChainID(A)%20%3D%20DiffID(A)%0AChainID(A%7CB)%20%3D%20Digest(ChainID(A)%20%2B%20%22%20%22%20%2B%20DiffID(B))%0AChainID(A%7CB%7CC)%20%3D%20Digest(ChainID(A%7CB)%20%2B%20%22%20%22%20%2B%20DiffID(C))%0A\ *%20ImageID%20%E6%AF%8F%E4%B8%AA%E9%95%9C%E5%83%8FID%E6%98%AF%E5%AE%83%E7%9A%84%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E7%9A%84SHA256%E7%9A%84hash%E5%80%BC%E3%80%82%0A*\ %20Properties%0ANote%3A%20Any%20OPTIONAL%20field%20MAY%20also%20be%20set%20to%20null%2C%20which%20is%20equivalent%20to%20being%20absent.%0Acreated%20string%2C%20OPTIONAL%0AAn%20combined%20date%20and%20time%20at%20which%20the%20image%20was%20created%2C%20formatted%20as%20defined%20by%20RFC%203339%2C%20section%205.6.%0Aauthor%20string%2C%20OPTIONAL%0AGives%20the%20name%20and%2For%20email%20address%20of%20the%20person%20or%20entity%20which%20created%20and%20is%20responsible%20for%20maintaining%20the%20image.%0Aarchitecture%20string%2C%20REQUIRED%0AThe%20CPU%20architecture%20which%20the%20binaries%20in%20this%20image%20are%20built%20to%20run%20on.%20Configurations%20SHOULD%20use%2C%20and%20implementations%20SHOULD%20understand%2C%20values%20listed%20in%20the%20Go%20Language%20document%20for%20GOARCH.%0Aos%20string%2C%20REQUIRED%0AThe%20name%20of%20the%20operating%20system%20which%20the%20image%20is%20built%20to%20run%20on.%20Configurations%20SHOULD%20use%2C%20and%20implementations%20SHOULD%20understand%2C%20values%20listed%20in%20the%20Go%20Language%20document%20for%20GOOS.%0Aconfig%20object%2C%20OPTIONAL%0AThe%20execution%20parameters%20which%20SHOULD%20be%20used%20as%20a%20base%20when%20running%20a%20container%20using%20the%20image.%20This%20field%20can%20be%20null%2C%20in%20which%20case%20any%20execution%20parameters%20should%20be%20specified%20at%20creation%20of%20the%20container.%0AUser%20string%2C%20OPTIONAL%0AThe%20username%20or%20UID%20which%20is%20a%20platform-specific%20structure%20that%20allows%20specific%20control%20over%20which%20user%20the%20process%20run%20as.%20This%20acts%20as%20a%20default%20value%20to%20use%20when%20the%20value%20is%20not%20specified%20when%20creating%20a%20container.%20For%20Linux%20based%20systems%2C%20all%20of%20the%20following%20are%20valid%3A%20user%2C%20uid%2C%20user%3Agroup%2C%20uid%3Agid%2C%20uid%3Agroup%2C%20user%3Agid.%20If%20group%2Fgid%20is%20not%20specified%2C%20the%20default%20group%20and%20supplementary%20groups%20of%20the%20given%20user%2Fuid%20in%20%2Fetc%2Fpasswd%20from%20the%20container%20are%20applied.%0AExposedPorts%20object%2C%20OPTIONAL%0AA%20set%20of%20ports%20to%20expose%20from%20a%20container%20running%20this%20image.%20Its%20keys%20can%20be%20in%20the%20format%20of%3A%20port%2Ftcp%2C%20port%2Fudp%2C%20port%20with%20the%20default%20protocol%20being%20tcp%20if%20not%20specified.%20These%20values%20act%20as%20defaults%20and%20are%20merged%20with%20any%20specified%20when%20creating%20a%20container.%20NOTE%3A%20This%20JSON%20structure%20value%20is%20unusual%20because%20it%20is%20a%20direct%20JSON%20serialization%20of%20the%20Go%20type%20map%5Bstring%5Dstruct%7B%7D%20and%20is%20represented%20in%20JSON%20as%20an%20object%20mapping%20its%20keys%20to%20an%20empty%20object.%0AEnv%20array%20of%20strings%2C%20OPTIONAL%0AEntries%20are%20in%20the%20format%20of%20VARNAME%3DVARVALUE.%20These%20values%20act%20as%20defaults%20and%20are%20merged%20with%20any%20specified%20when%20creating%20a%20container.%0AEntrypoint%20array%20of%20strings%2C%20OPTIONAL%0AA%20list%20of%20arguments%20to%20use%20as%20the%20command%20to%20execute%20when%20the%20container%20starts.%20These%20values%20act%20as%20defaults%20and%20may%20be%20replaced%20by%20an%20entrypoint%20specified%20when%20creating%20a%20container.%0ACmd%20array%20of%20strings%2C%20OPTIONAL%0ADefault%20arguments%20to%20the%20entrypoint%20of%20the%20container.%20These%20values%20act%20as%20defaults%20and%20may%20be%20replaced%20by%20any%20specified%20when%20creating%20a%20container.%20If%20an%20Entrypoint%20value%20is%20not%20specified%2C%20then%20the%20first%20entry%20of%20the%20Cmd%20array%20SHOULD%20be%20interpreted%20as%20the%20executable%20to%20run.%0AVolumes%20object%2C%20OPTIONAL%0AA%20set%20of%20directories%20describing%20where%20the%20process%20is%20likely%20write%20data%20specific%20to%20a%20container%20instance.%20NOTE%3A%20This%20JSON%20structure%20value%20is%20unusual%20because%20it%20is%20a%20direct%20JSON%20serialization%20of%20the%20Go%20type%20map%5Bstring%5Dstruct%7B%7D%20and%20is%20represented%20in%20JSON%20as%20an%20object%20mapping%20its%20keys%20to%20an%20empty%20object.%0AWorkingDir%20string%2C%20OPTIONAL%0ASets%20the%20current%20working%20directory%20of%20the%20entrypoint%20process%20in%20the%20container.%20This%20value%20acts%20as%20a%20default%20and%20may%20be%20replaced%20by%20a%20working%20directory%20specified%20when%20creating%20a%20container.%0ALabels%20object%2C%20OPTIONAL%0AThe%20field%20contains%20arbitrary%20metadata%20for%20the%20container.%20This%20property%20MUST%20use%20the%20annotation%20rules.%0AStopSignal%20string%2C%20OPTIONAL%0AThe%20field%20contains%20the%20system%20call%20signal%20that%20will%20be%20sent%20to%20the%20container%20to%20exit.%20The%20signal%20can%20be%20a%20signal%20name%20in%20the%20format%20SIGNAME%2C%20for%20instance%20SIGKILL%20or%20SIGRTMIN%2B3.%0Arootfs%20object%2C%20REQUIRED%0AThe%20rootfs%20key%20references%20the%20layer%20content%20addresses%20used%20by%20the%20image.%20This%20makes%20the%20image%20config%20hash%20depend%20on%20the%20filesystem%20hash.%0Atype%20string%2C%20REQUIRED%0AMUST%20be%20set%20to%20layers.%20Implementations%20MUST%20generate%20an%20error%20if%20they%20encounter%20a%20unknown%20value%20while%20verifying%20or%20unpacking%20an%20image.%0Adiff_ids%20array%20of%20strings%2C%20REQUIRED%0AAn%20array%20of%20layer%20content%20hashes%20(DiffIDs)%2C%20in%20order%20from%20first%20to%20last.%0Ahistory%20array%20of%20objects%2C%20OPTIONAL%0ADescribes%20the%20history%20of%20each%20layer.%20The%20array%20is%20ordered%20from%20first%20to%20last.%20The%20object%20has%20the%20following%20fields%3A%0Acreated%20string%2C%20OPTIONAL%0AA%20combined%20date%20and%20time%20at%20which%20the%20layer%20was%20created%2C%20formatted%20as%20defined%20by%20RFC%203339%2C%20section%205.6.%0Aauthor%20string%2C%20OPTIONAL%0AThe%20author%20of%20the%20build%20point.%0Acreated_by%20string%2C%20OPTIONAL%0AThe%20command%20which%20created%20the%20layer.%0Acomment%20string%2C%20OPTIONAL%0AA%20custom%20message%20set%20when%20creating%20the%20layer.%0Aempty_layer%20boolean%2C%20OPTIONAL%0AThis%20field%20is%20used%20to%20mark%20if%20the%20history%20item%20created%20a%20filesystem%20diff.%20It%20is%20set%20to%20true%20if%20this%20history%20item%20doesn’t%20correspond%20to%20an%20actual%20layer%20in%20the%20rootfs%20section%20(for%20example%2C%20Dockerfile’s%20ENV%20command%20results%20in%20no%20change%20to%20the%20filesystem).%0AAny%20extra%20fields%20in%20the%20Image%20JSON%20struct%20are%20considered%20implementation%20specific%20and%20MUST%20be%20ignored%20by%20any%20implementations%20which%20are%20unable%20to%20interpret%20them.%0AWhitespace%20is%20OPTIONAL%20and%20implementations%20MAY%20have%20compact%20JSON%20with%20no%20whitespace.%0AConversion%20-%20a%20document%20describing%20how%20this%20translation%20should%20occur%0ADescriptor%20-%20a%20reference%20that%20describes%20the%20type%2C%20metadata%20and%20content%20address%20of%20referenced%20content%0A\ *%20OCI%E9%95%9C%E5%83%8F%E7%9A%84%E5%B1%82%E6%98%AF%E9%80%9A%E8%BF%87Merkle%E6%A0%91%E6%9C%89%E5%90%91%E6%97%A0%E5%BE%AA%E7%8E%AF%E8%B0%83%E5%BA%A6%E6%9D%A5%E7%BB%84%E7%BB%87%E7%9A%84%E3%80%82%0A*\ %20%0A
