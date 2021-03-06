# MakeEpub

此工具用于将 *html* 文件转换为 *epub* 格式。它根据html文件中的 *\<h1\>*， *\<h2\>* 等标签，将其拆分为章节，并自动生成目录等信息。支持批处理模式，可一次性转换生成多个epub文件。

This tool helps to create *epub* file from *html* file. It split the html file into chapters by the tags *\<h1\>*, *\<h2\>* ..., and generate the TOC automatically. The batch mode can create multiple epub files in one execution.

它还支持打包、解包epub文件，合并html文件或文本文件；还可以作为一个web服务器，转换上传的zip文件为epub文件。

It also support pack/extract an epub file, merge html/text files. And can be used as a web server to convert an uploaded zip file to an epub file.

## 1. 命令行(Command Line)

	转换(Create)       : makeepub <VirtualFolder> [OutputFolder]
	批处理(Batch)      : makeepub -b <InputFolder> [OutputFolder]
                         makeepub -b <BatchFile> [OutputFolder]
	打包(Pack)         : makeepub -p <VirtualFolder> <OutputFile>
	解包(Extract)      : makeepub -e <EpubFile> <OutputFolder>
	合并(Merge) HTML   : makeepub -mh <VirtualFolder> <OutputFile>
	合并(Merge) Text   : makeepub -mt <VirtualFolder> <OutputFile>
	Web服务器(Server)  : makeepub -s [Port]

各参数含义如下：

The meaning of the arguments are as below:

+ **VirtualFolder** 一个文件夹(如example文件夹下的book文件夹)或zip文件(如example文件夹下的book.zip)，里面包含要处理的文件。(An OS folder (for example: folder *book* in folder *example*) or a zip file(for example: *book.zip* in folder *example*) which contains the input files.)
+ **OutputFolder** 一个文件夹，用于保存输出文件。(An OS folder to store the output file(s).)
+ **InputFolder**  : 一个文件夹，里面有输入文件或文件夹。(An OS folder which contains the input folder(s)/file(s).)
+ **BatchFile**    : 一个文本文件，里面列出了所有要处理的VirtualFolder，每行一个。(A text which lists the path of 'VirtualFolders' to be processed, one line for one 'VirtualFolder'.)
+ **OutputFile**   : 输出文件的路径。(The path of the output file.)
+ **EpubFile**     : 一个epub文件的路径。(The path of an EPUB file.)
+ **Port**         : Web服务器的监听端口，默认80。(The TCP port for the web server to listen to, default value is 80.)

## 2. 转换(Create)

	makeepub <VirtualFolder> [OutputFolder]

处理 *VirtualFolder* 中的文件，生成epub，并保存到 *OutputFolder* 中。在VirtualFolder中，必须有以下三个文件：

Process files in *VirtualFolder*, generate epub file and save it to *OutputFolder* . The 3 files below 3 are mandatory and must exist in VirtualFolder:

+ **book.ini** 配置文件，用于指定书名、作者等信息(configuration file to specify book name, author and etc.)
+ **book.html** 书的正文(The content of the book)
+ **cover.png** or **cover.jpg** or **cover.gif** 封面图片文件(The cover image of the book)

请 **务必** 使用 *UTF-8* 编码保存前两个文件，否则程序可能不能正确处理。

The first 2 files **MUST** stored in *UTF-8* encoding, otherwise, the tool may not able to process them correctly.

除以上文件外，其它书籍需要的文件，如层叠样式表（css），图片等也应保存到此文件夹中。如果文件内容是文本，建议也使用 *UTF-8* 编码保存。

Besides the 3 files above, other files required by the book, like sytle sheet (css) and images, are 
also required to be put into the folder. And if the content of a file is text, it is also recommend to store it in *UTF-8* encoding.

### 2.1 文件格式

下面是对主要文件的格式的简单介绍，更多信息请参考example文件夹中的示例。

Below is a brief introduction of the format of the mandatory files, more information please refer to the examples in the *example* folder.

#### book.ini

此文件基于通用的ini文件格式，每一行的行首不能有空白字符，以'#'开始的行将被视为注释，并被忽略。

This file is based on the common *INI* file format, blank characters are not allowed at the beginning of a line, and line start with '#' will be regard as comment and ignored.

这个文件包含两个节， *book* 和 *output*，book节指定书籍信息，output节指定输出文件信息。下面的列表将介绍其中每一个选项的作用。

This file contains two sections: *book* and *output*. section *book* is for the book information, while section *output* is for the output file. The below list explains the usage of each option.

+ Book节(Section Book)
	- **name**: 书名，如果没有提供会导致程序输出一个警告信息(Name of the book, if not specified, the tool will generate a warning)
	- **author**:  作者，如果没有提供会导致程序输出一个警告信息(Author of the book, if not specified, the tool will generate a warning)
	- **id**: 书的唯一标识，在正规出版的书中，它应该是ISBN编号，如果您没有指定，程序将随机生成一个(The unique identifier, it is the ISBN for a published book. If not specified, the tool will generate a random string for it.)
	- **depth**: 一个 *1* 到 *6* 之间的整数，用于指定章节拆分的粒度，默认为 *1*，即只根据 *h1* 标签拆分章节(An integer between *1* and *6*, specifis how to split the html file into chapters. Default value is *1*, which means the split is based on the *h1* tags)
	- **separator**: 章节分隔符，目前支持 *header* 和 *comment* 两种，默认为 *header* 。如果是 *header* 则按 *h1*, *h2* 等拆分章节；如果是 *comment* 则按 注释方式的 *h1*, *h2* 等拆分章节，如 `<!--<h1>chapter 1</h1>-->`。The chapter separator, can be *header* or *comment*, default is *header*. If it is *header*, then chapters will be split at *h1*, *h2* and etc; if it is *comment*, then split at the comment of *h1*, *h2* and etc, for example `<!--<h1>chapter 1</h1>-->`.
	
+ Output节(Section Output)
	- **path**: 输出epub文件的路径。如果没有指定，程序会产生一个警告且不会生成任何文件(The output path of the target epub file. If the path is not specified, the tool will generate a warning and no file will be created)

下面是book.ini的一个例子。

Below is an example for book.ini.

	[book]
	name=My First eBook
	author=Super Man
	id=ISBN XXXXXXXXXXXX
	depth=1
	separator=header
	
	[output]
	path=d:\MyBook.epub


#### book.html

它是一个标准的html文件，但必须保证以下两点：

This is a standard html file, but the 2 points below must be followed:

+ \<body\>标签必须独占一行 (The \<body\> tag must be in its own line)
+ “\<h1\>...\</h1\>”等标题必须独占一行(Headers like "\<h1\>...\</h1\>" must be in their own lines)

根据 *depth* 设置，程序会将此文件拆分成章节文件，\<body\>标签之前的内容会被复制到每个章节文件的开头。

According to the *depth* setting, the tool will split this file into chapter files, the content before \<body\> tag will be copied to the beginning of each chapter file.


#### cover.png/jpg/gif

一个图片文件，它将被用来生成封面。这个文件可以是cover.png、cover.jpg和cover.gif中的任意一个，如果存在多个，如同时有cover.png和cover.jpg，那么程序会随机使用其中一个生成封面。

An image file which will be used to create the book cover. It can be 'cover.png', 'cover.jpg' or 'cover.gif', if more than one file exists (for example: both 'cover.png' and 'cover.jpg'), the tool will select one randomly.

封面文件的名字是cover.html，所以请勿使用这个文件名，否则程序的行为将是未知的。

The file name of the cover page is 'cover.html', please don't use this name for any other purpose, otherwise the behavior of this tool is not defined.


### 2.2 输出文件的路径(path of the output file)

输出文件的路径取决于 *book.ini* 中 *output* 节的 *path* 选项，命令行中的 *OutputFolder* 参数，以及程序的当前工作文件夹。

The path of the out file is determined by the *path* option of section *output* in *book.ini*, argument *OutputFolder* in command line, and the current working folder of the tool.

如果缺少path选项，不会生成任何文件。

No file will be create if there's no option *path*.

如果没有OutputFolder参数，且path是相对路径，输出文件路径是相对于当前工作文件夹的path。

If argument *OutputFolder* is not specified, and *path* is relative, the output file path will be *path* relative to the current working folder.

如果没有OutputFolder参数，且path是绝对路径，输出文件路径是path。

If argument *OutputFolder* is not specified, and *path* is absolute, the output file path is *path*.

如果指定了OutputFolder，文件会被保存在OutputFolder，文件名是path中的文件名部分。

If *OutputFolder* is specified, output file will be save at *OutputFolder*, and file name is the 'file name' in *path*.

## 3. 批处理(Batch)

	makeepub -b <InputFolder> [OutputFolder]
	makeepub -b <BatchFile> [OutputFolder]

批处理模式，相当于对InputFolder中的(或BatchFile列出的)每个VirtualFolder **folder**，调用：

Batch mode, is equal to: for each *VirtualFolder* **folder** in *InputFolder* (or listed in *BatchFile), call:

	makeepub folder [OutputFolder]
	

## 4. 打包(Pack)

	makeepub -p <VirtualFolder> <OutputFile>

将VirtualFolder中的文件打包成一个EPUB文件，保存为OutputFile。

Pack the files in *VirtualFolder* into an EPUB and save it as *OutputFile*.

## 5. 解包(Extract)

	makeepub -e <EpubFile> <OutputFolder>

将EpubFile解包到OutputFolder中。

Extract *EpubFile* to folder *OutputFolder*.

## 6. 合并(Merge)

	makeepub -mh <VirtualFolder> <OutputFile>
	makeepub -mt <VirtualFolder> <OutputFile>

按文件名升序合并VirtualFolder中的文件，并将合并结果保存为OutputFile。合并模式可以使html模式(-mh)或文本(-mt)。

Sort files in *VirtualFolder* in ascend order by file name, merge them, and save the merge result as *OutputFile*. The merge mode can be *html*(-mh) or *text*(-mt).

文本模式是简单的将文件内容连接在一起，Html模式会分析文件，只保留一份文件头(&lt;body&gt;之前的部分)和文件尾(&lt;/body&gt;之后的部分)。

*text* mode is simply merge file content one by one. *html* mode will analysis the file to keep only one copy of file header (content before &lt;body&gt;) and file footer (content after &lt;/body&gt;).


## 7. Web服务器(Web Server)

	makeepub -s [Port]

以Web服务器形式运行，处理用户上传的zip文件，生成EPUB文件，供用户下载。

Run as a web server, process the user uploaded zip file, and generate the EPUB file for user to download.

如果不需要此功能，可将其删除以减小可执行文件的体积。

If you don't need this feature, it can be removed to reduce the size of the executable file.

## 8. 授权及其他(License & Others)

MakeEpub是自由软件，基于[MIT授权](http://opensource.org/licenses/mit-license.html)发布

MakeEpub is free software distributed under the terms of the [MIT license](http://opensource.org/licenses/mit-license.html).

此程序是根据我自己制作epub书籍的需要编写，同时也通过编写过程熟悉了[Go语言](http://golang.org/)(可能需翻墙)。今后，将仅修正bug，而不再增加新的功能。

This tool is developed for my own need when creating epub book, I also learned the [Go program language](http://golang.org/). From now on, I will only fix bugs and won't add new feature any more.