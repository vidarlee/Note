Grub 2包含下面几个部分：
 - /boot/grub/grub.cfg 文件
 - /etc/grub.d/ 文件夹
 - /etc/default/grub 文件

/boot/grub/grub.cfg不要手工修改，这个文件是运行 update-grub自动生成的。要修改配置文件的只要打开/boot/grub/grub.cfg文件，找到想修改的地方，然后根据注释找到相应的 /etc/default/grub或/etc/grub.d/ (folder)进行修改。

首先看下 /etc/default/grub，先从应用程序－附件里打开终端，输入
sudo gedit /etc/default/grub
看看打开的文件可作什么修改：
> If you change this file, run 'update-grub' afterwards to update
> /boot/grub/grub.cfg.
>
GRUB_DEFAULT=0 -------->设置默认启动项，按menuentry顺序。比如要默认从第四个菜单项启动，数字改为3，若改为 saved，则默认为上次启动项。

把各项修改后保存，然后
`# update-grub`


隔行输出
`cat filename | awk '{if(NR % 2==1) print}'`
