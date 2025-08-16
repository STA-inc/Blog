在运行SecGPT时报错：
Tokenizer class Qwen2Tokenizer does not exist or is not currently imported
原因：transformers版本太低，导致分词器Qwen2Tokenizer没在transformers里面
解决办法：
将python升级到3.8，然后将transformers升级到4.40.0：
![在这里插入图片描述](../image/9ba694122ca9313725050c0f0092e7d8.png)
