## 引言
软件程序设计一个永恒的主题就是**高内聚底耦合**,因为高耦合的代码在前期的问题不会很突出，而且代码有可能不低耦合的看起来更为直接简洁，开发的时候更方便，但是每个程序代码都有可能发生迭代，因此随着每个版本的迭代高耦合的代码模块的维护和新添的功能就会越来越困难，调试也会越来越难，导致牵一发而动全身的高耦合的代码面临的选择不多了,重构是其中的选择，但是现实中重构需要成本和时间的，不一定得到其他项目人员的理解... 所以最开始低耦合是相对比较好的选择，这样避免的后续问题。
## iOS解耦机制
 -  delegate 代理模式 
- block 模式
- notification 通知中心
- KVO 键值观察
delegate 与 block 解耦的作用相同，都是委托模式 需要双方同意
notification 作为通知中心虽然可以起到解耦的作用，但是会消耗大量的资源并且效果并不是很好
KVO 作为黑科技之一，属性键-值观察 与notification 都是属于观察者模式 
## 解耦目的
解耦的目的是各个模块相互调用时没有相互关联，按照指定协议进行运行工作 ，模块之间组件化也会减少干扰
## 解耦思路
1. 内部控制器之间的跳转
利用字符串转**目标控制器** ,KVC模式进行传入参数,block回调收集目标控制器完成的一系列操作
2. 指定方法的调用
    1. 实例方法调用  : 需要传入实例 与 方法字符串 , 通过block回调传入参数 返回执行结果      
    2. 类方法调用 : 传入类名 与 方法名称 ， 通过block回调传入参数 返回执行结果
3. 外部链接方式跳转
    外部链接进行解析拆成指定方法 然后执行 1 或者2
## Router实现
* 跳转模式

~~~objectivec
typedef NS_OPTIONS(NSInteger, RTTransition){
    /// present 跳转
    RTTransitionPresent,
    /// push 跳转
    RTTransitionPush,
    /// present 跳转 (带导航栏)
    RTTRansitionNAVPresent,
    /// 无需跳转 用作创建控制器
    RTTransitionNone
};
~~~
* block回调

~~~objectivec
typedef void(^RTCompletion)(NSError * __nullable error, id __nullable obj);
~~~
* 给控制器添加协议

~~~objectivec

@protocol RouterParamDelegate <NSObject>
/// 参数
@property (nonatomic, strong) NSDictionary *routerParams ;
/// 跳转完成后回调操作
@property (nonatomic, strong) RTCompletion routerCompletion ;
@optional
/// 需要参数的属性列表  利用 KVC 模式传参
-(NSArray<NSString *>*)routerTopropertys ;

@end
  /// 设置类别 代理模式
@interface UIViewController (RouterDelegate) <RouterParamDelegate>

@end
/// runtime 类别储存属性值
@implementation UIViewController (RouterDelegate)
-(NSDictionary *)routerParams{
   return objc_getAssociatedObject(self, @"routerparams");
}
-(void)setRouterParams:(NSDictionary *)routerParams{
    [self willChangeValueForKey:@"routerparams"];
    objc_setAssociatedObject(self, @"routerparams", routerParams, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
    [self didChangeValueForKey:@"routerparams"];
}

-(void)setRouterCompletion:(RTCompletion)routerCompletion{
    [self willChangeValueForKey:@"routerCompletion"];
    objc_setAssociatedObject(self, @"routerCompletion", routerCompletion, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
    [self didChangeValueForKey:@"routerCompletion"];
}
-(RTCompletion)routerCompletion{
    return objc_getAssociatedObject(self, @"routerCompletion");
}
@end
    
~~~
* 外部链接打开App是检测的链接的代理

~~~objectivec
@protocol RouterHandelDelegate <NSObject>
@required
-(NSString *)host ;

/**
 检测是否为登录状态
 */
-(BOOL)isLogin ;

@optional
-(void)transitionWithTargetName:(NSString *)viewControllerName Params:(NSDictionary *__nullable)params ;

@end
~~~
* 执行方法时需要传入参数对象

~~~ objectivec
/// 数据操作 传出参数按顺序对应
@interface RouterModel : NSObject

@property (nonatomic, strong) id first ;
@property (nonatomic, strong) id second ;
@property (nonatomic, strong) id third ;
@property (nonatomic, strong) id fouth ;
@property (nonatomic, strong) id fifth ;
@property (nonatomic, strong) id sex ;

@end
~~~
* 路由执行对象

~~~objectivec
@interface Router : NSObject
/// 跳转 导航类
+(void)navigationController:(Class)className ;
/// 项目名设置  主要针对 swift 控制器的跳转
+(BOOL)swiftClassWithProjectName:(NSString *)projectName;
/// 处理外部跳转
+(BOOL)routerHandleOpenURL:(NSURL *)url withDelegate:(id<RouterHandelDelegate>)delegate ;
/**
 进行跳转调整
 @param target 操作控制器
 @param viewControllerName 目标控制器名
 @param params 传入参数
 @param transition 跳转模式
 @return 目标控制器
 */
+(UIViewController *)transitionWithSource:(UIViewController *__nullable)target Target:(NSString *__nonnull)viewControllerName Params:(NSDictionary *__nullable)params TransitionMode:(RTTransition)transition ;
/**
 进行跳转调整
 @param target 操作控制器
 @param viewControllerName 目标控制器名
 @param params 传入参数
 @param transition 跳转模式
 @param complete 操作完成
 @return 目标控制器
 */
+(UIViewController *)transitionWithSource:(UIViewController *__nullable)target Target:(NSString *__nonnull)viewControllerName Params:(NSDictionary *__nullable)params TransitionMode:(RTTransition)transition complete:(RTCompletion __nullable)complete ;

/**
 方法的执行

 @param targetName 执行者名
 @param selectorName 执行方法名
 @param params 参数传递
 @return 返回值
 */
+(id)performTarget:(NSString *)targetName selectorName:(NSString *)selectorName Params:(void(^)(RouterModel *make))params ;

/**
 方法执行
 @param target target description
 @param selectorName selectorName description
 @param params params description
 @return return value description
 */
+(id)performObject:(NSObject *)target selectorName:(NSString *)selectorName Params:(void (^)(RouterModel * _Nonnull))params ;

@end
~~~
## Router的实现
* 基础配置 配置导航栏对象   swift跳转需要工程名

~~~objectivec
static NSString *__projectName = @"IM" ;
static Class __className ;
@implementation Router
+(void)navigationController:(Class)className{
    __className = className ;
}
+(BOOL)swiftClassWithProjectName:(NSString *)projectName{
    __projectName = projectName ;
    return true ;
}
~~~
* 外部链接打开App是解析链接

~~~objectivec
/// url 为外部连接  
+(BOOL)routerHandleOpenURL:(NSURL *)url withDelegate:(id<RouterHandelDelegate>)delegate{
    NSString *viewControllerName ;
    if (![delegate isLogin]) {
        return false ;
    }
    NSMutableDictionary *param = [[NSMutableDictionary alloc]init];
    if (![url.host isEqualToString:delegate.host]) {
        viewControllerName = @"web" ;
        NSString *http = [NSString stringWithFormat:@"http://%@%@?%@",url.host,url.path,url.query];
        [param setObject:http forKey:@"url"];
        [delegate transitionWithTargetName:viewControllerName Params:param];
        return true ;
    }
    NSString *path = url.path ;
    if (path && path.length>0) {
        path = [url.path substringFromIndex:1] ;
        NSArray *paths = [path componentsSeparatedByString:@"/"];
        viewControllerName = paths.firstObject ;
    }else{
        path = url.host ;
    }
    NSString *query = url.query;
    if (query && query.length>0) {
        NSArray <NSString *>*querys = [query componentsSeparatedByString:@"&"];
        [querys enumerateObjectsUsingBlock:^(NSString * _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
            if ([obj containsString:@"="]) {
                NSRange range = [obj rangeOfString:@"="];
                NSString *key = [obj substringToIndex:range.location];
                NSString *value = [obj substringFromIndex:range.location+1];
                [param setObject:value forKey:key];
            }
        }];
    }
  /// 数据解析成可执行的格式
    [delegate transitionWithTargetName:viewControllerName Params:param];
    return true ;
}
~~~
* 控制器切换跳转实现

~~~objectivec
/**
 本地组件(控制器)切换
 @param target 调用者
 @param viewControllerName targetName
 @param params params
 @param transition transition
 @return UIViewController object
 */
+(UIViewController *)transitionWithSource:(UIViewController *)target Target:(NSString *)viewControllerName Params:(NSDictionary *)params TransitionMode:(RTTransition)transition complete:(RTCompletion)complete{
    
    UIViewController *viewController;
    if (NSClassFromString(viewControllerName)) {
        viewController = [[NSClassFromString(viewControllerName) class] new];
    }else if(NSClassFromString([NSString stringWithFormat:@"%@.%@",__projectName,viewControllerName])){
        viewController = [[NSClassFromString([NSString stringWithFormat:@"%@.%@",__projectName,viewControllerName]) class]new];
    }
    
    if (viewController == nil) {
        // 处理无响应请求的地方之一。
        // 可以事先给一个固定的target专门用于在这个时候顶上，然后处理这种请求
        return nil;
    }
    
    if (![viewController isKindOfClass:[UIViewController class]]) {
        return nil;
    }
    /// 判断执行跳转对象
    if ([target isKindOfClass:[UIViewController class]]) {
        target = (UIViewController *)target;
    }
    else {
        target = [UIApplication sharedApplication].keyWindow.rootViewController;
    }
    /// KVC 给目标对象传值 避免目标对象的穿插
    if ([viewController respondsToSelector:@selector(routerTopropertys)]) {
        NSArray<NSString *>*list = [viewController routerTopropertys];
        for (NSString *key in list) {
            id object = [params objectForKey:key];
            if (object != nil) {
                 /// KVC
                [viewController setValue:object forKey:key];
            }
        }
    }
    viewController.routerParams = params ;
    viewController.routerCompletion = complete 
     /// 判断跳转方式
    switch (transition) {
        case RTTransitionPush:
        {
            if (!target.navigationController) {
                return viewController;
            }
            if ([target.navigationController.viewControllers count]==1) {
                viewController.hidesBottomBarWhenPushed = YES;
            }
            [target.navigationController pushViewController:viewController animated:true];
            
        }break;
        case RTTransitionPresent:
        {
            [target presentViewController:viewController animated:true completion:nil];
            
        }break;
        case RTTRansitionNAVPresent:
        {
            UINavigationController *navigationController ;
            if (__className) {
                navigationController = [[__className alloc]initWithRootViewController:viewController];
            }else{
                navigationController = [[UINavigationController alloc]initWithRootViewController:viewController];
            }
            [target presentViewController:navigationController animated:YES completion:nil];
        }break ;
        case RTTransitionNone:
        {
            return viewController;
            break;
        }
    }
    /// 返回目标控制器对象
    return viewController;
}
~~~
* 实例方法的调用

~~~ objectivec
/// target 目标实例     selectorName 被执行方法名  params 按顺序传入参数
+(id)performObject:(NSObject *)target selectorName:(NSString *)selectorName Params:(void (^)(RouterModel * _Nonnull))params{
    
    if (target == nil) {
        // 处理无响应请求的地方之一。
        // 可以事先给一个固定的target专门用于在这个时候顶上，然后处理这种请求,也可以跳转App内url。
        return nil;
    }
    
    SEL action = NSSelectorFromString(selectorName);
    if (action == nil) {
        // 处理无响应请求的地方之一。
        // 可以事先给一个固定的target专门用于在这个时候顶上，然后处理这种请求
        return nil;
    }
    RouterModel *model = [[RouterModel alloc]init];
    if (params) {
      /// 执行block接收参数
        params(model);
    }
    // 可以响应 说明方法可执行
    if ([target.class respondsToSelector:action] ||
        [target respondsToSelector:action]) {
        return [self performTarget:target Action:action Params:model];
    }
    
    // 有可能target是Swift对象
    NSString *actionString = [NSString stringWithFormat:@"%@WithParams:", selectorName];
    action = NSSelectorFromString(actionString);
    
    // 可以响应
    if ([target respondsToSelector:action]) {
        return [self performTarget:target Action:action Params:model];
    }
    
    // 处理无响应请求的地方之二
    // 如果无响应，则尝试调用对应target的notFound方法统一处理
    action = NSSelectorFromString(@"notFound:");
    if ([target respondsToSelector:action]) {
        return [self performTarget:target Action:action Params:model];
    }
    else {
        // 处理无响应请求的地方之三，
        // 在notFound都没有的时候，可以用之前的固定的target顶上
        return nil;
    }
    
    
}
~~~
* 私有方法 具体执行

~~~objectivec
/**
 私有方法，perform
 @param target target
 @param action action
 @param model params
 @return return value of method peformed
 */
+(id)performTarget:(NSObject *)target Action:(SEL)action  Params:(RouterModel *)model {
    
    NSMethodSignature* methodSig;
    // 优先调用类方法
    if ([target.class respondsToSelector:action]) {
        methodSig = [target.class methodSignatureForSelector:action];
    }
    else {
        methodSig = [target methodSignatureForSelector:action];
    }
    if(methodSig == nil) {
        return nil;
    }
    
    // 获取invocation
    NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:methodSig];
    
    // 入参  进行参数传入
    for (NSInteger idx = 2; idx < methodSig.numberOfArguments; idx ++ ) {
        id obj = nil;
        switch (idx) {
            case 2:
                obj = model.first ;
                break;
            case 3:
                obj = model.second ;
                break;
            case 4:
                obj = model.third ;
                break;
            case 5:
                obj = model.fouth ;
                break;
            case 6:
                obj = model.fifth ;
                break;
            case 7:
                obj = model.sex ;
                break;
            default:
                break;
        }
        // 类型判断，参数值校验传入
        if ([obj isKindOfClass:[NSNumber class]]) {
            
            // 由于传入的基础类型float cgfloat无法判断，所以用方法参数类型
            const char *type = [methodSig getArgumentTypeAtIndex:idx];
            
            NSNumber *number = (NSNumber *)obj;
            if (strcmp(type, @encode(CGFloat)) == 0) {
                CGFloat num = number.floatValue;
                [invocation setArgument:&num atIndex:idx];
            }
            else if (strcmp(type, @encode(NSInteger)) == 0) {
                NSInteger num = number.integerValue;
                [invocation setArgument:&num atIndex:idx];
            }
            else if (strcmp(type, @encode(NSUInteger)) == 0) {
                NSUInteger num = number.unsignedIntegerValue;
                [invocation setArgument:&num atIndex:idx];
            }
            else if (strcmp(type, @encode(float)) == 0) {
                CGFloat num = number.floatValue;
                [invocation setArgument:&num atIndex:idx];
            }
            else if (strcmp(type, @encode(double)) == 0) {
                double num = number.doubleValue;
                [invocation setArgument:&num atIndex:idx];
            }
            else if (strcmp(type, @encode(int)) == 0) {
                int num = number.intValue;
                [invocation setArgument:&num atIndex:idx];
            }
            else if (strcmp(type, @encode(BOOL)) == 0) {
                BOOL num = number.boolValue;
                [invocation setArgument:&num atIndex:idx];
            }
            else { // 就是NSNumber类型
                [invocation setArgument:&number atIndex:idx];
            }
        }
        else if ([obj isKindOfClass:[NSValue class]]) {
            
            NSValue *value = (NSValue *)obj;
            if (strcmp(value.objCType, @encode(CGSize)) == 0) {
                CGSize size = value.CGSizeValue;
                [invocation setArgument:&size atIndex:idx];
            }
            else if (strcmp(value.objCType, @encode(CGRect)) == 0) {
                CGRect rect = value.CGRectValue;
                [invocation setArgument:&rect atIndex:idx];
            }
            else if (strcmp(value.objCType, @encode(NSRange)) == 0) {
                NSRange range = value.rangeValue;
                [invocation setArgument:&range atIndex:idx];
            }
            else if (strcmp(value.objCType, @encode(CGPoint)) == 0) {
                CGPoint point = value.CGPointValue;
                [invocation setArgument:&point atIndex:idx];
            }
        }
        else {
            [invocation setArgument:&obj atIndex:idx];
        }
    }
    
    [invocation setSelector:action];
    if ([target.class respondsToSelector:action]) {
        [invocation setTarget:target.class];
    }
    else {
        [invocation setTarget:target];
    }
    [invocation invoke];
    
    // 处理返回值
    const char* retType = [methodSig methodReturnType];
    
    if (strcmp(retType, @encode(void)) == 0) {
        return nil;
    }
    else if (strcmp(retType, @encode(NSInteger)) == 0) {
        NSInteger result = 0;
        [invocation getReturnValue:&result];
        return @(result);
    }
    else if (strcmp(retType, @encode(BOOL)) == 0) {
        BOOL result = 0;
        [invocation getReturnValue:&result];
        return @(result);
    }
    else if (strcmp(retType, @encode(CGFloat)) == 0) {
        CGFloat result = 0;
        [invocation getReturnValue:&result];
        return @(result);
    }
    else if (strcmp(retType, @encode(NSUInteger)) == 0) {
        NSUInteger result = 0;
        [invocation getReturnValue:&result];
        return @(result);
    }
    
    void *result = nil;
    [invocation getReturnValue:&result];
    return (__bridge id)result;
    
}
~~~
* 类方法执行

~~~objectivec
/**
 本地组件调用
 
 @param targetName targetName
 @param selectorName actionName
 @param params params
 @return return value of method maped action
 */
+(id)performTarget:(NSString *)targetName selectorName:(NSString *)selectorName Params:(void(^)(RouterModel *make))params{
    
    NSObject *target;
    if (NSClassFromString(targetName)) {
        target = [[NSClassFromString(targetName) class] new];
    }else{
        return nil ;
    }
    if (target == nil) {
        // 处理无响应请求的地方之一。
        // 可以事先给一个固定的target专门用于在这个时候顶上，然后处理这种请求,也可以跳转App内url。
        return nil;
    }
    
    SEL action = NSSelectorFromString(selectorName);
    if (action == nil) {
        // 处理无响应请求的地方之一。
        // 可以事先给一个固定的target专门用于在这个时候顶上，然后处理这种请求
        return nil;
    }
    RouterModel *model = [[RouterModel alloc]init];
    if (params) {
        params(model);
    }
    // 可以响应
    if ([target.class respondsToSelector:action] ||
        [target respondsToSelector:action]) {
        return [self performTarget:target Action:action Params:model];
    }
    
    // 有可能target是Swift对象
    NSString *actionString = [NSString stringWithFormat:@"%@WithParams:", selectorName];
    action = NSSelectorFromString(actionString);
    
    // 可以响应
    if ([target respondsToSelector:action]) {
        return [self performTarget:target Action:action Params:model];
    }
    
    // 处理无响应请求的地方之二
    // 如果无响应，则尝试调用对应target的notFound方法统一处理
    action = NSSelectorFromString(@"notFound:");
    if ([target respondsToSelector:action]) {
        return [self performTarget:target Action:action Params:model];
    }
    else {
        // 处理无响应请求的地方之三，
        // 在notFound都没有的时候，可以用之前的固定的target顶上
        return nil;
    }
    

}

~~~