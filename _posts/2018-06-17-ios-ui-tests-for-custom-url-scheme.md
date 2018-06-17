---
layout:     post
title:      "iOS UI tests for custom URL scheme"
date:       2018-06-17
header_img: img/posts/2018-05-21/header.png
tags:
- ios
- development
- custom url scheme
---

UI tests is a great way to test that your application behaves as you expect it
to and also to verify that you don't break any features in the future. If you
are used to write unit tests you may also be used to write your code using
dependency injections or to mock and stub dependencies. In UI tests however,
you don't have access to your code and to simulate a user as close as possible
you will probably write your tests and interact with other services just as a
regular user. Then you can be quite sure that your app is working as it should.

In this post I will go through how to write UI tests for a custom URL scheme in
iOS.  If you haven't read or need to know how to implement custom URL scheme
you can take another look at my blog post
"[Implement custom URL scheme in iOS](/2018/05/21/implement-custom-url-scheme-in-ios.html)".
I will continue with the source code created using that blog post. So if you
would like to test it yourself you can go back and implement it or
download the source code by the link below.

[Download source code](https://github.com/alex-ross/URLViewer/archive/1.zip).

## Create a UI Test target

We need a UI test target to run UI tests on the simulator or our development
device. So lets create one first.

Go to the "test navigator", see the image below.

![](/img/show-test-navigator.png)

Then at the bottom left corner you will find a "+" button, click on that one
and then on "New UI Test Target..."

![](/img/add-ui-test-target-1.png)

You will probably have everything pre filled now, but make sure that the
project is the same as the current one and that "target to test" is the
actually iOS application that you are testing.

![](/img/add-ui-test-target-2.png)

Click "Finish" and your test target should now be created. It will also create
a test file that you can open, it will be named "URLViewerUITests.swift".

## Write the tests

Usually I start to remove all comments in the generated files because it just
creates a lot of noise. You can also remove the `tearDown`, we will not use
that right now. Finally I also like to have a variable called `app` for the
application itself. So lets add that one and set it up in the `setUp` method.
You should end up with something similar to the code below. Feel free to copy
that one if you need.

```swift
import XCTest

class URLViewerUITests: XCTestCase {
    var app: XCUIApplication!

    override func setUp() {
        super.setUp()
        continueAfterFailure = false
        app = XCUIApplication()
        app.launch()
    }

    func testExample() {
    }
}
```

You can run all tests using `⌘`+`u`. It should open the application in the
simulator and then tell you that all tests passes because we don't assert
anything right now.

In our first test we would like to test that the application launches as usual
and that it has the correct default text. This is a simple test and should look
like this.

```swift
func testDefaultLabelWhenOpenApp() {
    XCTAssert(app.staticTexts["Label"].exists)
}
```

It will basically check that we have the static text "Label" in our application
which we should have because it's the default text we did set to the label. So
if that will change anytime, the test will fail. If you run the tests using
`⌘`+`u` again it should still pass. But try to change the text to something
else and you will see that it will not pass any more.

But we would also like to test that the application changes the URL to the one
that it was opened with. We do know that we can use Safari in the simulator
simulate that the application was opened using an URL. We can automate other
applications in the UI tests as well and use Accessibility Inspector to find
the identifiers for the components we would like to tap or interact with.

Use Xcode meny bar and open Accessibility Inspector. You will find it at
**Xcode** ‣ **Open Developer Tools** ‣ **Accessibility Inspector**.

![](/img/accessibility-inspector-open.png)

Then you can activate the inspection by tapping the button in the circle at the
image below. You will se that the pointer is at the address bar and it will
tell you that the address bar component has the identifier "URL". We can use
that to target the component and tap it.

![](/img/accessibility-inspector-inspect.png)

Se the complete test below.

```swift
func testLabelTextWhenOpenedByURL() {
    // Get the safari app using its bundle identifier
    let safari = XCUIApplication(bundleIdentifier: "com.apple.mobilesafari")
    // Launch safari
    safari.launch()
    // Ensure that safari is running in foreground before we continue
    _ = safari.wait(for: .runningForeground, timeout: 30)

    // Access the address bar component and tap it
    safari.buttons["URL"].tap()
    // You will now have a cursor in the address field. So lets type in the text for our URL
    safari.typeText("urlviewer://example")
    // Make a new line character to simulate that the user taps the return key
    safari.typeText("\n")
    // An pop-up will open and ask you to launch the application. Tap it's "Open" button to confirm
    safari.buttons["Open"].tap()

    // Wait for your application to be the one in foreground
    _ = app.wait(for: .runningForeground, timeout: 5)

    // Now we can assert that we do display the URL in our application
    XCTAssert(app.staticTexts["urlviewer://example"].exists)
}
```

Run the test again and make sure it still passes.

