Unity官方团队出品的工具
# Asset-Auditing
Asset importing management
Asset Auditing主要用于执行各资源的审核工作，比如导入贴图时按照统一既定格式进行批次设置，简单易用，且具有高度可扩展性。

请访问Unity中国区Github页面下载Asset Auditing工具：https://github.com/unity-cn/Asset-Auditing
原贴地址：http://forum.china.unity3d.com/forum.php?mod=viewthread&tid=19957&extra=page%3D1%26filter%3Dtypeid%26typeid%3D18

项目介绍      
在开发Unity项目的时候，很多开发者都会遇到资源管理的问题。比如因为一些疏忽，你可能忘记把一张超大的4096分辨率的贴图设置成压缩格式，另外也有可能不小心把mesh文件设置成了read/write模式。靠人工来检查资源的导入设置往往是不靠谱的，因此我们开发了一个小工具，你可以用它来监督所有资源的导入设置，减少因为不正确的导入设置带来的问题。   

导入设置      

在Unity中，所有要导入到引擎的资源都有各种不同的导入选项。这些选项用来调整资源的属性、导入精度、以及压缩方法等等。某些设置还会包含一些重要的平台相关的选项。比如在IOS平台，你所使用的纹理有可能需要和Android平台使用不同的设置，如果使用手工来管理这些设置会是非常困难的，可能会导致很多错误。因此，通过自动化的管理工具来执行这些操作是一个非常必要的方式。
如下图所示，这是一张纹理的导入设置：   

  


实现方法    
  
Unity引擎提供了资源导入的回调函数，这些回调函数是通过AssetPostprocessor API来实现的。AssetPostprocessor中的这些函数会按照一定的顺序执行，它们的执行顺序如下：
OnPreprocess是导入过程中最早执行到的，你可以在这里重新指定新的导入设置值。我们在工具中会设置相同类型的资源使用统一的设置就是在这里完成的。
一旦模型和材质导入完成之后，引擎会根据资源内的数据创建GameObject层级。每个GameObject会拿到它对应的MeshFilter、MeshRenderer、MeshCollider组建等等。在把材质赋值给MeshRenderer之前会调用OnAssignMaterialModel函数。
在GameObject初始化MeshRenderer和用户数据之后，OnPostprocessGameObjectWithUserProperties会被调用。这会在它的子GameObject生成之前调用。
如果前面没有禁止动画生成设置的话（请查看ModelImporter.generateAnimation），接下来会生成SkinnedMesh和Animation。如果可能的话，Avatar也会被创建出来并且执行GameObject的层级优化，然后会调用OnPostprocessModel函数。
​对SpeedTree资源来说会调用OnPreprocessSpeedTreee和OnPostprocessSpeedTree函数，这和OnPreprocessModel以及OnPostprocessModel函数是类似的。
   
使用方法   
   
正确使用该工具，大致分为以下步骤；
一、将AssetAuditing工具导入到工程。
二、创建资源导入格式文件。

点击Unity编辑器的Assets/Create Asset Auditing Rule菜单，在编辑器的Project窗口里会创建一个New Asset Rule文件。如下图所示：

  

在上图中，Asset Rule Type包括Any，Texture，Mesh三个选项。TEXTURE SETTINGS下属内容对应贴图属性信息，即所有对应的Texture都将应用该面板中的属性设置。例如勾选下方的Generate Mip Maps则表示所有的Texture都将生成Mip Map。MESH SETTINGS下属内容对应网格的属性信息，即所有对应的Mesh都将应用该面板中的属性设置。 相应信息设置完毕后，只要点击Apply按钮就会对相应的资源类型执行相关的设置。

如果希望对该功能进行扩展，可以对Editor目录下的AssetRuleInspector脚本进行简单修改，以达到实际需求。  

三、设置导入参数。
导入格式文件跟其所在的文件夹相对应，如下图所示，Texture目录下对应的贴图都会统一采用Texture Rule格式文件所设置的属性信息：
  
  


同理，如果需要设置模型数据的导入格式，则如下图所示，只需要在模型文件所处的文件夹下存储一分格式文件即可：

  
0.png (133.71 KB, 下载次数: 0)
下载附件
5 天前 上传


设置完参数后请点击Apply按钮，然后工具会自动化处理相关的资源文件，将他们的导入设置按照配置文件的内容进行统一设置，并且重新导入资源文件。
