---
id: 372
title: How to Actually Change the System Theme in WPF
date: 2010-12-05T17:21:22+00:00
author: Daniel
layout: post
guid: http://northhorizon.net/?p=372
permalink: /2010/how-to-actually-change-the-system-theme-in-wpf/
categories:
  - Coding
  - Lab49
tags:
  - 'C#'
  - Reflection
  - Styles
  - Themes
  - WPF
  - XAML
---
When I first started working with WPF professionally, it wasn&#8217;t very long before I realized I needed to change the system theme of WPF to give my users a consistent experience across platforms. Not to mention that Vista&#8217;s theme was _much_ improved over XP and even more so over the classic theme. Conceptually, this should be feasible, since WPF has its own rendering engine, as opposed to WinForms relying on GDI.<!--more-->

Naturally, the first thing I did was to [Google the answer](http://www.google.com/search?q=wpf+force+vista+theme). The [first result](http://arbel.net/blog/archive/2006/11/03/Forcing-WPF-to-use-a-specific-Windows-theme.aspx) looked pretty good so I implemented that:

<pre class="brush: xml">&lt;Application.Resources&gt;
    &lt;ResourceDictionary&gt;
        &lt;ResourceDictionary.MergedDictionaries&gt;
            &lt;ResourceDictionary Source="/PresentationFramework.Aero;V4.0.0.0;31bf3856ad364e35;component/themes/aero.normalcolor.xaml" /&gt;
        &lt;/ResourceDictionary.MergedDictionaries&gt;
    &lt;/ResourceDictionary&gt;
&lt;/Application.Resources&gt;</pre>

This worked great up until I started styling things. Any time I created a style for one of the system controls, it&#8217;d change the underlying style back to the original system them, rather than the one I wanted. It turns out that naturally styles are based on the system theme, rather than implicit style (for reasons we&#8217;ll get into later). So the way to base a theme off of the implicit style is like this:

<pre class="brush: xml">&lt;Style x:Key="GreenButtonStyle" TargetType="Button" BasedOn="{StaticResource {x:Type Button}}"&gt;
	&lt;Setter Property="Background" Value="Green"/&gt;
	&lt;Setter Property="Foreground" Value="White"/&gt;
&lt;/Style&gt;</pre>

Maybe a little inconvenient, but it gets the job done:﻿

[<img class="aligncenter size-full wp-image-383" title="Dictionary Based Theme Change" src="http://northhorizon.net/wp-content/uploads/2010/12/dictionary-based-theme-change.png" alt="" width="525" height="350" srcset="http://northhorizon.net/wp-content/uploads/2010/12/dictionary-based-theme-change.png 525w, http://northhorizon.net/wp-content/uploads/2010/12/dictionary-based-theme-change-300x200.png 300w" sizes="(max-width: 525px) 85vw, 525px" />](http://northhorizon.net/wp-content/uploads/2010/12/dictionary-based-theme-change.png)

All was well up until I needed to make a new style based off of the default theme. Like many WPF applications, we had some default styles defined to give our controls a clean, consistent look. But in my particular case, I needed to make a whole new visual &#8220;branch,&#8221; using the default theme as the &#8220;trunk.&#8221; If I used my `BasedOn` workaround, I&#8217;d just get my implicit style; if I didn&#8217;t I&#8217;d get that nasty classic theme.

The more I thought about this, the more I realized that all of these problems were being caused by the fact that I was applying a system theme (aero.normalcolor) as a style on top of the actual system theme, rather than _actually_ changing it. So I set off on a journey in Reflector to find out how WPF picks the current theme. This sounds hard, and it&#8217;s actually a lot harder than it sounds. After a dozen or so hours (spread out over a few weeks) and a some guidance by [a blog that got really close](http://learnwpf.com/Posts/Post.aspx?postId=3f1f4b8b-b91a-442d-a531-919de70ac225) (unfortunately, my link is now dead), I found out that however WPF calls a native method in uxtheme.dll to get the actual system theme, then stores the result in `MS.Win32.UxThemeWrapper`, an internal static class (of course). Furthermore, the properties on the class are read only (and also marked internal), so the best way to change it was by directly manipulating the private fields. My solution looks like this:

<pre class="brush: csharp">public partial class App : Application
{
	public App()
	{
		SetTheme("aero", "normalcolor");
	}

	/// &lt;summary&gt;
	/// Sets the WPF system theme.
	/// &lt;/summary&gt;
	/// &lt;param name="themeName"&gt;The name of the theme. (ie "aero")&lt;/param&gt;
	/// &lt;param name="themeColor"&gt;The name of the color. (ie "normalcolor")&lt;/param&gt;
	public static void SetTheme(string themeName, string themeColor)
	{
		const BindingFlags staticNonPublic = BindingFlags.Static | BindingFlags.NonPublic;

		var presentationFrameworkAsm = Assembly.GetAssembly(typeof(Window));

		var themeWrapper = presentationFrameworkAsm.GetType("MS.Win32.UxThemeWrapper");

		var isActiveField = themeWrapper.GetField("_isActive", staticNonPublic);
		var themeColorField = themeWrapper.GetField("_themeColor", staticNonPublic);
		var themeNameField = themeWrapper.GetField("_themeName", staticNonPublic);

		// Set this to true so WPF doesn't default to classic.
		isActiveField.SetValue(null, true);

		themeColorField.SetValue(null, themeColor);
		themeNameField.SetValue(null, themeName);
	}
}</pre>

I call the method in the `App` constructor so it sets the values after WPF does its detection, but before the system theme is loaded for rendering.

Here&#8217;s what I got back:

<img class="aligncenter size-full wp-image-384" title="Basic Reflection Theme Change" src="http://northhorizon.net/wp-content/uploads/2010/12/basic-reflection-theme-change.png" alt="" width="525" height="350" srcset="http://northhorizon.net/wp-content/uploads/2010/12/basic-reflection-theme-change.png 525w, http://northhorizon.net/wp-content/uploads/2010/12/basic-reflection-theme-change-300x200.png 300w" sizes="(max-width: 525px) 85vw, 525px" />Success!

There&#8217;s still a couple downsides to this approach. First, I can&#8217;t change the theme whenever I want to, and secondly, if the user changes their Windows theme while the application is running, the theme I set gets wiped out.

Wait a minute. If Windows can change the application theme, I can too!

Poking around a little more, I found out that when WPF wants to find a system resource, it also calls `System.Windows.SystemResources.EnsureResourceChangeListener()` which makes sure there&#8217;s a listener attached to the Windows message pump, ready to interpret events like a theme change.

<pre class="brush:csharp">[SecurityCritical, SecurityTreatAsSafe]
private static void EnsureResourceChangeListener()
{
    if (_hwndNotify == null)
    {
        HwndWrapper wrapper = new HwndWrapper(0, -2013265920, 0, 0, 0, 0, 0, "SystemResourceNotifyWindow", IntPtr.Zero, null);
        _hwndNotify = new SecurityCriticalDataClass&lt;HwndWrapper&gt;(wrapper);
        _hwndNotify.Value.Dispatcher.ShutdownFinished += new EventHandler(SystemResources.OnShutdownFinished);
        _hwndNotifyHook = new HwndWrapperHook(SystemResources.SystemThemeFilterMessage);
        _hwndNotify.Value.AddHook(_hwndNotifyHook);
    }
}</pre>

The handler looks like this:

<pre class="brush: csharp">[SecurityTreatAsSafe, SecurityCritical]
private static IntPtr SystemThemeFilterMessage(IntPtr hwnd, int msg, IntPtr wParam, IntPtr lParam, ref bool handled)
{
    WindowMessage message = (WindowMessage) msg;
    switch (message)
    {
        // abridged

        case WindowMessage.WM_THEMECHANGED:
            SystemColors.InvalidateCache();
            SystemParameters.InvalidateCache();
            OnThemeChanged();
            InvalidateResources(false);
            break;
    }
    return IntPtr.Zero;
}</pre>

So the plan was to remove the hook so the message pump couldn&#8217;t change the theme out from under me and then call the handler whenever _I_ want. As it turns out, it&#8217;s even easier than that. Because the list of hooks is implemented with a WeakReferenceList, all I need to do is null out the private `_hwndNotifyHook` field. There&#8217;s just one problem here: killing the handler prevents us from receiving system notifications for the other messages like device changes, power updates, etc. So we really need to inject a filter into the pipeline and pass along everything except the ThemeChanged event.

Fair warning: if you get queasy around reflection, you should probably stop reading here.

<pre class="brush: csharp">public static class ThemeHelper
{
	private const BindingFlags InstanceNonPublic = BindingFlags.Instance | BindingFlags.NonPublic;
	private const BindingFlags StaticNonPublic = BindingFlags.Static | BindingFlags.NonPublic;

	private const int ThemeChangedMessage = 0x31a;

	private static readonly MethodInfo FilteredSystemThemeFilterMessageMethod =
		typeof(ThemeHelper).GetMethod("FilteredSystemThemeFilterMessage", StaticNonPublic);

	private static readonly Assembly PresentationFramework =
		Assembly.GetAssembly(typeof(Window));

	private static readonly Type ThemeWrapper =
		PresentationFramework.GetType("MS.Win32.UxThemeWrapper");
	private static readonly FieldInfo ThemeWrapper_isActiveField =
		ThemeWrapper.GetField("_isActive", StaticNonPublic);
	private static readonly FieldInfo ThemeWrapper_themeColorField =
		ThemeWrapper.GetField("_themeColor", StaticNonPublic);
	private static readonly FieldInfo ThemeWrapper_themeNameField =
		ThemeWrapper.GetField("_themeName", StaticNonPublic);

	private static readonly Type SystemResources =
		PresentationFramework.GetType("System.Windows.SystemResources");
	private static readonly FieldInfo SystemResources_hwndNotifyField =
		SystemResources.GetField("_hwndNotify", StaticNonPublic);
	private static readonly FieldInfo SystemResources_hwndNotifyHookField =
		SystemResources.GetField("_hwndNotifyHook", StaticNonPublic);
	private static readonly MethodInfo SystemResources_EnsureResourceChangeListener = 
 		SystemResources.GetMethod("EnsureResourceChangeListener", StaticNonPublic);
	private static readonly MethodInfo SystemResources_SystemThemeFilterMessageMethod =
		SystemResources.GetMethod("SystemThemeFilterMessage", StaticNonPublic);

	private static readonly Assembly WindowsBase =
		Assembly.GetAssembly(typeof(DependencyObject));

	private static readonly Type HwndWrapperHook =
		WindowsBase.GetType("MS.Win32.HwndWrapperHook");

	private static readonly Type HwndWrapper =
		WindowsBase.GetType("MS.Win32.HwndWrapper");
	private static readonly MethodInfo HwndWrapper_AddHookMethod =
		HwndWrapper.GetMethod("AddHook");

	private static readonly Type SecurityCriticalDataClass =
		WindowsBase.GetType("MS.Internal.SecurityCriticalDataClass`1")
		.MakeGenericType(HwndWrapper);
	private static readonly PropertyInfo SecurityCriticalDataClass_ValueProperty =
		SecurityCriticalDataClass.GetProperty("Value", InstanceNonPublic);

	/// &lt;summary&gt;
	/// Sets the WPF system theme.
	/// &lt;/summary&gt;
	/// &lt;param name="themeName"&gt;The name of the theme. (ie "aero")&lt;/param&gt;
	/// &lt;param name="themeColor"&gt;The name of the color. (ie "normalcolor")&lt;/param&gt;
	public static void SetTheme(string themeName, string themeColor)
	{
		SetHwndNotifyHook(FilteredSystemThemeFilterMessageMethod);

		// Call the system message handler with ThemeChanged so it
		// will clear the theme dictionary caches.
		InvokeSystemThemeFilterMessage(IntPtr.Zero, ThemeChangedMessage, IntPtr.Zero, IntPtr.Zero, false);

		// Need this to make sure WPF doesn't default to classic.
		ThemeWrapper_isActiveField.SetValue(null, true);

		ThemeWrapper_themeColorField.SetValue(null, themeColor);
		ThemeWrapper_themeNameField.SetValue(null, themeName);
	}

	public static void Reset()
	{
		SetHwndNotifyHook(SystemResources_SystemThemeFilterMessageMethod);
		InvokeSystemThemeFilterMessage(IntPtr.Zero, ThemeChangedMessage, IntPtr.Zero, IntPtr.Zero, false);
	}

	private static void SetHwndNotifyHook(MethodInfo method)
	{
		var hookDelegate = Delegate.CreateDelegate(HwndWrapperHook, FilteredSystemThemeFilterMessageMethod);

		// Note that because the HwndwWrapper uses a WeakReference list, we don't need
		// to remove the old value. Simply killing the reference is good enough.
		SystemResources_hwndNotifyHookField.SetValue(null, hookDelegate);

		// Make sure _hwndNotify is set!
		SystemResources_EnsureResourceChangeListener.Invoke(null, null);

		// this does SystemResources._hwndNotify.Value.AddHook(hookDelegate)
		var hwndNotify = SystemResources_hwndNotifyField.GetValue(null);
		var hwndNotifyValue = SecurityCriticalDataClass_ValueProperty.GetValue(hwndNotify, null);
		HwndWrapper_AddHookMethod.Invoke(hwndNotifyValue, new object[] { hookDelegate });
	}

	private static IntPtr InvokeSystemThemeFilterMessage(IntPtr hwnd, int msg, IntPtr wParam, IntPtr lParam, bool handled)
	{
		return (IntPtr)SystemResources_SystemThemeFilterMessageMethod.Invoke(null, new object[] { hwnd, msg, wParam, lParam, handled });
	}

	private static IntPtr FilteredSystemThemeFilterMessage(IntPtr hwnd, int msg, IntPtr wParam, IntPtr lParam, ref bool handled)
	{
		if (msg == ThemeChangedMessage)
			return IntPtr.Zero;

		return InvokeSystemThemeFilterMessage(hwnd, msg, wParam, lParam, handled);
	}
}</pre>

What a mess! What&#8217;s really annoying is that this entire process could have (and, I argue, should have) been made public to user code. In the very least, I&#8217;d like to be able to decorate my assembly with an attribute declaring which theme I&#8217;d like to run under, but I think it&#8217;s completely reasonable to have an public static class provide the same API I have here.

The whole test suite is available <del><a href="http://northhorizon.net/wp-content/uploads/2010/12/SystemThemeChangeTest.zip">here</a></del> on [GitHub](https://github.com/danielmoore/SystemThemeChange).