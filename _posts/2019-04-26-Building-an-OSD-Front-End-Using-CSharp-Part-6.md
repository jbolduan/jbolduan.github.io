---
title:  "Building an OSD Front End Using C#: Part 6 Applications Settings Using JSON"
date:   2019-04-26 17:32:40 +0000
categories: sccm frontend C#
---
Now that we're starting to want logic built behind our interface we've been building we're going to want some way of configuring things so we can enable/disable things or specify useful info. For this iteration of the front end I'm using a json file for the settings. Below is our initial design which we will expand on later in this series of posts.

This is the content of the AppSettings.json file.

```json
{
  "tabs":
    {
      "tabName": "TabComputerName",
      "enabled": true,
      "ruleGreaterThan": 5,
      "ruleGreaterThanEnabled": true,
      "ruleLessThan": 15,
      "ruleLessThanEnabled": true,
      "ruleStartsWith": "SW",
      "ruleStartsWithEnabled": true,
      "ruleEndsWith": "EW",
      "ruleEndsWithEnabled": false
    }
}
```

The first level for right now only contains an array of items called tabs. each tab will have a tab name and and enabled status. It will also sometimes have other items like here we specify a bunch of things about that tab.

In this case, we have specifics about the rules around computer names. These are all editable and will influence the code running behind the front end as we build it. This allows you to have multiple configurations if you need it.

Now let's do the first thing we need to do when working with settings in a json file, setup the class definitions around settings. This first class is called AppSettingsTab.cs and you'll need to add it by right clicking on your project and selecting Add -> Class. This would be the only content you need (edited for your namespace and code base).

```cs
namespace UMN_OSDFrontEnd {
    class AppSettingsTab {
        public string TabName { get; set; }
        public bool Enabled { get; set; } = true;
        public int RuleGreaterThan { get; set; }
        public bool RuleGreaterThanEnabled { get; set; }
        public int RuleLessThan { get; set; }
        public bool RuleLessThanEnabled { get; set; }
        public string RuleStartsWith { get; set; }
        public bool RuleStartsWithEnabled { get; set; }
        public string RuleEndsWith { get; set; }
        public bool RuleEndsWithEnabled { get; set; }
    }
}
```

All of the properties we saw in the settings file are defined here. The one issue is that this is only the settings for a single tab. Our settings file is going to have multiple tabs so we need to go up one level and setup a class file for our settings. In my case I called the class AppSettings.cs and it's added just as before.

```cs
namespace UMN_OSDFrontEnd {
    class AppSettings {
        public List<AppSettingsTab> Tabs;
    }
}
```

As you can see this is a very simple setup. We're just using a List with a type of AppSettingsTab to define the Tabs variable. This means that the Tabs in the settings object will be a list of objects of type AppSettingsTab which will get created when the json file is read.

Now that we have the settings defined in code we can read them in. Below is how we'll want to set up our MainWindow.xaml.cs file.

```cs
public partial class MainWindow : MetroWindow {
    private AppSettings Settings;
    private int ComputerNameGreaterThan;
    private int ComputerNameLessThan;
    private string ComputerNameStartsWith;
    private string ComputerNameEndsWith;

    public MainWindow() {
        InitializeComponent();

        string SettingsFile = File.ReadAllText( Path.Combine( System.AppDomain.CurrentDomain.BaseDirectory.ToString(), "AppSettings.json" ) );
        Settings = JsonConvert.DeserializeObject<AppSettings>( SettingsFile );

        foreach(AppSettingsTab Tab in Settings.Tabs) {
            if(Tab.TabName == "TabComputerName" && Tab.Enabled ) {
                if(!Tab.Enabled) {
                    TabControlMainWindow.Items.Remove( TabComputerName );
                }

                if(!Tab.RuleGreaterThanEnabled) {
                    GridComputerNameRules.Children.Remove( LabelRuleGreaterThan );
                    GridComputerNameRules.Children.Remove( LabelRuleGreaterThanStatus );
                } else {
                    LabelRuleGreaterThan.Content = "REQ - Length >= " + Tab.RuleGreaterThan + ":";
                    ComputerNameGreaterThan = Tab.RuleGreaterThan;
                }

                if(!Tab.RuleLessThanEnabled) {
                    GridComputerNameRules.Children.Remove( LabelRuleLessThan );
                    GridComputerNameRules.Children.Remove( LabelRuleLessThanStatus );
                } else {
                    LabelRuleLessThan.Content = "REQ - Length <= " + Tab.RuleLessThan + ":";
                    ComputerNameLessThan = Tab.RuleLessThan;
                }

                if(!Tab.RuleStartsWithEnabled) {
                    GridComputerNameRules.Children.Remove( LabelRuleStartsWith );
                    GridComputerNameRules.Children.Remove( LabelRuleStartsWithStatus );
                } else {
                    LabelRuleStartsWith.Content = "OPT - Starts With " + Tab.RuleStartsWith + ":";
                    ComputerNameStartsWith = Tab.RuleStartsWith;
                }

                if(!Tab.RuleEndsWithEnabled) {
                    GridComputerNameRules.Children.Remove( LabelRuleEndsWith );
                    GridComputerNameRules.Children.Remove( LabelRuleEndsWithStatus );
                } else {
                    LabelRuleEndsWith.Content = "OPT - Ends With " + Tab.RuleEndsWith + ":";
                    ComputerNameEndsWith = Tab.RuleEndsWith;
                }
            }
        }
    }
}
```

First we need to set up a few variables, we declare our `AppSettings Settings` variable as well as some variables for handling computer name settings within the form.

in the `public MainWindow() { }` function we're going to read in our settings file and use JsonConvert.DeserializeObject into our AppSettings structure and then store it into the Settings variable we declared earlier. This will read the json file and create a structured object how we defined it in Settings making all of the settings available throughout the form as we go through building.

The next bit of code is all around handling tab specific setup code. In this case we want to set up our computer name tab so we go through each tabs settings and determine if it's enabled. This design lets us quickly add new tabs to the form and simply add new setup and in the end we'll have logic around the form completion that has similar logic. In this case however we're going to go through all the rules around computer name and verify if they're enabled and then either enable or disable the labels for that rule.

With that we have the basics around the computer name tab. These basic foundations will let you build additional tabs and generate settings for them and get them initialized. In the future we'll cover using the data entered into the form at completion.
