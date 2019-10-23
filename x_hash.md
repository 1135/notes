
### 本机计算hash

>校验完整性：如果满足 本机计算得到校验和 = 官网提供校验和 ，则确保原文件没有被修改或替换


#### linux

```bash
md5sum test.sql
sha1sum test.sql
sha256sum test.sql
```

#### Mac

```
man md5 #查看帮助
md5 1.jpg
```

```
shasum -h #查看帮助

# 计算某文件的sha1
shasum -a 1 kali-linux-2017.1-amd64.iso
05765cbb81dfc898a4ddee4c1a27a7c2e888f796  kali-linux-2017.1-amd64.iso

# 计算某文件的sha256
shasum -a 256 kali-linux-2017.1-amd64.iso
49b1c5769b909220060dc4c0e11ae09d97a270a80d259e05773101df62e11e9d  kali-linux-2017.1-amd64.iso
```

#### Windows powershell自带hash工具

```
certutil -hashfile  <文件名>  <hash类型>
certutil -hashfile yourfilename.ext MD5
certutil -hashfile yourfilename.ext SHA1
certutil -hashfile yourfilename.ext SHA256
```
