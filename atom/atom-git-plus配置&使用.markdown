# atom 使用 git-plus 进行版本管理
## 配置git-plus
在 atom 菜单栏中： *File -> setting -> packages* 选择 **git-plus** ,选择 *settings* 进入 git-plus 设置界面。
- 在*Git Path* 项中输入你安装的 github 中的 **git.exe** 路径,例如：

![git path](http://7xrn7f.com1.z0.glb.clouddn.com/16-6-17/54263992.jpg)

__注：如果你的环境中除了github 还有其他git命令（如：git.exe,但不是github的git 命令）请一定使用github带的git.exe 路径__

## **atom git-plus**在 *github*上版本管理流程
### 1、在github上创建一个版本库
- 可以在[github.com](https://github.com) 上面创建
- 可以使用 *github* 桌面客户端建立
### 2、将新建好的版本库克隆到本地（*这里一定通过ssh协议来进行克隆*）
- 获取版本库.git连接：
![](http://7xrn7f.com1.z0.glb.clouddn.com/16-6-17/85493767.jpg)

- 在本地通过 github 所带的 git shell 在你本地库路径下执行克隆命令即可：`git clone git@github.com:SimonJoker/XXX.git`
- 使用*atom* 打开拉下来的版本库文件夹
- 在 atom中编辑文件，保存在当前的 folder下面即可，
- windows下快捷键`ctl + shift +p` 直接输入 git名令进行 `git add -> git commit -> git push` 即可
