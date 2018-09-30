---
title: jvm-specification
abbrlink: 3241459811
date: 2018-09-04 10:01:11
tags:
categories:
---
https://docs.oracle.com/javase/specs/jvms/se10/html/


# 1.简介
1.1。一点历史
1.2。 Java虚拟机
1.3。规范的组织
1.4。符号
1.5。反馈
# 2. Java虚拟机的结构
2.1。类文件格式
2.2。数据类型
2.3。原始类型和价值观
2.3.1。积分类型和值
2.3.2。浮点类型，值集和值
2.3.3。 returnAddress类型和值
2.3.4。布尔类型
2.4。参考类型和值
2.5。运行时数据区
2.5.1。电脑注册
2.5.2。 Java虚拟机堆栈
2.5.3。堆
2.5.4。方法区
2.5.5。运行时常量池
2.5.6。本机方法堆栈
2.6。框架
2.6.1。局部变量
2.6.2。操作数堆栈
2.6.3。动态链接
2.6.4。正常方法调用完成
2.6.5。突然的方法调用完成
2.7。对象的表示
2.8。浮点运算
2.8.1。 Java虚拟机浮点运算和IEEE 754
2.8.2。浮点模式
2.8.3。价值集转换
2.9。特殊方法
2.9.1。实例初始化方法
2.9.2。类初始化方法
2.9.3。签名多态方法
2.10。例外
2.11。指令集汇总
2.11.1。类型和Java虚拟机
2.11.2。加载和存储指令
2.11.3。算术指令
2.11.4。类型转换说明
2.11.5。对象创建和操作
2.11.6。操作数堆栈管理说明
2.11.7。控制转移指令
2.11.8。方法调用和返回指令
2.11.9。抛出异常
2.11.10。同步
2.12。类库
2.13。公共设计，私人实施
# 3.编译Java虚拟机
3.1。示例格式
3.2。使用常量，局部变量和控制结构
3.3。算术
3.4。访问运行时常量池
3.5。更多控制示例
3.6。接收参数
3.7。调用方法
3.8。使用类实例
3.9。数组
3.10。编译开关
3.11。操作数堆栈上的操作
3.12。投掷和处理例外
3.13。最后编译
3.14。同步
3.15。注释
3.16。模块
# 4.类文件格式
## 4.1。 ClassFile结构
## 4.2。名称
4.2.1。二进制类和接口名称
4.2.2。不合格的名称
4.2.3。模块和包名称
## 4.3。叙
4.3.1。语法符号
4.3.2。场描述符
4.3.3。方法描述符
## 4.4。恒定池
4.4.1。 CONSTANT_Class_info结构
4.4.2。 CONSTANT_Fieldref_info，CONSTANT_Methodref_info和CONSTANT_InterfaceMethodref_info结构
4.4.3。 CONSTANT_String_info结构
4.4.4。 CONSTANT_Integer_info和CONSTANT_Float_info结构
4.4.5。 CONSTANT_Long_info和CONSTANT_Double_info结构
4.4.6。 CONSTANT_NameAndType_info结构
4.4.7。 CONSTANT_Utf8_info结构
4.4.8。 CONSTANT_MethodHandle_info结构
4.4.9。 CONSTANT_MethodType_info结构
4.4.10。 CONSTANT_InvokeDynamic_info结构
4.4.11。 CONSTANT_Module_info结构
4.4.12。 CONSTANT_Package_info结构
## 4.5。字段
## 4.6。方法
## 4.7。属性
4.7.1。定义和命名新属性
4.7.2。 ConstantValue属性
4.7.3。代码属性
4.7.4。 StackMapTable属性
4.7.5。例外属性
4.7.6。 InnerClasses属性
4.7.7。 EnclosingMethod属性
4.7.8。合成属性
4.7.9。签名属性
4.7.9.1。签名
4.7.10。 SourceFile属性
4.7.11。 SourceDebugExtension属性
4.7.12。 LineNumberTable属性
4.7.13。 LocalVariableTable属性
4.7.14。 LocalVariableTypeTable属性
4.7.15。弃用属性
4.7.16。 RuntimeVisibleAnnotations属性
4.7.16.1。 element_value结构
4.7.17。 RuntimeInvisibleAnnotations属性
4.7.18。 RuntimeVisibleParameterAnnotations属性
4.7.19。 RuntimeInvisibleParameterAnnotations属性
4.7.20。 RuntimeVisibleTypeAnnotations属性
4.7.20.1。 target_info联合
4.7.20.2。 type_path结构
4.7.21。 RuntimeInvisibleTypeAnnotations属性
4.7.22。 AnnotationDefault属性
4.7.23。 BootstrapMethods属性
4.7.24。 MethodParameters属性
4.7.25。模块属性
4.7.26。 ModulePackages属性
4.7.27。 ModuleMainClass属性
## 4.8。格式检查
## 4.9。 Java虚拟机代码的约束
4.9.1。静态约束
4.9.2。结构约束
## 4.10。验证类文件
4.10.1。按类型检查验证
4.10.1.1。 Java虚拟机工件的访问器
4.10.1.2。验证类型系统
4.10.1.3。指令表示
4.10.1.4。堆栈映射帧和类型转换
4.10.1.5。类型检查抽象和本机方法
4.10.1.6。用代码键入检查方法
4.10.1.7。键入检查加载和存储指令
4.10.1.8。键入检查受保护的成员
4.10.1.9。键入检查说明

aaload
aastore
aconst_null
aload, aload_<n>
anewarray
areturn
arraylength
astore, astore_<n>
athrow
baload
bastore
bipush
caload
castore
checkcast
d2f, d2i, d2l
dadd
daload
dastore
dcmp<op>
dconst_<d>
ddiv
dload, dload_<n>
dmul
dneg
drem
dreturn
dstore, dstore_<n>
dsub
dup
dup_x1
dup_x2
dup2
dup2_x1
dup2_x2
f2d, f2i, f2l
fadd
faload
fastore
fcmp<op>
fconst_<f>
fdiv
fload, fload_<n>
fmul
fneg
frem
freturn
fstore, fstore_<n>
fsub
getfield
getstatic
goto, goto_w
i2b, i2c, i2d, i2f, i2l, i2s
iadd
iaload
iand
iastore
iconst_<i>
idiv
if_acmp<cond>
if_icmp<cond>
if<cond>
ifnonnull, ifnull
iinc
iload, iload_<n>
imul
ineg
instanceof
invokedynamic
invokeinterface
invokespecial
invokestatic
invokevirtual
ior, irem
ireturn
ishl, ishr, iushr
istore, istore_<n>
isub, ixor
l2d, l2f, l2i
ladd
laload
land
lastore
lcmp
lconst_<l>
ldc, ldc_w, ldc2_w
ldiv
lload, lload_<n>
lmul
lneg
lookupswitch
lor, lrem
lreturn
lshl, lshr, lushr
lstore, lstore_<n>
lsub, lxor
monitorenter, monitorexit
multianewarray
new
newarray
nop
pop, pop2
putfield
putstatic
return
saload
sastore
sipush
swap
tableswitch
wide

4.10.2。按类型推断进行验证
4.10.2.1。类推理的验证过程
4.10.2.2。字节码验证器
4.10.2.3。类型的值long和double
4.10.2.4。实例初始化方法和新创建的对象
4.10.2.5。例外，最后
4.11。 Java虚拟机的局限性
# 5.加载，链接和初始化
## 5.1。运行时常量池
## 5.2。 Java虚拟机启动
## 5.3。创作和加载
5.3.1。使用Bootstrap类加载器加载
5.3.2。使用用户定义的类加载器加载
5.3.3。创建数组类
5.3.4。加载约束
5.3.5。从类文件表示中派生类
5.3.6。模块和图层
## 5.4。链接
5.4.1。验证
5.4.2。制备
5.4.3。解析度
5.4.3.1。类和接口解析
5.4.3.2。现场分辨率
5.4.3.3。方法解决方案
5.4.3.4。接口方法解析
5.4.3.5。方法类型和方法句柄解析
5.4.3.6。呼叫站点说明符解析
5.4.4。访问控制
5.4.5。重写
## 5.5。初始化
5.6。绑定本机方法实现
5.7。 Java虚拟机退出
# 6. Java虚拟机指令集
6.1。假设：“必须”的含义
6.2。保留的操作码
6.3。虚拟机错误
6.4。指令描述的格式
助记符
6.5。说明
aaload
aastore
aconst_null
aload
aload_<n>
anewarray
areturn
arraylength
astore
astore_<n>
athrow
baload
bastore
bipush
caload
castore
checkcast
d2f
d2i
d2l
dadd
daload
dastore
dcmp<op>
dconst_<d>
ddiv
dload
dload_<n>
dmul
dneg
drem
dreturn
dstore
dstore_<n>
dsub
dup
dup_x1
dup_x2
dup2
dup2_x1
dup2_x2
f2d
f2i
f2l
fadd
faload
fastore
fcmp<op>
fconst_<f>
fdiv
fload
fload_<n>
fmul
fneg
frem
freturn
fstore
fstore_<n>
fsub
getfield
getstatic
goto
goto_w
i2b
i2c
i2d
i2f
i2l
i2s
iadd
iaload
iand
iastore
iconst_<i>
idiv
if_acmp<cond>
if_icmp<cond>
if<cond>
ifnonnull
ifnull
iinc
iload
iload_<n>
imul
ineg
instanceof
invokedynamic
invokeinterface
invokespecial
invokestatic
invokevirtual
ior
irem
ireturn
ishl
ishr
istore
istore_<n>
isub
iushr
ixor
jsr
jsr_w
l2d
l2f
l2i
ladd
laload
land
lastore
lcmp
lconst_<l>
ldc
ldc_w
ldc2_w
ldiv
lload
lload_<n>
lmul
lneg
lookupswitch
lor
lrem
lreturn
lshl
lshr
lstore
lstore_<n>
lsub
lushr
lxor
monitorenter
monitorexit
multianewarray
new
newarray
nop
pop
pop2
putfield
putstatic
ret
return
saload
sastore
sipush
swap
tableswitch
wide
