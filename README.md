# Rosetta分子对接流程指引

## 1. 前言
Rosetta是一款功能强大的蛋白质设计软件，它在蛋白质突变稳定性预测、分子对接等方面具有稳定的表现。Rosetta功能的调用需要一点命令行操作基础，本使用流程以知乎高质量流程为框架，官方文件了解细节，原英文文献指明适用范围，进行流程梳理，意图供公司里任何想在服务器上调用RosettaLigand进行分子对接的朋友。 
 
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
 
 
### 2.1. 蛋白质文件准备 
将蛋白质结构松弛。如果蛋白质结构是一个预测模型（而不是湿实验获得的模型），应该在松弛时加入选项``-relax:coord_constrain_sidechains``
 
### 2.2. 配体准备 
### 2.3. 运行Rosetta应用 
### 2.4. 筛选 
 
# 参考借鉴，推荐阅读 
1. zhihu.com 张自信 https://zhuanlan.zhihu.com/p/621751210
2. 文章 "Rosetta and the Design of Ligand Binding Sites". 
3. Rosetta official documents: 
https://rosettacommons.org/demos/latest/tutorials/ligand_docking/ligand_docking_tutorial
https://www.rosettacommons.org/docs/latest/application_documentation/docking/ligand-dock

