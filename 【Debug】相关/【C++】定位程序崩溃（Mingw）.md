# 站在巨人的肩膀上

> [Dr. Mingw](https://github.com/jrfonseca/drmingw)


# 概述
- 剖析 Dr. Mingw 的源代码，抽取最核心的代码，封装成一个动态库，名为 MiniDrMingw.dll。
- MiniDrMingw.dll 是面向过程的，容易看懂，没有什么跳转。
- 可以加入各种个性化需求（如在崩溃时，记录崩溃时的 CPU 占用率、内存占用情况等）。
- 大概使用方式就是：
    ```
    1、应用程序加上编译参数，生成 .map 文件，里面记录着程序的函数地址等。
    # gcc 编译选项，生成 .map 文件
    QMAKE_LFLAGS += -Wl,-Map=$$PWD'/bin/'$$TARGET'.map'
    
    2、应用程序在 main 函数里面，使用下面语句进行初始化：
    ExcHndlInit((a.applicationDirPath() +"/" + a.applicationName() + ".RPT").toLocal8Bit().data());
    ```
- 在崩溃时，如何定位出崩溃的函数，后面在讲 Demo 的时候再说清楚。


# MiniDrMingw.dll 的主要函数：SetUnhandledExceptionFilter-设置异常捕获函数。
- 当异常没有处理的时候,系统就会调用 SetUnhandledExceptionFilter 所设置异常处理函数.
- 当发生异常时，比如内存访问违例时，CPU 硬件会发现此问题，并产生一个异常（你可以把它理解为中断），然后 CPU 会把代码流程切换到异常处理服务例程。操作系统异常处理服务例程会查看当前进程是否处于调试状态。如果是，则通知调试器发生了异常，如果不是则操作系统会查看当前线程是否安装了的异常帧链(FS[0])，如果安装了 SEH（try.... catch....），则调用 SEH，并根据返回结果决定是否全局展开或局部展开。如果异常链中所有的 SEH 都没有处理此异常，而且此进程还处于调试状态，则操作系统会再次通知调试器发生异常（二次异常）。如果还没人处理，则调用操作系统的默认异常处理代码 UnhandledExceptionHandler，不过操作系统允许你 Hook 这个函数，就是通过 SetUnhandledExceptionFilter 函数来设置。大部分异常通过此种方法都能捕获，不过栈溢出、覆盖的有可能捕获不到。
- 该函数说明如下：
	```
	LPTOP_LEVEL_EXCEPTION_FILTER WINAPI SetUnhandledExceptionFilter( _In_LPTOP_LEVEL_EXCEPTION_FILTER lpTopLevelExceptionFilter);
	- 参数：lpTopLevelExceptionFilter，函数指针。当异常发生时，且程序不处于调试模式（在 vs 或者别的调试器里运行）则首先调用该函数。
	- 返回值：返回以前设置的回调函数。
	``` 


# MiniDrMingw.dll 的函数调用关系
```
void ExcHndlInit(const char *pCrashFileName)
	↓
SetUnhandledExceptionFilter(MyUnhandledExceptionFilter)
	↓
MyUnhandledExceptionFilter
	↓
GenerateExceptionReport
	↓
dumpException(hProcess, pExceptionRecord)、dumpStack(hProcess, GetCurrentThread(), pContext)、dumpModules(hProcess);
```
- 详细说明可以看下面源代码的注释。


# MiniDrMingw.dll 的源代码
- 虽然用 QtCreator 开发，但是源代码基本没有用到 Qt 的东西。
- 以动态库的形式。
- 分为以前几个文件：
	- main.cpp：导出函数、主要函数。
	- dllexport.h：导出接口。
	- log.h：写文件。
	- formatoutput.h：将崩溃信息格式为可读信息。
	- util.h：一些辅助类的函数。
- 工程文件 DrMingwDemo.pro 源代码如下：
    ```
	// MiniDrMingw.pro
	QT += core

    TARGET = MiniDrMingw
    TEMPLATE = lib
    DESTDIR = bin
    
    CONFIG += skip_target_version_ext
    
    DEFINES += __SHARE_EXPORT
    
    win32{
    QMAKE_LFLAGS += -Wl,--add-stdcall-alias
    }
    
    LIBS += -lpsapi -lversion -ldbghelp
    
    HEADERS += \
        dllexport.h \
        log.h \
        formatoutput.h \
        util.h
    
    unix {
        target.path = /usr/lib
        INSTALLS += target
    }
    
    SOURCES += \
        main.cpp
	```
- main.cpp 源代码如下：
	```
	// main.cpp
	#include "dllexport.h"
    #include <windows.h>
    #include <string>
    #include <imagehlp.h>
    #include <shlobj.h>
    #include <assert.h>
    #include <psapi.h>
    #include <tlhelp32.h>
    #include "log.h"
    #include "formatoutput.h"
    
    static std::string g_sCrashFileName;
    static BOOL g_bHandlerSet = FALSE;
    static LPTOP_LEVEL_EXCEPTION_FILTER g_prevUnhandledExceptionFilter = nullptr;
    
    DWORD SetSymOptions(BOOL fDebug)
    {
        DWORD dwSymOptions = SymGetOptions();
    
        // We have more control calling UnDecorateSymbolName directly Also, we
        // don't want DbgHelp trying to undemangle MinGW symbols (e.g., from DLL
        // exports) behind our back (as it will just strip the leading underscore.)
        if (0) {
            dwSymOptions |= SYMOPT_UNDNAME;
        } else {
            dwSymOptions &= ~SYMOPT_UNDNAME;
        }
    
        dwSymOptions |= SYMOPT_LOAD_LINES | SYMOPT_OMAP_FIND_NEAREST;
    
        if (TRUE) {
            dwSymOptions |= SYMOPT_DEFERRED_LOADS;
        }
    
        if (fDebug) {
            dwSymOptions |= SYMOPT_DEBUG;
        }
    
    #ifdef _WIN64
        dwSymOptions |= SYMOPT_INCLUDE_32BIT_MODULES;
    #endif
    
        return SymSetOptions(dwSymOptions);
    }
    
    BOOL InitializeSym(HANDLE hProcess, BOOL fInvadeProcess)
    {
        // Provide default symbol search path
        // https://msdn.microsoft.com/en-us/library/windows/desktop/ms680689.aspx
        // http://msdn.microsoft.com/en-gb/library/windows/hardware/ff558829.aspx
        std::string sSymSearchPathBuf;
        const char *szSymSearchPath = nullptr;
        if (getenv("_NT_SYMBOL_PATH") == nullptr && getenv("_NT_ALT_SYMBOL_PATH") == nullptr) {
            char szLocalAppData[MAX_PATH];
            HRESULT hr = SHGetFolderPathA(nullptr, CSIDL_LOCAL_APPDATA, nullptr, 0, szLocalAppData);
            assert(SUCCEEDED(hr));
            if (SUCCEEDED(hr)) {
                // Cache symbols in %APPDATA%\drmingw
                sSymSearchPathBuf += "srv*";
                sSymSearchPathBuf += szLocalAppData;
                sSymSearchPathBuf += "\\drmingw*http://msdl.microsoft.com/download/symbols";
                szSymSearchPath = sSymSearchPathBuf.c_str();
            } else {
                // No cache
                szSymSearchPath = "srv*http://msdl.microsoft.com/download/symbols";
            }
        }
    
        return SymInitialize(hProcess, szSymSearchPath, fInvadeProcess);
    }
    
    void GenerateExceptionReport(PEXCEPTION_POINTERS pExceptionInfo)
    {
        // pExceptionInfo 的结构体信息扩展开来如下：
        // typedef struct _EXCEPTION_POINTERS {
        //  DWORD ExceptionCode;
        //  DWORD ExceptionFlags;
        //  struct _EXCEPTION_RECORD *ExceptionRecord;
        //  PVOID ExceptionAddress;
        //  DWORD NumberParameters;
        //  ULONG_PTR ExceptionInformation[EXCEPTION_MAXIMUM_PARAMETERS];
        //
        //  DWORD ContextFlags;
        //  DWORD Dr0;
        //  DWORD Dr1;
        //  DWORD Dr2;
        //  DWORD Dr3;
        //  DWORD Dr6;
        //  DWORD Dr7;
        //  DWORD ControlWord;
        //  DWORD StatusWord;
        //  DWORD TagWord;
        //  DWORD ErrorOffset;
        //  DWORD ErrorSelector;
        //  DWORD DataOffset;
        //  DWORD DataSelector;
        //  BYTE RegisterArea[SIZE_OF_80387_REGISTERS];
        //  DWORD Cr0NpxState;
        //  DWORD SegGs;
        //  DWORD SegFs;
        //  DWORD SegEs;
        //  DWORD SegDs;
        //  DWORD Edi;
        //  DWORD Esi;
        //  DWORD Ebx;
        //  DWORD Edx;
        //  DWORD Ecx;
        //  DWORD Eax;
        //  DWORD Ebp;
        //  DWORD Eip;
        //  DWORD SegCs;
        //  DWORD EFlags;
        //  DWORD Esp;
        //  DWORD SegSs;
        //  BYTE ExtendedRegisters[MAXIMUM_SUPPORTED_EXTENSION];
        //} EXCEPTION_POINTERS,*PEXCEPTION_POINTERS;
    
        // Start out with a banner
        writeLog("-------------------\n\n");
    
        // 打印系统时间
        SYSTEMTIME SystemTime;
        GetLocalTime(&SystemTime);
        char szDateStr[128];
        LCID Locale = MAKELCID(MAKELANGID(LANG_ENGLISH, SUBLANG_ENGLISH_US), SORT_DEFAULT);
        GetDateFormatA(Locale, 0, &SystemTime, "dddd',' MMMM d',' yyyy", szDateStr, _countof(szDateStr));
        char szTimeStr[128];
        GetTimeFormatA(Locale, 0, &SystemTime, "HH':'mm':'ss", szTimeStr, _countof(szTimeStr));
        writeLog("Error occurred on %s at %s.\n\n", szDateStr, szTimeStr);
    
        // 获取当前进程的一个句柄
        HANDLE hProcess = GetCurrentProcess();
    
        SetSymOptions(FALSE);
    
        PEXCEPTION_RECORD pExceptionRecord = pExceptionInfo->ExceptionRecord;
        if(InitializeSym(hProcess, TRUE))
        {
            // 打印崩溃地址信息
            dumpException(hProcess, pExceptionRecord);
    
            PCONTEXT pContext = pExceptionInfo->ContextRecord;
            assert(pContext);
    
            // XXX: In 64-bits WINE we can get context record that don't match the exception record somehow
    #ifdef _WIN64
            PVOID ip = (PVOID)pContext->Rip;
    #else
            PVOID ip = (PVOID)pContext->Eip;
    #endif
            if(pExceptionRecord->ExceptionAddress != ip)
            {
                writeLog("warning: inconsistent exception context record\n");
            }
    
            // 函数调用栈信息
            dumpStack(hProcess, GetCurrentThread(), pContext);
    
            if (!SymCleanup(hProcess)) {
                assert(0);
            }
        }
    
        // 打印调用模块的信息
        dumpModules(hProcess);
        writeLog("\n");
    }
    
    // 当出现未被处理的异常时，会先调用该函数
    static LONG WINAPI MyUnhandledExceptionFilter(PEXCEPTION_POINTERS pExceptionInfo)
    {
        static LONG nLockNum = 0;
        // 实现数的原子性加减
        if(1 == InterlockedIncrement(&nLockNum))
        {
            // SetErrorMode() 函数控制 Windows 是否处理指定类型的严重错误或使调用应用程序来处理它们。
            // SEM_FAILCRITICALERRORS: 系统不显示关键错误处理消息框。相反，系统发送错误给调用进程。
            // SEM_NOGPFAULTERRORBOX: 系统不显示Windows错误报告对话框。
            // SEM_NOALIGNMENTFAULTEXCEPT: 系统会自动修复故障。此功能只支持部分处理器架构。
            // SEM_NOOPENFILEERRORBOX：当无法找到文件时不弹出错误对话框。相反，错误返回给调用进程。
            UINT oldErrorMode = SetErrorMode(SEM_FAILCRITICALERRORS | SEM_NOGPFAULTERRORBOX | SEM_NOOPENFILEERRORBOX);
    
            // 创建崩溃时写到磁盘的 .RPT 文件
            if(nullptr == g_handleCrashFile)
            {
                g_handleCrashFile = CreateFileA(g_sCrashFileName.c_str(), GENERIC_WRITE, FILE_SHARE_READ | FILE_SHARE_WRITE, 0, OPEN_ALWAYS, 0, 0);
            }
    
            if(nullptr != g_handleCrashFile)
            {
                // 在一个文件中设置新的读取位置
                SetFilePointer(g_handleCrashFile, 0, 0, FILE_END);
    
                // 主要通过该函数来生成异常报告信息，其中 pExceptionInfo 是异常信息结构体指针。
                GenerateExceptionReport(pExceptionInfo);
    
                // 刷新输出缓存区（也就是为了实时写磁盘）
                FlushFileBuffers(g_handleCrashFile);
            }
    
            // 把错误模式设置回去
            SetErrorMode(oldErrorMode);
        }
    
        // 实现数的原子性加减
        InterlockedDecrement(&nLockNum);
    
        // 以前也设置过异常捕获函数，那么处理完我们的异常信息之后，我们再调用它。
        if(g_prevUnhandledExceptionFilter)
            return g_prevUnhandledExceptionFilter(pExceptionInfo);
        else
            return EXCEPTION_CONTINUE_SEARCH;
    }
    
    void ExcHndlInit(const char *pCrashFileName)
    {
        if(nullptr != pCrashFileName)
            g_sCrashFileName = pCrashFileName;
        else
            g_sCrashFileName = "./crash.RPT";
    
        if(FALSE == g_bHandlerSet)
        {
            // 设置自己的异常捕获函数，返回以前设置的异常捕获函数
            g_prevUnhandledExceptionFilter = SetUnhandledExceptionFilter(MyUnhandledExceptionFilter);
            g_bHandlerSet = TRUE;
        }
    }
	```
- formatoutput.h 源代码如下：
	```
	// formatoutput.h
	#ifndef FORMATOUTPUT_H
    #define FORMATOUTPUT_H
    
    #include "log.h"
    #include "util.h"
    
    BOOL dumpSourceCode(LPCSTR lpFileName, DWORD dwLineNumber)
    {
        FILE *fp;
        unsigned i;
        DWORD dwContext = 2;
    
        if ((fp = fopen(lpFileName, "r")) == NULL)
            return FALSE;
    
        i = 0;
        while (!feof(fp) && ++i <= dwLineNumber + dwContext) {
            int c;
    
            if ((int)i >= (int)dwLineNumber - (int)dwContext) {
                writeLog(i == dwLineNumber ? ">%5i: " : "%6i: ", i);
                while (!feof(fp) && (c = fgetc(fp)) != '\n')
                    if (isprint(c))
                        writeLog("%c", c);
                writeLog("\n");
            } else {
                while (!feof(fp) && fgetc(fp) != '\n')
                    ;
            }
        }
    
        fclose(fp);
        return TRUE;
    }
    
    void dumpContext(
    #ifdef _WIN64
        const WOW64_CONTEXT *pContext
    #else
        const CONTEXT *pContext
    #endif
    )
    {
        // Show the registers
        writeLog("Registers:\n");
        if (pContext->ContextFlags & CONTEXT_INTEGER) {
            writeLog("eax=%08lx ebx=%08lx ecx=%08lx edx=%08lx esi=%08lx edi=%08lx\n", pContext->Eax,
                    pContext->Ebx, pContext->Ecx, pContext->Edx, pContext->Esi, pContext->Edi);
        }
        if (pContext->ContextFlags & CONTEXT_CONTROL) {
            writeLog("eip=%08lx esp=%08lx ebp=%08lx iopl=%1lx %s %s %s %s %s %s %s %s %s %s\n",
                    pContext->Eip, pContext->Esp, pContext->Ebp,
                    (pContext->EFlags >> 12) & 3,                  //  IOPL level value
                    pContext->EFlags & 0x00100000 ? "vip" : "   ", //  VIP (virtual interrupt pending)
                    pContext->EFlags & 0x00080000 ? "vif" : "   ", //  VIF (virtual interrupt flag)
                    pContext->EFlags & 0x00000800 ? "ov" : "nv",   //  VIF (virtual interrupt flag)
                    pContext->EFlags & 0x00000400 ? "dn" : "up",   //  OF (overflow flag)
                    pContext->EFlags & 0x00000200 ? "ei" : "di",   //  IF (interrupt enable flag)
                    pContext->EFlags & 0x00000080 ? "ng" : "pl",   //  SF (sign flag)
                    pContext->EFlags & 0x00000040 ? "zr" : "nz",   //  ZF (zero flag)
                    pContext->EFlags & 0x00000010 ? "ac" : "na",   //  AF (aux carry flag)
                    pContext->EFlags & 0x00000004 ? "po" : "pe",   //  PF (parity flag)
                    pContext->EFlags & 0x00000001 ? "cy" : "nc"    //  CF (carry flag)
            );
        }
        if (pContext->ContextFlags & CONTEXT_SEGMENTS) {
            writeLog("cs=%04lx  ss=%04lx  ds=%04lx  es=%04lx  fs=%04lx  gs=%04lx", pContext->SegCs,
                    pContext->SegSs, pContext->SegDs, pContext->SegEs, pContext->SegFs,
                    pContext->SegGs);
            if (pContext->ContextFlags & CONTEXT_CONTROL) {
                writeLog("             efl=%08lx", pContext->EFlags);
            }
        } else {
            if (pContext->ContextFlags & CONTEXT_CONTROL) {
                writeLog("                                                                       "
                        "efl=%08lx",
                        pContext->EFlags);
            }
        }
        writeLog("\n\n");
    }
    
    void dumpStack(HANDLE hProcess, HANDLE hThread, const CONTEXT *pContext)
    {
        DWORD MachineType;
    
        assert(pContext);
    
        STACKFRAME64 StackFrame;
        ZeroMemory(&StackFrame, sizeof StackFrame);
    
        BOOL bWow64 = FALSE;
        if (0) {       // if (HAVE_WIN64)
            IsWow64Process(hProcess, &bWow64);
        }
        if (bWow64) {
            const WOW64_CONTEXT *pWow64Context = reinterpret_cast<const WOW64_CONTEXT *>(pContext);
            // NOLINTNEXTLINE(clang-analyzer-core.NullDereference)
            assert((pWow64Context->ContextFlags & WOW64_CONTEXT_FULL) == WOW64_CONTEXT_FULL);
    #ifdef _WIN64
            dumpContext(pWow64Context);
    #endif
            MachineType = IMAGE_FILE_MACHINE_I386;
            StackFrame.AddrPC.Offset = pWow64Context->Eip;
            StackFrame.AddrStack.Offset = pWow64Context->Esp;
            StackFrame.AddrFrame.Offset = pWow64Context->Ebp;
        } else {
            // NOLINTNEXTLINE(clang-analyzer-core.NullDereference)
            assert((pContext->ContextFlags & CONTEXT_FULL) == CONTEXT_FULL);
    #ifndef _WIN64
            MachineType = IMAGE_FILE_MACHINE_I386;
            dumpContext(pContext);                      // 打印当时寄存器的信息
            StackFrame.AddrPC.Offset = pContext->Eip;
            StackFrame.AddrStack.Offset = pContext->Esp;
            StackFrame.AddrFrame.Offset = pContext->Ebp;
    #else
            MachineType = IMAGE_FILE_MACHINE_AMD64;
            StackFrame.AddrPC.Offset = pContext->Rip;
            StackFrame.AddrStack.Offset = pContext->Rsp;
            StackFrame.AddrFrame.Offset = pContext->Rbp;
    #endif
        }
        StackFrame.AddrPC.Mode = AddrModeFlat;
        StackFrame.AddrStack.Mode = AddrModeFlat;
        StackFrame.AddrFrame.Mode = AddrModeFlat;
    
        /*
         * StackWalk64 modifies Context, so pass a copy.
         */
        CONTEXT Context = *pContext;
    
        if (MachineType == IMAGE_FILE_MACHINE_I386) {
            writeLog("AddrPC   Params\n");
        } else {
            writeLog("AddrPC           Params\n");
        }
    
        BOOL bInsideWine = []() -> BOOL
        {
                HMODULE hNtDll = GetModuleHandleA("ntdll");
                if (!hNtDll) {
                    return FALSE;
                }
                return GetProcAddress(hNtDll, "wine_get_version") != NULL;
        }();
    
        DWORD64 PrevFrameStackOffset = StackFrame.AddrStack.Offset - 1;
        int nudge = 0;
    
        while (TRUE) {
            constexpr int MAX_SYM_NAME_SIZE = 512;
            char szSymName[MAX_SYM_NAME_SIZE] = "";
            char szFileName[MAX_PATH] = "";
            DWORD dwLineNumber = 0;
    
            if (!StackWalk64(MachineType, hProcess, hThread, &StackFrame, &Context,
                             NULL, // ReadMemoryRoutine
                             SymFunctionTableAccess64, SymGetModuleBase64,
                             NULL // TranslateAddress
                             ))
                break;
    
            if (MachineType == IMAGE_FILE_MACHINE_I386) {
                writeLog("%08lX %08lX %08lX %08lX", (DWORD)StackFrame.AddrPC.Offset,
                        (DWORD)StackFrame.Params[0], (DWORD)StackFrame.Params[1],
                        (DWORD)StackFrame.Params[2]);
            } else {
                writeLog("%016I64X %016I64X %016I64X %016I64X", StackFrame.AddrPC.Offset,
                        StackFrame.Params[0], StackFrame.Params[1], StackFrame.Params[2]);
            }
    
            BOOL bSymbol = TRUE;
            BOOL bLine = FALSE;
            DWORD dwOffsetFromSymbol = 0;
    
            DWORD64 AddrPC = StackFrame.AddrPC.Offset;
            HMODULE hModule = (HMODULE)(INT_PTR)SymGetModuleBase64(hProcess, AddrPC);
            char szModule[MAX_PATH];
            if (hModule && GetModuleFileNameExA(hProcess, hModule, szModule, MAX_PATH)) {
                writeLog("  %s", getBaseName(szModule));
    
                bSymbol = GetSymFromAddr(hProcess, AddrPC + nudge, szSymName, MAX_SYM_NAME_SIZE,
                                         &dwOffsetFromSymbol);
                if (bSymbol) {
                    writeLog("!%s+0x%lx", szSymName, dwOffsetFromSymbol - nudge);
    
                    bLine =
                        GetLineFromAddr(hProcess, AddrPC + nudge, szFileName, MAX_PATH, &dwLineNumber);
                    if (bLine) {
                        writeLog("  [%s @ %ld]", szFileName, dwLineNumber);
                    }
                } else {
                    writeLog("!0x%I64x", AddrPC - (DWORD64)(INT_PTR)hModule);
                }
            }
    
            writeLog("\n");
    
            if (bLine) {
                dumpSourceCode(szFileName, dwLineNumber);
            }
    
            // Basic sanity check to make sure  the frame is OK.  Bail if not.
            if (StackFrame.AddrStack.Offset <= PrevFrameStackOffset ||
                StackFrame.AddrPC.Offset == 0xBAADF00D) {
                break;
            }
            PrevFrameStackOffset = StackFrame.AddrStack.Offset;
    
            // Wine's StackWalk64 implementation on certain yield never ending
            // stack backtraces unless one bails out when AddrFrame is zero.
            if (bInsideWine && StackFrame.AddrFrame.Offset == 0) {
                break;
            }
    
            /*
             * When we walk into the callers, StackFrame.AddrPC.Offset will not
             * contain the calling function's address, but rather the return
             * address.  This could be the next statement, or sometimes (for
             * no-return functions) a completely different function, so nudge the
             * address by one byte to ensure we get the information about the
             * calling statement itself.
             */
            nudge = -1;
        }
    
        writeLog("\n");
    }
    
    void dumpException(HANDLE hProcess, PEXCEPTION_RECORD pExceptionRecord)
    {
        NTSTATUS ExceptionCode = pExceptionRecord->ExceptionCode;
    
        char szModule[MAX_PATH];
        LPCSTR lpcszProcess;
        HMODULE hModule;
    
        if (GetModuleFileNameExA(hProcess, NULL, szModule, MAX_PATH))
        {
            lpcszProcess = getBaseName(szModule);
        }
        else
        {
            lpcszProcess = "Application";
        }
    
        // First print information about the type of fault
        writeLog("%s caused", lpcszProcess);
    
        LPCSTR lpcszException = getExceptionString(ExceptionCode);
        if (lpcszException) {
            LPCSTR lpszArticle;
            switch (lpcszException[0]) {
            case 'A':
            case 'E':
            case 'I':
            case 'O':
            case 'U':
                lpszArticle = "an";
                break;
            default:
                lpszArticle = "a";
                break;
            }
    
            writeLog(" %s %s", lpszArticle, lpcszException);
        } else {
            writeLog(" an Unknown [0x%lX] Exception", ExceptionCode);
        }
    
        // Now print information about where the fault occurred
        writeLog(" at location %p", pExceptionRecord->ExceptionAddress);
        if((hModule = (HMODULE)(INT_PTR)
                 SymGetModuleBase64(hProcess, (DWORD64)(INT_PTR)pExceptionRecord->ExceptionAddress)) &&
            GetModuleFileNameExA(hProcess, hModule, szModule, sizeof szModule))
        {
            writeLog(" in module %s", getBaseName(szModule));
        }
    
        // If the exception was an access violation, print out some additional information, to the error
        // log and the debugger.
        // https://msdn.microsoft.com/en-us/library/windows/desktop/aa363082%28v=vs.85%29.aspx
        if ((ExceptionCode == EXCEPTION_ACCESS_VIOLATION || ExceptionCode == EXCEPTION_IN_PAGE_ERROR) &&
            pExceptionRecord->NumberParameters >= 2)
        {
            LPCSTR lpszVerb;
            switch (pExceptionRecord->ExceptionInformation[0]) {
            case 0:
                lpszVerb = "Reading from";
                break;
            case 1:
                lpszVerb = "Writing to";
                break;
            case 8:
                lpszVerb = "DEP violation at";
                break;
            default:
                lpszVerb = "Accessing";
                break;
            }
    
            writeLog(" %s location %p", lpszVerb, (PVOID)pExceptionRecord->ExceptionInformation[1]);
        }
    
        writeLog(".\n\n");
    }
    
    void dumpModules(HANDLE hProcess)
    {
        HANDLE hModuleSnap = CreateToolhelp32Snapshot(TH32CS_SNAPMODULE, GetProcessId(hProcess));
        if (hModuleSnap == INVALID_HANDLE_VALUE) {
            return;
        }
    
        DWORD MachineType;
    #ifdef _WIN64
        BOOL bWow64 = FALSE;
        IsWow64Process(hProcess, &bWow64);
        if (bWow64) {
            MachineType = IMAGE_FILE_MACHINE_I386;
        } else {
    #else
        {
    #endif
    #ifndef _WIN64
            MachineType = IMAGE_FILE_MACHINE_I386;
    #else
            MachineType = IMAGE_FILE_MACHINE_AMD64;
    #endif
        }
    
        MODULEENTRY32 me32;
        me32.dwSize = sizeof me32;
        if (Module32First(hModuleSnap, &me32)) {
            do {
                if (MachineType == IMAGE_FILE_MACHINE_I386) {
                    writeLog(
                        "%08lX-%08lX ",
                        (DWORD)(DWORD64)me32.modBaseAddr,
                        (DWORD)(DWORD64)me32.modBaseAddr + me32.modBaseSize
                    );
                } else {
                    writeLog(
                        "%016I64X-%016I64X ",
                        (DWORD64)me32.modBaseAddr,
                        (DWORD64)me32.modBaseAddr + me32.modBaseSize
                    );
                }
    
                char *ptr = nullptr;
                long len = WideCharToMultiByte(CP_ACP, 0, me32.szExePath, -1, NULL, 0, NULL, NULL);
                WideCharToMultiByte(CP_ACP, 0, me32.szExePath, -1, ptr, len + 1, NULL, NULL);
    
                const char *szBaseName = getBaseName(ptr);
    
                DWORD dwVInfo[4];
                if (getModuleVersionInfo(ptr, dwVInfo)) {
                    writeLog("%-12s\t%lu.%lu.%lu.%lu\n", szBaseName, dwVInfo[0], dwVInfo[1], dwVInfo[2],
                            dwVInfo[3]);
                } else {
                    writeLog("%s\n", szBaseName);
                }
            } while (Module32Next(hModuleSnap, &me32));
            writeLog("\n");
        }
    
        CloseHandle(hModuleSnap);
    }
    
    #endif // FORMATOUTPUT_H
	```
- util.h 源代码如下：
	```
	// util.h
	#ifndef UTIL_H
    #define UTIL_H
    
    #include <windows.h>
    #include <imagehlp.h>
    
    const char *getSeparator(const char *szFilename)
    {
        const char *p, *q;
        p = NULL;
        q = szFilename;
        char c;
        do {
            c = *q++;
            if (c == '\\' || c == '/' || c == ':')
            {
                p = q;
            }
        } while (c);
        return p;
    }
    
    const char *getBaseName(const char *szFilename)
    {
        const char *pSeparator = getSeparator(szFilename);
        if (!pSeparator)
        {
            return szFilename;
        }
        return pSeparator;
    }
    
    BOOL GetSymFromAddr(HANDLE hProcess,
                   DWORD64 dwAddress,
                   LPSTR lpSymName,
                   DWORD nSize,
                   LPDWORD lpdwDisplacement)
    {
        PSYMBOL_INFO pSymbol = (PSYMBOL_INFO)malloc(sizeof(SYMBOL_INFO) + nSize * sizeof(char));
    
        DWORD64 dwDisplacement =
            0; // Displacement of the input address, relative to the start of the symbol
        BOOL bRet;
    
        pSymbol->SizeOfStruct = sizeof(SYMBOL_INFO);
        pSymbol->MaxNameLen = nSize;
    
        DWORD dwOptions = SymGetOptions();
    
        bRet = SymFromAddr(hProcess, dwAddress, &dwDisplacement, pSymbol);
    
        if (bRet) {
            // Demangle if not done already
            if ((dwOptions & SYMOPT_UNDNAME) ||
                UnDecorateSymbolName(pSymbol->Name, lpSymName, nSize, UNDNAME_NAME_ONLY) == 0) {
                strncpy(lpSymName, pSymbol->Name, nSize);
            }
            if (lpdwDisplacement) {
                *lpdwDisplacement = dwDisplacement;
            }
        }
    
        free(pSymbol);
    
        return bRet;
    }
    
    BOOL GetLineFromAddr(HANDLE hProcess,
                    DWORD64 dwAddress,
                    LPSTR lpFileName,
                    DWORD nSize,
                    LPDWORD lpLineNumber)
    {
        IMAGEHLP_LINE64 Line;
        DWORD dwDisplacement =
            0; // Displacement of the input address, relative to the start of the symbol
    
        // Do the source and line lookup.
        memset(&Line, 0, sizeof Line);
        Line.SizeOfStruct = sizeof Line;
    
        if (!SymGetLineFromAddr64(hProcess, dwAddress, &dwDisplacement, &Line))
            return FALSE;
    
        assert(lpFileName && lpLineNumber);
    
        strncpy(lpFileName, Line.FileName, nSize);
        *lpLineNumber = Line.LineNumber;
    
        return TRUE;
    }
    
    BOOL getModuleVersionInfo(LPCSTR szModule, DWORD *dwVInfo)
    {
        DWORD dummy, size;
        BOOL success = FALSE;
    
        size = GetFileVersionInfoSizeA(szModule, &dummy);
        if (size > 0) {
            LPVOID pVer = malloc(size);
            ZeroMemory(pVer, size);
            if (GetFileVersionInfoA(szModule, 0, size, pVer)) {
                VS_FIXEDFILEINFO *ffi;
                if (VerQueryValueA(pVer, "\\", (LPVOID *)&ffi, (UINT *)&dummy)) {
                    dwVInfo[0] = ffi->dwFileVersionMS >> 16;
                    dwVInfo[1] = ffi->dwFileVersionMS & 0xFFFF;
                    dwVInfo[2] = ffi->dwFileVersionLS >> 16;
                    dwVInfo[3] = ffi->dwFileVersionLS & 0xFFFF;
                    success = TRUE;
                }
            }
            free(pVer);
        }
        return success;
    }
    
    #ifndef STATUS_FATAL_USER_CALLBACK_EXCEPTION
    #define STATUS_FATAL_USER_CALLBACK_EXCEPTION ((NTSTATUS)0xC000041DL)
    #endif
    #ifndef STATUS_CPP_EH_EXCEPTION
    #define STATUS_CPP_EH_EXCEPTION ((NTSTATUS)0xE06D7363L)
    #endif
    #ifndef STATUS_CLR_EXCEPTION
    #define STATUS_CLR_EXCEPTION ((NTSTATUS)0xE0434f4DL)
    #endif
    #ifndef STATUS_WX86_BREAKPOINT
    #define STATUS_WX86_BREAKPOINT ((NTSTATUS)0x4000001FL)
    #endif
    #ifndef STATUS_POSSIBLE_DEADLOCK
    #define STATUS_POSSIBLE_DEADLOCK ((DWORD)0xC0000194L)
    #endif
    
    LPCSTR getExceptionString(NTSTATUS ExceptionCode)
    {
        switch (ExceptionCode) {
        case EXCEPTION_ACCESS_VIOLATION: // 0xC0000005
            return "Access Violation";
        case EXCEPTION_IN_PAGE_ERROR: // 0xC0000006
            return "In Page Error";
        case EXCEPTION_INVALID_HANDLE: // 0xC0000008
            return "Invalid Handle";
        case EXCEPTION_ILLEGAL_INSTRUCTION: // 0xC000001D
            return "Illegal Instruction";
        case EXCEPTION_NONCONTINUABLE_EXCEPTION: // 0xC0000025
            return "Cannot Continue";
        case EXCEPTION_INVALID_DISPOSITION: // 0xC0000026
            return "Invalid Disposition";
        case EXCEPTION_ARRAY_BOUNDS_EXCEEDED: // 0xC000008C
            return "Array bounds exceeded";
        case EXCEPTION_FLT_DENORMAL_OPERAND: // 0xC000008D
            return "Floating-point denormal operand";
        case EXCEPTION_FLT_DIVIDE_BY_ZERO: // 0xC000008E
            return "Floating-point division by zero";
        case EXCEPTION_FLT_INEXACT_RESULT: // 0xC000008F
            return "Floating-point inexact result";
        case EXCEPTION_FLT_INVALID_OPERATION: // 0xC0000090
            return "Floating-point invalid operation";
        case EXCEPTION_FLT_OVERFLOW: // 0xC0000091
            return "Floating-point overflow";
        case EXCEPTION_FLT_STACK_CHECK: // 0xC0000092
            return "Floating-point stack check";
        case EXCEPTION_FLT_UNDERFLOW: // 0xC0000093
            return "Floating-point underflow";
        case EXCEPTION_INT_DIVIDE_BY_ZERO: // 0xC0000094
            return "Integer division by zero";
        case EXCEPTION_INT_OVERFLOW: // 0xC0000095
            return "Integer overflow";
        case EXCEPTION_PRIV_INSTRUCTION: // 0xC0000096
            return "Privileged instruction";
        case EXCEPTION_STACK_OVERFLOW: // 0xC00000FD
            return "Stack Overflow";
        case EXCEPTION_POSSIBLE_DEADLOCK: // 0xC0000194
            return "Possible deadlock condition";
        case STATUS_FATAL_USER_CALLBACK_EXCEPTION: // 0xC000041D
            return "Fatal User Callback Exception";
        case STATUS_ASSERTION_FAILURE: // 0xC0000420
            return "Assertion failure";
    
        case STATUS_CLR_EXCEPTION: // 0xE0434f4D
            return "CLR exception";
        case STATUS_CPP_EH_EXCEPTION: // 0xE06D7363
            return "C++ exception handling exception";
    
        case EXCEPTION_GUARD_PAGE: // 0x80000001
            return "Guard Page Exception";
        case EXCEPTION_DATATYPE_MISALIGNMENT: // 0x80000002
            return "Alignment Fault";
        case EXCEPTION_BREAKPOINT: // 0x80000003
            return "Breakpoint";
        case EXCEPTION_SINGLE_STEP: // 0x80000004
            return "Single Step";
    
        case STATUS_WX86_BREAKPOINT: // 0x4000001F
            return "Breakpoint";
        case DBG_TERMINATE_THREAD: // 0x40010003
            return "Terminate Thread";
        case DBG_TERMINATE_PROCESS: // 0x40010004
            return "Terminate Process";
        case DBG_CONTROL_C: // 0x40010005
            return "Control+C";
        case DBG_CONTROL_BREAK: // 0x40010008
            return "Control+Break";
        case 0x406D1388:
            return "Thread Name Exception";
    
        case RPC_S_UNKNOWN_IF:
            return "Unknown Interface";
        case RPC_S_SERVER_UNAVAILABLE:
            return "Server Unavailable";
    
        default:
            return NULL;
        }
    }
    
    #endif // UTIL_H
	```
- log.h 源代码如下：
	```
	// log.h
	#ifndef LOG_H
    #define LOG_H
    
    #include <windows.h>
    
    static HANDLE g_handleCrashFile = nullptr;
    
    int writeLog(const char *format, ...)
    {
        char szBuffer[1024];
        int retValue;
        va_list ap;
    
        va_start(ap, format);
        retValue = _vsnprintf(szBuffer, sizeof szBuffer, format, ap);
        va_end(ap);
    
        DWORD cbWritten;
        const char *szText = szBuffer;
        while (*szText != '\0') {
            const char *p = szText;
            while (*p != '\0' && *p != '\n') {
                ++p;
            }
            WriteFile(g_handleCrashFile, szText, p - szText, &cbWritten, 0);
            if (*p == '\n') {
                WriteFile(g_handleCrashFile, "\r\n", 2, &cbWritten, 0);
                ++p;
            }
            szText = p;
        };
    
        return retValue;
    }
    
    #endif // LOG_H
	```
- dllexport.h 源代码如下：
	```
	// dllexport.h
	#ifndef HEADERDEF_H
    #define HEADERDEF_H
    
    #ifdef __cplusplus
    #define D_EXTERN_C extern "C"
    #else
    #define D_EXTERN_C
    #endif
    
    #ifdef __SHARE_EXPORT
    #define D_SHARE_EXPORT D_DECL_EXPORT
    #else
    #define D_SHARE_EXPORT D_DECL_IMPORT
    #endif
    
    #if defined(WIN32) || defined(_WIN32) || defined(__WIN32) || defined(__WIN32__)
    #define D_CALLTYPE __stdcall
    #define D_DECL_EXPORT __declspec(dllexport)
    #define D_DECL_IMPORT
    #else
    #define D_CALLTYPE
    #define D_DECL_EXPORT __attribute__((visibility("default")))
    #define D_DECL_IMPORT __attribute__((visibility("default")))
    #endif
    
    D_EXTERN_C D_SHARE_EXPORT void D_CALLTYPE ExcHndlInit(const char *pCrashFileName);
    
    #endif // HEADERDEF_H
	```


# 测试工具 DrMingwDemo
- 工程文件 DrMingwDemo.pro 源代码如下：
    ```
    QT += core
    QT -= gui
    
    CONFIG += c++11
    
    TARGET = DrMingwDemo
    CONFIG += console
    CONFIG -= app_bundle
    
    TEMPLATE = app
    
    DESTDIR = $$PWD/bin
    
    LIBS += -L$$PWD/lib -lMiniDrMingw
    
    INCLUDEPATH += $$PWD/lib/include
    
    # gcc 编译选项，生成 .map 文件
    QMAKE_LFLAGS += -Wl,-Map=$$PWD'/bin/'$$TARGET'.map'
    
    SOURCES += main.cpp
    
    QMAKE_CXXFLAGS_RELEASE = $$QMAKE_CFLAGS_RELEASE_WITH_DEBUGINFO
    QMAKE_LFLAGS_RELEASE = $$QMAKE_LFLAGS_RELEASE_WITH_DEBUGINFO
    
    HEADERS +=
    ```
- main.cpp 源代码如下：
    ```
    #include <QCoreApplication>
    #include <QThread>
    
    extern "C" void ExcHndlInit(const char *pCrashFileName);
    
    class TestClass : public QThread
    {
    public:
        void run() override
        {
            int* a = (int*)(NULL);
            *a = 1;
        }
    };
    
    int main(int argc, char *argv[])
    {
        QCoreApplication a(argc, argv);
    
        ExcHndlInit((a.applicationDirPath() +"/" + a.applicationName() + ".RPT").toLocal8Bit().data());
    
        // 制造崩溃
        TestClass instance;
        instance.start();
    
        return a.exec();
    }
    ```

# 崩溃分析
- DrMingwDemo.exe 编译后，会在工程的 bin 目录下生成 DrMingwDemo.map。
- 双击运行 DrMingwDemo.exe，然后闪退，此时在程序的运行目录，产生了 DrMingwDemo.RPT。
- 打开 DrMingwDemo.RPT 文件，内容大概如下：
    ```
    -------------------

    Error occurred on Friday, September 9, 2022 at 20:54:00.
    
    DrMingwDemo.exe caused an Access Violation at location 004028A0 in module DrMingwDemo.exe Writing to location 00000000.
    
    Registers:
    eax=004042f4 ebx=0065fe78 ecx=0065fe78 edx=00000001 esi=0108b4a0 edi=0108b468
    eip=004028a0 esp=02c1fefc ebp=02c1ff28 iopl=0         nv up ei pl nz na po nc
    cs=0023  ss=002b  ds=002b  es=002b  fs=0053  gs=002b             efl=00010206
    
    AddrPC   Params
    004028A0 7606689C 76066CFF 0065FE78  DrMingwDemo.exe!0x28a0
    76066CFF 02C1FF80 769DFA29 0108C610  msvcrt.dll!_beginthreadex+0xcf
    76066DC1 0108C610 769DFA10 02C1FFDC  msvcrt.dll!_endthreadex+0x91
    769DFA29 0108C610 4F76EDC1 00000000  KERNEL32.DLL!BaseThreadInitThunk+0x19
    771A7A9E FFFFFFFF 771C8BAC 00000000  ntdll.dll!RtlGetAppContainerNamedObjectPath+0x11e
    771A7A6E 76066D60 0108C610 00000000  ntdll.dll!RtlGetAppContainerNamedObjectPath+0xee
    
    00400000-00451000 
    ```
- 由上面可以可以看出是 004028A0 这个地址出现了崩溃。此时，我们打开 DrMingwDemo.map 文件，找到离该地址最近的函数（一般函数地址会小于等于崩溃地址）。
- DrMingwDemo.map 的部分内容如下：
    ```
    ...
     *(SORT(.text$*))
     *fill*         0x00402818        0x8 
     .text$_ZN10QByteArrayD1Ev
                    0x00402820       0x40 release/main.o
                    0x00402820                QByteArray::~QByteArray()
     .text$_ZN7QStringD1Ev
                    0x00402860       0x40 release/main.o
                    0x00402860                QString::~QString()
     .text$_ZN9TestClass3runEv
                    0x004028a0       0x10 release/main.o
                    0x004028a0                TestClass::run()
     .text$_ZN9TestClassD0Ev
                    0x004028b0       0x20 release/main.o
                    0x004028b0                TestClass::~TestClass()
     .text$_ZN9TestClassD1Ev
                    0x004028d0       0x10 release/main.o
                    0x004028d0                TestClass::~TestClass()
     .text$_ZplRK7QStringPKc
                    0x004028e0       0xc0 release/main.o
                    0x004028e0                operator+(QString const&, char const*)
    ...
    ```
- 因此，可以看出是 TestClass::run() 函数发生了崩溃。
- 注意，该方法只能定位出哪个函数出现了崩溃，却难以定位出是哪行导致的崩溃，就算在项目工程中加入了 Debug 信息也如此。


# 广告时间：我的 GitHub (希望给个star)

> [MiniDrMingw](https://github.com/L-ge/MiniDrMingw)
