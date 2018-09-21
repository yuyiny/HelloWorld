# 数据分析需要安装的软件

* 用于数据质量评价的trimm和python环境

```
sudo apt-get -y update && \
sudo apt-get -y install trimmomatic python-pip \
   samtools zlib1g-dev ncurses-dev python-dev unzip \
   python3.5-dev python3.5-venv make \
   libc6-dev g++ zlib1g-dev
```

* 数据质控：FastQC

```
wget -c http://www.bioinformatics.babraham.ac.uk/projects/fastqc/fastqc_v0.11.5.zip
unzip fastqc_v0.11.5.zip
cd FastQC
chmod +x fastqc
cd
```

* 在python3.5环境中安装需要的包
 ```
 python3.5 -m venv ~/py3
. ~/py3/bin/activate
pip install -U pip
pip install -U Cython
pip install -U jupyter jupyter_client ipython pandas matplotlib scipy scikit-learn khmer
pip install -U https://github.com/dib-lab/sourmash/archive/master.zip
```

