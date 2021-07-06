---
title: 常用调试代码
date: 2020-09-13
---
## powershell策略改变
管理员权限
set-executionpolicy remotesigned

## .Net的DLL载入
手动反射加载Dll中的函数

Assembly.Load()

Assembly.LoadFile()

Assembly.LoadFrom()

### 示例代码
三者代码,调用dll中的方法几乎一致

Load一般针对字符串

LoadFile,LoadFrom加载程序,LoadFile加载单文件,LoadFrom会自动加载依赖的程序集.

```C#
string assemblyAndPathName = @"path";
            Assembly testAsm = Assembly.LoadFile(assemblyAndPathName);
            Type itype = testAsm.GetType("assemblyName.ClassName");//程序集名.类名
            object obj = System.Activator.CreateInstance(itype);
            MethodInfo method1 = itype.GetMethod("MethodName");
            method1.Invoke(obj, new object[]{
                "a","b"
            });//参数.
```

## 根据16进制字符串计算GUID
```C#
using System;

namespace ConsoleApp3
{
    class Program
    {
        static void Main()
        {
            // https://stackoverflow.com/questions/56638890/c-sharp-convert-active-directory-hexadecimal-to-guid
            byte[] bytearray = StringToByteArray("11F890453A1DD011891F00AA004B2E24");

            // https://docs.microsoft.com/en-us/dotnet/api/system.guid.tobytearray?view=netframework-4.8
            Guid guid = new Guid(bytearray);
            Console.WriteLine("Guid: {0}", guid);
            Byte[] bytes = guid.ToByteArray();
            foreach (var byt in bytes)
                Console.Write("{0:X2} ", byt);

            Console.WriteLine();
            Guid guid2 = new Guid(bytes);
            Console.WriteLine("Guid: {0} (Same as First Guid: {1})", guid2, guid2.Equals(guid));
            Console.ReadLine();

        }

        public static byte[] StringToByteArray(String hex)
        {
            int NumberChars = hex.Length;
            byte[] bytes = new byte[NumberChars / 2];
            for (int i = 0; i < NumberChars; i += 2)
                bytes[i / 2] = Convert.ToByte(hex.Substring(i, 2), 16);
            return bytes;
        }
    }
}
```

## 递归下载网页文件
wget -c -r -p -k -np url
