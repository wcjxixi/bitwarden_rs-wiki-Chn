# 1.从 Keepass 或 KeepassX 导入数据

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Importing-data-from-Keepass-or-KeepassX)
{% endhint %}

## 介绍 <a id="introduction"></a>

Bitwarden 可以导入来自许多[应用程序](https://help.bitwarden.in/import-export/import-data-to-your-vault)的数据。

当前的导入器让您仅选择导入文件的格式即可，而不用去管它是如何将数据转换为 Bitwarden 的。

## Keepass 与 KeepassX 不同的导入结果 <a id="different-import-results-for-keepass-and-keepassx"></a>

从 Keepass 或 KeepassX 导入会产生完全不同的结果，尽管它们使用相同的 Keepass 2.x kbdx 数据库：

* Keepass CSV 文件是在**组织**级别（每个条目的所有者）导入，Keepass 群组将转换为 Bitwarden 的**集合**。
* Keepass XML 文件是在**用户**级别（每个条目的所有者）导入，Keepass 群组将转换为 Bitwarden 的**文件夹**，其主文件夹为 Keepass 数据库的名称。

做导入操作时，Bitwarden 自己会自动做很多工作：比如将「集合」更改为「文件夹」以及转换所有条目的所有权等。因此，根据您的需要选择合适的方式！

## 示例 <a id="example"></a>

### 名称为「MyVault」的 Keepass 数据库 <a id="keepass-database-with-name-myvault"></a>

**群组为:**

* Group1
  * Group1Sub1
  * Group2Sub2
* Group2

### 通过 Keepass\(CSV\) 导入后 <a id="import-via-keepass-csv"></a>

**所有者** = 组织

**集合为:**

* Group1
  * Group1Sub1
  * Group2Sub2
* Group2

### 通过 Keepass\(XML\) 导入后 <a id="import-via-keepass-xml"></a>

**所有者** = 已登录的用户

**文件夹为:**

* MyVault
  * Group1
    * Group1Sub1
    * Group2Sub2
  * Group2

注意1：您必须手动创建主文件夹，否则导入后会将 MyVault/Group1 和 MyVault/Group2 显示为文件夹（因为没有上级 MyVault 文件夹）。创建 MyVault 文件夹后才会在 MMI 中显示子文件夹。（\[**译者注**\]：Bitwarden 文件夹规则：[官方 Help](https://help.bitwarden.com/article/folders/)，[中文版](https://help.bitwarden.in/your-vault/folders)）

注意2：在导入 Bitwarden 之前，您可以编辑文件夹以删除主文件夹「MyVault」，或编辑导出的 CSV 文件并删除每个条目中的「MyVault/」字符串。  


