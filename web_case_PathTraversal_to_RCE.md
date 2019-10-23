#### 基础概念

Atlassian Bitbucket Data Center：一个本地的Git仓库管理系统，具有高可用性等优点，适合大型企业使用。


-----

Bitbucket：被全球数以百万计的开发人员使用。


* Bitbucket中的四种用户角色(user roles)：
  * Bitbucket User
  * Project Creator
  * Admin
  * System Admin


-----

Git Hooks：当Git特定事件发生之前或之后（如commit、push等）触发执行的那些脚本。类似于监听事件、触发器。

* 按照Git Hooks脚本所在的位置可以分为两类：
  * 1.本地Hooks，触发事件如pull、push、commit、merge等。
  * 2.服务端Hooks，触发事件如receive等。

-----

* TAR压缩包文件(TAR archive)包含了压缩包中每个文件条目(a file entry)的元信息:
  * 修改日期(modification date)
  * 用户名(user name)
  * 组名(group name)
  * 文件权限(file permissions)
  * ...

#### 漏洞信息

漏洞描述：

Atlassian Bitbucket Data Center目录穿越漏洞(CVE-2019-3397)，具体在系统中的Data Center migration tool功能，存在目录穿越漏洞。能够实现Path Traversal to RCE。实际上是对“使用GZip压缩过的TAR存档文件”的不安全提取(解压)造成的，即提取的过程中未做足够的安全处理。

漏洞影响：

一个远程攻击者(具有管理员权限的已授权的用户)可以利用这个路径遍历漏洞，将文件写入任意位置，从而导致在运行Bitbucket Data Center的某些版本的系统上，可以执行远程代码。
没有Data Center license的Bitbucket Server版本，不受影响。

影响版本：
* Bitbucket Data Center
  * 5.13.0 <= version < 5.13.6
  * 5.14.0 <= version < 5.14.4
  * 5.15.0 <= version < 5.15.3
  * 5.16.0 <= version < 5.16.3
  * 6.0.0 <= version < 6.0.3
  * 6.1.0 <= version < 6.1.2


修复建议：

升级版本 version >= 6.1.2


漏洞评级：

CVSS v3 score: 9.1 => Critical severity

* Exploitability Metrics
  * Attack Vector:Network
  * Attack Complexity:Low
  * Privileges Required:High
  * User Interaction:None


利用条件：

具有Admin角色权限的用户(攻击者)，即可使用Data Center Migration tool将任意文件(如webshell)上传到任意目录。


所在功能：

存在该漏洞的功能是Data Center Migration tool，它是在Bitbucket Server的5.14版本中引入的，可以利用Bitbucket Data Center license进行利用。

RCE原理：

利用该目录穿越漏洞，看起来只能任意文件上传，所以攻击者借助Git Hooks可以释放(解压)一个Git hook，当这个存储库中发生了特定事件（如pull或push请求），会执行该Git hook，即执行脚本，从而实现RCE。

#### 漏洞分析

数据中心迁移工具(Data Center Migration tool)，能够使管理员(Admins)或系统管理员(System Admins)将若干个Git仓库从Bitbucket Server迁移到Bitbucket Data Center。

* 迁移过程
  * 1.把仓库导出：管理员必须先从Bitbucket Server实例中导出Git仓库。【得到一个TAR存档压缩包】
  * 2.把仓库导入：把Git仓库导入到目标系统(Bitbucket Data Center)。[Importing - Atlassian Documentation](https://confluence.atlassian.com/bitbucketserver/importing-957497836.html)
 【漏洞利用】

在导出过程中，会创建具有以下结构的TAR存档压缩包：

```
_/repository/hierarchy_begin/c3b3efc5cb93609ad4fc
_/repository/hierarchy_end/c3b3efc5cb93609ad4fc
com.atlassian.bitbucket.server.bitbucket-instance-migration_instanceDetails/instance-details.json.atl.gz
com.atlassian.bitbucket.server.bitbucket-instance-migration_metadata/project_68/project.json.atl.gz
com.atlassian.bitbucket.server.bitbucket-instance-migration_metadata/project_68/repository_59.json.atl.gz
com.atlassian.bitbucket.server.bitbucket-instance-migration_permissions/project/68/all-permissions.json.atl.gz
com.atlassian.bitbucket.server.bitbucket-instance-migration_permissions/project/68/permissions.json.atl.gz
com.atlassian.bitbucket.server.bitbucket-instance-migration_permissions/repository/59/permissions.json.atl.gz
com.atlassian.bitbucket.server.bitbucket-git_git/repositories/59/hooks/hooks.atl.tar.atl.gz
com.atlassian.bitbucket.server.bitbucket-git_git/repositories/59/contents/objects.atl.tar
com.atlassian.bitbucket.server.bitbucket-git_git/repositories/59/metadata/metadata.atl.tar.atl.gz
com.atlassian.bitbucket.server.bitbucket-git-lfs_gitLfsSettings/59/git-lfs-settings.json.atl.gz
```

可见这个TAR存档压缩包中包含了多个压缩包文件(GZIP和TAR)。

尤其要注意这个文件`hooks.atl.tar.atl.gz`，它包含了Git hooks，这些hook其实是脚本，每当Git仓库中发生特定事件时执行对应脚本。

通过构造包含`../`的TAR压缩包，即可在导入过程中导致远程代码执行（下面具体解释）。


**正常情况**

导入Git仓库到目标系统(Bitbucket Data Center)的过程中，文件hooks.atl.tar.atl.gz中的Git hooks按照预期都存储在目录`${BITBUCKET_DATA}/shared/data/repositories/${REPO_ID}/imported-hooks/`中，但是这些Git hooks都不会被执行，因为Git仓库的正常Git hooks都存储在目录`${BITBUCKET_DATA}/shared/data/repositories/${REPO_ID}/hooks/`中。

**目录穿越**

这个目录穿越漏洞，实际上是对“使用GZip压缩过的TAR存档文件”的不安全提取(解压)造成的。

在非正常情况下，攻击者控制了迁移过程中第一步的结果——导出的“TAR存档压缩包”，其实是控制了其中的关键文件`hooks.atl.tar.atl.gz`的内容，通过构造特制的TAR存档压缩包从而利用目录穿越漏洞，从预期的解压目标目录中做穿越，实现在任意目录中释放(解压出)一个文件，这个文件可以是Git hook。

看下Bitbucket Data Center如何对“使用GZip压缩过的TAR存档文件”做提取(解压)。


代码片段1: `extractToDisk`函数的具体实现

（具体实现经过一定简化）该函数将TAR存档压缩包中的关键文件压缩包`hooks.atl.tar.atl.gz`中具体条目`TAR archive entry`的“实际的文件路径”作为形参`target`的值。

这个lambda表达式(第4-9行)实现了接口`IoConsumer<T>`的`accept`函数。

>“Lambda 表达式”(lambda expression)是一个匿名函数，Lambda表达式基于数学中的λ演算得名，直接对应于其中的lambda抽象(lambda abstraction)，是一个匿名函数，即没有函数名的函数。Lambda表达式可以表示闭包。


代码片段1:
```java
public void extractToDisk(@Nonnull Path target, @Nonnull Predicate<String> filter) throws IOException {
        this.read((entrySource) -> {
            //lambda expression
            Path entryPath = entrySource.getPath();
            String filename = entryPath.getFileName().toString();

            entrySource.extractToDisk(target.resolve(entryPath));

        }, filter);
    }
```



代码片段2:

`read`函数的具体实现

```
public void read(@Nonnull IoConsumer<EntrySource> reader,
                 @Nonnull Predicate<String> filter) throws IOException {

    TarArchiveEntry entry;
    while ((entry = (TarArchiveEntry) inputStream.getNextEntry()) != null) {
        InputStream entryInputStream = new CloseShieldInputStream(inputStream);
        String name = entry.getName();
        if (filter.test(name)) {
            reader.accept(new TarEntrySource(entryInputStream, Paths.get(name), entry));
        }
    }
 }
```

使用while循环得到所有的归档条目(archive entries)；

最后一行，调用了reader对象的accept函数（传入的实参是一个`TarEntrySource`对象），`TarEntrySource`对象中的`name`是使用`entry.getName();`方法获得的，即`org.apache.commons.compress.archivers.tar.TarArchiveEntry.getName()`方法，这样得到的是“用户输入数据”且未做任何安全过滤，并作为实参直接传递给敏感的`java.nio.Paths.get()`。


因为`accept`函数是由定义的lambda表达式实现的，所以看下`TarEntrySource.extractToDisk()`函数的具体实现(代码片段3)。


可见，类`TarEntrySource`的`extractToDisk`函数的实参是：用户输入的未做任何安全过滤的“路径”。

对每一个压缩包中的每一个条目`TarArchiveEntry`（压缩包中的每个文件）所做的逻辑：
第5行 先使用`java.io.File.getParent()`获取该文件的父目录（即包含`../`的目录），`Files.createDirectories`会根据给定的“路径”，会依次创建所有不存在的目录。（如`/xx/yy/zz`则依次创建3个目录）
(补充对比 `Files.createDirectory()`方法只能创建一个目录，如果这个目录的上级目录是不存在的，就会报错）

第6行 `java.nio.file.Path.toFile`返回了一个File对象，表示这个真实路径的文件对象（空，待写入数据）。
参考 [Java Code Examples java.nio.file.Path.toFile](https://www.programcreek.com/java-api-examples/?class=java.nio.file.Path&method=toFile)

第8行`IoUtils.copy`将压缩包中的某个文件的二进制流，写到对应目录中。(buffer大小为32768 bytes即32kb)

成功将压缩包中的某个文件的二进制流，“复制”到了对应目录。

代码片段3:

```
private static class TarEntrySource extends DefaultEntrySource {

    public void extractToDisk(@Nonnull Path target) throws IOException {

     Files.createDirectories(target.getParent());
     OutputStream out = new FileOutputStream(target.toFile());

     IoUtils.copy(this.inputStream, out, 32768);

     PosixFileAttributeView fileAttributeView = (PosixFileAttributeView)Files.getFileAttributeView(target, PosixFileAttributeView.class);
     fileAttributeView.setPermissions(FilePermissionUtils.toPosixFilePermissions(this.tarArchiveEntry.getMode()));

    }
  }
```

这个路径穿越漏洞，使攻击者能够在攻击者可控的Bitbucket repository中释放(解压)出一个文件，如Git hook。

注意文件权限，如果没有正确设置shell脚本的文件权限，如没有或无法设置文件的执行位(execute bit)，那么Git Hook当然就无法被执行。

根据代码片段3可知，该系统在文件权限上，也直接信任了用户输入：

代码片段3的最后一行`fileAttributeView.setPermissions`函数，直接将文件权限设置为存档条目(archive entry)中原本的文件权限，即使用`this.tarArchiveEntry.getMode()`获取的元信息中的文件权限。

所以可以上传一个有执行权限的文件。

#### 总结

* 场景
  * 直接利用 - 获取Admin角色的用户登录凭证后，导入恶意的TAR存档压缩包文件以便控制Bitbucket server
  * 间接诱导 - 诱导Admin角色的用户，导入恶意的TAR存档压缩包文件以便控制Bitbucket server

* 参考
  * [[BSERV-11706] Bitbucket Data Center - Path traversal in the migration tool leads to RCE - CVE-2019-3397](https://jira.atlassian.com/browse/BSERV-11706)
  * [RIPS Code Analysis](https://blog.ripstech.com/2019/bitbucket-path-traversal-to-rce/)
