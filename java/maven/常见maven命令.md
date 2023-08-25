
~~~shell
# 查看maven依赖树并将输出追加文件末尾
mvn dependency:tree > test.txt


# 查看maven依赖树并将输出覆盖到指定文件
mvn dependency:tree >> test.txt

# 跳过测试的运行和编译直接进行打包
mvn package -Dmaven.test.skip=true 
~~~