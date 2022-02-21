# SC3D论文&代码阅读


《Leveraging_Shape_Completion_for_3D_Siamese_Tracking》论文&代码阅读

<!--more-->

---

- 作者：*Silvio Giancola*, Jesus Zarzar*, and Bernard Ghanem*
- 机构：*King Abdullah University of Science and Technology (KAUST)*
- 论文水平：**CVPR19**
- 关键词：**Shape Completion && Siamese Tracker && Model Update**

---

## 论文摘要

本文提出了一种基于形状补全网络以及孪生网络的单目标跟踪器，借鉴了《Learning representations and generative models for 3d point clouds 》这篇论文的思想，将形状补全网络中的自编码器融入到孪生网络的框架中，使用自编码器的编码结构作为孪生网络的特征提取网络，通过编码之后解码这一过程将形状补全损失加入进来，训练编码器网络，使其编码的特征带有形状信息，更好的用于孪生网络的匹配。

[代码](https://github.com/SilvioGiancola/ShapeCompletion3DTracking)

[视频](https://www.youtube.com/watch?v=2-NAaWSSrGA)

## 论文解读

---

### 1. 网络框架

![](https://raw.githubusercontent.com/shmilywh/PicturesForBlog/master/2021/07/05-10-25-32-2021-07-05-10-25-23-image.png)

如图所示，整体的网络架构分为两部分， 分别是自编码器网络和孪生网络，自编码器就是将输入编码再解码得到形状更加丰富的点云, 孪生网络就是使用编码器的输出向量计算相似度. 网络的具体描述如下:

**自编码器网络**是借鉴形状补全网络的思想，对输入的一组模板点云(*Model Shape*)以及一组搜索点云(*Candidate Shapes*)进行编码(**Φ**)和解码(**Ψ**)，二者组成了自编码器。实际上，代码中作者只用了三层一维卷积作为编码器，将输入的(**B, 3, 2048**)维度的点云编码为(**1, 128**)维度的向量形式的潜在表述，进而通过两层的全连接层，将(**1, 128**)的向量输出为(**1, 6144**), 进而**reshape**为(**3, 2048**)作为解码器的输出。使用自编码器的作用在于，通过训练，构建形状补全损失(*Shape Completion Loss*)，进而让网络学习如何通过自编码来得到相对于输入点云的更好的形状表述，同时让编码器编码得到的向量拥有点云的几何形状信息， 也让解码器解码得到的点云对应的 *bbox* 更加精准。 编码器产生的带有形状信息的向量也对后面的孪生网络计算相似度有帮助。

**孪生网络**在本文中由两个编码器网络组成，输入的两组点云通过相同的编码器网络，输出两个代表着潜在几何信息的向量，对这两个向量进行相似度度量，就可以得到相似度最大的 *Candidate pc*，本文使用的是余弦相似度，孪生网络对应的跟踪损失是均方差损失(*MSE Loss*)。

这里还有一点要说明的是，因为本论文的主要目的是做跟踪，所以作者强调网络的主要结构是孪生网络，自编码网络只是帮助孪生网络的相似度计算更加精准。

### 2. 模型的训练

![](https://raw.githubusercontent.com/shmilywh/PicturesForBlog/master/2021/07/05-10-25-38-2021-07-05-10-25-32-image.png)

Example of model completion

**输入输出分别是什么?**

在训练阶段，网络的两个输入维度都是(1, 3, 2048)，经过 *model* 得到的输出是

1) 当前搜索点云与模板点云的相似度: 其实也就是孪生网络的输出, 用来计算相似度损失
2) 自编码之后的点云: 自编码器的输出, 输出维度也是(3, 2048), 可以用来计算形状补全损

**输入的模板点云与搜索点云如何确定?**

*Model Shape* 是一个特定的 *track_id* 对应的所有帧的点云级联而成，级联之后还需要下采样到2048个点，以和模型的输入尺寸对齐，进而作为这一次训练的模板点云。 

*Candidate Shapes* 则是在某一帧的真值框附近根据 *Kalman filter* 采样生成的一定数量的候选框内的点云，比如当前帧是1，采样数量是125，那么输入给 *data_loader* 的参数就会是 0-124 之间， 如果输入57，代表的就是在第一帧点云真值框附近，采样得到的第58个框，框中对应的点云作为这一次训练的搜索点云。

👉🏿 **Loss是如何定义的?**    

在SC3D中, 总体的loss包括两部分, 第一个就是计算编码向量之间的相似度, 进而求得的相似度损失，这里其实就是典型的孪生网络结构，用两个编码器作为特征提取的网络，计算得到的向量之间的相似度，值得一提的是，本文在训练阶段使用的模板点云是一个序列所有帧级联之后下采样得到的点云，这样得到的点云形状信息更加完整，相比于以往的使用孪生网络跟踪的方法，这种方式可以尝试下，个人感觉应该能一定程度上解决三维点云感知中点云稀疏性带来的问题。(因为相当于训练时模板信息较为丰富，特征较为完整，进而提高搜索点云与模板点云之间的相似度)      

第二个损失是算模型的形状完成损失，作者想让自编码器网络具有能够补全输入点云的形状的能力, 那么就需要在训练阶段通过监督信号不断纠正网络的学习结果, 让网络能够学习到足够匹配的参数, 进而达到形状补全的目的, 所以这里作者加了一个形状补全损失, 用来让模型拥有补全形状的能力。        在做 *loss* 回传时，取两种损失的加权和，如下：

     loss = loss1 + lambda_completion * loss2

### 4. 模型的测试

**输入输出分别是什么?**

与训练不同，测试阶段模型的输入是一组数据，前文也提到了测试数据集的构建以及搜索框的生成策略，测试阶段是将生成的一组 *Candidate PCs* 级联到一起，*Model PC*则是根据不同的策略更新之后，与 *Candidate PCs* 的维度对齐(其实就是将 *Model PC* 重复了若干次)，所以将这两组数据一起输入给网络。

输出得到的相似度得分同样是一组，在这一组得分中选择分数最高的，根据对应的id找到在 *Candidate Boxes* 中的*Bbox*， 作为当前帧的搜索结果，这样就完成了一次测试的过程。

**输入的模板点云与搜索点云如何确定?**

在测试阶段，第一帧的 *Model Shape* 是由真值得到的，并且只有一帧的数据，在后续帧中不断融合新的信息，这里的数据融合分为两种思路，要么融合点云，要么融合编码之后的向量，作者提出融合 *latent vector* 会降低内存的消耗，但是论文给出的测试数据显然融合点云更为精准。对于融合之后如何更新模板点云，可以选定第一帧、选定前一帧或者选定前面所有帧进行级联，如果选择融合向量，同样有不同的融合方案，详见代码

Candidate Shapes 的选择同样也有几种不同的方案，因为作者提出的更新搜索空间的方式，是基于某一帧的bbox，在其周围根据不同的方式(*KalmanFilteringr / GaussianMixtureModel / ParticleFiltering / ExhaustiveSearch*) 生成若干个 *Candidate boxes，*而这个bbox的选取也有三种方式，分别是取前一帧的跟踪结果，前一帧的真值以及当前帧的真值来计算 *Candidate boxes，*详见代码

**测试结果如何得到？**

得到相似度向量，取得分最高的 ***id***，就可以找到在搜索点云中对应的 ***id***，这样就可以把这个 ***id*** 对应的 ***bbox*** 作为最终的跟踪结果。

## 代码阅读

### 1. *class kittiDataset()*

1. ***getSceneID(self, split):***
   
    针对 Kitti 数据集的 *tracking*下的序列，将 *train*下的 00-16 作为了训练集， 17-18作为验证集，19-20作为测试集

2. ***getListOfAnno(self, sceneID, category_name="Car"):***
   
   - 获取 scene 列表，然后遍历每一个 scene，得到对应的label
     
     - 先通过 pandas 读取label，存为 Dataframe格式
       
       ```python
       label_file = os.path.join(self.KITTI_label, scene + ".txt")
                   #读取标签txt文件
                   df = pd.read_csv(
                       label_file,
                       sep=' ',
                       names=[
                           "frame", "track_id", "type", "truncated", "occluded",
                           "alpha", "bbox_left", "bbox_top", "bbox_right",
                           "bbox_bottom", "height", "width", "length", "x", "y", "z",
                           "rotation_y"
                       ])
       ```
     
     - 筛选出当前跟踪的类别
       
       ```python
       df = df[df["type"] == category_name]  # 筛选出类别是car的标签
       ```
     
     - 在label中插入 scene
       
       ```python
       df.insert(loc=0, column="scene", value=scene)  # 在标签中插入一列表示这是哪个场景
       ```
     
     - 遍历 Dataframe 中的每一个 id，将所有帧的label 根据不同的 track_id 进行存储
       
       ```python
       for track_id in df.track_id.unique():
               # 找到所有属于当前id的数据,保留下来
               df_tracklet = df[df["track_id"] == track_id]
               # 因为行标签(行号)在前面的筛选中错乱了,这里重置一下,避免不必要的错误
               # drop=True: 把原来的索引index列去掉，丢掉。
               # drop=False: 保留原来的索引（以前的可能是乱的）
               df_tracklet = df_tracklet.reset_index(drop=True)
               # 使用df_tracklet.iterrows() 来进行迭代的生成, 返回值是对应的行号以及标签,这里舍弃行号,只保留了标签信息
               tracklet_anno = [anno for index, anno in df_tracklet.iterrows()]
               # 再存进列表中
               list_of_tracklet_anno.append(tracklet_anno)
       ```
     
     - 注意：这里遍历结束之后,因为循环时是取 df.track_id.unique() 中的元素, 其没有固定的排列顺序,所以得到的 list_of_tracklet_anno 的索引,与track_id没什么关系

3. ***getBBandPC(self, anno):***
   
   - 根据传入的label标签，得到calib文件，以及对应的 transf_mat
   - 根据 anno 和 transf_mat 读取对应的点云以及 box

4. ***getPCandBBfromPandas(self, anno, calib):***
   
   - 根据 label 以及 transf_mat 矩阵（传入的calib实际上就是 transf_mat 矩阵）得到整帧点云和box
   
   - 这里将点云和box抽象成了两个类，将读取到的数据以这两个类的形式存储，便于后面处理
     
     - PointCloud 类
       
         通过点云数据创建，点云的shape是（4，n）注意这里直接将点云进行了transform，**坐标系变换为相机坐标系**
       
       ```python
       velodyne_path = os.path.join(self.KITTI_velo, anno["scene"],
                                    '%06d.bin'%(anno["frame"])) #f'{box["frame"]:06}.bin')
       #从点云的.bin文件中读取点云数据并且转换为4*x的矩阵，且去掉最后的一行的点云的密度表示数据
       PC = PointCloud(
           np.fromfile(velodyne_path, dtype=np.float32).reshape(-1, 4).T)
       #将点云转换到相机坐标系下　因为label中的坐标和h,w,l在相机坐标系下的
       PC.transform(calib)
       ```
     
     - Box 类
       
         通过（center，size，orientation）创建，这里将读取到 yaw角转换成了四元数
       
       ```python
       center = [anno["x"], anno["y"] - anno["height"] / 2, anno["z"]]
       size = [anno["width"], anno["length"], anno["height"]]
       #下面这个函数是将roy角转换成四元数吧
       orientation = Quaternion(
           axis=[0, 1, 0], radians=anno["rotation_y"]) * Quaternion(
               axis=[1, 0, 0], radians=np.pi / 2)
       # 用中心点坐标和w,h,l以及旋转角来初始化BOX这个类
       BB = Box(center, size, orientation) 
       ```
       
         注意：kitti数据集的坐标系定义为
       
       - y轴向下为正，并且center是底面的中心（向下为正，所以也就是上面的中心）
       
       - 这里计算center的时候，计算的是box的中心，所以y方向坐标减去了半个高度，也是方便后续corners的计算
       
       - Box类中存放了四元数作为角度，也可以调用类中的属性 rotation_matrix 返回对应的角度
       
       - Box类中也有返回 corners 坐标的方法，返回（3，8）的坐标值
         
           这里的corners方法，分为以下三步
         
         - 根据长宽高，建立一个坐标在原点的 box
         - 根据 rotation_matrix 将box旋转对应的角度
         - 根据 center 的坐标将box平移至对应位置，这样就得到了最终的带有角度的corners
       
       - Box类中还有一些操作，旋转、坐标系变换、平移等

---

### 2. *class SiameseDataset(Dataset)*

- list_of_tracklet_anno
  
    装有所有 scene 下的每个track_id 的 label

- list_of_anno
  
    获取所有标签，不考虑track_id：根据得到的 *self***.***list_of_tracklet_anno*， 不论 *id*， 把每一帧的数据顺序放入一个列表中，得到 *self***.***list_of_anno*， 这里的 *self**.**list_of_anno* 就是把 *list_of_tracklet_anno* 的 *shape* 从(*track_id*, *frames*) 变成 (*track_id***frames*, )

---

### 3. ***class SiameseTrain(SiameseDataset)***

- SiameseTrain
  
  ```python
  class SiameseTrain(SiameseDataset):
      """
      用来训练SC3D的数据集, 几个重要函数说明如下:
      __init__ : 初始化各种参数,并且获得list_of_BBs以及list_of_PCs,注意这里的list_of_PCs指的是bbox中的点云,不是整帧点云
      __getitem__ : 调用getitem, 这里就是根据传入的index获得对应的训练数据并返回
      __len__ : 获取数据集的长度, 这里将实际的长度乘了一个数字, 这个数字是在每一帧的搜索区域周围生成候选区域的数量
  
      几个重要数据说明如下:
      num_candidates_perframe: [Int]在当前搜索点云框周围生成的候选框的数量
      list_of_anno: [List] 存放label的列表, 里面有 track_id*frames 个数据, 按顺序从第一个序列的第一个track_id开始,存放每一帧的label
      model_PC: [List] 存放模板点云的列表, 里面有 track_id 个PointCloud类的点云数据,将每一个id的所有帧级联在一起作为模板
      list_of_PCs: [List] 存放搜索点云的列表, 里面有 track_id*frames 个点云数据, 这里的不是整帧点云,是bbox内的点云
      list_of_BBs: [List] 存放label的列表, 里面有 track_id*frames 个label数据,对应label文件中的一行label数据
      """
      def __init__(self,
                   model,
                   path,
                   output,
                   split="",
                   category_name="Car",
                   regress="GAUSSIAN",
                   sigma_Gaussian=1,
                   offset_BB=0,
                   scale_BB=1.0):
          super().__init__(
              model=model,
              path=path,
              split=split,
              category_name=category_name,
              regress=regress,
              offset_BB=offset_BB,
              scale_BB=scale_BB)
  
          self.sigma_Gaussian = sigma_Gaussian
          self.offset_BB = offset_BB
          self.scale_BB = scale_BB
  
          self.num_candidates_perframe = 147
  
          logging.info("preloading PC...")
          self.list_of_PCs = [None] * len(self.list_of_anno)
          self.list_of_BBs = [None] * len(self.list_of_anno)
          # 遍历,找到每一帧对应的label
          for index in tqdm(range(len(self.list_of_anno))):
              anno = self.list_of_anno[index]
              # NOTE 获取对应的点云和bbox信息,注意这里的点云是整帧点云!
              PC, box = self.getBBandPC(anno)
              # 将得到的点云处理,通过bbox,裁剪出在bbox里面的点,这里扩充了点云的边界
              new_PC = utils.cropPC(PC, box, offset=10)
              # NOTE 将所有帧中的bbox以及bbox中的点云存成列表,这里的所有帧指的是不考虑track_id的情况,全部的点云帧
              self.list_of_PCs[index] = new_PC
              self.list_of_BBs[index] = box
          logging.info("PC preloaded!")
  
          logging.info("preloading Model..")
          # 将一个track_id轨迹中的所有帧的点云汇集到一个bbox下，作为模板点云， 也就是这里的 self.model_PC
          self.model_PC = [None] * len(self.list_of_tracklet_anno)
  
          # 遍历每一个track_id 这里的 len(self.list_of_tracklet_anno) 就是track_id的种类数量
          for i in tqdm(range(len(self.list_of_tracklet_anno))):
              list_of_anno = self.list_of_tracklet_anno[i]
              PCs = []
              BBs = []
              cnt = 0
              # 遍历同一个track_id下的每一帧
              for anno in list_of_anno:
                  #　获取整帧点云以及bbox信息
                  this_PC, this_BB = self.getBBandPC(anno)
                  PCs.append(this_PC)
                  BBs.append(this_BB)
                  # NOTE model_idx相当于是track_id, 但是不同, 其代表着排序之后的索引,每一个索引对应一个track_id, 但是具体对应哪个id, 随机
                  anno["model_idx"] = i
                  # reletive_idx相当于是对每一帧进行编号,一个轨迹序列的开始到结束
                  anno["relative_idx"] = cnt
                  cnt += 1
              # 通过这个函数获取模板点云, 模板点云是将同一个track_id的所有点云帧的bbox中的点云级联在一起的
              self.model_PC[i] = utils.getModelAndSave(PCs, BBs, offset=self.offset_BB, scale=self.scale_BB,
                                          save=True, path=os.path.join(output, category_name), name=category_name + str(i))
  
          logging.info("Model preloaded!")
  
      def __getitem__(self, index):
          return self.getitem(index)
  
      def getAnnotationIndex(self, index):
          # 每一个anno对应num_candidates_perframe个index, 所以需要除以num_candidates_perframe
          return int(index / self.num_candidates_perframe)
  
      def getSearchSpaceIndex(self, index):
          # 计算搜索空间的id,其实这个id没什么实际意义,如果非零, 那么代表当前这一批的搜索空间的序号
          return int(index % self.num_candidates_perframe)
  
      def getPCandBBfromIndex(self, anno_idx):
          this_PC = self.list_of_PCs[anno_idx]
          this_BB = self.list_of_BBs[anno_idx]
          return this_PC, this_BB
  
      # NOTE 这里传入的index, 实际上取决于构建的数据集的长度,在这个数据集类中,__len__对应的长度是将所有点云帧的长度乘以了 num_candidates_perframe
      # NOTE num_candidates_perframe的含义就是 "要生成的候选区域的数量", 而传入的index就是按照点云帧长度乘以num_candidates_perframe来计算的
      # NOTE 所以在通过index计算对应的label以及搜索空间的索引时,需要将index的数值除以 num_candidates_perframe
      def getitem(self, index):
          """根据index的传入,计算并得到 当前的点云(this_pc)\候选区域的点云(sample_pc)\上一帧的点云(gt_pc)\一整个序列的点云(model_pc)
          几个重要的变量说明如下:
              anno_idx: 搜索点云的id, 通过传入的index计算得到, 用来得到搜索点云的label以及bbox等信息
              sample_idx: 搜索空间的id, 用来计算sample_offsets,如果不是0,就需要通过滤波方法计算sample_offsets
              sample_offsets: 在搜索点云的周围采样一定数量的bbox, 这个offset相当于是一个偏置, 可以根据offset的值来计算当前搜索点云框周围的候选框
          """
          # 得到id
          anno_idx = self.getAnnotationIndex(index)
          sample_idx = self.getSearchSpaceIndex(index)
  
          # 计算得到sample_offsets
          if sample_idx == 0:
              sample_offsets = np.zeros(3)
          else:
              # PROBLEM 这里如何得到采样偏置的?具体没太看懂
              gaussian = KalmanFiltering(bnd=[1, 1, 5])
              sample_offsets = gaussian.sample(1)[0]
  
          # 搜索点云的label
          this_anno = self.list_of_anno[anno_idx]
  
          # NOTE 由于前面处理的方法, this_pc的边界比bbox的大小要大,所以后续得到的各种点云(sample_pc, gt_PC等都需要根据bbox重新裁剪)
          # 得到当前帧搜索点云中的bbox以及bbox中的点云
          this_PC, this_BB = self.getPCandBBfromIndex(anno_idx)
  
          # 根据当前的bbox以及offset, 创建搜索候选框
          sample_BB = utils.getOffsetBB(this_BB, sample_offsets)
  
          # 根据得到的搜索候选框, 从当前搜索点云中根据搜索候选框裁剪出候选区域的点云
          sample_PC = utils.cropAndCenterPC(
              this_PC, sample_BB, offset=self.offset_BB, scale=self.scale_BB)
  
          # 这里做了一个处理,如果采样得到的点云数量小于20, 那么随机选一个在(0,  self.__len__())之间的整数再输入,重新采样
          if sample_PC.nbr_points() <= 20:
              return self.getitem(np.random.randint(0, self.__len__()))
          # 对采样的点云进行标准化, 得到torch型的张量, 这里因为输出的是 (1, 3, N), 所以取0
          sample_PC = utils.regularizePC(sample_PC, self.model)[0]
  
          # 判断当前的id属于轨迹中的那一帧,如果是0, 意味着是第一帧, 那么前一帧设置为0, 否则就是当前帧减1
          if this_anno["relative_idx"] == 0:
              prev_idx = 0
          else:
              prev_idx = anno_idx - 1
  
          # 根据前一帧的id,计算对应的点云以及bbox, 并作中心化,作为真值groud-truth, 后来再进行点云数量的判断,以及尺寸标准化
          gt_PC, gt_BB = self.getPCandBBfromIndex(prev_idx)
          gt_PC = utils.cropAndCenterPC(
              gt_PC, gt_BB, offset=self.offset_BB, scale=self.scale_BB)
          if gt_PC.nbr_points() <= 20:
              return self.getitem(np.random.randint(0, self.__len__()))
          gt_PC = utils.regularizePC(gt_PC, self.model)[0]
  
          # 根据之前计算的model_idx获得当前track_id的model_pc, 因为点云数量比较多,所以没有进行数量判断
          model_idx = this_anno["model_idx"]
          model_PC = self.model_PC[model_idx]
          model_PC = utils.regularizePC(
              model_PC, self.model)[0]
  
          if "IOU" in self.regress.upper():
              # 直接计算bbox转换为二维的poly之后的交并比, 作为IOU
              score = utils.getScoreIoU(this_BB, sample_BB)
          elif "GAUSSIAN" in self.regress.upper():
              score = utils.getScoreGaussian(sample_offsets, self.sigma_Gaussian)
          elif "HINGE" in self.regress.upper():
              # 相比于第一个方法,这个IOU小于0.5都被当作0
              score = utils.getScoreHingeIoU(this_BB, sample_BB)
  
          # NOTE sample_PC, gt_PC, model_PC 都是 torch.Size([3, 2048])
          return sample_PC, gt_PC, model_PC, score
  
      def __len__(self):
          nb_anno = len(self.list_of_anno)
          return nb_anno * self.num_candidates_perframe
  ```

训练数据集的构建基于上面得到的 *SiameseDataset* 类，创建 *SiameseTrain*

代码中几个函数的说明：

`__init__ `: 初始化各种参数,并且获得 *list_of_BBs* 以及 *list_of_PCs* ,注意这里的 *list_of_PCs* 指的是 *bbox* 中的点云,不是整帧点云

`__getitem__` : 调用 *getitem*, 这里就是根据传入的 *index* 获得对应的训练数据并返回，这里输入的 *index* 其实指的是 candidate_box 的序号，如果每个 ref_box 周围需要生成 125个 *candidates*，那么第一个ref_box 对应的 *candidate* 的 *index*其实就是 0-124

*`__len__`* : 获取数据集的长度, 这里将实际的长度乘了 *num_candidates_perframe*, 是指在每一帧的搜索区域周围生成候选区域的数量

注意：

*SiameseTrain* 类返回的用于训练的数据分别是 *sample_PC, gt_PC, model_PC, score* ，分别代表着当前候选框中的点云，读上一帧的label获得的点云，以及对应track_id的轨迹上所有点云级联之后得到的点云， 这三种不同的点云都进行了同样的处理，中心化、旋转角度对齐、以及下采样到与模型一致的尺寸(一般为2048)。虽然 *model_PC* 级联了一个轨迹上所有的点云，但是为了计算相似度，也需要下采样到2048个点，只不过拥有了更加完整的形状信息。

---

### 4. *class SiameseTest(SiameseDataset)*

- SiameseTest
  
  ```python
  class SiameseTest(SiameseDataset):
  
      def __init__(self,
                   model,
                   path,
                   split="", #"Test"
                   category_name="Car",
                   regress="GAUSSIAN",
                   offset_BB=0,
                   scale_BB=1.0): # offset_BB = 0 scale_BB = 1.25
          super().__init__(
              model=model,
              path=path,
              split=split,
              category_name=category_name,
              regress=regress,
              offset_BB=offset_BB,
              scale_BB=scale_BB)
          self.split = split
          self.offset_BB = offset_BB
          self.scale_BB = scale_BB
  
          def getitem(self, index):
              list_of_anno = self.list_of_tracklet_anno[index]
              PCs = []
              BBs = []
              for anno in list_of_anno:
                  # this_PC里面是当前帧的所有点云转换到相机坐标系下的点云数据
                  # this_BB是一个字典　里面的键对对应着中心点，whl,以及四元数角
                  this_PC, this_BB = self.getBBandPC(anno)
                  PCs.append(this_PC)
                  BBs.append(this_BB)
              return PCs, BBs, list_of_anno
  
          def __len__(self):
              # 返回label中要跟踪的车辆的个数
              return len(self.list_of_tracklet_anno)
  ```

测试数据集的构建同样基于上面得到的 *SiameseDataset* 类，创建 *SiameseTest*

测试阶段输入的index与训练不同,这里只需要输入一个index, 这个index代表着 list_of_tracklet_anno 中的索引


