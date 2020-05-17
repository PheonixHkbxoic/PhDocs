

## node命令参数



-   [`-c`, `--check`](http://nodejs.cn/api/cli.html#cli_c_check) 不执行脚本，只是做语法检查

-   [`-e`, `--eval "script"`](http://nodejs.cn/api/cli.html#cli_e_eval_script) 执行脚本，cmd中不支持单引号，在 Powershell 或 Git bash 中， `'` 和 `"` 都可用。

-   [`-h`, `--help`](http://nodejs.cn/api/cli.html#cli_h_help) 帮助

-   [`-i`, `--interactive`](http://nodejs.cn/api/cli.html#cli_i_interactive) 交互模式 打开REPL

-   [`-p`, `--print "script"`](http://nodejs.cn/api/cli.html#cli_p_print_script) 等同于-e 但会打印结果

-   [`-r`, `--require module`](http://nodejs.cn/api/cli.html#cli_r_require_module) 在启动时预加载指定的模块。

    遵循 `require()` 的模块解析规则。 `module` 可以是一个文件的路径，或一个 node 模块名称。

    **require 方法的加载规则**

    1.  优先从缓存中加载
    2.  核心模块
    3.  路径形式的模块
    4.  第三方模块

-   [`-v`, `--version`](http://nodejs.cn/api/cli.html#cli_v_version) 打印版本



node命令如果没有参数时，会进入REPL(Read Eval Print Loop:交互式解释器)环境，类似shell

```js
.break    Sometimes you get stuck, this gets you out
.clear    Alias for .break
.editor   Enter editor mode
.exit     Exit the repl
.help     Print this help message
.load     Load JS from a file into the REPL session
.save     Save all evaluated commands in this REPL session to a file

ctrl + c - 退出当前终端。
ctrl + c 按下两次 - 退出 Node REPL。
ctrl + d - 退出 Node REPL.
向上/向下 键 - 查看输入的历史命令
tab 键 - 列出当前命令
```



node xxx.js 直接执行js脚本



**exports 和 module.exports 的使用**

如果要对外暴露属性或方法，就用 **exports** 就行，要暴露对象(类似class，包含了很多属性和方法)，就用 **module.exports**。