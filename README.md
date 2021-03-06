# CPP-automatic-update-options / Debug a memory allocation

## Inserting "#pragma once" preprocessor directive in the multiple files

"#pragma once" is a non-standard but widely supported preprocessor directive designed to cause the current source file to be included only once in a single compilation.

Using "#pragma once" allows the C preprocessor to include a header file when it is needed and to ignore an #include directive otherwise. 

Some projects do not use this directive and this is not correct.

Adding #pragma once of multiple headers *.h files with Notepad++ 

		Find :  \A^.*?
		Replace with : #pragma once\n\r
		Files: *.h
		Search Mode: Regular Expression

![](./images/notepad_insert_pragma_once.png)

## Find memory leaks in multiple CPP files

### Enable memory leak detection 


The preceding techniques identify memory leaks for memory allocated using the standard CRT malloc function. If your program allocates memory using the C++ new operator, however, you may only see the filename and line number where operator new calls _malloc_dbg in the memory-leak report.

Inserting redifinition into *.cpp files

		Find :  \A^.*?
		Replace with : #ifdef _DEBUG\n\r#define new DBG_NEW\n\r#endif\n\r
		Files: *.cpp
		Search Mode: Regular Expression


		
Inserting redifinition for all *.cpp files

		Find :  \A^.*?
		Replace with : #ifdef _DEBUG\n\r#define DBG_NEW new ( _NORMAL_BLOCK , __FILE__ , __LINE__ )\n\r#else\n\r#define DBG_NEW new\n\r#endif\n\r
		Files: *.cpp
		Search Mode: Regular Expression

![](./images/new_redifinition.png)

and then:

To enable all the debug heap functions, include the following statements in your C++ program

		#define _CRTDBG_MAP_ALLOC
		#include <stdlib.h>
		#include <crtdbg.h>

Adding heap malloc detection function of multiple *.cpp files with Notepad++ 

		Find :  \A^.*?
		Replace with : #define _CRTDBG_MAP_ALLOC\n\r#include <stdlib.h>\n\r#include <crtdbg.h>\n\r
		Files: *.cpp
		Search Mode: Regular Expression 

![](./images/enable_memory_leek_detection.png)

By results you see in all cpp files

		#define _CRTDBG_MAP_ALLOC
		#include <cstdlib>
		#include <crtdbg.h>

		#ifdef _DEBUG
			#define DBG_NEW new ( _NORMAL_BLOCK , __FILE__ , __LINE__ )
			// Replace _NORMAL_BLOCK with _CLIENT_BLOCK if you want the
			// allocations to be of _CLIENT_BLOCK type
		#else
			#define DBG_NEW new
		#endif

		#ifdef _DEBUG
			#define new DBG_NEW
		#endif

By manual setting

		#ifdef _DEBUG
		#undef DEBUG_NEW
		#define DEBUG_NEW new(__FILE__, __LINE__)
		#define _CRTDBG_MAP_ALLOC
		#define new DEBUG_NEW
		#undef THIS_FILE
		static char BASED_CODE THIS_FILE[] = __FILE__;
		#else
		#undef _CRTDBG_MAP_ALLOC
		#define DEBUG_NEW new
		#endif
		
When you run this code in the Visual Studio debugger, the call to _CrtDumpMemoryLeaks generates a report in the Output window.		

For example :

			void main() {
			    .......
			    ,,,,,,, 
			    _CrtDumpMemoryLeaks();
			}
			
## Get filename that does not contain the specified string	
	
Visual Studio 2019 > Tools > Command Line > Developer PowerShell		
		
ls ./\*/\*.cpp | Where{-not(select-string -path $_ -pattern "_CRTDBG_MAP_ALLOC" -Quiet)} | Select FullName				
			
## Find memory leaks into the custom DLL

To determine whether a memory leak has occurred in a section of code, you can take snapshots of the memory state before and after the section, and then use _ CrtMemDifference to compare the two states:

For example :


				_CrtMemState s1.s2,s3;


				BOOL APIENTRY DllMain( HMODULE hModule,
					   DWORD  ul_reason_for_call,
					   LPVOID lpReserved
				 )
				{
					switch (ul_reason_for_call)
					{
					case DLL_PROCESS_ATTACH:
					#ifdef _DEBUG	
						_CrtMemCheckpoint( &s1 );
					#endif	
					break;
					case DLL_THREAD_ATTACH:
					break;
					case DLL_THREAD_DETACH:
					break;
					case DLL_PROCESS_DETACH:
					#ifdef _DEBUG	
					   	 _CrtMemCheckpoint( &s2 );
							if ( _CrtMemDifference( &s3, &s1, &s2) 
 					  			_CrtMemDumpStatistics( &s3 );
					#endif					   
					break;
					}
					return TRUE;
				}
				}

## Use your own DllMain in MFC Dll's

If you try to write your own DllMain, you get a linker error saying "DllMain already defined". 

To resolve this issue :

1) copy "dllmodul.cpp" from MFC source directory to your project directory and include it in your own project.

2 Modify dllmodul.cpp:

				CATCH(CMemoryException, e)
				{
					e->Delete();
				//	DELETE_EXCEPTION(e);
					pApp->ExitInstance();
					AfxWinTerm();
					goto Cleanup;       // Init Failed
				}

## Adding breakpoint on memory leak detection items

1) If your app defines _CRTDBG_MAP_ALLOC, the memory-leak report looks like:

![](./images/memory_leak.png)

2) Start your debugging session:

![](./images/memory_leak_stop.png)

3) Type _crtBreakAlloc in the Watch window. 
{,,MSVCR160d.dll}*__p__crtBreakAlloc()

Where MSVCR: C Runtime Library (CRL)

Number at the end of DLL file corresponds to the Visual Studio version number.

			90: Visual Studio 2008 (Version 9.0)
			100: Visual Studio 2010 (Version 10.0)
			110: Visual Studio 2012 (Version 11.0)
			120: Visual Studio 2013 (Version 12.0)
			140: Visual Studio 2015 (Version 14.0)
			150: Visual Studio 2017 (Version 15.0)
			160: Visual Studio 2019 (Version 16.0)


![](./images/memory_leak_watch.png)

4) Double click on the -1 value, and enter the new allocation number that causes a user-defined breakpoint. After insert you see -1 ( current position )

5) After you set a breakpoint on a memory-allocation number, continue to debug. Make sure to run under the same conditions, so the memory-allocation number doesn't change.
