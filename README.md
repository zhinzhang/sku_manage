# iOS 商品库存管理 SKU的选择
##项目需求
简单说一下我的需求:服务端给我传了两组数据，要求实现淘宝选择商品规格的功能。
![选择对应商品规格，加入购物车](https://upload-images.jianshu.io/upload_images/3119643-6a8471ef5c4a626f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
//这是所有sku的数组，spec_array为每一条sku所包含的所有规格
"sku_list": [{
			"id": "5dea1d944a723c535f9a9354",
			"modified_date": "2019-12-06 17:21:24",
			"corp_code": "C00000",
			"modifier_id": "5a7c2050ca2d3c7baf85efd7",
			"spec_array": [{
				"spec_id": "5d00cf245d48ca10241adca3",
				"spec_item_name": "1颗\/约0.6分",
				"name": "主石重量",
				"spec_item_id": "5d00cf245d48ca10241adca8"
			}, {
				"spec_id": "5d00cca55d48ca10241adca0",
				"spec_item_name": "钻石",
				"name": "材质",
				"spec_item_id": "5d00cca55d48ca10241adca2"
			}, {
				"spec_id": "5de620914a723c0d7a971aeb",
				"spec_item_name": "A1",
				"name": "尺码",
				"spec_item_id": "5de620914a723c0d7a971aec"
			}],
			"last_stock": "15",
			"is_active": "Y",
			"created_date": "2019-12-06 17:21:24",
			"stock": 15,
			"market_price": 100,
			"price": 88,
			"product_id": "5ddf766f4a723c3cb8deffdb",
			"creator_id": "5a7c2050ca2d3c7baf85efd7"
		},
......
]
```
```
//既然sku_list中有所有规格列表了，那么还有一个spec_list就是所有可以选择的规格的数组了, value_list为每一个规格的属性数组
"spec_list": [{
		"spec_id": "5de620914a723c0d7a971aeb",
		"spec_name": "尺码",
		"value_list": [{
			"spec_item_id": "5de620914a723c0d7a971aed",
			"spec_item_name": "A2"
		}, {
			"spec_item_id": "5de620914a723c0d7a971aec",
			"spec_item_name": "A1"
		}, {
			"spec_item_id": "5de620914a723c0d7a971aee",
			"spec_item_name": "A3"
		}]
	}, {
		"spec_id": "5d00cf245d48ca10241adca3",
		"spec_name": "主石重量",
		"value_list": [{
			"spec_item_id": "5d00cf245d48ca10241adca6",
			"spec_item_name": "1颗\/约4.9分"
		}, {
			"spec_item_id": "5d00cf245d48ca10241adca8",
			"spec_item_name": "1颗\/约0.6分"
		}]
	}, {
		"spec_id": "5d00cca55d48ca10241adca0",
		"spec_name": "材质",
		"value_list": [{
			"spec_item_id": "5d00cca55d48ca10241adca2",
			"spec_item_name": "钻石"
		}, {
			"spec_item_id": "5d00cca55d48ca10241adca1",
			"spec_item_name": "K金"
		}]
	}]
```
##大佬[OrangeAL](https://www.jianshu.com/u/80c622a1fe98)的文章[iOS-SKU商品规格组合算法详解](https://www.jianshu.com/p/295737e2ac77)提供了一种解决思路

###数据处理
首先将数据整理成有效数据，将有效sku保存成一个按spec_list的顺序排列的数组**_skuData**
```
_skuData = [[NSMutableArray alloc]init];
    NSArray *spec_list = _productModel.spec_list;
    if (spec_list) {
        for (int i = 0; i<_productModel.skuList.count; i++) {
            NSMutableDictionary *skuObj = [[NSMutableDictionary alloc]init];
            NSDictionary *skuItem = _productModel.skuList[i];
            [skuObj setValue:skuItem[@"id"] forKey:@"sku_id"];
            NSMutableArray *specItems = [[NSMutableArray alloc]init];
            NSDictionary *spec_array = skuItem[@"spec_array"];
            for (NSDictionary *item in spec_array) {
                [specItems addObject:item[@"spec_item_id"]];
            }
            NSMutableArray *tempArray = [[NSMutableArray alloc]init];
            for (int j = 0; j < spec_list.count; j++) {
                NSArray *value_list = spec_list[j][@"value_list"];
                [value_list enumerateObjectsUsingBlock:^(id  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
                    if ([specItems containsObject:obj[@"spec_item_id"]]) {
                        [tempArray addObject:obj[@"spec_item_id"]];
                    }
                }];
            }
            NSString *spec_item_id_list = [tempArray componentsJoinedByString:@","];
            [skuObj setValue:spec_item_id_list forKey:@"spec_item_id_list"];
            [skuObj setValue:skuItem[@"stock"] forKey:@"store"];
            [skuObj setValue:skuItem[@"price"] forKey:@"price"];
            [_skuData addObject:skuObj];
        }
    }
```
```
_skuData的值为
[{
    price = 88;//价格
    "sku_id" = 5dea1d944a723c535f9a9354;
    "spec_item_id_list" = "5de620914a723c0d7a971aec,5d00cf245d48ca10241adca8,5d00cca55d48ca10241adca2";//当前sku所包含的规格（按spec_list中规格顺序排序，[尺码id,主石重量id,材质id]）
    store = 15;//库存
},
{
    price = 88;
    "sku_id" = 5dea1d944a723c535f9a9355;
    "spec_item_id_list" = "5de620914a723c0d7a971aee,5d00cf245d48ca10241adca8,5d00cca55d48ca10241adca2";
    store = 15;
},
{
    price = 88;
    "sku_id" = 5dea1d944a723c535f9a9356;
    "spec_item_id_list" = "5de620914a723c0d7a971aed,5d00cf245d48ca10241adca6,5d00cca55d48ca10241adca1";
    store = 15;
},
{
    price = 88;
    "sku_id" = 5dea1d944a723c535f9a9357;
    "spec_item_id_list" = "5de620914a723c0d7a971aee,5d00cf245d48ca10241adca6,5d00cca55d48ca10241adca1";
    store = 15;
}]
```

##自定义collectionView
```
#import <UIKit/UIKit.h>

NS_ASSUME_NONNULL_BEGIN

@class ItemView;

/// 视图的协议
@protocol ItemViewDelegate <NSObject>

@optional
-(void)didSelected:(ItemView *)itemView;

@end

/// 格子
@interface ItemView : UIView
/// 按钮事件代理
@property (nonatomic, weak) id<ItemViewDelegate> delegate;
/// 是否是多行
@property (nonatomic, assign) BOOL isMultiLines;
/// 位置
@property (nonatomic, strong) NSIndexPath *indexPath;
/// 设置选中状态
@property (nonatomic, assign) BOOL itemSelected;
/// 设置不可选
@property (nonatomic, assign) BOOL itemDisable;
/// 设置 文本框的文字 文字的最小宽度 最大宽度 文字的字体
- (void)setText:(NSString *)text minWith:(CGFloat)minWith maxWith:(CGFloat)maxWith font:(UIFont *)font;

@end

NS_ASSUME_NONNULL_END
```
```
#import "ItemView.h"

@interface ItemView ()

/// 文本框
@property (nonatomic, strong) UILabel *label;
/// 装载内容
@property (nonatomic, strong) UIView *contentView;
/// 最小宽度
@property (nonatomic, assign) CGFloat minWidth;
/// 选中的标题颜色  边框色
@property (nonatomic, strong) UIColor *selectedTitleColro;
/// 选中背景色
@property (nonatomic, strong) UIColor *selectedBgColor;
/// 未选中背景色
@property (nonatomic, strong) UIColor *unSelectedBgColor;
/// 未选中的标题颜色
@property (nonatomic, strong) UIColor *unSelectedTitleColor;
/// 不可点击的标题颜色
@property (nonatomic, strong) UIColor *disableTitleColor;
/// 点击事件(手势)
@property (nonatomic, strong) UITapGestureRecognizer *g;

@end

@implementation ItemView

- (instancetype)init {
    
    self = [super init];
    if (self) {
        
        _disableTitleColor = TextFColor;
        _selectedBgColor = ColorB;
        _unSelectedBgColor = ColorG2;
        _unSelectedTitleColor = TitleBColor;
        _selectedTitleColro = ButtonBColor;
        _minWidth = 70;
        
        _contentView = [[UIView alloc] init];
        _contentView.userInteractionEnabled = NO;
        _contentView.layer.cornerRadius = 4;
        _contentView.frameHeight = 28;
        [self addSubview:_contentView];
        
        _label = [[UILabel alloc] init];
        _label.textAlignment = NSTextAlignmentCenter;
        _label.userInteractionEnabled = NO;
        _label.font = ButtonBFont;
        _label.numberOfLines = 0;
        [self addSubview:_label];
        
        _isMultiLines = NO;

        _g = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(click)];
        _g.numberOfTapsRequired = 1;
        [self addGestureRecognizer:_g];
    }
    return self;
}

- (void)click{
    if (self.delegate && [self.delegate respondsToSelector:@selector(didSelected:)]) {
        [self.delegate didSelected:self];
    }
}

/// 设置 文本框的文字 文字的最小宽度 文字的字体
- (void)setText:(NSString *)text minWith:(CGFloat)minWith maxWith:(CGFloat)maxWith font:(UIFont *)font {
    CGFloat offset = 10;
    
    NSDictionary *dic = @{NSFontAttributeName: font};
    
    CGFloat maxW = [UIScreen mainScreen].bounds.size.width - offset * 2;
    
    CGSize stringSize = [text boundingRectWithSize:CGSizeMake(maxW, MAXFLOAT) options:NSStringDrawingUsesLineFragmentOrigin attributes:dic context:nil].size;
    
    CGFloat singleLineWidth = [text sizeWithAttributes:dic].width;  //计算单行字符串长度
    CGFloat height = stringSize.height + offset;
    if (height < 28) {
        height = 28;
    }
    if (singleLineWidth > maxW) { //判断是否多行
        stringSize.width = maxW;
        self.isMultiLines = YES;
    } else if (stringSize.width < minWith) {  //判断是否小于最小宽度
        stringSize.width = minWith;
    }

    self.label.frame = CGRectMake(offset, offset*0.5, stringSize.width, height);
    self.contentView.frame = CGRectMake(offset * 0.5, offset * 0.5, stringSize.width + offset, height);
    self.frame = CGRectMake(0, 0, self.label.frame.size.width + offset * 2, self.label.frame.size.height + offset * 2);
    self.label.text = text;
    self.label.font = font;
}

- (void)setItemSelected:(BOOL)itemSelected {
    _itemSelected = itemSelected;
    if (_itemSelected) {
        [self.g setEnabled:YES];
        self.contentView.layer.borderWidth = 0;
        self.contentView.layer.borderColor = self.selectedTitleColro.CGColor;
        self.contentView.backgroundColor = self.selectedBgColor;
        self.label.textColor = self.selectedTitleColro;
    } else {
        [self.g setEnabled:YES];
        self.contentView.layer.borderWidth = 0;
        self.contentView.backgroundColor = self.unSelectedBgColor;
        self.label.textColor = self.unSelectedTitleColor;
    }
}

- (void)setItemDisable:(BOOL)itemDisable {
    _itemDisable = itemDisable;
    if (_itemDisable) {
        [self.g setEnabled:NO];
        self.contentView.layer.borderWidth = 0;
        self.contentView.backgroundColor = self.unSelectedBgColor;
        self.label.textColor = self.disableTitleColor;
    }
}
```

```
#import <UIKit/UIKit.h>
#import "ItemView.h"

NS_ASSUME_NONNULL_BEGIN

@class ItemCollectionView;

/// 视图的协议
@protocol ItemViewDataSource <NSObject>
@required
/// 每一个格子是什么
- (ItemView *)itemCollectionView:(ItemCollectionView *)itemCollectionView cellForRowAtIndexPath:(NSIndexPath *)indexpath;
/// 每一行有多少个格子
- (NSInteger)itemCollectionView:(ItemCollectionView *)itemCollectionView numberOfRowsInSection:(NSInteger)section;
@optional
/// 一共有多少行
- (NSInteger)numberOfSectionsAt:(ItemCollectionView *)itemCollectionView;
/// 设置每一行的头视图
- (UIView *)itemCollectionView:(ItemCollectionView *)itemCollectionView headerInSection:(NSInteger)section;
/// 点击事件
- (void)itemCollectionView:(ItemCollectionView *)itemCollectionView didSelectedIndexPath:(NSIndexPath *)indexpath;
@end

/// 装载所有的小格子
@interface ItemCollectionView : UIScrollView
/// 数据代理
@property (nonatomic, weak) id<ItemViewDataSource> dataSource;
/// 重新创作视图
- (void)createCollectionView;

@end

NS_ASSUME_NONNULL_END
```
```
#import "ItemCollectionView.h"

@interface ItemCollectionView () <ItemViewDelegate>

/// 内容视图
@property (nonatomic, strong) UIView *contentView;
/// 每一行有多少个 默认 0 个
@property (nonatomic, assign) CGFloat rowCountOfSection;
/// 一共有多少行 默认1行
@property (nonatomic, assign) CGFloat sectionCount;
/// 临时数组
@property (nonatomic, strong) NSMutableArray *viewsArray;
/// 内部控件集合
@property (nonatomic, strong) NSArray *itemViewArray;

@end

@implementation ItemCollectionView

- (instancetype)initWithFrame:(CGRect)frame {
    self = [super initWithFrame:frame];
    if (self) {
        [self setUp];
    }
    return self;
}

- (void)awakeFromNib {
    [super awakeFromNib];
    [self setUp];
}

- (void)setUp {
    _rowCountOfSection = 0;
    _sectionCount = 1;
    _viewsArray = [NSMutableArray array];
}

- (void)initView {
    
    if (self.contentView != nil) {
        [self.contentView removeFromSuperview];
    }
    self.contentView = [[UIView alloc] init];
    [self addSubview:self.contentView];
    
    /// 有多少行, 可选代理
    if (self.dataSource && [self.dataSource respondsToSelector:@selector(numberOfSectionsAt:)]) {
        self.sectionCount = [self.dataSource numberOfSectionsAt:self];
    }
    
    /// 每一行有多少个格子,必选
    if (self.dataSource) {
        BOOL a = [self.dataSource respondsToSelector:@selector(itemCollectionView:numberOfRowsInSection:)];
        BOOL b = [self.dataSource respondsToSelector:@selector(itemCollectionView:cellForRowAtIndexPath:)];
        NSAssert((a && b), @"代理对象没有遵守协议!");
    } else {
        NSAssert(NO, @"代理对象为空!");
    }
    
    CGFloat allSectionHeight = 0;
    for (NSInteger i = 0; i < self.sectionCount; i++) {
        UIView *sectionView = [[UIView alloc] init];
        UIView *headView = [self.dataSource itemCollectionView:self headerInSection:i];
        headView.frame = CGRectMake(headView.frame.origin.x, 0, self.bounds.size.width, headView.frame.size.height);
        [sectionView addSubview:headView];
        CGFloat offsetY = headView.frame.size.height;
        UIView *lastrowCell;
        
        self.rowCountOfSection = [self.dataSource itemCollectionView:self numberOfRowsInSection:i];
        
        NSMutableArray *array = [NSMutableArray array];
        
        ItemView *lastCell = nil; //最后一个单元
        
        for (NSInteger k = 0; k < self.rowCountOfSection; k++) {
            NSIndexPath *index = [NSIndexPath indexPathForRow:k inSection:i];
            ItemView *rowCell = [self.dataSource itemCollectionView:self cellForRowAtIndexPath: index];
            rowCell.delegate = self;
            rowCell.indexPath = index;
            
            
            if (lastCell) {  //最后一个单元存在
                
                if (CGRectGetMaxX(lastCell.frame) + rowCell.frame.size.width > self.bounds.size.width) { //新来的这个在后面放不下
                    
                    offsetY += lastCell.frame.size.height;  //放不下就放到下一行
                    
                    rowCell.frame = CGRectMake(0, offsetY, rowCell.frame.size.width, rowCell.frame.size.height);
                    
                } else {
                    
                    if (lastCell.isMultiLines) { //上一个是多行, 当前就另起一行
                        offsetY += lastCell.frame.size.height;
                    }
                    
                    rowCell.frame = CGRectMake(CGRectGetMaxX(lastCell.frame), offsetY, rowCell.frame.size.width, rowCell.frame.size.height);
                }
                
            } else { //最后一个单元不存在
                
                rowCell.frame = CGRectMake(0, offsetY, rowCell.frame.size.width, rowCell.frame.size.height);
            }
            
            lastCell = rowCell;

            
            [sectionView addSubview:rowCell];
            lastrowCell = rowCell;
            [array addObject:rowCell];
        }
        CGFloat sectionHeight = CGRectGetMaxY(lastrowCell.frame) + 5; // +5是为了底部多留点空白
        sectionView.frame = CGRectMake(0, allSectionHeight, self.bounds.size.width, sectionHeight);
        [self.contentView addSubview:sectionView];
        allSectionHeight += sectionHeight;
                
        [self.viewsArray addObject:array];
    }
    self.contentView.frame = CGRectMake(0, 0, self.bounds.size.width, allSectionHeight);
    if (allSectionHeight < self.frameHeight) {
        self.contentSize = CGSizeMake(0, self.frameHeight+1);
    }
    else {
        self.contentSize = self.contentView.frame.size;
    }
    self.itemViewArray = self.viewsArray;
}


#pragma mark - ItemViewDelegate
- (void)didSelected:(ItemView *)itemView {
    
    for (ItemView *v in self.itemViewArray[itemView.indexPath.section]) {
        if (v.itemDisable) { continue; }
        v.itemSelected = NO;
    }
    itemView.itemSelected = YES;
    
    if (_dataSource && [self.dataSource respondsToSelector:@selector(itemCollectionView:didSelectedIndexPath:)]) {
        [self.dataSource itemCollectionView:self didSelectedIndexPath:itemView.indexPath];
    }
}

- (void)createCollectionView {
    [self setNeedsLayout];
    [self layoutIfNeeded];
    [self initView];
}


@end
```
```
#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

@interface ChooseSpecModel : NSObject

/// 名称
@property (strong,nonatomic) NSString *spec_name;
/// id
@property (strong,nonatomic) NSString *spec_id;
/// 详情列表
@property (strong,nonatomic) NSMutableArray *itemArray;

@end

typedef NS_ENUM(NSInteger, ItemType) {  // 0 是默认值, 不能设置为 0
    ItemDisable = 4,       //不可选
    ItemForceSelected = 1, //强制选中
    ItemSelected = 2,      //选中
    ItemUnSelected = 3     //未选中
};

@interface SpecItemModel : NSObject
/// 名称
@property (strong,nonatomic) NSString *spec_item_id;
/// id
@property (strong,nonatomic) NSString *spec_item_name;
/// 自定义属性 标记选项状态
@property (assign,nonatomic) ItemType itemType;

@end
```
```
#import "ChooseSpecModel.h"

@implementation ChooseSpecModel

- (instancetype)init {
    if (self = [super init]) {
        _itemArray = [NSMutableArray array];
    }
    return self;
}

- (void)setValue:(id)value forUndefinedKey:(NSString *)key {
    
}

@end


@implementation SpecItemModel

@end
```
#####创建collectionView，需导入ORSKUDataFiltergit地址[SKUDataFilter](https://github.com/SunriseOYR/SKUDataFilter).
```
/// 商品选择视图
@property (nonatomic, strong) ItemCollectionView *itemCollectionView;
/// 数据
@property (strong, nonatomic) NSMutableArray *dataSource;
//ORSKUDataFilter
@property (nonatomic, strong) ORSKUDataFilter *filter;

- (void)drawSpecView {
    self.itemCollectionView = [[ItemCollectionView alloc] initWithFrame:CGRectMake(20, 100, KScreenWidth-40, 300)];
    self.itemCollectionView.dataSource = self;
    [showView addSubview:self.itemCollectionView];
    _selectedIndexPaths = [NSMutableArray array];
    NSArray *spec_list = _productModel.spec_list;
    self.dataSource = [NSMutableArray array];
    for (NSDictionary *dic in spec_list) {
        ChooseSpecModel *info = [[ChooseSpecModel alloc] init];
        [info setValuesForKeysWithDictionary:dic];
        for (NSDictionary *subDic in dic[@"value_list"]) {
            SpecItemModel *item = [[SpecItemModel alloc] init];
            [item setValuesForKeysWithDictionary:subDic];
            item.itemType = ItemUnSelected;
            [info.itemArray addObject:item];
        }
        [self.dataSource addObject:info];
    }
    _filter = [[ORSKUDataFilter alloc] initWithDataSource:self];

    // 加载视图
    [self.itemCollectionView createCollectionView];
}

#pragma mark -- ORSKUDataFilterDataSource

- (NSInteger)numberOfSectionsForPropertiesInFilter:(ORSKUDataFilter *)filter {
    return self.dataSource.count;
}

- (NSArray *)filter:(ORSKUDataFilter *)filter propertiesInSection:(NSInteger)section {
    ChooseSpecModel *specModel = (ChooseSpecModel *)self.dataSource[section];
    NSMutableArray *temArr = [[NSMutableArray alloc]init];
    [specModel.itemArray enumerateObjectsUsingBlock:^(id  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        SpecItemModel *item = (SpecItemModel *)obj;
        [temArr addObject:item.spec_item_id];
    }];
    return temArr;
}

- (NSInteger)numberOfConditionsInFilter:(ORSKUDataFilter *)filter {
    return _skuData.count;
}

- (NSArray *)filter:(ORSKUDataFilter *)filter conditionForRow:(NSInteger)row {
    NSString *condition = _skuData[row][@"spec_item_id_list"];
    return [condition componentsSeparatedByString:@","];
}

- (id)filter:(ORSKUDataFilter *)filter resultOfConditionForRow:(NSInteger)row {
    NSDictionary *dic = _skuData[row];
    return @{@"price"   : dic[@"price"],
             @"store"   : dic[@"store"],
             @"sku_id"  : dic[@"sku_id"]};
}

#pragma mark - ItemViewDataSource 代理
- (NSInteger)numberOfSectionsAt:(ItemCollectionView *)itemCollectionView {
    return self.dataSource.count;
}

- (NSInteger)itemCollectionView:(ItemCollectionView *)itemCollectionView numberOfRowsInSection:(NSInteger)section {
    ChooseSpecModel *info = self.dataSource[section];
    return info.itemArray.count;
}

- (ItemView *)itemCollectionView:(ItemCollectionView *)itemCollectionView cellForRowAtIndexPath:(NSIndexPath *)indexpath {
    ChooseSpecModel *info = self.dataSource[indexpath.section];
    SpecItemModel *item = info.itemArray[indexpath.row];
    ItemView *v = [[ItemView alloc] init];
    [v setText:item.spec_item_name minWith:70 maxWith:showView.frameWidth-40 font:TitleBFont];
    
    if ([_filter.availableIndexPathsSet containsObject:indexpath]) {
        v.itemSelected = NO;
    }else {
        v.itemDisable = YES;
    }
    
    if ([_filter.selectedIndexPaths containsObject:indexpath]) {
        v.itemSelected = YES;
    }

    return v;
}

- (UIView *)itemCollectionView:(ItemCollectionView *)itemCollectionView headerInSection:(NSInteger)section {
    ChooseSpecModel *info = self.dataSource[section];
    UILabel *label = [[UILabel alloc] initWithFrame:CGRectMake(15, 0, 0, 40)];
    label.text = info.spec_name;
    label.font = TitleBFont;
    label.textColor = TitleBColor;
    return label;
}

- (void)itemCollectionView:(ItemCollectionView *)itemCollectionView didSelectedIndexPath:(NSIndexPath *)indexpath {
    [_filter didSelectedPropertyWithIndexPath:indexpath];
    [_itemCollectionView createCollectionView];

    NSSet *selectedIndexPaths = _filter.selectedIndexPaths;
    // 查看结果
    NSMutableString *tempresult = [NSMutableString string];
    [selectedIndexPaths enumerateObjectsUsingBlock:^(id  _Nonnull obj, BOOL * _Nonnull stop) {
        NSIndexPath *indexPath = (NSIndexPath *)obj;
        ChooseSpecModel *infoR = self.dataSource[indexPath.section];
        SpecItemModel *itemR = infoR.itemArray[indexPath.row];
        if (tempresult.length > 0) {
            [tempresult appendFormat:@","];
        }
        [tempresult appendFormat:@"%@", itemR.spec_item_name];
    }];
    
}
```

##以上为全部实现代码，具体理解如下
```
[indexPath(0,0),indexPath(0,1),indexPath(0,2),
indexPath(1,0),indexPath(1,1),
indexPath(2,0),indexPath(2,1)]
```
![每一个规格所在的indexPath](https://upload-images.jianshu.io/upload_images/3119643-7e64c05fc657c443.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

每一条sku都有按照section的顺序都可以根据indexPath.row生成一个所在位置的数组，例如SKU **A2+1颗/约4.9分+K金** 可以写成[0,0,1]，那么四种组合方式可以得到四个数组
`valid_paths = [[0,0,1],[2,0,1],[1,1,0],[2,1,0]]`

根据有效的sku获取到所有可以点击的item数组
```
NSMutableSet *set = [NSMutableSet set];//set为所有可点击的item的indexPath的集合
[valid_paths enumerateObjectsUsingBlock:^(id *_Nonnull obj, BOOL * _Nonnull stop) {
        [obj enumerateObjectsUsingBlock:^(NSNumber * _Nonnull obj1, NSUInteger idx, BOOL * _Nonnull stop1) {
            [set addObject:[NSIndexPath indexPathForItem:obj inSection:idx]];
        }];
 }];

```
当第一次点击了**K金**所在的item，selectedIndexPath的值为(2,1),遍历valid_paths数组，获取可点击indexPath的数组，再将当前可点击数组与之前所有可点击item数组取交集，就是当前可以点击的按钮。
```
[valid_paths enumerateObjectsUsingBlock:^(id *_Nonnull obj, BOOL * _Nonnull stop) {
       //假设obj为[2,1,0]时，即有效sku为A3+1颗/约0.6分+钻石时，选中的item为该sku中的一项规格。
      
//此时obj中的[2，1，0]转换成indexPath的数组时，应该是paths = [(0,2),(1,1),(2,0)],遍历paths
if(obj[selectedIndexPath.section] == selectedIndexPath.row){
 [paths enumerateObjectsUsingBlock:^(NSindexPath * _Nonnull indexPath, NSUInteger idx, BOOL * _Nonnull stop1) {
            if(selectedIndexPath.section == indexPath.section) {
                //兄弟item都为可选择
             }
            else {
              //如果其他section中indexPath.row == selectedIndexPath.row,则也为可选择
            }
        }];
    }
}
   
}];
```
刷新数据，页面就显示初当前选择的item和其他可选择的数据item，不可选择的则置灰

######已经选中了某个item，点击第二个item时则和上面方法一样，此时会得到一个新的可以选择的item的集合，将第二次的集合与第一次的集合取交集则得到一个新的可以选择的item的集合

##总结
我在一开始陷入了一个死循环，幸亏及时看到了大佬[OrangeAL](https://www.jianshu.com/u/80c622a1fe98)的文章[iOS-SKU商品规格组合算法详解](https://www.jianshu.com/p/295737e2ac77)的文章，git地址[SKUDataFilter](https://github.com/SunriseOYR/SKUDataFilter).
在此感谢！
