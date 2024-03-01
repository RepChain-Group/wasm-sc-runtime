# wasm-sc-runtime
wasm-sc-runtime提供了支持智能合约专用指令集的智能合约执行引擎，在支持解析执行标准WebAssembly格式的智能合约代码的同时，还支持解析运行包含智能合约专用指令序列的智能合约代码。

本软件包通过动态链接库/共享链接库的方式提供了C语言形式的API接口，开发者可使用各种流行编程语言（如C/C++, Java, Scala, Go, Python, Rust等）基于相应API接口集成使用智能合约引擎。

>Note： *本软件包当前版本只支持Linux AMD64/x86_64平台*

## 软件包目录结构
```bash
include  // 头文件目录
    wasm-sc-runtime.h
lib      // 链接库目录
    libwasm-sc-runtime.so
examples // 使用示例目录，包含示例代码
    ......
README.md
```
## 下载
下载对应的[预编译软件包]()

## 安装/Installation(非必须)
若不想在编译代码时添加额外的链接库及头文件目录参数,可以将链接库文件`lib/libwasm-sc-runtime.so`复制到链接器(linker)能找到的默认目录下，如Linux系统下的`/usr/local/lib`或`/usr/lib`等，可将头文件`include/wasm-sc-runtime.h`复制到编译器(compiler)能找到的默认目录下，如Linux系统下的`/usr/local/include`或`usr/include`

也可忽略上述操作，在编译自己的代码时添加相应编译参数，或者基于相应编程语言集成链接库的相关工具和方法直接加载使用链接库和头文件。

## 使用/Usage 
本软件包以链接库形式对外提供合约执行引擎的相应功能，以方便不同编程语言开发环境的集成，这里以开发者基于C语言来集成开发为例进行说明。

若已经执行上述安装操作，可运行编译命令(如使用gcc编译器)： 
```bash
// 编译源码生成可执行程序
$ gcc -o your_terget_name your_source_code.c -lwasm-sc-runtime
// 运行编译结果
$ ./your_target_name
```
若没有执行上部分的安装操作，可能需要在编译时添加相应参数，如当在本软件包根目录下编译时：
```bash
// 编译源码生成可执行程序
$ gcc -o your_terget_name your_source_code.c -lwasm-sc-runtime -I./include -L./lib -Wl,-rpath,./lib
// 运行编译结果
$ ./your_target_name
```

### API Reference
主要功能及其工作流程为：
1. 创建智能合约执行引擎runtime实例 -> 
2. 创建智能合约虚拟机vm实例 -> 
3. 调用智能合约方法。

其中，基于同一个智能合约引擎实例可以创建若干智能合约虚拟机实例。

wasm-sc-runtime主要提供了如下所示的C语言形式APIs：
- wasm_sc_runtime *smart_contract_runtime_new()
  
  创建智能合约执行引擎实例。
- wasm_sc_vm *smart_contract_vm_new(wasm_sc_runtime *runtime, BCEI *bcei, const char *vm_id, int vm_id_len, const char *wat_file_path, int wat_file_path_len);

  根据智能合约引擎实例创建智能合约虚拟机实例。所需参数包括：
  - wasm_sc_runtime *runtime: 智能合约引擎实例指针
  - BCEI *bcei: 区块链平台环境接口实例指针，该接口实例需要开发者自行实现，其中所有接口方法的第一个参数为合约方法调用上下文实例，开发者需结合该参数实现相应接口的具体逻辑
  - const char *vm_id: 自定义的虚拟机实例唯一标识字符串指针
  - int vm_id_len: 虚拟机实例唯一标识字符串长度 
  - const char *wat_file_path: 表示对应的智能合约代码文件路径的字符串指针，目前该文件内容需为文本格式
  - int wat_file_path_len: 表示智能合约代码文件路径的字符串的长度

- func_result *smart_contract_function_call(wasm_sc_runtime *runtime, wasm_sc_vm *vm, void *ctx, const char *call_id, int call_id_len, const char *func_name, int func_name_len, unsigned char *args[], int args_len[], int arg_count);

   调用具体的智能合约方法。所需参数包括：
   - wasm_sc_runtime *runtime: 智能合约引擎实例指针 
   - wasm_sc_vm *vm: 智能合约虚拟机实例指针
   - void *ctx: 合约方法调用上下文实例指针，该上下文实例为开发者自定义的数据结构实例，该上下文实例数据将被传递给区块链平台环境接口实例中的每一个接口方法
   - const char *call_id: 调用标识字符串指针, 即是给本次调用合约方法的行为赋予一个唯一标识，比如可以将引起本次调用合约方法的区块链交易txid作为该值
   - int call_id_len: 调用标识的字符串长度
   - const char *func_name: 要调用的合约方法名字符串指针
   - int func_name_len: 合约方法名字符串长度
   - unsigned char *args[]: 由若干字节数组表示的合约方法参数, 即每一个合约方法参数用一个字节数组来表示，如args[0]是表示第一个参数的字节数组，args[1]是表示第二个参数的字节数组
   - int args_len[]: 合约方法参数的长度数组，该数组中的每一个元素表示对应index的方法参数的长度值,如args_len[0]表示参数args[0]所占的字节长度
   - int arg_count: 合约方法参数的数量

- void smart_contract_func_result_delete(func_result *result)

   释放合约方法调用结果数据所占内存。

- void smart_contract_vm_delete(wasm_sc_runtime *runtime, const wasm_sc_vm *vm);

   释放智能合约虚拟机实例所占内存。

- void smart_contract_runtime_delete(wasm_sc_runtime *runtime)

   释放智能合约引擎实例所占内存。

可进一步参考头文件`include/wasm-sc-runtime.h`中的相关内容。

### 示例
本软件包目录下的子目录`examples`中提供了使用wasm-sc-runtime的代码示例，可参考相应代码内容。其中每个示例包括作为智能合约源码的`_<name>.wat`文件以及集成智能合约引擎运行该智能合约代码的程序源码文件`<name>.c`，可参考相应程序源码文件中顶部的注释内容编译运行该示例程序。
