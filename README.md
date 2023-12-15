# F8ExcelTool

[![license](http://img.shields.io/badge/license-MIT-green.svg)](https://opensource.org/licenses/MIT) [![Unity Version](https://img.shields.io/badge/unity-2021.3.15f1-blue)](https://unity.com) [![Platform](https://img.shields.io/badge/platform-Win%20%7C%20Android%20%7C%20iOS%20%7C%20Mac%20%7C%20Linux-orange)]()

## 简介（希望自己点击 F8，就能开始制作游戏，不想多余的事）
[README.md](README.md)
Unity 读取 Excel 的工具。

点击 F8 自动生成 C# 索引字段，随后：

支持的使用方式：

- **加载缓存**：高性能，加载手动生成的 Excel 二进制缓存

- **加载文件**：高适应性，运行时自动读取最新 Excel，无需人工干预

  - **必须**先完成本指南的[初始化](#初始化)部分（因：对于结构、类型等变动，运行时无法刷新 C# 代码）。

系统支持：Win/Android/iOS/Mac/Linux

## 使用步骤

### 导入

在 Unity 包管理器中，使用 Git URL 添加 F8 核心包、Excel 包。

1. 打开 Unity Editor

2. 点击菜单的 **Window** 项，再点击 **Package Manager** 子项

3. 点击左上角 **+** 号，选择 **Add Package from git URL**

4. 输入 <https://github.com/TippingGame/F8Framework.git>，请确认导入成功

5. 重复 4. 并输入 <https://github.com/TippingGame/F8ExcelTool.git>，请确认导入成功

#### 网络问题

若您的网络不佳，也可以下载核心包、Excel 包文件，直接修改 package.json 并放入 Packages 文件夹。

### 初始化

1. 在 Asserts 下，创建 StreamingAssets/config 目录，并创建你的 Excel

    1. Excel 每个 Sheet **务必**有一个 id 列，我们将使用它作为唯一索引列

2. 点击菜单的**开发工具**项 -> **导入配置表**\_F8（快捷键），以此生成二进制缓存

    1. 如果导入失败，Console 会输出对应的错误信息

3. 导入成功后，根目录下会生成 **F8ExcelTool** 目录和相关文件

4. （可选项）通过 Editor，要求在运行时读取 Excel 数据：点击菜单的**开发工具**项 -> **运行时读取 Excel**\_F7（快捷键）

### 使用范例

**在使用 Excel 数据前，需要执行**：

加载缓存的方式：

```C#
F8DataManager.Instance.LoadAll(); // 加载手动生成的 Excel 缓存
```

加载文件的方式：

```C#
ReadExcel.Instance.LoadAllExcelData(); // 运行时加载 Excel 最新文件
```

**打印数据**：

值类型，譬如 `int/float/string`，请参考[C# 类型系统 - Microsoft Document](https://learn.microsoft.com/zh-cn/dotnet/csharp/fundamentals/types/#value-types)：

```C#
        // 注意：GetTableByID 方法为自动生成的。
        // 注意：Table 需替换为实际 Sheet 名
        // 注意：name 需替换为实际表头
        // 注意：115 代表您设置的 ID 115 的行
        Debug.Log(F8DataManager.Instance.GetTableByID(115).name);
```

引用类型：

```C#
        foreach (var VARIABLE in F8DataManager.Instance.GetTableByID(113).llliststr)
        {
            foreach (var VARIABLE2 in VARIABLE)
            {
                Debug.Log(VARIABLE2);
            }
        }
```

## Excel 示例

值类型 Excel 示例：

| int | long       | float    | double      | str             |
| --- | ---------- | -------- | ----------- | --------------- |
| id  | name1      | name2    | name3       | name4           |
| 1   | 9935434343 | 2.725412 | 1.346655321 | 读取 Excel 工具 |
| 2   | 9935434343 | 2.725412 | 1.346655321 | 读取 Excel 工具 |

支持的引用类型 Excel 示例：
（更多引用类型譬如 `List/Dict` 等，不确定是否支持）

| int\[] | float\[]   | str\[]       | obj\[]                 | int\[]\[]       | float\[]\[]                   | str\[]\[]                       |
| ------ | ---------- | ------------ | ---------------------- | --------------- | ----------------------------- | ------------------------------- |
| name1  | name2      | name3        | name4                  | name5           | name6                         | name7                           |
| \[1,5] | \[1.5,5.8] | \[文件,支持] | \["生成",656,1.235999] | {\[1,6],\[2,8]} | {\[6.215,6.12],\[2.5,14.556]} | {\[自动,格式],\[tipping,excel]} |
| \[1,5] | \[1.5,5.8] | \[文件,支持] | \["生成",656,1.235999] | {\[1,6],\[2,8]} | {\[6.215,6.12],\[2.5,14.556]} | {\[自动,格式],\[tipping,excel]} |

## 目录结构

\---F8ExcelTool\
\----Editor（编辑器代码）\
\----Plugins（库文件）\
\----Runtime（运行时代码）\
\----Tests（测试场景 Demo）

## 使用到的库

Excel.dll\
I18N.CJK.dll\
I18N.dll\
I18N.MidEast.dll\
I18N.Other.dll\
I18N.Rare.dll\
I18N.West.dll\
ICSharpCode.SharpZipLib.dll

## 注意

### Android

加载文件模式在热加载插件(譬如 [HybridCLR](https://github.com/focus-creative-games/hybridclr))时，可能需要额外操作：

**由于 Android 资源都在包内，在 Android 上使用，需要先复制到可读写文件夹中再进行读取**

```C#
    IEnumerator Start()
    {
        //由于安卓资源都在包内，需要先复制到可读写文件夹1
        string assetPath = URLSetting.STREAMINGASSETS_URL + "config";
        string[] paths = null;
        WWW www = new WWW(assetPath + "/fileindex.txt");
        yield return www;
        if (www.error != null)
        {
            Debug.Log(www.error);
            yield return null;
        }
        else
        {
            string ss = www.text;
            paths = ss.Split("\n", StringSplitOptions.RemoveEmptyEntries);
        }

        for (int i = 0; i < paths.Length; i++)
        {
            yield return CopyAssets(paths[i].Replace("\r", ""));
        }
        //读取Excel文件
        ReadExcel.Instance.LoadAllExcelData();
        text.text += F8DataManager.Instance.GetfasdffByID(115).name;
        Debug.Log(F8DataManager.Instance.GetfasdffByID(115).name);
        foreach (var VARIABLE in F8DataManager.Instance.GetfasdffByID(113).llliststr)
        {
            foreach (var VARIABLE2 in VARIABLE)
            {
                text.text += VARIABLE2;
                Debug.Log(VARIABLE2);
            }
        }
    }
    //由于安卓资源都在包内，需要先复制到可读写文件夹2
    IEnumerator CopyAssets(string paths)
    {
        string assetPath = URLSetting.STREAMINGASSETS_URL + "config";
        string sdCardPath = Application.persistentDataPath + "/config";
        WWW www = new WWW(assetPath + "/" + paths);
        yield return www;
        if(www.error != null)
        {
            Debug.Log(www.error);
            yield return null;
        }
        else
        {
            FileTools.SafeWriteAllBytes(sdCardPath + "/" + paths, www.bytes);
        }
    }
```
