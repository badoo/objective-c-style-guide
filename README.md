# THIS REPO IS DEPRECATED, PLEASE USE [IOS-GUIDELINES](https://github.com/badoo/ios-guidelines)

# Badoo Objective-C Style Guide

This style guide outlines the coding conventions used at [Badoo](http://badoo.com). This style guide is based on the [NYTimes Objective-C Style Guide](https://github.com/NYTimes/objetive-c-style-guide).

## Introduction

Here are some of the documents from Apple that informed the style guide. If something isn't mentioned here, it's probably covered in great detail in one of these:

* [The Objective-C Programming Language](http://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/ObjectiveC/Introduction/introObjectiveC.html)
* [Cocoa Fundamentals Guide](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CocoaFundamentals/Introduction/Introduction.html)
* [Coding Guidelines for Cocoa](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html)
* [iOS App Programming Guide](http://developer.apple.com/library/ios/#documentation/iphone/conceptual/iphoneosprogrammingguide/Introduction/Introduction.html)

## Table of Contents

* [Nullability](#nullability)
* [Dot-Notation Syntax](#dot-notation-syntax)
* [Code width](#code-width)
* [Spacing](#spacing)
* [Conditionals](#conditionals)
* [Ternary Operator](#ternary-operator)
* [Error handling](#error-handling)
* [Methods](#methods)
* [Enumerations](#enumerations)
* [Properties](#properties)
* [Naming](#naming)
* [Comments](#comments)
* [Protocols](#protocols)
* [Init & Dealloc](#init-and-dealloc)
* [instancetype vs id](#instancetype-vs-id)
* [alloc-init vs new](#alloc-init-vs-new)
* [Literals](#literals)
* [CGRect Functions](#cgrect-functions)
* [Constants](#constants)
* [Enumerated Types](#enumerated-types)
* [Private Properties](#private-properties)
* [Private Headers](#private-headers)
* [Images](#images)
* [Booleans](#booleans)
* [Blocks](#blocks)
* [Type inference](#type-inference)
* [Singletons](#singletons)
* [Unit Tests](#unit-tests)
* [Xcode Project](#xcode-project)

## Nullability
Since Xcode 6.3, Apple has [introduced nullability annotations](https://developer.apple.com/swift/blog/?id=25) to the Objective-C compiler. This allows for a more expressive API and it is specially important for Objective-C code 'seen' from Swift.

We want to enforce usage of this annotations for all code, be it application level or iOS Badoo platform level.

The adoption of annotations is straightforward. The standard we adopt is using the `NS_ASSUME_NONNULL_BEGIN` and `NS_ASSUME_NONNULL_END` on all headers. Then indicate nullability for the pointers that can be so.

Let's review how you can use it in an actual example:
```objc
// MyClass.h

NS_ASSUME_NONNULL_BEGIN

@interface MyClass : NSObject
@property (nonatomic, copy) NSString *myString; //Assumed non-nil
@property (nonatomic, assign) BOOL myBoolean;
@property (nonatomic, copy, readonly, nullable) NSString *myOtherString;

// Weak properties must be nullable. Compiler will not complain if never set to nil by
// the programmer. If not annotated as nullable, property still set to nil by runtime
@property (nonatomic, weak, nullable) id delegate; though.

- (instancetype)initWithString:(NSString *)string otherString:(nullable NSString *)otherString;

@end

NS_ASSUME_NONNULL_END

// MyClass.m

NS_ASSUME_NONNULL_BEGIN

// Don't forget to annotate class extensions and categories
@interface MyClass()
//
@end

NS_ASSUME_NONNULL_END

@implementation MyClass
//
@end
```

There is not much to these annotations but you need to be aware of:
- Annotations don't change the generated code.
- Weak properties: They are nullified by the runtime. If not annotated with nil, API does not express intent fully. Only generates a warning when explicitly set to nil - [example](https://gist.github.com/DarthMike/1add91a7f5b5bf18c326)
- There is mostly no sense using nullability annotations outside of interface declarations.

###Gotcha
Nullability is just an annotation for the compiler. This means that runtime code does not change and there are times that annotation can receive (or return) differently as annotated. Here is an example:

```objc
// Your object

NS_ASSUME_NONNULL_BEGIN

@interface ViewController : UIViewController
@property (nonatomic, copy) NSString *nonNullString;
@end

NS_ASSUME_NONNULL_END

// Then you use it as:
NSString *aString = [NSString stringWithFormat:@"helloworld %.1f",1.0];
aString = nil;
controller.nonNullString = aString; // No warning. Generated code does not change.
controller.nonNullString = nil; // Warning
```

So Objective-C is as safe as is has always been. Annotations express intention but don't change the generated code. Checks for nil are still required in the implementation.

## Dot-Notation Syntax

Dot-notation should **always** be used for accessing and mutating properties. Bracket notation is preferred in all other instances.

**For example:**
```objc
view.backgroundColor = [UIColor orangeColor];
[UIApplication sharedApplication].delegate;
```

**Not:**
```objc
[view setBackgroundColor:[UIColor orangeColor]];
UIApplication.sharedApplication.delegate;
```

The use of dot notation to access other methods is not allowed, even if Objective-C as a language would allow it. Dot notation is just syntactic sugar for a method so it would work.

**Example. Not allowed:**
```objc
UIApplication *application = UIApplication.sharedApplication
```
Bracket notation is also to be used for [getter methods](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/EncapsulatingData/EncapsulatingData.html#//apple_ref/doc/uid/TP40011210-CH5-SW2):
```objc
@property (getter=isFinished) BOOL finished;
if ([operation isFinished]) // Correct
if (operation.finished) // Correct
if (operation.isFinished) // Wrong
```

## Code width
We don't impose a line width limit. We use modern and big screens, even when connected to laptops. Also Xcode offers autowrapping.

Even though we don't want to limit the code width, be aware that long method calls and declarations, because Objective-C is really verbose, should be separated by parameters with new lines. Use your judgement for this.


## Code Organisation

Use `#pragma mark -` to categorize methods in functional groupings and protocol/delegate implementations following this general structure.
Strive for short and descriptive names, and try not to leave them blank (like `#pragma mark - `).

```objc
#pragma mark - Lifecycle

- (instancetype)init {}
- (void)dealloc {}
- (void)viewDidLoad {}
- (void)viewWillAppear:(BOOL)animated {}
- (void)didReceiveMemoryWarning {}

#pragma mark - Custom Accessors

- (void)setCustomProperty:(id)value {}
- (id)customProperty {}

#pragma mark - IBActions

- (IBAction)submitData:(id)sender {}

#pragma mark - Public

- (void)publicMethod {}

#pragma mark - Private

- (void)privateMethod {}

#pragma mark - Protocol conformance
#pragma mark - UITextFieldDelegate
#pragma mark - UITableViewDataSource
#pragma mark - UITableViewDelegate

#pragma mark - NSCopying

- (id)copyWithZone:(NSZone *)zone {}

#pragma mark - NSObject

- (NSString *)description {}
```

## Spacing

* Indent using 4 spaces. Never indent with tabs. Be sure to set this preference in Xcode.

* There should be exactly one blank line between methods to aid in visual clarity and organisation. No multiple methods per line.

* The star for pointer types should be adjacent to the variable name, not the type. Applies for all uses (properties, local variables, constants, method types, ...):
**Example:**
```objc
NSString *message = NSLocalizedString(@"bma.intro.message", nil);
```
**Not:**
```objc
NSString* message = NSLocalizedString(@"bma.intro.message", nil);
```
## Brackets

Egyptian brackets are to be used for both method implementations and conditions (`if`/`else`/`switch`/`while` etc.):

**Like this:**
```objc
if ([user isHappy]) {
    // Do something
} else {
    // Do something else
}

- (void)myMethod {
    // Do something
}
```

**Not:**
```objc
if ([user isHappy]) {
    //Do something
}
else {
    //Do something else
}

- (void)myMethod
{
    //Do something
}
```

## Conditionals

Conditional bodies **should always** use braces even when a conditional body could be written without braces (e.g., it is one line only) to prevent [errors](https://www.imperialviolet.org/2014/02/22/applebug.html). These errors include adding a second line and expecting it to be part of the if-statement. Another, [even more dangerous defect](http://programmers.stackexchange.com/a/16530) may happen where the line "inside" the if-statement is commented out, and the next line unwittingly becomes part of the if-statement. In addition, this style is more consistent with all other conditionals, and therefore more easily scannable.

**For example:**
```objc
if (!error) {
    return success;
}
```

**Not:**
```objc
if (!error)
    return success;
```

or

```objc
if (!error) return success;
```

See [Apple SSL bug](https://www.imperialviolet.org/2014/02/22/applebug.html) if you're not sure about this recommendation.

Always avoid Yoda conditions.

**For example:**
```objc
if ([myValue isEqual:constant]) { ...
```

**Not:**
```objc
if ([constant isEqual:myValue]) { ...
```

Prefer extracting several properties to a meaningful expression, improving readability

***Example:***
```objc
BOOL stateForDismissalIsCorrect = [object something] && [object somethingElse] && ithinkSo;
if (stateForDismissalIsCorrect) {
}
```

***Refactor these:***
```objc
if ([object something] && [object somethingElse] && ithinkSo) {
}
```

Do not check for presence of delegate when checking optional methods:

**For example:**
```objc
if ([self.delegate respondsToSelector:@selector(...)]) { ...
```

**Not:**
```objc
if (self.delegate && [self.delegate respondsToSelector:@selector(...)]) { ...
```

Use the 'Golden Path' rule as in [ZDS code style](http://www.cimgf.com/zds-code-style-guide/).


## Ternary Operator

The Ternary operator, ? , should only be used when it increases clarity or code neatness. A single condition is usually all that should be evaluated. Evaluating multiple conditions is usually more understandable as an if statement, or refactored into instance variables.

**For example:**
```objc
result = a > b ? x : y;
string = fromServer ?: @"hardcoded";
```

**Not:**
```objc
result = a > b ? x = c > d ? c : d : y;
```

## Error handling

When methods return an error parameter by reference, switch on the returned value, not the error variable.

**For example:**
```objc
NSError *error;
if (![self trySomethingWithError:&error]) {
    // Handle Error
}
```

**Not:**
```objc
NSError *error;
[self trySomethingWithError:&error];
if (error) {
    // Handle Error
}
```

Some of Apple’s APIs write garbage values to the error parameter (if non-NULL) in successful cases, so switching on the error can cause false negatives (and subsequently crash).

## Methods

In method signatures, there should be a space after the scope (-/+ symbol). There should be a space between the method segments.

**For example:**
```objc
- (void)setExampleText:(NSString *)text image:(UIImage *)image;
```

**Not:**
```objc
-(void)setExampleText: (NSString *)text image: (UIImage *)image;
```

There are no special requirements for private methods. They can be named as normal methods, but do not use underscore prefix as it is [reserved for use by Apple](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CodingGuidelines/Articles/NamingMethods.html#//apple_ref/doc/uid/20001282-1003829-BCIBDJCA).

In class implementations, there should be one line between every methods, and one line before and after @implementation. Pragma marks should leave a line before and after.
***Example:***
```objc

@interface BMAPersonViewController ()
@property (nonatomic, weak) UIButton *settingsButton;
@end

@implementation BMAPersonViewController

#pragma mark - LifeCycle

- (void)viewDidLoad {
    // Really small implementation
}

- (void)viewDidAppear:(BOOL)animated {
    // Really small implementation
}

#pragma mark - Settings

- (IBAction)goToSettings:(id)sender {
    // Really small implementation
}

@end

```
- Method parameters should have no prefix. Use normal names.


## Enumerations
- We use modern Objective-C style, and enumerations should be declared using `NS_ENUM` macro.
- Also, when creating names, make the names autocomplete-friendly, not like they would be in English:

***Example:***
```objc
typedef NS_ENUM(BMACollectionViewLayoutMode, NSUInteger) {
     BMACollectionViewLayoutModeGrid,
     BMACollectionViewLayoutModeFullscreen
}
```

***Not:***
```objc
typedef NS_ENUM(BMACollectionViewLayoutMode, NSUInteger) {
     BMACollectionViewLayoutGridMode,
     BMACollectionViewLayoutFullscreenMode
}
```

## Properties

Prefer to use properties instead of methods when dealing with *state*, but not *behaviour*.

**For example:**
``` objc
@interface BMAProfileController : UIViewController

@property (nonatomic) BMAProfile *profile;

@end
```

**Not:**
``` objc
@interface BMAProfileController : UIViewController

- (BMAProfile *)profile;
- (void)setProfile:(BMAProfile *)profile;

@end
```

For sure, `init`, `dealloc` and accessors are to be methods. 

- Format for property declaration should have space after @property:
**Example:**
```objc
@interface BMAPerson
@property (nonatomic, copy, readonly) NSString *identifier;
@end
```
**Not:**
```objc
@interface BMAPerson
@property(nonatomic,copy,readonly) NSString* identifier;
@end
```

- `@synthesize` and `@dynamic` should each be declared on new lines in the implementation.

- Do not be too verbose with attributes: no necessity to specify attributes used by default, but be specific when it's needed, and don't forget `atomic` is used by default

**Example:**
```objc
@interface BMAPerson
@property (nonatomic) NSUInteger numberOfCommonPlaces;
@property (nonatomic) NSArray *commonPlaces;
@property (nonatomic, weak) id<BMAPersonDelegate> delegate;
@end
```
**Not:**
```objc
@interface BMAPerson
@property (nonatomic, assign) NSUInteger numberOfCommonPlaces; // assign is used by default for integers
@property (strong) NSArray *commonPlaces; // shouldn't be atomic unless it's really needed, and strong is used by default for instance types
@property (nonatomic) id<BMAPersonDelegate> delegate; // delegates are not owned
@end
```

- An atomic property should be marked as such, not left to the default value, which is `atomic`. This increases readability and awareness to other developers on the nature of this property.

- Prefer `atomic`/`nonatomic` to be first in the attribute list: keep consistency around the codebase.

- Prefer using properties for all ivar access. There are many good reasons to do it, as stated [here for example](http://blog.bignerdranch.com/4005-should-i-use-a-property-or-an-instance-variable/). Direct ivar access should be justified. Refactor recklessly legacy code which still uses instance variables.

- The only time when ivars should be used is `dealloc` and `init` methods, and implementation of properties based on them. This is because in `init` and `dealloc` it's generally best practice to avoid side effects of setting properties directly, and because inside `init`, the object is still in a partial state.

```objc
@implementation BMAProfile

- (void)dealloc {
    [_updateTimer invalidate];
}

- (BMATimer *)updateTimer {
    if (!_updateTimer) {
        _updateTimer = [[BMATimer alloc] init];
        // ...
    }
    return _updateTimer;
}

@end

```

>For more information on using Accessor Methods in Initialiser Methods and dealloc, see [here](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/mmPractical.html#//apple_ref/doc/uid/TP40004447-SW6).


## Naming

Apple naming conventions should be adhered to wherever possible, especially those related to [memory management rules](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/MemoryMgmt/Articles/MemoryMgmt.html) ([NARC](http://stackoverflow.com/a/2865194/340508)).

Variables should be named as descriptively as possible. Single letter variable names should be avoided except in `for()` loops. Long, descriptive method and variable names are good.

**For example:**

```objc
UIButton *settingsButton;
```

**Not:**

```objc
UIButton *setBut;
```

A three letter prefix should always be used for class names, categories (especially for categories on Cocoa classes) and constants. Constants should be camel-case with all words capitalised and prefixed by the related class name for clarity. That prefix depends on where the code lays, refer to architecture or tech lead to know which to use (`BPF`, `BPUI`, `BMA`, `HON`, etc)

**For example:**

```objc
static const NSTimeInterval BMAProfileViewControllerNavigationFadeAnimationDuration = 0.4;

@interface NSAttributedString (BMAHTMLParsing)

- (void)bma_attributedStringFromHTML:(NSString *)string;

@end
```

**Not:**

```objc
static const NSTimeInterval fadetime = 0.2;

@interface NSAttributedString (HTMLParsing)

- (void)attributedStringFromHTML:(NSString *)string;

@end
```

Properties and local variables should be camel-case with the leading word being lowercase.

Instance variables should be camel-case with the leading word being lowercase, and should be prefixed with an underscore. This is consistent with instance variables synthesised automatically by LLVM. **If LLVM can synthesise the variable automatically, then let it.**

**For example:**

```objc
// Let compiler to generate those, but if you use them, then write as:
@synthesize descriptiveVariableName = _descriptiveVariableName;
```

**Not:**

```objc
id varnm;
```

Delegate methods should be always have the caller as first parameter

**For example:**

```objc
- (void)lessonController:(LessonController *)lessonController didSelectLesson:(Lesson *)lesson;
- (void)lessonControllerDidFinish:(LessonController *)lessonController;
```

**Not:**

```objc
- (void)lessonControllerDidSelectLesson:(Lesson *)lesson;
```

### Underscores

When using properties, instance variables should always be accessed and mutated using `self.`. This means that all properties will be visually distinct, as they will all be prefaced with `self.`. Local variables should not contain underscores.

## Comments

When they are needed, comments should be used to explain **why** a particular piece of code does something. Any comments that are used must be kept up-to-date or deleted.
When comment is inserted, not allowed:
- Name of person writing the comment: version control will say already
- JIRA ticket references
- No code must be commented. Remove it as it will be tracked in version control. Tag the repo if you think it will be needed in future (rarely the case)

Block comments should generally be avoided, as code should be as self-documenting as possible, with only the need for intermittent, few-line explanations. This does not apply to those comments used to generate documentation. Having a lots of comments in a single method means that the developer has to excuse himself by not writing clear code. Prefer many small methods with long verbose names.

Prefer to document interesting pieces of API instead, specially application platform APIs, or reusable code.

Also attributes like `NS_REQUIRES_SUPER`, `NS_DESIGNATED_INITIALIZER`, `DEPRECATED_MSG_ATTRIBUTE`, and `DEPRECATED_ATTRIBUTE` to document API and ongoing work when refactoring.

## Protocols
Protocol format should be as follows:
```objc
@protocol BMAPerson <NSObject>
// Method rules
@end

@interface BMAMutualAttraction : NSObject <BMAPerson>
@property (nonatomic) id<BMAPerson> me;
@end
```

Prefer to use [properties](#properties), not [methods](#methods) in protocols.

## Header Documentation

The documentation of class should be done using the Doxygen/AppleDoc syntax only in the .h files when possible. Documentation should be provided for methods and properties.

**For example:**

```objc
/**
 *  Designated initializer.
 *
 *  @param  repository  The repository for CRUD operations.
 *  @param  searchService The search service used to query the repository.
 *
 *  @return A BMAScheduledOperationsProcessor object.
 */
- (instancetype)initWithScheduledOperationsRepository:(id<BMAGenericUGCRepositoryProtocol>)repository
                     scheduledOperationsSearchService:(id<BMAGenericSearchServiceProtocol>)searchService;

```

## init and dealloc

`init` methods should be placed at the top of the implementation, directly after the `@synthesize` and `@dynamic` statements. `dealloc` should be placed directly below the `init` methods of any class.

`init` methods should be structured like this:

```objc
- (instancetype)init {
    self = [super init]; // or call the designated initalizer
    if (self) {
        // Custom initialization
    }

    return self;
}

- (void)dealloc {
    // Necessary cleanup here
}

```

All classes should use `NS_DESIGNATED_INITIALIZER` for any declared init methods, even if there is only one. It documents the code in a proper way.

##instancetype vs id
- Read [this](http://nshipster.com/instancetype/) and [this](https://developer.apple.com/library/ios/releasenotes/ObjectiveC/ModernizationObjC/AdoptingModernObjective-C/AdoptingModernObjective-C.html#//apple_ref/doc/uid/TP40014150) if you don't know what instancetype is
- For init methods, we should use modern Objective-C conventions, so ***use instancetype always for init methods***.
- For factory methods, there are two cases. The two cases better document code by following conventions:
    - When the factory method can be subclassed: ***use instancetype***
    - When the factory method is not meant to be subclassed: ***use the type explicitly***

##alloc-init vs new

Do not use `-new`, but chained `-alloc` and `-init`:

```objc
BMAPerson *person = [[BMAPerson alloc] init]; // Correct
BMAPerson *person2 = [BMAPerson new]; // Wrong
```

## Literals

`NSString`, `NSDictionary`, `NSArray`, and `NSNumber` literals should be used whenever creating immutable instances of those objects. Pay special care that `nil` values not be passed into `NSArray` and `NSDictionary` literals, as this will cause a crash.

**For example:**

```objc
NSArray *names = @[@"Brian", @"Matt", @"Chris", @"Alex", @"Steve", @"Paul"];
NSDictionary *productManagers = @{@"iPhone" : @"Kate", @"iPad" : @"Kamal", @"Mobile Web" : @"Bill"};
NSNumber *shouldUseLiterals = @YES;
NSNumber *buildingZIPCode = @10018;
```

**Not:**

```objc
NSArray *names = [NSArray arrayWithObjects:@"Brian", @"Matt", @"Chris", @"Alex", @"Steve", @"Paul", nil];
NSDictionary *productManagers = [NSDictionary dictionaryWithObjectsAndKeys: @"Kate", @"iPhone", @"Kamal", @"iPad", @"Bill", @"Mobile Web", nil];
NSNumber *shouldUseLiterals = [NSNumber numberWithBool:YES];
NSNumber *buildingZIPCode = [NSNumber numberWithInteger:10018];
```

## CGRect Functions

When accessing the `x`, `y`, `width`, or `height` of a `CGRect`, always use the [`CGGeometry` functions](http://developer.apple.com/library/ios/#documentation/graphicsimaging/reference/CGGeometry/Reference/reference.html) instead of direct struct member access. From Apple's `CGGeometry` reference:

> All functions described in this reference that take CGRect data structures as inputs implicitly standardise those rectangles before calculating their results. For this reason, your applications should avoid directly reading and writing the data stored in the CGRect data structure. Instead, use the functions described here to manipulate rectangles and to retrieve their characteristics.

**For example:**

```objc
CGRect frame = self.view.frame;

CGFloat x = CGRectGetMinX(frame);
CGFloat y = CGRectGetMinY(frame);
CGFloat width = CGRectGetWidth(frame);
CGFloat height = CGRectGetHeight(frame);
```

**Not:**

```objc
CGRect frame = self.view.frame;

CGFloat x = frame.origin.x;
CGFloat y = frame.origin.y;
CGFloat width = frame.size.width;
CGFloat height = frame.size.height;
```

You can alternatively use the categories on UIView we have in our platform, which allow for a Three20 or autolayout style to get frame properties:

```objc
self.button.bma_bottom = self.header.bma_top;
self.button.bma_width = self.bma_width;
```
## Constants

Constants are preferred over in-line string literals or numbers, as they allow for easy reproduction of commonly used variables and can be quickly changed without the need for find and replace. Prefer class or instance methods for constants. The reason is that those constants can 'change' specially important for UI code where different styling can be applied at runtime.

If you declare constants, they should be declared as `static` constants and not `#define`s unless explicitly being used as a macro.

**For example:**

```objc
// Preferred
+ (CGFloat)thumbnailHeight {
    return 50.0;
}

+ (NSString *)nibName {
    return @"BMADefaultProfileViewController";
}

// Use those sparingly
static NSString * const BMADefaultProfileViewControllerNibName = @"nibName";
static const CGFloat BMAImageThumbnailHeight = 50.0;
```

**Not:**

```objc
#define BMADefaultProfileViewControllerNibName @"nibName"

#define BMAImageThumbnailHeight 50.0
```

## Enumerated Types

When using `enum`s, it is recommended to use the new fixed underlying type specification because it has stronger type checking and code completion. The SDK now includes a macro to facilitate and encourage use of fixed underlying types — `NS_ENUM()`

**Example:**

```objc
typedef NS_ENUM(NSInteger, BMAProfilePictureState) {
    BMAProfilePictureStateInactive,
    BMAProfilePictureStateLoading
};
```

## Private Properties

Private properties should be declared in class extensions (anonymous categories) in the implementation file of a class. Named categories (e.g. `BMAPrivate`) should never be used unless extending another class.

**For example:**

```objc
@interface BMAAdvertisement ()

@property (nonatomic) GADBannerView *googleAdView;
@property (nonatomic) ADBannerView *iAdView;
@property (nonatomic) UIWebView *adXWebView;

@end
```

## Private headers

Use private headers to define class extension and any internal methods and properties which can be used for [unit tests](#unit-tests).

Private header has to contain imports of headers for types declared in main header using `forward declaration` as well as types used in class extension, but not imports of headers for types used in implementation file.

**Example:**
```objc
// Foo.h

@class Bar;
@protocol Baz;

@interface Foo : NSObject

@property (nonatomic, readonly) Bar *bar;
@property (nonatomic, weak) id<Baz> baz;

@end

// Foo_Private.h

#import "Foo.h"

#import "Bar.h"
#import "Baz.h"
#import "Qux.h"

@interface Foo ()

@property (nonatomic) Qux *qux;

@end

// Foo.m

#import "Foo_Private.h"
...

// FooTests.m

#import "Foo_Private.h"
...

```

## Images
Image names should be named consistently to preserve organisation and developer sanity. Please use common judgement when adding them.

We should use asset catalogs for all newly added images for any new feature. The ideas is to have very granular asset catalogs so they can be distributed as modules as part of libraries in the future. When refactoring, keep an eye over legacy code and move images to asset catalogs.

We can have for example:
<App>.xcassets
Feature1.xcassets
Common.xcassets
...

For applications where we only support iOS7 and iPhone, there is no need to request and include non-retina images.

## Booleans

Since `nil` resolves to `NO` it is unnecessary to compare it in conditions. Never compare something directly to `YES`, because `YES` is defined to 1 and a `BOOL` can be up to 8 bits.

This allows for more consistency across files and greater visual clarity.

**For example:**

```objc
if (!someObject) {
}
```

**Not:**

```objc
if (someObject == nil) {
}
```

-----

**For a `BOOL`, here are two examples:**

```objc
if (isAwesome)
if (![someObject boolValue])
```

**Not:**

```objc
if (isAwesome == YES) // Never do this.
if ([someObject boolValue] == NO)
```

-----

If the name of a `BOOL` property is expressed as an adjective, the property can omit the “is” prefix but specifies the conventional name for the get accessor, for example:

```objc
@property (assign, getter=isEditable) BOOL editable;
```
Text and example taken from the [Cocoa Naming Guidelines](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CodingGuidelines/Articles/NamingIvarsAndTypes.html#//apple_ref/doc/uid/20001284-BAJGIIJE).

## Blocks
Using blocks has several memory caveats, which can cause memory leaks in the long term for an app. Because we want to reduce cognitive overload for reviews and devs, the style should reflect a good practice and reduce possible bugs.

When accessing self from **any** block, always declare a weak self. **Always**. It is true that it's not needed for any type of block, if the block is not retained by the caller, as for example in inline animation blocks, but doing so reduces side effects of refactoring. i.e: moving a block inserted in place to a property. We want to be agile and refactor ruthlessly if necessary, so this rule is very important to reduce side effects.

We use libextobjc macros [@weakify/@strongify](http://aceontech.com/objc/ios/2014/01/10/weakify-a-more-elegant-solution-to-weakself.html)


***Example:***
```objc
// Always use weak reference to self, even if it will not cause a retain cycle
@weakify(self);
[UIView animateWithDuration:(animated ? 0.2 : 0.0) animations:^{
    @strongify(self);
    self.inputView.hidden = hidden;
    self.inputView.userInteractionEnabled = !hidden;
    [self updateTableViewContentInsets];
    [self updateScrollIndicatorInsets];
}];
```

***Never:***
```objc
[UIView animateWithDuration:(animated ? 0.2 : 0.0) animations:^{
    self.inputView.hidden = hidden;
    self.inputView.userInteractionEnabled = !hidden;
    [self updateTableViewContentInsets];
    [self updateScrollIndicatorInsets];
}];
```
## Type Inference

Since Xcode 8 we can use __auto_type for type inference in Objective-C. We can mix use of __auto_type in our code with usual type definition.
```objc
- (void)sendSomeNetworkRequest {
    __auto_type completion = ^(NSError *_Nullable error) {
        if (error) {
            [self handleSomeNetworkRequestError:error];
        }
    };
    __auto_type someNetworkRequest = [self makeSomeNetworkRequest];
    [self.networkService sendRequest:someNetworkRequest completion:completion];
}
```

## Singletons

Generally avoid using them if possible, use dependency injection instead.
Nevertheless, needed singleton objects should use a thread-safe pattern for creating their shared instance.
```objc
+ (instancetype)sharedInstance {
   static id sharedInstance = nil;

   static dispatch_once_t onceToken;
   dispatch_once(&onceToken, ^{
      sharedInstance = [[self alloc] init];
   });

   return sharedInstance;
}
```
This will prevent [possible and sometimes prolific crashes](http://cocoasamurai.blogspot.com/2011/04/singletons-your-doing-them-wrong.html).

## Unit Tests
Unit tests is generally not considered by dev teams, but as code, documentation, and important quality tool, they deserve a lot of thought and care. Unit tests should allow developers to move faster and develop faster, not make them lag and suffer. That is why quality tests is as important as quality code, and we care as an engineering team.

Internally we use [XCTest](https://developer.apple.com/library/ios/documentation/DeveloperTools/Conceptual/testing_with_xcode/chapters/01-introduction.html) and [SenTestingKit](http://www.quantum-step.com/download/sources/mystep/OCUnit/SourceCode/SenTestingKit/Documentation/IntroSenTestingKit.html), just because the test base is large enough that switching them to other alternatives is a lot of work. Also alternatives are just syntactic sugar + matchers. We can use matcher libraries if needed.

Write tests. Period.
Prefer writing tests **before** the code under test. Period.

General rule is one assert per test. This should be taken extreme many times, as it
encourages refactoring tests to reuse small utility methods. It also improves tests
readability, which is of utterly importance once test is more than 5 mins old. There are rare cases where more asserts are needed, those are cases where we check values of the same object after some state is set:

```objc
- (void)testCase {
    [self stubModelWithFullData];
    MyCell* cell = [self.underTest provideMyCell];
    XCTAssertEqualObjects(cell.shownName, self.fakeModel.name);
    XCTAssertEqualObjects(cell.shownSurname, self.fakeModel.surname);
    XCTAssertNotNil(cell.avatar);
}
```

***Not allowed:***
```objc
test {
    XCTAssertTrue(self.myObject.correctState);
    [self stubDelegateAndExpectCallbackWithSuccess];
    [self.myObject performCalculations];
    XCTAssertNotNil(self.myObject.name);
    [self.myDelegateMock verify];
}
```
We follow a strict naming convention testcases. It is of great importance because ideally as a developer you want to read the test method and know what it does before even looking at the test code. Also important for error messages. Test names should follow the pattern:
```objc
    testThat_GivenPreconditions_WhenSomethingHappens_ThenIAmExpectingSomething
```

Generally if a test has 'and' in it, it generally means it should be split in two and dev is
lazy:

***Not allowed:***
```objc
testThat_GivenX_WhenServerLoadsWhileIReload_AndILookBack_AndStarsAlign_ThenMagicallyTrue
```

Try to keep tests 3 lines, generally matching the `GIVEN-WHEN-THEN` condition in the test name. Refactor ruthlessly if needed to achieve it. Sometimes it's not possible because of mocking setup, but long test methods are a different type of `smell` we want to avoid.

Don't use any of the 'string' parameters in macro. Leave them to nil or omit when using **XCTest**, and make the test method name be long and meaningful. Those string comments tend to be unmaintained, and copy pasted from somewhere else.

***Not allowed:***
```objc
STAssertTrue(myCondition, @"When moon moves, then the angle between our phone and user hand slightly moved. Thus expecting condition to be true");
```

***Should be:***
```objc
XCTAssertTrue(angleIsSlightlyMovedWhenMovedMoon);
```
Under test goes first in assertions. This improves readability as generally errors are '<first value> should be equal to <second value>. Thus <first value> is not constant, and <second value> is expected value:

***Example:***
```objc
XCTAssertEqualObjects(resultString, @"Hello");
```

***Incorrect:***
```objc
XCTAssertEqualObjects(@"Hello", resultString);
```

## Xcode project

**Always** turn on "Treat Warnings as Errors" in the target's Build Settings and enable as many [additional warnings](http://boredzo.org/blog/archives/2009-11-07/warnings) as possible. If you need to ignore a specific warning, use [Clang's pragma feature](http://clang.llvm.org/docs/UsersManual.html#controlling-diagnostics-via-pragmas).

# Other Objective-C Style Guides

If ours doesn't fit your tastes, have a look at some other style guides:

* [Google](http://google-styleguide.googlecode.com/svn/trunk/objcguide.xml)
* [GitHub](https://github.com/github/objective-c-conventions)
* [Adium](https://trac.adium.im/wiki/CodingStyle)
* [Sam Soffes](https://gist.github.com/soffes/812796)
* [CocoaDevCentral](http://cocoadevcentral.com/articles/000082.php)
* [Luke Redpath](http://lukeredpath.co.uk/blog/my-objective-c-style-guide.html)
* [Marcus Zarra](http://www.cimgf.com/zds-code-style-guide/)
