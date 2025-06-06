# apiVersioin


> K8s的apiVersion由group和version组成，不同的kind有对应的不同group，不同的group再根据不同的k8s版本有不同的version。

+ 选择apiVersion步骤
  - 1.确定kind：比如：namespace、deployment、service、pod等
  - 2.寻找group(选择出自己需要的kind对应的group)
	- kubectl api-resources -o wide