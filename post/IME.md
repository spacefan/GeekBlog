
##IMS简介

输入法这样的应用放在各自平台都至关重要，用我们实验室的教授的话就是：兵家必争之地。在安卓平台，输入法的开发基本上是基于google提供的框架来完成的。

inputMethodService正是为输入法提供了这样一个框架的核心组件。它继承于父类 AbstractInputMethodService 同时实现了 InputMethod 的众多接口。如果想要细究其内部实现的话，恐怕就要详细阅读一下这两个代码文件了。

作为一个Service组件，常见的生命周期方法无需赘言。同时IMS也提供了其他一些可能有用的方法。

- onInitializeInterface() 用于一些接口的初始化，比如IMS运行时处理一些配置参数的改变。（我也不太懂这个，待我实际去写的时候体验下再回来修改）

- onBindInput()用于输入法绑定到一个新的应用时获取该应用的信息。

- onStartInput()用于处理与一个窗口建立的联系

- onCreateInputView(), onCreateCandidatesView(), 以及 onCreateExtractTextView()都是用于UI的产生，具体什么UI，顾名思义啦。

- onStartInputView(EditorInfo, boolean)用于处理编辑文本区域的各种输入操作。

---

##IMS框架

IMS事实上已经为开发者提供了一个基本的UI框架。包括输入界面（键盘部分）、候选词界面、全屏模式。但是这个基础框架其实称之为基础选项也许更加恰当，因为这些组件都是可选的。换句话说，你可以创造一个只拥有候选词界面的输入法（通过语音或者手写输入），也可以创造一个只有输入界面，产生的文本直接打印到当前当前应用里的输入法。

所有的可视组件会放置在IMS管理的一个窗口里进行统一布局。也就是说，IMS是可以与输入法界面进行直接的信息传递，并通过一些专门的API控制它。

在这个基本框架的布局问题上，有一些需要注意的问题。

- 如果软键盘部分存在，会被自动安排到屏幕底部。

- 候选词界面显示时，会被自动安排到屏幕上端。

- 所谓全屏模式，即覆盖当前应用还是与当前应用共享同一空间。

---

##软键盘

大多数输入法可能最重要的界面就是软键盘了。在这里会发生用户大部分的输入操作：选择字符，书写字符或者其他任何产生文本的操作方式。大部分实现都是通过view自己去实现以上的功能并在onCreateInputView()被回调的时候创造这个view的实例。因此，只要软键盘出现在窗口里，那么用户就可以进行各种输入操作，并将输入操作提交到IMS里进行统一处理，视情况与当前应用产生互动。

有些情况下可能需要对软键盘是否显示进行控制。 实现onEvaluateInputViewShown()，返回值决定了是否在当前应用显示软键盘。如果有某些状态改变需要即时控制软键盘的显示，updateInputViewShown()可以重新绘制软键盘。默认的实现是总是显示软键盘，除非接入了物理键盘。这样的实现可能对于大多数输入法都是适用的。

---

##候选词界面

候选词界面顾名思义，即用户选完一定字符时，输入法会在这个界面推荐一些用户可能选择的词。在软键盘界面文件中实现onCreateCandidatesView()方法即可创造与有候选词界面的软键盘实例。

候选词界面的管理不同于输入界面，因为这个界面只有当用户输入文本有可能有候选词提供的时候才会显示。你可以通过调用setCandidatesViewShown(boolean)方法用来控制该界面是否显示。
需要记住的一点是，因为候选词界面需要频繁的出现消失，所以不会像软键盘界面一样影响当前应用UI，他永远不会使得当前应用重新计算界面大小，如果用户需要看到当前焦点位置，当前应用只会平移。

---

##全屏模式

全屏模式就是窗口内不再显示当前应用，而是显示软键盘、候选词界面以及文本编辑界面。文本编辑界面是指一个可以给用户看到他当前正在输入的文本的界面。文本编辑界面的位置不可改变。他的位置在输入法最上方。

通过实现onEvaluateFullscreenMode()方法控制IME是否启用全屏模式。如果状态改变将会影响到全屏模式的话，可以调用updateFullscreenMode()方法来动态控制模式的选择。

因为全屏模式下用户看不到当前应用的UI，所以你会有一些特殊要求。你可以通过调用onDisplayCompletions(CompletionInfo[])方法来显示一些你的应用的一些返回信息，比如候选词。

---

##产生文本

IME的按键部分理所当然的应该用来对应用产生文本。调用android.view.inputmethod.InputConnection接口可以实现与应用的信息传递，getCurrentInputConnection()可以取得这个传递。如果当前应用支持，这个接口允许你在应用中直接编辑文本。

当前应用需要、支持的信息可以通过 android.view.inputmethod.EditorInfo 类得到。这个类可以通过 getCurrentInputEditorInfo() 方法获得。这个类中最重要的部分是EditorInfo.inputType 。比如.inputType是EditorInfo.TYPE_NULL，那么说明当前应用不支持复杂输入，你只需要提供键盘输入就可以。输入法也想获得一些应用的信息，比如密码模式，电话号格式，自动完成文本等。

用户在输入法、应用之间切换的时候，输入法将回调 onFinishInput() 和 onStartInput(EditorInfo, boolean) 方法。你可以通过他们来针对当前应用，初始化或重设你的输入法状态。比如清空文本、更改软键盘类型来适应不同的应用。