# 第二周lab实验周记

第二个实验是拆炸弹实验，总共六个实验。
只有不执行每个实验中的explode_bomb函数才可以阻止炸弹爆炸。
实验给出了bomb.c文件，但是没有头文件所以只能通过反汇编来判断每个实验函数的内容

## phase_1

```
   0x0000000000400ee0 <+0>:	sub    rsp,0x8
   0x0000000000400ee4 <+4>:	mov    esi,0x402400
   0x0000000000400ee9 <+9>:	call   0x401338 <strings_not_equal>
   0x0000000000400eee <+14>:	test   eax,eax
   0x0000000000400ef0 <+16>:	je     0x400ef7 <phase_1+23>
   0x0000000000400ef2 <+18>:	call   0x40143a <explode_bomb>
   0x0000000000400ef7 <+23>:	add    rsp,0x8
   0x0000000000400efb <+27>:	ret    
```
phase_1的反汇编代码中，将0x402400赋给了esi，然后调用了函数strings_not_equal，当eax返回0的时候函数正常退出，否则炸弹爆炸。跟进看一下<strings_not_equal>里有什么
```
   0x0000000000401338 <+0>:	push   r12
   0x000000000040133a <+2>:	push   rbp
   0x000000000040133b <+3>:	push   rbx
   0x000000000040133c <+4>:	mov    rbx,rdi
   0x000000000040133f <+7>:	mov    rbp,rsi
   0x0000000000401342 <+10>:	call   0x40131b <string_length>
   0x0000000000401347 <+15>:	mov    r12d,eax
   0x000000000040134a <+18>:	mov    rdi,rbp
   0x000000000040134d <+21>:	call   0x40131b <string_length>
   0x0000000000401352 <+26>:	mov    edx,0x1
   0x0000000000401357 <+31>:	cmp    r12d,eax
   0x000000000040135a <+34>:	jne    0x40139b <strings_not_equal+99>
   0x000000000040135c <+36>:	movzx  eax,BYTE PTR [rbx]
   0x000000000040135f <+39>:	test   al,al
   0x0000000000401361 <+41>:	je     0x401388 <strings_not_equal+80>

   0x0000000000401363 <+43>:	cmp    al,BYTE PTR [rbp+0x0]
   0x0000000000401366 <+46>:	je     0x401372 <strings_not_equal+58>
   0x0000000000401368 <+48>:	jmp    0x40138f <strings_not_equal+87>
   0x000000000040136a <+50>:	cmp    al,BYTE PTR [rbp+0x0]
   0x000000000040136d <+53>:	nop    DWORD PTR [rax]
   0x0000000000401370 <+56>:	jne    0x401396 <strings_not_equal+94>
   0x0000000000401372 <+58>:	add    rbx,0x1
   0x0000000000401376 <+62>:	add    rbp,0x1
   0x000000000040137a <+66>:	movzx  eax,BYTE PTR [rbx]
   0x000000000040137d <+69>:	test   al,al
   0x000000000040137f <+71>:	jne    0x40136a <strings_not_equal+50>
   0x0000000000401381 <+73>:	mov    edx,0x0
   0x0000000000401386 <+78>:	jmp    0x40139b <strings_not_equal+99>
   0x0000000000401388 <+80>:	mov    edx,0x0
   0x000000000040138d <+85>:	jmp    0x40139b <strings_not_equal+99>
   0x000000000040138f <+87>:	mov    edx,0x1
   0x0000000000401394 <+92>:	jmp    0x40139b <strings_not_equal+99>
   0x0000000000401396 <+94>:	mov    edx,0x1
   0x000000000040139b <+99>:	mov    eax,edx
   0x000000000040139d <+101>:	pop    rbx
   0x000000000040139e <+102>:	pop    rbp
   0x000000000040139f <+103>:	pop    r12
   0x00000000004013a1 <+105>:	ret    
```
其中rbx中存放着我们输入的内容，rbp中存放着0x402400，后面程序调用了string_length函数，并判断两次函数的返回值是否一致，不一致则直接返回1并退出，题目中阻止炸弹爆炸的要求就是函数必须返回0，所以必须要让两次string_length的返回值一致才可以
跟进看一下反汇编代码
```
   0x000000000040131b <+0>:	cmp    BYTE PTR [rdi],0x0
   0x000000000040131e <+3>:	je     0x401332 
   0x0000000000401320 <+5>:	mov    rdx,rdi
   0x0000000000401323 <+8>:	add    rdx,0x1
   0x0000000000401327 <+12>:	mov    eax,edx
   0x0000000000401329 <+14>:	sub    eax,edi
   0x000000000040132b <+16>:	cmp    BYTE PThR [rdx],0x0
   0x000000000040132e <+19>:	jne    0x401323 <string_length+8>
   0x0000000000401330 <+21>:	repz ret 
   0x0000000000401332 <+23>:	mov    eax,0x0
   0x0000000000401337 <+28>:	ret    
```
一开始会判断是不是字节0，是的话直接退出并返回0；若不是则会将参数传给rdx并++，再通过eax来计数，判断参数的+1位置处是否为0，是的话返回不是则继续上面的操作。其实就是通过'\x00'来判断字符串的长度
通过在0x000000000040134d处下断点知道了长度为0x34，也就是我们必须输入0x34长度的内容才可以满足条件

![avatar](https://github.com/AmaIIl/AmaIIl.bomb.GitHub.io/blob/gh-pages/image1.png?raw=true)

了解完这个以后回过头来再看看<strings_not_equal>函数，程序给出了一个直接返回0的操作，需要让al 为0，但是前面的string_length那里就没法绕过了，所以这个是不可行的。
再往下走，程序会拿我们输入的字节和程序要求的字节进行对比，不成功则返回1退出函数，成功的话则两者共同+1，并判断是否为0，不是0则重复上述判断步骤，直到全部判断完毕返回0，也就是说我们输入的值必须和0x402400中的内容完全一致才可以满足要求，查看0x402400中的内容，完成本题
```
gef➤  x/s 0x402400
0x402400:	"Border relations with Canada have never been better."
```
![avatar](https://github.com/AmaIIl/AmaIIl.bomb.GitHub.io/blob/gh-pages/image2.png?raw=true)

## phase_2
再来看一下phase_2
程序一开始调用了函数<read_six_numbers>，跟进函数会发现里面有个炸弹函数
```
   0x000000000040145c <+0>:	sub    rsp,0x18
   0x0000000000401460 <+4>:	mov    rdx,rsi
   0x0000000000401463 <+7>:	lea    rcx,[rsi+0x4]
   0x0000000000401467 <+11>:	lea    rax,[rsi+0x14]
   0x000000000040146b <+15>:	mov    QWORD PTR [rsp+0x8],rax
   0x0000000000401470 <+20>:	lea    rax,[rsi+0x10]
   0x0000000000401474 <+24>:	mov    QWORD PTR [rsp],rax
   0x0000000000401478 <+28>:	lea    r9,[rsi+0xc]
   0x000000000040147c <+32>:	lea    r8,[rsi+0x8]
   0x0000000000401480 <+36>:	mov    esi,0x4025c3
   0x0000000000401485 <+41>:	mov    eax,0x0
   0x000000000040148a <+46>:	call   0x400bf0 <__isoc99_sscanf@plt>
   0x000000000040148f <+51>:	cmp    eax,0x5
   0x0000000000401492 <+54>:	jg     0x401499 <read_six_numbers+61>
   0x0000000000401494 <+56>:	call   0x40143a <explode_bomb>
   0x0000000000401499 <+61>:	add    rsp,0x18
   0x000000000040149d <+65>:	ret    
```
观察这个函数会发现调用了sscanf函数，通过这个函数将输入的内容写入栈内，然后判断返回值eax>5则正常退出，否则boom！
而sscanf函数的特点就是返回值是成功匹配的个数，并且遇到空格会结束一次匹配，也就是说只要在字符串间加上足够的空格就可以绕过炸弹了
再来看一下phase_2函数，第一个炸弹函数绕过的条件是[rsp]=0x1，其实现的功能是判断rsp与其高4位地址中的数字，是否满足后者是前者两倍的关系，共匹配六个数字，也就是说从第一个数字1开始每个乘二共六个，再以空格分隔开就可以了
```
   0x0000000000400efc <+0>:	push   rbp
   0x0000000000400efd <+1>:	push   rbx
   0x0000000000400efe <+2>:	sub    rsp,0x28
   0x0000000000400f02 <+6>:	mov    rsi,rsp
   0x0000000000400f05 <+9>:	call   0x40145c <read_six_numbers>
   0x0000000000400f0a <+14>:	cmp    DWORD PTR [rsp],0x1
   0x0000000000400f0e <+18>:	je     0x400f30 <phase_2+52>
   0x0000000000400f10 <+20>:	call   0x40143a <explode_bomb>
   0x0000000000400f15 <+25>:	jmp    0x400f30 <phase_2+52>
   0x0000000000400f17 <+27>:	mov    eax,DWORD PTR [rbx-0x4]
   0x0000000000400f1a <+30>:	add    eax,eax
   0x0000000000400f1c <+32>:	cmp    DWORD PTR [rbx],eax
   0x0000000000400f1e <+34>:	je     0x400f25 <phase_2+41>
   0x0000000000400f20 <+36>:	call   0x40143a <explode_bomb>
   0x0000000000400f25 <+41>:	add    rbx,0x4
   0x0000000000400f29 <+45>:	cmp    rbx,rbp
   0x0000000000400f2c <+48>:	jne    0x400f17 <phase_2+27>
   0x0000000000400f2e <+50>:	jmp    0x400f3c <phase_2+64>
   0x0000000000400f30 <+52>:	lea    rbx,[rsp+0x4]
   0x0000000000400f35 <+57>:	lea    rbp,[rsp+0x18]
   0x0000000000400f3a <+62>:	jmp    0x400f17 <phase_2+27>
   0x0000000000400f3c <+64>:	add    rsp,0x28
   0x0000000000400f40 <+68>:	pop    rbx
   0x0000000000400f41 <+69>:	pop    rbp
   0x0000000000400f42 <+70>:	ret    
```
![avatar](https://github.com/AmaIIl/AmaIIl.bomb.GitHub.io/blob/gh-pages/image3.png?raw=true)

## phase_3
查看反汇编结果
```
   0x0000000000400f43 <+0>:	sub    rsp,0x18
   0x0000000000400f47 <+4>:	lea    rcx,[rsp+0xc]
   0x0000000000400f4c <+9>:	lea    rdx,[rsp+0x8]
   0x0000000000400f51 <+14>:	mov    esi,0x4025cf
   0x0000000000400f56 <+19>:	mov    eax,0x0
   0x0000000000400f5b <+24>:	call   0x400bf0 <__isoc99_sscanf@plt>
   0x0000000000400f60 <+29>:	cmp    eax,0x1
   0x0000000000400f63 <+32>:	jg     0x400f6a <phase_3+39>
   0x0000000000400f65 <+34>:	call   0x40143a <explode_bomb>
   0x0000000000400f6a <+39>:	cmp    DWORD PTR [rsp+0x8],0x7
   0x0000000000400f6f <+44>:	ja     0x400fad <phase_3+106>
   0x0000000000400f71 <+46>:	mov    eax,DWORD PTR [rsp+0x8]
   0x0000000000400f75 <+50>:	jmp    QWORD PTR [rax*8+0x402470]
   0x0000000000400f7c <+57>:	mov    eax,0xcf
   0x0000000000400f81 <+62>:	jmp    0x400fbe <phase_3+123>
   0x0000000000400f83 <+64>:	mov    eax,0x2c3
   0x0000000000400f88 <+69>:	jmp    0x400fbe <phase_3+123>
   0x0000000000400f8a <+71>:	mov    eax,0x100
   0x0000000000400f8f <+76>:	jmp    0x400fbe <phase_3+123>
   0x0000000000400f91 <+78>:	mov    eax,0x185
   0x0000000000400f96 <+83>:	jmp    0x400fbe <phase_3+123>
   0x0000000000400f98 <+85>:	mov    eax,0xce
   0x0000000000400f9d <+90>:	jmp    0x400fbe <phase_3+123>
   0x0000000000400f9f <+92>:	mov    eax,0x2aa
   0x0000000000400fa4 <+97>:	jmp    0x400fbe <phase_3+123>
   0x0000000000400fa6 <+99>:	mov    eax,0x147
   0x0000000000400fab <+104>:	jmp    0x400fbe <phase_3+123>
   0x0000000000400fad <+106>:	call   0x40143a <explode_bomb>
   0x0000000000400fb2 <+111>:	mov    eax,0x0
   0x0000000000400fb7 <+116>:	jmp    0x400fbe <phase_3+123>
   0x0000000000400fb9 <+118>:	mov    eax,0x137
   0x0000000000400fbe <+123>:	cmp    eax,DWORD PTR [rsp+0xc]
   0x0000000000400fc2 <+127>:	je     0x400fc9 <phase_3+134>
   0x0000000000400fc4 <+129>:	call   0x40143a <explode_bomb>
   0x0000000000400fc9 <+134>:	add    rsp,0x18
   0x0000000000400fcd <+138>:	ret    
```
和phase_2一样程序调用了sscanf函数，并且将两个内容存入栈内。程序会判断长度是否大于1，若不大于则炸弹函数爆炸。再接着往下
我们在0x400f5b处（sscanf）下断点，将之前phase_1和2的结果保存在result中省事儿

![avatar](https://github.com/AmaIIl/AmaIIl.bomb.GitHub.io/blob/gh-pages/image4.png?raw=true)

在执行完sscanf后rax寄存器返回值变成了2，绕过第一个炸弹函数

![avatar](https://github.com/AmaIIl/AmaIIl.bomb.GitHub.io/blob/gh-pages/image5.png?raw=true)

再往下执行，程序将判断第一个输入的内容是否大于7，大于的话就爆炸，我们第一个输入的数是1满足要求绕过第二个炸弹
继续执行程序，将我们第一个输入的值赋给了rax并跳转到了一个地方，即[0x402478]，其中的内容是0x400fb9，也就是我们跳转到的地址

![avatar](https://github.com/AmaIIl/AmaIIl.bomb.GitHub.io/blob/gh-pages/image6.png?raw=true)

在这里会对我们输入的第二个值与0x137进行判断，相同完成phase_3

![avatar](https://github.com/AmaIIl/AmaIIl.bomb.GitHub.io/blob/gh-pages/image7.png?raw=true)

也就是说只要我们构建一个字符串1 311(0x137十进制)就可以绕过所有炸弹函数 

![avatar](https://github.com/AmaIIl/AmaIIl.bomb.GitHub.io/blob/gh-pages/image8.png?raw=true)

然后回去在品鉴一下反汇编代码，能看出可通过的字符串有8个
```
//0 207
//1 311
//2 707
//3 256
//4 389
//5 206
//6 682
//7 327
```
## phase_4
查看反汇编代码
```
   0x000000000040100c <+0>:	sub    rsp,0x18
   0x0000000000401010 <+4>:	lea    rcx,[rsp+0xc]
   0x0000000000401015 <+9>:	lea    rdx,[rsp+0x8]
   0x000000000040101a <+14>:	mov    esi,0x4025cf
   0x000000000040101f <+19>:	mov    eax,0x0
   0x0000000000401024 <+24>:	call   0x400bf0 <__isoc99_sscanf@plt>
   0x0000000000401029 <+29>:	cmp    eax,0x2
   0x000000000040102c <+32>:	jne    0x401035 <phase_4+41>
   0x000000000040102e <+34>:	cmp    DWORD PTR [rsp+0x8],0xe
   0x0000000000401033 <+39>:	jbe    0x40103a <phase_4+46>
   0x0000000000401035 <+41>:	call   0x40143a <explode_bomb>
   0x000000000040103a <+46>:	mov    edx,0xe
   0x000000000040103f <+51>:	mov    esi,0x0
   0x0000000000401044 <+56>:	mov    edi,DWORD PTR [rsp+0x8]

   0x0000000000401048 <+60>:	call   0x400fce <func4>
   0x000000000040104d <+65>:	test   eax,eax
   0x000000000040104f <+67>:	jne    0x401058 <phase_4+76>
   0x0000000000401051 <+69>:	cmp    DWORD PTR [rsp+0xc],0x0
   0x0000000000401056 <+74>:	je     0x40105d <phase_4+81>
   0x0000000000401058 <+76>:	call   0x40143a <explode_bomb>
   0x000000000040105d <+81>:	add    rsp,0x18
   0x0000000000401061 <+85>:	ret    
```
程序流程很简单，我们需要输入两个数并用空格间隔开，然后输入的第一个数必须是小于等于14的数，第二个数必须是0，关键就在调用的func4函数上，我们需要让其返回值为0才可以完成本题
查看func4函数反汇编代码
```
   0x0000000000400fce <+0>:	sub    rsp,0x8
   0x0000000000400fd2 <+4>:	mov    eax,edx
   0x0000000000400fd4 <+6>:	sub    eax,esi
   0x0000000000400fd6 <+8>:	mov    ecx,eax
   0x0000000000400fd8 <+10>:	shr    ecx,0x1f
   0x0000000000400fdb <+13>:	add    eax,ecx
   0x0000000000400fdd <+15>:	sar    eax,1
   0x0000000000400fdf <+17>:	lea    ecx,[rax+rsi*1]
   0x0000000000400fe2 <+20>:	cmp    ecx,edi
   0x0000000000400fe4 <+22>:	jle    0x400ff2 <func4+36>
   0x0000000000400fe6 <+24>:	lea    edx,[rcx-0x1]
   0x0000000000400fe9 <+27>:	call   0x400fce <func4>
   0x0000000000400fee <+32>:	add    eax,eax
   0x0000000000400ff0 <+34>:	jmp    0x401007 <func4+57>
   0x0000000000400ff2 <+36>:	mov    eax,0x0
   0x0000000000400ff7 <+41>:	cmp    ecx,edi
   0x0000000000400ff9 <+43>:	jge    0x401007 <func4+57>
   0x0000000000400ffb <+45>:	lea    esi,[rcx+0x1]
   0x0000000000400ffe <+48>:	call   0x400fce <func4>
   0x0000000000401003 <+53>:	lea    eax,[rax+rax*1+0x1]
   0x0000000000401007 <+57>:	add    rsp,0x8
   0x000000000040100b <+61>:	ret    
```
程序会对输入的第一个数与14除以二的结果存入ecx相比较，若不满足小于等于则将14/2结果减一再进入循环，直到满足结果跳转到0x400ff2处，将eax置零后判断两个值是否满足大于等于的条件，满足则直接退出程序，不满足则将ecx的值加一再次进入循环。
可以知道的是，一个数值去除以2的结果只能是0或1，也就是说我们第一个数输入0或1的时候可以满足要求令函数的返回值为0

![avatar](https://github.com/AmaIIl/AmaIIl.bomb.GitHub.io/blob/gh-pages/image9.png?raw=true)

## phase_5
查看反汇编代码
```
   0x0000000000401062 <+0>:	push   rbx
   0x0000000000401063 <+1>:	sub    rsp,0x20
   0x0000000000401067 <+5>:	mov    rbx,rdi
//rbx = input
   0x000000000040106a <+8>:	mov    rax,QWORD PTR fs:0x28
   0x0000000000401073 <+17>:	mov    QWORD PTR [rsp+0x18],rax
   0x0000000000401078 <+22>:	xor    eax,eax
//canary
   0x000000000040107a <+24>:	call   0x40131b <string_length>
   0x000000000040107f <+29>:	cmp    eax,0x6
   0x0000000000401082 <+32>:	je     0x4010d2 <phase_5+112>
//输入string长度必须为6
   0x0000000000401084 <+34>:	call   0x40143a <explode_bomb>
   0x0000000000401089 <+39>:	jmp    0x4010d2 <phase_5+112>
//将eax = 0后跳转回来
   0x000000000040108b <+41>:	movzx  ecx,BYTE PTR [rbx+rax*1]
   0x000000000040108f <+45>:	mov    BYTE PTR [rsp],cl
   0x0000000000401092 <+48>:	mov    rdx,QWORD PTR [rsp]
   0x0000000000401096 <+52>:	and    edx,0xf
   0x0000000000401099 <+55>:	movzx  edx,BYTE PTR [rdx+0x4024b0]
/*
maduiersnfotvbylSo you think you can stop the bomb with ctrl-c, do you?

*/
   0x00000000004010a0 <+62>:	mov    BYTE PTR [rsp+rax*1+0x10],dl
   0x00000000004010a4 <+66>:	add    rax,0x1
   0x00000000004010a8 <+70>:	cmp    rax,0x6
   0x00000000004010ac <+74>:	jne    0x40108b <phase_5+41>
//rax = 0x6
   0x00000000004010ae <+76>:	mov    BYTE PTR [rsp+0x16],0x0
   0x00000000004010b3 <+81>:	mov    esi,0x40245e
//flyers
   0x00000000004010b8 <+86>:	lea    rdi,[rsp+0x10]
   0x00000000004010bd <+91>:	call   0x401338 <strings_not_equal>
   0x00000000004010c2 <+96>:	test   eax,eax
//相等则完成题目
   0x00000000004010c4 <+98>:	je     0x4010d9 <phase_5+119>
   0x00000000004010c6 <+100>:	call   0x40143a <explode_bomb>
   0x00000000004010cb <+105>:	nop    DWORD PTR [rax+rax*1+0x0]
   0x00000000004010d0 <+110>:	jmp    0x4010d9 <phase_5+119>

   0x00000000004010d2 <+112>:	mov    eax,0x0
   0x00000000004010d7 <+117>:	jmp    0x40108b <phase_5+41>

   0x00000000004010d9 <+119>:	mov    rax,QWORD PTR [rsp+0x18]
   0x00000000004010de <+124>:	xor    rax,QWORD PTR fs:0x28
   0x00000000004010e7 <+133>:	je     0x4010ee <phase_5+140>
   0x00000000004010e9 <+135>:	call   0x400b30 <__stack_chk_fail@plt>
   0x00000000004010ee <+140>:	add    rsp,0x20
   0x00000000004010f2 <+144>:	pop    rbx
   0x00000000004010f3 <+145>:	ret    
```
汇编代码看着实在是太乱脑子了直接调试做出来的，先是在string_length处下断点并输入6个a

![avatar](https://github.com/AmaIIl/AmaIIl.bomb.GitHub.io/blob/gh-pages/image10.png?raw=true)

然后继续往下走，rbx+rax*1处存放着我们输入的字符串，并将第一个字符a赋给ecx

![avatar](https://github.com/AmaIIl/AmaIIl.bomb.GitHub.io/blob/gh-pages/image11.png?raw=true)

然后取字符低4位的值并通过这个值为edx赋值

![avatar](https://github.com/AmaIIl/AmaIIl.bomb.GitHub.io/blob/gh-pages/image12.png?raw=true)

查看0x4024b0处的内容

![avatar](https://github.com/AmaIIl/AmaIIl.bomb.GitHub.io/blob/gh-pages/image13.png?raw=true)

也就是说程序会根据我们输入字符串中每个字符的低4位来取字符串对应的字符，然后组成一个新的字符串与"flyers"进行比对，对应的字符如下所示
```
9  -> f
f  -> l
e  -> y
5  -> e
6  -> r
7  -> s
```
只要是低4位是对应的16进制数就可以完成题目，这里选择ionefg

![avatar](https://github.com/AmaIIl/AmaIIl.bomb.GitHub.io/blob/gh-pages/image14.png?raw=true)

## phase_6
查看源代码
第一段时输入6个数，范围需要在1-6之间
```
   0x0000000000401100 <+12>:	mov    r13,rsp
   0x0000000000401103 <+15>:	mov    rsi,rsp
   0x0000000000401106 <+18>:	call   0x40145c <read_six_numbers>
   0x000000000040110b <+23>:	mov    r14,rsp
   0x000000000040110e <+26>:	mov    r12d,0x0
   0x0000000000401114 <+32>:	mov    rbp,r13
   0x0000000000401117 <+35>:	mov    eax,DWORD PTR [r13+0x0]
   0x000000000040111b <+39>:	sub    eax,0x1
   0x000000000040111e <+42>:	cmp    eax,0x5
   0x0000000000401121 <+45>:	jbe    0x401128 <phase_6+52>
   0x0000000000401123 <+47>:	call   0x40143a <explode_bomb>
   0x0000000000401128 <+52>:	add    r12d,0x1
   0x000000000040112c <+56>:	cmp    r12d,0x6
   0x0000000000401130 <+60>:	je     0x401153 <phase_6+95>
   0x0000000000401132 <+62>:	mov    ebx,r12d
   0x0000000000401135 <+65>:	movsxd rax,ebx
```
判断是否有相同数
```
   0x0000000000401138 <+68>:	mov    eax,DWORD PTR [rsp+rax*4]
   0x000000000040113b <+71>:	cmp    DWORD PTR [rbp+0x0],eax
   0x000000000040113e <+74>:	jne    0x401145 <phase_6+81>
   0x0000000000401140 <+76>:	call   0x40143a <explode_bomb>
   0x0000000000401145 <+81>:	add    ebx,0x1
   0x0000000000401148 <+84>:	cmp    ebx,0x5
   0x000000000040114b <+87>:	jle    0x401135 <phase_6+65>
   0x000000000040114d <+89>:	add    r13,0x4
   0x0000000000401151 <+93>:	jmp    0x401114 <phase_6+32>
```
将输入的数字减去7后在放入内存中
```
   0x0000000000401153 <+95>:	lea    rsi,[rsp+0x18]
   0x0000000000401158 <+100>:	mov    rax,r14
   0x000000000040115b <+103>:	mov    ecx,0x7
   0x0000000000401160 <+108>:	mov    edx,ecx
   0x0000000000401162 <+110>:	sub    edx,DWORD PTR [rax]
   0x0000000000401164 <+112>:	mov    DWORD PTR [rax],edx
   0x0000000000401166 <+114>:	add    rax,0x4
   0x000000000040116a <+118>:	cmp    rax,rsi
   0x000000000040116d <+121>:	jne    0x401160 <phase_6+108>
```
前面这些还挺好理解的，后面都是一边调试一边分析的。。
程序再往下走，从rsp+0x20开始存入根据我们输入的值所得到的依次是
6 --> 14c=332
5 --> 0a8=168
4 --> 39c=924
3 --> 2b3=691
2 --> 1dd=477
1 --> 1bb=443
然后程序会将他们用从前往后的顺序依次两两进行比较大小，当前者大于后者时满足条件，否则炸弹爆炸，这样的话只需要将他们转换后的结果依次按照从大到小的顺序进行排列即可完成本题
```
   0x000000000040116f <+123>:	mov    esi,0x0
   0x0000000000401174 <+128>:	jmp    0x401197 <phase_6+163>
   0x0000000000401176 <+130>:	mov    rdx,QWORD PTR [rdx+0x8]
   0x000000000040117a <+134>:	add    eax,0x1
   0x000000000040117d <+137>:	cmp    eax,ecx
   0x000000000040117f <+139>:	jne    0x401176 <phase_6+130>
   0x0000000000401181 <+141>:	jmp    0x401188 <phase_6+148>
   0x0000000000401183 <+143>:	mov    edx,0x6032d0
   0x0000000000401188 <+148>:	mov    QWORD PTR [rsp+rsi*2+0x20],rdx
   0x000000000040118d <+153>:	add    rsi,0x4
   0x0000000000401191 <+157>:	cmp    rsi,0x18
   0x0000000000401195 <+161>:	je     0x4011ab <phase_6+183>
   0x0000000000401197 <+163>:	mov    ecx,DWORD PTR [rsp+rsi*1]
   0x000000000040119a <+166>:	cmp    ecx,0x1 
   0x000000000040119d <+169>:	jle    0x401183 <phase_6+143>
   0x000000000040119f <+171>:	mov    eax,0x1
   0x00000000004011a4 <+176>:	mov    edx,0x6032d0
   0x00000000004011a9 <+181>:	jmp    0x401176 <phase_6+130>
   0x00000000004011ab <+183>:	mov    rbx,QWORD PTR [rsp+0x20]
   0x00000000004011b0 <+188>:	lea    rax,[rsp+0x28]
   0x00000000004011b5 <+193>:	lea    rsi,[rsp+0x50]
   0x00000000004011ba <+198>:	mov    rcx,rbx
   0x00000000004011bd <+201>:	mov    rdx,QWORD PTR [rax]
   0x00000000004011c0 <+204>:	mov    QWORD PTR [rcx+0x8],rdx
   0x00000000004011c4 <+208>:	add    rax,0x8
   0x00000000004011c8 <+212>:	cmp    rax,rsi
   0x00000000004011cb <+215>:	je     0x4011d2 <phase_6+222>
   0x00000000004011cd <+217>:	mov    rcx,rdx
   0x00000000004011d0 <+220>:	jmp    0x4011bd <phase_6+201>
   0x00000000004011d2 <+222>:	mov    QWORD PTR [rdx+0x8],0x0
   0x00000000004011da <+230>:	mov    ebp,0x5
   0x00000000004011df <+235>:	mov    rax,QWORD PTR [rbx+0x8]
   0x00000000004011e3 <+239>:	mov    eax,DWORD PTR [rax]
   0x00000000004011e5 <+241>:	cmp    DWORD PTR [rbx],eax
   0x00000000004011e7 <+243>:	jge    0x4011ee <phase_6+250>
   0x00000000004011e9 <+245>:	call   0x40143a <explode_bomb>
   0x00000000004011ee <+250>:	mov    rbx,QWORD PTR [rbx+0x8]
   0x00000000004011f2 <+254>:	sub    ebp,0x1
   0x00000000004011f5 <+257>:	jne    0x4011df <phase_6+235>
   0x00000000004011f7 <+259>:	add    rsp,0x50
   0x00000000004011fb <+263>:	pop    rbx
   0x00000000004011fc <+264>:	pop    rbp
   0x00000000004011fd <+265>:	pop    r12
   0x00000000004011ff <+267>:	pop    r13
   0x0000000000401201 <+269>:	pop    r14
   0x0000000000401203 <+271>: ret   
```
![avatar](https://github.com/AmaIIl/AmaIIl.bomb.GitHub.io/blob/gh-pages/image15.png?raw=true)

### 小结
汇编代码感觉还是一边调试一边分析比较好读一些，然后就是这个lab真的弄得挺好玩的，期待下一个lab，打完收工！

