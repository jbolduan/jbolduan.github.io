---
title:  "Building an OSD Front End Using C#: Part 7 Pre Flight Checks"
date:   2019-04-26 17:49:32 +0000
categories: sccm frontend C#
---
The next phase worked on was to build out the pre-flight checks. The starting point was to build our GUI.

```xml
<TabItem x:Name="TabPreFlight" Header="Pre-Flight">
    <Grid Margin="0,0,0,0">
        <Grid.RowDefinitions>
            <RowDefinition Height="\*" />
            <RowDefinition Height="Auto" />
        </Grid.RowDefinitions>
        <DockPanel Grid.Row="0" VerticalAlignment="Stretch" HorizontalAlignment="Stretch">
            <GroupBox DockPanel.Dock="Top" Header="Pre-Flight Checks" VerticalAlignment="Top" Margin="5,5,5,5">
                <Grid HorizontalAlignment="Left" x:Name="GridPreFlightChecks">
                    <Grid.ColumnDefinitions>
                        <ColumnDefinition Width="Auto" />
                        <ColumnDefinition Width="Auto" />
                    </Grid.ColumnDefinitions>
                </Grid>
            </GroupBox>
        </DockPanel>
        <Button Grid.Row="1" VerticalAlignment="Bottom" HorizontalAlignment="Right" Margin="5,5,5,5" Content="Next >>" Width="125" Click="NextButtonHandler" IsEnabled="True" x:Name="ButtonPreFlightNext" Style="{StaticResource AccentedSquareButtonStyle}" />
    </Grid>
</TabItem>
```
