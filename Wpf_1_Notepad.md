# WPF: Főmenü, Gyorsmenü és TabControl a Jegyzetfüzethez

Ez a dokumentáció bemutatja, hogyan lehet WPF-ben egy **főmenüt**, **gyorsmenüt** és **TabControlt** létrehozni egy **jegyzetfüzet alkalmazás** számára.

A példa tartalmaz egy **menüsort**, egy **jobb kattintással megnyitható menüt**, valamint egy **több füles felületet** a jegyzetek kezeléséhez.

---

## 1. Főmenü (Menu + Access Keys)

A főmenü egy klasszikus alkalmazásmenü, amely a **Fájl, Szerkesztés, Kilépés** lehetőségeket tartalmazza.

Az **Access Keys** (_Fájl, _Szerkesztés) lehetővé teszik a gyorsbillentyűk használatát (**Alt + betű** kombinációval elérhetők).

### XAML kód:

```xml
<Menu VerticalAlignment="Top">
    <MenuItem Header="_Fájl">
        <MenuItem Header="Megnyitás" Click="Menu_Open_Click"/>
        <MenuItem Header="Mentés" Click="Menu_Save_Click"/>
        <Separator/>
        <MenuItem Header="Kilépés" Click="Menu_Exit_Click"/>
    </MenuItem>
</Menu>


private void Menu_Open_Click(object sender, RoutedEventArgs e)
{
    OpenFileDialog openFileDialog = new OpenFileDialog();
    if (openFileDialog.ShowDialog() == true)
    {
        string fileContent = File.ReadAllText(openFileDialog.FileName);
        CreateNewTab(fileContent, Path.GetFileName(openFileDialog.FileName));
    }
}

private void Menu_Save_Click(object sender, RoutedEventArgs e)
{
    if (TabControl.SelectedItem is TabItem selectedTab && selectedTab.Content is TextBox textBox)
    {
        SaveFileDialog saveFileDialog = new SaveFileDialog();
        if (saveFileDialog.ShowDialog() == true)
        {
            File.WriteAllText(saveFileDialog.FileName, textBox.Text);
        }
    }
}

private void Menu_Exit_Click(object sender, RoutedEventArgs e)
{
    Application.Current.Shutdown();
}



<Button Content="Új Jegyzet" Width="150" Height="30" Margin="10" Grid.Row="1" Click="NewTabButton_Click">
    <Button.ContextMenu>
        <ContextMenu>
            <MenuItem Header="Művelet 1" Click="Context_Action1_Click"/>
            <MenuItem Header="Művelet 2" Click="Context_Action2_Click"/>
        </ContextMenu>
    </Button.ContextMenu>
</Button>



private void Context_Action1_Click(object sender, RoutedEventArgs e)
{
    MessageBox.Show("Művelet 1 aktiválva.");
}

private void Context_Action2_Click(object sender, RoutedEventArgs e)
{
    MessageBox.Show("Művelet 2 aktiválva.");
}


<TabControl Name="TabControl" Grid.Row="2" Margin="10">
    <TabItem Header="Új Jegyzet">
        <TextBox Name="TextBox1" Margin="10" VerticalScrollBarVisibility="Auto" HorizontalScrollBarVisibility="Auto" AcceptsReturn="True"/>
    </TabItem>
</TabControl>

### XAML.CS

using System;
using System.IO;
using System.Windows;
using System.Windows.Controls;
using Microsoft.Win32;

namespace Wpf_1_Notepad
{
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();
        }

        // "Új Jegyzet" gomb, új tab létrehozása
        private void NewTabButton_Click(object sender, RoutedEventArgs e)
        {
            TabItem newTab = new TabItem
            {
                Header = "Új Jegyzet"
            };

            TextBox newTextBox = new TextBox
            {
                Margin = new Thickness(10),
                AcceptsReturn = true,
                VerticalScrollBarVisibility = ScrollBarVisibility.Auto,
                HorizontalScrollBarVisibility = ScrollBarVisibility.Auto
            };

            newTab.Content = newTextBox;

            TabControl.Items.Add(newTab);
            TabControl.SelectedItem = newTab;
        }

        // "Megnyitás" gomb - fájl megnyitása
        private void Menu_Open_Click(object sender, RoutedEventArgs e)
        {
            OpenFileDialog openFileDialog = new OpenFileDialog
            {
                Filter = "Text Files (*.txt)|*.txt|All Files (*.*)|*.*"
            };

            if (openFileDialog.ShowDialog() == true)
            {
                string filePath = openFileDialog.FileName;
                string fileContent = File.ReadAllText(filePath);

                TabItem newTab = new TabItem
                {
                    Header = Path.GetFileName(filePath)
                };

                TextBox newTextBox = new TextBox
                {
                    Text = fileContent,
                    Margin = new Thickness(10),
                    AcceptsReturn = true,
                    VerticalScrollBarVisibility = ScrollBarVisibility.Auto,
                    HorizontalScrollBarVisibility = ScrollBarVisibility.Auto
                };

                newTab.Content = newTextBox;
                TabControl.Items.Add(newTab);
                TabControl.SelectedItem = newTab;
            }
        }

        // "Mentés" gomb - fájl mentése
        private void Menu_Save_Click(object sender, RoutedEventArgs e)
        {
            if (TabControl.SelectedItem is TabItem selectedTab && selectedTab.Content is TextBox textBox)
            {
                SaveFileDialog saveFileDialog = new SaveFileDialog
                {
                    Filter = "Text Files (*.txt)|*.txt|All Files (*.*)|*.*"
                };

                if (saveFileDialog.ShowDialog() == true)
                {
                    string filePath = saveFileDialog.FileName;
                    File.WriteAllText(filePath, textBox.Text);
                }
            }
        }

        // "Kilépés" gomb - alkalmazás bezárása
        private void Menu_Exit_Click(object sender, RoutedEventArgs e)
        {
            Application.Current.Shutdown();
        }
    }
}
