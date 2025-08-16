```java
package Main;

import com.thoughtworks.qdox.JavaProjectBuilder;
import com.thoughtworks.qdox.library.JavaClassContext;
import com.thoughtworks.qdox.model.JavaClass;
import com.thoughtworks.qdox.model.JavaMethod;

import java.io.File;
import java.util.Collection;
import java.util.Iterator;
import java.util.List;

public class JavaParser {
    public static void main(String[] args) {
        JavaProjectBuilder builder =  new  JavaProjectBuilder();
        //源码目录
        builder.addSourceTree(new File("C:\\Users\\rooooot\\Desktop\\s01\\CWE89_SQL_Injection__connect_tcp_execute_01.java"));
        Collection<JavaClass> classes = builder.getClasses();
        Iterator<JavaClass> it = classes.iterator();
        //逐个类进行读取
        while(it.hasNext()) {
        	JavaClass cls = builder.getClassByName(it.next().getCanonicalName());
            List<JavaMethod> methods = cls.getMethods();
            for(JavaMethod method : methods){
                System.out.println(method.getCodeBlock()); //方法体的源码
            }
        }
    }
}

```
