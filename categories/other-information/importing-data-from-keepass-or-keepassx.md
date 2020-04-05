# 从Keepass或KeepassX导入数据

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Importing-data-from-Keepass-or-KeepassX)
{% endhint %}

### 介绍

Bitwarden可以导入来自许多[应用程序](https://help.bitwarden.com/article/import-data/)的数据。\[**译者注**\]：我翻译的[此链接的中文版](https://bitwardenhelp.ppgg.in/categories/getting-started/import-your-data-from-another-application)

当前的导入器仅允许您选择导入文件的格式，而不能告诉您是如何将数据转换为Bitwarden的。

### Keepass和KeepassX的导入结果不同

从Keepass或KeepassX导入会产生完全不同的结果，尽管它们使用相同的Keepass 2.x kbdx数据库：

* Keepass CSV文件是在**组织**级别（每个条目的所有者）导入，Keepass群组将转换为Bitwarden的**集合**。
* Keepass XML文件是在**用户**级别（每个条目的所有者）导入，Keepass群组将转换为Bitwarden的**文件夹**，其主文件夹为Keepass数据库的名称。

做导入操作时，Bitwarden自己会自动做很多工作：如将“集合”更改为“文件夹”以及转换所有条目的所有权等。因此，根据您的需要选择适当的方法！

### 示例

#### 名称为“MyVault”的Keepass数据库

**群组为:**

* Group1
  * Group1Sub1
  * Group2Sub2
* Group2

#### 通过Keepass（CSV）导入后：

**所有者** = 组织

**集合为:**

* Group1
  * Group1Sub1
  * Group2Sub2
* Group2

#### 通过Keepass（XML）导入后：

**所有者** = 登录的用户

**文件夹为:**

* MyVault
  * Group1
    * Group1Sub1
    * Group2Sub2
  * Group2

注意1：您可能必须手动创建主文件夹，否则导入后会将MyVault/Group1和MyVault/Group1显示为文件夹（因为没有上级的MyVault文件夹）。创建文件夹MyVault后才会在MMI中显示子文件夹。（\[**译者注**\]：Bitwarden文件夹规则：[官方](https://help.bitwarden.com/article/folders/)，[中文版](https://bitwardenhelp.ppgg.in/categories/features/organizing-your-vault-with-folders)）

注意2：在导入Bitwarden之前，您也可以编辑文件夹以删除主文件夹“MyVault”，或编辑导出的CSV文件并删除每个条目中的“MyVault/”字符串。  


