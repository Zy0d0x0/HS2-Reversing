# HS2-Reversing
How i took apart the HS2 Firmware

Download the firmware for the website this can be done with a linux program such as wget or you can do it with your web browser.

```
ubuntu@ubuntu2-VirtualBox:~/reverse$ wget https://www.ailunce.com/Assets/file/Ailunce-HS2-FW-V1.37.zip
--2021-06-08 22:19:19--  https://www.ailunce.com/Assets/file/Ailunce-HS2-FW-V1.37.zip
Resolving www.ailunce.com (www.ailunce.com)... 47.253.10.103
Connecting to www.ailunce.com (www.ailunce.com)|47.253.10.103|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 431910 (422K) [application/x-zip-compressed]
Saving to: ‘Ailunce-HS2-FW-V1.37.zip’

Ailunce-HS2-FW-V1.37.zip                           100%[===============================================================================================================>] 421.79K   923KB/s    in 0.5s    

2021-06-08 22:19:20 (923 KB/s) - ‘Ailunce-HS2-FW-V1.37.zip’ saved [431910/431910]

ubuntu@ubuntu2-VirtualBox:~/reverse$ 
```
When the firmware has been downloaded the next step is to extract the zip file that its currently compressed into.

```
ubuntu@ubuntu2-VirtualBox:~/reverse$ unzip Ailunce-HS2-FW-V1.37.zip 
Archive:  Ailunce-HS2-FW-V1.37.zip
  inflating: AilunceHS2-FW-V1.3.7.dfu  
  inflating: Ailunce HS2 FW-V1.3.7 changelog.txt  
ubuntu@ubuntu2-VirtualBox:~/reverse$ ls
'Ailunce HS2 FW-V1.3.7 changelog.txt'   AilunceHS2-FW-V1.3.7.dfu   Ailunce-HS2-FW-V1.37.zip
ubuntu@ubuntu2-VirtualBox:~/reverse$ 


```

When the firmware has been extracted it should be noted the firmware file is further compressed 
in a DFU extention. this can be extracted by using the dfuse-pack.py python application attached
to this git repo.


```
ubuntu@ubuntu2-VirtualBox:~/reverse$ python3 dfuse_pack.py -d AilunceHS2-FW-V1.3.7.dfu
File: "AilunceHS2-FW-V1.3.7.dfu"
b'DfuSe' v1, image size: 797169, targets: 1
b'Target' 0, alt setting: 0, name: "b'ST...'", size: 796884, elements: 1
  0, address: 0x08000000, size: 796876
    DUMPED IMAGE TO "AilunceHS2-FW-V1.3.7.dfu.target0.image0.bin"
usb: 0483:0000, device: 0x0000, dfu: 0x011a, b'UFD', 16, 0x16c30ebd
ubuntu@ubuntu2-VirtualBox:~/reverse$ ls
'Ailunce HS2 FW-V1.3.7 changelog.txt'   AilunceHS2-FW-V1.3.7.dfu   AilunceHS2-FW-V1.3.7.dfu.target0.image0.bin   Ailunce-HS2-FW-V1.37.zip   dfuse_pack.py
ubuntu@ubuntu2-VirtualBox:~/reverse$ 

```

As you can see in the above output a bin file is created called `AilunceHS2-FW-1.3.5.dfu.target0.image0.bin` this can then be inspected by a tool built into
linux called `strings` and the usage is as simple as using the tool name and inputing a file name. Lots of strings where found when searching the binary file
so i then decided to source out a few intresting ones.

```
ubuntu@ubuntu2-VirtualBox:~/reverse$ strings AilunceHS2-FW-V1.3.7.dfu.target0.image0.bin |grep setup
agent setup
factory setup
user setup
ubuntu@ubuntu2-VirtualBox:~/reverse$ 

```
In the above output it was possible to see their may be different levels of access, this was confirmwed by Ailuence had released a blog
post disclosing the route to enter the hidden menu screen within the SET menu on the HS2. Within the blog post it further exposed the "agent setup" 
user access. When confirming the access worked i tried to manully brute force different user accounts by typing in known default pin numbers 
that fit within the lenght of its policy. This is when the "user setup" acount was found by using "000000" as the pin number althought it was
found this user access was nothing useful and with even less rights then the "agent setup" account. 

This then made me want to further inspect the binary file and through expecericen i knew Ghidra would be the perfect tool for the job.
