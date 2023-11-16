# 使用 DiskGenius 手動克隆系統

## 克隆操作需要用到 DiskGenius
[https://download2.eassos.cn/DG5511508_x64.zip](https://download2.eassos.cn/DG5511508_x64.zip)

### 本範例使用 980 Pro (來源) 與 545S (目標) 操作

先確認<span style="color:Red">**目標磁碟**</span>的**分割表類型**是不是 GPT 格式  
若是 MBR 格式則需要轉換成 GPT 格式才能繼續操作

![](/Images/001.gif)

------------

先記住<span style="color:Red">**目標磁碟**</span>的**總磁區數**`500118192`  
以及<span style="color:Blue">**來源磁碟**</span>的<span style="color:Brown">**磁碟區(3)**</span>的**總磁區數**`1202176`  
#### 注意每個人的硬碟都不一樣，不要直接抄作業!!!
![](/Images/002.gif)

------------

建立 ESP/MSR 磁碟區，ESP 磁碟區容量改成`100`MB，對齊改成`2048`磁區

![](/Images/003.gif)

------------

在<span style="color:Red">**目標磁碟**</span>結尾建立 Recovery 磁碟區，請自行帶入上面記住的數字  
起始磁區號算法`500118192 - (500118192 mod 2048) - 1202176`等於`498915328`

![](/Images/004.gif)

新增完成後還需要標記為**隱藏磁碟區**以及**修改磁碟區參數**後才會被 Windows 認為是 Recovery 磁碟區

![](/Images/005.gif)

![](/Images/006.gif)

檢查**磁碟區類型 GUID** 是否為 `DE94BBA4-06D1-4D40-A16A-BFD50179D6AC`  
有時程式會抽風沒修改到，若不一樣就再重複一次修改的步驟

------------

再來建立**系統磁碟區**，如果硬碟太大想多切幾塊出來都可以

![](/Images/007.gif)

------------

至此磁碟區已建立完畢，最後依序克隆`ESP、Recovery、系統`磁碟區過去就可以了  
這邊因為系統太大了，所以先用 Windows 安裝光碟裡的 install.wim 的資料測試

![](/Images/008.gif)

------------

## OK讓我們來上電開機看看，輕鬆失敗(?

![](/Images/009.png)

------------

報錯原因是因為克隆的 ESP 磁碟區裡的 BCD 檔案還是使用<span style="color:Blue">**來源磁碟**</span>的磁碟區做開機  
因此在拔除<span style="color:Blue">**來源磁碟**</span>後就會找不到**目標裝置**，所以需要修正 BCD 檔案裡的開機紀錄

先分配一個**磁碟代號**給 ESP 磁碟區才能使用 bcdedit 操作  
拿到的 BCD 檔案路徑為`J:\EFI\Microsoft\Boot\BCD`

![](/Images/010.gif)

------------

先抄下`ESP、Recovery`磁碟區的**設備路徑**  
`\Device\HarddiskVolume36`、`\Device\HarddiskVolume39`

![](/Images/011.gif)

------------

接下來使用組合鍵 Win+R 輸入 cmd 叫出**命令提示字元**  
輸入指令`bcdedit /store "J:\EFI\Microsoft\Boot\BCD" /enum all`

![](/Images/012.png)

會得到下列的資料 (已省略一些不重要的資料)
```
Windows Boot Manager
--------------------
identifier              {bootmgr}
device                  partition=\Device\HarddiskVolume1
path                    \EFI\MICROSOFT\BOOT\BOOTMGFW.EFI
description             Windows Boot Manager
locale                  zh-TW
inherit                 {globalsettings}
default                 {default}
resumeobject            {0a336caa-1546-11ed-b055-819c3fda84ba}
displayorder            {default}
toolsdisplayorder       {memdiag}
timeout                 30

Windows Boot Loader
-------------------
identifier              {default}
device                  partition=C:
path                    \Windows\system32\winload.efi
description             Windows 10
locale                  zh-TW
inherit                 {bootloadersettings}
recoverysequence        {0a336cac-1546-11ed-b055-819c3fda84ba}
displaymessageoverride  Recovery
recoveryenabled         Yes
isolatedcontext         Yes
allowedinmemorysettings 0x15000075
osdevice                partition=C:
systemroot              \Windows
resumeobject            {0a336caa-1546-11ed-b055-819c3fda84ba}
nx                      OptIn
bootmenupolicy          Standard

Windows Boot Loader
-------------------
identifier              {0a336cac-1546-11ed-b055-819c3fda84ba}
device                  ramdisk=[\Device\HarddiskVolume4]\Recovery\WindowsRE\Winre.wim,{0a336cad-1546-11ed-b055-819c3fda84ba}
path                    \windows\system32\winload.efi
description             Windows Recovery Environment
locale                  zh-tw
inherit                 {bootloadersettings}
displaymessage          Recovery
osdevice                ramdisk=[\Device\HarddiskVolume4]\Recovery\WindowsRE\Winre.wim,{0a336cad-1546-11ed-b055-819c3fda84ba}
systemroot              \windows
nx                      OptIn
bootmenupolicy          Standard
winpe                   Yes

Resume from Hibernate
---------------------
identifier              {0a336caa-1546-11ed-b055-819c3fda84ba}
device                  partition=C:
path                    \Windows\system32\winresume.efi
description             Windows Resume Application
locale                  zh-TW
inherit                 {resumeloadersettings}
recoverysequence        {0a336cac-1546-11ed-b055-819c3fda84ba}
recoveryenabled         Yes
isolatedcontext         Yes
allowedinmemorysettings 0x15000075
filedevice              partition=C:
filepath                \hiberfil.sys
bootmenupolicy          Standard
debugoptionenabled      No

Windows Memory Tester
---------------------
identifier              {memdiag}
device                  partition=\Device\HarddiskVolume1
path                    \EFI\Microsoft\Boot\memtest.efi
description             Windows 記憶體診斷
locale                  zh-TW
inherit                 {globalsettings}
badmemoryaccess         Yes

Device options
--------------
identifier              {0a336cad-1546-11ed-b055-819c3fda84ba}
description             Windows Recovery
ramdisksdidevice        partition=\Device\HarddiskVolume4
ramdisksdipath          \Recovery\WindowsRE\boot.sdi
```

------------

得到的資料有很多 device partition 指向了<span style="color:Blue">**來源磁碟**</span>的**設備路徑**  
自己替換成<span style="color:Red">**目標磁碟**</span>的**設備路徑**後修改掉，下面提供本次範例用的指令  
#### 注意每個人的 identifier 跟設備路徑都會不一樣，不要直接抄作業!!!
```
bcdedit /store "J:\EFI\Microsoft\Boot\BCD" /set {bootmgr} device partition=\Device\HarddiskVolume36
bcdedit /store "J:\EFI\Microsoft\Boot\BCD" /set {default} device partition=I:
bcdedit /store "J:\EFI\Microsoft\Boot\BCD" /set {default} osdevice partition=I:
bcdedit /store "J:\EFI\Microsoft\Boot\BCD" /set {0a336cac-1546-11ed-b055-819c3fda84ba} device ramdisk=[\Device\HarddiskVolume39]\Recovery\WindowsRE\Winre.wim,{0a336cad-1546-11ed-b055-819c3fda84ba}
bcdedit /store "J:\EFI\Microsoft\Boot\BCD" /set {0a336cac-1546-11ed-b055-819c3fda84ba} osdevice ramdisk=[\Device\HarddiskVolume39]\Recovery\WindowsRE\Winre.wim,{0a336cad-1546-11ed-b055-819c3fda84ba}
bcdedit /store "J:\EFI\Microsoft\Boot\BCD" /set {0a336caa-1546-11ed-b055-819c3fda84ba} device partition=I:
bcdedit /store "J:\EFI\Microsoft\Boot\BCD" /set {0a336caa-1546-11ed-b055-819c3fda84ba} filedevice partition=I:
bcdedit /store "J:\EFI\Microsoft\Boot\BCD" /set {memdiag} device partition=\Device\HarddiskVolume36
bcdedit /store "J:\EFI\Microsoft\Boot\BCD" /set {0a336cad-1546-11ed-b055-819c3fda84ba} ramdisksdidevice partition=\Device\HarddiskVolume39
```

![](/Images/013.png)

------------

最後把 ESP 磁碟區的**磁碟代號**刪除掉，然後驗證結果

![](/Images/014.gif)

![](/Images/015.png)

# 完成
