# MyJava
你永远不会知道，之前学习的东西，会在未来以什么样的方式运用到。与君共勉。

## idea:

1. 键入psvm, 也就是public static void main的首字母。然后enter。    
2. 键入fori，enter即可。   
3. 如何像写一个System.out.println();就是sout   
4. view——>Tool windows——>structure或者alt+7。查看当前类的方法。
5. 选中方法，alt+F7。跳转到调用该方法的位置。
6. ctrl+shift+N，查文件名
6. ctrl+shift+A，查设置，比如设置背景background
1. ctrl+shift+T，创建测试类
1. Ctrl + Alt + 左箭头，跳转方法后返回。当快捷键失效时，打开按钮：view、appearence、toolbar打勾
1. ctrl + alt +B，查看接口实现
1. shift + F6，修改变量名。统一修改的那种。
1. shift + F9 Debug.........shit + F10 Run
1. edit -> find -> find in path，搜索更好用
1. Alt+Insert，插入代码，包括test、构造函数等很多代码。很常用。
1. Ctrl+tab，切换类视图，不用鼠标。
1. 重构：
   1. alt + delete 重构：安全删除，看有没有依赖
   1. ctrl + alt + M 抽方法。
   2. Ctrl + Alt + T将属性抽出参数。
   1. Ctrl+shift+Alt+T，抽类，选择delegate
   5. ctrl+alt+V，抽变量表达式，定义变量。Predicate这种变量很适合。
1. 比较同一个文件，在两个分支的区别：VCS->Git->Compare with Branch
1. 比较两个文件不同处：复制一个文件，全选。在另一个文件中右键，选择compare with clipBroad
1. 直接对比，开分隔窗。window->Editor tab-> split vertical
1. intellij IDEA Properties中文unicode自动转码处理:File | Settings | Editor | File Encoding 下，勾选Transparent native-to-ascii conversion,UTF-8.适用：IDEA Properties文件常用于存储国际化内容，为避免编码差异，文件的中文通常要进行unicode编码。
1. idea项目的.idea/workspace.xml中设置一行代码：`<property name="dynamic.classpath" value="true" />`。命令此选项控制如何通过命令行或文件将类路径传递给JVM。大多数操作系统都有最大命令行限制，如果超过该限制，IDEA将无法运行您的应用程序。当命令行的长度超过32768个字符时：
   1. IDEA建议您切换到动态类路径。长类路径被写入文件，然后由应用程序启动器读取并通过系统类加载器加载。
   2. 另一个建议是：配置中的shorten command line从user local 切换为classpath file。这个千万别切。在使用powerMock时导致类加载器用成了powermock的，而不是appclassLoader
----------
## git
1. `git remote update origin --prune`   # 更新远程主机origin 整理分支
2. `git branch -r`查看远程分支
3. `git push -f origin master `git push 强制提交
4. `git fetch -all`从远程仓库下载最新版本。`git reset --hard origin/master`将本地设为刚获取的最新的内容
5. `.gitignore`在已经push后，想重新添加忽略文件：

        git rm -r --cached .
        git add .
        git commit -m 'update .gitignore'
6. 只合并customize_new分支的部分文件到当前分支：`git checkout  customize_new /home/mi/miui-bi/miui-bi-web/src/main/java/com/MiuiAppPaiEntity.java`。
7. git提交后出现nano界面:Ctrl + X然后输入y再然后回车，就可以退出了
8. `git stash`将目前还不想提交的但是已经修改的内容进行保存至堆栈中，后续可以在某个分支上恢复出堆栈中的内容。这也就是说，stash中的内容不仅仅可以恢复到原先开发的分支，也可以恢复到其他任意指定的分支上。[来源](https://blog.csdn.net/stone_yw/article/details/80795669)
   - `git stash save ""`，作用等同于git stash，区别是可以加一些注释
   - `git stash list`,查看当前stash中的内容
   - `git stash pop`,将当前stash中的内容弹出，并应用到当前分支对应的工作目录上。
   - `git stash apply`,将堆栈中的内容应用到当前目录，不同于git stash pop，该命令不会将内容从堆栈中删除，也就说该命令能够将堆栈的内容多次应用到工作目录中，适应于多个分支的情况。
9. `git clone -b dev_jk http://10.1.1.11/service/tmall-service.git`指定分支下载
10. `git cherry-pick`命令的作用，就是将指定的提交（commit）应用于当前分支。
11. `git status`三个区（工作区、暂存区、存储区）的区别，`git diff`默认暂存区与工作区的区别。
12. `git merge branch name`合并到当前分支
13. `git pull <远程库名> <远程分支名>:<本地分支名>`从远程库中获取某个分支的更新，再与本地指定的分支进行自动merge。`git pull origin develop`如果是要与本地当前分支merge，则冒号后面的<本地分支名>可以不写.
14. rebase的作用简要概括为：可以对某一段线性提交历史进行编辑、删除、复制、粘贴；`git rebase -i HEAD~3 `,`git rebase -i 36224db`.
15. 将某个commit复用到另一个分支：`git checkout master`后`git cherry-pick 62ecb3`，将其用到master。commit可以跨分支
## maven
1. 在idea中，maven编译通过，但是idea启动debug失败，即build也不成功。
   - edit debug->bufore launch->删除build->添加maven goal->输入compile。
2. `mvn install --settings c:\user\settings.xml `,指定maven文件进行打包。

