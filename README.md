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
 如果你看见这篇文献的时候服务器上都没有Rosetta，来瞧一瞧看一看: 
``` 
https://github.com/holden-lyn/Rosetta_ddg_monomer_tutorial/blob/main/README.md
``` 
 
（3）小分子处理程序。可以通过图形界面（类似平时操作的电脑的界面，软件：OpenBabel，Avogadro），也可以通过命令行操作（OpenBabel），给小分子加氢，如果不给小分子加氢，会影响RosettaLigand的表现。 

（4）配体构象生成程序。和原教程一样推荐BCL，文献中的BCL是本地的BCL，在本次实验中遇到过BCL服务器不可用的情况，所以有考虑之后把BCL给本地化。这里还提供手动生成构象的方法，但是生成的构象集不会和BCL算法生成的一样严谨，也容易遇到构象集文件格式出现问题等不容易处理的状况。 

（5）蛋白质(.pdb)和小分子(.sdf)结构文件，此处以 pgmB 蛋白（结构地址 ``https://www.uniprot.org/uniprotkb/P77366/entry``）和小分子 D-Allulose (551-68-8) （结构地址 ``https://pubchem.ncbi.nlm.nih.gov/compound/90008``）对接为例。 
 
（6）准备工作文件夹可能会是一个好习惯，用来装运行之后步骤运行RosettaLigand需要准备的一切文件。这个流程生成的文件很多，我不会希望这些文件混在某个文件夹原有的一大堆文件里头的。 
 
 
### 2.1 蛋白质文件准备 
创建一个工作文件夹 ``mkdir <workdir>`` ``cd <workdir>`` 
本教程在服务器上的工作路径为 /mnt/4T_sdb/LHL/test/RosettaLigand_tutorial 
把蛋白质文件用xftp上传到工作文件夹 /mnt/4T_sdb/LHL/test/RosettaLigand_tutorial 准备将蛋白质结构松弛。如果蛋白质结构的侧链是来自预测模型（而不是湿实验获得的模型），应该在松弛时去掉选项 

``-relax:coord_constrain_sidechains``。 
 
```
/mnt/4T_sdb/LHL/test/rosetta_src_2021.16.61629_bundle/main/source/bin/relax.mpi.linuxgccrelease -ignore_unrecognized_res -ignore_zero_occupancy false -use_input_sc -flip_HNQ -no_optH false -relax:constrain_relax_to_start_coords -relax:coord_constrain_sidechains -relax:ramp_constraints false -s AF-P77366-F1-model_v4.pdb
``` 
 
运行结束之后，获得文件"AF-P77366-F1-model_v4.pdb_0001.pdb"（这里我选用了一个AlphaFold的预测模型）,重命名输出结构为"pgmB_relaxed.pdb" 
``` 
mv AF-P77366-F1-model_v4_0001.pdb pgmB_relaxed.pdb
```
松弛后的蛋白质文件（pgmB_relaxed.pdb）准备就绪 
 
### 2.2. 配体准备 
#### 2.2.1 为小分子按照pH值编辑氢原子 
（1）从pubchem上下载小分子结构文件.sdf，这里使用的是**3D模型**，``https://pubchem.ncbi.nlm.nih.gov/compound/90008``。 
（2）打开Avogadro，可以直接将小分子结构文件拖入窗口，也可以点"File"-->"New"从路径中选择打开的文件。文件在窗口中显示之后，从"build"中选择"add Hydrogen for pH"，在弹出的小窗口中输入pH值，点击"OK"确定，完成加氢原子。可以用文本编辑器，如windows自带记事本、VScode等分别打开加氢前后的小分子结构.sdf文件进行比对确认。 
#### 2.2.2 BCL服务器生成小分子构象库，用Rosetta功能转换成可供Rosetta读取的.params格式 
（1）进入Meiler Lab的BCL::Conf网络服务器 
```
http://carbon.structbio.vanderbilt.edu/index.php/bclconf
```
（2）上传加氢之后的.sdf小分子文件，输入需要的构象数量，这里为了加快运行速度，加上对构象数量的需求不高，选择了10个构象。网站上提供了将结果发送到邮箱的选项，可能会需要将"admin@meilerlab.org"加入受信邮箱地址，在原网页上等待结果，通常来说，一分钟左右能够生成结果。 
 
（3）将BCL生成的结果重命名为D-Allulose_conf_test.sdf，上传到服务器里的工作文件夹，用Rosetta自带的功能转换成.params格式（此处直接引用：params为Rosetta可读取的、用于存储小分子形状及化学性质信息的专有文件格式）。输入命令转换文件： 
```
/mnt/4T_sdb/LHL/test/rosetta_src_2021.16.61629_bundle/main/source/scripts/python/public/molfile_to_params.py -n LIG -p D-Allulose --conformers-in-one-file D-Allulose_conf_test.sdf
``` 
“其中，-n 指定在 pdb 和 params 文件中用来表示配体名称的 3 字符缩写，这里命名为 LIG 即配体 ligand 的缩写（需要注意的是，这里不能沿用 GLY 或者其他氨基酸、金属原子的缩写，否则生成的 params 文件会在后续突变结构时，与 Rosetta 自带的氨基酸或部分金属原子的 params 文件发生冲突）；-p 指定生成文件的命名。” （知乎：张自信） 
 
输入``ls``，可以看到生成了三个文件"D-Allulose", "D-Allulose_conformers.pdb", "D-Allulose.params"，三个文件依次的作用是： 
D-Allulose: 标注小分子的对接的位置，所以在之后的步骤需要在模型可视化软件里手动进行对接。
D-Allulose_conformers.pdb: 
 
 
### 2.3 运行Rosetta应用 
### 2.4 筛选 
 
# 参考借鉴，推荐阅读 
1. zhihu.com 张自信 https://zhuanlan.zhihu.com/p/621751210
2. 文章 "Rosetta and the Design of Ligand Binding Sites". 
3. Rosetta official documents: 
https://rosettacommons.org/demos/latest/tutorials/ligand_docking/ligand_docking_tutorial
https://www.rosettacommons.org/docs/latest/application_documentation/docking/ligand-dock

