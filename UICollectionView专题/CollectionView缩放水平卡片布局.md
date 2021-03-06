#概述

本篇一起来学习如何使用UICollectionView来实现水平滚动的缩放式卡片布局，就像Nice App中的卡片布局。

前一篇中讲了如何实现[CollectionView旋转水平卡片布局](http://www.henishuo.com/collectiontion-card-layout/)，如果还没有阅读过，不防先看看再继续往下阅读。

#实现效果

![image](http://www.henishuo.com/wp-content/uploads/2016/03/cardscaledemo.gif)

#实现思路

从Demo效果图中，可以看出来，主要是缩放系数的计算。对于不同距离的cell，其缩放系数要变化，以便整体协调显示。

所以，我们必须重写-layoutAttributesForElementsInRect:方法来实现所有当前可见的cell的attributes。

计算比例，通过获取当前偏移rect的最小坐标x，再与atribute的中心x相减，再除以高度，就是高度的缩放倍数scaleForDistance。

最后，通过一个公式来计算缩放的Y系数。公式为：

```
scale = 1 + scaleFactor * (1 - fabs(scaleForDistance))
```

scaleFactor因子可以自由调整，值越大，显示就越大。

#核心代码

```
#pragma mark - Override
- (void)prepareLayout {
  self.scrollDirection = UICollectionViewScrollDirectionHorizontal;
  self.minimumLineSpacing = 20;
  //  self.minimumInteritemSpacing = 20;
  self.sectionInset = UIEdgeInsetsMake(0, 30, 0, 30);
  self.itemSize = CGSizeMake(self.collectionView.frame.size.width - 60,
                             self.collectionView.frame.size.height - 180);
  [super prepareLayout];
}

- (BOOL)shouldInvalidateLayoutForBoundsChange:(CGRect)newBounds {
  return YES;
}

- (NSArray<UICollectionViewLayoutAttributes *> *)layoutAttributesForElementsInRect:(CGRect)rect {
  NSArray *superAttributes = [super layoutAttributesForElementsInRect:rect];
  NSArray *attributes = [[NSArray alloc] initWithArray:superAttributes copyItems:YES];
  
  CGRect visibleRect = CGRectMake(self.collectionView.contentOffset.x,
                                  self.collectionView.contentOffset.y,
                                  self.collectionView.frame.size.width,
                                  self.collectionView.frame.size.height);
  CGFloat offset = CGRectGetMidX(visibleRect);
  
  [attributes enumerateObjectsUsingBlock:^(UICollectionViewLayoutAttributes *attribute, NSUInteger idx, BOOL * _Nonnull stop) {
    CGFloat distance = offset - attribute.center.x;
    // 越往中心移动，值越小，那么缩放就越小，从而显示就越大
    // 同样，超过中心后，越往左、右走，缩放就越大，显示就越小
    CGFloat scaleForDistance = distance / self.itemSize.height;
    // 0.2可调整，值越大，显示就越大
    CGFloat scaleForCell = 1 + 0.2 * (1 - fabs(scaleForDistance));
    
    // only scale y-axis
    attribute.transform3D = CATransform3DMakeScale(1, scaleForCell, 1);
    attribute.zIndex = 1;
  }];
  
  return attributes;
}
```

#实现pagingEnabled的样式

```
- (CGPoint)targetContentOffsetForProposedContentOffset:(CGPoint)proposedContentOffset withScrollingVelocity:(CGPoint)velocity {
  BOOL pagingEnabled = NO;
  if (pagingEnabled) {
    // 分页以1/3处
    if (proposedContentOffset.x > self.previousOffsetX + self.itemSize.width / 3.0) {
      self.previousOffsetX += self.collectionView.frame.size.width - self.minimumLineSpacing * 2;
    } else if (proposedContentOffset.x < self.previousOffsetX  - self.itemSize.width / 3.0) {
      self.previousOffsetX -= self.collectionView.frame.size.width - self.minimumLineSpacing * 2;
    }
    
    proposedContentOffset.x = self.previousOffsetX;
  } else {
    CGFloat x = proposedContentOffset.x / (self.itemSize.width + self.minimumLineSpacing);
    int base = (int)x;
    
    proposedContentOffset.x = base * (self.itemSize.width + self.minimumLineSpacing);
  }
  
  return proposedContentOffset;
}
```

这里是以1/3.0为分界，左、右的1/3作为分界线，超过才会切换过去！

#感谢

感谢评论的朋友们的一句话，点醒了笔者。对于不分页的情况下，其实只要使用当前的偏移x除(itemSize.width + minimumLineSpacing)就得到一个倍数，然后四舍五入。比如，4.3取整得到4，那么就是没有超过一半，就要往回滚。而4.6取整得到5，表示要滚动到下一个。所以在不分页的情况下，其实也是非常简单的。


#结尾

本篇文章经过朋友们的评论及反馈，可以说已经完善了！


#源代码

[CollectionViewDemos](https://github.com/CoderJackyHuang/CollectionViewDemos)

**提示：**本篇文章的demo对应于工程中的**Demo3-CardLayout-scale**分组。





