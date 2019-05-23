### 结构化与版本控制方案设计

#### 存储方式：

项目的代码存放在：/projects/$uuid/$project_name/code，映射到容器里是：/code

数据集存放在：/datasets/$uuid/$dataset_name，映射到容器是：/data/$dataset_name

项目的输出存放在: /projects/$uuid/$project_name/outputs/$job_id，映射到容器里是：/output。

可以选择某个项目的输出作为输入，即将/projects/$uuid/$project_name/outputs/$job_id映射到容器里的/input。规定：一次运行只能选择一个input来源

属于同一job的containers都做上述4项同样的映射。

#### 权限管理

当任务是以debug模式运行时：

- /code目录可写，永久存储，允许用户修改代码
- /data/$dataset_name中，如果是用户自己的数据集可写，不是自己的数据则只读。已公开或分享给部分人的数据集都不允许修改。
- /input目录只读
- /output目录可写

当任务以非debug模式运行时：

- /code目录可写，非永久存储，用户修改的代码并不会被保存
- /data/$dataset_name目录只读
- /input目录只读
- /output目录可写，用户仅可以往/output中输出及保存相应文件

#### 版本控制

* 使用git进行版本控制
* 代码放置在NFS上，如果用户的项目目录没有进行git init，则程序要替它进行
* 在集群里建立一个git服务器，用于存放代码.（高可用）
* 当用户使用debug运行时，/code目录是NFS上的目录，这样用户可以在debug任务里改代码；当用户非debug运行时，*/code目录实际上并不映射，而是在容器内创建*，并从git服务器上把代码拉下来
* 当项目的任务运行时，后台要先做这些操作：对项目目录进行git commit、git push、git tag，并把tag和任务关联
* 当用户在debug模式下，对code目录进行了破坏，由UI2.0对它进行处理

#### 项目公开和克隆的逻辑

公开：

1. 选择欲公开的项目的某次任务（debug任务不允许公开）
2. 后端记录这次公开的代码的tag、数据集
3. 如果有input依赖的任务，暂时不允许公开
4. 同一个项目可以再次公开，后端记录公开历史

克隆：

1. 选择某个已公开的项目
2. git clone响应tag的代码到 /projects/$uuid/$project_name/code
3. 同时，在git服务器上建立相应的git仓库（~~先按建立repo处理，也许建立分支也是一个不错的选择~~），将git仓库和项目关联
4. 上述的2、3步骤在新建项目时同样要做

#### 在网页端（以及未来的CLI）

- 可以通过项目文件管理及代码编辑器对项目存在在NFS上的/code目录读写，修改、新增、删除文件
- 可以在web端对数据集中的readme.md做修改
- 支持能直接将项目的output转化为数据集

#### 过渡时期

过渡期间，将原先的/userhome改名为/userhome_obsoleted（当以2.0方式启动时，只读挂载），这样用户可以在容器内把原先的内容拷贝到新的目录下，以完成升级。当过渡完毕后，/userhome_obsoleted不再映射，保留一段时间就后，原/userhome删除。**在7月31日完全取消/userhome**。

老2.0到新2.0的迁移工具开发

#### 其他

1. rest-server接收job时要强制检验版本
2. 二季度中，如果用户需要上传预训练模型，则通过数据集的形式管理

