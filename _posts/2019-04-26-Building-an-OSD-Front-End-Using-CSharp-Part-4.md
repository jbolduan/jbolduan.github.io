---
title:  "Building an OSD Front End Using C#: Part 4 Adding Controls and Going Deeper"
date:   2019-04-26 17:00:40 +0000
categories: sccm frontend C#
---
Up until now we've been very focused on getting everything ready. Now we're ready to actually start building out our form controls and some of the logic behind them.

Let's get our MainWindow.Xaml setup to start adding tabs, for this we'll need to add some basic framework controls where we can build everything inside (I've cut all the extra stuff out of the Controls:MetroWindow tag):

```xml
<Controls:MetroWindow>
    <Grid>
        <TabControl x:Name="TabControlMainWindow" Margin="0,0,0,0" TabStripPlacement="Left" FontFamily="Arial" FontSize="14" FontWeight="Bold" RenderTranformOrigin="0.5,0.5">

        </TabControl>
    </Grid>
</Controls:MetroWindow>
```

Everything we write for the main form from here on out will be within the TabControl. Let's go through a little of what we have here.

The first new control we've added is a `<Grid>` which is needed as the base. We don't need to add anything to it since it will just be the primary container on our form.

Next we have the main piece of our form which is a `<TabControl>` control. We need to define several properties of the TabControl to make sure that it functions properly on our form:

1. x:Name - Name that the control will be called including in code later on.
2. Margin - We want the TabControl going all the way out to the edge of the form so we make this 0 on all sides.
3. TabStripPlacement - Moves the tabs from the top of the form to the left side as a vertical list.
4. FontFamily - This just changes the font used on the tab names.
5. FontSize - Change the font size for the tab names.
6. FontWeight - Make the text bold for the tab names.
7. RenderTransformOrigin - Determines where the center point of the control.

The next thing we're going to want to add is our first `<TabItem>` control. Here's the code for the computer name tab and below the code we'll discuss what we're doing.

```xml
<TabItem x:Name="tabComputerName" Header="Computer Name">
    <Grid x:Name="gridComputerName" Margin="0,0,0,0">
        <Grid.RowDefinitions>
            <RowDefinition Height="\*" />
            <RowDefinition Height="Auto" />
        </Grid.RowDefinitions>
        <Button Grid.Row="1" x:Name="buttonComputerNameNext" VerticalAlignment="Bottom" HorizontalAlignment="Right" Margin="5,5,5,5" Content="Next >>" Width="125" Click="NextButtonHandler" />
    </Grid>
</TabItem>
```

Inside a `<TabControl>` you have `<TabItem>`'s. Think of a tab item as just a container for all the controls on a given tab. I've found the best way for me when building a tab in the front end is to put down a `<Grid>` control like we had before but this time to define two rows. The first row will expand to fill as much space as it can. The second row will grow to the height it requires but no more.

Now, you'll see I've added a button called buttonComputerNameNext. You'll also notice it's in the Grid.Row 1. This is the second row (numbering starts from 0) and we're putting it in the bottom right hand corner. This is the next button. I've also added a click event handler. Below is the code you'll want to add to the MainWindow.xaml.cs file.

```cs
private void NextButtonHandler( object sender, RoutedEventArgs e ) {
    TabControlMainWindow.SelectedIndex++;
    ( TabControlMainWindow.Items\[tabControlMainWindow.SelectedIndex\] as TabItem ).IsEnabled = true;
}
    ```

This simple button handler will increase the SelectedIndex property of the TabControlMainWindow. The selected index is the index of the currently selected tab. This code allows all our next buttons to share the same event handler and will also allow us to enable or disable tabs without needing special code to handle those cases. We can also re-order tabs through the designer without worrying about updating code to change tab orders on the next buttons either.

It also then enables the control which is disabled by default so it cannot be selected. To do that we need to get the item and the selected index and cast it to be a TabItem so we can use it's properties.
