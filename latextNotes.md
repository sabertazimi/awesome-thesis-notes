# LaTex Basic Notes

## Installation

```bash
sudo apt-get install libdigest-perl-md5-perl perl-tk
wget http://mirrors.ustc.edu.cn/CTAN/systems/texlive/Images/texlive2018.iso
sudo mount -o loop textlive2018.iso /mnt/iso
cd /mnt/iso
sudo ./install-tl -gui perltk
sudo umount /mnt/iso

export MANPATH=${MANPATH}:/usr/local/texlive/2018/texmf-dist/doc/man
export INFOPATH=${INFOPATH}:/usr/local/texlive/2018/texmf-dist/doc/info
export PATH=${PATH}:/usr/local/texlive/2018/bin/x86_64-linux
```

