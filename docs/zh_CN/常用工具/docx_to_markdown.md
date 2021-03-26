docx 格式的word文档转为markdown
---------------------------------------------------

1. 安装pandoc
    1. exe 安装包，一般需要vpn下载安装
    2. Chocolatey安装
    
        * 以管理员打开powershell 执行

        ```powershell
        Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
        ```
  
        * 安装pandoc及相关依赖

        ```powershell
        choco install pandoc
        choco install rsvg-convert python miktex
        ```
        * Pandoc creates PDFs using LaTeX. We recommend installing it via MiKTeX.
            `下载安装mitmproxy-6.0.2-windows-installer.exe`

2. 在终端执行转换命令
   
* docx 到 markdown 格式转换，带图片等多媒体

    **在本地目录下指定：**
    + 图片多媒体目录：./MediaFolder
    + 输入的word文件: input.docx
    + 输出的markdown文件：output.md

    `pandoc --extract-media ./MediaFolder input.docx -o output.md`

