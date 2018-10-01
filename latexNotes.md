# LaTex Basic Notes

## Installation

https://liam0205.me/texlive/

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

## Basis

- [Simple Introduction](https://liam0205.me/2014/09/08/latex-introduction)
- [Writing Scientific Documents Using LATEX](ftp://ftp.dante.de/tex-archive/info/intro-scientific/scidoc.pdf)
- [Haiyang Liu's Tutorial](https://github.com/wuzhouhui/misc/blob/master/LaTeX%E5%85%A5%E9%97%A8%20%E5%88%98%E6%B5%B7%E6%B4%8B.pdf)

## Math

### Symbol

[The Comprehensive LATEX Symbol List](http://ctan.math.illinois.edu/info/symbols/comprehensive/symbols-a4.pdf)

