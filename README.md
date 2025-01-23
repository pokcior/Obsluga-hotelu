# Obsluga-hotelu

## Spis treści
- [Opis projektu](#opis-projektu)
- [Funkcjonalności](#funkcjonalności)
- [Technologie](#technologie)
- [Przykłady mechanizmów OOP](#przykłady-mechanizmów-oop)
  - [Hermetyzacja](#1-hermetyzacja-encapsulation)
  - [Dziedziczenie](#2-dziedziczenie-inheritance)
  - [Polimorfizm](#3-polimorfizm-polymorphism)
- [Wymagania systemowe](#wymagania-systemowe)

---

## Opis Projektu

Aplikacja jest systemem zarządzania operacjami hotelowymi, zaprojektowanym w technologii WinForms. Umożliwia ona efektywne zarządzanie hotelem zarówno z poziomu administratora, jak i gościa hotelowego. Administrator ma dostęp do narzędzi zarządzających, takich jak edytowanie statusów pokoi, zarządzanie gośćmi i usługami. Goście hotelowi mogą rezerwować pokoje, przeglądać swoje rezerwacje oraz zamawiać dodatkowe usługi hotelowe.

---

## Funkcjonalności

### Panel Administratora
- Zarządzanie gośćmi hotelowymi (dodawanie, edytowanie, usuwanie).
- Zarządzanie pokojami (dodawanie, edytowanie, zmiana statusów, np. wolny/zajęty).
- Zarządzanie usługami hotelu (np. dodawanie nowych usług, aktualizacja cen).

### Tryb Gościa
- Rezerwacja pokoju przez gościa.
- Zamawianie usług (np. sprzątanie, room service).
- Podgląd rezerwacji (aktualny stan rezerwacji, historia zamówień).

---

## Technologie

- **Język programowania:** C#
- **Framework:** .NET Framework 5.0
- **Baza danych:** MySQL
- **Technologia UI:** WinForms

---

## Przykłady mechanizmów OOP

### **1. Hermetyzacja**
Hermetyzacja polega na ukrywaniu szczegółów implementacji oraz ograniczaniu dostępu do danych poprzez właściwości i metody.

#### **Przykład z `GuestLogin`**
Dane logowania są chronione i walidowane przed wykonaniem operacji:
```csharp
string reservationCode = textBox1?.Text?.Trim() ?? string.Empty;
string roomID = textBox2?.Text?.Trim() ?? string.Empty;

if (string.IsNullOrEmpty(reservationCode) || string.IsNullOrEmpty(roomID))
{
    MessageBox.Show("Proszę wypełnić wszystkie pola!", "Błąd", MessageBoxButtons.OK, MessageBoxIcon.Warning);
    return;
}
```
Dzięki temu tylko poprawne dane przechodzą do dalszego przetwarzania.

#### **Przykład z `AddRoomForm`**
Hermetyczne właściwości umożliwiają dostęp do danych w kontrolowany sposób:
```csharp
public string RoomNumber => textBoxRoomNumber.Text.Trim();
public string RoomType => comboBoxRoomType.SelectedItem?.ToString();
public decimal Price => numericUpDownPrice.Value;
public string Status => comboBoxStatus.SelectedItem?.ToString();
```
Dane wejściowe są weryfikowane i pobierane w sposób ograniczający błędy.

#### **Przykład z `reservation`**
Hermetyzacja umożliwia przetwarzanie danych wejściowych i dynamiczne obliczanie ceny na podstawie wybranych dat:
```csharp
private decimal CalculatePrice()
{
    if (!string.IsNullOrEmpty(selectedRoomID) && selectedCheckInDate < selectedCheckOutDate)
    {
        int days = (selectedCheckOutDate - selectedCheckInDate).Days;

        string connectionString = "Server=localhost;Port=3306;uid=root;pwd=;database=the garden hotel2;";
        using (MySqlConnection conn = new MySqlConnection(connectionString))
        {
            conn.Open();

            string query = "SELECT Price FROM rooms WHERE RoomID = @RoomID";
            using (MySqlCommand cmd = new MySqlCommand(query, conn))
            {
                cmd.Parameters.AddWithValue("@RoomID", selectedRoomID);
                object result = cmd.ExecuteScalar();
                if (result != null)
                {
                    decimal pricePerNight = Convert.ToDecimal(result);
                    return days * pricePerNight;
                }
            }
        }
    }
    return 0;
}
```
Dzięki temu metoda ukrywa szczegóły implementacji i eksponuje jedynie wynik dla logiki wyceny.
Hermetyzacja polega na ukrywaniu szczegółów implementacji oraz ograniczaniu dostępu do danych poprzez właściwości i metody.

#### **Przykład z `GuestLogin`**
Dane logowania są chronione i walidowane przed wykonaniem operacji:
```csharp
string reservationCode = textBox1?.Text?.Trim() ?? string.Empty;
string roomID = textBox2?.Text?.Trim() ?? string.Empty;

if (string.IsNullOrEmpty(reservationCode) || string.IsNullOrEmpty(roomID))
{
    MessageBox.Show("Proszę wypełnić wszystkie pola!", "Błąd", MessageBoxButtons.OK, MessageBoxIcon.Warning);
    return;
}
```
Dzięki temu tylko poprawne dane przechodzą do dalszego przetwarzania.

#### **Przykład z `AddRoomForm`**
Hermetyczne właściwości umożliwiają dostęp do danych w kontrolowany sposób:
```csharp
public string RoomNumber => textBoxRoomNumber.Text.Trim();
public string RoomType => comboBoxRoomType.SelectedItem?.ToString();
public decimal Price => numericUpDownPrice.Value;
public string Status => comboBoxStatus.SelectedItem?.ToString();
```
Dane wejściowe są weryfikowane i pobierane w sposób ograniczający błędy.

---

### **2. Dziedziczenie**
Dziedziczenie umożliwia ponowne wykorzystanie kodu poprzez rozszerzanie klas bazowych.

#### **Przykład z `DatabaseForm`**
Klasa bazowa `DatabaseForm` zapewnia wspólne metody pracy z bazą danych:
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
Klasy pochodne, takie jak `AddRoomForm` i `DeleteServiceForm`, dziedziczą tę funkcjonalność, eliminując potrzebę powtarzania kodu.

#### **Przykład z `ValidationForm`**
Klasa `ValidationForm` dziedziczy po `DatabaseForm`, rozszerzając ją o funkcje walidacji:
```csharp
public abstract class ValidationForm : DatabaseForm
{
    protected bool ValidateEmail(string email) { ... }
    protected bool ValidatePhoneNumber(string phone) { ... }
}
```
Klasa `reservation` automatycznie zyskuje dostęp do tych funkcji:
```csharp
public partial class reservation : ValidationForm { ... }
```

---

### **3. Polimorfizm**
Polimorfizm umożliwia różne zachowania tej samej metody w zależności od klasy lub kontekstu.

#### **Przykład dynamicznego ładowania danych w `Admin`**
Metoda `LoadDataIntoDataGridView` może być używana dla różnych tabel:
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
Dzięki temu jedna metoda obsługuje różne źródła danych, np. pokoje, gości, rezerwacje.

#### **Przykład z formularzami**
Dynamiczne otwieranie różnych formularzy za pomocą wspólnych metod:
```csharp
private void buttonAddRoom_Click(object sender, EventArgs e)
{
    using (AddRoomForm addRoomForm = new AddRoomForm())
    {
        addRoomForm.ShowDialog(); // Wywołanie metody z klasy bazowej Form
    }
}
```
Formularze są tworzone dynamicznie i obsługiwane w zależności od potrzeb.

#### **Przykład z `EditServiceForm`**
Polimorfizm jest widoczny w metodzie `ExecuteQuery`, gdzie różne zapytania SQL mogą być dynamicznie wykonywane:
```csharp
private void ExecuteUpdateService(DataRow row)
{
    string query = @"UPDATE services
                    SET ServiceName = @ServiceName,
                        Description = @Description,
                        PriceService = @PriceService
                    WHERE ServiceID = @ServiceID";

    ExecuteQuery(query, cmd =>
    {
        cmd.Parameters.AddWithValue("@ServiceID", row["ServiceID"]);
        cmd.Parameters.AddWithValue("@ServiceName", row["ServiceName"]);
        cmd.Parameters.AddWithValue("@Description", row["Description"]);
        cmd.Parameters.AddWithValue("@PriceService", row["PriceService"]);
    });
}
```
Dzięki temu metoda `ExecuteQuery` w `DatabaseForm` umożliwia wielokrotne wykorzystanie tej samej logiki dla różnych operacji w systemie.
Polimorfizm umożliwia różne zachowania tej samej metody w zależności od klasy lub kontekstu.

#### **Przykład dynamicznego ładowania danych w `Admin`**
Metoda `LoadDataIntoDataGridView` może być używana dla różnych tabel:
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
Dzięki temu jedna metoda obsługuje różne źródła danych, np. pokoje, gości, rezerwacje.

#### **Przykład z formularzami**
Dynamiczne otwieranie różnych formularzy za pomocą wspólnych metod:
```csharp
private void buttonAddRoom_Click(object sender, EventArgs e)
{
    using (AddRoomForm addRoomForm = new AddRoomForm())
    {
        addRoomForm.ShowDialog(); // Wywołanie metody z klasy bazowej Form
    }
}
```
Formularze są tworzone dynamicznie i obsługiwane w zależności od potrzeb.

---

## Wymagania Systemowe

- **System operacyjny:** Windows 10 lub nowszy
- **Framework:** .NET Framework 5.0
- **Baza danych:** MySQL

Aplikacja umożliwia wygodne zarządzanie zasobami hotelu oraz poprawia komfort korzystania z usług hotelarskich przez gości.

