# Example - PEB

Get PEB by reading the `gs` (x64) or `fs` (x86) register.

## Create Visual Studio Project

Follow the instructions [here](VisualStudioProject.md)

# Program.cs


```C#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;


using SharpASM = SharpAssembly.SharpASM;

namespace UsageExample
{
    class Program
    {
        static void Main(string[] args)
        {
            IntPtr ptr = PEB.GetPEB();
            Console.WriteLine("Peb @ 0x" + string.Format("{0:X}", ptr.ToInt64()));
            Console.ReadLine();
        }
    }
            

    class PEB
    {

        static bool is64BitProcess = (IntPtr.Size == 8);

        // PEB ptr - We will search PEB's address just once
        static IntPtr peb;

        // __readgsqword(0x60)
        static byte[] bReadgsqword =
        {
            0x65, 0x48, 0x8B, 0x04, 0x25, 0x60,     // mov rax, qword ptr gs:[0x60]
            0x00, 0x00, 0x00,
            0xc3                                    // ret
        };


        // __readfsdword(0x30)
        static byte[] bReadfsdword =
        {
            //0x55, // push ebp             
            //0x8B, 0xEC, // mov ebp,esp
            0x64, 0xA1, 0x30, 0x00, 0x00, 0x00, // mov eax,dword ptr fs:[30]                          
            //0x5D, // pop ebp      
            0xC3 // ret
        };

        public static IntPtr GetPEB()
        {
            if (peb.Equals(IntPtr.Zero))
            {
#if WIN64
                    peb = SharpASM.callASM(bReadgsqword);
#else

                    peb = SharpASM.callASM(bReadfsdword);
#endif

            }
            return peb;
        }
    }
}
```