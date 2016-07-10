---
id: 498
title: Better Commands in WPF
date: 2012-06-05T17:51:22+00:00
author: Daniel
layout: post
guid: http://northhorizon.net/?p=498
permalink: /2012/better-commands-in-wpf/
categories:
  - Coding
  - Lab49
tags:
  - commands
  - linq
  - WPF
---
When I explain commands in WPF, I tend to say they are simply “bindable methods.” I like this definition not only for its brevity, but for its composition of powerful ideas into a single construct. It reinforces the axiom of “no code left behind” that we push so hard in WPF. But this leaves behind almost two thirds of the functionality given to us by `ICommand` the parameter and `CanExecute`. Parameters, admittedly, are needed less often in most designs, as commands tend to operate on the view model to which they are attached. `CanExecute`, on the other hand, is used fairly often and is also fairly broken in most implementations. <!--more-->

On my last project, I had a teammate who was learning WPF and was confused why a button bound to a command was not becoming enabled after its `CanExecute` conditions became `true`. Our implementation of `ICommand` relied on `CommandManager.RequerySuggested` to raise the `ICommand.CanExecuteChanged` event and therein lied the problem. Here’s what [MSDN says about `RequerySuggested`](http://msdn.microsoft.com/en-us/library/system.windows.input.commandmanager.requerysuggested.aspx):

> Occurs when the CommandManager detects conditions that might change the ability of a command to execute.

Hm, well that’s a little vague for my taste. What conditions? And _might_ change? So, after a little reflection, I found that the internal class `System.Windows.Input.CommandDevice` is responsible for raising `RequerySuggested` based on UI events. Here is the relevant method:

<pre class="brush:csharp">private void PostProcessInput(object sender, ProcessInputEventArgs e)
{
   if (e.StagingItem.Input.RoutedEvent == InputManager.InputReportEvent)
   {
      // abridged
   }
   else if (
     e.StagingItem.Input.RoutedEvent == Keyboard.KeyUpEvent ||
     e.StagingItem.Input.RoutedEvent == Mouse.MouseUpEvent || 
     e.StagingItem.Input.RoutedEvent == Keyboard.GotKeyboardFocusEvent || 
     e.StagingItem.Input.RoutedEvent == Keyboard.LostKeyboardFocusEvent)
   {
       CommandManager.InvalidateRequerySuggested();
   }
}</pre>

As you can see, this is really a poor way to detect if `CanExecute` has changed. In many circumstances, view model state changes are based on far more than mouse and keyboard events. In fact, when Microsoft Patterns & Practices made their `DelegateCommand`, they avoided `RequerySuggested` for [just this reason](http://compositewpf.codeplex.com/discussions/44750).

So I advised my colleague to create a normal bindable property called `CanFooCommandExecute` and bind the `IsEnabled` property of the button to that. I think he ended up calling `CommandManager.InvalidateRequerySuggested` himself to limit the amount of rework needed to be done, but that seems far from ideal to me.

This got me thinking, though: why not leverage a bindable property to enable and disable commands? Or, even better, why not define the conditions of a command being enabled in terms of bindable properties?

Fortunately, we have everything we need already to accomplish this. What we can do is instead of taking a `Func<bool>` for our `CanExecute` delegate, we’ll take an `Expression<Func<bool>>`. Then we can look at the AST given to us and find any member accesses with an `INotifyPropertyChanged` root. While we’re at it, we could also look for method calls to `INotifyCollectionChanged` and `IBindingList` objects, since they tell us when their items change.

I have devised a few acceptance tests that demonstrate this behavior. First, let’s define a view model:

<pre class="brush:csharp">class TestViewModel : BindableBase
{
    public TestViewModel()
    {
        SimpleCommand = new ExpressionCommand(OnExecute, () =&gt; Value1 &gt; 5);
    }

    public ICommand SimpleCommand { get; private set; }

    private int _value1;
    public int Value1
    {
        get { return _value1; }
        set { SetProperty(ref _value1, value, "Value1"); }
    }

    private void OnExecute() { }
}</pre>

And let’s just test the basic behavior of the command.

<pre class="brush:csharp">[Test]
public void TestSimpleCommand()
{
    var sut = new TestViewModel();

    int updateCount = 0;
    sut.SimpleCommand.CanExecuteChanged += (s, e) =&gt; updateCount++;

    sut.Value1 = 3;

    updateCount.ShouldEqual(1);
}</pre>

And there it is. No extra management of `CanExecute`, just the syntax you’re probably used to already with `DelegateCommand` Except this time, it works the way you think it should.

Now lets try out a property chain and an `ObservableCollection`. Here are our view models:

<pre class="brush:csharp">private class TestViewModel : BindableBase
{
    public TestViewModel()
    {
        ComplexCommand = new ExpressionCommand(OnExecute, () =&gt; Value2.Value4.Value3 &gt; 5);

        ComplexCollectionCommand = new ExpressionCommand(OnExecute, () =&gt; Value2.Value5.Contains(Value2));
    }

    public ICommand ComplexCommand { get; private set; }

    public ICommand ComplexCollectionCommand { get; private set; }

    private int _value1;
    public int Value1
    {
        get { return _value1; }
        set { SetProperty(ref _value1, value, "Value1"); }
    }

    private TestSubViewModel _value2;
    public TestSubViewModel Value2
    {
        get { return _value2; }
        set { SetProperty(ref _value2, value, "Value2"); }
    }

    private void OnExecute() { }
}

private class TestSubViewModel : BindableBase
{
    private int _value3;
    public int Value3
    {
        get { return _value3; }
        set { SetProperty(ref _value3, value, "Value3"); }
    }

    private TestSubViewModel _value4;
    public TestSubViewModel Value4
    {
        get { return _value4; }
        set { SetProperty(ref _value4, value, "Value4"); }
    }

    public ObservableCollection&lt;TestSubViewModel&gt; Value5 = new ObservableCollection&lt;TestSubViewModel&gt;();
}</pre>

And the tests:

<pre class="brush:csharp">[Test]
public void TestComplexCommand()
{
    var sut = new TestViewModel
    {
        Value2 = new TestSubViewModel()
    };

    int updateCount = 0;
    sut.ComplexCommand.CanExecuteChanged += (s, e) =&gt; updateCount++;

    sut.Value2 = new TestSubViewModel();

    updateCount.ShouldEqual(1);

    sut.Value2.Value4 = new TestSubViewModel();

    updateCount.ShouldEqual(2);

    sut.Value2.Value4.Value3 = 3;

    updateCount.ShouldEqual(3);
}

[Test]
public void TestComplexCollectionCommand()
{
    var sut = new TestViewModel
    {
        Value2 = new TestSubViewModel()
    };

    int updateCount = 0;
    sut.ComplexCollectionCommand.CanExecuteChanged += (s, e) =&gt; updateCount++;

    sut.Value2 = new TestSubViewModel();

    updateCount.ShouldEqual(1);

    sut.Value2.Value5.Add(sut.Value2);

    updateCount.ShouldEqual(2);
}</pre>

Even when complex properties are completely reassigned, `ExpressionCommand` knows to resubscribe to the whole tree again.

I won’t get into the implementation of `ExpressionCommand` in this article as it’s pretty heavy into LINQ expressions, but I have added all the code into my [InpcTemplate on GitHub](https://github.com/danielmoore/InpcTemplate/).