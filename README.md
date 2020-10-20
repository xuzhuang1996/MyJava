# MyJava
学习路上的一些知识点

  
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
1. 比较同一个文件，在两个分支的区别：VCS->Git->Compare with Branch
1. 比较两个文件不同处：复制一个文件，全选。在另一个文件中右键，选择compare with clipBroad
1. 方法注释：
   1. 方法注释头：
   
           **
           * @Description
           *   
          $param$
           * @return 
           **/
    2. edit变量：
    
            groovyScript("def result=''; def params=\"${_1}\".replaceAll('[\\\\[|\\\\]|\\\\s]', '').split(',').toList(); for(i = 0; i < params.size(); i++) {result+=' * @param ' + params[i] + ((i < params.size() - 1) ? '\\n' : '')}; return result", methodParameters())
1. 直接对比，开分隔窗。window->Editor tab-> split vertical
------------------
## markdown：

1. 图片添加：
    - `<img src="https..." width = 30% height = 30% />`
    - `<div align=center>![这里写图片描述](http:...)`
    - `<img src="https://github.com/xuzhuang1996/MyJava/blob/master/img/面试/1-B+树插入过程.gif" width=50% height=50% />`

-------------
## java
1. 正则表达式过滤出字母、数字和中文
   - 过滤出字母: `[^(A-Za-z)]`
   - 过滤出数字: `[^(0-9)]`
   - 过滤出中文: `[^(\\u4e00-\\u9fa5)]`,不过直接复制这个到idea会自动补/，因此记住这里只有2个/
   - 过滤字母数字：`s.replaceAll("[^A-Za-z0-9]", ""); `
   
2. [通配符和边界](https://www.cnblogs.com/drizzlewithwind/p/6100164.html)来源。
   - `<? extends T>`,上界通配符。能放一切继承自T的类.频繁往外读取内容的,适合用<? extends T >。
   - `<? super T>`,下界通配符。能放T及T继承的类，但不能放T的派生类。经常往里插入的,适合用 <? super T> 。
   

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
12. git merge branchname合并到当前分支
------------
## mysql
1. Ubuntu下sudo能无密码访问，但是普通用户无法登录。这样就可以密码登录
   - update mysql.user set authentication_string=PASSWORD('12345678'), plugin='mysql_native_password' where user='root';
   - flush privileges;
   
   
   
--------------------
## pom
1. [jar打包依赖](https://www.ibm.com/developerworks/cn/java/j-5things13/index.html)。springboot

            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <!-- 告知 maven-jar-plugin 添加一个 Class-Path 元素到 MANIFEST.MF 文件 -->
                    <!--<addClasspath>true</addClasspath>-->
                    <!-- 如果您计划在同一目录下包含有您的所有依赖项，作为您将构建的 JAR，那么您可以忽略它，否则所有的依赖项应该位于 “lib” 文件夹-->
                    <!--<classpathPrefix>lib/</classpathPrefix>-->
                    <!-- 指定该Main Class为全局的唯一入口 -->
                    <mainClass>com.example.demo.DemoApplication</mainClass>
                    <layout>ZIP</layout>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal><!--可以把依赖的包都打包到生成的Jar包中-->
                        </goals>
                    </execution>
                </executions>
            </plugin>
## maven
1. 在idea中，maven编译通过，但是idea启动debug失败，即build也不成功。
   - edit debug->bufore launch->删除build->添加maven goal->输入compile。
## restful
1. REST相关的概念：资源，集合，URL
   1. 资源：资源是某种东西的对象或表示。例如, 动物，学校和员工是资源; 删除，添加，更新是对这些资源执行的相关操作
   2. 集合：资源集合，例如，公司是资源的集合
   3. URL: 可以通过其定位资源的路径，并且可以对其执行某些操作
2. URL设计：
   1. 动词
      - POST：新建（Create）
      - GET：读取（Read）
      - PUT：更新（Update）
      - PATCH：更新（Update），通常不分更新，也很少用到
      - DELETE：删除（Delete）
   2. 名词设计，表示一个资源或者服务，用名词复数的形式描述某一资源
3. RESTful 幂等性（多次调用是否会对资源产生影响）原则
http://www.yymp3.com/Play/12813/163839.htm

1. tmp
   - https://potoyang.gitbook.io/spring-in-action-v5/di-11-zhang-kai-fa-xiang-ying-shi-api/11.1-shi-yong-spring-webflux
   - https://developer.ibm.com/zh/depmodels/reactive-systems/tutorials/j-whats-new-in-spring-framework-5-theedom/
   - https://developer.ibm.com/zh/depmodels/reactive-systems/articles/spring5-webflux-reactive/

