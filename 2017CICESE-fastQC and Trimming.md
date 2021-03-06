# 数据质控

这里我们先假装已经有了python及相关的包。  


## 准备要使用的数据

首先建立一个专门的文件夹用于存储数据。姑且按照window的习惯叫文件夹吧。
```
mkdir ~/data
cd ~/data
```
* mkdir = make directory。建立文件目录。  ~主目录。 即在主目录下面建立一个叫data的文件夹。  
* cd =change directory。改变当前目录到data文件夹，相对路径和绝对路径都可以。绝对路径的情况相当于在window系统的目录里使用路径直接定位到一个文件夹。

下载数据
```
curl -O -L https://s3-us-west-1.amazonaws.com/dib-training.ucdavis.edu/metagenomics-scripps-2016-10-12/SRR1976948_1.fastq.gz
curl -O -L https://s3-us-west-1.amazonaws.com/dib-training.ucdavis.edu/metagenomics-scripps-2016-10-12/SRR1976948_2.fastq.gz
curl -O -L https://s3-us-west-1.amazonaws.com/dib-training.ucdavis.edu/metagenomics-scripps-2016-10-12/SRR1977249_1.fastq.gz
curl -O -L https://s3-us-west-1.amazonaws.com/dib-training.ucdavis.edu/metagenomics-scripps-2016-10-12/SRR1977249_2.fastq.gz
```
curl是一个利用URL规则传输文件的命令。简单地说，下载很好用，在屏幕上给出文件大小和当前速度和进度。详见[这里](http://man.linuxde.net/curl)。  
其中-O:把输出写到该文件，保留远程的文件名。 -L可能是支持自动跳转。

然后进行md5sum校验：
```
md5sum SRR1976948_1.fastq.gz SRR1976948_2.fastq.gz SRR1977249_1.fastq.gz SRR1977249_2.fastq.gz
```
md5sum这个命令用于生成和校验文件的md5值，对于网络传输的大体积测序文件，进行md5sum校验、验证文件传输的完整性是非常必要的。关于md5sum，可以看[这里](https://blog.csdn.net/hxHardway/article/details/78088640)。  
这一步的结果是这样的
```
37bc70919a21fccb134ff2fefcda03ce  SRR1976948_1.fastq.gz
29919864e4650e633cc409688b9748e2  SRR1976948_2.fastq.gz
ee25ae14f84308f972cebd42e55502dd  SRR1977249_1.fastq.gz
84b15b4f5975cd7941e211336ff27993  SRR1977249_2.fastq.gz
```

查看一下当前文件夹中的文件列表（长格式 -l）：
```
ls -l
```
结果是这样的：
```
total 704316
-rw-rw-r-- 1 ubuntu ubuntu 169620631 Sep 22 12:26 SRR1976948_1.fastq.gz
-rw-rw-r-- 1 ubuntu ubuntu 185636992 Sep 22 12:28 SRR1976948_2.fastq.gz
-rw-rw-r-- 1 ubuntu ubuntu 177682886 Sep 22 12:30 SRR1977249_1.fastq.gz
-rw-rw-r-- 1 ubuntu ubuntu 188269868 Sep 22 12:32 SRR1977249_2.fastq.gz
```

为了避免误操作影响原始数据，取消user的write权限：
```
chmod u-w *
```
可以再用ls -l查看一下，会发现前面的权限有变化。

## 准备工作文件夹
在根目录~下建立一个工作文件夹，并定位到工作文件夹：
```
mkdir ~/work
cd ~/work
```
把data文件夹中的数据拷贝过去。
```
ln -fs ~/data/* .
```
这里星号是通配符，点号是文件的标识。注意星号和点号之间有空格。  
这些是压缩(gz)的fastq文件，也就是最通常的测序下机文件，内容含义可以看[维基百科](https://en.wikipedia.org/wiki/FASTQ_format)和[一个简单的说明](https://blog.csdn.net/GodSunshine/article/details/51946165)。  
可以鼠标上下滚动查看序列。直接按键盘上的q退出。

## FastQC

下载和安装
```
cd
wget -c http://www.bioinformatics.babraham.ac.uk/projects/fastqc/fastqc_v0.11.5.zip
unzip fastqc_v0.11.5.zip
cd FastQC
chmod +x fastqc
cd
``` 
wget是另一个相当稳定好用的下载命令。用于从指定URL下载文件，也可以用于从ftp下载。支持断点继续下载，所以当网有点卡或者文件很大时尤其适用。详细看[这里](http://man.linuxde.net/wget)。   
下载的是一个zip文件。zip文件的压缩和解压缩都非常简单，直接就是zip或upzip。  
设置fastqc为可执行的(+x)。  

接下来就可以运行FastQC：
```
~/FastQC/fastqc SRR1976948_1.fastq.gz
~/FastQC/fastqc SRR1976948_2.fastq.gz
~/FastQC/fastqc SRR1977249_1.fastq.gz
~/FastQC/fastqc SRR1977249_2.fastq.gz
```
这需要一些时间，屏幕上会显示进度。  
结束后可以查看一下结果文件：
```
ls -d *fastqc.zip*
```
同样，星号还是通配符。因为我们知道FastQC输出的结果形式是类似"xxxxfastqc.zip"的，所以可以直接这样查看。当然也可以查看文件夹中所有的文件（考虑到只做了4个文件，输出并不多）。

这时候可以打开Jupyter Notebook，查看和下载html格式的质控结果。直接在文件前面打钩，然后点上面的下载，就可以下载到本地。  
这里建议使用命令```jupyter notebook &```让jupyter在后台运行，控制台中正常进行其他操作，关闭时ctrl+c。  

## Trimmomatic
接下来使用Trimmomatic进行数据的修剪(Trimming，姑且这么叫吧)。  
先下载要trim off 的adaptor：
```
curl -O -L http://dib-training.ucdavis.edu.s3.amazonaws.com/mRNAseq-semi-2015-03-04/TruSeq2-PE.fa
```
然后运行程序(需要一些时间慢慢完成，在此期间不显示命令提示符$)：
```
for filename in *_1.fastq.gz
do

#Use the program basename to remove _1.fastq.gz to generate the base
base=$(basename $filename _1.fastq.gz)
echo $base

TrimmomaticPE ${base}_1.fastq.gz \
              ${base}_2.fastq.gz \
     ${base}_1.qc.fq.gz ${base}_s1_se \
     ${base}_2.qc.fq.gz ${base}_s2_se \
     ILLUMINACLIP:TruSeq2-PE.fa:2:40:15 \
     LEADING:2 TRAILING:2 \
     SLIDINGWINDOW:4:2 \
     MINLEN:25
done
```
是一个很直接的for循环。从之前的文件名可以看出来，实际上我们处理了两个文库(对应两个不同的序列号)，每个文库分成了两个文件储存，分别是-1和-2结尾。循环进行了两次，第一次是对文库A，分别对文库A_1.fastq.gz和A_2.fastq.gz运行Trim程序。下面几行大写字母开头的是运行参数：
* ILLUMINACLIP：adaptor文件(TruSeq2-PE.fa):最大允许错配数(2):palindrome模式下匹配碱基数阈值(40):simple模式下匹配碱基数阈值(15)
* LEADING: 切除首端碱基质量小于(2)的
* TRAILING：切除末端碱基质量小于(2)的，LEADING和TRAILING可以根据需要调整，也有人用3
* SLIDINGWINDOW: 窗口大小(4):平均碱基质量小于(2)时切除。
* MINLEN: 最小读长(36)，如果序列太短就不要了。
上面没有用到的还包括：
* TOPHRED33:将碱基质量转换为pred33格式
* TOPHRED64:将碱基质量转换为pred64格式
* AVGUAL: 丢弃质量低于设置值的reads
* CROP: 保留的reads的长度值
* HEADCROP: 在reads的首端切除指定的长度

## 再次运行FastQC
```
~/FastQC/fastqc SRR1976948_1.qc.fq.gz
~/FastQC/fastqc SRR1976948_2.qc.fq.gz
~/FastQC/fastqc SRR1977249_1.qc.fq.gz
~/FastQC/fastqc SRR1977249_2.qc.fq.gz
```
还是在jupyter notebook中查看，这次整齐多了。

## 多样本比较
先安装MultiQC:
```
pip install git+https://github.com/ewels/MultiQC.git
```
然后在整个工作目录运行：
```
multiqc .
```
运行之后输出是这样的：
```
[INFO   ]         multiqc : This is MultiQC v1.7.dev0
[INFO   ]         multiqc : Template    : default
[INFO   ]         multiqc : Searching '.'
Searching 29 files..  [####################################]  100%
[INFO   ]          fastqc : Found 8 reports
[INFO   ]         multiqc : Compressing plot data
[INFO   ]         multiqc : Report      : multiqc_report.html
[INFO   ]         multiqc : Data        : multiqc_data
[INFO   ]         multiqc : MultiQC complete
```
根据结果信息，在jupyter notebook里找到multiqc_report.html，就是多样本比较的结果了。
后缀有qc的是刚才Trimming之后的输出。  
每一项质控的名称后面会显示有几个样本pass、warning和fail，鼠标移上去可以查看详细。



## 参考资料
关于质控，可以参考[这个中文资料](http://www.360doc.com/content/17/0717/11/44784011_672199078.shtml)。  
[2016 Short read quality evaluation and trimming](https://2016-short-read-trimming.readthedocs.io/en/latest/)

