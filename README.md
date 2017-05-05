＃Objc-52Tips-读书笔记以及知识扩展

/*第一条： 了解OC语言的起源*/
	
	OC语言使用的是“消息结构”，并非“函数调用”，是由SmallTalk演化而来；

	消息结构和函数调用的区别：

		消息结构：运行时所执行的代码由运行环境决定
		
		函数调用：运行时所执行的代码是由编译器决定

		如果函数是多态的，则函数调用语言要按照“虚方法表”，查出到底执行哪一个函数实现；而采用消息

		结构的语言，无论该方法是否多态，总是要在运行时才会去查找所要执行的方法（动态绑定）

	OC的重要工作都有“运行期组件”而非编译器完成的

/*第二条： 在类的头文件里尽量少引用其他头文件*/

/*第三条： 多用字面量语法，少用与之等价的方法*/

/*第四条： 多用类常量，少用#define预处理指令*/

	例：

	#define  ANIMATION_DURATION  0.3

		编译器会把所有ANIMATION_DURATION替换成0.3 ，如果在头文件里定义，则引入的文件中ANIMATION_DURATION也会替换成0.3

		且上述例子命名不好；可用 static const NSTimeInterval kAnimationDuration = 0.3 替换；

		常量名称方法：若常量局限于“编译单元”（实现文件）之内，则前面加字母k；若常量在类之外可见，则通常以类名为前缀。

		定义常量的位置：若不打算公开，则定义在实现文件里且同时用static和const声明，这样若果试图修改const修饰符所声明的常量，则会报错；

				      static 修饰符意味着该变量仅在定义此变量的编译单元中可见；

	编译器每收到一份编译单元就会输出一份目标文件；上述例子中如果不加 static 关键字，编辑器就会给它创建一个“外部符号”，此时如果另一个编译单元也声明了同名变量，则会报错。

		一个变量同时声明 static 和 const ，那么编辑器根本不会创建符号，类似于 #define 一样，只是作用域不一样。

	公开常量问题：

		例：

		调用通知，发送通知，用字符串表示通知名称，则这个名称就可以声明成外部可见的常值变量，此类常量放在“全局符号表”中，以便于在定义该常量的编译单元之外使用。

			 比如 在头文件中这样写：extern NSString *const EOCStringConstant ；		

			 在实现文件中可以这样定义： NSString *const  EOCStringConstant  = @"VALUE";

		extern 关键字告诉编辑器在全局表中有个叫  EOCStringConstant 的符号，无需查看其定义，允许代码使用该常量；此类常量必须定义，且只能定义一次；

/*第五条： 用枚举表示状态、选项、状态码*/

	枚举类型示例：

	 	enum ZFConnectState {

	 		ZFConnectStateDisconnected,
	 		ZFConnectStateConnecting,
	 		ZFConnectStateConnected,
	 	}

	 定义枚举变量： enum ZFConnectState state =  ZFConnectStateDisconnected; 定义时太麻烦，所以使用关键字 typedef 关键字重新定义枚举类型：

	 	enum ZFConnectState {

	 		ZFConnectStateDisconnected,
	 		ZFConnectStateConnecting,
	 		ZFConnectStateConnected,
	 	}

	 	typedef enum ZFConnectState  ZFConnectState;  

	 	就可以这样定义常量了 ： ZFConnectState state =  ZFConnectStateDisconnected

	 向前声明枚举变量： 需要指定枚举的底层数据类型。

	 	例：

	 	 enum ZFConnectStateConnectionState  : NSInteger {  /* */  }

	 枚举选项彼此组合定义方法： 略

/*第六条：理解 “属性” 这一概念*/

	 属性是OC的一种特性，用于封装对象中的数据； OC对象通常会把其所使用到的数据保存为各种实例变量，实例变量一般通过存取方法来访问。

	 属性和实例变量的区别：
	
	Objective-C 1.0 中我们为interface同时声明了属性和底层实例变量，那时，属性是oc语言的一个新的机制，并且要求你必须声明与之对应的实例变量
		
		例：

		@interface MyViewController :UIViewController {
    			
    			UIButton *myButton; // 实例变量
		}
		
		@property (nonatomic, retain) UIButton *myButton;   //属性
		
		@end	 

	在Objective-C 2.0中(使用的编译器是LLVM，之前使用的是另一种编译器GCC)，@property它将自动创建一个以下划线开头的实例变量。因此，在这个版本中，我们不再为interface声明实例变量；编译器也会自动的生成一个实例变量_myButton

	@synthesize 语句只能被用在 @implementation 代码段中，@synthesize的作用就是让编译器为你自动生成setter与getter方法同时指定与属性对应的实例变量；

	@dynamic 关键字和 @synthesize 作用相反，告诉编译器不要自动创建实现属性所用的实例变量也不要自动生成setter与getter方法

	例：

	@synthesize myButton = xxx；那么self.myButton其实是操作的实例变量xxx，而不是_myButton了。

	在类别中添加的属性，不会自动合成实例变量，只是单纯的添加了getter 和 setter 方法；但可以利用 Runtime 机制来弥补这种不足

	例：

	#import xxx 
	
	static const void * externVariableKey =& externVariableKey;
	
	@implementation NSObject (Category)
	
	@dynamic variable；
	
	- (id) variable {
      	 
      	 return objc_getAssociatedObject(self, externVariableKey);
	
	}
	
	- (void)setVariable:(id) variable {
	
	    objc_setAssociatedObject(self, externVariableKey, variable, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
	
	}

	属性特质：属性的各种特质设定会影响编译器所生成的存取方法

		原子性：默认情况编译器合成的方法会通过锁定机制确保其原子性。如果属性具备nonatomic 特质，则不适用同步锁

			   atomic 与 nonatomic 的区别：具备atomic特质的获取方法会通过锁定机制来确保其操作的原子性，也就是说如果两个线程读写同一属性，那么不论何时总能看到

			   		有效的属性值。若是 nonatomic ，那么当其中一个线程在改写某种属性值时，另外一个线程也会突然闯入，把尚未修改好的属性值读取出来，导致读错

			iOS中所有属性值均是 nonatomic 的原因：在 iOS  中使用同步锁的开销较大，这样会带来性能问题；一般情况下并不要求属性必须是原子的，因为这样并不能保证线程安全，

				若要保证线程安全，则需要采用更深层次的锁定机制。


		读 / 写权限：  readwrite 拥有读写方法。该属性由@synthesize 实现，则编译器会自动合成这两个方法

				readonly 只读 只有获取方法

		内存管理语义：assign  纯量类型的简单赋值操作

				   strong   该属性设定一种拥有关系。为这种属性设置新值时，先保留新值，并放弃旧值，然后将新值设置上去

				   weak    该属性设定一种非拥有关系。为这种属性设置新值时，既不保留新值，也不放弃旧值。同assign ，当属性所指的对象摧毁时，属性值也会清空

				   	 什么情况使用 weak 关键字： 1，在 ARC 中,在有可能出现循环引用的时候,往往要通过让其中一端使用 weak 来解决,比如: delegate 代理属性

				   	 				  2，自身已经对它进行一次强引用,没有必要再强引用一次,此时也会使用 weak,自定义 IBOutlet 控件属性一般也使用 weak；当然，也可以使用strong

						那么 runtime 如何实现 weak 变量的自动置nil：

							runtime 对注册的类， 会进行布局，对于 weak 对象会放入一个 hash 表中。 用 weak 指向的对象内存地址作为 key，当此对象的引用计数为0的时候会 dealloc，

							假如 weak 指向的对象内存地址是a，那么就会以a为键， 在这个 weak 表中搜索，找到所有以a为键的 weak 对象，从而设置为 nil。

				   unsafe_unretained    和 assign 相同，但是它适用于“对象类型”，表示一种非拥有关系；不会自动清空，这一点与weak 有区别

				   copy  与strong类似，设置方法并不保留新值，而是将其拷贝

				   	注：在非集合类对象中：对 不可变 对象进行 copy 操作，是指针复制，mutableCopy 操作时内容复制；对 可变 对象进行 copy 和 mutableCopy 都是内容复制

				   		例： 
				   			[immutableObject copy] // 浅复制
							
							[immutableObject mutableCopy] //深复制
							
							[mutableObject copy] //深复制
							
							[mutableObject mutableCopy] //深复制
							
						但是：集合对象的内容复制仅限于对象本身，对象元素仍然是指针复制。

			ARC下，不显式指定任何属性关键字时，默认的关键字都有哪些：

				对应基本数据类型默认关键字是 atomic , readwrite , assign 

				对于普通的 Objective-C 对象默认关键字是 atomic, readwrite , strong
			
			注：

				 block 为什么使用copy修饰：block 使用 copy 是从 MRC 遗留下来的“传统”,在 MRC 中,方法内部的 block 是在栈区的,使用 copy 可以把它放到堆区.

			   		在 ARC 中写不写都行：对于 block 使用 copy 还是 strong 效果是一样的，但写上 copy 也无伤大雅，还能时刻提醒我们：编译器自动对 block 进行了 copy 操作

				用 @property 声明 NSString、NSArray、NSDictionary 经常使用 copy 关键字，是因为他们有对应的可变类型：NSMutableString、NSMutableArray、NSMutableDictionary，

				他们之间可能进行赋值操作，为确保对象中的字符串值不会无意间变动，应该在设置新属性值时拷贝一份。

					若用 copy 修饰  NSMutableString、NSMutableArray、NSMutableDictionary（等可变类型），则在使用可变类型相应的方法添加,删除,修改，会崩溃

						因为 copy 就是复制一个不可变 NSArray 的对象；

		如何让自己的类使用 copy 修饰符？

			若想令自己所写的对象具有拷贝功能，则需实现 NSCopying 协议。如果自定义的对象分为可变版本与不可变版本，那么就要同时实现 NSCopying 与 NSMutableCopying 协议。

			具体步骤：1，需声明该类遵从 NSCopying 协议  2，实现 NSCopying 协议。该协议只有一个方法: - (id)copyWithZone:(NSZone *)zone;

			例：


				   typedef NS_ENUM(NSInteger, CYLSex) {
				  
				       CYLSexMan,
				  
				       CYLSexWoman
				   };

				   @interface CYLUser : NSObject<NSCopying>

				   @property (nonatomic, readonly, copy) NSString *name;
				  
				   @property (nonatomic, readonly, assign) NSUInteger age;
				  
				   @property (nonatomic, readonly, assign) CYLSex sex;

				   - (instancetype)initWithName:(NSString *)name age:(NSUInteger)age sex:(CYLSex)sex;
				  
				   + (instancetype)userWithName:(NSString *)name age:(NSUInteger)age sex:(CYLSex)sex;

				   @end

				  - (id)copyWithZone:(NSZone *)zone {

					CYLUser *copy = [[[self class] allocWithZone:zone] 

						             initWithName:_name
				 					age:_age
								          sex:_sex];
						//	
						copy->_friends = [_friends mutableCopy];
					return copy;
				} 

			若CYLUser 有个数组，那么拷贝的同时也要把数组拷贝过去，放朋友对象的 set 是用 “copyWithZone:” 方法来拷贝的，这种浅拷贝方式不会逐个复制 set 中的元素。

				若需要深拷贝的话，则可像下面这样，编写一个专供深拷贝所用的方法:

					- (id)deepCopy {
					   CYLUser *copy = [[[self class] alloc]
					                    initWithName:_name
					                    age:_age
					                    sex:_sex];
					   copy->_friends = [[NSMutableSet alloc] initWithSet:_friends
					                                            copyItems:YES];
					   return copy;
					}

		property 在runtime 中是 objc_properety_t 定义如下：

			typedef struct objc_property *objc_property_t;

			objc_property是一个结构体：

				struct property_t {
				    const char *name;
				    const char *attributes;
				};

				attributes本质是objc_property_attribute_t，定义了property的一些属性，定义如下：

					/// Defines a property attribute
					typedef struct {
					    const char *name;           /**< The name of the attribute */
					    const char *value;          /**< The value of the attribute (usually empty) */
					} objc_property_attribute_t;

					attributes的具体内容包括：类型，原子性，内存语义和对应的实例变量

				例如：我们定义一个string的property@property (nonatomic, copy) NSString *string;，

					通过 property_getAttributes(property)获取到attributes并打印出来之后的结果为T@"NSString",C,N,V_string

					其中T就代表类型，可参阅Type Encodings，C就代表Copy，N代表nonatomic，V就代表对于的实例变量。

		@property 的本质是：实例变量 + 存取方法

	 @protocol 和 category 中如何使用 @property  ： 在 protocol 中使用 property 只会生成 setter 和 getter 方法声明,我们使用属性的目的,是希望遵守我协议的对象能实现该属性；

			category 使用 @property 也是只会生成 setter 和 getter 方法的声明,如果我们真的需要给 category 增加属性的实现,需要借助于运行时的两个函数：

			objc_setAssociatedObject

			objc_getAssociatedObject



/*第七条：在对象内部尽量直接访问实例变量*/

	在对象内部读取数据时，应该直接通过实例变量来读取，而写入数据时，则应该通过属性来写

	在初始化方法以及dealloc方法中，总是应该直接通过实例变量来读写数据

	有时候会使用懒加载初始化技术配置数据，这种情况要通过属性来读取数据

/*第八条：理解 “对象等同性” 这一概念*/

	NSObject 协议中有两个用于判断等同性的关键方法:

		-(BOOL)isEqual:(id)object;

		-(NSUInteger)hash; //返回的是该对象的哈希码

	这两个方法的默认实现是：当且仅当其指针值完全相同时，这两个对象才相等。 如果“isEqual：”方法判断两个对象相等，那么 “hash：”方法也必须返回同一个值；反之却不一定

	注：把多个可变对象放入集合后，再改变其中的对象，可能会导致难预料的结果。

	例：

	把两个可变数组arr1，arr2(arr1 不等于 arr2 )放入set中，然后修改arr2 ，改成和arr1 一样，此时输出set时，会发现set的两个数组是相同的对象

	相同的对象 起hash码肯定相同，反之却不一定

/*第九条：以 “类族模式” 隐藏实现细节*/

	类族  ( Class Clusters) 是抽象工厂模式在iOS下的一种实现，类族可以隐藏 “抽象基类” 背后的实现细节

	众多常用类：
			 如NSString，NSArray，NSDictionary，NSNumber , UIButton  都运作在这一模式下，它是接口简单性和扩展性的权衡体现，在我们完全不知情的情况下，

			 偷偷隐藏了很多具体的实现类，只暴露出简单的接口

	NSArray 的类簇：

		数组也是以类族的形式实现的，调用 NSArray  的 alloc 方法获取数组实例时，该方法会首先分配某个类的实例，以此来充当 “占位数组”（__NSPlacehodlerArray）

		例： 将 NSArray 的 alloc 和 init 方法拆开

			id obj1 = [NSArray alloc]; // __NSPlacehodlerArray *

			id obj2 = [NSMutableArray alloc];  // __NSPlacehodlerArray *
			
			id obj3 = [obj1 init];  // __NSArrayI *
			
			id obj4 = [obj2 init];  // __NSArrayM *

		使用 alloc 方法后并非生成一个我们期望的实例，而是生成一个中间对象（__NSPlacehodlerArray），后面的 - init  或者 - initWithXXXX 方法是发给这个中间对象的，

		 再由这个中间对象做工厂，生成正在的对象。__NSPlacehodlerArray必定用某种方式存储了它是由谁alloc出来的这个信息，才能在init的时候知道要创建的是可变数组还是不可变数组

		 经过研究发现，Foundation用了一个很贱的比较静态实例地址方式来实现，伪代码如下：

		 	static __NSPlacehodlerArray *GetPlaceholderForNSArray() {
    			
    			static __NSPlacehodlerArray *instanceForNSArray;
			
			    if (!instanceForNSArray) {
        
        				instanceForNSArray = [[__NSPlacehodlerArray alloc] init];
    				
    				}
    				
    				return instanceForNSArray;
			}

			static __NSPlacehodlerArray *GetPlaceholderForNSMutableArray() {
    
    			static __NSPlacehodlerArray *instanceForNSMutableArray;
    
    			if (!instanceForNSMutableArray) {
        			
        				instanceForNSMutableArray = [[__NSPlacehodlerArray alloc] init];
    			
    				}
    
    			return instanceForNSMutableArray;
			}

			// NSArray实现
			
			+ (id)alloc {
    			
    				if (self == [NSArray class]) {
        		
        				return GetPlaceholderForNSArray()
    				
    				}

			}

			// NSMutableArray实现
			
			+ (id)alloc {
    				
    				if (self == [NSMutableArray class]) {
        
        				return GetPlaceholderForNSMutableArray()
    				
    				}

			}

			// __NSPlacehodlerArray实现

			- (id)init {
    
    				if (self == GetPlaceholderForNSArray()) {
        
        					self = [[__NSArrayI alloc] init];
    
    				}
    
    				else if (self == GetPlaceholderForNSMutableArray()) {
        
        					self = [[__NSArrayM alloc] init];
    				}
    
    				return self;
			}

	静态不可变空对象：

			除此之外，Foundation对不可变版本的空数组也做了个小优化：

			NSArray *arr1 = [[NSArray alloc] init];

			NSArray *arr2 = [[NSArray alloc] init];
			
			NSArray *arr3 = @[];
			
			NSArray *arr4 = @[];

			NSArray *arr5 = @[@1];
			
			上边1-4号都指向了同一个对象，而arr5指向了另一个对象。若干个不可变的空数组间没有任何特异性，返回一个静态对象也理所应当。

			不仅是NSArray，Foundation中如NSString, NSDictionary, NSSet等区分可变和不可变版本的类，空实例都是静态对象（NSString的空实例对象是常量区的@""）

			所以也给用这些方法来测试对象内存管理的同学提个醒，很容易意料之外的。

/*第十条：在既有类中使用关联对象存放自定义数据*/

	关联对象：是指某个OC对象通过一个唯一的key连接到一个类的实例上。

	runtime提供关联对象的方法： 

		//关联对象
		void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy)
		
		//获取关联的对象
		id objc_getAssociatedObject(id object, const void *key)
		
		//移除关联的对象
		void objc_removeAssociatedObjects(id object)

		//参数说明
		id object：被关联的对象（如xiaoming）
		
		const void *key：关联的key，要求唯一
		
		id value：关联的对象（如dog）
		
		objc_AssociationPolicy policy：内存管理的策略（枚举）

		关联的key值：

			关于前两个函数中的 key 值是我们需要重点关注的一个点，这个 key 值必须保证是一个对象级别的唯一常量。

			有以下三种推荐的 key 值：

			a, 声明 static char kAssociatedObjectKey; ，使用 &kAssociatedObjectKey 作为 key 值;
			
			b, 声明 static void *kAssociatedObjectKey = &kAssociatedObjectKey; ，使用 kAssociatedObjectKey 作为 key 值；
			
			c,用 selector ，使用 getter 方法的名称作为 key 值 （推荐使用）

		内存管理策略枚举：

			OBJC_ASSOCIATION_ASSIGN = 0,          
			
			OBJC_ASSOCIATION_RETAIN_NONATOMIC = 1, 
			
			OBJC_ASSOCIATION_COPY_NONATOMIC = 3,  
			
			OBJC_ASSOCIATION_RETAIN = 01401,       
			
			OBJC_ASSOCIATION_COPY = 01403

		当对象被释放时，会根据这个策略来决定是否释放关联的对象，当策略是RETAIN/COPY时，会释放（release）关联的对象，当是ASSIGN，将不会释放。

		关联对象的释放时机与移除时机并不总是一致，用关联策略 OBJC_ASSOCIATION_ASSIGN 进行关联对象时，很早就会被释放了，但是并没有被移除，而再使用这个关联对象时就会造成 Crash 。

		注：void objc_removeAssociatedObjects(id object) 函数我们一般是用不上的，因为这个函数会移除一个对象的所有关联对象，将该对象恢复成“原始”状态。

		        这样做就很有可能把别人添加的关联对象也一并移除，这并不是我们所希望的。所以一般的做法是通过给 objc_setAssociatedObject 函数传入 nil 来移除某个已有的关联对象。

	例：

		iOS 中给 UIAlert 添加关联对象

		_alertView = [[UIAlertView alloc] initWithTitle:@"Alert" message:@"This is deprecated?" 
									     delegate:self 
									cancelButtonTitle:@"Cancel" 
								otherButtonTitles:@"Ok", nil];
    
    		void (^block)(NSInteger) = ^(NSInteger buttonIndex) {
        
        			if (buttonIndex == 0) {
            		
            			//  doCancel
        			} else {
            
            			//  doOk
        			}
    		};
    
    		objc_setAssociatedObject(self.alertView, MyAlertViewKey, block, OBJC_ASSOCIATION_COPY);

		#pragma -mark UIAlertViewDelegate
		
		- (void)alertView:(UIAlertView *)alertView clickedButtonAtIndex:(NSInteger)buttonIndex {
    			
    			void (^block)(NSInteger) = objc_getAssociatedObject(alertView, MyAlertViewKey);
    			
    			block(buttonIndex);
		}

/*第十 一条：理解 objc_msgSend 的作用*/
	
	objc_msgSend(someObject, @selector(methodName:), parameter);

	这是个参数可变的函数，能接受两个和两个以上的参数，第一个参数代表接收者，第二个参数是选择子 (SEL是选择子的类型)，后续的参数就是消息中的那些参数，其顺序不变

	objc_msgSend 先在接受者的方法列表里面搜索，若能找到与 methodName 相符合的，则跳到该方法的实现代码执行；若找不到则沿着继承体系向上执行，找到之后再跳转；

	若始终找不到，则执行消息转发操作。找到之后将该方法名缓存到 “快速映射表” 中，方便下次使用；

/*第十 二条：理解消息转发机制*/

	对象接收到无法理解的方法时就会执行消息转发机制

	消息转发步骤：

		第一阶段会先征询接受者所属的类，看是否能动态添加方法，以处理当前这个未知的方法，这叫 “动态方法解析”；

		第二阶段如果运行期已经把第一阶段执行完了，那么接受者就无法动态添加方法了，此时就会选择其他手段进行处理：

			首先会请接受者看看有没有其他对象能处理这条消息，若有，则运行期系统会把消息转给该对象，一切正常；若没有

			运行期系统会把与消息有关的全部细节（包括方法名称，参数，接受者）封装成NSInvocation 对象中，再给接受者一次机会，让其设法解决这个未处理的消息

			如果没有实现动态转发机制或者转发失败且实现了消息转发机制。就会进入消息转发流程。否则程序Crash.

		消息转给其他对象步骤：

			当前类会调用这个方法 - (id)forwardingTargetForSelector:(SEL)aSelector；若找到能响应未知方法的对象就将该对象返回否则返回 nil  （可以用这个模拟多重继承）

		动态方法解析：

			对象接收到无法解读的消息后会调用其所属类的下列类方法：

				+ (BOOL)resolveInstanceMethod:(SEL)name; 参数就是未知的方法，表示此类是否能够新增一个方法来处理这个未知的方法 （选择器），在

				继续往下执行消息转发机制之前，此类有机会新增一个处理该方法选择子的方法。如果该选择子是类方法，则运行期系统会调用另一个与之

				类似的方法 + (BOOL)resolveClassMethod:(SEL)name; 使用该方法的前提是处理方法已经写好，等着运行期系统插入在该类里面

		完整的消息转发：

			此时说明上述情况都不行，到这一步能做的就是启用完整的消息转发机制；首页创建 NSInvocation 对象，将该选择子所相关的全部细节封装到此，

			包括选择子、目标、以及参数；出发这个对象时，会调用 - (void)forwardInvocation:(NSInvocation *)invocation

			如果本类不响应 则会沿着继承体系往上找，直到 NSObject，如果还不响应 则会调用  doesNotRecognizeSelector 抛出异常。

/*第十 三条：用 "方法调配技术" 调试 "黑盒方法"*/

	方法调配： （method swizzling）

		通过runtime 替换两个方法的实现

			//交换方法
			void method_exchangeImplementations(Method m1, Method m2);

			//获取某个方法
			Method class_getInstanceMethod(Class aClass, SEL aSelector);

		为现有的方法添加新功能；

/*第十 四条：理解 "类对象"的用意 */

	类型信息查询：在运行期检视对象类型，这个特性内置于 Foundation 框架中的 NSObject 协议里；凡是由公共根类（ NSObject 和 NSProxy ）继承而来的对象都要继承此协议

	Objective-C 对象的本质：是指向某块内存数据的指针；所以声明变量时，类型后面要跟个 ” * “字符。 对象类型 id 本身就是指针；

	查询类型信息：

		isMemberOfClass: 查询对象是否为某个特定的实例；

		isKindOfClass: 判断对象是否为某个类或者其派生类的实例

		这两个方法是使用 isa 指针获取对象的所属类，然后通过 super_class  指针在继承体系中游走；

	每一个实例都有一个指向class 对象的指针，用以表明其类型，这些class对象则构成类的继承体系

	如果对象类型无法在编译器确定，那么就应该使用类型信息查询方法来探知

	尽量使用类型信息查询方法来确定对象类型，而不要直接比较类对象，因为某些类对象可能实现了消息转发功能

/*第十 五条：使用前缀避免命名空间冲突 */

	OC没有内置的命名空间机制。避免此问题的方法 就是加前缀

/*第十 六条：提供全能初始化方法 */

/*第十 七条：实现description方法 */

	实现description方法返回一个有意义的字符串，用以描述该实例

	若想调试时打印出更详尽的信息，则应实现 debugDescription 方法

/*第十 八条：尽量使用不可变方法 */

/*第十 九条：使用清晰而协调的命名方式 */

/*第二 十条：为私有方法名添加前缀 */

	给私有方法添加前缀可以很容易同公共方法区分开

	不要用单个下划线做前缀，这种做法是苹果公司预留用的

/*第二 十 一条：理解OC的错误类型 */

/*第二 十 二条：理解NSCopying协议 */

	NSCopying协议包含一个方法：- (id)copyWithZone:(NSZone *)zone；

	例：
		- (id)copyWithZone:(nullable NSZone *)zone {
    
 			 ViewControllerC *copyVC = [[[self class] allocWithZone:zone] init];
    
    			 return copyVC;
		}
	NSMutableCopying 协议与之类似

/*第二 十 三条：通过委托与数据源协议进行对象间通信 */

/*第二 十 四条：将类的实现分散到便于管理的数个分类之中  */

/*第二 十 五条：总是为第三方的分类名称加前缀 */

/*第二 十 六条：请勿在分类中声明属性*/

	分类中可以声明属性，但是编译器无法为这些属性自动合成实例变量，只是单纯的生成存取方法，可以通过runtime来实现，但是不推荐

	那样做在内存管理问题上很容易出问题；

/*第二 十 七条：使用 ”class-continuation 分类 （延展）“ 隐藏实现细节 */ 

	这样可以在里面定义方法和实例变量，在实现文件里面这么写

	如果某属性在主接口中声明为 ”只读“，而类的内部又要用设置方法修改此属性，那么就在 ”class-continuation ” 中将其扩展为 “可读写”

	把私有方法的原型声明在 ”class-continuation 里面

/*第二 十 八条：通过协议实现匿名对象*/

/*第二 十 九条：理解引用计数*/
	
	/*
	.	
	.
	.
	.

	*/

/*第三 十 七条 ---  第四 十条：Block*/

	block 的数据结构定义如下：

		struct Block_descriptor {
    		
    			unsigned long int reserved;
    		
    			unsigned long int size;
    		
    			void (*copy)(void *dst, void *src);
    		
    			void (*dispose)(void *);
		};
		
		struct Block_layout {
    		
    			void *isa;
    		
    			int flags;
    		
    			int reserved;
    		
    			void (*invoke)(void *, ...);
    		
    			struct Block_descriptor *descriptor;
    			
    			/* Imported variables. */
		};
	
	参数说明：

		isa 指针，所有对象都有该指针，用于实现对象相关的功能。
		
		flags，用于按 bit 位表示一些 block 的附加信息，本文后面介绍 block copy 的实现代码可以看到对该变量的使用。
		
		reserved，保留变量。
		
		invoke，函数指针，指向具体的 block 实现的函数调用地址。
		
		descriptor，指向结构体的指针， 表示该 block 的附加描述信息，主要是 size 大小，以及 copy 和 dispose 函数的指针。
		
		variables，capture 过来的变量，block 能够访问它外部的局部变量，就是因为将这些变量（或变量的地址）复制到了结构体中。

	在 Objective-C 语言中，一共有 3 种类型的 block：
		
		_NSConcreteGlobalBlock 全局的静态 block，不会访问任何外部变量，不会捕捉任何状态。
		
		_NSConcreteStackBlock 保存在栈中的 block，当函数返回时会被销毁。
		
		_NSConcreteMallocBlock 保存在堆中的 block，当引用计数为 0 时会被销毁。

	Block捕获的外部变量可以改变值的是静态变量，静态全局变量，全局变量。

	在Block中改变变量值有2种方式，一是传递内存地址指针到Block中，二是改变存储区方式(__block)。

/*第四 十 一条：GCD*/

	多线程要执行同一份代码：GCD之前： 1，使用 @synchronized () {} 方式给代码加锁，滥用的话 会降低代码效率；

						      2， 直接使用NSLock对象 进行加锁

	GCD 概要:

		基于队列的并发编程 API

		公开的5个不同队列 ： 运行在主线程的主队列 main queue；

					 3个不同优先级的后台队列（High Priority Queue，Default Priority /* praɪ'ɔrəti */ Queue，Low Priority Queue）

					 优先级更低的后台队列 Background Priority Queue（用于I/O）

		可创建自定义队列：串行或并行；自定义队列一般放在 Default Priority Queue和Main Queue里。

		操作是在多线程上还是单线程主要是看队列的类型和执行方法，并行队列异步执行才能在多线程，并行队列同步执行就只会在这个并行队列在队列中被分配的那个线程执行。

	系统标准的两个队列：

		//全局队列，一个并行的队列
		
		dispatch_get_global_queue

		//主队列，主线程中的唯一队列，一个串行队列
		
		dispatch_get_main_queue

	自定义队列：

		//串行队列
		dispatch_queue_create("com.starming.serialqueue", DISPATCH_QUEUE_SERIAL)
		
		//并行队列
		dispatch_queue_create("com.starming.concurrentqueue", DISPATCH_QUEUE_CONCURRENT)

	同步异步线程创建：

		//同步线程
		dispatch_sync(..., ^(block))
		
		//异步线程
		dispatch_async(..., ^(block))

	队列：

		Serial ：同时只能执行一个任务，Serial queue 常用同步访问特定的资源或数据；当创建多个 Serial queue 时，各自是同步的，但是各个之间是并发执行的

		Main dispatch queue : 全局可用的 serial queue，在应用程序主线程上执行

		Concurrent ： 又叫global dispatch queue 可以并发执行多个任务，执行顺序是随机的；系统提供四个全局并发队列，这四个队列有对应的优先级；用户是不能够创建全局队列的，只能获取。

				dipatch_queue_t queue;
				
				queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH,0);

		自定义队列：创建自己定义的队列，可以用dispatch_queue_create函数，函数有两个参数，第一个自定义的队列名，

				第二个参数是队列类型，默认NULL或者DISPATCH_QUEUE_SERIAL的是串行，

				参数为DISPATCH_QUEUE_CONCURRENT为并行队列。

				dispatch_queue_t queue
				
				queue = dispatch_queue_create("com.starming.gcddemo.concurrentqueue", DISPATCH_QUEUE_CONCURRENT);

		自定义队列的优先级：可以通过 dipatch_queue_attr_make_with_qos_class 或 dispatch_set_target_queue 方法设置队列的优先级

				例：
					//dipatch_queue_attr_make_with_qos_class
					
					dispatch_queue_attr_t attr = dispatch_queue_attr_make_with_qos_class(DISPATCH_QUEUE_SERIAL, QOS_CLASS_UTILITY, -1);
					
					dispatch_queue_t queue = dispatch_queue_create("com.starming.gcddemo.qosqueue", attr);

					//dispatch_set_target_queue
					dispatch_queue_t queue = dispatch_queue_create("com.starming.gcddemo.settargetqueue",NULL); //需要设置优先级的queue

					dispatch_queue_t referQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_LOW, 0); //参考优先级
	
					dispatch_set_target_queue(queue, referQueue); //设置queue和referQueue的优先级一样

				自定义的 Dispatch Queue 无论是 serial 还是 concurrent  其优先级都与  dispatch_get_global_queue 的优先级相同

	/*具体api*/

		dispatch_once用法: 设置单例；dispatch_once_t 要是全局或 static 变量，保证 dispatch_once_t 只有一份实例

				例：
					+ (UIColor *)boringColor;
					{
					     static UIColor *color;
					     //只运行一次
					     static dispatch_once_t onceToken;
					     dispatch_once(&onceToken, ^{
					          color = [UIColor colorWithRed:0.380f green:0.376f blue:0.376f alpha:1.000f];
					     });
					     return color;
					}

		dispatch_async：可以避免界面会被一些耗时的操作卡死，比如读取网络数据，大数据IO，还有大量数据的数据库读写，这时需要在另一个线程中处理，然后通知主线程更新界面，GCD使用起来比NSThread和NSOperation方法要简单方便。

				例：
					//代码框架
					dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
					    
					     // 耗时的操作
					    
					    //切换到主线程队列进行 UI 更新
					     dispatch_async(dispatch_get_main_queue(), ^{
					       
					          // 更新界面
					     
					     });
					});

		dispatch_after延后执行：dispatch_after只是延时提交block，不是延时立刻执行。

				例：
					double delayInSeconds = 2.0;
					
					dispatch_time_t popTime = dispatch_time(DISPATCH_TIME_NOW, (int64_t) (delayInSeconds * NSEC_PER_SEC));
					
					dispatch_after(popTime, dispatch_get_main_queue(), ^(void){
					
					          [self bar];
					});
		
		暂停/恢复某个队列：

			dispatch_suspend(queue) //暂停某个队列  
			
			dispatch_resume(queue)  //恢复某个队列  

		GCD 定时器的实现:

			//获取队列、也可创建队列
			
			dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_VNODE_DELETE, 0);
			
			// 创建一个 timer 类型定时器 （ DISPATCH_SOURCE_TYPE_TIMER）
			
			dispatch_source_t timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, queue);

			//设置定时器的各种属性（何时开始，间隔多久执行）
			
			// GCD 的时间参数一般为纳秒 （1 秒 = 10 的 9 次方 纳秒）
			
			// 指定定时器开始的时间和间隔的时间
			
			dispatch_time_t start = dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1.0 * NSEC_PER_SEC));
			
			uint64_t interval = (uint64_t)(1.0 * NSEC_PER_SEC);
			
			dispatch_source_set_timer(timer, start, interval, 0);

			//设置任务回调
			 dispatch_source_set_event_handler(timer, ^{
			     
			     NSLog(@"------------%d", count);
			     
			     count++;
			     
			     if (count == 4) {
			     
			         // 取消定时器
			     
			         NSLog(@"+++++%d", count);
			     
			         dispatch_cancel(timer);
			     
			         timer = nil;
			     }
			 });

			 停止 Dispatch Timer 有两种方法，一种是使用 dispatch_suspend，另外一种是使用 dispatch_source_cancel。

			 	dispatch_suspend 严格上只是把 Timer 暂时挂起，它和 dispatch_resume 是一个平衡调用，两者分别会减少和增加 dispatch 对象的挂起计数；//暂定timer

			 	dispatch_source_cancel 则是真正意义上的取消 Timer。被取消之后如果想再次执行 Timer，只能重新创建新的 Timer。这个过程类似于对 NSTimer 执行 invalidate

			/**
			     1,若timer是局部变量，则需要写 dispatch_cancel(timer) 或者 timer = nil;才能启动定时器，否则不会走任务回调
			     2,若timer是全局变量，则无论写不写 dispatch_cancel(timer) 或者 timer = nil
			       定时器都会走任务回调；(一般情况下要写)
			     */
		

		dispatch_barrier_async 使用 Barrier Task 方法 Dispatch Barrier 解决多线程并发读写同一个资源发生死锁: 

				Dispatch Barrier 确保提交的闭包是指定队列中在特定时段唯一在执行的一个。

				在所有先于Dispatch Barrier的任务都完成的情况下这个闭包才开始执行。

				轮到这个闭包时barrier会执行这个闭包并且确保队列在此过程不会执行其它任务。

				闭包完成后队列恢复。

				需要注意dispatch_barrier_async 只在自己创建的队列上有这种作用，在全局并发队列和串行队列上，效果和dispatch_sync一样

		dispatch_apply进行快速迭代:  类似for循环，但是在并发队列的情况下dispatch_apply会并发执行block任务。dispatch_apply能避免线程爆炸，因为GCD会管理并发

				例：

					// dispatch_apply 类似for循环
					    dispatch_queue_t concurrentQueue = dispatch_queue_create("QFzf.com", 0);
					    
					    dispatch_apply(10, concurrentQueue, ^(size_t x) {
					        
					        NSLog(@"%zu",x);//从0 打印到9
					    });

		Block组合Dispatch_groups： 当group里所有事件都完成GCD API有两种方式发送通知，第一种是dispatch_group_wait，会阻塞当前进程，等所有任务都完成或等待超时。

														  第二种方法是使用dispatch_group_notify，异步执行闭包，不会阻塞。

				例：
					    dispatch_group_t goupTest = dispatch_group_create();
    
					    dispatch_queue_t t1 = dispatch_queue_create("QFZF", 0);
					    
					    dispatch_queue_t t2 = dispatch_queue_create("QFZF", 0);
					    
					    dispatch_group_async(goupTest, t1, ^{
					       
					        [NSThread sleepForTimeInterval:3];
					       
					        NSLog(@" t1  3 late");
					    });
					    
					    dispatch_group_async(goupTest, t2, ^{
					       
					        [NSThread sleepForTimeInterval:3];
					       
					        NSLog(@" t2  3 late");
					    });
					    
					//    dispatch_group_wait(goupTest, DISPATCH_TIME_FOREVER);
					    
					    // dispatch_group_notify 有回调 且不阻塞当前线程
					    dispatch_group_notify(goupTest, t1, ^{
					       
					        NSLog(@"dispatch_group_notify");
					    });

				dispatch_group_async等价于dispatch_group_enter() 和 dispatch_group_leave()的组合。
				
				dispatch_group_enter() 必须运行在 dispatch_group_leave() 之前。
				
				dispatch_group_enter() 和 dispatch_group_leave() 需要成对出现的

		Dispatch Block：队列执行任务都是block的方式

				创建 block

					例：
						dispatch_queue_t queue = dispatch_queue_create("QFZF", 0);
    
						    dispatch_block_t block = dispatch_block_create(0, ^{
						        
						        NSLog(@"block");
						    });
						    
						    dispatch_async(queue, block);

				dispatch_block_wait：可以根据dispatch block来设置等待时间，参数DISPATCH_TIME_FOREVER会一直等待block结束

					例：
						dispatch_block_wait(^{
						        
						        //  do something

						    }, DISPATCH_TIME_FOREVER);

				dispatch_block_cancel：iOS8后GCD支持对dispatch block的取消

					例：
						dispatch_queue_t queue = dispatch_queue_create("QFZF", 0);
    
						    dispatch_block_t block = dispatch_block_create(0, ^{
						        NSLog(@"block");
						    });
						    
						    dispatch_block_t block2 = dispatch_block_create(0, ^{
						        NSLog(@"block2");
						    });

						dispatch_async(queue, block);

						dispatch_async(queue, block2);

						dispatch_block_cancel(block2);

						只会输出  “block”，“block2” 不会输出

		Dispatch Semaphore ：持有计数的信号，是多线程编程中的计数类型信号  计数为0时等待，计数为1或者大于1时，计数件减1而不等待

			与之相关的三个函数 ：dispatch_semaphore_create 

						   dispatch_semaphore_signal

						   dispatch_semaphore_wait

				dispatch_semaphore_create 函数的声明为：

						　　dispatch_semaphore_t  dispatch_semaphore_create(long value);

						　　传入的参数为long，输出一个dispatch_semaphore_t类型且值为value的信号量。

						　　值得注意的是，这里的传入的参数value必须大于或等于0，否则dispatch_semaphore_create会返回NULL

				dispatch_semaphore_signal的声明为：

						　　long dispatch_semaphore_signal(dispatch_semaphore_t dsema)

						　　这个函数会使传入的信号量dsema的值加1

						       dispatch_semaphore_signal的返回值为long类型，当返回值为0时表示当前并没有线程等待其处理的信号量，其处理

						　　的信号量的值加1即可。当返回值不为0时，表示其当前有（一个或多个）线程等待其处理的信号量，并且该函数唤醒了一

						　　个等待的线程（当线程有优先级时，唤醒优先级最高的线程；否则随机唤醒）。

						　　dispatch_semaphore_wait的返回值也为long型。当其返回0时表示在timeout之前，该函数所处的线程被成功唤醒。

						　　当其返回不为0时，表示timeout发生

				dispatch_semaphore_wait的声明为：

						　　long dispatch_semaphore_wait(dispatch_semaphore_t dsema, dispatch_time_t timeout)；

						　　这个函数会使传入的信号量dsema的值减1；

						　　这个函数的作用是这样的，如果dsema信号量的值大于0，该函数所处线程就继续执行下面的语句，并且将信号量的值减1；

						　　如果desema的值为0，那么这个函数就阻塞当前线程等待timeout（注意timeout的类型为dispatch_time_t，

						　　不能直接传入整形或float型数），如果等待的期间desema的值被dispatch_semaphore_signal函数加1了，

						　　且该函数（即dispatch_semaphore_wait）所处线程获得了信号量，那么就继续向下执行并将信号量减1。

						　　如果等待期间没有获取到信号量或者信号量的值一直为0，那么等到timeout时，其所处线程自动执行其后语句

		Dispatch IO 文件操作：dispatch io读取文件的方式类似于下面的方式，多个线程去读取文件的切片数据，对于大的数据文件这样会比单线程要快很多

					例：
						dispatch_async(queue,^{/*read 0-99 bytes*/});
						
						dispatch_async(queue,^{/*read 100-199 bytes*/});
						
						dispatch_async(queue,^{/*read 200-299 bytes*/});
			
			dispatch_io_create：创建dispatch io
			
			dispatch_io_set_low_water：指定切割文件大小
			
			dispatch_io_read：读取切割的文件然后合并。



/*第四 十 二条：多用 GCD 少用 performSelector*/

	ARC 环境下，使用performSelector 方法为什么会出现内存泄露：

		1、使用该方法时，编译器并不知道要调用的方法是什么，因此，也就不了解其方法签名和返回值；由于编译器

		不知道方法名，所以就没有办法使用ARC的内存管理规则来判定返回值是不是应该释放，所以ARC采用了比较

		谨慎的做法，就是不添加释放操作，然而这么做可能会导致内存泄露，因为该方法在返回对象时可能已经将其保留。

		2、该方法返回值是 id 类型， 如果调用方法返回值是 数值类型，那么还要经过一列席的转换操作；如果目标函数的

		参数类型是数值类型，那么使用这个方法也要经过一系列的转换；如果目标函数的参数个数大于三个，则不能使用这个

		方法。 

/*第四 十 四条：通过Dispatch Group机制，根据系统资源状况来执行任务*/

	一些列任务可以归入一个dispatch group中，开发者可以在这组任务执行完毕之后获得通知

	通过 dispatch group ，可以在并发式派发队列里同时执行多项任务。此时GCD会根据系统资源状况来调度这些并发执行的任务。

		开发者若是自己来实现此功能，则需要编写大量代码

/*第四 十 五条：使用 dispatch_once 来执行只需运行一次的线程安全代码*/

	经常需要编写 “只需要执行一次的线程安全代码”，通过GCD提供的 dispatch_once 函数很容易实现

	标记应该声明在static 或者 global作用域中，这样的话，在把只需要执行一次的block传递给 dispatch_once函数时，传进去的标记也是相同的

/*第四 十 六条：不要使用 dispatch_get_current_queue*/

	dispatch_get_current_queue 函数的行为常常与开发者所预期的不同，此函数已经废除，只应做调试之用；

	由于派发队列是按层级结构来组织的，所以无法单用某个队列对象来描述 “当前队列”这一概念

	dispatch_get_current_queue函数用于解决不可重入的代码所引发的死锁，然而能用此函数解决的问题，通常也能用“队列特定数据”来解决

/*第四 十 七条：熟悉系统框架*/

/*第四 十 八条：多用枚举块，少用for循环*/

/*第四 十 九条：对自定义其内存管理语义的collection使用无缝桥接*/

	无缝桥接：可以使定义于Foundation框架中的OC类和定义与CoreFoundation框架中的C数据结构直接互相转换；

	例：

		NSArray *anNSArray = @[@1,@2,@3];

		CFArrayRef aCFArray = (__briage CFArrayRef)anNSArray;

		NSLog(@"Size of array = %li", CFArrayGetCount(aCFArray));  // 输出 Size of array =  3

	Foundation框架中的OC对象 转 CoreFoundation框架中的数据结构（c语言无对象）：
	
		__briage 关键字说明：ARC仍具备这个OC对象的所有权；

		__briage__retain则与相反，意味着ARC放弃该对象的所有权，释放该对象时，需要手动调用对应的 CFRelease(aCFArray) 方法释放

	CoreFoundation框架中的数据结构（c语言无对象） 转  Foundation框架中的OC对象：

		__briage__transfer: 把目标对象转成OC对象的同时，ARC也获得该对象的所有权

/*第五 十条：构建缓存时首选 NSCache 而非 NSDictionary*/

/*第五 十 一条：精简initialize 和 load 的实现代码*/

	这两个方法是NSObject的两个类方法；他们会在运行期自动调用

	+ (void)initialize 方法：

		1，这个方法是在该类接收到第一个消息之前被调用；说明：对于NSObject 的 Runtime 机制而言，如果像普通方法那样调用该类的 +(void)load 方法，则会引起 + (void)initialize的调用；

			反之，没有向NSObject 发送第一个消息，+ (void)initialize就不会被自动调用

		2，在应用程序的生命周期中，runtime 只会向每一个类发送一次 + (void)initialize 消息；如果该类是子类，且该子类中没有实现 + (void)initialize 消息，或者该子类显示的调用了父类的

			实现 [super initialize] ,那么则会调用父类的实现。也就是说父类的 + (void)initialize 方法可能被调用多次。

		3，如果类包含分类，且分类重写了+ (void)initialize方法，怎么则会调用分类的 initialize 实现，原类的该方法实现则不会别调用；这种机制同 NSObject 的其他方法一样，分类中的方法的优先级

			高

		4，父类的 initialize 方法优先于子类的 initialize 方法调用

	+ (void)load  方法：

		1，+ (void)load 会在类或者类的分类添加到 Objective-c runtime 时调用，该调用发生在 application:willFinishLaunchingWithOptions: 调用之前调用。

		2，父类的 + (void)load 方法先于子类的+ (void)load 方法调用；类本身的+ (void)load 方法先于分类的+ (void)load 调用；

	要点：

		1，在加载阶段，如果类实现了 load 方法，那么系统就会调用它。分类里面也可以定义此方法，类的load方法比分类中的先调用。与其他方法不同，load 方法 不参与复写机制

		2，首次使用某个类之前，系统会向其发送 initialize 消息。由于此方法遵守普通的复写机制，所以通常应该在里面判断当前要初始化的是哪个类

		3， load 与 initialize 方法都应该写的精简，这样有助于程序的响应能力

		4， 无法再编译器设定的全局变量，可以方法initialize 方法里面初始化

/*第五 十 二条：别忘了 NSTimer 会保留其目标对象*/

	NSTimer 对象会保留其目标，直到计时器本身失效，调用 invalidate 方法也可令计时器失效，另外，一次性的计时器在出发完任务之后也会失效

	反复执行任务的计时器，很容易引入保留环，如果这种计时器的目标对象又保留了计时器本身，那肯定会导致保留环。

	可以扩充NSTimer 的功能，使用 Block 来打破保留环，不过除非 NSTimer 将来在公共接口里面提供此功能，否则必须要创建分类，将相关代码加入其中
