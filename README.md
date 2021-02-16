# tianchi-multi-task-nlp
NLP中文预训练模型泛化能力挑战赛

## 环境介绍

```
机器信息：NVIDIA-SMI 440.33.01    Driver Version: 440.33.01    CUDA Version: 10.2
pytorch 版本 1.6.0

机器信息：NVIDIA-SMI 460.32.03    Driver Version: 460.32.03    CUDA Version: 11.2
pytorch 版本 1.7.1
```

python依赖：
```
pip install transformers
```

## 运行过程

1. 下载Bert全权重

1. 下载 https://huggingface.co/bert-base-chinese/tree/main 下载config.json vocab.txt pytorch_model.bin，把这三个文件放进tianchi-multi-task-nlp/bert_pretrain_model文件夹下。
2. 下载比赛数据集，把三个数据集分别放进 `tianchi-multi-task-nlp/tianchi_datasets/数据集名字/` 下面：
  - OCEMOTION/total.csv: http://tianchi-competition.oss-cn-hangzhou.aliyuncs.com/531841/OCEMOTION_train1128.csv
  - OCEMOTION/test.csv: http://tianchi-competition.oss-cn-hangzhou.aliyuncs.com/531841/b/ocemotion_test_B.csv
  - TNEWS/total.csv: http://tianchi-competition.oss-cn-hangzhou.aliyuncs.com/531841/TNEWS_train1128.csv
  - TNEWS/test.csv: http://tianchi-competition.oss-cn-hangzhou.aliyuncs.com/531841/b/tnews_test_B.csv
  - OCNLI/total.csv: http://tianchi-competition.oss-cn-hangzhou.aliyuncs.com/531841/OCNLI_train1128.csv
  - OCNLI/test.csv: http://tianchi-competition.oss-cn-hangzhou.aliyuncs.com/531841/b/ocnli_test_B.csv

文件目录样例：
```
tianchi-multi-task-nlp/tianchi_datasets/OCNLI/total.csv
tianchi-multi-task-nlp/tianchi_datasets/OCNLI/test.csv
```

3. 分开训练集和验证集，默认验证集是各3000条数据，参数可以自己修改：
```
python ./generate_data.py
```
4. 训练模型：
```
python ./train.py
```
会保存验证集上平均f1分数最高的模型到 ./saved_best.pt

5. 用训练好的模型 ./saved_best.pt 生成结果：
```
python ./inference.py
```

6. 打包预测结果。
```
zip -r ./result.zip ./*.json
```
7. 生成Docker并进行提交。

## 改进思路

1. 修改 calculate_loss.py 改变loss的计算方式，从平衡子任务难度以及各子任务类别样本不均匀入手
2. 修改 net.py 改变模型的结构
3. 使用 cleanlab 等工具对训练数据进行清洗
4. 使用 k-fold 交叉验证
5. 对训练好的模型再在完整数据集（包括验证集和训练集）上用小的学习率训练一个epoch
6. 调整bathSize和a_step，变更梯度累计的程度，当前是batchSize=16，a_step=16，累计256个数据的梯度再一次性更新权重
7. 做文本数据增强
8. 用 chinese-roberta-wwm-ext 作为预训练模型
