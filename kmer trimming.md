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

建议查看[官方说明](https://khmer.readthedocs.io/en/latest/user/guide.html)。


### 宏基因组

* 计数，运行```load-into-counting``` 和 ```abundance-dist.py```
	* 可选支线任务：绘制kmer统计图
* 标准化，运行```normalize-by-median.py```，设置cutoff为20  
* 过滤低丰度kmer，运行```filter-below-abund.py```，设置cutoff为100
* 分区组，使用```load-graph.py```进行partition
* 正常装配，可以用```extract-paired-reads.py```提取成对序列
 
### 宏转录组

* 标准化，运行```normalize-by-median.py```，参数cutoff=20
* 提取余下的pair-end reads等等，```extract-paired-reads.py```
* 使用基因组或宏基因组专业的组装软件进行序列组装

### mRNA-seq

* 标准化与宏基因组、宏转录组相同
* 运行```extract-paired-reads.py```
* 序列组装


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

一个说明：khmer2.0以后的版本会根据给定的-M自动计算-x。然而这就转到了-M的话题。  
-M 参数设置最大允许使用的内存。简单的说，建议设置M为你实际使用的服务器的最大可用内存(或略微小一点点)，不用设置x。  
khmer对kmer进行计数和其他一些处理用的数据结构比较特殊，是概率性的并且使用固定的内存容量("probabilistic and constant memory")。前一个特征决定了，设置的内存过小时，犯错的可能性很大，所以在条件允许范围内尽量把M设大一点。  
关于内存使用问题的[官方说法](https://khmer.readthedocs.io/en/latest/user/choosing-table-sizes.html)。


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

### 低丰度过滤

```
filter-abund.py -x 5e7 -k 20 -C 2 stamps-dn.ct stamps-reads.fa.gz.keep
```
如果不对输出文件名做说明的话，输出的名称是inputfilename.abundtrim。
参数C是cutoff，缺省为2。


### 分区组(partitioning)

尝试以下命令：
```
do-partition.py -k 32 -x 1e8 -s 1e4 -T 8 stamps-part stamps-reads.fa.gz
```

这里用单个脚本执行了所有partitioning的步骤，会花一点时间。
关于partition可以看[一篇文章](http://www.pnas.org/content/early/2012/07/25/1121464109)

或者，如果更细致地逐步进行的话：
```
load-graph.py -k 32 -N 4 -x 16e9 50m iowa-corn-50m.fa.gz
```
这个脚本的用途是，根据序列创建graph，并保存到 <ptname>，形式如```python scripts/load-graph.py <ptname> <data1>```

接下来根据图分区组：
```
partition-graph.py --threads 4 -s 1e5 50m
```
其中--threads参数设置线程数，根据服务器实际核数调整。  
这个脚本后得到一系列分组文件，命名类似50m.subset.*.pmap

然后把这些文件合并：
```
merge-partitions.py 50m
```
并把分组注释到原测序文件：
```
annotate-partitions.py 50m iowa-corn-50m.fa.gz
```
得到新的文件iowa-corn-50m.fa.gz.part。

提取分组：
```
extract-partitions.py iowa-corn-50m iowa-corn-50m.fa.gz.part
```
然后就可以分别进行后续组装了。

## 其他话题

### 内存问题

担心内存不够的一个可选处理方式：

* 运行```unique-kmers.py```，得到估计的kmer数量
* 在接下来要运行的脚本设置参数-U，khmer会根据U和M估计内存使用，如果内存不够，会直接报错并终止脚本，节省时间。



