### 文件列表及封面获取流程

###### 入口是CoolReader的showDirectory方法,通过Scanner扫描完成后，会把文件的结构存放到FileInfo里   
……CoolReader | showDirectory(x1)   
> x1是FileInfo，指定的目录*   

…………FileBrowser | showDirectory(x1,x2)  
> x1为FileInfo类型，根目录，x2为null    

………………Scanner | scanDirectory(x1,x2,x3,x4,x5)  
> x1为数据库，x2为FileInfo,x3 为Callback   
> 这一步会把目录下的文件和子目录扫描出来  
> 如果是zip文件，通过Engine调用getArchiveItemsInternal方法，进入cr3engine.cpp扫描zip信息，先不研究   

……………………Scanner | scanDirectoryFiles(x1,x2,x3,x4,x5)   
> x1为数据库，x2为FileInfo,x5 为Callback  
> 这一步主要是获取文件的信息，如果数据库有，从数据库取，如果没有，通过Engine继续扫描，并存库  

…………………………Engine | scanBookProperties(x1) => scanBookPropertiesInternal(x1)   
> 进入cr3engine.cpp,主要扫描Epub书籍的标题、作者、语言等

…………FileBrowser | showDirectoryInternal
> 在扫描完成后，文件的结构信息已经关联到了currentDirectory里，这里通过adapter的更新机制，列表中的图标信息和封面信息会在getView里边获取

………………FileListAdapter.ViewHolder | setItem(x1,x2)
> 这一步绑定列表item数据，通过CoverpageManager获取封面

……………………CoverpageManager | getCoverpageDrawableFor(x1,x2)
> x1为数据库，x2为FileInfo
> 这个方法返回了一个CoverImage对象，这是一个Drawable的子类，当调用到draw方法时，继续执行封面获取流程

…………………………Engine | scanBookCover(x1) => scanBookCoverInternal(x1)
> 进入cr3engine.cpp，扫描书籍封面，返回封面的byte数据，用于显示并存入数据库

……………………CoverpageManager | drawCoverpage(x1,x2)
> x1 为上一步获取到的byte数据

…………………………Engine | drawBookCover(x1,x2,x3,x4,x5,x6,x7,x8) => drawBookCoverInternal(x1,x2,x3,x4,x5,x6,x7,x8)
> x1 为Bitmap对象，x2为图片的byte数据，x3为字体，x4为标题
> 这一步将图片数据绘制到Bitmap里，如果没有图片数据，绘制默认封面+标题


### Engine的初始化和字体加载流程
字体的加载流程需要从Engine的初始化说起，整个初始化流程在Engine的静态代码块中，也就是第一次访问Engine这个类的时候，会执行初始化
可以static代码块中加个Log打印调用栈，简单跟踪一下初始化的时机：   
```
Log.d(TAG, Log.getStackTraceString(new Throwable())); 
```
……Engine | findFonts()
> 这个方法会遍历三个文件夹，去寻找字体文件，返回一个字体文件的路径数组   
> 但是这块代码有一个问题，对于中文字体的扫描，少了".ttc"格式，很可能扫不到中文字体，导致所有字体都不支持中文，出现乱码，这里顺手加上
>   - .cr3目录下的fonts目录
>   - sd卡上的fonts目录
>   - /system/fonts  

……Engine |  initInternal(x1)
> 在扫描完字体之后，会调用jni方法，进入cr3engine.cpp,执行初始化，x1是扫描到的字体路径数组
> 这里涉及到的cpp文件：
> - hyphman.cpp:断字的管理，在这里会进行初始化
> - lvfntman.cpp:字体的管理，基于freetype实现，也是在这里初始化






