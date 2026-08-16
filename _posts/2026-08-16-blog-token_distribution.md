# Token Distribution

动机：统计模型生成的token概率分布，用来更好地理解模型的预测过程。


分不同领域（数学、代码、通用）统计token概率分布
一些有趣的发现（通过case study）：
数学领域：
低熵token：
- 公式/答案的token

高熵token：
- 模型生成自然语言描述的token

代码领域：
高熵token：
- 开头部分类似于special token
    - 类似于<repo_name>, <branch_name>, <commit_id>, 等，使用"<"预测"repo"，模型预测不确定
- 代码中的注释、字符串等，部分确定部分不确定

低熵token：
- 代码中的变量、函数、类等



通用领域：
高熵token：
- 模型预测数字的部分

低熵模式：
- 现实生活的语法：
    - 例如把一个单词分割为几个token，在预测后面的token的时候模型确定。
