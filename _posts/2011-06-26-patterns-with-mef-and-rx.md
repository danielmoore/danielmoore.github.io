---
id: 443
title: Patterns with MEF and RX
date: 2011-06-26T17:47:57+00:00
author: Daniel
layout: post
guid: http://northhorizon.net/?p=443
permalink: /2011/patterns-with-mef-and-rx/
categories:
  - Coding
  - Lab49
tags:
  - 'C#'
  - Moq
  - patterns
  - RX
---
Since I started using RX, events have played less and less of a role in my code. It&#8217;s true that in the same way LINQ has relegated for-loops to only niche situations, RX is making events all but obsolete. What does this mean for the `EventAggregator`? Certainly, <a href="http://joseoncode.com/2010/04/29/event-aggregator-with-reactive-extensions/" target="_blank">you can implement your own version using RX</a>. But, if you happen to be using MEF (preferably as your IOC container) you can even get rid of that. <!--more-->

## Introducing the Streams Pattern

First we need to create a &#8220;stream provider&#8221;. The purpose of this class is to establish a single place that &#8220;owns&#8221; the stream. I like to use the stream provider to establish default values and behavior:

<pre class="brush:csharp">public static class StreamNames 
{
  private const string Prefix = "Streams_";

  public const string Accounts = Prefix + "Accounts";
  public const string SelectedAccount = Prefix + "SelectedAccount";
}

// No need to export this type. An instance will be 
// shared between the two child exports.
public class AccountStreamProvider
{
  [ImportingConstructor]
  public AccountStreamProvider(IAccountService accountService)
  {
    // Defer the query for the list of accounts until the first subscription
    var publishedAccounts = Observable
      .Defer(() =&gt; Observable.Return(accountService.GetAccounts()))
      .Publish();

    publishedAccounts.Connect();
    Accounts = publishedAccounts;

    SelectedAccount = new ReplaySubject(1);

    // Take the first account and set it as the initially selected account.
    Accounts
      .Take(1)
      .Select(Enumerable.FirstOrDefault)
      .Subscribe(SelectedAccount.OnNext);
  }

  [Export(StreamNames.Accounts)]
  public IObservable&lt;IEnumerable&lt;Account&gt;&gt; Accounts { get; set; }

  [Export(StreamNames.SelectedAccount)]
  [Export(StreamNames.SelectedAccount, typeof(IObservable&lt;Account&gt;)]
  public ISubject&lt;Account&gt; SelectedAccount { get;set; }
}</pre>

Already the benefits of RX over the `EventAggregator` are showing.

Now we just need to get a reference to our exports in a relevant view model:

<pre class="brush:csharp">public static class Extensions
{
  public static void DisposeWith(this IDisposable source, CompositeDisposable disposables)
  {
    disposables.Add(source);
  }
}

[Export, PartCreationPolicty(CreationPolicy.NotShared)]
public class AccountViewModel : BindableBase, IDisposable
{
  private readonly Streams _streams;

  private readonly CompositeDisposable _disposables;

  [Export]
  public class Streams
  {
    [Import(StreamNames.Accounts)]
    public IObservable&lt;IEnumerable&lt;Account&gt;&gt; Accounts { get; set; }

    [Import(StreamNames.SelectedAccount)]
    public ISubject&lt;Account&gt; SelectedAccount { get; set; }
  }

  [ImportingConstructor]
  public AccountViewModel(Streams streams)
  {
    _streams = streams;

    _disposables = new CompositeDisposable();

    _streams
      .Accounts
      .Subscribe(a =&gt; Accounts = a)
      .DisposeWith(_disposables);

    _streams
      .SelectedAccount
      .Subscribe(a =&gt; SelectedAccount = a)
      .DisposeWith(_disposables);
  }

  private IEnumerable&lt;Account&gt; _accounts;
  public IEnumerable&lt;Account&gt; Accounts
  {
    get { return _accounts; }
    private set { SetProperty(ref _accounts, value, "Accounts"); }
  }

  private Account _selectedAccount;
  public Account SelectedAccount
  {
    get { return _selectedAcccount; }
    private set { SetProperty(ref _selectedAccount, value, "SelectedAccount", OnSelectedAccountChanged); }
  }

  // This method is only called when _selectedAccount 
  // actually changes, so there's no indirect recursion. 
  private void OnSelectedAccountChanged()
  {
    _streams.SelectedAccount.OnNext(_selectedAccount);
  }

  public void Dispose()
  {
    _disposables.Dispose();
  }
}</pre>

By the way, I&#8217;m using `BinableBase` from my [INPC template](https://github.com/danielmoore/InpcTemplate/blob/master/InpcTemplate/BindableBase.cs).

## Introducing the Config Pattern

If you&#8217;ve been using RX for a while, you might have noticed that I forgot to put my setters on the dispatcher. Sure, I could rely on automatic dispatching to fix that problem, but if my queries got any more complicated It&#8217;d be better for me to take care of the dispatch myself.

Of course, the problem with `ObserveOnDispatcher` is that it makes testing a huge pain. Fortunately, we can use MEF to get around that problem, too.

<pre class="brush:csharp; highlight:[19, 34, 40]">public class AccountViewModel : BindableBase, IDisposable
{
  private readonly Streams _streams;
  private readonly Config _config;

  private readonly CompositeDisposable _disposables;

  [Export]
  public class Streams
  {
    [Import(StreamNames.Accounts)]
    public IObservable&lt;IEnumerable&lt;Account&gt;&gt; Accounts { get; set; }

    [Import(StreamNames.SelectedAccount)]
    public ISubject&lt;Account&gt; SelectedAccount { get; set; }
  }

  [Export] 
  public class Config 
  {
    public virtual IScheduler DispatcherScheduler { get { return Scheduler.Dispatcher; } }
  }

  [ImportingConstructor]
  public AccountViewModel(Streams streams, Config config)
  {
    _streams = streams;
    _config = config;

    _disposables = new CompositeDisposable();

    _streams
      .Accounts
      .ObserveOn(_config.DispatcherScheduler)
      .Subscribe(a =&gt; Accounts = a )
      .DisposeWith(_disposables);

    _streams
      .SelectedAccount
      .ObserveOn(_config.DispatcherScheduler)
      .Subscribe(a =&gt; SelectedAccount = a)
      .DisposeWith(_disposables);
  }
  // ...
}</pre>

Since the `DispatcherScheduler` property is virtual, it&#8217;s easy to mock it out. Using Moq, all you need to do is:

<pre class="brush:csharp; gutter:false">Mock.Of&lt;AccountViewModel.Config&gt;(m =&gt; m.DispatcherScheduler == Scheduler.Immediate)</pre>