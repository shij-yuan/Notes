# Arrays.sort

## 排序选择

- 检查数组大小
    - 若大于阈值
        - 评估无序程度（递增或递减子序列的个数）
            - 若基本有序，使用优化归并
            - 若基本无序，使用优化快排
    - 若小于阈值
        - 小于插入排序阈值
            - 使用插入排序
        - 大于插入排序阈值
            - 使用优化快排

## 优化快排

三向切分

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gig0ecvxgqj30hs044glo.jpg)

双轴快排

![](https://img-blog.csdn.net/20170504130926371?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvSG9sbW9meQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



## 堆排序

父节点 = (x - 1) / 2

左子节点 = x * 2 + 1

右子节点 = x * 2 + 2

### 插入（建堆）

每一个插入的节点放在树的最后一个位置，与其父节点比较

### 调整（shiftUp）

只能删除头节点，将最后一个节点替代头节点，然后与左右子节点中小的节点进行交换