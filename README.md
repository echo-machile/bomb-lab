# bomb-lab
# 1. ![image](https://user-images.githubusercontent.com/76896357/114028768-6bc78700-98ab-11eb-87e6-dbd6321039a6.png)

* 首先用gdb调试bomb,输入disas phse_1反汇编第一段
**查看汇编代码**

```
400ee0:	48 83 ec 08          	  sub    $0x8,%rsp                            # 创建大小为8字节的栈
  400ee4:	be 00 24 40 00       	mov    $0x402400,%esi                 # 在%esi寄存器中存放0x402400的值，传参数
  400ee9:	e8 4a 04 00 00       	callq  401338 <strings_not_equal>     # 调用函数，可以将函数反汇编
  400eee:	85 c0                	test   %eax,%eax                      # 测试返回值
  400ef0:	74 05                	je     400ef7 <phase_1+0x17>          # 为0，就跳到400ce7，从这里推测上一个函数估计是比较
  400ef2:	e8 43 05 00 00       	callq  40143a <explode_bomb>          # 查看函数，如果跳转语句没有执行就会爆炸
  400ef7:	48 83 c4 08          	add    $0x8,%rsp                      # %rsp加%0x8即让指针上移，减少栈空间
  400efb:	c3                   	retq   
```
![image](https://user-images.githubusercontent.com/76896357/114177586-2b314180-996f-11eb-9450-4c2dde37cef3.png)

![image](https://user-images.githubusercontent.com/76896357/114031662-29537980-98ae-11eb-81bd-066043d81673.png)

__函数的作用是判断两个字符串是否相等__

既然是比较大小，而我们又传进去一个参数，说明函数里面一定有另一个参数

__根据test指令，以及第三章所学内容，第一个参数被放在%rsi中，查看一下%rsi的值，他就应该是答案__

![image](https://user-images.githubusercontent.com/76896357/114178706-a9daae80-9970-11eb-9ca0-1f17767a9a1e.png)

之后运行程序，复制字符串
![image](https://user-images.githubusercontent.com/76896357/114181273-e52aac80-9973-11eb-9909-1b4fa9a5889a.png)

成功哦！

# 2. 查看反汇编代码
![image](https://user-images.githubusercontent.com/76896357/114186193-22456d80-9979-11eb-85ee-883fa6ef872b.png)
```
Dump of assembler code for function phase_2:
   0x0000000000400efc <+0>:	push   %rbp                                               # 将程序的入口压入栈中
   0x0000000000400efd <+1>:	push   %rbx                                               # 将被调用者保存在寄存器的值保存到栈中，以便调用完之后进行恢复
   0x0000000000400efe <+2>:	sub    $0x28,%rsp                                         # 为栈分配空间
   0x0000000000400f02 <+6>:	mov    %rsp,%rsi                                          # 把栈指针指向的值赋值给第一个参数
   0x0000000000400f05 <+9>:	callq  0x40145c <read_six_numbers>                        # 调用函数，结果是六个数字
   0x0000000000400f0a <+14>:	cmpl   $0x1,(%rsp)                                      # 将0x1和%rsp所存地址对应的值进行比较
   0x0000000000400f0e <+18>:	je     0x400f30 <phase_2+52>                            # 相等的话，就跳到<phase_2+52>的位置
   0x0000000000400f10 <+20>:	callq  0x40143a <explode_bomb>                          # 不相等就会爆炸
   0x0000000000400f15 <+25>:	jmp    0x400f30 <phase_2+52>                            # 爆炸后，跳到<phase_2+52>的位置
   0x0000000000400f17 <+27>:	mov    -0x4(%rbx),%eax                                  # 将值-0x4(%rdx)保存到%eax的寄存器中
   0x0000000000400f1a <+30>:	add    %eax,%eax                                        # 将%eax的值进行double
   0x0000000000400f1c <+32>:	cmp    %eax,(%rbx)                                      # 将%eax的值与%rbx所存地址对应的值进行比较
   0x0000000000400f1e <+34>:	je     0x400f25 <phase_2+41>                            # 如果相等，就跳到<phase_2+41>
   0x0000000000400f20 <+36>:	callq  0x40143a <explode_bomb>                          # 不相等的话，就发生爆炸
   0x0000000000400f25 <+41>:	add    $0x4,%rbx                                        # 爆炸完之后，将%rbx的值加0x4
   0x0000000000400f29 <+45>:	cmp    %rbp,%rbx                                        # 比较%rbp和%rbx的值
   0x0000000000400f2c <+48>:	jne    0x400f17 <phase_2+27>                            # 如果不相等的话就跳到<phaes_2+27>
   0x0000000000400f2e <+50>:	jmp    0x400f3c <phase_2+64>                            # 相等的话直接跳到<phase_2+64>
   0x0000000000400f30 <+52>:	lea    0x4(%rsp),%rbx                                   # 将(%rsp对应地址+4)的内存地址放在%rbx
   0x0000000000400f35 <+57>:	lea    0x18(%rsp),%rbp                                  # 将(栈指针+0x18)的地址放进%rbp
   0x0000000000400f3a <+62>:	jmp    0x400f17 <phase_2+27>                            # 放完之后，跳到<phase_2+2的位置>
   0x0000000000400f3c <+64>:	add    $0x28,%rsp                                       # 解除栈使用的空间
--Type <RET> for more, q to quit, c to continue without paging--c
   0x0000000000400f40 <+68>:	pop    %rbx
   0x0000000000400f41 <+69>:	pop    %rbp
   0x0000000000400f42 <+70>:	retq   
End of assembler dump.
```

* 第42行调用函数，并且比较0x1和%rsp对应内存地址的值，在这里可知，第一个值应该是1

* 第58行得到一个循环，循环的条件就是%rbx和%rbp不相等，循环体中，把%rbx对应的内存地址减少4(之前的%rax的值)赋给%eax

* %eax接收到值后，double，再与%rbx对应内存的值比较相等就跳过去，不相等爆炸

* 跳完后，再给%rbx+4再比较%rbx

![image](https://user-images.githubusercontent.com/76896357/114255754-d41b8300-99e7-11eb-97ed-393fab797339.png)

# 3.查看反汇编代码
```
Dump of assembler code for function phase_3:
   0x0000000000400f43 <+0>:	sub    $0x18,%rsp                                                        # 创建栈空间
   0x0000000000400f47 <+4>:	lea    0xc(%rsp),%rcx                                                    # 添加栈底
   0x0000000000400f4c <+9>:	lea    0x8(%rsp),%rdx                                                    # 添加第三个栈指针
   0x0000000000400f51 <+14>:	mov    $0x4025cf,%esi                                                  # 将0x4025cf放进%esi中
   0x0000000000400f56 <+19>:	mov    $0x0,%eax                                                       # 将0x0放进%eax中
   0x0000000000400f5b <+24>:	callq  0x400bf0 <__isoc99_sscanf@plt>                                  # 调用函数
   0x0000000000400f60 <+29>:	cmp    $0x1,%eax                                                       # 将0x1与返回值相比较
   0x0000000000400f63 <+32>:	jg     0x400f6a <phase_3+39>                                           # 如果大于就跳到39的位置
   0x0000000000400f65 <+34>:	callq  0x40143a <explode_bomb>                                         # 否则爆炸
   0x0000000000400f6a <+39>:	cmpl   $0x7,0x8(%rsp)                                                  # 比较0x7与(%rdx)
   0x0000000000400f6f <+44>:	ja     0x400fad <phase_3+106>                                          # 如果是负数，即(%rdx)大于0x7跳到106的位置
   0x0000000000400f71 <+46>:	mov    0x8(%rsp),%eax                                                  # 将栈指针对应内存地址+8位置的值放置到%eax
   0x0000000000400f75 <+50>:	jmpq   *0x402470(,%rax,8)                                              # 间接跳转至8*%rax+0x400fbe的值
   0x0000000000400f7c <+57>:	mov    $0xcf,%eax                                                      # 把0xcf的值放进%rax的寄存器
   0x0000000000400f81 <+62>:	jmp    0x400fbe <phase_3+123>                                          # 直接跳到phase_123的位置
   0x0000000000400f83 <+64>:	mov    $0x2c3,%eax                                                     # 将0x2c3的值放在%rax
   0x0000000000400f88 <+69>:	jmp    0x400fbe <phase_3+123>                                          # 再跳
   0x0000000000400f8a <+71>:	mov    $0x100,%eax                                                     # 把0x100的值放在%rax里面
   0x0000000000400f8f <+76>:	jmp    0x400fbe <phase_3+123>                                          # 再跳
   0x0000000000400f91 <+78>:	mov    $0x185,%eax                                                     # 把0x185放进%eax
   0x0000000000400f96 <+83>:	jmp    0x400fbe <phase_3+123>                                          # 再跳
   0x0000000000400f98 <+85>:	mov    $0xce,%eax                                                      # 把0xce放进%eax
--Type <RET> for more, q to quit, c to continue without paging--c       
   0x0000000000400f9d <+90>:	jmp    0x400fbe <phase_3+123>                                          # 再跳
   0x0000000000400f9f <+92>:	mov    $0x2aa,%eax                                                     # 将0x2aa放进%rax中
   0x0000000000400fa4 <+97>:	jmp    0x400fbe <phase_3+123>                                          # 再跳
   0x0000000000400fa6 <+99>:	mov    $0x147,%eax                                                     # 把0x147的值移到%eax
   0x0000000000400fab <+104>:	jmp    0x400fbe <phase_3+123>                                          # 再跳
   0x0000000000400fad <+106>:	callq  0x40143a <explode_bomb>                                         # 不然就爆炸
   0x0000000000400fb2 <+111>:	mov    $0x0,%eax                                                       # 将0保存在%eax
   0x0000000000400fb7 <+116>:	jmp    0x400fbe <phase_3+123>                                          # 再跳
   0x0000000000400fb9 <+118>:	mov    $0x137,%eax                                                     # 将0x137放进%rax
   0x0000000000400fbe <+123>:	cmp    0xc(%rsp),%eax                                                  # 比较%rsp对应内存地址+0xc里的值和%eax的值
   0x0000000000400fc2 <+127>:	je     0x400fc9 <phase_3+134>                                          # 相等的话，就跳到134
   0x0000000000400fc4 <+129>:	callq  0x40143a <explode_bomb>                                         # 不然爆炸
   0x0000000000400fc9 <+134>:	add    $0x18,%rsp                                                      # 释放栈空间
   0x0000000000400fcd <+138>:	retq                                                                   # 返回值
```

* 根据<+14>处%rsi是用来放参数的以及<+24>函数是要输入，就想着查一下输入地址的那个值？

![image](https://user-images.githubusercontent.com/76896357/114268791-947b8800-9a35-11eb-89d5-c4423e6d2d49.png)

可知，我们输入的形式应该是两个整数

由第三行和第四行可知，我们输入的参数被放在%rcx和%rdx

由44行那里可以知道，参数不能(%rdx)所对应的值不能大于0x7即*(%rso+0x8)<=0x7

第46行把第一个参数的值赋给了%rax

先令第一个参数为1

第50行查看间接跳转的位置

![image](https://user-images.githubusercontent.com/76896357/114269550-69dffe00-9a3a-11eb-9093-014bc8d56e9c.png)

在这里可以知道%eax是0x137，也就是说第二个参数应该是0x137

![image](https://user-images.githubusercontent.com/76896357/114270090-18853e00-9a3d-11eb-9346-24c34e91aaf8.png)

另外答案是有7种情况的

![image](https://user-images.githubusercontent.com/76896357/114270185-98aba380-9a3d-11eb-900c-48db14fa856a.png)

破了！！！

# 4.查看反汇编代码
```
Dump of assembler code for function phase_4:
   0x000000000040100c <+0>:	sub    $0x18,%rsp                                                               # 创建栈空间
   0x0000000000401010 <+4>:	lea    0xc(%rsp),%rcx                                                           # 把内存(%rsp+c)对应的地址放在%rcx的寄存器中
   0x0000000000401015 <+9>:	lea    0x8(%rsp),%rdx                                                           # 把内存(%rsp+8)对应的地址放在%rcx的寄存器中
   0x000000000040101a <+14>:	mov    $0x4025cf,%esi                                                         # 把0x4025cf放进%esi中
   0x000000000040101f <+19>:	mov    $0x0,%eax                                                              # 把0x0放进%eax的寄存器器中
   0x0000000000401024 <+24>:	callq  0x400bf0 <__isoc99_sscanf@plt>                                         # 调用输入函数
   0x0000000000401029 <+29>:	cmp    $0x2,%eax                                                              # 比较0x2和%eax的值
   0x000000000040102c <+32>:	jne    0x401035 <phase_4+41>                                                  # 如果不相等就跳到41处，即爆炸
   0x000000000040102e <+34>:	cmpl   $0xe,0x8(%rsp)                                                         # 比较0xe和(%rsp+8)的值
   0x0000000000401033 <+39>:	jbe    0x40103a <phase_4+46>                                                  # 如果是小于就跳转
   0x0000000000401035 <+41>:	callq  0x40143a <explode_bomb>                                                # 否则爆炸
   0x000000000040103a <+46>:	mov    $0xe,%edx                                                              # 把0xe的值放在%edx里
   0x000000000040103f <+51>:	mov    $0x0,%esi                                                              # 把0x0放在%esi里面
   0x0000000000401044 <+56>:	mov    0x8(%rsp),%edi                                                         # 把(%rsp+8)的值放进%edi
   0x0000000000401048 <+60>:	callq  0x400fce <func4>                                                       # 调用fun4
   0x000000000040104d <+65>:	test   %eax,%eax                                                              # 测试eax对应的值
   0x000000000040104f <+67>:	jne    0x401058 <phase_4+76>                                                  # 不等于0就跳到76的地方
   0x0000000000401051 <+69>:	cmpl   $0x0,0xc(%rsp)                                                         # 比较0和(c+%rsp)的值
   0x0000000000401056 <+74>:	je     0x40105d <phase_4+81>                                                  # 相等的情况下，跳到81的位置
   0x0000000000401058 <+76>:	callq  0x40143a <explode_bomb>                                                # 否则爆炸
   0x000000000040105d <+81>:	add    $0x18,%rsp                                                             # 释放栈空间
   0x0000000000401061 <+85>:	retq   
End of assembler dump.
```

第一步类似于第三个实验，查看一下地址0x4025cf的值

![image](https://user-images.githubusercontent.com/76896357/114275143-f814ae00-9a53-11eb-8477-c1eab0958b32.png)

* 还是输入两个值

* 在第46和60给函数func_4传了3个参数

* 查看一下函数的反汇编代码

![image](https://user-images.githubusercontent.com/76896357/114275335-c223f980-9a54-11eb-892d-75432a7384ae.png)

```
Dump of assembler code for function func4:
   0x0000000000400fce <+0>:	sub    $0x8,%rsp                                            # 为函数拓展栈空间
   0x0000000000400fd2 <+4>:	mov    %edx,%eax                                            # 把%edx的值赋给%eax
   0x0000000000400fd4 <+6>:	sub    %esi,%eax                                            # 将%eax的值减去%esi
   0x0000000000400fd6 <+8>:	mov    %eax,%ecx                                            # 把%eax的值赋给%ecx
   0x0000000000400fd8 <+10>:	shr    $0x1f,%ecx                                         # 将%ecx右移0x1f位,就是要看一下前两个参数的大小，值返回1或者0因此可以用三幕运算法表示
   0x0000000000400fdb <+13>:	add    %ecx,%eax                                          # 将%eax加上%ecx
   0x0000000000400fdd <+15>:	sar    %eax                                               # 对%eax进行算术右移
   0x0000000000400fdf <+17>:	lea    (%rax,%rsi,1),%ecx                                 # 将(%rax+%rsi)的值对应的内存地址放进%ecx
   0x0000000000400fe2 <+20>:	cmp    %edi,%ecx                                          # 比较%edi和%ecx
   0x0000000000400fe4 <+22>:	jle    0x400ff2 <func4+36>                                # 小于时跳转到36的位置
   0x0000000000400fe6 <+24>:	lea    -0x1(%rcx),%edx                                    # 将(%rcx-0x1)对应值的内存地址交给%edx
   0x0000000000400fe9 <+27>:	callq  0x400fce <func4>                                   # 回调
   0x0000000000400fee <+32>:	add    %eax,%eax                                          # 将%eax进行double
   0x0000000000400ff0 <+34>:	jmp    0x401007 <func4+57>                                # 跳到57
   0x0000000000400ff2 <+36>:	mov    $0x0,%eax                                          # 将0赋给%eax
   0x0000000000400ff7 <+41>:	cmp    %edi,%ecx                                          # 比较edi和ecx的值，如果c=t返回t
   0x0000000000400ff9 <+43>:	jge    0x401007 <func4+57>                                # edi>ecx时跳到57
   0x0000000000400ffb <+45>:	lea    0x1(%rcx),%esi                                     # 将(%rcx+0x1)值对应的内存地址赋给%esi
   0x0000000000400ffe <+48>:	callq  0x400fce <func4>                                   # 调用函数4
   0x0000000000401003 <+53>:	lea    0x1(%rax,%rax,1),%eax                              # 将(2*%rax+0x1)对应的内存地址给%eax
   0x0000000000401007 <+57>:	add    $0x8,%rsp                                          # 退出栈空间
   0x000000000040100b <+61>:	retq   
End of assembler dump.
```
翻译成c语言代码：

```
int func4(a,b,c)
{
int t.h;
  t=b+((c-b)>0?0:1+c-b)/2
  if(t<=a)
  {
  if(t==a)
  {return 0;}
    else 
    return *(2*func4(a,t+1,c)+1);
  }else
  {
    c=t-1
    return 2*func4(a,b,c);
  }
```

由34和39说明我们输入的第二个参数小于15

由汇编代码46 51，56可知，送入的参数分别是0，14，未知数


并且由69之我们输入的第一个数是0 

最后运行程序
```
#include<iostream>
using namespace std;
int func4(int a,int b,int c)
{
int t,h;
    t=b+(c-b)/2;
  if(t<=a)
  {
  if(t==a)
  {return 0;}
    else
    return 2*func4(a,t+1,c)+1;
  }else
  {
      c=t-1;
    return 2*func4(a,b,c);
  }
}
 int main()
 {
    for(int i=0;i<=14;i++)
    {
        if(func4(i,0,14)==0)    cout<<i<<" ";
    }
    return 0;
 }
```
得到最终结果

![image](https://user-images.githubusercontent.com/76896357/114282193-15a63f80-9a75-11eb-8d5c-c4b82c75939b.png)

* 注意：编程比大小的时候不能写反，血一样的教训
* 答案也有多种

![image](https://user-images.githubusercontent.com/76896357/114282245-530acd00-9a75-11eb-9baf-b75facecf74e.png)

破解成功！！！
