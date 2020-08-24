## iOS 多个后台接口 颜色 图片 国际化等配置你确定会集中管理....

- 引言

  在iOS开发过程中一边会遇到后台接口会连接多个服务器,每个地方都在用颜色 图片，在项目最开始时候用起来确实便捷，但是一个项目会出想迭代的情况，后台更改服务器，接口时候怎么能一次性更改或者更换图片的时候你会发现需要改的地方太多了，如果需求增加主题更换呢？所以这些配置系列的东西都需要集中管理

- 解决方案

  - 后台接口与颜色配置

    利用json文件或者plist文件配置后台与颜色值，通过单例解析方式调取对应所需的值

  - 图片集中管理

    图片使用需要考虑主题更换和网络更换主题 ，所以考虑用bundle包模式来使用图片

- 解决用法

  - 后台接口管理

    json文件内容格式如下  打包工程师可以使用加密工具进行加密json文件 代码解密使用 这样避免解包是泄露接口

    ```json
    {
        "UserApi":{ ## 用户模块
            "host":"http://xxx.xxx/xx", ## 主机地址      
            "loginString":"/api/login" ## 登录
        },
        "IMApi":{   ## 其他模块
            "host":"",
            "loginString":"dffdasfdsafdsafds" ## 
        }  
    }
    ```

    UserApi  IMApi 建议首字母大写  单例封装讲解

    ```objectivec
    @interface VCApiManager ()
    /// 储存json文件内容
    @property (nonatomic, strong) NSDictionary <NSString *,NSDictionary *>*dictioanry ;
    @end
    -(void)reloadApi{
        NSArray <NSString *> *classArray = self.dictioanry.allKeys ; // 类名
        for (NSString *classString in classArray) {
            if (classString == nil || classString.length==0) {
                continue ;
            }
            id objc = [[NSClassFromString(classString) alloc]init];  // 实例化类
            if (objc == nil) {
                NSLog(@"%@ class is null",classString);
                continue ;
            }
          	// KVC 给单例设置属性值
            [self setValue:objc forKey:[self lowercastFirstString:classString]];
          /// json 二级
            NSDictionary *valueDictionary = [self.dictioanry valueForKey:classString];
          /// 获取服务地址
            NSString *host = [valueDictionary valueForKey:@"host"];
            if (host == nil) {
                host = [objc valueForKey:@"host"];
                if (host == nil) {
                    host = self.host;
                }
            }
            unsigned int propertyCount = 0;
          /// 获取实例属性
            objc_property_t *properties = class_copyPropertyList(object_getClass(objc), &propertyCount);
            for (unsigned int i = 0; i < propertyCount; ++i) {
                objc_property_t property = properties[i];
                const char * name = property_getName(property);//获取属性名字
                NSString *key = [NSString stringWithUTF8String:name];
                NSString *value = [valueDictionary objectForKey:key];
                NSString *url = [host stringByAppendingString:value];
              /// kvc 赋值
                [objc setValue:url forKey:key];
            }
        }
        
    }
    ```

    使用方式

    ```objectivec
    @interface UserApi : NSObject
      /// 登录
    @property (nonatomic, strong) NSString *loginString ;
    @end
     /// 继承单例
    @interface ApiShare : VCApiManager
      /// 用户模块
    @property (nonatomic, strong) UserApi *userApi ;
    @end
      /// 直接使用  ApiShare.shareInstance.userApi.loginString  就可以获取登录接口
    ```

  - 颜色集中管理也是类似的

    json文件格式如下

    ```objectivec
    {
    	"default":{   ## 默认使用颜色
    		"navigationTitleColor":"#ffffff"
    	},
      "sim":{ ## 其他主题颜色
        "navigationTitleColor":"#000000"
      }
    }
    ```

    单例模式如后台api几种管理类似 解析json文件赋值到新的对象上 直接调用即可 也方便切换主题颜色

  - 图片的集中管理模式

    1. 创建一个默认的bundle包

    2. 创建一个实例管理bundle包

       ```objectivec
       // bundle包的路径  切换主题即更换bundle包即可
       -(void)setImageBundle:(NSString *)path{
           NSString *url = [NSBundle.mainBundle pathForResource:path ofType:nil];
           if (url) {
               self.bundleString = path ;
           }else{
               NSBundle *bundle = [NSBundle bundleWithPath:path];
               if (bundle) {
                   self.bundleString = bundle.bundlePath ;
               }
           }
       }
       ```

    3. UIImage交换方法

       ```objectivec
       +(void)load{
           __weak typeof(self) weakSelf = self;
           static dispatch_once_t onceToken;
           dispatch_once(&onceToken, ^{
               Class class = [weakSelf class];
               Method oldMethod = class_getClassMethod(class, @selector(imageNamed:));
               Method newMethod = class_getClassMethod(class, @selector(imageCustomName:));
               method_exchangeImplementations(oldMethod, newMethod);
           });
       }
       
       +(UIImage *)imageCustomName:(NSString *)name{
         /// 过滤
           if (AFBundleManager.shareBundleManager.imageNames && [AFBundleManager.shareBundleManager.imageNames containsObject:name]) {
               return [UIImage imageCustomName:name];
           }
         /// 访问图片是 指向指定的bundle包里的图片
           NSString *img = [NSString stringWithFormat:@"%@/%@",AFBundleManager.shareBundleManager.bundleString,name];
           return [UIImage imageCustomName:img];
       }
       ```

    4. 使用方式  

       ```objectivec
       [UIImage imageNamed:""];  // 使用系统方法即可
       ```

    5. 切换主题直接更换访问的bundle包即可 也可以从服务器上下载bundle包(每个包对应的图片的名字要一致)

  - 国际化

    1. 创建一个xxx.bundle包

    2. bundle包的文件夹使用各语言简称

    3. 解析bundle包

       ~~~objectivec
       /// 设置语言bundle包
       -(void)setBundlePath:(NSString *)path type:(NSInteger)type{
           self.resourceName = path ;
           self.type = type ;
           [self setProjectLanguage];
       }
       /// 切换语言
       -(void)setLanguage:(NSString *)language{
           [[NSUserDefaults standardUserDefaults] setObject:language forKey:@"sys_language"];
           [[NSUserDefaults standardUserDefaults] synchronize];
           [self setProjectLanguage];
       }
       /// 更新语言
       -(void)setProjectLanguage{
           // 然后将设置好的语言存储好，下次进来直接加载
           [[NSUserDefaults standardUserDefaults] setObject:self.currentLanguage forKey:@"sys_language"];
           [[NSUserDefaults standardUserDefaults] synchronize];
           if (self.type == 1) {
               NSBundle *fbundle = [NSBundle bundleWithPath:[NSBundle.mainBundle pathForResource:self.resourceName ofType:@"bundle"]];
               objc_setAssociatedObject(NSBundle.mainBundle, &_bundlekey, self.currentLanguage ? [NSBundle bundleWithPath:[fbundle pathForResource:self.currentLanguage ofType:@"lproj"]] : nil, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
           }else if (self.type == 2){
               NSBundle *fbundle = [NSBundle bundleWithPath:self.resourceName];
               objc_setAssociatedObject(NSBundle.mainBundle, &_bundlekey, self.currentLanguage ? [NSBundle bundleWithPath:[fbundle pathForResource:self.currentLanguage ofType:@"lproj"]] : nil, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
           }else{
               objc_setAssociatedObject(NSBundle.mainBundle, &_bundlekey, self.currentLanguage ? [NSBundle bundleWithPath:[NSBundle.mainBundle pathForResource:self.currentLanguage ofType:@"lproj"]] : nil, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
           }
           
       }
       -(NSString *)currentLanguage{
           NSString *savedLanguage = [[NSUserDefaults standardUserDefaults] objectForKey:@"sys_language"];
           return savedLanguage ?: [[NSBundle mainBundle] preferredLocalizations].firstObject;
       }
       ~~~

    4. 使用方式 就用系统方式调用即可

       ~~~objectivec
       static const char *_bundlekey = "_bundle_key";
       /// 继承NSBundle 使用方式
       @interface Language : NSBundle
           
       @end
       @implementation Language
         /// 重写国际化调用语言的方法
       -(NSString *)localizedStringForKey:(NSString *)key value:(NSString *)value table:(NSString *)tableName {
           NSBundle *bundle = objc_getAssociatedObject(self, &_bundlekey);
           return bundle ? [bundle localizedStringForKey:key value:value table:tableName] : [super localizedStringForKey:key value:value table:tableName];
       }
        @end
       ~~~

- 后序

  这些集中管理方便后续迭代 ，也可以用于组件化与模块化的开发，能够使每个模块进行独立避免交叉。如有更好建议希望不吝啬赐教！

