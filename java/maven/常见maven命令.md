
~~~shell
# 查看maven依赖树并将输出追加文件末尾
mvn dependency:tree > test.txt

# 查看maven依赖树并将输出覆盖到指定文件
mvn dependency:tree >> test.txt

# 分析项目中的依赖关系，包括未使用的直接和间接依赖
mvn dependency:analyze
# - 项目中直接依赖但是未使用的依赖项（ Unused declared dependencies found）
# - 项目中直接或间接依赖但在编译或测试阶段确实用到的依赖项（ Used undeclared dependencies found）

# 跳过测试的运行和编译直接进行打包
mvn package -Dmaven.test.skip=true 
~~~