# windows软件列表

[toc]

### 7-zip

### autodarkmode

> 设置启动时间

### Baidupan

>文件路径，同时上传下载任务数，禁止开机自启

### Baidu Ai studio

> ```shell
> //环境
> pip install -i https://pypi.tuna.tsinghua.edu.cn/simple torch==1.6.0+cu101 torchvision==0.7.0+cu101 -f https://download.pytorch.org/whl/torch_stable.html
> pip install -i https://pypi.tuna.tsinghua.edu.cn/simple numpy matplotlib  scipy  pandas scikit-learn tensorboard 
> pip install -i https://pypi.tuna.tsinghua.edu.cn/simple opencv jupyterlab sympy pylint seaborn
> //查看占用
> nvidia-smi -l 3
> ```
>

### Chrome

> https://www.microsoft.com/en-us/edge/business/download 下载beta版本
>
> 文件路径，搜索引擎，Ghelper，油猴，ublock，darkreader，有道划词翻译， bitwarden

```
// ==UserScript==
// @name         知乎助手，独家、原创
// @namespace    http://tampermonkey.net/
// @version      1.0.28
// @description  功能简介：设置面板默认隐藏，按右下角黑色+号，显示。一，屏蔽时间线中纯视频营销号回答，二屏蔽各类广告。三，根据关键词屏蔽回答
// @author       桃源隐叟
// @match        *://www.zhihu.com/*
// @match        *://www.zhihu.com
// @grant        none
//@require https://code.jquery.com/jquery-2.1.4.min.js
// ==/UserScript==

(function() {
    'use strict';
    /* globals jQuery, $, waitForKeyElements */

    // Your code here...
    var controlPanel=`<p class="toggle-control" style="z-index:201;position:fixed;right:100px;bottom:100px;margin:2px 1px 1px 2px;text-decoration:underline;">
<img src="http://pic.90sjimg.com/design/00/21/84/57/58fd89ee39300.png!/fw/250/quality/90/unsharp/true/compress/true/canvas/250x250/cvscolor/FFFFFFFF" style="width:30px;height:30px;"></p>
<div style="z-index:200;position:fixed;right:100px;bottom:100px;border:1px solid #888;padding:30px;border-radius:5px;background-color:white;display:none" id="control-div">
<h2>设置屏蔽选项</h2>
<br>
<span>屏蔽购物推荐</span><input type="radio" name="recommend" value="on" checked>开<input type="radio" name="recommend" value="off">关<br>
<span>屏蔽信息流广告</span><input type="radio" name="ads" value="on" checked>开<input type="radio" name="ads" value="off">关<br>
<span>屏蔽首页关键词</span><input type="radio" name="keyword" value="on" checked>开<input type="radio" name="keyword" value="off">关<br>
<input type="text" placeholder="test1,test2" class="blockkeyword"><br>
<span>屏蔽问题关键词</span><input type="radio" name="qKeyword" value="on" checked>开<input type="radio" name="qKeyword" value="off">关<br>
<input type="text" placeholder="知乎盐选" class="questionB"><br>
<span>屏蔽知乎</span><input type="radio" name="zhihu" value="on" >开<input type="radio" name="zhihu" value="off" checked>关<br>
<input type="text" placeholder="好好工作，暂时别看知乎，目前XX还没有完成" class="blocksite"><br>
</div>`

    document.body.insertAdjacentHTML("afterBegin",controlPanel);


    window.onload=()=>{
        initSetting();
        loadSetting();
        funcBlockAds();
        funcBlockByKeyWord();
        funcBlockSite();
        funcBlockQuestion();
    }

    document.body.onscroll=function(){
        funcBlockRecommend();
        funcBlockAds();
        funcBlockByKeyWord();
        funcBlockSite();
        funcBlockQuestion();
    }


    function funcBlockRecommend(){
        if($("[name='recommend']:checked")[0].value==="on"){
            $(".RichText-MCNLinkCardContainer").css("display","none");
        }else{
            $(".RichText-MCNLinkCardContainer").css("display","block");
        }
    }
    function funcBlockAds(){
        if($("[name='ads']:checked")[0].value==="on")
        {
            $(".Card").find(".ZVideoItem").parent().parent().css("display","none");
            $(".TopstoryItem--advertCard").css("display","none");
            $(".Pc-card").css("display","none");
        }else{
            $(".Card").find(".ZVideoItem").parent().parent().css("display","block");
            $(".TopstoryItem--advertCard").css("display","block");
            $(".Pc-card").css("display","block");
        }
    }

    function funcBlockByKeyWord(){
        var blockKeywords=$(".blockkeyword")[0].value;
        if(blockKeywords!=""){
            var bkArray=blockKeywords.split(",");
            for(let i=0;i<bkArray.length;i++){
                if($("[name='keyword']:checked")[0].value==="on"){
                    $(`.TopstoryItem:contains(${bkArray[i]})`).css("display","none");
                }else{
                    $(`.TopstoryItem:contains(${bkArray[i]})`).css("display","block");
                }
            }
        }

    }

    function funcBlockQuestion(){
        var questionBs=$(".questionB")[0].value;
        if(questionBs!=""){
            var qb=questionBs.split(",");
            for(let i=0;i<qb.length;i++){
                if($("[name='qKeyword']:checked")[0].value==="on"){
                    $(`.List-item:contains(${qb[i]})`).css("display","none");
                }else{
                    $(`.List-item:contains(${qb[i]})`).css("display","block");
                }
            }
        }

    }

    function funcBlockSite(){
        if($("[name='zhihu']:checked")[0].value==="on"){
            var blockTip=$(".blocksite")[0].value?$(".blocksite")[0].value:$(".blocksite")[0].placeholder;
            var blockHtml=`<h1 style="text-align:center;font-size:50px;">${blockTip}</h1>`;
            //$("body").css("display","none");

            //$("body").html(blockHtml);

            var bodyChildren=$("body").children();
            for(let i=0;i<bodyChildren.length;i++){
                if(bodyChildren[i].id!="control-div"){
                    $(bodyChildren[i]).css("display","none")
                }
            }
            //$("#control-div").css("display","block");
            $(".toggle-control").css("display","block");
            $("body").prepend(blockHtml);
            $("#container").css("display","none");
            $("iframe").css("display","none");
        }else{
            //$("body").html("");
        }
    }


    $("[name='recommend']").on("click",function(){
        setCookie('recommend',$("[name='recommend']:checked")[0].value);
    });

    $("[name='ads']").on("click",function(){
        setCookie('ads',$("[name='ads']:checked")[0].value);
    });

    $("[name='keyword']").on("click",function(){
        setCookie('blockkeywordSwitch',$("[name='keyword']:checked")[0].value);
        setCookie('blockkeyword',$(".blockkeyword")[0].value);
    });

    $("[name='qKeyword']").on("click",function(){
        setCookie('questionBlockSwitch',$("[name='qKeyword']:checked")[0].value);
        setCookie('questionKeyword',$(".questionB")[0].value);
    });

    $("[name='zhihu']").on("click",function(){
        setCookie('blocksiteswitch',$("[name='zhihu']:checked")[0].value);
        setCookie('blocksiteTip',$(".blocksite")[0].value);
    });

    $(".blockkeyword").blur(function(){
        setCookie('blockkeyword',$(".blockkeyword")[0].value);
    });

    $(".questionB").blur(function(){
        setCookie('questionKeyword',$(".questionB")[0].value);
    });

    $(".blocksite").blur(function(){
        setCookie('blocksiteTip',$(".blocksite")[0].value);
    });

    $(".toggle-control").click(function(){
        $("#control-div").toggle();
    });


    function setCookie(name,value)
    {
        var Days = 30;
        var exp = new Date();
        exp.setTime(exp.getTime() + Days*24*60*60*1000);
        document.cookie = name + "="+ escape (value) + ";expires=" + exp.toGMTString();
    }

    function getCookie(name)
    {
        var arr,reg=new RegExp("(^| )"+name+"=([^;]*)(;|$)");

        if(arr=document.cookie.match(reg))

            return unescape(arr[2]);
        else
            return null;
    }

    function loadSetting(){
        if(getCookie("recommend")!=null){
            $(`[name='recommend'][value=${getCookie("recommend")}]`)[0].checked=true;
        }else{
        }

        if(getCookie("ads")!=null){
            $(`[name='ads'][value=${getCookie("ads")}]`)[0].checked=true;
        }else{
        }

        if(getCookie("blockkeywordSwitch")!=null){
            $(`[name='keyword'][value=${getCookie("blockkeywordSwitch")}]`)[0].checked=true;
            $(".blockkeyword")[0].value=getCookie("blockkeyword");
        }else{
        }

        if(getCookie("questionBlockSwitch")!=null){
            $(`[name='qKeyword'][value=${getCookie("questionBlockSwitch")}]`)[0].checked=true;
            $(".questionB")[0].value=getCookie("questionKeyword");
        }else{
        }

        if(getCookie("blocksiteswitch")!=null){
            $(`[name='zhihu'][value=${getCookie("blocksiteswitch")}]`)[0].checked=true;
            $(".blocksite")[0].value=getCookie("blocksiteTip");
        }else{
        }
    }

    function initSetting(){
        if(getCookie("recommend")==null){
            setCookie('recommend',$("[name='recommend']:checked")[0].value);
        }else{
        }

        if(getCookie("ads")==null){
            setCookie('ads',$("[name='ads']:checked")[0].value);
        }else{
        }

        if(getCookie("blockkeywordSwitch")==null){
            setCookie('blockkeywordSwitch',$("[name='keyword']:checked")[0].value);
            setCookie('blockkeyword',$(".blockkeyword")[0].value);
        }else{
        }

        if(getCookie("questionBlockSwitch")==null){
            setCookie('questionBlockSwitch',$("[name='qKeyword']:checked")[0].value);
            setCookie('questionKeyword',$(".questionB")[0].value);
        }else{
        }

        if(getCookie("blocksiteswitch")==null){
            setCookie('blocksiteswitch',$("[name='zhihu']:checked")[0].value);
            setCookie('blocksiteTip',$(".blocksite")[0].value);
        }else{
        }
    }

})();

```



> ```
> ! 2021-04-05 https://www.bilibili.com
> www.bilibili.com##.b-wrap.first-screen
> www.bilibili.com###bili_live > .report-scroll-module.report-wrap-module.space-between
> www.bilibili.com###bili_douga > .report-scroll-module.report-wrap-module.space-between
> www.bilibili.com###bili_report_anime > div.space-between
> www.bilibili.com###bili_report_guochuang > div.space-between
> www.bilibili.com###bili_manga > .report-scroll-module.report-wrap-module.space-between
> www.bilibili.com###bili_music > .report-scroll-module.report-wrap-module.space-between
> www.bilibili.com###bili_dance > .report-scroll-module.report-wrap-module.space-between
> www.bilibili.com###bili_game > .report-scroll-module.report-wrap-module.space-between
> www.bilibili.com###bili_technology > .report-scroll-module.report-wrap-module.space-between
> www.bilibili.com###bili_report_cheese > .space-between
> www.bilibili.com###bili_digital > .report-scroll-module.report-wrap-module.space-between
> www.bilibili.com###bili_life > .report-scroll-module.report-wrap-module.space-between
> www.bilibili.com###bili_food > .report-scroll-module.report-wrap-module.space-between
> www.bilibili.com###bili_animal > .report-scroll-module.report-wrap-module.space-between
> www.bilibili.com###bili_fashion > .report-scroll-module.report-wrap-module.space-between
> www.bilibili.com###bili_information > .report-scroll-module.report-wrap-module.space-between
> www.bilibili.com###bili_ent > .report-scroll-module.report-wrap-module.space-between
> www.bilibili.com###bili_read > .report-scroll-module.report-wrap-module.space-between
> www.bilibili.com###bili_movie > .report-scroll-module.report-wrap-module.space-between
> www.bilibili.com###bili_teleplay > .report-scroll-module.report-wrap-module.space-between
> www.bilibili.com###bili_cinephile > .report-scroll-module.report-wrap-module.space-between
> www.bilibili.com###bili_documentary > .report-scroll-module.report-wrap-module.space-between
> www.bilibili.com##.report-scroll-module.report-wrap-module.b-wrap.s-rec.space-between
> www.bilibili.com##.international-footer
> www.bilibili.com##.list-box
> www.bilibili.com##.comment-list
> www.bilibili.com##.common
> www.bilibili.com##.report-scroll-module.report-wrap-module.recommend-list
> www.bilibili.com##.r-con
> www.bilibili.com##.report-scroll-module.report-wrap-module.recom-wrapper
> 
> ! 2021-04-05 https://live.bilibili.com
> live.bilibili.com##.left-container > .room-info-ctnr
> live.bilibili.com##.room-feed
> live.bilibili.com##.footer-content
> live.bilibili.com##.link-footer
> live.bilibili.com##.z-link-footer-ctnr.link-footer-ctnr
> live.bilibili.com##.z-section-blocks.f-clear.section-block
> 
> 
> ```
>
> 

### CUDA

> cuda安装除Geforce外组件，
>
> cudnn内容复制到C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v9.2

### Cmder

> powershell管理员权限运行  .\Cmder.exe /REGISTER ALL
>
> git添加到环境变量：`C:\DevTools\cmder\vendor\git-for-windows\bin`
>
> 更换字体：concolas
>
> 关闭热键：ctr+, 字体 字号
>
> ```
> git config --global user.name "huangqiang"
> git config --global user.email "huangqiang97@126.com"
> ssh-keygen -t rsa -C "huangqiang97@126.com"
> ```
>
> 把id_rsa.pub添加到github SSH密钥中
>
> ---------
>
> 博客设置：
>
> ```git clone https://github.com/HuangQiang97/HuangQiang97.github.io.git```
>
> Create git repository:
>
> ```
> mkdir huangqiang97
> cd huangqiang97
> git init
> git add .
> git commit -m "first commit"
> git remote add origin git@gitee.com:huangqiang97/huangqiang97.git
> git push -u origin master
> ```
> 
>Existing repository
> 
>```
> cd existing_git_repo
> git remote add origin git@gitee.com:huangqiang97/huangqiang97.git
> git push -u origin master
> ```

### Ditto

> 设定保存时间与条数，开机自启

###  Dotnet

> 将安装路径加入环境变量

### Everything

> 开机自启

### Edge

> https://www.microsoft.com/en-us/edge/business/download 下载beta版本
>
> 文件路径，搜索引擎，Ghelper，油猴，ublock，darkreader，有道划词翻译 bitwaeden,Listen1,idm

### Eudic

> 安装词库
>
> 鼠标取词
>
> 开机启动

### Firefox

> 文件路径，搜索引擎，Ghelper，油猴，uBlock Origin，darkreader，沙拉划词翻译，IDM
>
> 百度云脚本：网盘助手、网盘链接检查：应用ID修改为：778750

### FluentTerminal

> 添加配色：
>
> Monokai.flutecolors
>
> ```
> {
> "Name": "Monokai",
> "Author": "David Refoua <David@Refoua.me>",
> "Colors": {
> "Foreground": "#F8F8F0",
> "Background": "#282828",
> "Cursor": "#f8f8f0",
> "CursorAccent": "#272822",
> "Selection": "rgba(73, 72, 62, 0.3)",
> "Black": "#2E3436",
> "Red": "#CB064D",
> "Green": "#8dd006",
> "Yellow": "#e6db74",
> "Blue": "#0376dd",
> "Magenta": "#9d74e6",
> "Cyan": "#52aebf",
> "White": "#F8F8F2",
> "BrightBlack": "#555753",
> "BrightRed": "#F92672", // #f3044b
> "BrightGreen": "#A6E22E",
> "BrightYellow": "#FED330",
> "BrightBlue": "#0383f5",
> "BrightMagenta": "#AE81FF",
> "BrightCyan": "#66D9EF",
> "BrightWhite": "#F8F8F0"
> }
> }
> ```
>
> -------------------------
>
> 添加配置：
>
> 新建环境变量
>
> ```
> CMDER_ROOT=Cmder/installpath
> ```
>
> Bash
>
> ```
> Bash
> C:\DevTools\cmder\vendor\git-for-windows\bin\bash.exe
> C:\Users\huangqiang\Desktop
> --login -i
> Monokai
> ```
>
> 加入鼠标右键
>
> bat文件：
>
> ```
> reg add "HKCU\Software\Classes\Directory\shell\FluentTerminal here\command" /d "\"%LOCALAPPDATA%\Microsoft\WindowsApps\flute.exe\" new \"%%1\"" /f
> 
> reg add "HKCU\Software\Classes\Directory\Background\shell\FluentTerminal here\command" /d "\"%LOCALAPPDATA%\Microsoft\WindowsApps\flute.exe\" new \"%%V\"" /f
> 
> reg add "HKCU\Software\Classes\LibraryFolder\Background\shell\FluentTerminal here\command" /d "\"%LOCALAPPDATA%\Microsoft\WindowsApps\flute.exe\" new \"%%V\"" /f
> ```
>
> 从鼠标右键删除
>
> bat文件：
>
> ```
> reg delete "HKCU\Software\Classes\Directory\shell\FluentTerminal here" /f
> reg delete "HKCU\Software\Classes\Directory\Background\shell\FluentTerminal here" /f
> reg delete "HKCU\Software\Classes\LibraryFolder\Background\shell\FluentTerminal here" /f
> ```
>
> 

### PDF X-change viewer

> 简体文字

### Huorong

> 开机自启菜单，文件右键菜单，垃圾清理，

### Wox

> 开机自启，自动隐藏

### Go

> Go开启代理：```go env -w GOPROXY=https://goproxy.cn,direct```
>
> 新建环境变量：```set GOPATH=D:\GoBasePath;D:\Go```，环境变量-用户变量：```GOPATH=D:\GoBasePath;D:\Go```
>
> 查看环境参数：```go env```
>
> path:        ``` GoInstallPath\bin```
>
> GOROOT: ```GoInstallPath```
>
> GOPATH:  工作目录，下有：src pkg bin子目录，可存在多个，优先级依次降低
>
> 找包顺序：```gomod =off : goroot/src->gopath0/src->gopath2/src```
>
> ​				   ``` gomod =on  : goroot/pkg/mod->gopath0/pkg/mod->gopath2/pkg/mod```
>
> ------------------
>
> 自动初始化目录：```sh ./create_go_proj.sh porject_name```
>
> ```sh
> #!/bin/bash
> 
> #########################################################
> 
> # Useage : ./create_go_proj.sh
> # sh ./create_go_proj.sh porject_name
> # Description: 创建一个go可编译的工程
> #————————————–————————————–
> # 默认情况下运行本程序，会生成如下目录和文件:
> # test
> # ├── bin
> # ├── install.sh
> # ├── pkg
> # └── src
> # ├── config
> # │   └── config.go
> # └── test
> # └── main.go
> #————————————–————————————–
> # 5 directories, 3 files
> # 其中:
> # 1, install.sh为安装文件，
> # 2, config.go为test项目的配置文件
> # 3, main.go
> # 生成完毕之后运行进入test目录，运行install.sh会生成如下文件和目录
> # ├── bin
> # │   └── test
> # ├── install.sh
> # ├── pkg
> # │   └── darwin_amd64
> # │   └── config.a
> # └── src
> # ├── config
> # │   └── config.go
> # └── test
> # └── main.go
> # 6 directories, 5 files
> #
> # 多了两个文件
> # 1, bin目录下的test，这个是可执行文件
> # 2, pkg/darwin_amd64下的config.a，这个是config编译后产生的文件
> #
> #########################################################
> PWD=$(pwd)
> cd $PWD
> 
> if [[ "$1" = "" ]]; then
> echo "Useage: ./mk_go_pro.sh porject_name"
> echo -ne "Please input the Porject Name[test]"
> read Answer
> if [ "$Answer" = "" ]; then
> echo -e "test";
> PRO_NAME=test;
> else
> PRO_NAME=$Answer;
> fi
> else
> PRO_NAME=$1;
> fi
> 
> #########################################################
> 
> #创建目录
> echo "Init Directory …"
> mkdir -p $PRO_NAME/bin
> mkdir -p $PRO_NAME/pkg
> mkdir -p $PRO_NAME/src/config
> mkdir -p $PRO_NAME/src/$PRO_NAME
> 
> 
> #########################################################
> #创建 install.sh 文件
> echo "Create install/install.sh …"
> cd $PRO_NAME
> echo "#!/bin/bash" > install.sh
> echo "if [ ! -f install.sh ]; then" >> install.sh
> echo "echo "install must be run within its container folder" 1>&2" >> install.sh
> echo "exit 1" >> install.sh
> echo "fi" >> install.sh
> echo >> install.sh
> echo "CURDIR=\`pwd\`" >> install.sh
> echo "OLDGOPATH=\"\$GOPATH\"" >> install.sh
> echo "export GOPATH=\"\$CURDIR\"" >> install.sh
> echo >> install.sh
> echo "gofmt -w src" >> install.sh
> echo "go install $PRO_NAME" >> install.sh
> echo "export GOPATH=\"\$OLDGOPATH\"" >> install.sh
> echo >> install.sh
> echo "echo "finished"" >>install.sh
> chmod +x install.sh
> 
> #创建 config.go 文件
> echo "Create src/config/config.go …"
> cd src/config
> echo package config > config.go
> echo >> config.go
> echo func LoadConfig\(\) { >> config.go
> echo >> config.go
> echo "}" >> config.go
> 
> #创建 main.go
> echo "Create src/$PRO_NAME/main.go …"
> cd ../$PRO_NAME/
> echo "package main" > main.go
> echo >> main.go
> echo "import (" >> main.go
> echo " \"config\"" >> main.go
> echo " \"fmt\"" >> main.go
> echo ")" >> main.go
> echo >> main.go
> echo "func main() {" >> main.go
> echo " config.LoadConfig()" >> main.go
> echo " fmt.Println(\"Hello $PRO_NAME!\")" >> main.go
> echo "}" >> main.go
> echo "All Done!"
> ```
>

### Goland

> 字体
>
> 将当前工程路径加入project gopath
>
> proj gopath 优先于 glob gopath
>
> solarized  themes
>
> 

### Intel 核显驱动

### IDEA

> 字体
>
> class注释：
> ```
> 
> #if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")package ${PACKAGE_NAME};#end
> #parse("File Header.java")
> /**
>  * @author  huangqiang
>  * @date    ${DATE} ${TIME}
>  * @description TODO
>  * @modified    
>  * @version TODO
>  */
> public class ${NAME} {
> }
> 勾选 enable live Templates
> 
> ```
>
> 方法注释：
>
> ```
> 快捷方式：/*
> /**
> 
>  * @create  huangqiang
>  * @description TODO
>  * @time    $date$ $time$
>  * @param   $params$
>  * @return  $return$
>  */
> 
> ```
>
> alibaba插件 , gruvbox ,solarized themes , junitGenerator
>
> ```
> ######################################################################################## 
> ## 
> ## Available variables: 
> ##         $entryList.methodList - List of method composites 
> ##         $entryList.privateMethodList - List of private method composites 
> ##         $entryList.fieldList - ArrayList of class scope field names 
> ##         $entryList.className - class name 
> ##         $entryList.packageName - package name 
> ##         $today - Todays date in MM/dd/yyyy format 
> ## 
> ##            MethodComposite variables: 
> ##                $method.name - Method Name 
> ##                $method.signature - Full method signature in String form 
> ##                $method.reflectionCode - list of strings representing commented out reflection code to access method (Private Methods) 
> ##                $method.paramNames - List of Strings representing the method's parameters' names 
> ##                $method.paramClasses - List of Strings representing the method's parameters' classes 
> ## 
> ## You can configure the output class name using "testClass" variable below. 
> ## Here are some examples: 
> ## Test${entry.ClassName} - will produce TestSomeClass 
> ## ${entry.className}Test - will produce SomeClassTest 
> ## 
> ######################################################################################## 
> ## 
> #macro (cap $strIn)$strIn.valueOf($strIn.charAt(0)).toUpperCase()$strIn.substring(1)#end 
> ## Iterate through the list and generate testcase for every entry. 
> #foreach ($entry in $entryList) 
> #set( $testClass="${entry.className}Test") 
> ## 
> package test.$entry.packageName; 
> 
> import static org.junit.jupiter.api.Assertions.*;
> 
> import org.junit.jupiter.api.*;
> 
> /** 
> * ${entry.className} Tester. 
> * 
> * @author huangqiang
> * @since $date
> * @version 1.0 
> */ 
> public class $testClass { 
> 
> @BeforeAll
> public void beforeAll() throws Exception { 
> } 
> 
> 
> @BeforeEach
> public void beforeEach() throws Exception { 
> } 
> 
> @AfterEach
> public void afterEach() throws Exception { 
> } 
> 
> @AfterAll
> public void afterAll() throws Exception { 
> } 
> 
> #foreach($method in $entry.methodList) 
> /** 
> * 
> * Method: $method.signature 
> * 
> */ 
> @Test
> public void test#cap(${method.name})() throws Exception { 
> //TODO: Test goes here... 
> } 
> 
> #end 
> 
> #foreach($method in $entry.privateMethodList) 
> /** 
> * 
> * Method: $method.signature 
> * 
> */ 
> @Test
> public void test#cap(${method.name})() throws Exception { 
> //TODO: Test goes here... 
> #foreach($string in $method.reflectionCode) 
> $string 
> #end 
> } 
> 
> #end 
> } 
> #end
> ```
>
> 

### IDM

> 下载路径，线程数，菜单图标样式：3D,下载图标：mini
>
> chrome 插件：https://chrome.google.com/webstore/detail/idm-integration-module/ngpampappnmepgilojfohadhhmbhlaek

### Imagine

### Itools

### JJDown

> 下载路径

### Listen1

> 主题

### JDK

> JDK/bin 加入环境变量

### Matlab

> 修改快捷方式起始目录
>
> 220200896@seu.edu.cn 
>
> 112358Hq

### MDict

### Mysql

> VC_redist.exe
>
> bin加入环境变量
>
> my.ini
>
> ```
> [client]
> # 设置mysql客户端默认字符集
> default-character-set=utf8mb4
> 
> [mysqld]
> # 设置3306端口
> port = 3306
> # 设置mysql的安装目录
> basedir=D:\\mysql
> # 允许最大连接数
> max_connections=20
> # 服务端使用的字符集默认为8比特编码的latin1字符集
> character-set-server=utf8mb4
> # 创建新表时将使用的默认存储引擎
> default-storage-engine=INNODB
> ```
> 
> ```shell
>
> cd mysql/bin
> mysqld --initialize --console   //会生成临时密码
> mysqld install
> net start mysql
> // 更改密码 o%>yI)Bmd2ad
> ALTER user 'root'@'localhost' IDENTIFIED BY 'root';
> FLUSH PRIVILEGES;
> 
> 
> 
> // 更改加密方式
> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'root';
> flush privileges；
> ```
>
> IDEA :mysql serverTimeZone :  Asia/Shanghai

### MongDB

> 自定义安装，取消compass组件
>
> 加入环境变量
>
> ```go
> 启动服务
> cd bin
> mongod --dbpath=F:\MongDB\data --logpath=F:\MongDB\log\mongod.log  --logappend
> 开始使用
> mongo
> ```
>

### MinGW

> ```
> 加入系统环境
> D:\MinGw_x86_64\mingw64\bin
> 
> D:\MinGw_x86_64\mingw64\lib
> 
> D:\MinGw_x86_64\mingw64\include
> ```

### Mathpix

> 禁止开机自启
>
> 登录

### Miniconda

>```
>conda create -n ML python=3.7
>activate ML
>conda install pytorch torchvision torchaudio cpuonly -c pytorch
>conda install numpy matplotlib imageio scipy 
>conda install seaborn pandas scikit-learn jupyterlab sympy pylint 
>conda  install tensorboard tensorflow
>
>
>
>conda config
>.condarc
>
>channels:
>  - defaults
>show_channel_urls: true
>default_channels:
>  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
>  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
>  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2
>custom_channels:
>  conda-forge: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
>  msys2: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
>  bioconda: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
>  menpo: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
>  pytorch: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
>  simpleitk: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
>
>jupyter插件：lsp,jupyterlab_variableinspector，drawio
>pip install jupyter-lsp
>conda install  nodejs
>jupyter labextension install @krassowski/jupyterlab-lsp
>pip install python-language-server[python]
>重建：jupyter lab build
>```
>
>win + r 运行 regedit 
>
>打开：计算机\HKEY_CLASSES_ROOT\Directory\Background\shell\   
>
>新建项 ： jupyter
>
>把jupyter项中默认字符串数据改为 ： jupyter lab
>
>在jupyter项中新建字符串：名称为 Icon ，数据指向jupyter icon
>
>在jupyter项中新建项 command 默认字符串数据改为  "D:\MiniConda\envs\pytorch\Scripts\jupyter-lab.exe" "%V"
>
>

### Notion

### Notepad3 

>  字体

### Nutstore

> 同步文件夹

### NeteaseMail

> 文件夹

### NeteaseDict

> 取词：  shift+鼠标
>
> 离线词典
>
> 开机自启

### NodeJS

### Office

> 关闭自动更新
>
> 更改文件夹

### Potplayer

> lavfilter
>
> 独显解码

### Pycharm

### PyQt

> pip install pyqt5
>
> pip install  pyqt5-tools
>
> 新建环境变量： QT_QPA_PLATFORM_PLUGIN_PATH=D:\MiniConda\Lib\site-packages\pyqt5_tools\Qt\plugins
>
> 
>
> PyCharm的File->Settings->Tools->External tools 添加工具：
>
> name = QtDesigner
>
> Program = D:\MiniConda\Lib\site-packages\pyqt5_tools\Qt\bin\designer.exe
>
> working dir =  E:\PycharmProject\QtProject\UI
>
> 
>
> name = PyUIC
>
> Program = D:\MiniConda\python.exe
>
> Argu = -m PyQt5.uic.pyuic  $FileName$ -o $FileNameWithoutExtension$.py
>
> working dir = E:\PycharmProject\QtProject\UI

### Simplestickynotes

### Snipaste

> 开机自启

### TexLive

> ```D:\Texlive\texlive\2019\bin\win32``` 加入系统路径

### Texstudio

### Typora

> ```C:\Users\huang\AppData\Roaming\Typora\themes``` 加入主题
>
> pandoc

### thunder

> Thunder\Data\ThunderPush\updatethunder\Program\OnlineInstall.exe 只读
>
> 修改文件夹

### Tim

> 缓存文件路径，禁止开机自启
>
> https://www.geek-share.com/detail/2705472906.html

### TrafficMonitor

> 状态栏，开机自启

### v2rayN

开机自启动

### VMware

> Manjaro
>
> 换源 
>
> ```
> sudo pacman-mirrors -i -c China -m rank
> 
> sudo vi /etc/pacman.conf
> 修改/etc/pacman.conf,在最后一行添加：
> [archlinuxcn]
> SigLevel = Optional TrustedOnly
> #中科大源
> Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
> #清华源
> Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
> [antergos]
> SigLevel = TrustAll
> Server = https://mirrors.tuna.tsinghua.edu.cn/antergos/$repo/$arch
> [arch4edu]
> SigLevel = TrustAll
> Server = https://mirrors.tuna.tsinghua.edu.cn/arch4edu/$arch
> 
> 安装archlinuxcn签名钥匙：
> sudo pacman-mirrors -g
> sudo pacman -Syyu
> sudo pacman -Syy
> sudo pacman -S archlinuxcn-keyring
> sudo pacman -S antergos-keyring
> 
> ```
>
> 安装工具
>
> ```
> sudo pacman -S yaourt
> sudo yaourt -S yay
> 
> ```
>
> Chrome浏览器
>
> ```
> sudo pacman -S google-chrome
> ```
>
> 中文输入法
>
> ```
> sudo pacman -S fcitx-im  # 默认全部安装
> sudo pacman -S fcitx-configtool
> sudo pacman -S fcitx-googlepinyin  # 安装谷歌拼音
> sudo pacman -S fcitx-rime    
> 配置 ~/.xprofile、~/.profile 添加：
> 		export GTK_IM_MODULE=fcitx
> 		export QT_IM_MODULE=fcitx
> 		export XMODIFIERS="@im=fcitx"
> source ~/.xprofile
> source ~/.profile
> fctix 配置中添加Google输入法，中州毫
> ```
>
> Vim
>
> ```
> sudo pacman -S vim
> ```
>
> Java
>
> ```
> 安装JDK：yaourt jdk
> 查看可选环境：archlinux-java status
> 设置环境：sudo archlinux-java set jdk-9.0.4
> ```
>
> Go
>
> ```
> 解压并移动 tar -C /usr/local -xzf go1.10.3.linux-amd64.tar.gz
> 编辑 /etc/profile 添加：
> export GOROOT=/usr/local/go
> export GOPATH=/home/huangqiang/GoBase
> export PATH=$PATH:/usr/local/go/bin
> 使用命令 source /etc/profile 生效。
> 设置代理 go env -w GOPROXY=https://goproxy.cn,direct
> 验证	go env
> ```
>
> Python
>
> ```
> MiniConda
> 
> 安装 bash Miniconda3-latest-Linux-x86_64.sh 
> cd ~
> 激活 source .bashrc
> 生成配置文件 conda config --set show_channel_urls yes
> 编辑 /home/huangqiang/.condarc 修改为：
> channels:
>   - defaults
> show_channel_urls: true
> channel_alias: https://mirrors.tuna.tsinghua.edu.cn/anaconda
> default_channels:
>   - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
>   - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
>   - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
>   - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/pro
>   - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2
> custom_channels:
>   conda-forge: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
>   msys2: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
>   bioconda: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
>   menpo: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloudy
>   pytorch: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
>   simpleitk: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
> 清除索引缓存 conda clean -i 
> 安装包： matplotlib pandas imageio scipy sympy jupyterlab searorn numpy pylint
> 加入变量
> echo 'export PATH="/home/huangqiang/miniconda3/bin/:$PATH"' >> ~/.bashrc 
> echo 'export PATH="/home/huangqiang/miniconda3/bin:$PATH"' >> ~/.zshrc
> source ~/.bashrc
> source ~/.zshrc
> ```
>
>  Typora
>
> ```
> yaourt typora
> ```
>
> Vscode
>
> ```
> sudo pacman -S visual-studio-code-bin 
> 
> launch.json
> 
> {
>     // 使用 IntelliSense 了解相关属性。 
>     // 悬停以查看现有属性的描述。
>     // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
>     "version": "0.2.0",
>     "configurations": [
>         {
>             "name": "Go",
>             "type": "go",
>             "request": "launch",
>             "mode": "debug",
>             "program": "${file}"
>         },
>         {
>             "type": "java",
>             "name": "Java",
>             "request": "launch",
>             "mainClass": ""
>         },
> 
>         {
>             "name": "Python",
>             "type": "python",
>             "request": "launch",
>             "program": "${file}",
>             "console": "integratedTerminal"
>         },
>         {
>             "name": "C/C++",
>             "type": "cppdbg",
>             "request": "launch",
>             "program": "${fileDirname}/${fileBasenameNoExtension}",
>             "args": [],
>             "stopAtEntry": false,
>             "cwd": "${workspaceFolder}",
>             "environment": [],
>             "externalConsole": false,
>             "MIMode": "gdb",
>             "setupCommands": [
>                 {
>                     "description": "为 gdb 启用整齐打印",
>                     "text": "-enable-pretty-printing",
>                     "ignoreFailures": true
>                 }
>             ],
>             "preLaunchTask": "g++ build active file",
>             "miDebuggerPath": "/usr/bin/gdb"
>         },
>     ]
> }
> 
> tasks.json
> 
> 
> {
>     "version": "2.0.0",
>     "tasks": [
>         {
>             "type": "shell",
>             "label": "g++ build active file",
>             "command": "/usr/bin/g++",
>             "args": [
>                 "-g",
>                 "${file}",
>                 "-o",
>                 "${fileDirname}/${fileBasenameNoExtension}"
>             ],
>             "options": {
>                 "cwd": "/usr/bin"
>             },
>             "problemMatcher": [
>                 "$gcc"
>             ],
>             "group": {
>                 "kind": "build",
>                 "isDefault": true
>             },
>             "presentation": {
>                 "echo": true,
>                 "reveal": "always",
>                 "focus": false,
>                 "panel": "new",
>                 "showReuseMessage": true,
>                 "clear": false
>             },
>         },
> 
>     ]
> }
> 
> ```
>
>  Notepadqq
>
> ```
> yaourt notepadqq
> ```
>
>  ZSH
>
> ```
>  安装zsh : 		 sudo  pacman -S zsh
>  zsh设为默认shell:  sudo chsh -s /usr/bin/zsh
>  				  reboot 
>  安装oh my zsh :   sh -c "$(wget -O- https://gitee.com/shmhlsy/oh-my-zsh-install.sh/raw/master/install.sh)"
>  安装插件：			cd ~/.oh-my-zsh/plugins/
>  					mkdir incr && cd incr
> 					 wget http://mimosa-pudica.net/src/incr-0.2.zsh
>  					cd ~/.oh-my-zsh/plugins/
>  git clone https://github.com/zsh-users/zsh-syntax-highlighting.git
>  echo "source ${(q-)PWD}/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh" >> ${ZDOTDIR:-$HOME}/.zshrc
>  编辑 ~/.zshrc 
>  	  修改：
>       ZSH_THEME="agnoster"
>       plugins=(git extract z)
>       添加：
>       source ~/.oh-my-zsh/plugins/incr/incr*.zsh
>  修改生效：source ~/.zshrc 
> ```
>
> 清除系统中无用的包
>
> ```
> sudo pacman -R $(pacman -Qdtq)
> ```
>
>  清除已下载的安装包
>
> ```
> sudo pacman -Scc
> ```
>
>  开启系统监视器
>
> ```
> 设置 - 面板 - 项目 - 添加 - 系统负载监视器
> ```
>
>  字体
>
> ```
> 设置终端字体： liberation mino regular
> ```
>
>  更改显示
>
> ```
> 设置-> 外观 -> 字体 -> DPI=128
> 设置-> 外观 -> 样式 -> dark
> ```
>
> ### Tim
>
> ```
> sudo pacman -S deepin.com.qq.office
> 中文无法输入：
> 找到如下路径: /opt/deepinwine/apps/Deepin-TIM
> 编辑其中的run.sh文件,在最开头添加,
> export GTK_IM_MODULE=fcitx
> export QT_IM_MODULE=fcitx
> export XMODIFIERS="@im=fcitx"
> ```
>
> ### WPS
>
> ```
> sudo pacman -S wps-office
> sudo pacman -S ttf-wps-fonts
> 解决无法输入中文问题：
> /usr/bin/wps
> /usr/bin/wpp
> /usr/bin/et
> 中最前面添加：
> export XMODIFIERS="@im=fcitx"
> export GTK_IM_MODULE="fcitx"
> export QT_IM_MODULE="fcitx"
> ```
>
> ### 网易云音乐
>
> ```
> sudo pacman -S netease-cloud-music
> ```
>
> ### IDEA
>
> ```
> sudo mkdir /opt/Intellij
> sudo tar -zxvf xxx.tar.gz -C /opt/Intellij
> cd /opt/Intellij/xxx
> sudo chmod a=+rx bin/idea.sh    
> bin/idea.sh
> ```
>
> ### 去除蜂鸣警告音
>
> ```
> 新建文件：/etc/modprobe.d/blacklist.conf
> 写入：blacklist pcspkr
> ```
>
> ​	


### Virtual Box

> ubuntu server 64

### VScode

> 插件：C/C++,java,Go,Matlab,Python,Latex,Markdown,vscode icon,l gruvbox ,Draw.io  Integration,code-runner
>
> ```json
> {
>  "editor.suggestSelection": "first",
>  "vsintellicode.modify.editor.suggestSelection": "automaticallyOverrodeDefaultValue",
>  "editor.fontSize": 18,
>  "debug.console.fontSize": 18,
>  "markdown.preview.fontSize": 18,
>  "terminal.integrated.fontSize": 18,
>  /////////////////////////////////////////////////////////////////////////////////
>  //Matlab 设置
>  "matlab.linterEncoding": "GB 2312",
>  "matlab.linterConfig": "D:\\Matlab\\bin\\win64\\mlint.exe",
>  "matlab.matlabpath": "D:\\Matlab\\\\bin\\matlab.exe",
>  "matlab.mlintpath": "D:\\Matlab\\\\bin\\win64\\mlint.exe",
>  "files.associations": {
>      "*.m": "matlab"
>  },
>  "[matlab]": {
>      "files.encoding": "gb2312"
>  },
>  "files.autoGuessEncoding": true,
>  "editor.snippetSuggestions": "top",
>  /////////////////////////////////////////////////////////////////////////////////////
>  "files.exclude": {
>      "**/.classpath": true,
>      "**/.project": true,
>      "**/.settings": true,
>      "**/.factorypath": true
>  },
>  "diffEditor.ignoreTrimWhitespace": true,
>  "explorer.confirmDelete": false,
>  // icon主题 
>  "workbench.iconTheme": "vscode-icons",
>  "C_Cpp.updateChannel": "Insiders",
>  // java error path
>  "java.errors.incompleteClasspath.severity": "ignore",
>  // python 函数补全括号
>  "python.autoComplete.addBrackets": true,
>  // python 开启代码检查
>  "python.linting.enabled": true,
>  "python.linting.pylintEnabled": true,
>  // Java path
>  "java.home": "D:\\JDK\\JDK11",
>  // 3S自动保存
>  "files.autoSave": "afterDelay",
>  "python.linting.pylintPath": "D:\\MiniConda\\Scripts\\pylint",
>  "editor.formatOnPaste": true,
>  "editor.formatOnType": true,
>  "editor.formatOnSave": true,
>  "vsicons.dontShowNewVersionMessage": true,
>  "git.path": "D:\\cmder\\vendor\\git-for-windows\\git-cmd.exe",
>  "workbench.startupEditor": "newUntitledFile",
>  "terminal.integrated.shell.windows": "C:\\Windows\\System32\\cmd.exe",
>  "breadcrumbs.enabled": true,
>  "editor.renderWhitespace": "none",
>  "editor.renderControlCharacters": false,
>  "editor.minimap.enabled": true,
>  "java.semanticHighlighting.enabled": true,
>  "latex-workshop.view.pdf.viewer": "tab",
>  "python.languageServer": "Microsoft",
>  "go.formatTool": "goimports",
>  "go.useLanguageServer": true,
>  "go.autocompleteUnimportedPackages": true,
>  // gopath支持多个路径，路径间用 ; 分割，src优先级依次降低。vscode中gopath会覆盖系统变量中GOPATH
>  // F:\\GoBasePath中安装vscode运行go的包
>  "go.gopath": "D:\\GoBasePath;D:\\Go",
>  "workbench.colorTheme": "Solarized Dark",
>     
> }
> 
> 设置： "code-runner.executorMap" ：加上"matlab": "cd $dir && matlab -nosplash -nodesktop -r $fileNameWithoutExt"
> ```
>
> 版本一：
>
> ```json
> c_cpp_properties.json
> {
> "configurations": [
> {
> "name": "Win32",
> "includePath": [
> "${workspaceRoot}",
> "D:/MinGw_x86_64/mingw64/lib/gcc/x86_64-w64-mingw32/8.1.0/include/c++",
> "D:/MinGw_x86_64/mingw64/lib/gcc/x86_64-w64-mingw32/8.1.0/include/c++/x86_64-w64-mingw32",
> "D:/MinGw_x86_64/mingw64/lib/Dgcc/x86_64-w64-mingw32/8.1.0/include/c++/backward",
> "D:/MinGw_x86_64/mingw64/lib/gcc/x86_64-w64-mingw32/8.1.0/include",
> "D:/MinGw_x86_64/mingw64/lib/gcc/x86_64-w64-mingw32/8.1.0/include/c++/tr1",
> "D:/MinGw_x86_64/mingw64/x86_64-w64-mingw32/include"
> D],
> "defines": [
> "_DEBUG",
> "UNICODE",
> "__GNUC__=6",
> "__cdecl=__attribute__((__cdecl__))"
> ],
> "intelliSenseMode": "${default}",
> "browse": {
> "path": [
> "${workspaceRoot}",
> "D:/MinGw_x86_64/mingw64/lib/gcc/x86_64-w64-mingw32/8.1.0/include/c++",
> "D:/MinGw_x86_64/mingw64/lib/gcc/x86_64-w64-mingw32/8.1.0/include/c++/x86_64-w64-mingw32",
> "D:/MinGw_x86_64/mingw64/lib/gcc/x86_64-w64-mingw32/8.1.0/include/c++/backward",
> "D:/MinGw_x86_64/mingw64/lib/gcc/x86_64-w64-mingw32/8.1.0/include",
> "D:/MinGw_x86_64/mingw64/lib/gcc/x86_64-w64-mingw32/8.1.0/include/c++/tr1",
> "D:/MinGw_x86_64/mingw64/x86_64-w64-mingw32/include"
> ],
> "limitSymbolsToIncludedHeaders": true
> },
> "limitSymbolsToIncludedHeaders": true,
> "databaseFilename": "",
> "compilerArgs": [],
> "compilerPath": "D:/MinGw_x86_64/mingw64/bin/gcc.exe"
> }
> ],
> "version": 4
> }
> ```
>
> ```json
> launch.json
> {
> "name": "C/C++",
> "type": "cppdbg",
> "request": "launch",
> "program": "${workspaceFolder}/ExeFile/helloworld.exe",
> "args": [],
> "stopAtEntry": false,
> "cwd": "${workspaceFolder}",
> "environment": [],
> "externalConsole": false,
> "MIMode": "gdb",
> "miDebuggerPath": "D:\\MinGw_x86_64\\mingw64\\bin\\gdb.exe",
> "setupCommands": [
> {
> "description": "为 gdb 启用整齐打印",
> "text": "-enable-pretty-printing",
> "ignoreFailures": true
> }
> ]
> }
> ```
>
> ```json
> task.json
> {
> "name": "C/C++",
> "type": "cppdbg",
> "request": "launch",
> "program": "${workspaceFolder}/ExeFile/helloworld.exe",
> "args": [],
> "stopAtEntry": false,
> "cwd": "${workspaceFolder}",
> "environment": [],
> "externalConsole": false,
> "MIMode": "gdb",
> "miDebuggerPath": "D:\\MinGw_x86_64\\mingw64\\bin\\gdb.exe",
> "setupCommands": [
> {
> "description": "为 gdb 启用整齐打印",
> "text": "-enable-pretty-printing",
> "ignoreFailures": true
> }
> ]
> }
> ```
>
> 版本二：
>
> ```json
> tasks.json
> {
> "version": "2.0.0",
> "tasks": [
>  { //这个大括号里是‘构建（build）’任务
>    "label": "CPPbuild", //任务名称，可以更改，不过不建议改
>    "type": "shell", //任务类型，process是vsc把预定义变量和转义解析后直接全部传给command；shell相当于先打开shell再输入命令，所以args还会经过shell再解析一遍
>    //##################################可变##################################//
>    "command": "g++", //编译命令，这里是gcc，编译c++的话换成g++
>    "args": [ //方括号里是传给gcc命令的一系列参数，用于实现一些功能
>      //-----------------------------可变-----------------------------//
>      "${file}", //单文件程序编译，指定要编译的是当前文件
>      //"${fileDirname}\\*.c",       //多文件编译编译，把当前文件夹下文件全部编译，写c++把 *.c 换成 *.cpp
>      "-o", //指定输出文件的路径和名称
>      //-----------------------------可变-----------------------------//
>      "${fileDirname}\\bin\\${fileBasenameNoExtension}.exe", //承接上一步的-o，让可执行文件输出到源码文件所在的文件夹下的bin文件夹内，并且让它的名字和源码文件相同
>      "-g", //生成和调试有关的信息
>      //"-Wall", // 开启额外警告
>      //"-static-libgcc",  // 静态链接libgcc
>      "-fexec-charset=GBK", // 生成的程序使用GBK编码，不加这一条会导致Win下输出中文乱码
>      //##################################可变##################################//
>      "-std=c++11", // 语言标准，可根据自己的需要进行修改，写c++要换成c++的语言标准，比如c++11
>    ],
>    "group": { //group表示‘组’，我们可以有很多的task，然后把他们放在一个‘组’里
>      "kind": "build", //表示这一组任务类型是构建
>      "isDefault": true //表示这个任务是当前这组任务中的默认任务
>    },
>    "presentation": { //执行这个任务时的一些其他设定
>      "echo": true, //表示在执行任务时在终端要有输出
>      "reveal": "always", //执行任务时是否跳转到终端面板，可以为always，silent，never
>      "focus": false, //设为true后可以使执行task时焦点聚集在终端，但对编译来说，设为true没有意义，因为运行的时候才涉及到输入
>      "panel": "shared" //每次执行这个task时都新建一个终端面板，也可以设置为share，共用一个面板，不过那样会出现‘任务将被终端重用’的提示，比较烦人
>    },
>    "problemMatcher": "$gcc" //捕捉编译时编译器在终端里显示的报错信息，将其显示在vscode的‘问题’面板里
>  },
> 
>  { //这个大括号里是‘构建（build）’任务
>    "label": "Cbuild", //任务名称，可以更改，不过不建议改
>    "type": "shell", //任务类型，process是vsc把预定义变量和转义解析后直接全部传给command；shell相当于先打开shell再输入命令，所以args还会经过shell再解析一遍
>    //##################################可变##################################//
>    "command": "gcc", //编译命令，这里是gcc，编译c++的话换成g++
>    "args": [ //方括号里是传给gcc命令的一系列参数，用于实现一些功能
>      //-----------------------------可变-----------------------------//
>      "${file}", //单文件程序编译，指定要编译的是当前文件
>      //"${fileDirname}\\*.c",       //多文件编译编译，把当前文件夹下文件全部编译，写c++把 *.c 换成 *.cpp
>      "-o", //指定输出文件的路径和名称
>      //-----------------------------可变-----------------------------//
>      "${fileDirname}\\bin\\${fileBasenameNoExtension}.exe", //承接上一步的-o，让可执行文件输出到源码文件所在的文件夹下的bin文件夹内，并且让它的名字和源码文件相同
>      "-g", //生成和调试有关的信息
>      //"-Wall", // 开启额外警告
>      //"-static-libgcc",  // 静态链接libgcc
>      "-fexec-charset=GBK", // 生成的程序使用GBK编码，不加这一条会导致Win下输出中文乱码
>      //##################################可变##################################//
>      "-std=c11", // 语言标准，可根据自己的需要进行修改，写c++要换成c++的语言标准，比如c++11
>    ],
>    "group": { //group表示‘组’，我们可以有很多的task，然后把他们放在一个‘组’里
>      "kind": "build", //表示这一组任务类型是构建
>      "isDefault": true //表示这个任务是当前这组任务中的默认任务
>    },
>    "presentation": { //执行这个任务时的一些其他设定
>      "echo": true, //表示在执行任务时在终端要有输出
>      "reveal": "always", //执行任务时是否跳转到终端面板，可以为always，silent，never
>      "focus": false, //设为true后可以使执行task时焦点聚集在终端，但对编译来说，设为true没有意义，因为运行的时候才涉及到输入
>      "panel": "shared" //每次执行这个task时都新建一个终端面板，也可以设置为share，共用一个面板，不过那样会出现‘任务将被终端重用’的提示，比较烦人
>    },
>    "problemMatcher": "$gcc" //捕捉编译时编译器在终端里显示的报错信息，将其显示在vscode的‘问题’面板里
>  },
> 
>  { //这个大括号里是‘运行(run)’任务，一些设置与上面的构建任务性质相同
>    "label": "Crun",
>    "type": "shell",
>    "dependsOn": "Cbuild", //任务依赖，因为要运行必须先构建，所以执行这个任务前必须先执行build任务，
>    //-----------------------------可变-----------------------------//
>    "command": "${fileDirname}\\bin\\${fileBasenameNoExtension}.exe", //执行exe文件，只需要指定这个exe文件在哪里就好
>    "group": {
>      "kind": "test", //这一组是‘测试’组，将run任务放在test组里方便我们用快捷键执行
>      "isDefault": true
>    },
>    "presentation": {
>      "echo": true,
>      "reveal": "always",
>      "focus": true, //这个就设置为true了，运行任务后将焦点聚集到终端，方便进行输入
>      "panel": "shared" //每次执行这个task时都新建一个终端面板，也可以设置为share，共用一个面板，不过那样会出现‘任务将被终端重用’的提示，比较烦人
>    }
>  },
>  { //这个大括号里是‘运行(run)’任务，一些设置与上面的构建任务性质相同
>    "label": "CPPrun",
>    "type": "shell",
>    "dependsOn": "CPPbuild", //任务依赖，因为要运行必须先构建，所以执行这个任务前必须先执行build任务，
>    //-----------------------------可变-----------------------------//
>    "command": "${fileDirname}\\bin\\${fileBasenameNoExtension}.exe", //执行exe文件，只需要指定这个exe文件在哪里就好
>    "group": {
>      "kind": "test", //这一组是‘测试’组，将run任务放在test组里方便我们用快捷键执行
>      "isDefault": true
>    },
>    "presentation": {
>      "echo": true,
>      "reveal": "always",
>      "focus": true, //这个就设置为true了，运行任务后将焦点聚集到终端，方便进行输入
>      "panel": "shared" //每次执行这个task时都新建一个终端面板，也可以设置为share，共用一个面板，不过那样会出现‘任务将被终端重用’的提示，比较烦人
>    }
>  }
> ]
> }
> ```
>
> ```json
> lunch.json
> {
>  // 使用 IntelliSense 了解相关属性。 
>  // 悬停以查看现有属性的描述。
>  // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
>  "version": "0.2.0",
>  "configurations": [
>      {
>          "name": "Python",
>          "type": "python",
>          "request": "launch",
>          "program": "${file}",
>          "stopOnEntry": false,
>          "console": "integratedTerminal",
>          "cwd": "${workspaceFolder}"
>      },
>      {
>          "type": "java",
>          "name": "Java",
>          "request": "launch",
>          "mainClass": "${file}",
>          "stopOnEntry": false,
>          "cwd": "${workspaceFolder}",
>          "console": "integratedTerminal"
>      },
>      {
>          "name": "Go",
>          "type": "go",
>          "request": "launch",
>          "mode": "debug",
>          "program": "${file}"
>      },
> 
>      {
>          "name": "Javascript",
>          "program": "${file}",
>          "request": "launch",
>          "skipFiles": [
>              "<node_internals>/**"
>          ],
>          "type": "pwa-node"
>      },
> 
>      { //C
>          "name": "C", // 配置名称
>          "type": "cppdbg", // 配置类型，cppdbg对应cpptools提供的调试功能；可以认为此处只能是cppdbg
>          "request": "launch", // 请求配置类型，可以为launch（启动）或attach（附加）
>          //-----------------------------可变-----------------------------//
>          "program": "${fileDirname}\\bin\\${fileBasenameNoExtension}.exe", // 将要进行调试的程序的路径
>          "args": [], // 程序调试时传递给程序的命令行参数，这里设为空即可
>          "stopAtEntry": false, // 设为true时程序将暂停在程序入口处，相当于在main上打断点
>          "cwd": "${fileDirname}", // 调试程序时的工作目录，此处为源码文件所在目录
>          "environment": [], // 环境变量，这里设为空即可
>          "externalConsole": false, // 为true时使用单独的cmd窗口，跳出小黑框；设为false则是用vscode的内置终端，建议用内置终端
>          "internalConsoleOptions": "neverOpen", // 如果不设为neverOpen，调试时会跳到“调试控制台”选项卡，新手调试用不到
>          "MIMode": "gdb", // 指定连接的调试器，gdb是minGW中的调试程序
>          //-----------------------------可变-----------------------------//
>          "miDebuggerPath": "D:\\MinGw_x86_64\\mingw64\\bin\\gdb.exe", // 指定调试器所在路径，如果你的minGW装在别的地方，则要改成你自己的路径，注意间隔是\\
>          //-----------------------------可变-----------------------------//
>          "preLaunchTask": "Crun" // 调试开始前执行的任务，与tasks.json的label相对应，build:在调试前要编译构建。run:在调试前自动构建再运行
>      },
>      {//C++
>      "name": "CPP", // 配置名称
>      "type": "cppdbg", // 配置类型，cppdbg对应cpptools提供的调试功能；可以认为此处只能是cppdbg
>      "request": "launch", // 请求配置类型，可以为launch（启动）或attach（附加）
>      //-----------------------------可变-----------------------------//
>      "program": "${fileDirname}\\bin\\${fileBasenameNoExtension}.exe", // 将要进行调试的程序的路径
>      "args": [], // 程序调试时传递给程序的命令行参数，这里设为空即可
>      "stopAtEntry": false, // 设为true时程序将暂停在程序入口处，相当于在main上打断点
>      "cwd": "${fileDirname}", // 调试程序时的工作目录，此处为源码文件所在目录
>      "environment": [], // 环境变量，这里设为空即可
>      "externalConsole": false, // 为true时使用单独的cmd窗口，跳出小黑框；设为false则是用vscode的内置终端，建议用内置终端
>      "internalConsoleOptions": "neverOpen", // 如果不设为neverOpen，调试时会跳到“调试控制台”选项卡，新手调试用不到
>      "MIMode": "gdb", // 指定连接的调试器，gdb是minGW中的调试程序
>      //-----------------------------可变-----------------------------//
>      "miDebuggerPath": "D:\\MinGw_x86_64\\mingw64\\bin\\gdb.exe", // 指定调试器所在路径，如果你的minGW装在别的地方，则要改成你自己的路径，注意间隔是\\
>      //-----------------------------可变-----------------------------//
>      "preLaunchTask": "CPPrun" // 调试开始前执行的任务，与tasks.json的label相对应，build:在调试前要编译构建。run:在调试前先构建再运行
>      },
>  ]
> }
> ```
>
> ```json
> c_cpp_properties.json 可选
> 
> {
> "configurations": [
> {
> "name": "Win32",
> "includePath": [
>     "${workspaceRoot}",
>     "${workspaceFolder}/**",
>     "D:/MinGw_x86_64/mingw64/lib/gcc/x86_64-w64-mingw32/8.1.0/include/c++",
>     "D:/MinGw_x86_64/mingw64/lib/gcc/x86_64-w64-mingw32/8.1.0/include/c++/x86_64-w64-mingw32",
>     "D:/MinGw_x86_64/mingw64/lib/gcc/x86_64-w64-mingw32/8.1.0/include/c++/backward",
>     "D:/MinGw_x86_64/mingw64/lib/gcc/x86_64-w64-mingw32/8.1.0/include",
>     "D:/MinGw_x86_64/mingw64/lib/gcc/x86_64-w64-mingw32/8.1.0/include/c++/tr1",
>     "D:/MinGw_x86_64/mingw64/x86_64-w64-mingw32/include"
> ],
> "defines": [
>     "_DEBUG",
>     "UNICODE",
>     "__GNUC__=6",
>     "__cdecl=__attribute__((__cdecl__))"
> ],
> "browse": {
>     "path": [
>         "${workspaceRoot}",
>         "D:/MinGw_x86_64/mingw64/lib/gcc/x86_64-w64-mingw32/8.1.0/include/c++",
>         "D:/MinGw_x86_64/mingw64/lib/gcc/x86_64-w64-mingw32/8.1.0/include/c++/x86_64-w64-mingw32",
>         "D:/MinGw_x86_64/mingw64/lib/gcc/x86_64-w64-mingw32/8.1.0/include/c++/backward",
>         "D:/MinGw_x86_64/mingw64/lib/gcc/x86_64-w64-mingw32/8.1.0/include",
>         "D:/MinGw_x86_64/mingw64/lib/gcc/x86_64-w64-mingw32/8.1.0/include/c++/tr1",
>         "D:/MinGw_x86_64/mingw64/x86_64-w64-mingw32/include"
>     ],
>     "limitSymbolsToIncludedHeaders": true
> },
> "compilerArgs": [],
> // -----------------可变---------------------//
> "cStandard": "c11",
> "cppStandard": "gnu++14",
> "intelliSenseMode": "clang-x64",
> "compilerPath": "D:/MinGw_x86_64/mingw64/bin/gcc.exe"
> }
> ],
> "version": 4
> }
> ```
>
> 

### waifu2x

### Wechat

> 更改文件夹

### Zotero

> 修改文件存储路径

### ZYtrans Xtransltor

> 登录
>
> 百度翻译API：20200807000535541	KeCMk7T53dhu9qOcSfuR

### DDM 

>  intel 核显驱动,DDM驱动



### WSL

>开启系统支持
>
>```
>dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
>
>dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
>```

### Sublime

>
>
>```
>----- BEGIN LICENSE -----
>Member J2TeaM
>Single User License
>EA7E-1011316
>D7DA350E 1B8B0760 972F8B60 F3E64036
>B9B4E234 F356F38F 0AD1E3B7 0E9C5FAD
>FA0A2ABE 25F65BD8 D51458E5 3923CE80
>87428428 79079A01 AA69F319 A1AF29A4
>A684C2DC 0B1583D4 19CBD290 217618CD
>5653E0A0 BACE3948 BB2EE45E 422D2C87
>DD9AF44B 99C49590 D2DBDEE1 75860FD2
>8C8BB2AD B2ECE5A4 EFC08AF2 25A9B864
>------ END LICENSE ------
>```
>
>插件： https://zhuanlan.zhihu.com/p/91942738

### PS

> AdobeSerialization.exe 管理员连续运行三次，每次间隔20秒。

### 关闭触控板

> 安装触控板驱动

### 关闭自动更新

> win + r 运行 services.msc -> windows update -> 常规：启动类型：禁用，停止 -> 恢复：无操作
>
> win + r 运行 gpedit.msc -> 计算机配置 -> 管理模板 -> Windows组件 -> Windows更新 -> 配置自动更新 -> 已禁用
>
> NoUpdate.exe

### 启用模糊拼音

### 关闭盖子无操作

> 控制面板  -> 硬件 和声音 -> 电源选项 

### 高性能模式

> 控制面板  -> 硬件 和声音 -> 电源选项 -> 创建电源选项 -> 高性能

### 图标

> 个性化 -> 主题 -> 桌面图标

### 去除快捷方式箭头

> win+R ==> regedit ==> HKEY_CLASSES_ROOT ==> lnkfile ==> 删除lsShortcut



### Steam

>
>
>显卡驱动
>
>

### 商店

> 

### todoist



### 键盘

>   ```
>   sc config i8042prt start= disabled
>   sc config i8042prt start= demand
>   ```