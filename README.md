# BoolDebug
- Judge Debug
- 代码来自书籍《有趣的二进制》
- 学校破网上传一直失败，我在这直接拷贝代码上来好了

```c
#include <iostream>
#include <Windows.h>
using namespace std;

int TempEsp;

int BoolDebug0()
{
	return IsDebuggerPresent();
}

int BoolDebug1()
{
	BOOL t;
	CheckRemoteDebuggerPresent((HANDLE)-1, &t);
	return t;
}

int __declspec(naked) BoolDebug2()
{
	__asm
	{
		pushad;
		push ok;
		push dword ptr fs : [0];
		mov dword ptr fs : [0], esp;
		mov TempEsp, esp;
		push 100h;
		popf;
		jmp error;
	ok:
		mov esp, TempEsp;
		pop dword ptr fs : [0];
		add esp, 4;
		popad;
		xor eax, eax;
		ret;
	error:
		mov esp, TempEsp;
		pop dword ptr fs : [0];
		add esp, 4;
		popad;
		xor eax, eax;
		inc eax;
		ret;
	}
}

int __declspec(naked) BoolDebug3()
{
	__asm
	{
		pushad;
		push ok;
		push dword ptr fs : [0];
		mov dword ptr fs : [0], esp;
		mov TempEsp, esp;
		xor eax, eax;
		int 2dh;								//这个代码似乎是有问题的，因为这里的int 2d中断最终的处理就是扔给了kitrap03，如果存在编译器，这里会直接让编译器来处理int 3会出现一个断点
		jmp error;
	ok:
		mov esp, TempEsp;
		pop dword ptr fs : [0];
		add esp, 4;
		popad;
		xor eax, eax;
		ret;
	error:
		mov esp, TempEsp;
		pop dword ptr fs : [0];
		add esp, 4;
		popad;
		xor eax, eax;
		inc eax;
		ret;
	}
}

/*IDA内部的反汇编的代码是这样的
.text:004118A0 ?BoolDebug4@@YAHXZ proc near            ; CODE XREF: BoolDebug4(void)↑j
.text:004118A0                                         ; BoolDebug4(void)↑j
.text:004118A0                 jmp     short near ptr ?BoolDebug4@@YAHXZ+1 ; BoolDebug4(void)
.text:004118A2 ; ---------------------------------------------------------------------------
.text:004118A2                 adc     eax, offset __imp__IsDebuggerPresent@0 ; IsDebuggerPresent()
.text:004118A7                 retn
.text:004118A7 ?BoolDebug4@@YAHXZ endp

EB FF的意思是跳转到当前指令的下一个字节，而当前指令的下一个字节就是正确的call IsDebuggerPresent了
*/
int __declspec(naked) BoolDebug4()
{
	__asm
	{
		_emit 0xEB;
		call IsDebuggerPresent;
		ret;
	}
}

int main()
{
	BOOL tFlag;

	tFlag = BoolDebug4();

	if (tFlag)
		MessageBoxA(GetDesktopWindow(), "当前被调试！", "", MB_OK);
	else
		MessageBoxA(GetDesktopWindow(), "当前未被调试！", "", MB_OK);

	system("pause");
	return 0;
}
```
