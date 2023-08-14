# Rosetta分子对接流程指引

## 1. 前言
Rosetta是一款功能强大的蛋白质设计软件，它在蛋白质突变稳定性预测、分子对接等方面具有稳定的表现。Rosetta功能的调用需要一点命令行操作基础，本使用流程以***知乎：张自信***高质量流程为模板，官方文件了解细节，原英文文献指明适用范围，进行流程梳理，意图供公司里任何想在服务器上调用RosettaLigand进行分子对接的朋友。 
 
## 2. RosettaLigand 的应用 
### 运行前准备 
（1）服务器账号 

（2）Rosetta，附当前（2023-8-14）时Rosetta在服务器上的路径： 
```
/mnt/4T_sdb/LHL/test/rosetta_src_2021.16.61629_bundle
``` 
（3）小分子处理程序 

（4）配体构象生成程序 

（5）蛋白质(.pdb)和小分子(.sdf)结构文件，此处以 pgmB 蛋白和小分子 D-Allulose (551-68-8) 对接为例 
 
（6）准备工作文件夹，用来装运行之后步骤运行RosettaLigand需要准备的一切文件。 
 
 
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

