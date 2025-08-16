# 0x00 前言
最近在研究抽象语法树的编码，需要使用Python的Javalang库解析Java源代码为抽象语法树，记录一波该库的一些用法。
这里javalang的版本选择最新的0.13.0，老版本的0.11.0显示信息不全，不便于学习。
# 0x01 抽象语法树节点类型
通过查阅javalang库的源代码，分析其构成可以发现，其将java代码解析成抽象语法树（以下简称AST），AST中包含节点Node，Node有以下类型，我对其类名作了简单的注释：

```python
from .ast import Node

# ------------------------------------------------------------------------------
# 编译单元
class CompilationUnit(Node):
    attrs = ("package", "imports", "types")
# 包导入类型
class Import(Node):
    attrs = ("path", "static", "wildcard")
# 
class Documented(Node):
    attrs = ("documentation",)

class Declaration(Node):
    attrs = ("modifiers", "annotations")
# 类型描述类
class TypeDeclaration(Declaration, Documented):
    attrs = ("name", "body")

    @property
    def fields(self):
        return [decl for decl in self.body if isinstance(decl, FieldDeclaration)]

    @property
    def methods(self):
        return [decl for decl in self.body if isinstance(decl, MethodDeclaration)]

    @property
    def constructors(self):
        return [decl for decl in self.body if isinstance(decl, ConstructorDeclaration)]
# 包描述类
class PackageDeclaration(Declaration, Documented):
    attrs = ("name",)
# 类描述类
class ClassDeclaration(TypeDeclaration):
    attrs = ("type_parameters", "extends", "implements")
# 枚举类型描述类
class EnumDeclaration(TypeDeclaration):
    attrs = ("implements",)

    @property
    def fields(self):
        return [decl for decl in self.body.declarations if isinstance(decl, FieldDeclaration)]

    @property
    def methods(self):
        return [decl for decl in self.body.declarations if isinstance(decl, MethodDeclaration)]
# 接口描述类
class InterfaceDeclaration(TypeDeclaration):
    attrs = ("type_parameters", "extends",)
# 注解描述类
class AnnotationDeclaration(TypeDeclaration):
    attrs = ()

# ------------------------------------------------------------------------------
# 类型节点的基类
class Type(Node):
    attrs = ("name", "dimensions",)
# 基础类型，比如int、long等
class BasicType(Type):
    attrs = ()
# 引用类型，该类描述各种Java类的实例化，并且包括String等自带的类
class ReferenceType(Type):
    attrs = ("arguments", "sub_type")

class TypeArgument(Node):
    attrs = ("type", "pattern_type")

# ------------------------------------------------------------------------------

class TypeParameter(Node):
    attrs = ("name", "extends")

# ------------------------------------------------------------------------------

class Annotation(Node):
    attrs = ("name", "element")

class ElementValuePair(Node):
    attrs = ("name", "value")

class ElementArrayValue(Node):
    attrs = ("values",)

# ------------------------------------------------------------------------------
#类体成员的基类，该基类的子类主要描述Java代码类体中的成员，分为方法、变量、构造器三种
class Member(Documented):
    attrs = ()
# 方法描述类
class MethodDeclaration(Member, Declaration):
    attrs = ("type_parameters", "return_type", "name", "parameters", "throws", "body")
# 变量描述类
class FieldDeclaration(Member, Declaration):
    attrs = ("type", "declarators")
# 构造器描述类
class ConstructorDeclaration(Declaration, Documented):
    attrs = ("type_parameters", "name", "parameters", "throws", "body")

# ------------------------------------------------------------------------------

class ConstantDeclaration(FieldDeclaration):
    attrs = ()

class ArrayInitializer(Node):
    attrs = ("initializers",)

class VariableDeclaration(Declaration):
    attrs = ("type", "declarators")

class LocalVariableDeclaration(VariableDeclaration):
    attrs = ()

class VariableDeclarator(Node):
    attrs = ("name", "dimensions", "initializer")

class FormalParameter(Declaration):
    attrs = ("type", "name", "varargs")

class InferredFormalParameter(Node):
    attrs = ('name',)

# ------------------------------------------------------------------------------

class Statement(Node):
    attrs = ("label",)

class IfStatement(Statement):
    attrs = ("condition", "then_statement", "else_statement")

class WhileStatement(Statement):
    attrs = ("condition", "body")

class DoStatement(Statement):
    attrs = ("condition", "body")

class ForStatement(Statement):
    attrs = ("control", "body")

class AssertStatement(Statement):
    attrs = ("condition", "value")

class BreakStatement(Statement):
    attrs = ("goto",)

class ContinueStatement(Statement):
    attrs = ("goto",)

class ReturnStatement(Statement):
    attrs = ("expression",)

class ThrowStatement(Statement):
    attrs = ("expression",)

class SynchronizedStatement(Statement):
    attrs = ("lock", "block")

class TryStatement(Statement):
    attrs = ("resources", "block", "catches", "finally_block")

class SwitchStatement(Statement):
    attrs = ("expression", "cases")

class BlockStatement(Statement):
    attrs = ("statements",)

class StatementExpression(Statement):
    attrs = ("expression",)

# ------------------------------------------------------------------------------

class TryResource(Declaration):
    attrs = ("type", "name", "value")

class CatchClause(Statement):
    attrs = ("parameter", "block")

class CatchClauseParameter(Declaration):
    attrs = ("types", "name")

# ------------------------------------------------------------------------------

class SwitchStatementCase(Node):
    attrs = ("case", "statements")

class ForControl(Node):
    attrs = ("init", "condition", "update")

class EnhancedForControl(Node):
    attrs = ("var", "iterable")

# ------------------------------------------------------------------------------

class Expression(Node):
    attrs = ()

class Assignment(Expression):
    attrs = ("expressionl", "value", "type")

class TernaryExpression(Expression):
    attrs = ("condition", "if_true", "if_false")

class BinaryOperation(Expression):
    attrs = ("operator", "operandl", "operandr")

class Cast(Expression):
    attrs = ("type", "expression")

class MethodReference(Expression):
    attrs = ("expression", "method", "type_arguments")

class LambdaExpression(Expression):
    attrs = ('parameters', 'body')

# ------------------------------------------------------------------------------

class Primary(Expression):
    attrs = ("prefix_operators", "postfix_operators", "qualifier", "selectors")

class Literal(Primary):
    attrs = ("value",)

class This(Primary):
    attrs = ()

class MemberReference(Primary):
    attrs = ("member",)

class Invocation(Primary):
    attrs = ("type_arguments", "arguments")

class ExplicitConstructorInvocation(Invocation):
    attrs = ()

class SuperConstructorInvocation(Invocation):
    attrs = ()

class MethodInvocation(Invocation):
    attrs = ("member",)

class SuperMethodInvocation(Invocation):
    attrs = ("member",)

class SuperMemberReference(Primary):
    attrs = ("member",)

class ArraySelector(Expression):
    attrs = ("index",)

class ClassReference(Primary):
    attrs = ("type",)

class VoidClassReference(ClassReference):
    attrs = ()

# ------------------------------------------------------------------------------

class Creator(Primary):
    attrs = ("type",)

class ArrayCreator(Creator):
    attrs = ("dimensions", "initializer")

class ClassCreator(Creator):
    attrs = ("constructor_type_arguments", "arguments", "body")

class InnerClassCreator(Creator):
    attrs = ("constructor_type_arguments", "arguments", "body")

# ------------------------------------------------------------------------------

class EnumBody(Node):
    attrs = ("constants", "declarations")

class EnumConstantDeclaration(Declaration, Documented):
    attrs = ("name", "arguments", "body")

class AnnotationMethod(Declaration):
    attrs = ("name", "return_type", "dimensions", "default")


```

# 0x02 编译单元CompilationUnit
以最简单的代码为例，学习javalang的结构：

```java
package fuck;
import com.sta.aaa;
import com.sta.bbb;
public class Main {
	private String s;
}
```
代码仅包含一个类，且类中仅包含一个变量声明。使用Python的javalang库来解析它：

```python
import javalang
fd = open("C:/Users/root/Desktop/java.java", "r", encoding="utf-8")  #读取Java源代码
tree = javalang.parse.parse(fd.read())               # 根据源代码解析出一颗抽象语法树
print(tree)            
```
效果：

```python
# 打印整颗语法树：CompilationUnit(imports=[Import(path=com.sta.aaa, static=False, wildcard=False), Import(path=com.sta.bbb, static=False, wildcard=False)], package=PackageDeclaration(annotations=None, documentation=None, modifiers=None, name=fuck), types=[ClassDeclaration(annotations=[], body=[FieldDeclaration(annotations=[], declarators=[VariableDeclarator(dimensions=[], initializer=None, name=s)], documentation=None, modifiers={'private'}, type=ReferenceType(arguments=None, dimensions=[], name=String, sub_type=None))], documentation=None, extends=None, implements=None, modifiers={'public'}, name=Main, type_parameters=None)])
```
这里返回了一个javalang.tree.CompilationUnit类型，表示整颗抽象语法树（以后简称AST）
在print(tree)后面添加以下代码，打印该编译单元的各个子节点：
```python
for i in range(0,len(tree.children)):
    print(tree.children[i])
```
效果：

```python
children[0]:PackageDeclaration(annotations=None, documentation=None, modifiers=None, name=fuck)
chileren[1]:[Import(path=com.sta.aaa, static=False, wildcard=False), Import(path=com.sta.bbb, static=False, wildcard=False)]
children[2]:[ClassDeclaration(annotations=[], body=[FieldDeclaration(annotations=[], declarators=[VariableDeclarator(dimensions=[], initializer=None, name=s)], documentation=None, modifiers={'private'}, type=ReferenceType(arguments=None, dimensions=[], name=String, sub_type=None))], documentation=None, extends=None, implements=None, modifiers={'public'}, name=Main, type_parameters=None)]
```
可以看到：CompilationUnit（编译单元）的children是一个数组，由三个元素构成：[包声明,Import声明数组,类声明]
值得注意的是，如果源代码中不存在包声明、Import声明、类声明，相应的children[0]、children[1]、children[2]只会被置为“[]”，依然要占位置，也就是说CompilationUnit。children的结构是固定的。
包声明和Import声明数组没什么好说的，接下来要研究ClassDeclaration。

# 0x02 类声明ClassDeclaration
为了简化问题，再次把Java源代码精简一下：
```java
package fuck;
import com.sta.aaa;
import com.sta.bbb;
public class Main {
}
```
只包含一个类，且该类是空的。
得到的类声明如下：
```python
ClassDeclaration(annotations=[], body=[], documentation=None, extends=None, implements=None, modifiers={'public'}, name=Main, type_parameters=None)
```
这里annotations表示注释，body表示类括号里面的内容，我们主要看body里面的内容。
这里空类不行了，要在里面加点内容：
```java
package fuck;
import com.sta.aaa;
import com.sta.bbb;
public class Main {
	private String s;
	public static void main(String[] args) {
		System.out.println();
	}
}
```
然后打印body里面的内容：`print(tree.children[2][0].body)`
```python
[FieldDeclaration(annotations=[], declarators=[VariableDeclarator(dimensions=[], initializer=None, name=s)], documentation=None, modifiers={'private'}, type=ReferenceType(arguments=None, dimensions=[], name=String, sub_type=None)), MethodDeclaration(annotations=[], body=[StatementExpression(expression=MethodInvocation(arguments=[], member=println, postfix_operators=[], prefix_operators=[], qualifier=System.out, selectors=[], type_arguments=None), label=None)], documentation=None, modifiers={'static', 'public'}, name=main, parameters=[FormalParameter(annotations=[], modifiers=set(), name=args, type=ReferenceType(arguments=None, dimensions=[None], name=String, sub_type=None), varargs=False)], return_type=None, throws=None, type_parameters=None)]
```
这里打印body的type可以知道body也是一个数组，包含不同的声明，可以有FieldDeclaration、MethodDeclaration、LocalVariableDeclaration等，这里声明了一个变量，一个函数，所以body的长度就是2，如果只有一个变量被声明，那么body的长度就是1，此外，注释不算在body里面。

变量声明FieldDeclaration放到后面再说，按照编译单元-----类声明------函数声明的顺序，从大到小来解析。下面到函数声MethodDeclaration

# 0x03 函数声明MethodDeclaration
Java里面函数也叫方法，是面向对象的称呼。
这里源码的函数声明也很简单：
打印函数声明看看：
```python
MethodDeclaration(annotations=[], body=[StatementExpression(expression=MethodInvocation(arguments=[], member=println, postfix_operators=[], prefix_operators=[], qualifier=System.out, selectors=[], type_arguments=None), label=None)], documentation=None, modifiers={'static', 'public'}, name=main, parameters=[FormalParameter(annotations=[], modifiers=set(), name=args, type=ReferenceType(arguments=None, dimensions=[None], name=String, sub_type=None), varargs=False)], return_type=None, throws=None, type_parameters=None)
```
函数声明的结构和类声明类似，也是看body，body也是一个数组，根据代码顺序确定里面的元素。
body里面的元素就是表达式语句StatementExpression。

# 0x05 表达式语句StatementExpression
# 0x06 深度优先遍历一颗抽象语法树（内置方法）
以一段简单的Java代码为例：

```java
package javalang.brewtab.com; 
class Test {}
class Test2 {}
```
javalang库提供了直接遍历的方法，能够直接遍历路径和节点：

```python
import javalang
tree = javalang.parse.parse("package javalang.brewtab.com; class Test {}; class Test2 {}")
for path, node in tree:
    print(path)
```
显示的结果：

```bash
()
(CompilationUnit(imports=[], package=PackageDeclaration(annotations=None, documentation=None, modifiers=None, name=javalang.brewtab.com), types=[ClassDeclaration(annotations=[], body=[], documentation=None, extends=None, implements=None, modifiers=set(), name=Test, type_parameters=None), ClassDeclaration(annotations=[], body=[], documentation=None, extends=None, implements=None, modifiers=set(), name=Test2, type_parameters=None)]),)
(CompilationUnit(imports=[], package=PackageDeclaration(annotations=None, documentation=None, modifiers=None, name=javalang.brewtab.com), types=[ClassDeclaration(annotations=[], body=[], documentation=None, extends=None, implements=None, modifiers=set(), name=Test, type_parameters=None), ClassDeclaration(annotations=[], body=[], documentation=None, extends=None, implements=None, modifiers=set(), name=Test2, type_parameters=None)]), [ClassDeclaration(annotations=[], body=[], documentation=None, extends=None, implements=None, modifiers=set(), name=Test, type_parameters=None), ClassDeclaration(annotations=[], body=[], documentation=None, extends=None, implements=None, modifiers=set(), name=Test2, type_parameters=None)])
(CompilationUnit(imports=[], package=PackageDeclaration(annotations=None, documentation=None, modifiers=None, name=javalang.brewtab.com), types=[ClassDeclaration(annotations=[], body=[], documentation=None, extends=None, implements=None, modifiers=set(), name=Test, type_parameters=None), ClassDeclaration(annotations=[], body=[], documentation=None, extends=None, implements=None, modifiers=set(), name=Test2, type_parameters=None)]), [ClassDeclaration(annotations=[], body=[], documentation=None, extends=None, implements=None, modifiers=set(), name=Test, type_parameters=None), ClassDeclaration(annotations=[], body=[], documentation=None, extends=None, implements=None, modifiers=set(), name=Test2, type_parameters=None)])

```
# 0x07 层次遍历一颗抽象语法树
其实就是层次遍历多叉树的问题，使用队列来完成，这里直接给出代码：

```python
def get_blocks_improved_BFS(root):
    queue = []
    queue.append(root)
    val = []
    while queue:
        for i in range(len(queue)):
            node = queue.pop(0)
            children = node.children
            if children != None:
                logic = ['SwitchStatement', 'IfStatement', 'ForStatement', 'WhileStatement', 'DoStatement']
                if name in ['MethodDeclaration', 'ConstructorDeclaration']:
                    body = node.body
                    for child in body:
                        val.append(child.val)  #这里就看需要打印的是节点的什么信息了
                        queue.append(child)
    return val
```
----------未完待续
