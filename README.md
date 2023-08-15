# Rosetta分子对接流程指引

## 1. 前言
Rosetta是一款功能强大的蛋白质设计软件，它在蛋白质突变稳定性预测、分子对接等方面具有稳定的表现。Rosetta功能的调用需要一点命令行操作基础，本使用流程以***知乎：张自信***高质量流程为模板，Rosetta官方文档了解细节，原英文文献指明适用范围与注意事项，进行流程梳理，意图供公司里任何想在服务器上调用RosettaLigand进行分子对接的朋友。 
 
## 2. RosettaLigand 的应用 
### 运行前准备 
（1）服务器账号。运行Rosetta应用需要在Unix操作系统上进行，推荐在集成（clustered）计算机上运行，调用RosettaLigand还是需要一些算力的。如果条件不允许，本地的计算机其实也不是不行。 

（2）Rosetta。附当前（2023-8-14）时Rosetta在服务器上的路径： 
```
/mnt/4T_sdb/LHL/test/rosetta_src_2021.16.61629_bundle
```
 如果你看见这篇文献的时候服务器上都没有Rosetta，来瞧一瞧看一看:``https://github.com/holden-lyn/Rosetta_ddg_monomer_tutorial/blob/main/README.md`` 
 
（3）小分子处理程序。和原教程一样，推荐OpenBabel，可以通过图形界面（类似平时操作的电脑的界面），也可以通过命令行操作，给小分子加氢，如果不给小分子加氢，会影响RosettaLigand的表现。 

（4）配体构象生成程序。和原教程一样推荐BCL，文献中的BCL是本地的BCL，在本次实验中遇到过BCL服务器不可用的情况，所以有考虑之后把BCL给本地化。这里还提供手动生成构象的方法，但第一是生成的构象集不会和BCL算法生成的一样严谨，也容易遇到构象集文件格式出现问题等不容易处理的状况。 

（5）蛋白质(.pdb)和小分子(.sdf)结构文件，此处以 pgmB 蛋白（结构地址 ``https://www.uniprot.org/uniprotkb/P77366/entry``）和小分子 D-Allulose (551-68-8) （结构地址 ``https://pubchem.ncbi.nlm.nih.gov/compound/90008``）对接为例。 
 
（6）准备工作文件夹可能会是一个好习惯，用来装运行之后步骤运行RosettaLigand需要准备的一切文件。这个流程生成的文件很多，我不会希望这些文件混在某个文件夹原有的一大堆文件里头的。 
 
 
### 2.1. 蛋白质文件准备 
将蛋白质结构松弛。如果蛋白质结构的侧链是来自预测模型（而不是湿实验获得的模型），应该在松弛时去掉选项 

``-relax:coord_constrain_sidechains``。 
 
```
/mnt/4T_sdb/LHL/test/rosetta_src_2021.16.61629_bundle/main/source/bin/relax.mpi.linuxgccrelease -ignore_unrecognized_res -ignore_zero_occupancy false -use_input_sc -flip_HNQ -no_optH false -relax:constrain_relax_to_start_coords -relax:coord_constrain_sidechains -relax:ramp_constraints false -s pgmB.pdb
``` 
 
运行结束之后，获得文件"pgmB_0001.pdb",重命名输出结构为"pgmB_relaxed.pdb" 
``` 
mv pgmB_0001.pdb pgmB_relaxed.pdb
```
蛋白质文件准备就绪 
 
### 2.2. 配体准备 
 
### 2.3. 运行Rosetta应用 
### 2.4. 筛选 
 
# 参考借鉴，推荐阅读 
1. zhihu.com 张自信 https://zhuanlan.zhihu.com/p/621751210
2. 文章 "Rosetta and the Design of Ligand Binding Sites". 
3. Rosetta official documents: 
https://rosettacommons.org/demos/latest/tutorials/ligand_docking/ligand_docking_tutorial
https://www.rosettacommons.org/docs/latest/application_documentation/docking/ligand-dock

