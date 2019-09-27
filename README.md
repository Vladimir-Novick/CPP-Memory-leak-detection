# CPP-automatic-update-options

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

## Find memory leaks in multiple the DLL library

### Enable memory leak detection 

To enable all the debug heap functions, include the following statements in your C++ program

Adding heap malloc detection function of multiple *.cpp files with Notepad++ 

		Find :  \A^.*?
		Replace with : #define _CRTDBG_MAP_ALLOC\n\r#include <stdlib.h>\n\r#include <crtdbg.h>\n\r
		Files: *.cpp
		Search Mode: Regular Expression 

![](./images/enable_memory_leek_detection.png)

The preceding techniques identify memory leaks for memory allocated using the standard CRT malloc function. If your program allocates memory using the C++ new operator, however, you may only see the filename and line number where operator new calls _malloc_dbg in the memory-leak report.

Inserting redifinition into *.h files

		Find :  \A^.*?
		Replace with : #ifdef _DEBUG\n\r#define new DBG_NEW\n\r#endif\n\r
		Files: *.h
		Search Mode: Regular Expression



Inserting redifinition for all *.h files

		Find :  \A^.*?
		Replace with : #ifdef _DEBUG\n\r#define DBG_NEW new ( _NORMAL_BLOCK , __FILE__ , __LINE__ )\n\r#else\n\r#define DBG_NEW new\n\r#endif\n\r
		Files: *.h
		Search Mode: Regular Expression

![](./images/new_redifinition.png)


When you run this code in the Visual Studio debugger, the call to _CrtDumpMemoryLeaks generates a report in the Output window.		

For example :

				BOOL APIENTRY DllMain( HMODULE hModule,
									   DWORD  ul_reason_for_call,
									   LPVOID lpReserved
									 )
				{
					switch (ul_reason_for_call)
					{
					case DLL_PROCESS_ATTACH:
					case DLL_THREAD_ATTACH:
					case DLL_THREAD_DETACH:
					case DLL_PROCESS_DETACH:
					   _CrtDumpMemoryLeaks();
						break;
					}
					return TRUE;
				}


