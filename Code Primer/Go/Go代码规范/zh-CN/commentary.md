# 注释规范

- 所有导出对象都需要注释说明其用途；非导出对象根据情况进行注释。
- 如果对象可数且无明确指定数量的情况下，一律使用单数形式和一般进行时描述；否则使用复数形式。
- 包、函数、方法和类型的注释说明都是一个完整的句子。
- 句子类型的注释首字母均需大写；短语类型的注释首字母需小写。
- 注释的单行长度不能超过 80 个字符。

### 包级别

- 包级别的注释就是对包的介绍，只需在同个包的任一源文件中说明即可有效。
- 对于 `main` 包，一般只有一行简短的注释用以说明包的用途，且以项目名称开头：

	```Go
	// Gogs (Go Git Service) is a painless self-hosted Git Service.
	package main
	```

- 对于一个复杂项目的子包，一般情况下不需要包级别注释，除非是代表某个特定功能的模块。
- 对于简单的非 `main` 包，也可用一行注释概括。
- 对于相对功能复杂的非 `main` 包，一般都会增加一些使用示例或基本说明，且以 `Package <name>` 开头：

	```Go
	/*
	Package regexp implements a simple library for regular expressions.

	The syntax of the regular expressions accepted is:

	    regexp:
	        concatenation { '|' concatenation }
	    concatenation:
	        { closure }
	    closure:
	        term [ '*' | '+' | '?' ]
	    term:
	        '^'
	        '$'
	        '.'
	        character
	        '[' [ '^' ] character-ranges ']'
	        '(' regexp ')'
	*/
	package regexp
	```

- 特别复杂的包说明，可单独创建 [`doc.go`](https://github.com/robfig/cron/blob/master/doc.go) 文件来加以说明。

### 结构、接口及其它类型

- 类型的定义一般都以单数形式描述：

	```Go
	// Request represents a request to run a command.
	type Request struct { ...
	```

- 如果为接口，则一般以以下形式描述：

	```Go
	// FileInfo is the interface that describes a file and is returned by Stat and Lstat.
	type FileInfo interface { ...
	```


### 函数与方法

- 函数与方法的注释需以函数或方法的名称作为开头：

	```Go
	// Post returns *BeegoHttpRequest with POST method.
	```

- 如果一句话不足以说明全部问题，则可换行继续进行更加细致的描述：

	```Go
	// Copy copies file from source to target path.
	// It returns false and error when error occurs in underlying function calls.
	```

- 若函数或方法为判断类型（返回值主要为 `bool` 类型），则以 `<name> returns true if` 开头：

	```Go
	// HasPrefix returns true if name has any string in given slice as prefix.
	func HasPrefix(name string, prefixes []string) bool { ...
	```

### 其它说明


- 当某个部分等待完成时，可用 `TODO:` 开头的注释来提醒维护人员。
- 当某个部分存在已知问题进行需要修复或改进时，可用 `FIXME:` 开头的注释来提醒维护人员。
- 当需要特别说明某个问题时，可用 `NOTE:` 开头的注释：

	```Go
	// NOTE: os.Chmod and os.Chtimes don't recognize symbolic link,
	// which will lead "no such file or directory" error.
	return os.Symlink(target, dest)
	```
