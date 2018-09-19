---
layout: post
title:  "RxSwift-RxDataSources实现原理"
categories: blog, RxSwift
---

### 具体实现

#### SectionModelType协议

SectionModelType协议是数据相关的核心类

	public protocol SectionModelType {
	    associatedtype Item
	    var items: [Item] { get }
	    init(original: Self, items: [Item])
	}

- 一个SectionModelType对象 对应一个Section的数据，其中 一个Item对象 对应一个tableview cell对象。
- 每个tableview对应一个实现了SectionModelType类 和 对应cell的 Item类，这两个类用于封装数据。
- 写一个Array<SectionModelType>，array的数量就是tableview的section的count，而 SectionModelType中items的count就是每个section中cell的count



#### TableViewSectionedDataSource是核心类，封装了几个closure用于初始化tableview

TableViewSectionedDataSource是核心类，封装了几个closure用于初始化tableview

	// closure 根据参数indexpath，cellModel(最后一个参数I = S.Item)，返回tableview cell
	configureCell: ConfigureCell
	typealias ConfigureCell = (TableViewSectionedDataSource<S>, UITableView, IndexPath, I) -> UITableViewCell

	// closure 返回 section header title
	titleForHeaderInSection:TitleForHeaderInSection
	typealias TitleForHeaderInSection = (TableViewSectionedDataSource<S>, Int) -> String?

	// closure 返回 section footer title
	titleForFooterInSection: TitleForFooterInSection
	typealias TitleForFooterInSection = (TableViewSectionedDataSource<S>, Int) -> String?

	// 是否可编辑
	canEditRowAtIndexPath: CanEditRowAtIndexPath
	typealias CanEditRowAtIndexPath = (TableViewSectionedDataSource<S>, IndexPath) -> Bool

	canMoveRowAtIndexPath: CanMoveRowAtIndexPath
	typealias CanMoveRowAtIndexPath = (TableViewSectionedDataSource<S>, IndexPath) -> Bool

	// tableview 索引
	sectionIndexTitles: SectionIndexTitles
	typealias SectionIndexTitles = (TableViewSectionedDataSource<S>) -> [String]?

	sectionForSectionIndexTitle: SectionForSectionIndexTitle
	typealias SectionForSectionIndexTitle = (TableViewSectionedDataSource<S>, _ title: String, _ index: Int) -> Int


#### 数据与tableview绑定

我们定义一个Array<SectionModelType>封装成Observable，当数据有变化时发送signal。TableViewSectionedDataSource内定义好了怎么用数据，收到signal后怎么处理对应的cell。

	/* 
	tableview定义了在哪里展示数据
	datasource用于封装怎么使用数据
	写代码把tableview、数据、datasource组合在一起
	*/
	let sections: Array<SectionModelType>
	sections.asObservable()
            .bind(to: tableView.rx.items(dataSource: dataSource))
            .disposed(by: disposeBag)
			

#### UITableView的Rx
	

##### 事件处理
	
    tableView.rx.itemSelected
        .subscribe(onNext: { [weak self] indexPath in
            ///
        })
        .disposed(by: disposeBag)
    
    tableView.rx.willDisplayCell
        .subscribe(onNext: { [weak self] (cell, indexPath) in
            guard let `self` = self else { return }
            
            print("==== will display")
        })
        .disposed(by: disposeBag)

    tableView.rx.didEndDragging
        .subscribe(onNext: { [weak self] willDecelerate in
            guard let `self` = self else { return }
            
        })
                .disposed(by: bag)

 事件不支持，可以设置delegate来处理

    tableView
        .rx
        .setDelegate(self)
        .disposed(by: disposeBag)

**注意** 如果同时设置了 delegate 又设置了对应事件的rx处理 当触发事件后，两者都会执行，优先执行delegate的代码。例如：
	
    tableView.rx.itemSelected
        .subscribe(onNext: { [weak self] indexPath in
            print("select 后执行")
        })
        .disposed(by: disposeBag)
    
    tableView
        .rx
        .setDelegate(self)
        .disposed(by: disposeBag)

    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        print("select 先执行")
    }

为什么先执行delegate，后执行rx代码？ 具体原理在_RXDelegateProxy.m中定义
		
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



