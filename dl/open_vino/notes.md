Notes
---

1. NormalizeTI
2. RemoveConstOps
    删掉从模型文件里读数据的返回 常量类型数据 的算子
3. CreateConstNodesReplacement

1. 融合 fill-value = 0 的pad，
2. 融合 Mul-Add 为 Scaleshift
3. 融合 Add-Mul 为 Normalize
4. 融合 conv-bn 
5. 融合 Conv/FC-Mul/Add