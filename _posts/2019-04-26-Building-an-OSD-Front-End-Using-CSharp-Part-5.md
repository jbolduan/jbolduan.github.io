---
title:  "Building an OSD Front End Using C#: Part 5 Computer Name Form Design"
date:   2019-04-26 17:15:30 +0000
categories: sccm frontend C#
---
Now that we have the basic layout of the computer name tab built but we don't have any of the computer naming logic that's our next step.

When it comes to the layout, at the same level as the button from our last example or right below the closing `</Grid.RowDefinitions>` tag we're going to start building our computer name input form. We'll start by adding a `<DockPanel>` control.

```xml
<DockPanel Grid.Row="0" HorizontalAlignment="Stretch" VerticalAlignment="Stretch">
</DockPanel>
```

A dock panel is a control which will allow us to dock various controls inside it and get them to stretch to the full width. In this case, I wanted to have our group box controls stretch to fill the width of the tab so we don't have weird ratios of empty space as we move between tabs or between different group box controls on the same tab. As with the button we assign the dock panel to a row but in this case we assign to Grid.Row="0" which is the very first row in the grid. The horizontal and vertical alignments are set to stretch so the dock panel fills all the space available.

Now, let's build our computer name section of the form.

```xml
<GroupBox DockPanel.Dock="Top" Header="Computer Name" VerticalAlignment="Top" Margin="5,5,5,5">
    <Grid HorizontalAlignment="Center">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto" />
        </Grid.RowDefinitions>
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="Auto" />
            <ColumnDefinition Width="Auto" />
        </Grid.ColumnDefinitions>
        <Label Grid.Row="0" Grid.Column="0" Content="Enter Computer Name:" HorizontalAlignment="Left" Margin="10,10,10,10" VerticalAlignment="Top" />
        <TextBox Grid.Row="0" Grid.Column="1" x:Name="TextBoxComputerName" Width="175" HorizontalAlignment="Left" Margin="10,10,10,10" VerticalAlignment="Top" CharacterCasing="Upper" TextChanged="TextBoxComputerName\_TextChanged" />
    </Grid>
</GroupBox>
```

We're using a new control called a group box. It lets us put groups of content together into a single control when they're related. In this case it's to contain a label and a text box which prompts for a computer name to be inputted.

You'll notice that we again use a grid and this time we have both rows and columns defined to help us align everything and make it look neat and orderly.

There is a TextChanged event trigger that we've added to the text box. We can ignore that for now and we'll get to that later on in the behind the form code section.

The next part of the computer name entry form is the table where we show any validation rules and if they pass or fail:

```xml
<GroupBox DockPanel.Dock="Top" Header="Computer Name Validation" VerticalAlignment="Top" Margin="5,5,5,5">
    <Grid HorizontalAlignment="Left" x:Name="GridComputerNameRules">
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="Auto" />
            <ColumnDefinition Width="Auto" />
        </Grid.ColumnDefinitions>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto" />
            <RowDefinition Height="Auto" />
            <RowDefinition Height="Auto" />
            <RowDefinition Height="Auto" />
        </Grid.RowDefinitions>
        <Label Grid.Column="0" Grid.Row="0" Content="REQ - Length >= 5:" x:Name="LabelRuleGreaterThan" />
        <Label Grid.Column="1" Grid.Row="0" Content="False" x:Name="LabelRuleGreaterThanStatus" Foreground="{StaticResource InvalidItemBrush}" />
        <Label Grid.Column="0" Grid.Row="1" Content="REQ - Length <= 15:" x:Name="LabelRuleLessThan" />
        <Label Grid.Column="1" Grid.Row="1" Content="False" x:Name="LabelRuleLessThanStatus" Foreground="{StaticResource InvalidItemBrush}" />
        <Label Grid.Column="0" Grid.Row="2" Content="OPT - Starts with UMN:" x:Name="LabelRuleStartsWith" />
        <Label Grid.Column="1" Grid.Row="2" Content="False" x:Name="LabelRuleStartsWithStatus" Foreground="{StaticResource InvalidItemBrush}" />
        <Label Grid.Column="0" Grid.Row="3" Content="OPT - Ends with blah:" x:Name="LabelRuleEndsWith" />
        <Label Grid.Column="1" Grid.Row="3" Content="False" x:Name="LabelRuleEndsWithStatus" Foreground="{StaticResource InvalidItemBrush}" />
    </Grid>
</GroupBox>
```

Here we see a lot of things we've built into the form already just in a slightly different way. You will notice we're leveraging a `{StaticResource}` for InvalidItemBrush. This lets us define the color of an invalid item and then simply reference it and change it by changing our theme file. If you didn't use the UMN theme file you may need to configure this in a xaml file.

Now we can move behind the scenes to look at how this works. However, before we move behind the form we'll need to spend a little time on how we're going to handle settings and configuration. In the next post we'll go over the AppSettings.json file and how we parse the json file and use those settings in the application.
