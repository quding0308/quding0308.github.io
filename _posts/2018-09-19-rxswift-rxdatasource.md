---
layout: post
title:  "RxDataSources实现原理"
categories: blog RxSwift
---

* 目录
{:toc}

## 使用

### 数据与tableview绑定

```
// 数据来源，当数据有变化时发送signal
var sectionModelSubject: BehaviorRelay<[SectionModelType]> 

// 封装了配置 tableview 的逻辑，定义了怎么使用数据
var datasource: RxTableViewSectionedReloadDataSource<SectionModelType> 

// 把tableview、数据、datasource组合在一起
sectionModelSubject
	.bind(to: tableView.rx.items(dataSource: dataSource))
	.disposed(by: bag)
```

### UITableView初始化配置

#### 使用 RxTableViewDataSourceType 配置

``` Swift
// 必须
numberOfSections
numberOfRowsInSection
cellForRowAtIndexPath

height of cell default 44

// closure 根据参数indexpath，cellModel(最后一个参数I = S.Item)，返回tableview cell
let dataSource = 
RxTableViewSectionedReloadDataSource<SettingSectionModel>(configureCell: { (dataSource, tableView, indexPath, rowModel) -> UITableViewCell in
	let cell = SettingTableCell(style: .default, reuseIdentifier: "SettingTableCell")
	cell.setModel(rowModel)
	
	return cell
})

// closure 返回 section header title
dataSource.titleForHeaderInSection = { dataSource, section in
	return "title \(section)"
}

// closure 返回 section footer title
dataSource.titleForFooterInSection = { dataSource, section in
	return "footer \(section)"
}

dataSource.canEditRowAtIndexPath = { dataSource, indexPath in
	return true
}

dataSource.canMoveRowAtIndexPath = { dataSource, indexPath in
	return true
}

// section index 
dataSource.sectionIndexTitles = { dataSource in
	return ["1", "2", "3"]
}
dataSource.sectionForSectionIndexTitle = { dataSource, title, index in
	return index
}

```

### 事件处理
#### 使用 rx 处理事件
```
tableView.rx.itemSelected
	.subscribe(onNext: { indexPath in
		//
	})
	.disposed(by: bag)

tableView.rx.willDisplayCell
	.subscribe(onNext: { [weak self] (cell, indexPath) in
		guard let `self` = self else { return }
		
		print("==== will display")
	})
	.disposed(by: bag)

tableView.rx.didEndDragging
	.subscribe(onNext: { [weak self] willDecelerate in
		guard let `self` = self else { return }
		//
	})
	.disposed(by: bag)

```

#### 使用 UITableViewDelegate 处理事件
```
// 设置 delegate 
// tableView.delegate = self
tableView.rx
	.setDelegate(self)
	.disposed(by: bag)
	
// 正常使用
extension SettingVC: UITableViewDelegate {
    func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
        return 60
    }
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        print("delegate selected \(indexPath.row)")
    }
}

```

#### 同时使用 rx 和 UITableViewDelegate

**注意** 如果同时使用 delegate 和 rx处理同一个事件，当触发事件后，两者都会执行，优先执行delegate的代码。

例如：
``` Swift
tableView.rx.itemSelected
	.subscribe(onNext: { [weak self] indexPath in
		print("select 后执行")
	})
	.disposed(by: bag)

tableView.rx
	.setDelegate(self)
	.disposed(by: bag)

func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
	print("select 先执行")
}
```

### RxDataSource提供的两类DataSource

#### RxTableViewSectionedReloadDataSource

每次有数据更新，会调用 tableView.reloadData()

核心代码：
```
 open func tableView(_ tableView: UITableView, observedEvent: Event<Element>) {
        Binder(self) { dataSource, element in
            dataSource.setSections(element)
            tableView.reloadData()
        }.on(observedEvent)
    }
```

#### RxTableViewSectionedAnimatedDataSource

通过Diff算法会对前后数据进行比较，最后 得出 insert、delete、move、reload 的数据，再刷新。刷新时可以指定动画。

核心代码：
```
let oldSections = dataSource.sectionModels

let differences = try Diff.differencesForSectionedView(initialSections: oldSections, finalSections: newSections)

for difference in differences {
	dataSource.setSections(difference.finalSections)
	tableView.performBatchUpdates(difference, animationConfiguration: self.animationConfiguration)
}
```


**具体使用**

DataSource 初始化
```
// 创建
let dataSource = RxTableViewSectionedAnimatedDataSource<AboutSectionAnimationModel>(configureCell: { (dataSource, tableView, indexPath, rowModel) -> UITableViewCell in
	let cell = AboutTableCell(style: .default, reuseIdentifier: "AboutTableCell")
	cell.setModel(rowModel)
	return cell
})

// 动画样式
let animationConfig = AnimationConfiguration(insertAnimation: .fade, reloadAnimation: .left, deleteAnimation: .none)
dataSource.animationConfiguration = animationConfig

// animate or reload    reload 会直接刷新
dataSource.decideViewTransition = { dataSource, tableView, changeSet in
//            return ViewTransition.animated
	return ViewTransition.reload    // tableview reload
}
```

SectionModel：
```
struct AboutSectionAnimationModel {
    var items: [Item]
}

extension AboutSectionAnimationModel: AnimatableSectionModelType {
    typealias Item = AboutRowAnimationModel
    init(original: AboutSectionAnimationModel, items: [Item]) {
        self = original
        self.items = items
    }
}

// SectionModel 需要实现 IdentifiableType 协议
extension AboutSectionAnimationModel: IdentifiableType {
    typealias Identity = String

    var identity: String {
        return ""
    }
}
```

RowModel：
```
// 需要实现 IdentifiableType 与 Equatable 在 Diff 算法中 需要
struct AboutRowAnimationModel {
    var text: String
    var detailText: String
}

extension AboutRowAnimationModel: IdentifiableType {
    typealias Identity = String

    var identity: String {
        return self.text
    }
}

extension AboutRowAnimationModel: Equatable {
    static func == (lhs: AboutRowAnimationModel, rhs: AboutRowAnimationModel) -> Bool {
        return lhs.text == rhs.text
    }
}
```

## 其他
### SectionModelType

SectionModelType协议是数据相关的核心类

```
public protocol SectionModelType {
	associatedtype Item
	var items: [Item] { get }
	init(original: Self, items: [Item])
}
```
- 一个SectionModelType对象 对应一个Section的数据，其中 一个Item对象 对应一个tableview cell对象。
- 每个tableview对应一个实现了SectionModelType类 和 对应cell的 Item类，这两个类用于封装数据。
- 写一个Array<SectionModelType>，array的数量就是tableview的section的count，而 SectionModelType中items的count就是每个section中cell的count

viewModel 是数据来源，封装了 SectionModelType 数组。一个 SectionModelType 对应一个 section， SectionModelType 中的 items 对应的是 section 中cell 的数量。

### IdentifiableType

在 RxDataSources 中 提供了一些String、各种Int、Float、Double的默认实现

### 设置 delegate 两种方式的区别

```
tableView.delegate = self

delegate 放在 bind 代码之前 和 之后 有区别

tableView.rx
	.setDelegate(self)
	.disposed(by: disposeBag)
	
```

### 同一事件 为什么先执行delegate，后执行rx代码？ 
具体原理在_RXDelegateProxy.m中定义
```
-(void)forwardInvocation:(NSInvocation *)anInvocation {
	BOOL isVoid = RX_is_method_signature_void(anInvocation.methodSignature);
	NSArray *arguments = nil;
	if (isVoid) {
		arguments = RX_extract_arguments(anInvocation);
		[self _sentMessage:anInvocation.selector withArguments:arguments];
	}
	
	// 先执行delegate
	if (self._forwardToDelegate && [self._forwardToDelegate respondsToSelector:anInvocation.selector]) {
		[anInvocation invokeWithTarget:self._forwardToDelegate];
	}

	// 后执行 rx
	if (isVoid) {
		[self _methodInvoked:anInvocation.selector withArguments:arguments];
	}
}
```

### DataSource 可以 通过 代理 和 Rx同时设置吗？

不可以，只能二选一。

会有assert宏 报错。

## 参考
- 
代码：https://github.com/ReactiveX/RxSwift/blob/master/RxCocoa/iOS/UITableView%2BRx.swift