---
layout: post
title: '如何使用Shapeit2对人类基因组数据进行phasing'
categories:
    - phasing
tags:
    - shapeit2
---
[SHAPEIT(2.0)](https://mathgen.stats.ox.ac.uk/genetics_software/shapeit/shapeit.html)是专门用于对推断基因组单体型的软件，有牛津大学的团队所开发，并且一直应用与千人基因组计划中。

以下，我将记录如何通过shapeit2对人群的变异数据集（VCF 格式）进行phasing，并构造出reference panel的过程。

首先，准备文件。整个过程只需要变异数据集（VCF 格式），样本信息文件(sample.fam)，genetic_map文件和参考序列（fasta格式）。对于genetic_map文件需要单独做些说明，这个记录的是基因组中各个位点的重组率和物理距离之间的关系，这是phasing过程非常重要的一个文件。它来自于[人类单体型计划-Hapmap计划](http://hapmap.ncbi.nlm.nih.gov/)，可以[下载](http://hapmap.ncbi.nlm.nih.gov/downloads/recombination/),
[最新版是b37或者说hg19](http://hapmap.ncbi.nlm.nih.gov/downloads/recombination/2011-01_phaseII_B37/genetic_map_HapMapII_GRCh37.tar.gz)，如果你的reference版本高于hg19，则需要liftover，liftover之后那些顺序发生交叉的位点，是liftover的错误导致的，要去掉。但是要注意的是，genetic-map中两个位点之间的重组率（recombination rate）是不变的，这其实也很好理解，reference之所以需要升级，是因为它的组装结果并非是百分百符合真实情形的，随着技术的进步，我们会不断去升级逼近这个真实情况，但重组率是根据群体的重组情况来计算的，它是由真实情况反映出来的，因此即便reference版本改变了，它的值也不需要改变。不过对于两个位点之间的物理距离（physical distance）就不同了，leftover之后，这个距离是会发生变化的，通过和原点（一般是重组率为0的点或者就是各个染色体的第一个位点）的距离比例来调节。

至于样本信息文件，格式如下：
```
1009	1009-01	0	0	1	34
1009	1009-02	0	0	2	33
1009	1009-06	1009-01	1009-02	2	67
1030	1030-01	0	0	1	43
1030	1030-02	0	0	2	44
1030	1030-06	1030-01	1030-02	1	71
```

其他的两份文件就不必多说了。

准备好以上文件之后接下来就是主要的步骤了：

## 第一步，将vcf转化为bed/bim/fam

bed/bim/fam这三个文件是phasing的常用谱系文件格式。做这一步转换的工具有很多，我们这里直接借助GATK的[VariantsToBinaryPed](https://www.broadinstitute.org/gatk/guide/tooldocs/org_broadinstitute_gatk_tools_walkers_variantutils_VariantsToBinaryPed.php)模块进行转换：

{% highlight bash %}

time java -Xmx8g -jar GenomeAnalysisTK.jar -T VariantsToBinaryPed \
    -R hg20.fa \
    -V chr22.vcf \
    --metaData sample.fam \
    -mgq 0 \
    -bed chr22.bed \
    -bim chr22.bim \
    -fam chr22.fam \
    -log gatk.log && echo "** done **" && sed 's/^chr//g' chr22.bim > t.bim && mv -f t.bim chr22.bim

{% endhighlight %}

这个执行命令的最后多加了一小步，将原来输出的.bim文件中第一列的chr22换成了22，这是因为如果不做这个小操作，接下来的plink步骤中，会直接报`ERROR: Problem reading BIM file, line 1`退出，原因就是它不允许chr的开头，至于具体的原因我也没去细查。

## 第二步，过滤genotype高missing rate和孟德尔错误的位点

{% highlight bash %}

plink=/com/extra/testing/bin/plink
time $plink --noweb --bfile chr22 --keep-allele-order --me 1 1 --set-me-missing --make-bed --out chr22.nomendel && echo "** nomendel done **" && time $plink --noweb --bfile chr22.nomendel --keep-allele-order --geno 0.05 --make-bed --out chr22.nomendel.filter && echo "** fileter done **"

{% endhighlight %}

## 第三步，phasing
这是phasing的最后一步了，这里分成两小步，phasing和输出格式转换：

{% highlight bash %}
# phasing
time shapeit2 \
    --duohmm \
    -W 5 \
    --input-bed chr22.nomendel.filter.bed chr22.nomendel.filter.bim chr22.nomendel.filter.fam \
    --input-map genetic_map.chr22.txt \
    -O hapData \
    --thread 1 && echo "** panel done **"

# 格式转换
time $shapeit2 -convert \
    --input-haps hapData \
    --output-vcf chr22.haps.vcf \
    --output-ref chr22.phased.hap chr22.phased.leg chr22.phased.sam && echo "** all done **"

{% endhighlight %}

以上输出结果中，`chr22.haps.vcf`便是进行phasing之后的结果，而`chr22.phased.hap`和`chr22.phased.leg`这两个文件是从`chr22.haps.vcf`中得到的，它们便是Imputation分析中的reference panel。










