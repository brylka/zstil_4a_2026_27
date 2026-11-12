# Android – start (Java)

**Programowanie aplikacji mobilnych | klasa 4 | materiał M01**

Ścieżka mobilna idzie **równolegle** do webowej (materiały 10–20). Tu zaczynamy od zera: Android Studio, pierwszy projekt, co jest w tych wszystkich plikach, uruchomienie na emulatorze i na telefonie. Język: **Java** – tak jak w podstawie INF.04 i tak, jak będziemy pisać przez cały rok.

---

## Spis treści

1. Czym jest aplikacja na Androida
2. Instalacja Android Studio
3. Pierwszy projekt
4. Co jest w tych plikach
5. Hello World – i co robi każda linia
6. Emulator i prawdziwy telefon
7. Pierwsza zmiana: tekst i przycisk
8. Logcat – gdzie patrzeć, gdy nie działa
9. Najczęstsze problemy
10. Do repozytorium
11. Zadania
12. Pytania kontrolne

---

## 1. Czym jest aplikacja na Androida

Strona WWW działa w przeglądarce: serwer wysyła HTML, przeglądarka rysuje. Aplikacja mobilna działa **bezpośrednio na telefonie**: Twój kod jest skompilowany do pliku `.apk`, zainstalowany, i to on rysuje ekran, reaguje na dotyk, ma dostęp do aparatu, GPS, pamięci.

Co to zmienia w porównaniu z tym, co robisz w Django:

| | Django (web) | Android |
|---|---|---|
| gdzie działa kod | na serwerze | na telefonie użytkownika |
| co rysuje ekran | przeglądarka z HTML | Twoja aplikacja z XML + Java |
| jak dostaje dane | z własnej bazy | z API przez internet (to spięcie zrobimy w M06) |
| aktualizacja | wgrywasz na serwer, wszyscy mają | użytkownik musi pobrać nową wersję |
| język | Python | Java (lub Kotlin) |
| „kontroler” | funkcja w `views.py` | klasa `Activity` |
| „szablon” | `templates/*.html` | `res/layout/*.xml` |

Ostatnie dwa wiersze to ten sam podział, który znasz z MVC: wygląd osobno (XML), logika osobno (Java). Android nie nazywa tego MVC, ale struktura jest ta sama.

Technicznie: aplikacja to zbiór **ekranów** (`Activity`), każdy z własnym **układem** (`layout` w XML) i własną klasą Java, która ten układ wczytuje i obsługuje zdarzenia. Na początek jeden ekran.

> 💡 Android powstał w 2003 jako system dla aparatów cyfrowych. Google kupił firmę w 2005 za ok. 50 mln dolarów. Dziś to ponad 70% telefonów na świecie. Java była jedynym oficjalnym językiem do 2017, Kotlin dołączył i jest dziś preferowany przez Google – ale Java działa, jest w podstawie programowej i wszystko, co znajdziesz w dokumentacji, ma wersję w obu językach.

---

## 2. Instalacja Android Studio

### 2.1 Wymagania

- Windows 10/11 64-bit,
- **minimum 8 GB RAM**, lepiej 16 – emulator jest żarłoczny,
- **minimum 20 GB wolnego miejsca** – samo Studio to 3 GB, SDK kolejne kilka, każdy obraz emulatora 2–8 GB,
- włączona wirtualizacja w BIOS (VT-x / AMD-V) – bez tego emulator nie wystartuje albo będzie pełzał.

Na komputerze szkolnym sprawdź wolne miejsce **zanim** zaczniesz pobierać.

### 2.2 Pobranie

https://developer.android.com/studio → *Download Android Studio*. Aktualna wersja (listopad 2026) nazywa się **Rabbit 2** (`2026.2.2`). Android Studio ma nazwy zwierząt w kolejności alfabetycznej, jak Ubuntu – nie przejmuj się, jeśli u Ciebie jest nowsze.

Instalator: *Next*, *Next*, zostaw domyślne. Przy pierwszym uruchomieniu kreator (*Setup Wizard*): wybierz *Standard*, zaakceptuj licencje, poczekaj – pobierze SDK (zestaw narzędzi i bibliotek dla konkretnej wersji Androida). To może potrwać 10–20 minut.

### 2.3 Co się zainstalowało

| Rzecz | Co to | Gdzie |
|---|---|---|
| Android Studio | IDE (edytor + narzędzia), oparty na IntelliJ, tym samym co PyCharm | `C:\Program Files\Android\Android Studio` |
| Android SDK | biblioteki Androida, narzędzia (`adb`, emulator, kompilator) | `C:\Users\NAZWA\AppData\Local\Android\Sdk` |
| JDK | Java – Studio ma własną w środku, nie musisz instalować osobno | w folderze Studio |
| Gradle | system budowania – zamienia Twoje pliki w `.apk`. Pobiera się przy pierwszym projekcie | `C:\Users\NAZWA\.gradle` |

Folder `.gradle` i `Sdk` urosną do kilkunastu GB. To normalne.

> ⚠️ Komputery w pracowni są wspólne. Android Studio nie trzyma Twojego logowania do GitHuba – ale Twój projekt zostaje na dysku. Projekty trzymaj w repozytorium, nie na pulpicie, i pamiętaj o procedurze z materiału 01.

---

## 3. Pierwszy projekt

*File → New → New Project*. Zobaczysz galerię szablonów. **To jest pierwsza pułapka**: domyślnie podświetlony jest *Empty Activity*, który generuje projekt w **Kotlinie z Jetpack Compose**. Nie tego chcemy.

Wybierz **Empty Views Activity** (ikona z telefonem i napisem „Views”). Potem:

| Pole | Wpisz | Dlaczego |
|---|---|---|
| Name | `HelloApp` | nazwa aplikacji, widoczna pod ikoną |
| Package name | `pl.edu.4tp.helloapp` – albo `com.twojenazwisko.helloapp` | unikalny identyfikator aplikacji w systemie i w sklepie. Konwencja: odwrócona domena. **Małe litery, bez myślników** |
| Save location | folder w Twoich repozytoriach | nie pulpit |
| Language | **Java** | tu drugi raz można się pomylić – domyślnie jest Kotlin |
| Minimum SDK | **API 26** (Android 8.0) | najstarszy Android, na którym apka zadziała. 26 pokrywa ~98% telefonów i daje nowoczesne API |
| Build configuration language | Kotlin DSL (domyślne) | to tylko format plików konfiguracji Gradle, nie język aplikacji. Zostaw |

*Finish*. Studio założy projekt i **zacznie go budować** – pasek postępu na dole. Pierwszy raz: 2–5 minut, bo Gradle pobiera zależności. Poczekaj, aż na dole zobaczysz *BUILD SUCCESSFUL* albo zniknie pasek.

---

## 4. Co jest w tych plikach

Po lewej panel *Project*. Przełącz widok z *Android* na **Project** (lista rozwijana u góry panelu) – zobaczysz prawdziwą strukturę folderów, nie uproszczoną. Potem wróć do *Android*, bo w nim pracuje się wygodniej.

Z całego drzewa interesują Cię na razie cztery pliki:

```text
HelloApp/
├── app/
│   ├── build.gradle.kts                      <- konfiguracja aplikacji (wersje, zależności)
│   └── src/main/
│       ├── AndroidManifest.xml               <- „dowód osobisty” aplikacji
│       ├── java/pl/edu/_4tp/helloapp/
│       │   └── MainActivity.java             <- logika pierwszego ekranu
│       └── res/
│           ├── layout/activity_main.xml      <- wygląd pierwszego ekranu
│           ├── values/strings.xml            <- teksty
│           ├── values/colors.xml             <- kolory
│           └── mipmap-*/ic_launcher.png      <- ikona w różnych rozmiarach
├── build.gradle.kts                          <- konfiguracja całego projektu
├── settings.gradle.kts
└── gradle/libs.versions.toml                 <- wersje bibliotek w jednym miejscu
```

| Plik | Odpowiednik w Django | Co robi |
|---|---|---|
| `MainActivity.java` | `views.py` | klasa ekranu: wczytuje layout, obsługuje kliknięcia |
| `activity_main.xml` | `templates/index.html` | wygląd: co, gdzie, jakiego rozmiaru |
| `AndroidManifest.xml` | `settings.py` + `urls.py` | nazwa, ikona, lista ekranów, uprawnienia (internet, aparat), który ekran startowy |
| `strings.xml` | – | wszystkie teksty aplikacji. Tłumaczenie = drugi `strings.xml` w `values-en/` |
| `build.gradle.kts` (app) | `requirements.txt` | zależności, `minSdk`, `targetSdk`, wersja aplikacji |
| `res/` | `static/` | zasoby: obrazki, kolory, wymiary, layouty. **Każdy plik w `res/` dostaje identyfikator w klasie `R`** – o tym za chwilę |

Pakiet `pl.edu._4tp` – Java nie pozwala na nazwę zaczynającą się cyfrą, więc Studio dodało podkreślnik. Jeśli użyłeś `com.nazwisko.helloapp`, nie ma tego problemu.

---

## 5. Hello World – i co robi każda linia

Szablon już jest Hello World. Otwórz oba pliki.

### 5.1 `res/layout/activity_main.xml`

Na dole edytora przełącz z *Design* na **Code** (albo *Split*, żeby widzieć oba).

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

| Element | Co robi |
|---|---|
| `<ConstraintLayout>` | kontener – pojemnik na inne elementy. Rozmieszcza je przez „ograniczenia” (constraints): „lewa krawędź tego przy lewej krawędzi rodzica” itd. |
| `xmlns:android`, `xmlns:app`, `xmlns:tools` | przestrzenie nazw atrybutów. `android:` – wbudowane, `app:` – z bibliotek (ConstraintLayout), `tools:` – tylko dla podglądu w Studio, nie trafia do aplikacji |
| `android:id="@+id/main"` | identyfikator. `@+id/` = „utwórz nowy id o nazwie `main`”. Przez ten id Java znajduje element |
| `layout_width` / `layout_height` | **każdy** element musi mieć oba. `match_parent` – tyle, ile rodzic. `wrap_content` – tyle, ile treść. Albo konkretna wartość w `dp` |
| `<TextView>` | napis. Odpowiednik `<p>` |
| `android:text="Hello World!"` | treść. Studio podkreśli na żółto: „hardcoded string” – teksty mają być w `strings.xml`. Poprawimy w rozdziale 7 |
| `app:layout_constraint…="parent"` | cztery ograniczenia: góra, dół, lewo, prawo do rodzica → element wyśrodkowany |

### 5.2 `MainActivity.java`

```java
package pl.edu._4tp.helloapp;

import android.os.Bundle;

import androidx.activity.EdgeToEdge;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.graphics.Insets;
import androidx.core.view.ViewCompat;
import androidx.core.view.WindowInsetsCompat;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        EdgeToEdge.enable(this);
        setContentView(R.layout.activity_main);
        ViewCompat.setOnApplyWindowInsetsListener(findViewById(R.id.main), (v, insets) -> {
            Insets systemBars = insets.getInsets(WindowInsetsCompat.Type.systemBars());
            v.setPadding(systemBars.left, systemBars.top, systemBars.right, systemBars.bottom);
            return insets;
        });
    }
}
```

| Linia | Co robi |
|---|---|
| `package pl.edu._4tp.helloapp;` | pakiet – jak w każdej Javie |
| `public class MainActivity extends AppCompatActivity` | ekran = klasa dziedzicząca po `AppCompatActivity`. Dziedziczenie z programowania obiektowego: dostajesz cały cykl życia ekranu, dopisujesz tylko to, co Twoje |
| `@Override protected void onCreate(Bundle savedInstanceState)` | metoda wywoływana przez system, gdy ekran **powstaje**. Tu robisz całą inicjalizację. `Bundle` – zapisany stan, jeśli ekran wraca po obrocie telefonu (na razie ignoruj) |
| `super.onCreate(savedInstanceState);` | **zawsze pierwsza linia**. Rodzic musi zrobić swoje, zanim Ty zaczniesz |
| `EdgeToEdge.enable(this);` | aplikacja rysuje się pod paskiem statusu i paskiem nawigacji (nowy standard od Androida 15) |
| `setContentView(R.layout.activity_main);` | **najważniejsza linia**: „wczytaj ten layout XML i pokaż go”. Łączy Javę z XML-em. Odpowiednik `render(request, "index.html")` |
| `R.layout.activity_main` | klasa `R` jest **generowana automatycznie** z folderu `res/`. Każdy plik i każdy `@+id/` dostaje stałą: `R.layout.activity_main`, `R.id.main`, `R.string.app_name`. Nie edytujesz jej, tylko używasz |
| `ViewCompat.setOnApplyWindowInsetsListener(...)` | skoro rysujemy pod paskami systemowymi, dodaj padding, żeby treść nie wjechała pod zegar. Szablon to generuje – zostaw |
| `findViewById(R.id.main)` | znajdź w layoucie element o id `main`. Tak Java dostaje się do elementów z XML-a. Użyjesz tego setki razy |

Dwa pliki, jedno zdanie: **Activity wczytuje layout przez `setContentView`, a do elementów dociera przez `findViewById`.** Reszta tego materiału i następnych to warianty tego zdania.

### 5.3 `AndroidManifest.xml`

```xml
<application
    android:icon="@mipmap/ic_launcher"
    android:label="@string/app_name"
    android:theme="@style/Theme.HelloApp">
    <activity
        android:name=".MainActivity"
        android:exported="true">
        <intent-filter>
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.LAUNCHER" />
        </intent-filter>
    </activity>
</application>
```

- `android:label="@string/app_name"` – nazwa z `strings.xml`. Składnia `@typ/nazwa` to odwołanie do zasobu – tak samo jak `R.string.app_name` w Javie.
- `<activity android:name=".MainActivity">` – **każdy ekran musi być tu zgłoszony**. Dodasz drugi ekran bez wpisu → aplikacja się wywali. Studio dopisuje wpis automatycznie, gdy tworzysz Activity przez kreator, ale ręcznie – pamiętaj.
- `<intent-filter>` z `MAIN` + `LAUNCHER` – „ten ekran ma ikonę w menu i od niego startujemy”. Dokładnie jeden ekran powinien go mieć.

---

## 6. Emulator i prawdziwy telefon

### 6.1 Emulator

Pasek narzędzi → ikona telefonu (*Device Manager*) → *Create Virtual Device* (albo `+`). Wybierz **Pixel 8** (albo dowolny *Pixel* bez *Fold*/*Tablet*), *Next*. Obraz systemu: najnowszy z napisem *Recommended*, np. **Android 17 (API 37)** – pobierze się 1–2 GB. *Next*, *Finish*.

Na pasku u góry, obok zielonego trójkąta, pojawi się nazwa emulatora. Kliknij ▶ (*Run*, `Shift + F10`). Pierwsze uruchomienie emulatora: 1–3 minuty. Kolejne – kilkanaście sekund, bo Studio zostawia go włączonego.

Zobaczysz telefon z Twoją aplikacją i napisem „Hello World!”.

> 💡 Emulator możesz obracać (`Ctrl + ←`/`→`), symulować GPS, dzwonić, wysyłać SMS – wszystko w panelu z prawej (trzy kropki). Przyda się przy lokalizacji w M05.

### 6.2 Prawdziwy telefon

Emulator jest wolny i zjada pamięć. Jeśli masz Androida w kieszeni, użyj go:

1. Telefon: *Ustawienia → Informacje o telefonie* → stuknij **7 razy** w *Numer kompilacji* → „Jesteś programistą”.
2. *Ustawienia → Opcje programisty* → włącz **Debugowanie USB**.
3. Podłącz kablem. Na telefonie: *Zezwolić na debugowanie USB?* → *Zawsze zezwalaj z tego komputera* → OK.
4. W Studio telefon pojawi się na liście obok ▶. Wybierz, uruchom.

Aplikacja instaluje się i odpala na telefonie w kilka sekund. Zostaje tam po odłączeniu – możesz pokazać w domu.

Jeśli nie widzi telefonu: zmień kabel (wiele kabli jest „tylko do ładowania”), w telefonie przełącz tryb USB na *Przesyłanie plików*, albo doinstaluj sterownik producenta (Samsung, Xiaomi mają własne).

> ⚠️ Nie zostawiaj debugowania USB włączonego na stałe – wyłącz po zajęciach. Komputer szkolny, do którego dałeś „zawsze zezwalaj”, ma pełny dostęp do telefonu.

---

## 7. Pierwsza zmiana: tekst i przycisk

### 7.1 Tekst do `strings.xml`

`res/values/strings.xml`:

```xml
<resources>
    <string name="app_name">HelloApp</string>
    <string name="greeting">Cześć, 4TP!</string>
    <string name="button_click">Kliknij mnie</string>
    <string name="clicked">Kliknięto!</string>
</resources>
```

W `activity_main.xml` zamień `android:text="Hello World!"` na `android:text="@string/greeting"`. Żółte podkreślenie znika. Każdy tekst, który użytkownik zobaczy, ma być tutaj. Powód ten sam, co angielskie nazwy w kodzie z materiału 08: tłumaczenie, jeden punkt zmiany, i Studio potrafi to sprawdzić.

### 7.2 Przycisk w layoucie

Dodaj `android:id` do `TextView` i wstaw `Button` pod nim. Cały layout:

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:id="@+id/text_greeting"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/greeting"
        android:textSize="24sp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <Button
        android:id="@+id/button_click"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:text="@string/button_click"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/text_greeting" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

- `android:id="@+id/text_greeting"` – nazwy id: `typ_opis`, snake_case. `text_greeting`, `button_click`, `edit_name`. Po angielsku.
- `app:layout_constraintTop_toBottomOf="@id/text_greeting"` – „góra przycisku do dołu napisu”. `@id/` bez plusa, bo id już istnieje.
- `android:layout_marginTop="16dp"` – odstęp. **`dp`** (density-independent pixels) do rozmiarów i odstępów, **`sp`** (scalable pixels) do tekstu – skaluje się z ustawieniem rozmiaru czcionki w telefonie. Nigdy `px`.

Przełącz na *Design* – zobaczysz podgląd. Przeciągnij przycisk myszą; zobacz, jak zmieniają się atrybuty w *Code*. Edytor graficzny generuje ten sam XML – ale XML musisz umieć czytać, bo na kartkówce nie będzie myszy.

### 7.3 Obsługa kliknięcia w Javie

```java
package pl.edu._4tp.helloapp;

import android.os.Bundle;
import android.widget.Button;
import android.widget.TextView;
import android.widget.Toast;

import androidx.activity.EdgeToEdge;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.graphics.Insets;
import androidx.core.view.ViewCompat;
import androidx.core.view.WindowInsetsCompat;

public class MainActivity extends AppCompatActivity {

    private TextView textGreeting;
    private Button buttonClick;
    private int clickCount = 0;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        EdgeToEdge.enable(this);
        setContentView(R.layout.activity_main);
        ViewCompat.setOnApplyWindowInsetsListener(findViewById(R.id.main), (v, insets) -> {
            Insets systemBars = insets.getInsets(WindowInsetsCompat.Type.systemBars());
            v.setPadding(systemBars.left, systemBars.top, systemBars.right, systemBars.bottom);
            return insets;
        });

        textGreeting = findViewById(R.id.text_greeting);
        buttonClick = findViewById(R.id.button_click);

        buttonClick.setOnClickListener(v -> {
            clickCount++;
            textGreeting.setText("Kliknięto " + clickCount + " razy");
            Toast.makeText(this, R.string.clicked, Toast.LENGTH_SHORT).show();
        });
    }
}
```

| Linia | Co robi |
|---|---|
| `private TextView textGreeting;` | pole klasy na element z layoutu. Nazwa jak id, tylko camelCase – konwencja Javy |
| `textGreeting = findViewById(R.id.text_greeting);` | połączenie: zmienna Java ↔ element XML. **Po** `setContentView`, inaczej `null` |
| `buttonClick.setOnClickListener(v -> { ... })` | „gdy klikną, zrób to”. Lambda – jak `addEventListener` w JS. `v` to kliknięty widok, zwykle nieużywane |
| `clickCount++` | zwykłe pole klasy. Activity to obiekt – ma stan |
| `textGreeting.setText(...)` | zmiana tekstu z kodu. Każdy atrybut XML ma setter: `setText`, `setTextSize`, `setVisibility` |
| `Toast.makeText(this, R.string.clicked, Toast.LENGTH_SHORT).show()` | **Toast** – mały dymek na dole ekranu, znika sam. `this` = kontekst (Activity). Bez `.show()` nic się nie pokaże – klasyczny błąd |

Uruchom (▶). Kliknij przycisk. Tekst się zmienia, dymek wyskakuje.

Obróć emulator (`Ctrl + ←`). Licznik wraca do zera. Dlaczego? Bo obrót **niszczy i tworzy Activity od nowa** – `onCreate` leci drugi raz, `clickCount = 0`. To jest cykl życia Activity, materiał M02. Na razie zapamiętaj objaw.

---

## 8. Logcat – gdzie patrzeć, gdy nie działa

Na dole Studio zakładka **Logcat**. To jest terminal z Flaska i Django w jednym – wszystko, co aplikacja i system wypisują. Aplikacja się wywaliła? Tu jest traceback (po javowemu: *stack trace*).

Filtr u góry: wpisz `package:mine` – zobaczysz tylko swoją aplikację. Poziomy: *Verbose* (wszystko), *Debug*, *Info*, *Warn*, *Error*. Przy awarii szukaj czerwonych linii z `FATAL EXCEPTION`.

Własne logi z Javy – odpowiednik `print()`:

```java
import android.util.Log;

private static final String TAG = "MainActivity";

Log.d(TAG, "Kliknięto, licznik = " + clickCount);
```

`Log.d` (debug), `Log.i` (info), `Log.w` (warn), `Log.e` (error). `TAG` – żeby dało się filtrować: w Logcat wpisz `tag:MainActivity`.

Zrób celowo błąd: w `onCreate` przestaw `findViewById` **przed** `setContentView`. Uruchom, kliknij przycisk. Aplikacja się zamknie. W Logcat: `NullPointerException ... buttonClick is null`. Przeczytaj cały stack trace – pierwsza linia z nazwą Twojej klasy mówi, która linia zawiniła. Cofnij zmianę.

---

## 9. Najczęstsze problemy

| Objaw | Przyczyna i rozwiązanie |
|---|---|
| projekt w Kotlinie, pliki `.kt` | wybrałeś *Empty Activity* zamiast *Empty Views Activity*, albo Language: Kotlin. Załóż od nowa |
| `Gradle sync failed` przy pierwszym projekcie | brak internetu albo proxy szkolne. Gradle musi pobrać zależności. Spróbuj *File → Sync Project with Gradle Files* po podłączeniu |
| `BUILD FAILED` z `Could not resolve ...` | jak wyżej – sieć |
| budowanie trwa 10 minut | pierwszy raz: normalne. Kolejne: 10–30 s. Jeśli zawsze długo: za mało RAM, zamknij Chrome |
| emulator nie startuje / `HAXM`/`Hypervisor` error | wirtualizacja wyłączona w BIOS. Na komputerze szkolnym – zgłoś, sam nie włączysz. Użyj telefonu |
| emulator czarny ekran przez 3 minuty | czeka. Pierwsze uruchomienie obrazu jest wolne |
| `R` na czerwono, `cannot resolve symbol R` | błąd w którymś pliku XML – `R` nie wygenerowało się. Szukaj czerwonego w `res/`. Potem *Build → Clean Project* |
| `NullPointerException` przy `findViewById` | wywołane przed `setContentView`, albo złe id, albo element jest w innym layoucie |
| Toast nie pokazuje się | brak `.show()` na końcu |
| tekst podkreślony na żółto | hardcoded string – przenieś do `strings.xml` |
| element w podglądzie leci w lewy górny róg | brak constraintów. Każdy element w ConstraintLayout potrzebuje co najmniej jednego poziomego i jednego pionowego |
| telefon nie widnieje na liście | kabel „tylko ładowanie”, debugowanie USB wyłączone, nie zaakceptowałeś dialogu na telefonie |
| `Default Activity not found` | brak `<intent-filter>` z `MAIN`/`LAUNCHER` w manifeście |

---

## 10. Do repozytorium

Studio generuje `.gitignore` samo. Sprawdź, że są w nim:

```text
*.iml
.gradle/
/local.properties
/.idea/
/build/
/app/build/
.DS_Store
/captures
.externalNativeBuild
.cxx
```

`local.properties` ma ścieżkę do SDK na **Twoim** komputerze – u kolegi jest inna, dlatego nie idzie do repo. `build/` to wynik kompilacji, setki MB – nie idzie.

Studio ma wbudowanego Gita (*Git → Commit*, `Ctrl + K`), ale terminal jest na dole (*Terminal*) – komendy z materiału 01 działają tak samo.

```bash
git init
git add .
git commit -m "chore: init Android project (Java, API 26)"
git branch -M main
git remote add origin https://github.com/TWOJ_LOGIN/hello-app.git
git push -u origin main
```

README według szablonu z materiału 09. W sekcji *Technologie*: Java, minSdk, wersja Android Studio. W *Uruchomienie*: „otwórz w Android Studio, poczekaj na Gradle sync, ▶”.

---

## 11. Zadania

### Zadanie 1: Środowisko

Zainstaluj Android Studio, przejdź kreator, załóż projekt `HelloApp` według rozdziału 3. Pokaż nauczycielowi: *BUILD SUCCESSFUL* na dole, `MainActivity.java` (nie `.kt`), minSdk 26 w `build.gradle.kts`.

### Zadanie 2: Emulator lub telefon

Uruchom aplikację. Zrzut ekranu z „Hello World!” do README. Jeśli masz telefon – uruchom też na nim i dopisz w README model telefonu i wersję Androida.

### Zadanie 3: Mapa plików

W README tabela: plik → za co odpowiada → odpowiednik w Django. Minimum 6 wierszy. Bez zaglądania do rozdziału 4.

### Zadanie 4: Teksty

Wszystkie teksty do `strings.xml`. Dodaj folder `res/values-en/` z `strings.xml` po angielsku (te same `name`, inne wartości). Zmień język emulatora na angielski (*Settings → System → Languages*) i uruchom – aplikacja ma się przetłumaczyć sama.

### Zadanie 5: Przycisk i licznik

Zrób rozdział 7. Potem dodaj drugi przycisk „Reset”, który zeruje licznik i tekst. Id według konwencji, teksty ze `strings.xml`.

### Zadanie 6: Twoja domena

Zmień aplikację na pierwszy ekran z Twojej domeny projektu (sklep / ogłoszenia / rezerwacje / blog): nagłówek z nazwą, jeden opis, jeden przycisk „Start”, który na razie pokazuje Toast „Wkrótce”. Ikona z `mipmap` może zostać.

### Zadanie 7: Logcat

Dodaj `Log.d` w `onCreate` i w obsłudze kliknięcia. Uruchom, pokaż w Logcat z filtrem `tag:`. Potem wywołaj celowo `NullPointerException` (rozdział 8), zrób zrzut stack trace do README i opisz jedną linią, która linia Twojego kodu zawiniła.

### Zadanie 8: Obrót

Obróć emulator po kilku kliknięciach. Opisz w README, co się stało z licznikiem i dlaczego (jedno zdanie – pełne wyjaśnienie w M02).

### Zadanie 9: Repo

`.gitignore` sprawdzony, `local.properties` i `build/` **nie ma** na GitHubie. Commity: `chore: init Android project`, `feat: add greeting text and click counter`, `feat: add English translation`, `docs: add README with file map`.

---

## 12. Pytania kontrolne

Sprawdź, czy opanowałeś materiał. Odpowiedz sobie sam, bez zaglądania.

1. Czym różni się aplikacja mobilna od strony WWW pod względem tego, gdzie działa kod i kto rysuje ekran?
2. Który szablon projektu wybierasz i dlaczego nie *Empty Activity*?
3. Co to jest package name, jaka jest konwencja i dlaczego musi być unikalny?
4. Co oznacza Minimum SDK 26? Co się stanie na telefonie z Androidem 7?
5. Cztery pliki, od których zaczynasz w nowym projekcie – wymień i powiedz, za co każdy odpowiada.
6. `MainActivity.java` i `activity_main.xml` – który to „kontroler”, który „szablon”? Która linia je łączy?
7. Co to jest klasa `R`? Kto ją pisze? Co oznacza `R.id.button_click`?
8. `@+id/main` vs `@id/main` – różnica?
9. Dlaczego `super.onCreate(savedInstanceState)` musi być pierwsze?
10. `findViewById` przed `setContentView` – co się stanie i dlaczego?
11. `match_parent` vs `wrap_content`. Kiedy które?
12. `dp` vs `sp` vs `px` – kiedy które i dlaczego nie `px`?
13. Po co teksty w `strings.xml`, skoro `android:text="Cześć"` działa?
14. Co musi być w `AndroidManifest.xml` dla każdego ekranu? Co się stanie bez tego?
15. Co robi `<intent-filter>` z `MAIN` i `LAUNCHER`? Ile ekranów powinno go mieć?
16. Toast bez `.show()` – co się stanie?
17. Po obrocie telefonu licznik wraca do zera. Dlaczego?
18. Gdzie szukasz stack trace po awarii? Jak go przefiltrować, żeby widzieć tylko swoją aplikację?
19. Dlaczego `local.properties` i `build/` nie idą do Gita?
20. Otwórz swój `activity_main.xml` i wskaż dowolny element: jaki ma id, jakie constrainty, jak dostaniesz się do niego z Javy i jaką metodą zmienisz jego tekst.
