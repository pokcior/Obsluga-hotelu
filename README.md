# Obsluga-hotelu
Aplikacja Okienkowa WinForms - Obsługa Hotelu

## 1. Opis Projektu

Aplikacja jest systemem zarządzania operacjami hotelowymi, zaprojektowanym w technologii WinForms. Umożliwia ona efektywne zarządzanie hotelem zarówno z poziomu administratora, jak i gościa hotelowego. Administrator ma dostęp do narzędzi zarządzających, takich jak edytowanie statusów pokoi, zarządzanie gośćmi i usługami. Goście hotelowi mogą rezerwować pokoje, przeglądać swoje rezerwacje oraz zamawiać dodatkowe usługi hotelowe.

## 2. Funkcjonalności

### a) Panel Administratora:
- Zarządzanie gośćmi hotelowymi (dodawanie, edytowanie, usuwanie).
- Zarządzanie pokojami (dodawanie, edytowanie, zmiana statusów, np. wolny/zajęty).
- Zarządzanie usługami hotelu (np. dodawanie nowych usług, aktualizacja cen).

### b) Tryb Gościa:
- Rezerwacja pokoju przez gościa.
- Zamawianie usług (np. sprzątanie, room service).
- Podgląd rezerwacji (aktualny stan rezerwacji, historia zamówień).

## 3. Mechanizmy OOP Zastosowane w Projekcie

### **1. Encapsulacja (Encapsulation)**
Encapsulacja jest widoczna w hermetycznych właściwościach i metodach:

**Przykład z `AddRoomForm`**
```csharp
public string RoomNumber => textBoxRoomNumber.Text.Trim();
public string RoomType => comboBoxRoomType.SelectedItem?.ToString();
public decimal Price => numericUpDownPrice.Value;
public string Status => comboBoxStatus.SelectedItem?.ToString();
```
Powyższe właściwości ograniczają możliwość bezpośredniej manipulacji danymi i umożliwiają ich odczyt w kontrolowany sposób.

**Przykład z `GuestLogin`**
```csharp
string reservationCode = textBox1?.Text?.Trim() ?? string.Empty;
string roomID = textBox2?.Text?.Trim() ?? string.Empty;

if (string.IsNullOrEmpty(reservationCode) || string.IsNullOrEmpty(roomID))
{
    MessageBox.Show("Proszę wypełnić wszystkie pola!", "Błąd", MessageBoxButtons.OK, MessageBoxIcon.Warning);
    return;
}
```
Hermetyzacja zmusza użytkownika do podania poprawnych danych przed dalszym działaniem.

---

### **2. Dziedziczenie (Inheritance)**
Wiele klas dziedziczy wspólne funkcjonalności, co upraszcza i ujednolica projekt.

**Przykład z `DatabaseForm`**
```csharp
public abstract class DatabaseForm : Form
{
    protected string ConnectionString { get; } = "Server=localhost;Port=3306;uid=root;pwd=;database=the garden hotel2;";

    protected MySqlConnection GetConnection()
    {
        return new MySqlConnection(ConnectionString);
    }
}
```
Klasy pochodne, takie jak `AddRoomForm` i `DeleteServiceForm`, dziedziczą tę funkcjonalność, umożliwiając łatwą obsługę bazy danych.

**Przykład z `ValidationForm`**
```csharp
public abstract class ValidationForm : DatabaseForm
{
    protected bool ValidateEmail(string email) { ... }
    protected bool ValidatePhoneNumber(string phone) { ... }
}
```
Klasa `reservation` korzysta z tych metod, dziedzicząc je automatycznie:
```csharp
public partial class reservation : ValidationForm { ... }
```

---

### **3. Polimorfizm (Polymorphism)**
Polimorfizm umożliwia różne zachowania tej samej metody w zależności od klasy lub kontekstu.

**Przykład dynamicznego ładowania danych (`Admin`)**
```csharp
private void LoadDataIntoDataGridView(string query, DataGridView dataGridView)
{
    using (MySqlCommand cmd = new MySqlCommand(query, connection))
    {
        MySqlDataAdapter adapter = new MySqlDataAdapter(cmd);
        DataTable dataTable = new DataTable();
        adapter.Fill(dataTable);
        dataGridView.DataSource = dataTable;
    }
}
```
To samo API działa dla tabel takich jak `guests`, `rooms`, `bookings` itd.

**Przykład z formularzami**
```csharp
private void buttonAddRoom_Click(object sender, EventArgs e)
{
    using (AddRoomForm addRoomForm = new AddRoomForm())
    {
        addRoomForm.ShowDialog(); // Wywołanie metody ShowDialog() z klasy bazowej Form
    }
}
```

---

### **4. Abstrakcja (Abstraction)**

**Przykład z `DatabaseForm`**
```csharp
protected void ExecuteQuery(string query, Action<MySqlCommand> parameterizeCommand)
{
    using (var conn = GetConnection())
    {
        conn.Open();
        using (var cmd = new MySqlCommand(query, conn))
        {
            parameterizeCommand(cmd);
            cmd.ExecuteNonQuery();
        }
    }
}
```
Dzięki temu klasy takie jak `DeleteServiceForm` i `EditRoomForm` mogą korzystać z tej metody bez konieczności implementowania logiki połączenia z bazą danych.

**Przykład z kalendarzem w `reservation`**
```csharp
private void LoadRoomAvailability()
{
    roomAvailability = new Dictionary<DateTime, string>();
    using (MySqlConnection conn = new MySqlConnection(connectionString))
    {
        string query = "SELECT CheckINDate, CheckOUTDate FROM bookings WHERE RoomID = @RoomID";
        using (MySqlCommand cmd = new MySqlCommand(query, conn))
        {
            cmd.Parameters.AddWithValue("@RoomID", selectedRoomID);
            using (MySqlDataReader reader = cmd.ExecuteReader())
            {
                while (reader.Read())
                {
                    DateTime checkInDate = reader.GetDateTime("CheckINDate");
                    DateTime checkOutDate = reader.GetDateTime("CheckOUTDate");
                    for (DateTime date = checkInDate; date < checkOutDate; date = date.AddDays(1))
                    {
                        roomAvailability[date] = "Zajęty";
                    }
                }
            }
        }
    }
}
```
Inne metody używają tych danych bez konieczności znajomości szczegółów implementacji.

---

## 4. Wymagania Systemowe
- System operacyjny: Windows 10 lub nowszy.
- Wersja .NET Framework: 5.0 lub nowsza.

Aplikacja umożliwia wygodne zarządzanie zasobami hotelu oraz poprawia komfort korzystania z usług hotelarskich przez gości.

