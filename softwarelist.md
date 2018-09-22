# 软件安装列表

* trimmomatic和python环境相关的

```
sudo apt-get -y update && \
sudo apt-get -y install trimmomatic python-pip \
   samtools zlib1g-dev ncurses-dev python-dev unzip \
   python3.5-dev python3.5-venv make \
   libc6-dev g++ zlib1g-dev
```

* 数据质控：FastQC

```
cd
wget -c http://www.bioinformatics.babraham.ac.uk/projects/fastqc/fastqc_v0.11.5.zip
unzip fastqc_v0.11.5.zip
cd FastQC
chmod +x fastqc
cd
```

* 一些必要的python组件，建议在python3.5环境中安装和运行
 ```
pip install -U pip
pip install -U Cython
pip install -U jupyter jupyter_client ipython pandas matplotlib scipy scikit-learn khmer
pip install -U https://github.com/dib-lab/sourmash/archive/master.zip
```

* 用于序列装配的MEGAHIT
```
cd
git clone https://github.com/voutcn/megahit.git
cd megahit
make
cd
```
* 评价装配结果的Quast
```
cd
git clone https://github.com/ablab/quast.git -b release_4.5
export PYTHONPATH=$(pwd)/quast/libs/
cd
```
