# WPF: Főmenü, Gyorsmenü és TabControl a Jegyzetfüzethez

![image](https://github.com/user-attachments/assets/68b1678c-3648-4fe5-b77d-348fd4020d7d)

Ez a dokumentáció bemutatja, hogyan lehet WPF-ben egy **főmenüt**, **gyorsmenüt** és **TabControlt** létrehozni egy **jegyzetfüzet alkalmazás** számára.

A példa tartalmaz egy **menüsort**, egy **jobb kattintással előhozható gyorsmenüt**, valamint egy **több füles felületet** a jegyzetek kezeléséhez.

---

## 1. Főmenü (Menu + Access Keys)

A **főmenü** egy klasszikus alkalmazásmenü, amely az **Új, Betöltés, Mentés, Kilépés** lehetőségeket tartalmazza.
Az **Access Key** (_Fájl) lehetővé teszi a gyorsbillentyűk használatát (**Alt + betű** kombinációval elérhetők).

# XAML kódrészlet

```xml
<Menu VerticalAlignment="Top" Grid.Row="0">
    <MenuItem Header="Fájl">
        <MenuItem Header="Új" Click="NewFileButton_Click"/>
        <MenuItem Header="Betöltés" Click="LoadButton_Click"/>
        <MenuItem Header="Mentés" Click="SaveButton_Click"/>
        <Separator/>
        <MenuItem Header="Kilépés" Click="ExitButton_Click"/>
    </MenuItem>
</Menu>
```

## 2. TabControl
            
A **TabControl** lehetővé teszi, hogy egyszerre több jegyzetet nyissunk meg, mindegyik külön fülön.

### XAML kód:
```xml
<TabControl x:Name="MyTabControl" Grid.Row="1">
    <TabItem Header="Új jegyzet">
        <TextBox x:Name="TextTextBox"
                 Margin="10"
                 VerticalScrollBarVisibility="Auto"
                 HorizontalScrollBarVisibility="Auto"
                 AcceptsReturn="True"
                 TextWrapping="Wrap"/>
    </TabItem>
</TabControl>
```
## 3. Főablak teljes XAML-je

```xml
<Window x:Class="Wpf_1_Notepad.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:teszt"
        mc:Ignorable="d"
        Title="Jegyzettömb" Height="450" Width="800">
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="auto"/>
            <RowDefinition Height="auto"/>
        </Grid.RowDefinitions>

        <Menu VerticalAlignment="Top" Grid.Row="0">
            <MenuItem Header="Fájl">
                <MenuItem Header="Új" Click="NewFileButton_Click"/>
                <MenuItem Header="Betöltés" Click="LoadButton_Click"/>
                <MenuItem Header="Mentés" Click="SaveButton_Click"/>
                <Separator/>
                <MenuItem Header="Kilépés" Click="ExitButton_Click"/>
            </MenuItem>
        </Menu>
        <TabControl x:Name="MyTabControl" Grid.Row="1">
            <TabItem Header="Új jegyzet">
                <TextBox x:Name="TextTextBox" Margin="10" VerticalScrollBarVisibility="Auto" HorizontalScrollBarVisibility="Auto" Width="auto" Height="350"/>
            </TabItem>
        </TabControl>
    </Grid>
</Window>

```


## 4. Magyarázat - MainWindow.xaml.cs

A program 5 fő függvényt tartalmaz:

* MainWindow(): Konstruktor, ami inicializálja az ablakot (pl. betölti a MainWindow.xaml fájlhoz tartozó vizuális elemeket).
```cs
public MainWindow()
        {
            InitializeComponent();
        }
```
* NewFileButton_Click(): Egy új lapot (TabItem) hoz létre „Új jegyzet” címmel, amibe egy TextBox kerül, amiben írhatunk. A lapot hozzáadja a MyTabControl-hoz, és automatikusan kiválasztásra kerül.
  ```cs
  private void NewFileButton_Click(object sender, RoutedEventArgs e)
        {
            TabItem newTab = new TabItem
            {
                Header = "Új jegyzet"
            };

            TextBox textTextBox = new TextBox
            {
                Margin = new Thickness(10),
                Height = 350,
                Width = 750,
                AcceptsReturn = true,
                VerticalScrollBarVisibility = ScrollBarVisibility.Auto,
                HorizontalScrollBarVisibility = ScrollBarVisibility.Auto,
            };

            newTab.Content = textTextBox;

            MyTabControl.Items.Add(newTab); 
            MyTabControl.SelectedItem = newTab; 
        }
  ```
* LoadButton_Click(): Megnyit egy fájlválasztó ablakot (OpenFileDialog), majd kiválaszthatunk a Filternek("Text files (*.txt)|*.txt|All files (*.*)|*.*") megfelelően egy fájlt, ami beolvasásra kerül. Ezt követően egy új lap jelenik meg, melynek a neve a fájl neve, tartalma pedig megegyezik a kiválasztott fájl tartalmával.
```cs
private void LoadButton_Click(object sender, RoutedEventArgs e)
        {
            OpenFileDialog open = new OpenFileDialog
            {
                Filter = "Text files (*.txt)|*.txt|All files (*.*)|*.*"
            };

            if (open.ShowDialog() == true)
            {
                string filePath = open.FileName;
                string fileContent = File.ReadAllText(filePath);
                string header = System.IO.Path.GetFileName(filePath);

                TabItem newTab = new TabItem
                {
                    Header = header
                };

                TextBox textBox = new TextBox
                {
                    Margin = new Thickness(10),
                    Height = 350,
                    Width = 750,
                    AcceptsReturn = true,
                    VerticalScrollBarVisibility = ScrollBarVisibility.Auto,
                    HorizontalScrollBarVisibility = ScrollBarVisibility.Auto,
                    Text = fileContent
                };

                newTab.Content = textBox;

                MyTabControl.Items.Add(newTab); 
                MyTabControl.SelectedItem = newTab;
            }
        }
```
* SaveButton_Click(): Megnyit egy mentési ablakot, melyben ki tudjuk választani, hogy az adott lapot hová mentsük. A lap tartalma, illetve neve mentésre kerül egy ".txt" fájlban. Sikeres mentésnél egy ablak jelenik meg, mely közli, hogy sikerült a mentés.
```cs
 private void SaveButton_Click(object sender, RoutedEventArgs e)
        {
            if (MyTabControl.SelectedItem is TabItem selectedTab && selectedTab.Content is TextBox textTextBox)
            {
                SaveFileDialog saveFileDialog = new SaveFileDialog
                {
                    Filter = "Text files (*.txt)|*.txt|All files (*.*)|*.*"
                };

                if (saveFileDialog.ShowDialog() == true)
                {
                    string filePath = saveFileDialog.FileName;
                    File.WriteAllText(filePath, textTextBox.Text);

                    selectedTab.Header = System.IO.Path.GetFileName(filePath);

                    MessageBox.Show("Sikeres mentés!", "Mentés", MessageBoxButton.OK, MessageBoxImage.Information);
                }
            }
            else
            {
                MessageBox.Show("Nincs kiválasztva menthető jegyzet.", "Hiba", MessageBoxButton.OK, MessageBoxImage.Warning);
            }
        }
```
* ExitButton_Click(): A "Kilépés" menüfűlre kattintva a program kilép. 
```cs
private void ExitButton_Click(object sender, RoutedEventArgs e)
        {
            System.Windows.Application.Current.Shutdown();
        }
```

<details><summary>## 5. C# Kód - MainWindow.xaml.cs</summary>
```cs
using System;
using System.IO;
using System.Windows;
using System.Windows.Controls;
using Microsoft.Win32;

namespace Wpf_1_Notepad
{
    /// <summary>
    /// Interaction logic for MainWindow.xaml
    /// </summary>
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();
        }

        private void NewFileButton_Click(object sender, RoutedEventArgs e)
        {
            TabItem newTab = new TabItem
            {
                Header = "Új jegyzet"
            };

            TextBox textTextBox = new TextBox
            {
                Margin = new Thickness(10),
                Height = 350,
                Width = 750,
                AcceptsReturn = true,
                VerticalScrollBarVisibility = ScrollBarVisibility.Auto,
                HorizontalScrollBarVisibility = ScrollBarVisibility.Auto,
            };

            newTab.Content = textTextBox;

            MyTabControl.Items.Add(newTab); 
            MyTabControl.SelectedItem = newTab; 
        }


        private void LoadButton_Click(object sender, RoutedEventArgs e)
        {
            OpenFileDialog open = new OpenFileDialog
            {
                Filter = "Text files (*.txt)|*.txt|All files (*.*)|*.*"
            };

            if (open.ShowDialog() == true)
            {
                string filePath = open.FileName;
                string fileContent = File.ReadAllText(filePath);
                string header = System.IO.Path.GetFileName(filePath);

                TabItem newTab = new TabItem
                {
                    Header = header
                };

                TextBox textBox = new TextBox
                {
                    Margin = new Thickness(10),
                    Height = 350,
                    Width = 750,
                    AcceptsReturn = true,
                    VerticalScrollBarVisibility = ScrollBarVisibility.Auto,
                    HorizontalScrollBarVisibility = ScrollBarVisibility.Auto,
                    Text = fileContent
                };

                newTab.Content = textBox;

                MyTabControl.Items.Add(newTab); 
                MyTabControl.SelectedItem = newTab;
            }
        }

        private void SaveButton_Click(object sender, RoutedEventArgs e)
        {
            if (MyTabControl.SelectedItem is TabItem selectedTab && selectedTab.Content is TextBox textTextBox)
            {
                SaveFileDialog saveFileDialog = new SaveFileDialog
                {
                    Filter = "Text files (*.txt)|*.txt|All files (*.*)|*.*"
                };

                if (saveFileDialog.ShowDialog() == true)
                {
                    string filePath = saveFileDialog.FileName;
                    File.WriteAllText(filePath, textTextBox.Text);

                    selectedTab.Header = System.IO.Path.GetFileName(filePath);

                    MessageBox.Show("Sikeres mentés!", "Mentés", MessageBoxButton.OK, MessageBoxImage.Information);
                }
            }
            else
            {
                MessageBox.Show("Nincs kiválasztva menthető jegyzet.", "Hiba", MessageBoxButton.OK, MessageBoxImage.Warning);
            }
        }

        private void ExitButton_Click(object sender, RoutedEventArgs e)
        {
            System.Windows.Application.Current.Shutdown();
        }
    }
}
```
</details>
