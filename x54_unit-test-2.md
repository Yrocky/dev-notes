实际开发中如何使用单元测试

### TTTAttributedLabel

根据TTTAttributedLabel的[测试用例](https://github.com/TTTAttributedLabel/TTTAttributedLabel/blob/master/Example/TTTAttributedLabelTests/TTTAttributedLabelTests.m)来学习单元测试，首先是继承于FBSnapshot子类的测试用例，并且这个测试用例拥有一些属性，其中的Mock是用来模拟实现label代理的。

```objective-c
@interface TTTAttributedLabelTests : FBSnapshotTestCase

@end

@implementation TTTAttributedLabelTests{
    TTTAttributedLabel *label; // system under test
    NSURL *testURL;
    OCMockObject *TTTDelegateMock;
}

- (void)setUp {
    [super setUp];
    
    label = [[TTTAttributedLabel alloc] initWithFrame:CGRectMake(0, 0, 300, 100)];
    label.numberOfLines = 0;
    
    testURL = [NSURL URLWithString:@"http://helios.io"];
    
    TTTDelegateMock = OCMProtocolMock(@protocol(TTTAttributedLabelDelegate));
    label.delegate = (id <TTTAttributedLabelDelegate>)TTTDelegateMock;
}

```

在整个测试用例中，分为了基础功能测试、性能测试、截图测试



### XCTest + [KIF](https://github.com/kif-framework/KIF) + OCMock + FBSnapshotTestCase



### [BDD](https://en.wikipedia.org/wiki/Behavior-driven_development)

