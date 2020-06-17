# Fix-ZE3kr-Cloudflare
当然,此问题早已经修复.可以去其仓库获取最新消息 https://github.com/ZE3kr/Cloudflare-CNAME-Setup

----------

其实和白嫖不白嫖的关系不大。白嫖一样能用，不过CF的API解析参数严格了一些。

## 问题出现在：

### 添加DNS记录：

actions/add_record.php 第20行往后  

```php
	$options = [
		'type' => $_POST['type'],
		'name' => $_POST['name'],
		'content' => $_POST['content'],
		'proxied' => $_POST['proxied'],
		'ttl' => intval($_POST['ttl']),
		'data' => $dns_data, 
	];

	if ($_POST['type'] == 'MX') {
		$options['priority'] = intval($_POST['priority']);
	}
	try {
		$dns = $adapter->post('zones/' . $_GET['zoneid'] . '/dns_records', $options);
```

上面的data在非CAA、SRV记录中是空的（见record_data.php中定义）。但是CF的API无法解析这个空数组，所以就无法添加DNS记录。

### 修改DNS记录：

actions/edit_record.php 第31行记录

巴拉巴拉巴拉吧（同样的问题，不再赘述）

## 解决：

传入之前删除这个空数组。

```php
$options = [
	'type' => $_POST['type'],
	'name' => $_POST['name'],
	'content' => $_POST['content'],
	'proxied' => $_POST['proxied'],
	'ttl' => intval($_POST['ttl']),
	'data' => $dns_data, 
];

if ($_POST['type'] == 'MX') {
	$options['priority'] = intval($_POST['priority']);
}
try {
	if(empty($options['data']))unset($options['data']);##就是这一行啦。
	$dns = $adapter->post('zones/' . $_GET['zoneid'] . '/dns_records', $options);
```
当然你也可以直接下载 

https://github.com/yumusb/Fix-ZE3kr-Cloudflare/blob/master/add_record.php 替换掉 actions/add_record.php

https://github.com/yumusb/Fix-ZE3kr-Cloudflare/blob/master/edit_record.php 替换掉 actions/edit_record.php

## 最后

如果有帮助到你，请不要吝啬。 http://33.al/donate
