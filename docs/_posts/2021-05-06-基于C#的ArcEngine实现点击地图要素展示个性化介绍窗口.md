---
layout: page
title: "基于C#的ArcEngine实现点击地图要素展示个性化介绍窗口"
permalink: /blog/20210506
---

本科的课程小实习，嘿嘿。
# 一、简介
这篇博文实现的功能是，在地图上选择一个要素，然后弹出它对应的信息窗口。比如我的实习主题是武汉大学内的历史建筑，我可以选择宋卿体育馆，像这样。
	![在这里插入图片描述](https://img-blog.csdnimg.cn/20210506173552330.png)
	然后点击“建筑介绍”，接着软件就会弹出像下面这样的窗口。
	![在这里插入图片描述](https://img-blog.csdnimg.cn/20210506173744697.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0xleGllbG92ZXNoYWly,size_16,color_FFFFFF,t_70)

	
# 二、实现介绍
## 2.1 要素的选择

```csharp
        private void 要素选择ToolStripMenuItem_Click(object sender, EventArgs e)
        {
            this.axMapControl1.CurrentTool = null;
            //Tool的定义和初始化
            ControlsSelectFeaturesToolClass pTool = new ESRI.ArcGIS.Controls.ControlsSelectFeaturesToolClass();
            //Tool通过ICommand与MapControl的关联
            pTool.OnCreate(this.axMapControl1.Object);
            //MapControl的当前工具设定为tool 
            this.axMapControl1.CurrentTool = pTool as ITool; 
        }

```
但是实现上面这个方法之后，你会惊讶地发现自己就仅仅是在地图上勾了一个要素而已，就是高亮了个好看！做到这里你可以给房地产开发商高亮标记告诉他们你要投资这里，这里是哪里不知道，总之一定要投。
## 2.2 已选要素的获取
你现在有了一块地，可是不知道它叫什么名字，没有控制权，除了钱和拿下它的决心你一无所有，OK，下面我来介绍怎么获取它的信息。

```csharp
//获得已选要素的feature对象（注意，是可以多选的，已选要素全部会存在pEnumFeature里面，可以一直Next()直到没有下一位）

  IMap map = axMapControl1.Map;
            ISelection selection = map.FeatureSelection;
            IEnumFeatureSetup iEnumFeatureSetup = (IEnumFeatureSetup)selection;
            iEnumFeatureSetup.AllFields = true;
            IEnumFeature pEnumFeature = (IEnumFeature)iEnumFeatureSetup;
            pEnumFeature.Reset();
            IFeature pFeature = pEnumFeature.Next();
            
            //如果你拿到的第一个要素很奇怪，也许会需要下面这条
            //pFeature = pEnumFeature.Next(); 
```
现在拿到了这个pFeature对象，你就拿到了房产证，可以查看feature的属性了。

## 2.3 要素和相应介绍信息的绑定
动态加载介绍窗口的关键是需要一个ID（标识符）把你预先存储的介绍信息（包括图片）和选择的地图要素绑定起来，我用的建筑的名字。

```csharp
//获取你需要的字段的值，2是索引号，请根据自己的数据修改
String name = pFeature.get_Value(2).ToString();
```
然后你需要拿着feature和name去实例化一个介绍窗口。
## 2.4 介绍窗口类
新建一个Windows窗体类FormIntro;
自己根据显示需求画好Form的模板。
我的是这样：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210506182108240.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0xleGllbG92ZXNoYWly,size_16,color_FFFFFF,t_70)

添加两个成员变量和一个字典变量（把name作为键，介绍的文本作为值，实现文本调取）

```csharp
//窗体类的变量
public IFeature pfeature;
public string bname;
Dictionary<string, string> dict = new Dictionary<string, string>();
```
下面是调取相关信息显示到窗口上的方法：

```csharp
        public void showDetails(string bname)
        {
            string intro = getInfo(bname);
            string pic = @"自己的路径"+bname+".jpg";
            //设置文本信息
            label1.Text= bname;
            label2.Text = arrlist[1].ToString();
            //设置图片适应pictureBox的大小
            pictureBox1.SizeMode = PictureBoxSizeMode.StretchImage;
            //设置图片
            pictureBox1.Image = Image.FromFile(pic);

        }
        
        //这个函数很简单也可以不写。但是当你需要调取的字段有很多的话，就需要考虑看上去比较聪明的写法^^
	    public string getInfo(String bname)
        {
            string intro=dict[bname];
            return intro;
        }
```
最后，只需要在点击查询按钮之后的响应代码里调用这个showDetails方法来设置窗口信息，并且让窗口Show()就好了；

```csharp
		    FormIntro intro = new FormIntro();
            while(pFeature != null)
            {                             
                string name = null;
                try
                {
                    name = pFeature.get_Value(2).ToString();
                }
                catch (NullReferenceException)
                {
                    MessageBox.Show("didn't get buildingname");   
                }
                finally
                {
                }
                pFeature = pEnumFeature.Next(); 
                intro.showDetails(name);
                intro.Show();                              
            }
        }
```
至此，你就可以给网友们在线展示一下你的校园了。（雾）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210506184829635.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0xleGllbG92ZXNoYWly,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021050618490249.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0xleGllbG92ZXNoYWly,size_16,color_FFFFFF,t_70)
# 三、参考资料和slogan
这个功能的实现也获得了不少互联网的帮助，你也许需要一些扩展知识：

[[1] RTFM        (*F=friendly)](https://desktop.arcgis.com/en/arcobjects/latest/net/webframe.htm#welcome.htm)
[[2] 模仿ArcEngine的 Identify功能展示要素的属性信息](https://blog.csdn.net/the_snail/article/details/82285064?utm_medium=distribute.pc_relevant_download.none-task-blog-baidujs-1.nonecase&depth_1-utm_source=distribute.pc_relevant_download.none-task-blog-baidujs-1.nonecase)
[[3] AE要素选择（点选和拉框选择）](https://www.cnblogs.com/arxive/p/6109959.html)
[[4]珞珈史迹知多少：武大“国保”名单内外的历史建筑](https://baijiahao.baidu.com/s?id=1669347956531297602&wfr=spider&for=pc)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210506191253134.png)


