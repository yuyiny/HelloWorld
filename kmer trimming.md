# k-mer trimming

运行[官网](https://khmer.readthedocs.io/en/latest/user/examples.html)上的一个栗子。


## 准备工作

在根目录或者自己用的工作目录下建一个kmer文件夹：
```
mkdir kmer
cd kmer
```

由于kmer非常烧内存，我们使用一个模拟的[小数据集stamp](https://github.com/dib-lab/khmer/blob/stable/data/stamps-reads.fa.gz)来进行尝试。
下载这个数据集：

```
wget https://github.com/dib-lab/khmer/raw/stable/data/stamps-reads.fa.gz
```

这个数据集的大小大约是3M，包含一堆100bp的序列，错误率大约是1%。

## 工作流程

建议查看[官方说明](https://khmer.readthedocs.io/en/latest/user/guide.html)

### 宏基因组

* 运行```load-into-counting```  
* 运行```normalize-by-median.py```，设置cutoff为20  
* 运行```filter-below-abund.py```，设置cutoff为100
* 使用```load-graph.py```进行partition
* 正常装配，可以用```extract-paired-reads.py```提取成对序列
 

## 使用方法示例

### 计数

```
load-into-counting.py -x 1e8 -k 20 stamps-reads.ct stamps-reads.fa.gz
```
这个脚本的用法是：
```
usage: load-into-counting.py [--version] [--info] [-h] [-k KSIZE]
                             [-U UNIQUE_KMERS] [--fp-rate FP_RATE]
                             [-M MAX_MEMORY_USAGE] [--small-count]
                             [-T THREADS] [-b] [-s FORMAT] [-f] [-q]
                             output_countgraph_filename
                             input_sequence_filename
                             [input_sequence_filename ...]
```
我们输入了stamps-reads.fa.gz文件，设置-x(max tablesize)为10^8，-k(kmer)大小为20bp，输出coutgraph(ct)文件。  
简单地说，数了一下每个kmer出现了多少次。


### 统计

```
abundance-dist.py stamps-reads.ct stamps-reads.fa.gz stamps-reads.hist
```
得到频数分布、累计频数、累计频率统计表(hist)。  
可以用 ```head stams-reads.hist```看一下前几行。

### 标准化

这个算法的一个说明参见[这里](http://ivory.idyll.org/blog/what-is-diginorm.html)。  
尝试以下命令：

```
normalize-by-median.py -k 20 -C 10 -x 1e8 stamps-reads.fa.gz -s stamps-dn.ct
```

用法：
```
normalize-by-median.py [--version] [--info] [-h] [-k KSIZE]
                              [-U UNIQUE_KMERS] [--fp-rate FP_RATE]
                              [-M MAX_MEMORY_USAGE] [--small-count] [-q]
                              [-C CUTOFF] [-p] [--force_single]
                              [-u unpaired_reads_filename] [-s filename]
                              [-R report_filename]
                              [--report-frequency report_frequency] [-f]
                              [-o filename] [-l filename] [--gzip | --bzip]
                              input_sequence_filename
                              [input_sequence_filename ...]
```

然后再看一下统计结果
```
abundance-dist.py stamps-dn.ct stamps-reads.fa.gz.keep stamps-dn.hist
less stamps-dh.hist
```
可以发现频数特别少的基本都被过滤掉了。  按q退出查看。  


###  尝试画kmer spectrum

先从git上clone项目：
```
git clone https://github.com/MG-RAST/kmerspectrumanalyzer.git
```

然后运行脚本：
```
/kmerspectrumanalyzer/scripts/plotkmerspectrum.py stamps-dn.hist
```
如果安装的系统是非GUI的，不能直接查看，可以用jupyter notebook打开，就可以查看和下载了。

比较重要的参数是画图选项，1、3、5、6、20比较常用，其中6是缺省值：
```
  -g OPTION, --graph OPTION
                        -3: No graphs, produce stratify one-line summary 
                        -2: No graphs, but produce stratify table 
                        -1: no graphs, only append summary to kmers.log 
                        0 : number of kmers vs. kmer abundance (basic spectrum) 
                        1 : kmers observed
                        vs. kmer abundance (scaled spectrum) 
                        2 : kmer abundance vs. basepairs observed 
                        3 : kmer abundance vs. fraction of observed data 
                        4 : kmer abundance vs. fraction of distinct kmers 
                        5 : fraction of observed vs. kmer rank (kmer k-dominance curve) 
                        6 : kmer abundance vs. kmer rank (kmer rank-abundance) 
                        24: band-colored variant 
                        25: band-colored variant of kmer k-dominance curve 
                        26: band-colored variant of kmer rank-abundance curve 
                        30: Renyi entropy (transformation, function of lambda)
```

#### trouble shooting

执行上述命令时可能会遇到以下问题：
```
Traceback (most recent call last):
  File "/home/ubuntu/kmerspectrumanalyzer/scripts/plotkmerspectrum.py", line 10, in <module>
    from ksatools.ksatools import getmgrkmerspectrum, printstats, loadfile, makegraphs
ImportError: No module named 'ksatools'
```

这种情况是由于ksatools没有正确安装。(在kmerspectrumanalyzer目录下有ksatools目录，可以在github查看具体的细节)。  
可以尝试运行setup.py脚本：
```
sudo python setup.py install
```








然后
```
do-partition.py -k 32 -x 1e8 -s 1e4 -T 8 stamps-part stamps-reads.fa.gz
```

这个会花一点时间。
关于partition可以看[一篇文章](http://www.pnas.org/content/early/2012/07/25/1121464109)



待整理示例代码：

  
#do-partition.py -k 32 -x 1e8 -s 1e4 -T 8 stamps-part \
	../../data/stamps-reads.fa.gz


#../../sandbox/error-correct-pass2.py -C 10 stamps-dn.ct \
	../../data/stamps-reads.fa.gz
  
load-into-counting.py -x 1e8 -k 20 stamps-corr.ct stamps-reads.fa.gz.corr
abundance-dist.py stamps-corr.ct stamps-reads.fa.gz.corr stamps-corr.hist
extract-partitions.py stamps-part stamps-reads.fa.gz.part
extract-partitions.py -X 1 stamps-part stamps-reads.fa.gz.part
load-into-counting.py -x 1e8 -k 20 stamps-part.g0.ct stamps-part.group0000.fa
load-into-counting.py -x 1e8 -k 20 stamps-part.g1.ct stamps-part.group0001.fa
abundance-dist.py stamps-part.g0.ct stamps-part.group0000.fa stamps-part.g0.hist
abundance-dist.py stamps-part.g1.ct stamps-part.group0001.fa stamps-part.g1.hist

filter-abund.py stamps-dn.ct stamps-reads.fa.gz.keep
normalize-by-median.py -x 1e8 -k 20 -C 10 stamps-reads.fa.gz.keep.abundfilt \
	--savetable stamps-dn3.ct

abundance-dist.py stamps-dn3.ct stamps-reads.fa.gz.keep.abundfilt.keep \
	stamps-dn3.hist
  
  
trim-low-abund
```

.
