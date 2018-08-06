# <p align = "center"> SVN和Git笔记 </p>
## 概念
1. SVN是集中式版本控制系统，Git是分布式版本控制系统。

2. Git也有自己的集中式版本库或服务器，同时git在自己电脑上clone的本地版本库和中心版本库一模一样，可以在本地进行SVN下需要联网才能操作的版本控制工作。Git通过`git pull`与`git push`与中心版本库保持同步。

3. 在SVN和Git中，我们可以直接操作的工程目录叫做工作区。个人理解SVN就是工作区与中心版本库之间的交互。而Git是先将工作区的改动添加（`git add`）到其特有的暂存区，再从暂存区提交到本地版本库，最后是本地版本库与中心版本库的同步，是工作区、暂存区、本地版本库、中心版本库之间的交互。

4. 关于分支，在SVN中，因为大家都与中心版本库交互，并且分支都在中心版本库里，会有频繁的主线到分支、分支到主线的合并过程，某种程度上会对版本控制造成不便。在Git里，可以在本地版本库随意建立分支，只需将合并后的本地主线同步到中心版本库对应的线路（可能不止一条线路）即可。

5. 关于可视化工具，SVN推荐使用CornerStone，git推荐使用Github桌面版或者SourceTree。

## SVN配置
- 在本地创建svn目录,如将其放在/usr/local/中（或其他你想要的文件夹）。

	```
	$ sudo mkdir /usr/local/svn
	```

- 在svn目录下创建一个名为code的仓库。

	```
	$ cd /usr/local/svn
	$ sudo svnadmin create code
	```

- 执行`ls /usr/local/svn/code`查看code目录下的文件。结构如下所示：

	```
	README.txt	db		hooks
	conf		format		locks
	```

- 执行`ls /usr/local/svn/code/conf`查看conf目录如下：

	```
	authz		hooks-env.tmpl	passwd		svnserve.conf
	```
- 在Finder中，对这3个文件进行修改，即可完成配置。配置前，可能需要先修改这些文件的访问权限。

	1. 配置svn用户权限

		修改svnserve.conf，将如下几项的"#"和"每行开头的空格"去掉(如果不去掉每行开头的空格会导致运行时配置报错)
	
		```
		# anon-access = read   	
		# auth-access = write  
 		# password-db = passwd 
		# authz-db = authz  
		```
	2. 添加用户,修改passwd文件

		在[user] 下面添加用户,格式为username = password
		
		```
		abc = 123
		Will = 123456
		```
	3. 配置用户组和用户的权限，修改authz文件

		在[groups] 下添加用户组,格式为groupname=user1,user2,user3....
	
		```
		[groups]
		rootgroup = abc, will, Tom
		```
		在[/] 下面对用户组进行权限配置（组名前需要添加@）
	
		```
		[/]
   		@rootgroup = rw
		```
		或者在[/] 下面对指定用户进行权限配置
	
		```
  	 	[/]
   		Will = rw
   		Tom = rw
		```	

- 启动和关闭svn
	1. 在终端中输入`svnserve -d -r /usr/local/svn/code`来启动svn，没有任何提示说明启动成功了。
	2. 首次配置完先关闭svn服务器再进行数据的上传和下载操作，可以通过Mac的活动监视器来关闭svn服务器。

## SVN命令
### 本地的代码导入服务器

```
svn import project_path server_path --username=Will --password=123456 -m "本地代码导入服务器"
```
project\_path是本地工程路径，server\_path是服务器上放置工程的路径。

### 服务器代码检出到本地

```
svn checkout server_path --username=Will --password=123456 local_path
```
server\_path是服务器上放置工程的路径，local\_path是要检出的本地路径。

### 更新服务器代码到本地

```
svn update
```
将本地仓库更新到中心版本库最新版。

```
svn update file_name
```
将指定文件更新到最新版。

```
svn update -r version_num file_name
```
将指定文件更新到指定版本。

### 提交代码
```
svn commit -m "message"
```

message是提交的说明信息，前面以-m修饰。

### 查看SVN状态
```
svn status
```
正常状态不显示。   
?：不在svn的控制中；M：内容被修改；C：发生冲突；A：预定加入到版本库；D: 被删除；K：被锁定。

```
svn status file_name
```
查看指定文件状态。

```
svn status -v file_name 
```
显示文件和其子目录状态。

### 添加文件
```
svn add file_name
```
添加指定文件到本地仓库，比如项目中新建的文件或者从外部移过来的文件还不在svn的控制内，此时需要使用这条命令来添加。

```
svn add xxx@2x.png@
```   
添加图片或带@符号的文件时，需要在文件名后面再加一个@，这是因为svn命令最后需要用@符号来指定一个版本导致的。

### 删除文件
```
svn delete file_nae
```
执行该命令后，文件在status中会被标记为D（删除），在commit后，文件确定被删除。有些被直接从本地移除的文件，也需要通过该命令标记为删除后再提交，从而与中心版本库保持同步。

### 比较差异
```
svn diff file_name
```
比较指定文件与未改动之前有哪些差异。

```
svn diff -r m:n file_name
```
对比指定文件版本m与版本n之间的差异。

### 恢复本地修改
```
svn revert
```
恢复未提交的修改。

```
svn revert file_name
```
恢复指定文件未提交的修改。

### 显示文件内容
```
svn cat file_name
```

### 查看日志
```
svn log
```

### 重命名目录/文件
```
svn move file_name new_file_name
```

### 创建版本控制下的新目录
```
svn mkdir local_folder_name
```

创建一个本地文件夹，该改动不会立刻提交到中心版本库，在下次提交更新时会以Add形式提交到中心版本库。

```
svn mkdir -m “message” server_folder_url
```
在中心版本指定位置创建一个文件夹。本地需要`update`显示。

### 解决冲突
```
svn resolved
```
该命令通常只是移除冲突状态，要真正解决冲突，往往还要到代码中去根据提示选择要删除和保留的代码，然后再次提交。

### 加锁/解锁
```
svn lock -m "message" file_name
```
锁住文件让其他用户不能提交改动。message是输入的锁信息。

```
svn unlock file_name
```
解锁指定文件。

### 复制文件及创建分支
```
svn copy file_name copied_file_name
```
file\_name是要复制的文件名，copied\_name是复制后的文件名。该改动和`svn mkdir`一样在本地版本库中对应文件夹执行，然后等待提交到中心版本库。

```
svn copy -m "message" target_path copied_target_path
```
target\_path是中心版本库里要复制的路径，copied\_target\_path是放置复制文件的路径。

```
svn copy -m "creat a branch" master_path branch_path
```
SVN里分支的创建本质是对主线的一种复制。

### 合并文件及合并分支
```
svn merge -r m:n file_path
```
将file_path在m和n版本号中的差异合并到指定文件中。

在分支上，想合并主线上的代码，先要进入分支的本地仓库，然后再合并。

```
cd local_branch_path
svn merge master_server_path
```

在主线上，想合并分支上的代码，要先进入主线的本质仓库，然后再合并。

```
cd local_master_path
svn merge --reintegrate branch_server_path
```
--reintegrate参数表示重新整理代码。

### 删除分支
```
svn delete -m "delete message" branch_server_path
```

删除分支即是直接删除中心版本库上的资料。

### Switch
SVN的中心版本库里可以有多个项目仓库，每个项目仓库也可以有多个`Branch`和`Tag`，这些仓库、分支、标签对应的地址都不同， 使用switch即是将本地项目仓库或者仓库里的文件的对应服务器地址切换为要更改的地址。所有的更改也将提交到更改地址对应的服务器仓库上。

```
svn switch another_server_path
```
`switch`执行后会自动更新一次。

## Git命令
### Git配置
```bash
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"
```
`git config`命令的`--global`参数，表明这台机器上的所有Git仓库都会使用这个配置，也可以对某个仓库指定不同的用户名和邮箱地址。

### 创建版本库
初始化一个Git仓库

```bash
$ git init
```
添加文件到Git仓库，包括两步：

```bash
$ git add <file>   
$ git commit -m "description"
```
`git add`可以反复多次使用，添加多个文件，`git commit`可以一次提交很多文件，`-m`后面输入的是本次提交的说明，可以输入任意内容。

### 查看工作区状态
```bash
$ git status
```
### 查看修改内容
```bash
$ git diff
```
```bash
$ git diff --cached
```
```bash
$ git diff HEAD -- <file>
```
- `git diff` 可以查看工作区(work dict)和暂存区(stage)的区别
- `git diff --cached` 可以查看暂存区(stage)和分支(master)的区别
- `git diff HEAD -- <file>` 可以查看工作区和版本库里面最新版本的区别

### 查看提交日志
```bash
$ git log
```
简化日志输出信息
```bash
$ git log --pretty=oneline
```
### 查看命令历史
```bash
$ git reflog
```
### 版本回退
```bash
$ git reset --hard HEAD^
```
以上命令是返回上一个版本，在Git中，用`HEAD`表示当前版本，上一个版本就是`HEAD^`，上上一个版本是`HEAD^^`，往上100个版本写成`HEAD~100`。
### 回退指定版本号
```bash
$ git reset --hard commit_id
```
commit_id是版本号，是一个用SHA1计算出的序列

### 工作区、暂存区和版本库
工作区：在电脑里能看到的目录；
版本库：在工作区有一个隐藏目录`.git`，是Git的版本库。
Git的版本库中存了很多东西，其中最重要的就是称为stage（或者称为index）的暂存区，还有Git自动创建的`master`，以及指向`master`的指针`HEAD`。

![理解](https://cdn.liaoxuefeng.com/cdn/files/attachments/001384907720458e56751df1c474485b697575073c40ae9000/0)

进一步解释一些命令：
- `git add`实际上是把文件添加到暂存区
- `git commit`实际上是把暂存区的所有内容提交到当前分支
### 撤销修改
##### 1. 丢弃工作区的修改
```bash
$ git checkout -- <file>
```
该命令是指将文件在工作区的修改全部撤销，这里有两种情况：   
1. 一种是file自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；   
2. 一种是file已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。

总之，就是让这个文件回到最近一次git commit或git add时的状态。

##### 2. 丢弃暂存区的修改
分两步：
第一步，把暂存区的修改撤销掉(unstage)，重新放回工作区：

```bash
$ git reset HEAD <file>
```   

第二步，撤销工作区的修改

```bash
$ git checkout -- <file>
```
小结：  

1. 当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令`git checkout -- <file>`。   
2. 当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用命令`git reset HEAD <file>`，就回到了第一步，第二步按第一步操作。
3. 已经提交了不合适的修改到版本库时，想要撤销本次提交，进行版本回退，前提是没有推送到远程库。

### 删除文件
```bash
$ git rm <file>
```
`git rm <file>`相当于执行
```bash
$ rm <file>
$ git add <file>
```
#### 进一步的解释
Q：比如执行了`rm text.txt` 误删了怎么恢复？   
A：执行`git checkout -- text.txt` 把版本库的东西重新写回工作区就行了

Q：如果执行了`git rm text.txt`我们会发现工作区的text.txt也删除了，怎么恢复？   
A：先撤销暂存区修改，重新放回工作区，然后再从版本库写回到工作区

```bash
$ git reset head text.txt
$ git checkout -- text.txt
```
Q：如果真的想从版本库里面删除文件怎么做？   
A：执行`git commit -m "delete text.txt"`，提交后最新的版本库将不包含这个文件

### 远程仓库
1. 创建SSH Key
 
	```bash
	$ ssh-keygen -t rsa -C "youremail@example.com"
	```
	
2. 关联远程仓库

	```bash
	$ git remote add origin https://github.com/username/	repositoryname.git
	```
3. 推送到远程仓库

	```bash
	$ git push -u origin master
	```
	`-u` 表示第一次推送master分支的所有内容，此后，每次本地提交后，只要	有必要，就可以使用命令`git push origin master`推送最新修改。

4. 从远程克隆

	```bash
	$ git clone https://github.com/usern/repositoryname.git
	```

### 分支
1. 创建分支

	```bash
	$ git branch <branchname>
	```
2. 查看分支

	```bash
	$ git branch
	```
	`git branch`命令会列出所有分支，当前分支前面会标一个*号。

3. 切换分支

	```bash
	$ git checkout <branchname>
	```
4. 创建+切换分支

	```bash
	$ git checkout -b <branchname>
	```

5. 合并某分支到当前分支

	```bash
	$ git merge <branchname>
	```

6. 删除分支

	```bash
	$ git branch -d <branchname>
	```
7. 查看分支合并图

	```bash
	$ git log --graph
	```
	当Git无法自动合并分支时，就必须首先解决冲突。解决冲突后，再提交，合并完成。用`git log --graph`命令可以看到分支合并图。

8. 普通模式合并分支

	```bash
	$ git merge --no-ff -m "description" <branchname>
	```
	因为本次合并要创建一个新的commit，所以加上`-m`参数，把commit描述写进去。合并分支时，加上`--no-ff`参数就可以用普通模式合并，能看出来曾经做过合并，包含作者和时间戳等信息，而fast forward合并就看不出来曾经做过合并。

#### 保存工作现场
```bash
$ git stash
```
#### 查看工作现场
```bash
$ git stash list
```
#### 恢复工作现场
```bash
$ git stash pop
```
#### 丢弃一个没有合并过的分支
```bash
$ git branch -D <branchname>
```

#### 查看远程库信息
```bash
$ git remote -v
```

#### 在本地创建和远程分支对应的分支
```bash
$ git checkout -b branch-name origin/branch-name，
```
本地和远程分支的名称最好一致；

#### 建立本地分支和远程分支的关联
```bash
$ git branch --set-upstream branch-name origin/branch-name；
```
#### 从本地推送分支
```bash
$ git push origin branch-name
```
如果推送失败，先用git pull抓取远程的新提交；
#### 从远程抓取分支
```bash
$ git pull
```
如果有冲突，要先处理冲突。

### 标签
tag就是一个让人容易记住的有意义的名字，它跟某个commit绑在一起。
#### 新建一个标签
```bash
$ git tag <tagname>
```
命令`git tag <tagname>`用于新建一个标签，默认为HEAD，也可以指定一个commit id。
#### 指定标签信息
```bash
$ git tag -a <tagname> -m <description> <branchname> or commit_id
```
`git tag -a <tagname> -m "blablabla..."`可以指定标签信息。
#### PGP签名标签
```bash
$ git tag -s <tagname> -m <description> <branchname> or commit_id
```
`git tag -s <tagname> -m "blablabla..."`可以用PGP签名标签。
#### 查看所有标签
```bash
$ git tag
```
#### 推送一个本地标签
```bash
$ git push origin <tagname>
```
#### 推送全部未推送过的本地标签
```bash
$ git push origin --tags
```
#### 删除一个本地标签
```bash
$ git tag -d <tagname>
```
#### 删除一个远程标签
```bash
$ git push origin :refs/tags/<tagname>
```
#### 配置忽略文件
在Git工作区的根目录下创建一个```.gitignore```文件, 然后把要忽略的文件名填进去即可。配置文件可以参考[github/gitignore](https://github.com/github/gitignore)。如:

```
# Windows:
Thumbs.db
ehthumbs.db
Desktop.ini

# Python:
*.py[cod]
*.so
*.egg
*.egg-info
dist
build

# My configurations:
db.ini
deploy_key_rsa
```




	
	





























