---
title:  "Building an OSD Front End Using C#: Part 3 Styling Our WPF Form"
date:   2019-04-26 17:00:34 +0000
categories: sccm frontend C#
---
All we have so far is a blank Xaml form without much content. Let's get some basic styles applied to the form and make it look a little more respectable. I have created a custom accent theme for UMN WPF forms using MahApps.Metro. The code for this is below, if you'd like to use it feel free to download and add it to your project as a Xaml file.

There are some good [guides](https://mahapps.com/guides/styles.html) on the web page for MahApps.Metro around how to build a theme file like I've included. You'll notice I basically took their theme template and adapted it to our color scheme.

```xml
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
                    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
                    xmlns:options="http://schemas.microsoft.com/winfx/2006/xaml/presentation/options"
                    mc:Ignorable="options">
    <Color x:Key="HighlightColor">#FFFFCC33</Color>
    <Color x:Key="AccentBaseColor">#FF7A0019</Color>
    <!-- 80% -->
    <Color x:Key="AccentColor">#FF7A0019</Color>
    <!-- 60% -->
    <Color x:Key="AccentColor2">#997A0019</Color>
    <!-- 40% -->
    <Color x:Key="AccentColor3">#667A0019</Color>
    <!-- 20% -->
    <Color x:Key="AccentColor4">#337A0019</Color>

    <Color x:Key="ValidItem">#FF27BC1A</Color>
    <Color x:Key="InvalidItem">#FFBC1A1A</Color>

    <!-- re-set brushes too -->
    <SolidColorBrush x:Key="HighlightBrush" Color="{StaticResource HighlightColor}" options:Freeze="True" />
    <SolidColorBrush x:Key="AccentBaseColorBrush" Color="{StaticResource AccentBaseColor}" options:Freeze="True" />
    <SolidColorBrush x:Key="AccentColorBrush" Color="{StaticResource AccentColor}" options:Freeze="True" />
    <SolidColorBrush x:Key="AccentColorBrush2" Color="{StaticResource AccentColor2}" options:Freeze="True" />
    <SolidColorBrush x:Key="AccentColorBrush3" Color="{StaticResource AccentColor3}" options:Freeze="True" />
    <SolidColorBrush x:Key="AccentColorBrush4" Color="{StaticResource AccentColor4}" options:Freeze="True" />
    <SolidColorBrush x:Key="ValidItemBrush" Color="{StaticResource ValidItem}" options:Freeze="True" />
    <SolidColorBrush x:Key="InvalidItemBrush" Color="{StaticResource InvalidItem}" options:Freeze="True" />

    <SolidColorBrush x:Key="WindowTitleColorBrush" Color="{StaticResource AccentColor}" options:Freeze="True" />

    <LinearGradientBrush x:Key="ProgressBrush" StartPoint="1.002,0.5" EndPoint="0.001,0.5" options:Freeze="True">
        <GradientStop Offset="0" Color="{StaticResource HighlightColor}" />
        <GradientStop Offset="1" Color="{StaticResource AccentColor3}" />
    </LinearGradientBrush>

    <SolidColorBrush x:Key="CheckmarkFill" Color="{StaticResource AccentColor}" options:Freeze="True" />
    <SolidColorBrush x:Key="RightArrowFill" Color="{StaticResource AccentColor}" options:Freeze="True" />

    <Color x:Key="IdealForegroundColor">White</Color>
    <SolidColorBrush x:Key="IdealForegroundColorBrush" Color="{StaticResource IdealForegroundColor}" options:Freeze="True" />
    <SolidColorBrush x:Key="IdealForegroundDisabledBrush" Opacity="0.4" Color="{StaticResource IdealForegroundColor}" options:Freeze="True" />
    <SolidColorBrush x:Key="AccentSelectedColorBrush" Color="{StaticResource IdealForegroundColor}" options:Freeze="True" />

    <!--  DataGrid brushes  -->
    <SolidColorBrush x:Key="MetroDataGrid.HighlightBrush" Color="{StaticResource AccentColor}" options:Freeze="True" />
    <SolidColorBrush x:Key="MetroDataGrid.HighlightTextBrush" Color="{StaticResource IdealForegroundColor}" options:Freeze="True" />
    <SolidColorBrush x:Key="MetroDataGrid.MouseOverHighlightBrush" Color="{StaticResource AccentColor3}" options:Freeze="True" />
    <SolidColorBrush x:Key="MetroDataGrid.FocusBorderBrush" Color="{StaticResource AccentColor}" options:Freeze="True" />
    <SolidColorBrush x:Key="MetroDataGrid.InactiveSelectionHighlightBrush" Color="{StaticResource AccentColor2}" options:Freeze="True" />
    <SolidColorBrush x:Key="MetroDataGrid.InactiveSelectionHighlightTextBrush" Color="{StaticResource IdealForegroundColor}" options:Freeze="True" />

    <SolidColorBrush x:Key="MahApps.Metro.Brushes.ToggleSwitchButton.OnSwitchBrush.Win10" Color="{StaticResource AccentColor}" options:Freeze="True" />
    <SolidColorBrush x:Key="MahApps.Metro.Brushes.ToggleSwitchButton.OnSwitchMouseOverBrush.Win10" Color="{StaticResource AccentColor2}" options:Freeze="True" />
    <SolidColorBrush x:Key="MahApps.Metro.Brushes.ToggleSwitchButton.ThumbIndicatorCheckedBrush.Win10" Color="{StaticResource IdealForegroundColor}" options:Freeze="True" />
</ResourceDictionary>
```

Now we just need to setup our App.Xaml file to include a `<Application.Resources>` section as seen below:

```xml
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.MergedDictionaries>
            <!-- MahApps.Metro resource dictionaries. -->
            <ResourceDictionary Source="pack://application:,,,/MahApps.Metro;component/Styles/Controls.xaml" />
            <ResourceDictionary Source="pack://application:,,,/UMN-OSDFrontend;component/Resources/Icons.xaml" />
            <ResourceDictionary Source="pack://application:,,,/MahApps.Metro;component/Styles/Fonts.xaml" />
            <ResourceDictionary Source="pack://application:,,,/MahApps.Metro;component/Styles/Colors.xaml" />
            <ResourceDictionary Source="pack://application:,,,/UMN-OSDFrontend;component/Styles/CustomAccentUMN.xaml" />
            <!-- Accent and AppTheme Settings -->
            <!-- Un-comment the following line if you're not using the UMN theme -->
            <!--<ResourceDictionary Source="pack://application:,,,/MahApps.Metro;component/Styles/Accents/Blue.xaml" />-->
            <ResourceDictionary Source="pack://application:,,,/MahApps.Metro;component/Styles/Accents/BaseDark.xaml" />
        </ResourceDictionary.MergedDictionaries>
    </ResourceDictionary>
</Application.Resources>
```

![xmal form themed](/assets/images/2019-04-26-Building-an-OSD-Front-End-Using-CSharp-Part-3/xamlformthemed.png)

Now we have a themed window. While we don't have anything on it yet we're now fully ready to build on top of this foundation. I encourage you to play around a little with the theme's and colors while you're building your front end to get a better idea of what looks good and what works for your purposes.

Next post we're going to dive deep into the meat of building the form and talk about the first two tabs for computer naming and pre-flight checks.
