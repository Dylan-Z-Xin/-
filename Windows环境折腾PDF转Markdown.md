### 前言
Marker 是 VikParuchuri 开发的一款将 PDF、EPUB 和 MOBI 转换为 Markdown的工具。据称比nougat快 10 倍，在大多数文档上更准确，并且产生错误的风险较低。
https://github.com/VikParuchuri/marker?tab=readme-ov-file
可能大多数人都不需要这玩意儿，毕竟这年头除了程序员谁会用 Markdown 格式啊？
当然还有 Obsidian 折腾型选手！
说实话这年头各种 ocr 准确率已经很高了，只要把 pdf 转成 word，然后复制粘贴进 markdown 文件也一样。
然而痛点在于，专业书中的各种公式，识别率那是惨不忍睹，就算准确率很高，在md文件中也只是一坨数字，还要手动一个个改成 LaTeX 公式。
可能有人会说，你看 pdf 或者纸质书不也一样吗？
我就不，我就要 All-in-One！

Marker 作者提供了在Linux和Mac系统下的安装方法，然而他并没有Windows系统，因此全文操作参照这篇如何在win环境下安装的教程，当然我们中国人有自己的坑。
https://github.com/VikParuchuri/marker/issues/12

### 1. 安装3.9版本以上的Python
我就选择3.9了，因为`texify`要求 Python 版本为 3.9 或更高版本，版本太高的话也可能跟其他的库有不兼容问题。
https://www.python.org/downloads/release/python-390/

### 2. pip换清华源
由于pip的下载速度很慢，所以我们要先把下载方式改成清华镜像。
在“C:\Users\[你的用户名]”中建一个名为“pip”的文件夹，再在里面新建一个txt文本，填入如下代码：

```
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
[install]
trusted-host=mirrors.aliyun.com
```

之后将txt文件名改为“pip.ini”即可。

### 3.下载代码
不知道大家的习惯是怎样的，毕竟我不是专业的程序员，可能就随意了一些。因此我首先用git下载了一下这个项目的代码。
 `git clone https://github.com/VikParuchuri/marker.git`
然后切换到目录里面：
 `cd marker`
开始安装detectron2：
`git clone https://github.com/facebookresearch/detectron2.git`
`pip install -e detectron2`
原文说需要修复什么错误，但我没遇到也就不管它了。
### 4. 安装软件
下载安装Visual Studio: [https://visualstudio.microsoft.com/vs/community/](https://visualstudio.microsoft.com/vs/community/)
（意味不明，说是需要C/C++环境）
安装CUDA : https://developer.nvidia.com/cuda-downloads
有很多相关的教程，这里就不多赘述了, 注意pytorch版本要支持对应版本的CUDA。
安装tesseract-ocr-w64-setup-5.3.3.20231005.exe [https://digi.bib.uni-mannheim.de/tesseract/](https://digi.bib.uni-mannheim.de/tesseract/)
安装时有两个选项要下载语言的应该可以不勾，虽然本来就没勾上，但是我觉得没啥就让它下载了，结果下了接近两个小时。
安装Ghostscript  gs10021w64.exe [https://ghostscript.readthedocs.io/en/gs10.02.0/Install.html](https://ghostscript.readthedocs.io/en/gs10.02.0/Install.html)

### 5. 安装各种依赖项
直接复制到控制台粘贴就好了：
```
pip install nougat-ocr
pip install wheel
pip install torch 
pip install python-magic
pip install python-magic-bin
pip install ftfy  
pip install spellchecker  
pip install pyspellchecker  
pip install ocrmypdf  
pip install nltk  
pip install thefuzz
pip install ray==2.7.1
pip install PyMuPDF
pip install pydantic
pip install --upgrade fastapi
pip install python-dotenv
pip install pydantic-settings
pip install texify
pip install pyspellchecker
pip install ocrmypdf
pip install scikit-learn
pip install ray

```

可能还有其它依赖项没装上，这时候直接把报错信息扔给GPT就可以了，它会告诉你需要安装什么。
![[Pasted image 20240111043451.png]]

### 6. 下载预训练模型
由于特殊的网络原因，大部分人没法连接到huggingface，经过摸索我终于知道怎么用离线模型了。
这个项目主要用了4个预训练模型：
https://huggingface.co/vikp/layout_segmenter/tree/main
https://huggingface.co/vikp/pdf_postprocessor_t5/tree/main
https://huggingface.co/vikp/column_detector/tree/main
https://huggingface.co/vikp/texify/tree/main
在项目里新建一个叫“vikp”的文件夹，然后把4个预训练模型都下载分别放进各自名字的文件夹里。
![[Pasted image 20240111043932.png]]
正如上文所说，我不是那么专业，并不知道哪些文件可以不用下载，因此我一股脑儿都下了。
![[1704919266371.png]]

### 7.运行代码
作者给出的转换单本书的示例代码是
`python convert_single.py /path/to/file.pdf /path/to/output.md --parallel_factor 2 --max_pages 10`
`--max_pages`是要转化多少页，我一般都是整本书所以不需要。
`--parallel_factor` 数字越高占用的显存和CPU越多，默认是1。
因此我想转化一本路径是`E:\Windows lock\Downloads\Documents\Econometric Analysis Global Edition (William H. Greene) (Z-Library).pdf`的书到同目录下，代码就是：
`python convert_single.py "E:\Windows lock\Downloads\Documents\Econometric Analysis Global Edition (William H. Greene) (Z-Library).pdf" "E:\Windows lock\Downloads\Documents\Econometric Analysis.md" --parallel_factor 2`

但很快我就发现很多不对劲了，首先这本书很大，1000多页，而且控制台也没显示进度条。等了很久终于来了一句潜在的报错：您的输入文本经过分词后的长度超过了模型设定的最大序列长度（384），可能导致模型无法处理。

然后我仔细看了看，如果要用CUDA还是要自己调的。
![[Pasted image 20240111052916.png]]
首先`TORCH_DEVICE: Optional[str] = NONE` 要改成 cuda，然后是 `INFERENCE_RAM` 改成自己显存大小，项目作者原来填的40，可真是有钱呀。
`VRAM_PER_TASK` 是每个任务分配到的显存，并行任务数乘以这个数最好远远低于你的显存，爆显存了好像进度条就不动了，`DEFAULT_LANG` 是书所用的语言。

随后我使用 Acrobat 将 pdf 分割成了38份扔到了名为“Econometric Analysis”的文件夹中，或许在此之前应该优化一下文件大小。
![[Pasted image 20240111054225.png]]

再将“Econometric Analysis”文件夹放到项目文件夹下，这样运行代码就可以只使用相对地址。
因此我认为将大 pdf 拆分进行多线程任务是比较方便的，作者给出的示例代码是：
`python convert.py /path/to/input/folder /path/to/output/folder --workers 10 --max 10 --metadata_file /path/to/metadata.json --min_length 10000`

`--workers`是一次要转换的 pdf 数量。默认情况下设置为 1，上面也说了，数量不要太多避免超过显存，因此我设置的2
`--max`是要转换的 pdf 的最大数量。不填的话就会转换文件夹中的所有 pdf 文件，只能说非常好这就是我想要的。
`--metadata_file`是 json 文件的可选路径，其中包含有关 pdf 的元数据（主要是语言）。如果没有，就会使用默认语言 `DEFAULT_LANG`。
`--min_length`是在考虑处理之前需要从 pdf 中提取的最少字符数。这项可以避免对主要是图像的 pdf 进行 OCR 处理。（减慢一切速度）

因此我这边最后的指令就是：
`python convert.py "./Econometric Analysis" "./Econometric Analysis" --workers 2  --min_length 10000`

最后出来的结果虽然有些乱码，但已经很满意了，毕竟能节省一分时间也是好的。
![[Pasted image 20240111060408.png]]


