![渐变.gif](http://upload-images.jianshu.io/upload_images/2954364-5096825effb3ce78.gif?imageMogr2/auto-orient/strip)

实现过程简介:
- 界面组成:
底部是TableView(占据控制器的整个View)
顶部的一个View以及其内部的两个ImageView
中间的红色分隔条View

- 控件约束:
tableView:距上下左右均为0,与控制器View等大
顶部View:距上左右为0,然后定高(控制器需要获得这个高属性来修改它)
顶部View内部第一个ImageView:距上下左右均为0,与顶部View等大,拥有下拉放大效果显示模式应该选择UIViewContentModeScaleAspectFill(关于ImageView的图片显示模式,请见我的另一篇文章:http://www.jianshu.com/p/d38d3e6f3db4)
顶部View内部第二个ImageView:在父控件中水平居中,距下40,定高定宽(不会缩小变形)
红色分隔条View:紧贴顶部View,距左右为0,定高44

- ####设计思路:
首先要知道设置导航条的透明度是不能让导航条的透明度发生变化的(无效)
所以我们设置导航条背景图的透明度,来实现导航条看上去透明了的效果
另外导航条还有一个ShadowImage(直接隐藏掉好了)
是导航条下面的那根线:
![BackgroundImage和ShadowImage.png](http://upload-images.jianshu.io/upload_images/2954364-ed4688ba3f777d1c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
初始状态如下设置:
```
.
//设置tableView的contentInset 让tableView的第一行从红色View下部开始显示
    self.tableView.contentInset = UIEdgeInsetsMake(244, 0, 0, 0);
.
//设置无颜色ImageView给navigationBar,实现navigationBar透明效果
    [self.navigationController.navigationBar setBackgroundImage:[[UIImage alloc]init] forBarMetrics:UIBarMetricsDefault];
    [self.navigationController.navigationBar setShadowImage:[[UIImage alloc]init]];//导航条图片下面的线
.
//设置标题
    UILabel *title = [[UILabel alloc] init];
    title.text = @"我在渐变";
    [title sizeToFit];
    title.textColor = [UIColor colorWithWhite:0 alpha:0];
    self.navigationItem.titleView = title;
```
然后在ScrollView的滚动监听方法里面根据TableView滚动了多少来改变图片透明度:
```
-(void)scrollViewDidScroll:(UIScrollView *)scrollView{
//一堆计算过程:
    CGFloat offset = scrollView.contentOffset.y - originY;
    //当前值 - 原始值 = 偏移量
    CGFloat h = originH - offset;
    if (h<64) {
        h = 64;
    }
    
    CGFloat alpha = offset * 1 / 136.0;
    if (alpha >=1) {
        alpha = 0.99;
    }
//计算结束
.
    //修改顶部View的高度
    self.viewHeight.constant = h;

    //拿到标题 设置标题透明度
    UILabel *titleL = (UILabel *)self.navigationItem.titleView;
    titleL.textColor = [UIColor colorWithWhite:0 alpha:alpha];
  
    //修改导航栏的背景图片
    //把颜色生成图片
    UIColor *alphaColor = [UIColor colorWithWhite:1 alpha:alpha];
    //把颜色生成图片
    UIImage *alphaImage = [self imageWithColor:alphaColor];
    //修改导航条背景图片
    [self.navigationController.navigationBar setBackgroundImage:alphaImage forBarMetrics:UIBarMetricsDefault];
}
//根据颜色来生成一张图片
-(UIImage *)imageWithColor:(UIColor *)color
{
    // 描述矩形
    CGRect rect = CGRectMake(0.0f, 0.0f, 1.0f, 1.0f);
    // 开启位图上下文
    UIGraphicsBeginImageContext(rect.size);
    // 获取位图上下文
    CGContextRef context = UIGraphicsGetCurrentContext();
    // 使用color演示填充上下文
    CGContextSetFillColorWithColor(context, [color CGColor]);
    // 渲染上下文
    CGContextFillRect(context, rect);
    // 从上下文中获取图片
    UIImage *theImage = UIGraphicsGetImageFromCurrentImageContext();
    // 结束上下文
    UIGraphicsEndImageContext();
    return theImage;
}
```
谢谢阅读
有不合适的地方请指教
喜欢请点个赞
抱拳了!
