## 前言 关于SARD 
  软件保障参考数据集(SARD)包含118种CWE漏洞类型，是专门为评估静态分析工具的能力而开发的。数据集中的每一个样本都以其函数名称为标签，后缀字符串为 “good “或 “bad”，因此我们可以很容易地利用这些后缀信息来生成每个样本的标签。
为了从SARD中构建一个java漏洞数据集，需要对SARD进行一定的预处理。
这里SARD使用Juiletjava。
## 一、删除文件中的antbuild、web.xml等无用文件和文件夹：

```java
/**
	 * SARD数据集预处理：去除web.xml、antbuild等无用文件和文件夹
	 * @param filePath
	 */
	public static void getFile(String filePath) {
		File file = new File(filePath);
        if(file.isDirectory()){
            File[] files = file.listFiles();
            for (File file1 : files) {
                if(file1.isDirectory()){
                	if("antbuild".equals(file1.getName())){
                		deleteFileOrDirectory(file1.getAbsolutePath());
                	}
                    getFile(file1.getAbsolutePath());
                }else{
                	if("ServletMain.java".equals(file1.getName())){
                		deleteFileOrDirectory(file1.getAbsolutePath());
                	}
                }
            }
        }else{
        	if("ServletMain.java".equals(file.getName())){
        		deleteFileOrDirectory(file.getAbsolutePath());
        	}
        }
	}
	/**
     * 删除文件或文件夹
     *
     * @param fileName 文件名
     * @return 删除成功返回true,失败返回false
     */
    public static boolean deleteFileOrDirectory(String fileName) {
        File file = new File(fileName);  // fileName是路径或者file.getPath()获取的文件路径
        if (file.exists()) {
            if (file.isFile()) {
                return deleteFile(fileName);  // 是文件，调用删除文件的方法
            } else {
                return deleteDirectory(fileName);  // 是文件夹，调用删除文件夹的方法
            }
        } else {
            System.out.println("文件或文件夹删除失败：" + fileName);
            return false;
        }
    }

    /**
     * 删除文件
     *
     * @param fileName 文件名
     * @return 删除成功返回true,失败返回false
     */
    public static boolean deleteFile(String fileName) {
        File file = new File(fileName);
        if (file.isFile() && file.exists()) {
            file.delete();
            System.out.println("删除文件成功：" + fileName);
            return true;
        } else {
            System.out.println("删除文件失败：" + fileName);
            return false;
        }
    }

    /**
     * 删除文件夹
     * 删除文件夹需要把包含的文件及文件夹先删除，才能成功
     *
     * @param directory 文件夹名
     * @return 删除成功返回true,失败返回false
     */
    public static boolean deleteDirectory(String directory) {
        // directory不以文件分隔符（/或\）结尾时，自动添加文件分隔符，不同系统下File.separator方法会自动添加相应的分隔符
        if (!directory.endsWith(File.separator)) {
            directory = directory + File.separator;
        }
        File directoryFile = new File(directory);
        // 判断directory对应的文件是否存在，或者是否是一个文件夹
        if (!directoryFile.exists() || !directoryFile.isDirectory()) {
            System.out.println("文件夹删除失败，文件夹不存在" + directory);
            return false;
        }
        boolean flag = true;
        // 删除文件夹下的所有文件和文件夹
        File[] files = directoryFile.listFiles();
        for (int i = 0; i < files.length; i++) {  // 循环删除所有的子文件及子文件夹
            // 删除子文件
            if (files[i].isFile()) {
                flag = deleteFile(files[i].getAbsolutePath());
                if (!flag) {
                    break;
                }
            } else {  // 删除子文件夹
                flag = deleteDirectory(files[i].getAbsolutePath());
                if (!flag) {
                    break;
                }
            }
        }

        if (!flag) {
            System.out.println("删除失败");
            return false;
        }
        // 最后删除当前文件夹
        if (directoryFile.delete()) {
            System.out.println("删除成功：" + directory);
            return true;
        } else {
            System.out.println("删除失败：" + directory);
            return false;
        }
    }
```
## 二、二分类问题数据集的构建
111
## 三、多分类问题数据集的构建

```java
package Main;

import com.thoughtworks.qdox.JavaProjectBuilder;
import com.thoughtworks.qdox.model.JavaClass;
import com.thoughtworks.qdox.model.JavaMethod;

import java.io.BufferedReader;
import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.FileOutputStream;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;
import java.util.ArrayList;
import java.util.Collection;
import java.util.HashMap;
import java.util.Iterator;
import java.util.List;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

import org.dom4j.Document;
import org.dom4j.DocumentHelper;
import org.dom4j.Element;
import org.dom4j.io.OutputFormat;
import org.dom4j.io.XMLWriter;

public class JavaParser {
	/**
	 * SARD数据集预处理：去除web.xml、antibuild等无用文件和文件夹
	 * @param filePath
	 */
	public static void getFile(String filePath) {
		File file = new File(filePath);
        if(file.isDirectory()){
            File[] files = file.listFiles();
            for (File file1 : files) {
                if(file1.isDirectory()){
                	if("antbuild".equals(file1.getName())){
                		deleteFileOrDirectory(file1.getAbsolutePath());
                	}
                    getFile(file1.getAbsolutePath());
                }else{
                	if("ServletMain.java".equals(file1.getName())){
                		deleteFileOrDirectory(file1.getAbsolutePath());
                	}
                }
            }
        }else{
        	if("ServletMain.java".equals(file.getName())){
        		deleteFileOrDirectory(file.getAbsolutePath());
        	}
        }
	}
   
	public static void main(String[] args) throws IOException {
		String filePath = "C:\\Users\\rooooot\\Desktop\\SARD-Java\\generality test\\CWE614_Sensitive_Cookie_Without_Secure";
		String targetPath = "C:\\Users\\rooooot\\Desktop\\SARD-Java\\multi-class classification";
		HashMap<String, String> cweMap = new HashMap<String, String>();
    	cweMap.put("CWE643", "xpathi");
    	cweMap.put("CWE614", "securecookie");
    	cweMap.put("CWE327", "crypto");
    	cweMap.put("CWE328", "hash");
    	cweMap.put("CWE90", "ldapi");
    	cweMap.put("CWE89", "sqli");
    	cweMap.put("CWE78", "cmdi");
		GenerateClassFile(filePath,targetPath,cweMap);
  }
    /**
     * 根据CWE编号生成类级Java文件以及xml标签
     * @param filePath
     * @throws IOException 
     */
    public static void GenerateClassFile(String Path,String targetPath,HashMap<String, String> cweMap) throws IOException {
    	File directory = new File(Path);
    	int i = 0;
    	for(File javaFile:directory.listFiles()) {
    		//无漏洞函数缓存表
        	ArrayList<String> goodList = new ArrayList<String>();
        	//有漏洞函数缓存表
        	ArrayList<String> badList = new ArrayList<String>();
        	JavaProjectBuilder builder =  new JavaProjectBuilder();
            //源码目录
            builder.addSourceTree(javaFile);
            //读取CWE编号
            String[] filenames = javaFile.getName().split("_");
            String cwe = filenames[0];
            //读取文件编号
            String fileno = filenames[filenames.length-1].split("\\.")[0];
            //不包含跨文件代码，直接生成
            if(!check(fileno)) {
            	//读取漏洞函数及无漏洞函数
                Collection<JavaClass> classes = builder.getClasses();
                Iterator<JavaClass> it = classes.iterator();
                //逐个类进行读取
                while(it.hasNext()) {
                	JavaClass cls = builder.getClassByName(it.next().getCanonicalName());
                    List<JavaMethod> methods = cls.getMethods();
                    for(JavaMethod method : methods){
                        String methodName = method.getName(); //方法名
                        if(ignoreCaseIndexOf(methodName,"bad")!=-1) {
                        	badList.add(method.getSourceCode());
                        }else if(ignoreCaseIndexOf(methodName,"good")!=-1){
                        	goodList.add(method.getSourceCode());
    					}
                    }
                }
                //生成无漏洞类文件及其xml说明文件
                //重建java类文件
                File f = new File(targetPath+"\\"+javaFile.getName());//新建一个文件对象，如果不存在则创建一个该文件
                FileWriter fw;
                StringBuffer codeFile = new StringBuffer(readFile(javaFile.getAbsolutePath()));
                try {
                	fw = new FileWriter(f);
                	for(String text:badList) {
                		int start = ignoreCaseIndexOf(codeFile.toString(), text);
                		if(start!=-1) {
                			codeFile.replace(start, start+text.length(),"");
                		}else {
    						System.out.println("文件解析异常:"+javaFile.getAbsolutePath());
    					}
                    }
                	fw.write(codeFile.toString());//将字符串写入到指定的路径下的文件中
                	fw.close();
                } catch (IOException e) { 
                	e.printStackTrace(); 
                	}
                String[] args = {javaFile.getName().split("\\\\.")[0],cweMap.get(cwe),Integer.toString(i),"false",cwe.replaceAll("CWE", "")};
                i++;
                createXML(targetPath, args);
                //生成有漏洞类文件及其xml说明文件
                //重建java类文件
                File fv = new File(targetPath+"\\"+"v_"+javaFile.getName());//新建一个文件对象，如果不存在则创建一个该文件
                FileWriter fwv;
                StringBuffer codeFilev = new StringBuffer(readFile(javaFile.getAbsolutePath()));
                try {
                	fwv = new FileWriter(fv);
                	for(String text:goodList) {
                		int start = ignoreCaseIndexOf(codeFilev.toString(), text);
                		if(start!=-1) {
                			codeFilev.replace(start, start+text.length(),"");
                		}else {
    						System.out.println("文件解析异常:"+javaFile.getAbsolutePath());
    					}
                    }
                	fwv.write(codeFilev.toString());//将字符串写入到指定的路径下的文件中
                	fwv.close();
                } catch (IOException e) { 
                	e.printStackTrace(); 
                	}
                String[] argsv = {"v_"+javaFile.getName().split("\\\\.")[0],cweMap.get(cwe),Integer.toString(i),"true",cwe.replaceAll("CWE", "")};
                i++;
                createXML(targetPath, argsv);
            }else {   //包含跨文件代码，暂时忽略
    			
    		}
    	}
    	System.out.println("总文件数:"+i);
    }
    /**
     * 根据CWE编号生成函数片段级Java文件及xml标签
     * @param filePath
     */
    public static void GenerateSliceFile(String filePath) {
    	
    }
    /**
     * 删除文件或文件夹
     * @param fileName 文件名
     * @return 删除成功返回true,失败返回false
     */
    public static boolean deleteFileOrDirectory(String fileName) {
        File file = new File(fileName);  // fileName是路径或者file.getPath()获取的文件路径
        if (file.exists()) {
            if (file.isFile()) {
                return deleteFile(fileName);  // 是文件，调用删除文件的方法
            } else {
                return deleteDirectory(fileName);  // 是文件夹，调用删除文件夹的方法
            }
        } else {
            System.out.println("文件或文件夹删除失败：" + fileName);
            return false;
        }
    }
    /**
     * 删除文件
     *
     * @param fileName 文件名
     * @return 删除成功返回true,失败返回false
     */
    public static boolean deleteFile(String fileName) {
        File file = new File(fileName);
        if (file.isFile() && file.exists()) {
            file.delete();
            System.out.println("删除文件成功：" + fileName);
            return true;
        } else {
            System.out.println("删除文件失败：" + fileName);
            return false;
        }
    }

    /**
     * 删除文件夹
     * 删除文件夹需要把包含的文件及文件夹先删除，才能成功
     *
     * @param directory 文件夹名
     * @return 删除成功返回true,失败返回false
     */
    public static boolean deleteDirectory(String directory) {
        // directory不以文件分隔符（/或\）结尾时，自动添加文件分隔符，不同系统下File.separator方法会自动添加相应的分隔符
        if (!directory.endsWith(File.separator)) {
            directory = directory + File.separator;
        }
        File directoryFile = new File(directory);
        // 判断directory对应的文件是否存在，或者是否是一个文件夹
        if (!directoryFile.exists() || !directoryFile.isDirectory()) {
            System.out.println("文件夹删除失败，文件夹不存在" + directory);
            return false;
        }
        boolean flag = true;
        // 删除文件夹下的所有文件和文件夹
        File[] files = directoryFile.listFiles();
        for (int i = 0; i < files.length; i++) {  // 循环删除所有的子文件及子文件夹
            // 删除子文件
            if (files[i].isFile()) {
                flag = deleteFile(files[i].getAbsolutePath());
                if (!flag) {
                    break;
                }
            } else {  // 删除子文件夹
                flag = deleteDirectory(files[i].getAbsolutePath());
                if (!flag) {
                    break;
                }
            }
        }

        if (!flag) {
            System.out.println("删除失败");
            return false;
        }
        // 最后删除当前文件夹
        if (directoryFile.delete()) {
            System.out.println("删除成功：" + directory);
            return true;
        } else {
            System.out.println("删除失败：" + directory);
            return false;
        }
    }
    /**
     * 根据传入的参数创建XML标签文件
     * @param path:目标文件路径
     * @param args {文件名,category的值,test-number的值,vulnerability的值,cwe编号}
     */
    public static void createXML(String path,String[] args) {
    	// 1.声明文件名称
        String fileName = args[0];
        // 2.创建dom对象
        Document document = DocumentHelper.createDocument();
        // 3.添加节点，根据需求添加，这里我只是设置了一个head节点，下面有name和age两个子节点
        Element test_metadata = document.addElement("test-metadata");
        Element wbe_version = test_metadata.addElement("wbe-version");
        wbe_version.setText("1.1");
        Element category = test_metadata.addElement("category");
        category.addText(args[1]);
        Element test_number = test_metadata.addElement("test-number");
        test_number.addText(args[2]);
        Element vulnerability = test_metadata.addElement("vulnerability");
        vulnerability.addText(args[3]);
        Element cwe = test_metadata.addElement("cwe");
        cwe.addText(args[4]);
        // 4、格式化模板
        //OutputFormat format = OutputFormat.createCompactFormat();
        OutputFormat format = OutputFormat.createPrettyPrint();
        format.setEncoding("UTF-8");
        // 5、生成xml文件
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        try {
            XMLWriter writer = new XMLWriter(out, format);
            writer.write(document);
            writer.close();
        } catch (IOException e) {
            System.out.println("生成xml文件失败。文件名【" + fileName + "】");
        }
        // 6、生成的XML利用文件输出流输出到文件
        try (FileOutputStream fos = new FileOutputStream(path+"\\"+fileName + ".xml")) {
            fos.write(out.toByteArray());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    /**
     * 判断字符串中是否包含字母
     * @param str
     * @return
     */
    public static boolean check(String str) {
    	String regex=".*[a-zA-Z]+.*";
    	Matcher m=Pattern.compile(regex).matcher(str);
    	return m.matches();
    }
    /** 
     * 返回指定子字符串在此字符串中第一次出现处的索引，从指定的索引开始，不区分大小。 
     *  
     * @param subject 被查找字符串。 
     * @param search 要查找的子字符串。 
     * @return 指定子字符串在此字符串中第一次出现处的索引，从指定的索引开始。 
     */  
    public static int ignoreCaseIndexOf(String subject, String search) {  
        return ignoreCaseIndexOf(subject, search,-1);  
    }  
      
    /** 
     * 返回指定子字符串在此字符串中第一次出现处的索引，从指定的索引开始，不区分大小。 
     *  
     * @param subject 被查找字符串。 
     * @param search 要查找的子字符串。 
     * @param fromIndex 开始查找的索引位置。其值没有限制，如果它为负，则与它为 0 的效果同样：将查找整个字符串。 
     *          如果它大于此字符串的长度，则与它等于此字符串长度的效果相同：返回 -1。 
     * @return 指定子字符串在此字符串中第一次出现处的索引，从指定的索引开始。 
     */  
    public static int ignoreCaseIndexOf(String subject, String search, int fromIndex) {  
        //当被查找字符串或查找子字符串为空时，抛出空指针异常。  
        if (subject == null || search == null) {  
            throw new NullPointerException("输入的参数为空");  
        }  
        fromIndex = fromIndex < 0 ? 0 : fromIndex;  
        if (search.equals("")) {  
            return fromIndex >= subject.length() ? subject.length() : fromIndex;  
        }  
        int index1 = fromIndex;  
        int index2 = 0;   
        char c1;  
        char c2;  
        loop1: while (true) {  
            if (index1 < subject.length()) {  
                c1 = subject.charAt(index1);  
                c2 = search.charAt(index2);  
            } else {  
                break loop1;  
            }  
            while (true) {  
                if (isEqual(c1, c2)) {  
                    if (index1 < subject.length() - 1  
                            && index2 < search.length() - 1) {  
                        c1 = subject.charAt(++index1);  
                        c2 = search.charAt(++index2);  
                    } else if (index2 == search.length() - 1) {  
                        return fromIndex;  
                    } else {  
                        break loop1;  
                    }  
                } else {  
                    index2 = 0;  
                    break;  
                }  
            }  
            //重新查找子字符串的位置  
            index1 = ++fromIndex;  
        }  
        return -1;  
    }  
     
    /** 
     * 判断两个字符是否相等。 
     * @param c1 字符1 
     * @param c2 字符2 
     * @return 若是英文字母，不区分大小写，相等true，不等返回false； 
     *          若不是则区分，相等返回true，不等返回false。 
     */  
    private static boolean isEqual(char c1,char c2){  
            //  字母小写                   字母大写  
        if(((97<=c1 && c1<=122) || (65<=c1 && c1<=90))  
                && ((97<=c2 && c2<=122) || (65<=c2 && c2<=90))  
                && ((c1-c2==32) || (c2-c1==32))){  
            return true;  
        }  
        else if(c1==c2){  
            return true;  
        }  
        return false;  
    }  
    /**
     * 将java文件读取成字符串返回
     * @param file
     * @return
     * @throws IOException
     */
    public static String readFile(String file) throws IOException {
    	BufferedReader reader = new BufferedReader(new FileReader (file));
    	String line = null;
    	StringBuilder stringBuilder = new StringBuilder();
    	String ls = System.getProperty("line.separator");
    	try {
    	while((line = reader.readLine()) != null) {
    	stringBuilder.append(line);
    	stringBuilder.append(ls);
    	}
    	return stringBuilder.toString();
    	} finally {
    	reader.close();
    	}
    	}
}

```

2024.06.14更新：
我把我处理好的数据集放在[这里](https://github.com/STA-inc/Blog/blob/main/MachineLearning/SARD.7z)，需要的自行下载。

