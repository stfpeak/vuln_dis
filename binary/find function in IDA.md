#### find function in IDA

```c
#include <idc.idc>

/*
程序功能：在main()里枚举所有函数，判断每个函数名是否带有memcpy、strncpy等关键字，
如果有则过滤危险函数的调用点（包括指针call等调用），并把栈缓冲区标志打印出来。
本程序只适用于所有代码都编在一起的执行文件，如果代码分属多个so文件，则可能不适合。
*/

/*
函数功能：字符串大写转小写，IDA未提供该函数，需要自己实现
*/
static strlwr(szString)
{
	auto szDstStr = "";
	auto iLen = strlen(szString);
	auto i;

	for (i = 0; i < iLen; i++)
	{
		auto szChar = substr(szString, i, i + 1);
		if (-1 != strstr(szChar, "A"))
		{
			szDstStr = szDstStr + "a";
		}
		else if (-1 != strstr(szChar, "B"))
		{
			szDstStr = szDstStr + "b";
		}
		else if (-1 != strstr(szChar, "C"))
		{
			szDstStr = szDstStr + "c";
		}
		else if (-1 != strstr(szChar, "D"))
		{
			szDstStr = szDstStr + "d";
		}
		else if (-1 != strstr(szChar, "E"))
		{
			szDstStr = szDstStr + "e";
		}
		else if (-1 != strstr(szChar, "F"))
		{
			szDstStr = szDstStr + "f";
		}
		else if (-1 != strstr(szChar, "G"))
		{
			szDstStr = szDstStr + "g";
		}
		else if (-1 != strstr(szChar, "H"))
		{
			szDstStr = szDstStr + "h";
		}
		else if (-1 != strstr(szChar, "I"))
		{
			szDstStr = szDstStr + "i";
		}
		else if (-1 != strstr(szChar, "J"))
		{
			szDstStr = szDstStr + "j";
		}
		else if (-1 != strstr(szChar, "K"))
		{
			szDstStr = szDstStr + "k";
		}
		else if (-1 != strstr(szChar, "L"))
		{
			szDstStr = szDstStr + "l";
		}
		else if (-1 != strstr(szChar, "M"))
		{
			szDstStr = szDstStr + "m";
		}
		else if (-1 != strstr(szChar, "N"))
		{
			szDstStr = szDstStr + "n";
		}
		else if (-1 != strstr(szChar, "O"))
		{
			szDstStr = szDstStr + "o";
		}
		else if (-1 != strstr(szChar, "P"))
		{
			szDstStr = szDstStr + "p";
		}
		else if (-1 != strstr(szChar, "Q"))
		{
			szDstStr = szDstStr + "q";
		}
		else if (-1 != strstr(szChar, "R"))
		{
			szDstStr = szDstStr + "r";
		}
		else if (-1 != strstr(szChar, "S"))
		{
			szDstStr = szDstStr + "s";
		}
		else if (-1 != strstr(szChar, "T"))
		{
			szDstStr = szDstStr + "t";
		}
		else if (-1 != strstr(szChar, "U"))
		{
			szDstStr = szDstStr + "u";
		}
		else if (-1 != strstr(szChar, "V"))
		{
			szDstStr = szDstStr + "v";
		}
		else if (-1 != strstr(szChar, "W"))
		{
			szDstStr = szDstStr + "w";
		}
		else if (-1 != strstr(szChar, "X"))
		{
			szDstStr = szDstStr + "x";
		}
		else if (-1 != strstr(szChar, "Y"))
		{
			szDstStr = szDstStr + "y";
		}
		else if (-1 != strstr(szChar, "Z"))
		{
			szDstStr = szDstStr + "z";
		}
		else
		{
			szDstStr = szDstStr + szChar;
		}
	}

	return szDstStr;
}

/*
函数功能：检查memcpy的长度参数，判断一个函数调用点是否可疑
参数：
information = 信息
func_entry = 函数入口
call_point = 函数调用点
hfile = 文件句柄
*/
static check_call_len_args(information, func_entry, call_point, hfile) //("JAL", @VOS_MemCpy, @call VOS_MemCpy, hfile)
{
	auto lpLenArg_Max   = 0; //保存给长度寄存器赋值的指令地址 Max
	auto lpLenArg_Count = 0; //保存给长度寄存器赋值的指令地址 Count
	
	auto lpFunctionStart = GetFunctionAttr(call_point, FUNCATTR_START);
	auto lpFunctionEnd = GetFunctionAttr(call_point, FUNCATTR_END);

	auto prev_instr_tmp = PrevNotTail(call_point); //定位到前条指令
	auto prev_instr = PrevNotTail(call_point); //定位到前条指令
	for (; lpFunctionStart < prev_instr && prev_instr < lpFunctionEnd; prev_instr_tmp = RfirstB(prev_instr)) //往前枚举指令
	{
		if (-1 == prev_instr_tmp) prev_instr = PrevNotTail(prev_instr);
		else if (prev_instr < prev_instr_tmp) prev_instr = PrevNotTail(prev_instr);
		else if (-1 != prev_instr_tmp) prev_instr = prev_instr_tmp;

		if (0 == strstr(GetMnem(prev_instr), "mov") && -1 != strstr(GetOpnd(prev_instr, 0), "[esp+4]")) //判断是否是给长度寄存器赋值 第二个参数 目标缓冲区的最大长度
		{
			lpLenArg_Max = prev_instr; //保存给长度寄存器赋值的指令地址
			//Message("\nlpLenArg_Max=%x", lpLenArg_Max);
			break; //只要目标操作数符合条件就要及时退出，而不管源操作数
		}
		if (0 == strstr(GetMnem(prev_instr), "mov") && -1 != strstr(GetOpnd(prev_instr, 0), "[esp+0C]")) //判断是否是给长度寄存器赋值 第二个参数 目标缓冲区的最大长度
		{
			lpLenArg_Count = prev_instr; //保存给长度寄存器赋值的指令地址
			//Message("\n lpLenArg_Count=%x", lpLenArg_Count);
			break; //只要目标操作数符合条件就要及时退出，而不管源操作数
		}
	}

	if (0 == lpLenArg_Max) return; //lpLenArg_Max = lpFunctionStart;
	if ((5 == GetOpType(lpLenArg_Max,1))|| (5 == GetOpType(lpLenArg_Count,1)))//获取第二操作数的类型
	{
	    return;
	}
	
	Message("\r\n need record.");
	//Message("\r\n GetOpType(lpLenArg_Max,1) = %d",GetOpType(lpLenArg_Max,1));
	Message("\r\n call_point = %x,lpLenArg_Max = %x",call_point,lpLenArg_Max);
	
	auto bHasLenArg = "HasLenArg"; //设置memcpy默认为有长度参数

	if (5 != GetOpType(lpLenArg_Max, 1)) //判断给长度寄存器赋值的源操作数是否立即数，如果不是立即数，或者没有赋值，则是一个疑点
	{
		//auto szStack = estimate_call_dst_args(call_point); //判断memcpy类函数的目标缓冲区是堆还是栈空间
		auto szResultStr = sprintf("\r\n%s|%s|%s|0x%08x|%s|%s|", information, bHasLenArg, 0, call_point, GetFunctionName(func_entry), GetFuncOffset(call_point));
		fprintf(hfile, "%s", szResultStr);
		Message("%s", szResultStr);
	}
}



/*
函数功能：枚举所有函数调用点
参数：
func_entry = 函数入口
hfile = 文件句柄
*/
static enumerate_callpoint(func_entry, hfile) //(@"VOS_MemCpy", hfile)
{
	auto ulTextSegStart = SegStart(SegByBase(SegByName(".text"))); //获得text段的开始地址
	auto ulTextSegEnd = SegEnd(SegByBase(SegByName(".text"))); //获得text段的结束地址
	auto  m     = 0;  //计数器，这里为了调试，m > 1是 break
	

	auto call_point = RfirstB(func_entry); //搜索函数第一个调用点
	for (; -1 != call_point; call_point = RnextB(func_entry, call_point)) //循环枚举所有call调用点
	{
	    if(m > 0)
		{
			break;
		}
		if (call_point < ulTextSegStart || call_point > ulTextSegEnd) continue;
        {
		    check_call_len_args("CALL", func_entry, call_point, hfile); //call调用点
			//m++;
		}
		
	}
}

static main()
{
	Message("\r\n.textStart = %x, .textEnd = %x", SegStart(SegByBase(SegByName(".text"))), SegEnd(SegByBase(SegByName(".text"))));
	Message("\r\nSearching...");
	auto hfile = fopen(GetInputFile() + "_scan_suspicion_results_x86linux.txt", "w");
	fprintf(hfile, "%s", "Type|ArgFlag|StackFlag|SuspicionAddr|SuspicionPoint|caller1|caller2|caller3|caller4"); //在文件第一行写入标题
	Message("\r\nType|ArgFlag|StackFlag|SuspicionAddr|SuspicionPoint|caller1|caller2|caller3|caller4");

	auto lpAddr = 0;
	for (; -1 != lpAddr; lpAddr = NextFunction(lpAddr)) //循环枚举所有函数
	{
		auto func_name = strlwr(GetFunctionName(lpAddr)); //获取小写函数名
		
        
		//if (-1 != strstr(func_name, "_safe")) continue; //如果在函数名里包含_safe，则忽略

		if (-1 != strstr(func_name, "vos_memcpy_s")) //判断函数名是否包含敏感子串
		{
			enumerate_callpoint(lpAddr, hfile);
		}
		else if (-1 != strstr(func_name, "strncpy_s"))
		{
			enumerate_callpoint(lpAddr, hfile);
		}
		else if (-1 != strstr(func_name, "memcpy_safe"))
		{
			enumerate_callpoint(lpAddr, hfile);
		}
	}

	//fclose(hfile);
	Message("\r\nCompleted.");
}


```
