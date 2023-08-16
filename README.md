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
<***记得插入图片说明***> 
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
- D-Allulose: 标注小分子的对接的位置，所以在之后的步骤需要在模型可视化软件里手动进行对接。 
- D-Allulose_conformers.pdb: 这是帮助params文件指定小分子构象库的文件，包含了之前生成的所有小分子构象。如果需要移动文件，应该和对应的.params文件一同移动。 
- D-Allulose.params: Rosetta运行RosettaLigand所能读取的文件格式。和构象库文件绑定，运行程序的时候需要和构象库文件处在同一个文件夹下。

<***记得插入图片，展示文件生成时候的输入日志，还有结构可视化文件里选定的一个配体构象，以及构象库，帮助理解***>
 
#### 2.2.3 用Chimera（或者其他结构可视化软件）将配体移动到蛋白质口袋中 
“Rosetta要求原子名称与molfile_to_params.py步骤中生成的名称匹配。即使具有正确放置了配体的起始结构，也应该将molfile_to_params.py生成的结构对齐到口袋中，以便原子命名正确。 
 
在进行蛋白设计时会包含蛋白与配体的多次对接，因此需要先指定配体每个原子的起始坐标，这样配体才能在指定的对接盒子中移动和扭转，与结合口袋发生相互作用。” 

**UCSF Chimera** 是本次教程中使用的，将小分子放进蛋白质口袋的软件。确认蛋白质的口袋位置需要一些先验知识，这次教程中情况相对简单。在https://www.uniprot.org/uniprotkb/P77366/entry 上可以看到蛋白质的活性位点是第11位的氨基酸，在Chimera中打开蛋白质.pdb结构文件，可以在Chimera中“选择Select”这个氨基酸达到高亮它的效果，同时也能发现这个蛋白质的口袋相当明显，活性位点所在的位置也与口袋的位置相吻合。 

#### Chimera 基本操作 
- 鼠标左键按住拖动：拖动模型结构旋转 
 
- 鼠标右键按住上下移动：模型结构 
 
- 鼠标中键按住移动：移动模型。 
 
如果需要模型直接的相对移动，例如蛋白质和小分子之间，需要在顶部的工具栏依次选择："Tools","General Controls","Model Panel"调出窗口，在弹出的窗口中取消勾选蛋白质结构模型的"A"也就是Active，单独移动小分子，再勾选上蛋白质结构的"A"将蛋白质和小分子同时旋转角度。重复这个过程，直到在多个角度上确认小分子已经移动到了想到对接的蛋白质口袋中。

<***记得插入图片显示：1.Chimera文件导入，modelpanel打开，disactive protein。 2. 放进口袋之后的小分子***>

 
### 2.3 运行Rosetta应用 
#### 2.3.1 编辑.resfile文件声明残基设计 
**resfile** 全称Residue Specification file，文件为.resfile格式。 
 
突变位点的声明可以决定输入的蛋白质结构里面有哪些位点可以突变，可以按照需求自由指定。本流程中，将pgmB与D-Allulose对接的操作，其实是来源于同时将pgmBmut（pgmB第十三位氨基酸G突变为R的突变体）与D-Allulose对接以对比突变前后结合亲和力的实验设计中。故设计两个resfile： 
- 其中一个指定不能改变氨基酸，但是可以进行蛋白质结构的微调：
```
NATAA #只允许原本的氨基酸（NATive Amino Acid）
start #"start"以上是resfile的“标题”，以下则是“主体”
```
- 另一个指定一个定向的点突变，同时允许蛋白质结构微调：
```
NATAA
start
13 A PIKAA R #四个空格分开的部分分别指定：第13个氨基酸，A链上，指定突变氨基酸，突变为R（如果可以突变为多个，则多个氨基酸不带空格的一输入进来，例如ACFYRT）。
```
注：resfile文件"start"以上的部分是**标题**，以下的部分是**主体**。标题声明了对所有未在主体中出现的部分要进行何种操作“NATAA”即允许在不改变氨基酸的情况下发生构象变化；主体部分作为特别指定的部分，会被优先确定，即标题和主体对同一残基都指明了操作的情况下，会以主体指定的操作为准。 

两个文件分别命名为"pgmB.resfile"和"pgmBmut.resfile"保存。 
  
 
#### 2.3.2 设计.xml文件（Rosetta脚本，RosettaScript） 
编辑调用RosettaScript需要的脚本文件（.xml格式），我理解的Rosetta脚本是可以将其他多个Rosetta功能一起调用起来的一个功能，具有较强的可编辑性，这里直接复制原文中的设计：
```
<ROSETTASCRIPTS>
    <SCOREFXNS>
        <ScoreFunction name="ligand_soft_rep" weights="ligand_soft_rep.wts"/>
        <ScoreFunction name="hard_rep" weights="ligandprime.wts"/>
    </SCOREFXNS>
    <TASKOPERATIONS>
        <DetectProteinLigandInterface name="design_interface" cut1="6.0" cut2="8.0" cut3="10.0" cut4="12.0" design="1" resfile="pgmB.resfile"/>
    </TASKOPERATIONS>
    <LIGAND_AREAS>
        <LigandArea name="docking_sidechain" chain="X" cutoff="6.0" add_nbr_radius="true" all_atom_mode="true" minimize_ligand="10"/>
        <LigandArea name="final_sidechain" chain="X" cutoff="6.0" add_nbr_radius="true" all_atom_mode="true"/>
        <LigandArea name="final_backbone" chain="X" cutoff="7.0" add_nbr_radius="false" all_atom_mode="true" Calpha_restraints="0.3"/>
    </LIGAND_AREAS>
    <INTERFACE_BUILDERS>
        <InterfaceBuilder name="side_chain_for_docking" ligand_areas="docking_sidechain"/>
        <InterfaceBuilder name="side_chain_for_final" ligand_areas="final_sidechain"/>
        <InterfaceBuilder name="backbone" ligand_areas="final_backbone" extension_window="3"/>
    </INTERFACE_BUILDERS> 
    <MOVEMAP_BUILDERS>
        <MoveMapBuilder name="docking" sc_interface="side_chain_for_docking" minimize_water="true"/>
        <MoveMapBuilder name="final" sc_interface="side_chain_for_final" bb_interface="backbone" minimize_water="true"/>
    </MOVEMAP_BUILDERS>
    <SCORINGGRIDS ligand_chain="X" width="15">
        <ClassicGrid grid_name="vdw" weight="1.0"/>
    </SCORINGGRIDS>
    <MOVERS>
        <FavorNativeResidue name="favor_native" bonus="1.00"/>
        <Transform name="transform" chain="X" box_size="3.0" move_distance="0.1" angle="5" cycles="500" repeats="1" temperature="5" rmsd="4.0"/>
        <HighResDocker name="high_res_docker" cycles="6" repack_every_Nth="3" scorefxn="ligand_soft_rep" movemap_builder="docking"/>
        <PackRotamersMover name="design_interface" scorefxn="hard_rep" task_operations="design_interface"/>
        <FinalMinimizer name="final" scorefxn="hard_rep" movemap_builder="final"/>
        <InterfaceScoreCalculator name="add_scores" chains="X" scorefxn="hard_rep"/>
        <ParsedProtocol name="low_res_dock">
            <Add mover_name="transform"/>
        </ParsedProtocol>
        <ParsedProtocol name="high_res_dock">
            <Add mover_name="high_res_docker"/>
            <Add mover_name="final"/>
        </ParsedProtocol>
    </MOVERS>
    <PROTOCOLS>
        <Add mover_name="favor_native"/>
        <Add mover_name="low_res_dock"/>
        <Add mover_name="design_interface"/>
        <Add mover_name="high_res_dock"/>
        <Add mover_name="add_scores"/>
    </PROTOCOLS>
</ROSETTASCRIPTS>
```
还是要知道基本的设计理念： 
 
“建议的方案基于使用RosettaScripts框架的RosettaLigand对接。它将优化配体在结合口袋中的位置（low_res_dock），重新设计周围的侧链（design_interface），并在设计的环境中优化相互作用（high_res_dock）。为了避免虚假突变，在每个位置（favor_native）给输入残基一个轻微的能量奖励。” 

将以上的设计保存为.xml格式文件， 此处命名为"**pgmB-DA_docking.xml**"，注意``resfile="pgmB.resfile"``处要编辑成对应的resfile；计算突变后pgmB与阿洛酮糖结合的设计文件中相同的地方则为``resfile="pgmBmut.resfile"``，设计文件命名为"**pgmbmut-DA_docking.xml**"。

#### 2.3.3 运行设计
确认所有所需文件都在工作文件夹： 
- 松弛后的蛋白质pgmB_relaxed.pdb
- 小分子三件套 D-Allulose.pdb (在图形软件中手动对接过), D-Allulose_conformers.pdb, D-Allulose.params
- 为野生型和突变体分别设计的残基说明文件 pgmB.resfile, pgmBmut.resfile
- Rosetta Design 设计文件（.xml格式） pgmB-DA_docking.xml, pgmBmut-DA_docking.xml
 
在工作文件夹里运行设计： 
```
$ROSETTA3/bin/rosetta_scripts.mpi.linuxgccrelease -ex1 -ex2 -linmem_ig 10 -restore_pre_talaris_2013_behavior -parser:protocol pgmB-DA_docking.xml -extra_res_fa D-Allulose.params -s "pgmB_relaxed.pdb D-Allulose.pdb" -nstruct 10 -out:file:scorefile result_pgmB-DA.sc
```
标注："-nstruct 10" 生成10个对接结构，"-out:file:scorefile result_pgmB-DA.sc" 指定输出的打分文件名为"result_pgmB-DA.sc"。等十个.pdb对接结构文件输出之后，和打分文件.sc一起移动到另一个文件夹"Result"，分别存放输入文件和输出文件会比较整洁。

接下来运行pgmBmut和DA的对接预测，将上述指令中设计文件替换为预先准备好的"pgmBmut-DA_docking.xml"，输出文件名更改为"result_pgmBmut-DA.sc"，运行：  
```
$ROSETTA3/bin/rosetta_scripts.mpi.linuxgccrelease -ex1 -ex2 -linmem_ig 10 -restore_pre_talaris_2013_behavior -parser:protocol pgmBmut-DA_docking.xml -extra_res_fa D-Allulose.params -s "pgmB_relaxed.pdb D-Allulose.pdb" -nstruct 10 -out:file:scorefile result_pgmBmut-DA.sc
```
同样是输出10个结构文件.pdb，一个打分文件.sc，整理之后可以选用打分较为理想的结构在UCSF Chimera或其他能以图像显示.pdb文件的软件中打开进行目视检测，一般打分文件中total_score越小（负值越大），则说明结构能量较低，相对稳定，被看好。根据需要获取输出结构中的信息，或者利用输出结构进行进一步筛选。
 
 
  
### 2.4 筛选 （做过之后编辑）


通过计算机模拟的对接结构，是比较难直接进行亲和力（Km）的预测的，但是通过RosettaLigand输出评分较高的对接结构仍有意义。本流程的其他适用场合，包括一个较通用的resfile设计，会在文末讨论。 
 
## 3. 参考借鉴，推荐阅读 
1. zhihu.com 张自信 https://zhuanlan.zhihu.com/p/621751210
2. 文章 "Rosetta and the Design of Ligand Binding Sites". 
3. Rosetta official documents: 
https://rosettacommons.org/demos/latest/tutorials/ligand_docking/ligand_docking_tutorial
https://www.rosettacommons.org/docs/latest/application_documentation/docking/ligand-dock

