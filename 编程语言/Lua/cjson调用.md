## cjson调用

### decode

``` lua
local cjson = require "cjson"
local sampleJson = [[{"age":"23","testArray":{"array":[8,9,11,14,25]},"Himi":"himigame.com"}]];
--解析json字符串
local data = cjson.decode(sampleJson);
--打印json字符串中的age字段
print(data["age"]);
--打印数组中的第一个值(lua默认是从0开始计数)
print(data["testArray"]["array"][1]);
```

### encode

``` lua
local cjson = require "cjson"
local retTable = {};    --最终产生json的表
--顺序数值
local intDatas = {};
intDatas[1] = 100;
intDatas[2] = "100";
--数组
local aryDatas = {};
aryDatas[1] = {};
aryDatas[1]["键11"] = "值11";
aryDatas[1]["键12"] = "值12";
aryDatas[2] = {};
aryDatas[2]["键21"] = "值21";
aryDatas[2]["键22"] = "值22";
--对Table赋值
retTable["键1"] = "值1";
retTable[2] = 123;
retTable["int_datas"] = intDatas;
retTable["aryDatas"] = aryDatas;
--将表数据编码成json字符串
local jsonStr = cjson.encode(retTable);
print(jsonStr);
--结果是:{"int_datas":[100,"100"],"2":123,"键1":"值1","aryDatas":[{"键12":"值12","键11":"值11"},{"键21":"值21","键22":"值22"}]}

```