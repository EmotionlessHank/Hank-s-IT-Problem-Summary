## C语言 VSCode 在 mac 上的配置
* 最近我的mac更新了新的系统 Catalina 之前的Clion 的配置全不能用了，研究了2天，无果，果断转战VSCode（Assigment 快要due了）。
### 开始前的准备
* XCode
* VSCode
* 进入VSCode 里面 install各种插件: C/C++, CodeLLDB, C/C++ Clang Command Adapter
### 开始配置
1. 在命令行执行 
$ xcode-select --install  
2. 进入VSCode, 使用快捷键 “command+shift+p” 打开命令行工具窗口， 输入 edit 找到“ C/Cpp: Edit Configurations...”,  
点击，此时会在当前工作空间目录生成.vscode配置目录，同时在配置目录会生成一个c_cpp_properties.json文件。
配置include目录:(下面是我的个人设置，在网上找的别人的配置，自己进行了修改，其中clang文件夹要根据自己的版本填写)

```
{
    "configurations": [
        {
            "name": "Mac",
            "includePath": [
                "${workspaceFolder}/**",
                "/Library/Developer/CommandLineTools/usr/include/c++/v1",
                "/usr/local/include",
                "/Library/Developer/CommandLineTools/usr/lib/clang/11.0.0/include",
                "/Library/Developer/CommandLineTools/usr/include",
                "/usr/include"
            ],
            "defines": [],
            "macFrameworkPath": [
                "/System/Library/Frameworks",
                "/Library/Frameworks"
            ],
            "compilerPath": "/usr/bin/clang",
            "cStandard": "c11",
            "cppStandard": "c++17",
            "intelliSenseMode": "${default}"
        }
    ],
    "version": 4
}
```

3. 继续在命令行工具窗口，输入或者选择“Tasks: Configure Task”，进行配置 task.json   
```
{
    "version": "2.0.0",
    "tasks": [{
        "label": "Compile", // 任务名称，与launch.json的preLaunchTask相对应
        "command": "clang",   // 要使用的编译器，C++用g++
        "args": [
            //"-g",    // 生成和调试有关的信息
            "${file}",
            "-std=c11", 
            "-stdlib=libc++",
            "-o",    // 指定输出文件名，不加该参数则默认输出a.exe，Linux下默认a.out
            "${fileDirname}/${fileBasenameNoExtension}.out",
            "--debug"
            
            //"-Wall", // 开启额外警告
            //"-static-libgcc",     // 静态链接libgcc，一般都会加上
            //"-fexec-charset=GBK", // 生成的程序使用GBK编码，不加这一条会导致Win下输出中文乱码
            // C++最新标准为c++17，或根据自己的需要进行修改
        ], // 编译的命令，其实相当于VSC帮你在终端中输了这些东西
        "type": "shell", // process是vsc把预定义变量和转义解析后直接全部传给command；shell相当于先打开shell再输入命令，所以args还会经过shell再解析一遍
        "group": {
            "kind": "build",
            "isDefault": true // 不为true时ctrl shift B就要手动选择了
        },
        "presentation": {
            "echo": true,  
            "reveal": "always", // 执行任务时是否跳转到终端面板，可以为always，silent，never。具体参见VSC的文档
            "focus": false,     // 设为true后可以使执行task时焦点聚集在终端，但对编译C/C++来说，设为true没有意义
            "panel": "shared"   // 不同的文件的编译信息共享一个终端面板
        },
        // "problemMatcher":"$gcc" // 此选项可以捕捉编译时终端里的报错信息；但因为有Lint，再开这个可能有双重报错
    }]
}       
```

4. 配置launch.json  
打开命令行工具窗口，输入或者选择Debug: Open launch.json命令。  
```
// https://github.com/Microsoft/vscode-cpptools/blob/master/launch.md
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "c Launch", // 配置名称，将会在启动配置的下拉菜单中显示
            "type": "lldb", // 配置类型，cppdbg对应cpptools提供的调试功能；可以认为此处只能是cppdbg
            "request": "launch", // 请求配置类型，可以为launch（启动）或attach（附加）
            "program": "${fileDirname}/${fileBasenameNoExtension}.out", // 将要进行调试的程序的路径
            //"program": "${workspaceFolder}/${fileBasenameNoExtension}.out",
            "args": [
                "-arg1",
                "-arg2"
            ], // 程序调试时传递给程序的命令行参数，一般设为空即可
            "stopAtEntry": false, // 设为true时程序将暂停在程序入口处，相当于在main上打断点
            "cwd": "${workspaceFolder}", // 调试程序时的工作目录，此为工作区文件夹；改成${fileDirname}可变为文件所在目录
            "environment": [], // 环境变量
            "externalConsole": true, // 为true时使用单独的cmd窗口，与其它IDE一致；18年10月后设为false可调用VSC内置终端
            "internalConsoleOptions": "neverOpen", // 如果不设为neverOpen，调试时会跳到“调试控制台”选项卡，你应该不需要对gdb手动输命令吧？
            "MIMode": "lldb", // 指定连接的调试器，可以为gdb或lldb。但我没试过lldb
            //"miDebuggerPath": "gdb.exe", // 调试器路径，Windows下后缀不能省略，Linux下则不要
            /*"setupCommands": [
            { // 模板自带，好像可以更好地显示STL容器的内容，具体作用自行Google
                "description": "Enable pretty-printing for gdb",
                "text": "-enable-pretty-printing",
                "ignoreFailures": false
            }
        ],*/
            "preLaunchTask": "Compile", // 调试会话开始前执行的任务，一般为编译程序。与tasks.json的label相对应
        }
    ]
}
```

5. 打开工作文件夹，使用快捷键“command + shift + .” 显示出被隐藏的".vscode"文件夹  
进入文件夹，新建"setting.json"并编辑
```
{
    "files.defaultLanguage": "c", 
    "editor.formatOnType": true,  
    "editor.suggest.snippetsPreventQuickSuggestions": false, 
    "editor.acceptSuggestionOnEnter": "off", 
    

    "code-runner.runInTerminal": true, 
    "code-runner.preserveFocus": true,     
    "code-runner.clearPreviousOutput": false, 
    "code-runner.ignoreSelection": true,   

    "C_Cpp.clang_format_sortIncludes": true
}
```

6. 开始调试
点击编辑器的debug按钮，绿色的启动按钮就被激活啦！！！
7. 参考资料(感谢大佬写的文章)
* https://zhuanlan.zhihu.com/p/48233069
无废话--Mac OS, VS Code 搭建c/c++基本开发环境
* https://blog.csdn.net/fujz123/article/details/104426121
解决Mac上VSCdoe断点失效问题
* https://blog.csdn.net/Tuine/article/details/87858745
Mac OS找不到/usr/include文件夹的解决办法



