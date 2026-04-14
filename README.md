# Ex

using TEST;

namespace ShoesShop.Data
{
    public static class AppData
    {
        public static TESTEntities db = new TESTEntities();
        public static Users CurrentUser;
    }
}

----------------------
<Window x:Class="ShoesShop.Views.LoginWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        Title="Авторизация" Height="250" Width="300">

    <StackPanel Margin="20">

        <TextBox x:Name="LoginBox" Margin="5" />
        <PasswordBox x:Name="PasswordBox" Margin="5"/>

        <Button Content="Войти" Click="Login_Click" Margin="5"/>
        <Button Content="Гость" Click="Guest_Click" Margin="5"/>

    </StackPanel>
</Window>

-------------------------------
using System.Linq;
using System.Windows;
using ShoesShop.Data;

namespace ShoesShop.Views
{
    public partial class LoginWindow : Window
    {
        public LoginWindow()
        {
            InitializeComponent();
        }

        private void Login_Click(object sender, RoutedEventArgs e)
        {
            var user = AppData.db.Users.FirstOrDefault(u =>
                u.Login == LoginBox.Text &&
                u.Password == PasswordBox.Password);

            if (user != null)
            {
                AppData.CurrentUser = user;

                new ProductsWindow().Show();
                this.Close();
            }
            else
            {
                MessageBox.Show("Ошибка входа");
            }
        }

        private void Guest_Click(object sender, RoutedEventArgs e)
        {
            AppData.CurrentUser = null;

            new ProductsWindow().Show();
            this.Close();
        }
    }
}
---------------------------
<StackPanel>

    <!-- Поиск -->
    <TextBox x:Name="SearchBox"
             Margin="5"
             TextChanged="FilterChanged"/>

    <!-- Фильтр + сортировка -->
    <StackPanel Orientation="Horizontal" Margin="5">

        <ComboBox x:Name="SortBox"
                  Width="150"
                  SelectionChanged="FilterChanged">
            <ComboBoxItem Content="Без сортировки"/>
            <ComboBoxItem Content="Цена ↑"/>
            <ComboBoxItem Content="Цена ↓"/>
        </ComboBox>

        <ComboBox x:Name="FilterBox"
                  Width="150"
                  Margin="5,0,0,0"
                  SelectionChanged="FilterChanged">
            <ComboBoxItem Content="Все"/>
            <ComboBoxItem Content="Дорогие (>5000)"/>
            <ComboBoxItem Content="Дешевые (<5000)"/>
        </ComboBox>

    </StackPanel>

    <!-- ListView -->
    <ListView x:Name="ProductList" Margin="5">
        <ListView.View>
            <GridView>
                <GridViewColumn Header="Название" DisplayMemberBinding="{Binding Name}"/>
                <GridViewColumn Header="Цена" DisplayMemberBinding="{Binding Price}"/>
            </GridView>
        </ListView.View>
    </ListView>

    <!-- Кнопки -->
    <StackPanel Orientation="Horizontal" HorizontalAlignment="Center">

        <Button x:Name="AddBtn" Content="Добавить" Click="Add_Click" Margin="5"/>
        <Button x:Name="EditBtn" Content="Редактировать" Click="Edit_Click" Margin="5"/>
        <Button x:Name="DeleteBtn" Content="Удалить" Click="Delete_Click" Margin="5"/>

        <!-- НАВИГАЦИЯ -->
        <Button Content="Заказы" Click="OpenOrders_Click" Margin="5"/>

    </StackPanel>

</StackPanel>
-----------------
using System.Collections.Generic;
using System.Linq;
using System.Windows;
using System.Windows.Controls;
using ShoesShop.Data;
using TEST;

namespace ShoesShop.Views
{
    public partial class ProductsWindow : Window
    {
        private List<MainItems> allItems;

        public ProductsWindow()
        {
            InitializeComponent();
            LoadData();
            SetupAccess();
        }

        private void LoadData()
        {
            allItems = AppData.db.MainItems.ToList();

            ProductList.ItemsSource = allItems.Select(x => new
            {
                x.Id,
                Name = x.Name,
                Price = x.Price
            }).ToList();
        }

        private void SetupAccess()
        {
            if (AppData.CurrentUser == null)
            {
                AddBtn.Visibility = Visibility.Collapsed;
                EditBtn.Visibility = Visibility.Collapsed;
                DeleteBtn.Visibility = Visibility.Collapsed;
            }
        }

        private void SearchBox_TextChanged(object sender, TextChangedEventArgs e)
        {
            string search = SearchBox.Text.ToLower();

            ProductList.ItemsSource = allItems
                .Where(x => x.Name.ToLower().Contains(search))
                .Select(x => new
                {
                    x.Id,
                    Name = x.Name,
                    Price = x.Price
                }).ToList();
        }

        private void Add_Click(object sender, RoutedEventArgs e)
        {
            var item = new MainItems()
            {
                Name = "Новый товар",
                Price = 1000
            };

            AppData.db.MainItems.Add(item);
            AppData.db.SaveChanges();

            LoadData();
        }

        private void Edit_Click(object sender, RoutedEventArgs e)
        {
            dynamic selected = ProductList.SelectedItem;
            if (selected == null) return;

            var item = AppData.db.MainItems.Find(selected.Id);

            new EditProductWindow(item).ShowDialog();

            LoadData();
        }

        private void Delete_Click(object sender, RoutedEventArgs e)
        {
            dynamic selected = ProductList.SelectedItem;
            if (selected == null) return;

            var item = AppData.db.MainItems.Find(selected.Id);

            AppData.db.MainItems.Remove(item);
            AppData.db.SaveChanges();

            LoadData();
        }
    }
}

private void ApplyFilters()
{
    var query = allItems.AsQueryable();

    // 🔎 Поиск
    if (!string.IsNullOrWhiteSpace(SearchBox.Text))
    {
        string search = SearchBox.Text.ToLower();
        query = query.Where(x => x.Name.ToLower().Contains(search));
    }

    // 🎯 Фильтр
    if (FilterBox.SelectedIndex == 1)
        query = query.Where(x => x.Price > 5000);

    if (FilterBox.SelectedIndex == 2)
        query = query.Where(x => x.Price < 5000);

    // 🔃 Сортировка
    if (SortBox.SelectedIndex == 1)
        query = query.OrderBy(x => x.Price);

    if (SortBox.SelectedIndex == 2)
        query = query.OrderByDescending(x => x.Price);

    // 📦 Вывод
    ProductList.ItemsSource = query
        .Select(x => new
        {
            x.Id,
            Name = x.Name,
            Price = x.Price
        })
        .ToList();
}


--------------------
