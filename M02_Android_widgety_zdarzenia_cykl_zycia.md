# Android – widgety, zdarzenia, cykl życia

**Programowanie aplikacji mobilnych | klasa 4 | materiał M02**

Pierwsza prawdziwa aplikacja na jednym ekranie: pola tekstowe, przyciski, checkboxy, lista wyboru. Obsługa wejścia użytkownika i jego walidacja – odpowiednik formularzy z materiału 11, tylko bez Django, które sprawdzało za Ciebie. Do tego cykl życia Activity, czyli odpowiedź na pytanie z M01: dlaczego obrót telefonu zeruje licznik.

---

## Spis treści

1. LinearLayout – prostszy kontener
2. Widgety: co jest w pudełku
3. EditText – wejście od użytkownika
4. Zdarzenia: kliknięcia i nie tylko
5. Walidacja wejścia
6. Cykl życia Activity
7. Zapisywanie stanu przy obrocie
8. Aplikacja: kalkulator BMI
9. Najczęstsze problemy
10. Zadania
11. Pytania kontrolne

---

## 1. LinearLayout – prostszy kontener

`ConstraintLayout` z M01 jest potężny, ale na prosty formularz w pionie to przesada. **`LinearLayout`** układa dzieci jedno pod drugim (`vertical`) albo obok siebie (`horizontal`). Bez constraintów – kolejność w XML = kolejność na ekranie.

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@string/title"
        android:textSize="24sp" />

    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="8dp"
        android:text="@string/button_ok" />

</LinearLayout>
```

| Atrybut | Co robi |
|---|---|
| `android:orientation="vertical"` | **obowiązkowy** w LinearLayout. Bez niego domyślnie `horizontal` i wszystko leci w jednym wierszu poza ekran |
| `android:padding="16dp"` | odstęp **wewnątrz** elementu (od krawędzi do treści). `margin` – odstęp **na zewnątrz** (od sąsiadów). Jak w CSS |
| `android:layout_weight="1"` | podział miejsca między dzieci. Dwa przyciski z `weight=1` i `layout_width="0dp"` w poziomym LinearLayout → po połowie ekranu |
| `android:gravity="center"` | wyrównanie **treści** wewnątrz elementu. `layout_gravity` – wyrównanie **elementu** w rodzicu |

Zagnieżdżanie: poziomy LinearLayout wewnątrz pionowego daje wiersz z dwoma przyciskami obok siebie. Do trzech poziomów w porządku; głębiej – wróć do ConstraintLayout, bo zagnieżdżone LinearLayouty są wolne.

Kiedy który:

- formularz, lista pól jedno pod drugim → `LinearLayout vertical`,
- dwa elementy obok siebie → `LinearLayout horizontal`,
- cokolwiek, co ma być „przy prawej krawędzi, pod tamtym, wyśrodkowane” → `ConstraintLayout`.

---

## 2. Widgety: co jest w pudełku

**Widget** (albo *View*) to element interfejsu. Wszystkie dziedziczą po klasie `View`, więc mają wspólne: `id`, `layout_width/height`, `visibility`, `setOnClickListener`. Najczęstsze:

| Widget | HTML | Do czego | Kluczowe atrybuty / metody |
|---|---|---|---|
| `TextView` | `<p>`, `<h1>` | napis | `text`, `textSize`, `textColor`, `textStyle="bold"` / `setText()` |
| `EditText` | `<input>` | pole do wpisania | `hint`, `inputType` / `getText()` |
| `Button` | `<button>` | przycisk | `text` / `setOnClickListener()` |
| `ImageView` | `<img>` | obrazek | `src="@drawable/…"` / `setImageResource()` |
| `CheckBox` | `<input type="checkbox">` | tak/nie | `checked` / `isChecked()` |
| `Switch` | – | przełącznik (to samo co CheckBox, inny wygląd) | `isChecked()` |
| `RadioGroup` + `RadioButton` | `<input type="radio">` | jeden z kilku | `getCheckedRadioButtonId()` |
| `Spinner` | `<select>` | lista rozwijana | `entries="@array/…"` / `getSelectedItem()` |
| `SeekBar` | `<input type="range">` | suwak | `max`, `progress` / `getProgress()` |
| `ProgressBar` | – | kółko ładowania | `visibility` |

### 2.1 Visibility

Trzy stany, których będziesz używać ciągle:

```java
view.setVisibility(View.VISIBLE);    // widać
view.setVisibility(View.INVISIBLE);  // nie widać, ale zajmuje miejsce
view.setVisibility(View.GONE);       // nie widać i nie zajmuje miejsca
```

W XML: `android:visibility="gone"`. Typowy wzorzec: `ProgressBar` na `gone`, przełączany na `visible` w czasie ładowania danych (M06).

### 2.2 Obrazki

Pliki `.png`/`.jpg`/`.webp` wrzucasz do `res/drawable/`. Nazwa: małe litery, cyfry, podkreślniki, **bez myślników i bez polskich znaków** – to będzie nazwa stałej w `R.drawable`. Potem `android:src="@drawable/logo"`.

Ikony systemowe: prawy klik na `res` → *New → Vector Asset* → wybierz z biblioteki Material. Dostaniesz `.xml` z wektorem, skaluje się bez utraty jakości.

---

## 3. EditText – wejście od użytkownika

```xml
<EditText
    android:id="@+id/edit_weight"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:hint="@string/hint_weight"
    android:inputType="numberDecimal"
    android:autofillHints="" />
```

| Atrybut | Co robi |
|---|---|
| `hint` | szary tekst-podpowiedź, znika po wpisaniu. Odpowiednik `placeholder` |
| `inputType` | **jaka klawiatura się pokaże i co wolno wpisać**: `text`, `textEmailAddress`, `textPassword`, `number`, `numberDecimal`, `numberSigned`, `phone`, `date`. Używaj zawsze – użytkownik z klawiaturą numeryczną nie wpisze liter, co załatwia część walidacji za darmo |
| `maxLength="50"` | limit znaków |
| `lines="3"` | pole wielowierszowe (z `inputType="textMultiLine"`) |

W Javie:

```java
EditText editWeight = findViewById(R.id.edit_weight);

String text = editWeight.getText().toString();     // zawsze String, nawet dla number
String trimmed = text.trim();                      // obetnij spacje
```

`getText()` zwraca `Editable`, nie `String` – stąd `.toString()`. Zawsze. Potem konwersja:

```java
double weight = Double.parseDouble(trimmed);       // NumberFormatException, jeśli puste lub "abc"
int age = Integer.parseInt(trimmed);
```

Wpisywanie z kodu: `editWeight.setText("70")`. Wyczyść: `editWeight.setText("")`. Kursor i klawiatura: `editWeight.requestFocus()`.

---

## 4. Zdarzenia: kliknięcia i nie tylko

### 4.1 Trzy sposoby na kliknięcie

**Lambda** – używaj tego:

```java
buttonCalc.setOnClickListener(v -> calculate());
```

**Klasa anonimowa** – to samo, co lambda, tylko dłużej. Zobaczysz w starych tutorialach i w Stack Overflow:

```java
buttonCalc.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        calculate();
    }
});
```

**Atrybut w XML** – `android:onClick="calculate"` + metoda `public void calculate(View v)` w Activity. Wygląda prosto, ale kompilator nie sprawdza, czy metoda istnieje – błąd wyjdzie dopiero przy kliknięciu. Nie używaj.

Jeden listener na kilka przycisków: `v.getId()` mówi, który kliknięto:

```java
View.OnClickListener listener = v -> {
    int id = v.getId();
    if (id == R.id.button_add) {
        add();
    } else if (id == R.id.button_clear) {
        clear();
    }
};
buttonAdd.setOnClickListener(listener);
buttonClear.setOnClickListener(listener);
```

(`switch` na `R.id.*` nie działa od kilku lat – id nie są już stałymi kompilacji. `if/else`.)

### 4.2 Inne zdarzenia

```java
// długie przytrzymanie – zwróć true, jeśli obsłużone (inaczej poleci też zwykły click)
button.setOnLongClickListener(v -> { ...; return true; });

// checkbox / switch – zmiana stanu
checkBox.setOnCheckedChangeListener((buttonView, isChecked) -> { ... });

// radio – który wybrano
radioGroup.setOnCheckedChangeListener((group, checkedId) -> { ... });

// suwak – w trakcie przesuwania
seekBar.setOnSeekBarChangeListener(new SeekBar.OnSeekBarChangeListener() {
    @Override public void onProgressChanged(SeekBar sb, int progress, boolean fromUser) { ... }
    @Override public void onStartTrackingTouch(SeekBar sb) { }
    @Override public void onStopTrackingTouch(SeekBar sb) { }
});

// pole tekstowe – przy każdej zmianie (walidacja na żywo)
editText.addTextChangedListener(new TextWatcher() {
    @Override public void beforeTextChanged(CharSequence s, int start, int count, int after) { }
    @Override public void onTextChanged(CharSequence s, int start, int before, int count) { }
    @Override public void afterTextChanged(Editable s) { ... }
});
```

Tam, gdzie interfejs ma jedną metodę – lambda. Tam, gdzie kilka (`SeekBar`, `TextWatcher`) – klasa anonimowa, bo lambda nie umie trzech metod naraz. Niepotrzebne zostawiasz puste.

> 💡 Wzorzec, który widzisz, to **Observer** z listy wzorców projektowych w podstawie (3.8): widok „obserwuje” zdarzenia i woła Twój kod, gdy coś się stanie. Listener = obserwator. Będziesz o to pytany na egzaminie z nazwy.

---

## 5. Walidacja wejścia

W Django walidację robił `forms.Form`. Tu robisz ją sam – ale wzorzec jest ten sam: sprawdź, jeśli źle – pokaż błąd przy polu, jeśli dobrze – działaj.

```java
private boolean validate() {
    String weightText = editWeight.getText().toString().trim();
    String heightText = editHeight.getText().toString().trim();
    boolean valid = true;

    if (weightText.isEmpty()) {
        editWeight.setError(getString(R.string.error_required));
        valid = false;
    } else {
        double weight = Double.parseDouble(weightText);
        if (weight < 20 || weight > 300) {
            editWeight.setError(getString(R.string.error_weight_range));
            valid = false;
        }
    }

    if (heightText.isEmpty()) {
        editHeight.setError(getString(R.string.error_required));
        valid = false;
    }

    return valid;
}
```

- `setError("…")` – czerwony wykrzyknik i dymek przy polu. Znika sam, gdy użytkownik zacznie pisać. Odpowiednik `form.errors` wyświetlonego przy polu.
- `getString(R.string.error_required)` – tekst ze `strings.xml` jako `String`. W Activity masz `getString()` zawsze pod ręką.
- `Double.parseDouble` po sprawdzeniu `isEmpty()` i przy `inputType="numberDecimal"` jest bezpieczne – klawiatura nie pozwoli wpisać liter. Ale jeśli pole ma `inputType="text"`, opakuj w `try { } catch (NumberFormatException e) { }`.
- Sprawdzasz **wszystkie** pola, nie przerywasz na pierwszym – użytkownik ma zobaczyć wszystkie błędy naraz, nie po jednym.

Wzorzec w obsłudze przycisku:

```java
buttonCalc.setOnClickListener(v -> {
    if (!validate()) {
        return;
    }
    calculate();
});
```

Zasada z materiału 08 obowiązuje tutaj podwójnie: **walidacja po stronie klienta to wygoda, nie bezpieczeństwo**. Gdy w M06 apka zacznie wysyłać dane do Twojego API, serwer (Django) sprawdzi je drugi raz. Telefon można zrootować, aplikację zdekompilować, request podrobić. Serwerowi nie wolno ufać aplikacji.

---

## 6. Cykl życia Activity

Activity nie jest zwykłym obiektem, który tworzysz i masz. **System** go tworzy, pokazuje, chowa, niszczy – zależnie od tego, co robi użytkownik (telefon dzwoni, przełączył apkę, obrócił ekran, system zwolnił pamięć). Na każdy etap masz metodę do nadpisania:

```text
          uruchomienie
               |
               v
          onCreate()      <- raz; setContentView, findViewById, listenery
               |
               v
          onStart()       <- ekran staje się widoczny
               |
               v
          onResume()      <- ekran na wierzchu, użytkownik może klikać
               |
         [użytkownik działa]
               |
               v
          onPause()       <- coś przysłoniło ekran (dialog, inna apka, telefon)
               |
          +----+----+
          |         |
     wraca      idzie dalej
          |         |
       onResume   onStop()    <- ekran niewidoczny
                    |
               +----+----+
               |         |
           wraca     koniec
               |         |
          onRestart  onDestroy()  <- ekran zniszczony
             -> onStart
```

| Metoda | Kiedy | Co tam robisz |
|---|---|---|
| `onCreate` | tworzenie | wszystko z M01: layout, `findViewById`, listenery |
| `onStart` | staje się widoczna | rzadko używana |
| `onResume` | na wierzchu, interaktywna | wznów to, co kosztowne: GPS, aparat, animacje, odświeżanie danych |
| `onPause` | traci pierwszy plan | **zatrzymaj** GPS, aparat, zapisz szkic – masz na to ułamek sekundy |
| `onStop` | niewidoczna | zwolnij, co się da |
| `onDestroy` | zniszczona | sprzątanie; **nie licz, że zostanie wywołana** – system może ubić proces bez niej |

Dodaj logi i zobacz na własne oczy:

```java
private static final String TAG = "MainActivity";

@Override protected void onStart()   { super.onStart();   Log.d(TAG, "onStart"); }
@Override protected void onResume()  { super.onResume();  Log.d(TAG, "onResume"); }
@Override protected void onPause()   { super.onPause();   Log.d(TAG, "onPause"); }
@Override protected void onStop()    { super.onStop();    Log.d(TAG, "onStop"); }
@Override protected void onDestroy() { super.onDestroy(); Log.d(TAG, "onDestroy"); }
```

Uruchom, otwórz Logcat z filtrem `tag:MainActivity`. Naciśnij Home → `onPause`, `onStop`. Wróć → `onRestart`, `onStart`, `onResume`. **Obróć** → `onPause`, `onStop`, `onDestroy`, `onCreate`, `onStart`, `onResume`. Activity została zniszczona i zbudowana od nowa – z `clickCount = 0`. To jest odpowiedź na pytanie z M01.

Każde nadpisanie zaczyna od `super.onXxx()` – jak `onCreate`. Zapomnisz → `SuperNotCalledException`.

---

## 7. Zapisywanie stanu przy obrocie

System niszczy Activity przy obrocie, bo może chcieć wczytać inny layout (`res/layout-land/`). Ale zanim zniszczy, daje Ci szansę zapisać to, co ważne:

```java
private static final String KEY_COUNT = "click_count";

@Override
protected void onSaveInstanceState(@NonNull Bundle outState) {
    super.onSaveInstanceState(outState);
    outState.putInt(KEY_COUNT, clickCount);
}

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    // ... findViewById ...

    if (savedInstanceState != null) {
        clickCount = savedInstanceState.getInt(KEY_COUNT, 0);
        textGreeting.setText("Kliknięto " + clickCount + " razy");
    }
}
```

- `onSaveInstanceState` – wywoływane **przed** `onStop`, gdy system może zniszczyć Activity. Wkładasz do `Bundle` to, co chcesz odzyskać. `Bundle` to słownik klucz → wartość prosta (`int`, `String`, `boolean`, tablice, `Serializable`).
- `savedInstanceState` w `onCreate` – ten sam `Bundle`, jeśli Activity wraca po zniszczeniu. `null` przy pierwszym uruchomieniu. **Stąd ten parametr**, który w M01 kazałem ignorować.
- Klucz jako stała – literówka w stringu to `null` bez ostrzeżenia.

Co zapisuje się samo: zawartość `EditText`, stan `CheckBox`, pozycja przewinięcia – **ale tylko dla widoków z `android:id`**. Widok bez id nie zapisze stanu. Kolejny powód, żeby każdy interaktywny widget miał id.

Co się nie zapisze: Twoje pola klasy (`clickCount`), wyniki obliczeń, dane pobrane z sieci. Te zapisujesz sam. Dla większych danych (lista z API) jest `ViewModel` – poznasz w M06.

> ⚠️ Kusi, żeby zablokować obrót w manifeście (`android:screenOrientation="portrait"`) i mieć spokój. Nie rób tego. Activity jest niszczona też, gdy system potrzebuje pamięci, gdy użytkownik zmieni język, czcionkę, tryb ciemny. Obrót to tylko najłatwiejszy sposób, żeby to **przetestować**.

---

## 8. Aplikacja: kalkulator BMI

Wszystko z tego materiału w jednym ekranie. Nowy projekt `BmiApp` (Empty Views Activity, Java, API 26) albo kolejna Activity w `HelloApp` – jak wolisz.

`res/values/strings.xml`:

```xml
<resources>
    <string name="app_name">BMI</string>
    <string name="title">Kalkulator BMI</string>
    <string name="hint_weight">Waga (kg)</string>
    <string name="hint_height">Wzrost (cm)</string>
    <string name="label_unit">Jednostki metryczne</string>
    <string name="button_calc">Oblicz</string>
    <string name="button_clear">Wyczyść</string>
    <string name="result_format">BMI: %.1f – %s</string>
    <string name="error_required">Pole wymagane</string>
    <string name="error_weight_range">Waga 20–300 kg</string>
    <string name="error_height_range">Wzrost 50–250 cm</string>
    <string name="bmi_under">niedowaga</string>
    <string name="bmi_normal">waga prawidłowa</string>
    <string name="bmi_over">nadwaga</string>
    <string name="bmi_obese">otyłość</string>
</resources>
```

`result_format` ma `%.1f` i `%s` – to `String.format` zaszyty w zasobie: `getString(R.string.result_format, bmi, label)`.

`res/layout/activity_main.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="24dp">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@string/title"
        android:textSize="24sp"
        android:textStyle="bold" />

    <EditText
        android:id="@+id/edit_weight"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:hint="@string/hint_weight"
        android:inputType="numberDecimal"
        android:autofillHints="" />

    <EditText
        android:id="@+id/edit_height"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="8dp"
        android:hint="@string/hint_height"
        android:inputType="numberDecimal"
        android:autofillHints="" />

    <CheckBox
        android:id="@+id/check_metric"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="8dp"
        android:checked="true"
        android:text="@string/label_unit" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:orientation="horizontal">

        <Button
            android:id="@+id/button_calc"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="@string/button_calc" />

        <Button
            android:id="@+id/button_clear"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_marginStart="8dp"
            android:layout_weight="1"
            android:text="@string/button_clear" />
    </LinearLayout>

    <TextView
        android:id="@+id/text_result"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="24dp"
        android:textSize="20sp"
        android:visibility="gone" />

</LinearLayout>
```

`MainActivity.java`:

```java
package pl.edu._4tp.bmiapp;

import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.CheckBox;
import android.widget.EditText;
import android.widget.TextView;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity {

    private static final String KEY_RESULT = "result";

    private EditText editWeight;
    private EditText editHeight;
    private CheckBox checkMetric;
    private TextView textResult;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        editWeight = findViewById(R.id.edit_weight);
        editHeight = findViewById(R.id.edit_height);
        checkMetric = findViewById(R.id.check_metric);
        textResult = findViewById(R.id.text_result);
        Button buttonCalc = findViewById(R.id.button_calc);
        Button buttonClear = findViewById(R.id.button_clear);

        buttonCalc.setOnClickListener(v -> {
            if (validate()) {
                calculate();
            }
        });
        buttonClear.setOnClickListener(v -> clear());

        if (savedInstanceState != null) {
            String result = savedInstanceState.getString(KEY_RESULT);
            if (result != null) {
                textResult.setText(result);
                textResult.setVisibility(View.VISIBLE);
            }
        }
    }

    @Override
    protected void onSaveInstanceState(@NonNull Bundle outState) {
        super.onSaveInstanceState(outState);
        if (textResult.getVisibility() == View.VISIBLE) {
            outState.putString(KEY_RESULT, textResult.getText().toString());
        }
    }

    private boolean validate() {
        boolean valid = true;

        String weightText = editWeight.getText().toString().trim();
        if (weightText.isEmpty()) {
            editWeight.setError(getString(R.string.error_required));
            valid = false;
        } else {
            double weight = Double.parseDouble(weightText);
            if (weight < 20 || weight > 300) {
                editWeight.setError(getString(R.string.error_weight_range));
                valid = false;
            }
        }

        String heightText = editHeight.getText().toString().trim();
        if (heightText.isEmpty()) {
            editHeight.setError(getString(R.string.error_required));
            valid = false;
        } else {
            double height = Double.parseDouble(heightText);
            if (height < 50 || height > 250) {
                editHeight.setError(getString(R.string.error_height_range));
                valid = false;
            }
        }

        return valid;
    }

    private void calculate() {
        double weight = Double.parseDouble(editWeight.getText().toString().trim());
        double heightCm = Double.parseDouble(editHeight.getText().toString().trim());

        if (!checkMetric.isChecked()) {
            weight = weight * 0.4536;     // funty -> kg
            heightCm = heightCm * 2.54;   // cale -> cm
        }

        double heightM = heightCm / 100;
        double bmi = weight / (heightM * heightM);

        textResult.setText(getString(R.string.result_format, bmi, categoryFor(bmi)));
        textResult.setVisibility(View.VISIBLE);
    }

    private String categoryFor(double bmi) {
        if (bmi < 18.5) return getString(R.string.bmi_under);
        if (bmi < 25) return getString(R.string.bmi_normal);
        if (bmi < 30) return getString(R.string.bmi_over);
        return getString(R.string.bmi_obese);
    }

    private void clear() {
        editWeight.setText("");
        editHeight.setText("");
        editWeight.setError(null);
        editHeight.setError(null);
        textResult.setVisibility(View.GONE);
        editWeight.requestFocus();
    }
}
```

Co tu jest z tego materiału: LinearLayout zagnieżdżony z `weight`, cztery typy widgetów, dwa listenery, walidacja z `setError`, `getString` z formatem, `visibility`, `onSaveInstanceState` dla wyniku. Pola `EditText` i `CheckBox` zapisują się same, bo mają id – sprawdź obrotem.

Zauważ podział: `validate()`, `calculate()`, `categoryFor()`, `clear()` – osobne metody. `onCreate` tylko podpina. To samo, co „cienki kontroler” z MVC. W M04 wyciągniemy obliczenia do osobnej klasy, żeby dało się je przetestować bez telefonu.

---

## 9. Najczęstsze problemy

| Objaw | Przyczyna i rozwiązanie |
|---|---|
| wszystko w jednym wierszu, wylatuje poza ekran | LinearLayout bez `orientation="vertical"` |
| `layout_weight` nie działa | `layout_width` musi być `0dp` (w poziomym) lub `layout_height` `0dp` (w pionowym) |
| `NumberFormatException` przy `parseDouble` | puste pole albo `inputType="text"` – sprawdź `isEmpty()`, ustaw `numberDecimal`, albo `try/catch` |
| `getText()` – `cannot be converted to String` | brak `.toString()` |
| `setError` nie znika | znika po zmianie tekstu; przy czyszczeniu ustaw `setError(null)` |
| `switch` na `R.id.x` – `constant expression required` | id nie są stałymi; użyj `if/else` |
| `SuperNotCalledException` | nadpisałeś `onPause`/`onStop` bez `super.onXxx()` |
| stan `EditText` nie wraca po obrocie | pole nie ma `android:id` |
| licznik/wynik znika po obrocie | pole klasy – zapisz w `onSaveInstanceState`, odczytaj w `onCreate` |
| `savedInstanceState` zawsze `null` | pierwsze uruchomienie – tak ma być. Obróć |
| obrazek nie pojawia się, `R.drawable.x` nie istnieje | nazwa pliku z wielką literą, myślnikiem lub polskim znakiem |
| lambda podkreślona: `incompatible types` | listener ma więcej niż jedną metodę (TextWatcher, SeekBar) – klasa anonimowa |
| klawiatura nie chowa się po obliczeniu | to normalne; wymuszenie: `InputMethodManager` – na razie pomiń |

---

## 10. Zadania

### Zadanie 1: LinearLayout

Przerób layout z M01 na `LinearLayout vertical`. Dodaj poziomy LinearLayout z dwoma przyciskami po połowie szerokości (`weight`). Sprawdź w *Design*, potem zmień `orientation` na `horizontal` i zobacz, co się dzieje.

### Zadanie 2: Przegląd widgetów

Ekran „katalog”: po jednym `TextView`, `EditText`, `Button`, `CheckBox`, `Switch`, `RadioGroup` z trzema `RadioButton`, `SeekBar`, `ImageView` z własnym obrazkiem w `drawable`. Przycisk zbiera stan wszystkich i wyświetla w `TextView` (np. „Imię: Ania, zgoda: tak, rozmiar: M, głośność: 70”).

### Zadanie 3: Cykl życia

Nadpisz wszystkie sześć metod z logiem. Przetestuj: Home i powrót, obrót, przełączenie na inną aplikację, zablokowanie ekranu. Do README: tabela „akcja → kolejność wywołań”.

### Zadanie 4: BMI

Zbuduj kalkulator z rozdziału 8. Uruchom, przetestuj: puste pola, wartości spoza zakresu, obrót po obliczeniu (wynik ma zostać), „Wyczyść” po błędzie (wykrzykniki mają zniknąć).

### Zadanie 5: Walidacja na żywo

`TextWatcher` na polu wagi: gdy wartość wychodzi poza 20–300, `setError` natychmiast, bez klikania „Oblicz”. Gdy wraca do zakresu – `setError(null)`.

### Zadanie 6: Twoja domena

Ekran z Twojej domeny z formularzem: minimum 4 pola różnych typów, walidacja każdego, przycisk „Zapisz” pokazujący Toast z podsumowaniem, „Wyczyść”. Stan po obrocie ma przetrwać. Teksty w `strings.xml`.

### Zadanie 7: Jednostki

W BMI: zamiast `CheckBox` – `RadioGroup` „kg/cm” vs „lb/in”. Zmiana wyboru ma zmieniać `hint` obu pól (`setHint`). Wynik przeliczany poprawnie dla obu.

### Zadanie 8: Repo

Commity: `feat: add BMI calculator layout`, `feat: add validation and calculation`, `feat: preserve result on rotation`. README: zrzut ekranu z błędem walidacji i tabela cyklu życia z zadania 3.

---

## 11. Pytania kontrolne

Sprawdź, czy opanowałeś materiał. Odpowiedz sobie sam, bez zaglądania.

1. LinearLayout vs ConstraintLayout – kiedy który?
2. Co się stanie z LinearLayout bez `orientation`?
3. `padding` vs `margin` – różnica? `gravity` vs `layout_gravity`?
4. Jak podzielić ekran na dwa przyciski po połowie? Jakie dwa atrybuty są potrzebne?
5. Trzy stany `visibility` – czym różni się `INVISIBLE` od `GONE`?
6. Po co `inputType` w `EditText`? Podaj trzy wartości i co dają.
7. `editText.getText()` – jaki typ zwraca? Jak dostać `String`? Jak `double`?
8. Trzy sposoby obsługi kliknięcia – który używasz i dlaczego nie `android:onClick`?
9. Dlaczego `switch (v.getId())` nie działa? Co zamiast?
10. Kiedy lambda, a kiedy klasa anonimowa przy listenerach?
11. Jaki wzorzec projektowy realizuje listener? (Nazwa z podstawy programowej.)
12. `setError` – co robi, kiedy znika, jak usunąć ręcznie?
13. Dlaczego walidacja w aplikacji nie zwalnia serwera z walidacji?
14. Wymień metody cyklu życia w kolejności od uruchomienia do zamknięcia.
15. Co robisz w `onPause`, a co w `onResume`? Podaj przykład zasobu, który trzeba zatrzymać.
16. Obrót telefonu: jakie metody lecą i w jakiej kolejności? Dlaczego system w ogóle niszczy Activity?
17. `onSaveInstanceState` – kiedy jest wywoływane, co do niego wkładasz, skąd to potem odczytujesz?
18. Co zapisuje się samo przy obrocie, a co nie? Jaki warunek musi spełnić widget, żeby zapisał się sam?
19. Dlaczego nie blokować orientacji w manifeście „dla świętego spokoju”?
20. Otwórz swój kalkulator: wskaż, gdzie jest walidacja, gdzie obliczenie, gdzie zapis stanu. Co by się stało, gdybyś usunął `super.onSaveInstanceState(outState)`?
