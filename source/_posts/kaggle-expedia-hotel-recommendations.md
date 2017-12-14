title: kaggle-expedia-hotel-recommendations学习笔记
tags:
  - kaggle
  - 随机森林
  - python
categories:
  - kaggle
date: 2017-12-05 16:31:00
---

# 题目地址
https://www.kaggle.com/c/expedia-hotel-recommendations#description

# 学习地址
https://www.dataquest.io/blog/kaggle-tutorial/

# 目标
- 初步了解机器学习流程
- 通过实际代码,了解python代码的语法

<!-- more -->

# 学习心得
- 数据探索
``` python
import pandas as pd

destinations = pd.read_csv("destinations.csv")
test = pd.read_csv("test.csv")
train = pd.read_csv("train.csv")

#train文件有4个g大,读了半天都不出来
```
    + 数据量
    + 数据分布
- 目标变量
    + 目标变量是什么
    + 探索目标变量
    + 探索用户id 
- 采样统计
    + 添加时间和日期
    + 挑选1000名用户
    + 区分测试集和验证集
    + 移除无关数据(click 事件)
- 一个简单的算法
    + 生成预测
```
predictions = [most_common_clusters for i in range(t2.shape[0])]
# 不是很理解 [] 和 shape[0]的含义
# shape()方法返回 行数 和 维度数 .其中:0:行数 1:维度数
# pyhton中的遍历:
for iterating_var in sequence:
   statements(s)
# 但是 for前面的most_common_clusters 是什么意思?
# 通过测试了解到是复制一个变量n遍的意思
aa=["abc" for i in range(10)]
print(aa)
输出了10个 abc
```
    + 评估效果
```
import ml_metrics as metrics
target = [[l] for l in t2["hotel_cluster"]]
metrics.mapk(target, predictions, k=5)
# 不是很理解 [l]代表什么
# 把t2["hotel_cluster"]列表中的变量l,外面 前一层 列表 []
xx=[1,2,3,4]
target = [[l] for l in xx]
print(target)
[[1], [2], [3], [4]]
```
    + 查找相关因素
- 更近一步
    + 生成特征
        * 从destinations生成特征
```
from sklearn.decomposition import PCA

pca = PCA(n_components=3)
dest_small = pca.fit_transform(destinations[["d{0}".format(i + 1) for i in range(149)]])
dest_small = pd.DataFrame(dest_small)
dest_small["srch_destination_id"] = destinations["srch_destination_id"]
# 解释 取d1-d149字段
print(["d{0}".format(i + 1) for i in range(149)])
['d1', 'd2', 'd3', 'd4', 'd5', 'd6', 'd7', 'd8', 'd9', 'd10', 'd11', 'd12', 'd13', 'd14', 'd15', 'd16', 'd17', 'd18', 'd19', 'd20', 'd21', 'd22', 'd23', 'd24', 'd25', 'd26', 'd27', 'd28', 'd29', 'd30', 'd31', 'd32', 'd33', 'd34', 'd35', 'd36', 'd37', 'd38', 'd39', 'd40', 'd41', 'd42', 'd43', 'd44', 'd45', 'd46', 'd47', 'd48', 'd49', 'd50', 'd51', 'd52', 'd53', 'd54', 'd55', 'd56', 'd57', 'd58', 'd59', 'd60', 'd61', 'd62', 'd63', 'd64', 'd65', 'd66', 'd67', 'd68', 'd69', 'd70', 'd71', 'd72', 'd73', 'd74', 'd75', 'd76', 'd77', 'd78', 'd79', 'd80', 'd81', 'd82', 'd83', 'd84', 'd85', 'd86', 'd87', 'd88', 'd89', 'd90', 'd91', 'd92', 'd93', 'd94', 'd95', 'd96', 'd97', 'd98', 'd99', 'd100', 'd101', 'd102', 'd103', 'd104', 'd105', 'd106', 'd107', 'd108', 'd109', 'd110', 'd111', 'd112', 'd113', 'd114', 'd115', 'd116', 'd117', 'd118', 'd119', 'd120', 'd121', 'd122', 'd123', 'd124', 'd125', 'd126', 'd127', 'd128', 'd129', 'd130', 'd131', 'd132', 'd133', 'd134', 'd135', 'd136', 'd137', 'd138', 'd139', 'd140', 'd141', 'd142', 'd143', 'd144', 'd145', 'd146', 'd147', 'd148', 'd149']
```
        * 生成其他特征
```
def calc_fast_features(df):
    df["date_time"] = pd.to_datetime(df["date_time"])
    df["srch_ci"] = pd.to_datetime(df["srch_ci"], format='%Y-%m-%d', errors="coerce")
    # 参数errors="coerce" 遇到错误可以赋值为空。
    df["srch_co"] = pd.to_datetime(df["srch_co"], format='%Y-%m-%d', errors="coerce")
    
    props = {}
    for prop in ["month", "day", "hour", "minute", "dayofweek", "quarter"]:
        props[prop] = getattr(df["date_time"].dt, prop)
    
    carryover = [p for p in df.columns if p not in ["date_time", "srch_ci", "srch_co"]]
    for prop in carryover:
        props[prop] = df[prop]
    
    date_props = ["month", "day", "dayofweek", "quarter"]
    for prop in date_props:
        props["ci_{0}".format(prop)] = getattr(df["srch_ci"].dt, prop)
        props["co_{0}".format(prop)] = getattr(df["srch_co"].dt, prop)
    props["stay_span"] = (df["srch_co"] - df["srch_ci"]).astype('timedelta64[h]')
        
    ret = pd.DataFrame(props)
    
    ret = ret.join(dest_small, on="srch_destination_id", how='left', rsuffix="dest")
    ret = ret.drop("srch_destination_iddest", axis=1)
    return ret

df = calc_fast_features(t1)
df.fillna(-1, inplace=True)
```

```
新的列:
[                  'channel',                    'ci_day',
                    'ci_dayofweek',                  'ci_month',
                      'ci_quarter',                       'cnt',
                          'co_day',              'co_dayofweek',
                        'co_month',                'co_quarter',
                             'day',                 'dayofweek',
                   'hotel_cluster',           'hotel_continent',
                   'hotel_country',              'hotel_market',
                            'hour',                'is_booking',
                       'is_mobile',                'is_package',
                          'minute',                     'month',
       'orig_destination_distance',            'posa_continent',
                         'quarter',                 'site_name',
                 'srch_adults_cnt',         'srch_children_cnt',
             'srch_destination_id',  'srch_destination_type_id',
                     'srch_rm_cnt',                 'stay_span',
                         'user_id',        'user_location_city',
           'user_location_country',      'user_location_region',
                            'year',                           0,
                                 1,                           2]
```

    + 机器学习
        * 随机森林
```
predictors = [c for c in df.columns if c not in ["hotel_cluster"]]
from sklearn import cross_validation
from sklearn.ensemble import RandomForestClassifier

clf = RandomForestClassifier(n_estimators=10, min_weight_fraction_leaf=0.1)
scores = cross_validation.cross_val_score(clf, df[predictors], df['hotel_cluster'], cv=3)
scores
```
        * 二分分类
```
from sklearn.ensemble import RandomForestClassifier
from sklearn.cross_validation import KFold
from itertools import chain

all_probs = []
unique_clusters = df["hotel_cluster"].unique()
for cluster in unique_clusters:
    df["target"] = 1
    df["target"][df["hotel_cluster"] != cluster] = 0
    predictors = [col for col in df if col not in ['hotel_cluster', "target"]]
    probs = []
    cv = KFold(len(df["target"]), n_folds=2)
    # 交叉验证(CrossValidation)
    # 方法思想是为了在不动用测试集之前，就评估一下模型是否过于复杂而引起过度拟合
    clf = RandomForestClassifier(n_estimators=10, min_weight_fraction_leaf=0.1)
    for i, (tr, te) in enumerate(cv):
    # 不是很理解 上面一句
        clf.fit(df[predictors].iloc[tr], df["target"].iloc[tr])
        preds = clf.predict_proba(df[predictors].iloc[te])
        probs.append([p[1] for p in preds])
    full_probs = chain.from_iterable(probs)
    all_probs.append(list(full_probs))

prediction_frame = pd.DataFrame(all_probs).T
prediction_frame.columns = unique_clusters
def find_top_5(row):
    return list(row.nlargest(5).index)

preds = []
for index, row in prediction_frame.iterrows():
    preds.append(find_top_5(row))

metrics.mapk([[l] for l in t2.iloc["hotel_cluster"]], preds, k=5)
```
- 在目的地下的最受欢迎的酒店选择
```
def make_key(items):
    return "_".join([str(i) for i in items])

match_cols = ["srch_destination_id"]
cluster_cols = match_cols + ['hotel_cluster']
groups = t1.groupby(cluster_cols)
top_clusters = {}
for name, group in groups:
    clicks = len(group.is_booking[group.is_booking == False])
    bookings = len(group.is_booking[group.is_booking == True])
    
    score = bookings + .15 * clicks
    
    clus_name = make_key(name[:len(match_cols)])
    if clus_name not in top_clusters:
        top_clusters[clus_name] = {}
    top_clusters[clus_name][name[-1]] = score



import operator

cluster_dict = {}
for n in top_clusters:
    tc = top_clusters[n]
    top = [l[0] for l in sorted(tc.items(), key=operator.itemgetter(1), reverse=True)[:5]]
    cluster_dict[n] = top
```
    + 给予目的地,进行预测
```
preds = []
for index, row in t2.iterrows():
    key = make_key([row[m] for m in match_cols])
    if key in cluster_dict:
        preds.append(cluster_dict[key])
    else:
        preds.append([])

```
    + 评估错误
```
metrics.mapk([[l] for l in t2["hotel_cluster"]], preds, k=5)
```
- 更好的结果
    + 根据用户
```
match_cols = ['user_location_country', 'user_location_region', 'user_location_city', 'hotel_market', 'orig_destination_distance']

groups = t1.groupby(match_cols)
    
def generate_exact_matches(row, match_cols):
    index = tuple([row[t] for t in match_cols])
    try:
        group = groups.get_group(index)
    except Exception:
        return []
    clus = list(set(group.hotel_cluster))
    return clus

exact_matches = []
for i in range(t2.shape[0]):
    exact_matches.append(generate_exact_matches(t2.iloc[i], match_cols))
```
    + 合并预测
```
def f5(seq, idfun=None): 
    if idfun is None:
        def idfun(x): return x
    seen = {}
    result = []
    for item in seq:
        marker = idfun(item)
        if marker in seen: continue
        seen[marker] = 1
        result.append(item)
    return result
    
full_preds = [f5(exact_matches[p] + preds[p] + most_common_clusters)[:5] for p in range(len(preds))]
mapk([[l] for l in t2["hotel_cluster"]], full_preds, k=5)
```
- 生成提交文件
```
write_p = [" ".join([str(l) for l in p]) for p in full_preds]
write_frame = ["{0},{1}".format(t2["id"][i], write_p[i]) for i in range(len(full_preds))]
write_frame = ["id,hotel_clusters"] + write_frame
with open("predictions.csv", "w+") as f:
    f.write("\n".join(write_frame))
```
- 总结
    + 下一步


