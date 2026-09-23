# Django – start

**Zaawansowane aplikacje webowe | klasa 4 | materiał 10**

Instalacja, struktura projektu, Hello World, dynamiczne ścieżki. Wszystko, co robiłeś we Flasku w materiałach 04–05 – teraz w Django. Po drodze zobaczysz, co Django robi za Ciebie i za co każe płacić strukturą.

---

## Spis treści

1. Czym jest Django i czym się różni od Flaska
2. Instalacja i pierwszy projekt
3. Co jest w tych plikach
4. Aplikacja w projekcie
5. Hello World
6. Co robi każda linia
7. Kilka widoków
8. Dynamiczne ścieżki
9. Query string
10. reverse, redirect, Http404
11. Gdzie tu jest MTV
12. Najczęstsze problemy
13. Do repozytorium
14. Zadania
15. Pytania kontrolne

---

## 1. Czym jest Django i czym się różni od Flaska

**Django** to framework webowy „z bateriami” (*batteries included*). Flask dał Ci routing i szablony, resztę dokładałeś sam. Django daje od razu: ORM, migracje, panel admina, logowanie, formularze, ochronę przed CSRF, system szablonów, testy. Nie musisz wybierać bibliotek – jest jeden sposób i jest wbudowany.

Cena: struktura. Flask to jeden plik, jeśli chcesz. Django od pierwszej minuty każe mieć folder projektu, folder aplikacji, osobny plik na URL-e, osobny na widoki, osobny na modele. Na początku wygląda to na przerost formy. Po dwóch tygodniach zobaczysz, że to jest MTV wymuszone przez framework – i że dokładnie to samo robiłeś ręcznie w materiale 07, rozbijając `app.py` na `models.py`.

| | Flask | Django |
|---|---|---|
| Struktura projektu | dowolna | narzucona: projekt → aplikacje |
| Trasy | `@app.route` nad funkcją | osobny plik `urls.py` |
| Widoki (kontrolery) | funkcje z dekoratorem | funkcje w `views.py` |
| Szablony | Jinja2, folder `templates/` | własny silnik, prawie ta sama składnia |
| Baza danych | dokładasz SQLAlchemy | wbudowany ORM |
| Migracje | dokładasz Flask-Migrate | wbudowane: `makemigrations`, `migrate` |
| Panel admina | brak | wbudowany, generowany z modeli |
| Logowanie użytkowników | dokładasz | wbudowane |
| Kto tego używa | API, mikroserwisy, małe rzeczy | Instagram, Pinterest, Mozilla, zawodowe.edu.pl |

Django jest w podstawie programowej INF.04 z nazwy. Flask nie. Flask był po to, żebyś zobaczył gołe żądanie i odpowiedź. Teraz zobaczysz, jak to samo wygląda, gdy framework ma zdanie na temat tego, gdzie co leży.

> 💡 Nazwa pochodzi od Django Reinhardta, gitarzysty jazzowego. „D” jest nieme: *dżango*. Powstał w 2003 w redakcji gazety w Kansas, bo dziennikarze potrzebowali szybko stawiać strony. Stąd panel admina od pierwszej wersji – redakcja musiała wpisywać treści bez programisty.

---

## 2. Instalacja i pierwszy projekt

### 2.1 Python – sprawdź wersję

Instalujemy **Django 6.x**. Wymaga **Pythona 3.12 lub nowszego**. Sprawdź, zanim cokolwiek zrobisz:

```powershell
python --version
```

Jeśli masz 3.11 albo starszego, `pip` zainstaluje Ci po cichu Django 5.2 (ostatnie, które działa na 3.11) – i wszystko będzie działać, ale nie tego uczymy. Zaktualizuj Pythona z https://www.python.org/downloads/ (przy instalacji zaznacz *Add python.exe to PATH*), potem sprawdź jeszcze raz.

### 2.2 Nowy folder, nowy venv

Osobny projekt, osobne środowisko. Nie mieszaj z Flaskiem.

```powershell
mkdir django-shop
cd django-shop
python -m venv venv
venv\Scripts\activate
pip install "django>=6,<7"
pip freeze > requirements.txt
```

Sprawdź:

```powershell
django-admin --version
```

Powinno wypisać `6.1.x` (albo `6.0.x`). Jeśli widzisz `5.2.x` – wróć do 2.1, masz za stary Python.

> 💡 Django przechodzi na numerowanie latami: po 6.1 następna duża wersja nie będzie 7.0, tylko **Django 2028**. Jak Ubuntu. Więc jeśli w dokumentacji zobaczysz „RemovedInDjango2028Warning” – to nie literówka.

### 2.3 Projekt

```powershell
django-admin startproject config .
```

Kropka na końcu jest **ważna** – znaczy „w tym folderze”, bez niej Django założy dodatkowy folder `config/config/`. Nazwa `config` to konwencja: ten folder trzyma ustawienia, nie kod aplikacji. Zobaczysz też nazwy `core`, `mysite`, albo nazwę projektu – nie ma znaczenia, byle nie `django`.

Po tej komendzie masz:

```text
django-shop/
├── manage.py
├── config/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── asgi.py
│   └── wsgi.py
├── venv/
└── requirements.txt
```

### 2.4 Uruchom

```powershell
python manage.py runserver
```

W terminalu:

```text
You have 18 unapplied migration(s). ...
Django version 6.1, using settings 'config.settings'
Starting development server at http://127.0.0.1:8000/
```

Otwórz http://127.0.0.1:8000 – zobaczysz rakietę i „The install worked successfully!”. Serwer deweloperski, jak we Flasku z `debug=True`: przeładowuje się po każdej zmianie, Ctrl + C zatrzymuje.

Komunikat o 18 migracjach zignoruj na razie. To tabele wbudowanych aplikacji (użytkownicy, sesje, admin). Zajmiemy się nimi przy modelach. Port jest inny niż we Flasku: **8000**, nie 5000.

---

## 3. Co jest w tych plikach

| Plik | Co robi | Dotykasz? |
|---|---|---|
| `manage.py` | narzędzie do wszystkiego: uruchom serwer, zrób migrację, otwórz konsolę. Odpowiednik `flask` z terminala | nigdy nie edytujesz, tylko uruchamiasz |
| `config/settings.py` | ustawienia: baza, zainstalowane aplikacje, szablony, język, strefa czasowa | tak, często |
| `config/urls.py` | główna mapa URL → widok. Odpowiednik wszystkich `@app.route` w jednym miejscu | tak |
| `config/asgi.py`, `wsgi.py` | punkt wejścia dla serwera produkcyjnego | nie, aż do wdrożenia |
| `config/__init__.py` | pusty plik, który mówi Pythonowi „to jest pakiet” | nie |

Zajrzyj do `settings.py`. Zobaczysz około 120 linii. Trzy rzeczy, które warto poznać od razu:

```python
DEBUG = True          # jak debug=True we Flasku – tylko lokalnie!

ALLOWED_HOSTS = []    # przy DEBUG=True działa localhost; przy wdrożeniu wpiszesz domenę

INSTALLED_APPS = [    # lista aplikacji, które Django ma widzieć
    "django.contrib.admin",
    "django.contrib.auth",
    ...
]
```

Na dole ustaw język i strefę czasową – przyda się przy panelu admina i datach:

```python
LANGUAGE_CODE = "pl"
TIME_ZONE = "Europe/Warsaw"
```

> ⚠️ Nad `DEBUG` jest `SECRET_KEY`. To klucz do podpisywania sesji i tokenów. W repozytorium szkolnym może zostać, w prawdziwym projekcie idzie do `.env`. Ale zapamiętaj, że tam jest.

---

## 4. Aplikacja w projekcie

Django rozróżnia **projekt** (cała strona, jeden `settings.py`) i **aplikację** (jeden moduł funkcjonalny: sklep, blog, użytkownicy). Projekt może mieć wiele aplikacji, aplikacja może być przenoszona między projektami. Dla nas: jeden projekt, na razie jedna aplikacja.

```powershell
python manage.py startapp shop
```

Nazwa aplikacji z Twojej domeny: `shop`, `listings`, `bookings`, `blog`. Po angielsku, liczba mnoga albo rzeczownik zbiorowy. Powstaje:

```text
shop/
├── __init__.py
├── admin.py        <- rejestracja modeli w panelu admina (później)
├── apps.py         <- konfiguracja aplikacji (nie dotykasz)
├── migrations/     <- historia zmian bazy (później)
├── models.py       <- M z MTV (później)
├── tests.py        <- testy (później)
└── views.py        <- V z MTV, czyli kontrolery – dziś
```

Nie ma `urls.py` – zaraz go dodasz ręcznie. Nie ma `templates/` – dodasz w następnym materiale.

**Zarejestruj aplikację** w `config/settings.py`. Bez tego Django jej nie widzi:

```python
INSTALLED_APPS = [
    "shop",
    "django.contrib.admin",
    ...
]
```

Kolejność: własne aplikacje na górze, wbudowane pod spodem. Ma to znaczenie przy szablonach – Twoje nadpisują wbudowane.

---

## 5. Hello World

Trzy pliki. We Flasku był jeden – to jest ta cena za strukturę, o której mówiłem.

### 5.1 Widok – `shop/views.py`

```python
from django.http import HttpResponse


def index(request):
    return HttpResponse("Hello World!")
```

### 5.2 URL-e aplikacji – `shop/urls.py` (nowy plik)

```python
from django.urls import path

from . import views

app_name = "shop"

urlpatterns = [
    path("", views.index, name="index"),
]
```

### 5.3 URL-e projektu – `config/urls.py`

Zastąp zawartość:

```python
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path("admin/", admin.site.urls),
    path("", include("shop.urls")),
]
```

### 5.4 Sprawdź

```powershell
python manage.py runserver
```

http://127.0.0.1:8000 → „Hello World!”. Zamiast rakiety.

---

## 6. Co robi każda linia

### `views.py`

| Linia | Co robi |
|---|---|
| `from django.http import HttpResponse` | klasa odpowiedzi. We Flasku zwracałeś string i Flask opakowywał go sam. Django każe opakować jawnie |
| `def index(request):` | widok = funkcja, która **zawsze** dostaje `request` jako pierwszy argument. We Flasku `request` był globalny i importowałeś go; tu przychodzi jako parametr |
| `return HttpResponse("Hello World!")` | odpowiedź: treść, domyślnie status 200 i `Content-Type: text/html` |

### `shop/urls.py`

| Linia | Co robi |
|---|---|
| `from django.urls import path` | funkcja do definiowania jednej trasy |
| `from . import views` | import widoków z tego samego folderu (kropka = „stąd”) |
| `app_name = "shop"` | przestrzeń nazw. Dzięki niej trasa nazywa się `shop:index`, a nie `index` – gdy dojdzie druga aplikacja z własnym `index`, nie będzie konfliktu |
| `urlpatterns = [...]` | lista tras. Django szuka **tej nazwy**, nie innej |
| `path("", views.index, name="index")` | trasa: wzorzec URL, funkcja widoku, nazwa. Pusty string = główna strona aplikacji |

### `config/urls.py`

| Linia | Co robi |
|---|---|
| `path("admin/", admin.site.urls)` | panel admina pod `/admin/`. Zostaw – użyjesz przy modelach |
| `path("", include("shop.urls"))` | „wszystko, co zaczyna się od pustego prefiksu, przekaż do `shop/urls.py`”. Gdyby było `path("sklep/", include(...))`, aplikacja żyłaby pod `/sklep/` |

### Jak Django znajduje widok

```text
GET /
  |
  v
config/urls.py:  "admin/"?  nie.  ""?  tak -> include("shop.urls")
  |
  v
shop/urls.py:    ""?  tak -> views.index
  |
  v
views.index(request) -> HttpResponse("Hello World!")
```

Dwa poziomy. Projekt mówi „ta aplikacja obsługuje ten prefiks”, aplikacja mówi „ten widok obsługuje tę resztę”. Dzięki temu aplikację `shop` możesz wpiąć do innego projektu jedną linią.

> 💡 We Flasku dekorator `@app.route("/")` łączył trasę z funkcją w jednym miejscu. Django rozdziela: funkcja nie wie, pod jakim adresem żyje. To ta sama funkcja może być pod dwoma adresami, albo przenosisz ją bez ruszania kodu. Na początku irytuje, potem doceniasz.

---

## 7. Kilka widoków

`shop/views.py`:

```python
from django.http import HttpResponse, JsonResponse


def index(request):
    return HttpResponse("Strona główna")


def about(request):
    return HttpResponse("Jesteśmy klasą 4TP. Robimy sklep.")


def status(request):
    return JsonResponse({"ok": True, "version": "0.1"})


def forbidden(request):
    return HttpResponse("Brak dostępu", status=403)
```

`shop/urls.py`:

```python
urlpatterns = [
    path("", views.index, name="index"),
    path("about/", views.about, name="about"),
    path("api/status/", views.status, name="status"),
    path("admin-panel/", views.forbidden, name="forbidden"),
]
```

Sprawdź `/about/`, `/api/status/`, `/admin-panel/` (F12 → Network → status 403).

Różnice względem Flaska:

- **JSON**: Flask zamieniał słownik sam. Django ma osobną klasę `JsonResponse`. Jawnie, jak wszystko tutaj.
- **Status**: `HttpResponse("...", status=403)` zamiast `return "...", 403`.
- **Slash na końcu**: `about/`, nie `about`. O tym za chwilę, bo to jest pułapka numer jeden.

### 7.1 Ukośnik na końcu

Django ma konwencję: **każdy adres kończy się ukośnikiem**. `/about/`, nie `/about`. Jeśli wejdziesz na `/about` (bez), Django odpowie **301** i przekieruje na `/about/`. Zobaczysz to w Network jako dwa żądania.

Wzorce w `urls.py` piszesz **bez** ukośnika na początku i **z** ukośnikiem na końcu: `"about/"`. Flask miał odwrotnie: `"/about"`. Przez pierwszy tydzień będziesz to mylił. Normalne.

---

## 8. Dynamiczne ścieżki

To samo, co w materiale 05, tylko składnia jest w `urls.py`, nie w dekoratorze.

### 8.1 Parametr w URL

`shop/urls.py`:

```python
path("hello/<str:name>/", views.hello, name="hello"),
```

`shop/views.py`:

```python
def hello(request, name):
    return HttpResponse(f"Cześć, {name}!")
```

`/hello/Ania/` → „Cześć, Ania!”. Trzy rzeczy muszą się zgadzać, jak we Flasku: nazwa w `<...>`, nazwa argumentu w `def`, użycie w ciele. Plus `request` na pierwszym miejscu – zawsze.

### 8.2 Konwertery

| Zapis | Typ | Pasuje do | Nie pasuje do |
|---|---|---|---|
| `<str:name>` | str (bez `/`) | `hello/abc/`, `hello/12/` | `hello/a/b/` |
| `<int:id>` | int | `products/12/` | `products/abc/` → 404 |
| `<slug:slug>` | str: litery, cyfry, `-`, `_` | `post/moj-pierwszy-post/` | `post/mój post/` |
| `<uuid:id>` | UUID | `order/123e4567-e89b-.../` | – |
| `<path:rest>` | str ze slashami | `files/a/b/c.txt` | – |

`str` jest domyślne – `<name>` to to samo co `<str:name>`. Django nie ma `float` w standardzie (Flask miał). `slug` za to jest nowy i bardzo przydatny: ładne adresy typu `/products/laptop-dell-15/` zamiast `/products/17/`.

### 8.3 Kalkulator

```python
path("add/<int:a>/<int:b>/", views.add, name="add"),
path("divide/<int:a>/<int:b>/", views.divide, name="divide"),
```

```python
def add(request, a, b):
    return HttpResponse(f"{a} + {b} = {a + b}")


def divide(request, a, b):
    if b == 0:
        return HttpResponse("Nie dzielimy przez zero", status=400)
    return HttpResponse(f"{a} / {b} = {a / b}")
```

`/add/2/3/` → „2 + 3 = 5”. `/add/a/b/` → 404, bo `int` nie pasuje – Django nawet nie wywoła widoku. `/divide/7/0/` → 400.

---

## 9. Query string

We Flasku: `request.args`. W Django: **`request.GET`**. Nazwa myląca – to nie „metoda GET”, tylko słownik parametrów z adresu. Jest też `request.POST` dla danych z formularza.

```python
path("greeting/", views.greeting, name="greeting"),
```

```python
def greeting(request):
    name = request.GET.get("name", "nieznajomy")
    hour = request.GET.get("hour")
    if hour is not None and int(hour) < 12:
        return HttpResponse(f"Dzień dobry, {name}!")
    return HttpResponse(f"Witaj, {name}!")
```

`/greeting/?name=Ola&hour=9` → „Dzień dobry, Ola!”. `/greeting/` → „Witaj, nieznajomy!”.

Uwaga na `int(hour)` – jeśli ktoś wpisze `?hour=abc`, poleci `ValueError` i dostaniesz 500. Flask miał `type=int`, Django nie. Walidację wejścia zrobimy porządnie przy formularzach; na razie wiedz, że tu jest dziura.

Zasada z materiału 05 obowiązuje: identyfikator zasobu w ścieżce (`/products/5/`), filtrowanie i sortowanie w query stringu (`/products/?category=laptops&sort=price`).

---

## 10. reverse, redirect, Http404

### 10.1 reverse – nie wpisuj adresów ręcznie

Odpowiednik `url_for`. Bierze **nazwę** trasy (z `app_name` i `name`) i buduje adres:

```python
from django.urls import reverse

reverse("shop:index")                            # "/"
reverse("shop:hello", args=["Ania"])             # "/hello/Ania/"
reverse("shop:product_detail", args=[5])         # "/products/5/"
reverse("shop:add", kwargs={"a": 2, "b": 3})     # "/add/2/3/"
```

`shop:` to przestrzeń nazw z `app_name`. Bez niej Django nie znajdzie trasy. To jest dokładnie ten moment, w którym `app_name = "shop"` z rozdziału 5 przestaje być dekoracją.

### 10.2 redirect

```python
from django.shortcuts import redirect


def old_address(request):
    return redirect("shop:index")
```

`redirect` przyjmuje nazwę trasy bezpośrednio – nie musisz wołać `reverse`. Można też podać adres: `redirect("/")`. Nazwa jest lepsza, bo przeżyje zmianę adresu. Kod 302, jak we Flasku.

### 10.3 Http404

```python
from django.http import Http404

PRODUCTS = {1: "Laptop", 2: "Mysz", 3: "Klawiatura"}


def product_detail(request, product_id):
    if product_id not in PRODUCTS:
        raise Http404("Nie ma takiego produktu")
    return HttpResponse(f"Produkt: {PRODUCTS[product_id]}")
```

```python
path("products/<int:product_id>/", views.product_detail, name="product_detail"),
```

We Flasku był `abort(404)`, tu `raise Http404`. Wyjątek, nie funkcja – bo to Python: rzucasz, Django łapie i zamienia na odpowiedź 404. Przy `DEBUG=True` zobaczysz żółtą stronę z komunikatem i listą wszystkich tras, które Django próbowało. Ta strona jest **bardzo** przydatna do debugowania URL-i – czytaj ją, zamiast od razu pytać agenta.

Słownik `PRODUCTS` to znowu zalążek modelu. W następnym materiale zamienimy go na `models.Model` – i zobaczysz, że Django ma `get_object_or_404`, dokładnie jak Flask-SQLAlchemy miał `get_or_404`.

---

## 11. Gdzie tu jest MTV

| Warstwa | MVC | Django (MTV) | U nas dziś |
|---|---|---|---|
| dane | Model | **Model** | słownik `PRODUCTS` (tymczasowo) |
| prezentacja | View | **Template** | jeszcze nie – zwracamy goły tekst |
| obsługa żądań | Controller | **View** | funkcje w `views.py` |
| mapa adresów | (część kontrolera) | **urls.py** | `shop/urls.py` + `config/urls.py` |

Przypomnienie z materiału 03: **View w Django to Controller z MVC**. Funkcja `product_detail` w `views.py` to kelner, nie talerz. Talerz (szablon) będzie w następnym materiale.

Django dokłada czwarty element, którego w klasycznym MVC nie ma jako osobnego pliku: **routing** w `urls.py`. We Flasku był sklejony z widokiem przez dekorator. Tu jest osobno – i to jest pierwszy plik, który otwierasz, gdy chcesz zrozumieć cudzy projekt Django: mapa całej strony w jednym miejscu.

---

## 12. Najczęstsze problemy

| Objaw | Przyczyna i rozwiązanie |
|---|---|
| `ModuleNotFoundError: No module named 'django'` | venv nieaktywny. `venv\Scripts\activate` |
| `django-admin --version` pokazuje `5.2.x` | Python starszy niż 3.12. Zaktualizuj Pythona, usuń `venv/`, załóż od nowa |
| `ERROR: No matching distribution found for django>=6` | jak wyżej – pip nie znajdzie 6.x dla Pythona 3.11 |
| `ModuleNotFoundError: No module named 'shop'` | aplikacja nie jest w `INSTALLED_APPS` albo `runserver` uruchomiony z innego folderu niż `manage.py` |
| `ImportError: cannot import name 'views'` | brak `from . import views` w `shop/urls.py`, albo literówka |
| `NoReverseMatch: 'shop' is not a registered namespace` | brak `app_name = "shop"` w `shop/urls.py` |
| `NoReverseMatch: Reverse for 'hello' with arguments '()' not found` | trasa wymaga parametru, a nie podałeś `args=[...]` |
| 404 na adresie, który „na pewno jest” | brak ukośnika na końcu wzorca, albo `/` na początku wzorca (`"/about/"` zamiast `"about/"`). Przeczytaj żółtą stronę – Django wypisuje, czego próbowało |
| 301 zamiast 200 | wszedłeś bez ukośnika na końcu, Django przekierował. To nie błąd |
| `Page not found` na `/` po `startproject` | nie dodałeś `path("", include("shop.urls"))` w `config/urls.py` |
| `TypeError: index() got an unexpected keyword argument 'name'` | nazwa w `<str:name>` ≠ nazwa argumentu w `def` |
| `TypeError: hello() missing 1 required positional argument: 'request'` | zapomniałeś `request` jako pierwszego parametru widoku |
| zmiany nie działają | `runserver` nie przeładowuje po błędzie składni – spójrz w terminal, tam jest traceback |

---

## 13. Do repozytorium

`.gitignore`:

```text
venv/
__pycache__/
*.pyc
db.sqlite3
.env
.idea/
.vscode/
```

`db.sqlite3` – Django tworzy ten plik przy pierwszym `migrate`. Nie idzie do repo, jak `instance/` we Flasku.

`README.md` według szablonu z materiału 09. W sekcji Struktura opisz **oba** `urls.py` i po co są dwa.

```bash
git init
git add .
git commit -m "chore: init Django project with shop app"
git branch -M main
git remote add origin https://github.com/TWOJ_LOGIN/django-shop.git
git push -u origin main
```

---

## 14. Zadania

### Zadanie 1: Projekt i aplikacja

Sprawdź wersję Pythona (≥ 3.12). Załóż `django-shop` (lub nazwę z Twojej domeny) według rozdziału 2 i 4. `django-admin --version` ma pokazać `6.x`. Nazwa aplikacji z Twojej domeny, po angielsku. `runserver` → rakieta. Commit: `chore: init Django project`.

### Zadanie 2: Hello World

Trzy pliki z rozdziału 5. `/` ma zwracać nazwę Twojego projektu. Zanim uruchomisz, narysuj na kartce ścieżkę żądania `GET /` przez oba `urls.py` do widoku – jak w rozdziale 6.

### Zadanie 3: Cztery widoki

`/`, `/about/`, `/contact/`, `/terms/` – każdy zwraca tekst związany z Twoją domeną. Plus `/api/info/` zwracający `JsonResponse` z kluczami `name`, `author`, `version`. Plus `/admin-panel/` ze statusem 403.

### Zadanie 4: Ukośnik

Wejdź na `/about` bez ukośnika. W Network pokaż nauczycielowi 301 → 200. Potem zmień wzorzec w `urls.py` na `"about"` (bez ukośnika) i sprawdź, co się dzieje z `/about/` i `/about`. Wróć do wersji z ukośnikiem. Zapisz w README jedno zdanie, co zaobserwowałeś.

### Zadanie 5: Dynamiczne trasy

`hello/<str:name>/` i `hello/<str:name>/<int:age>/`. Kalkulator: `add`, `subtract`, `multiply`, `divide` z `<int:a>/<int:b>/`; dzielenie przez zero → 400. Sprawdź `/add/a/b/` – jaki kod i dlaczego widok się nie wywołał?

### Zadanie 6: Slug

Trasa `products/<slug:slug>/` zwracająca „Produkt: {slug}”. Sprawdź `/products/laptop-dell-15/` oraz `/products/laptop dell/` (ze spacją). Który działa i dlaczego?

### Zadanie 7: Query string

`/items/` obsługuje `?category=` i `?sort=`. Zwraca „Kategoria: X, sortowanie: Y”, domyślnie „all, default”. Dodatkowo: `?page=` zamieniany na `int` – zabezpiecz przed `?page=abc` tak, żeby nie było 500 (podpowiedź: `try/except ValueError` albo `str.isdigit()`).

### Zadanie 8: Mini-baza i 404

Słownik z 5 elementami z Twojej domeny. `items/<int:item_id>/` zwraca nazwę albo `raise Http404`. Wejdź na nieistniejący id i przeczytaj żółtą stronę – co Django wypisało? Zapisz w README.

### Zadanie 9: reverse i redirect

`/start/` przekierowuje na `/` przez `redirect("shop:index")`. `/links/` zwraca tekst z trzema adresami zbudowanymi przez `reverse`, w tym jeden z parametrem. Usuń na chwilę `app_name` z `shop/urls.py` i zobacz, jaki błąd dostaniesz. Przywróć.

### Zadanie 10: Repo

`.gitignore`, README z tabelą tras (metoda, adres, nazwa trasy, opis), commity według Conventional Commits. Push. Sprawdź na GitHubie, że `venv/` nie ma.

---

## 15. Pytania kontrolne

Sprawdź, czy opanowałeś materiał. Odpowiedz sobie sam, bez zaglądania. Jeśli przy którymś się zatniesz – wróć do rozdziału, zanim wrócę do tego ja.

1. Dlaczego w projekcie są dwa pliki `urls.py`? Co jest w każdym z nich?
2. Co robi `include("shop.urls")` i co by się stało, gdybyś wpisał `path("sklep/", include("shop.urls"))`?
3. Po co jest `app_name = "shop"`? Jaki błąd dostaniesz bez niego i kiedy?
4. Czym się różni `request.GET` od metody HTTP GET? Do czego służy `request.POST`?
5. Wchodzisz na `/about` bez ukośnika. Jaki kod odpowiedzi, co robi przeglądarka, co widać w Network?
6. Wzorzec `"about/"` vs `"/about/"` w `urls.py` – który jest poprawny i co się stanie z drugim?
7. Dlaczego każdy widok w Django ma `request` jako pierwszy parametr, skoro we Flasku go nie było?
8. `/add/a/b/` przy wzorcu `add/<int:a>/<int:b>/` – jaki kod i czy widok w ogóle się wywołał?
9. Czym się różni `<str:name>` od `<slug:name>`? Podaj adres, który pasuje do jednego, a nie do drugiego.
10. `reverse("shop:product_detail", args=[5])` – co zwróci? Co się stanie, jeśli trasa wymaga parametru, a nie podasz `args`?
11. Czym się różni `redirect("shop:index")` od `redirect("/")`? Który przeżyje zmianę adresu?
12. `raise Http404` to wyjątek, nie funkcja. Kto go łapie i co robi?
13. `request.GET.get("hour")` zwróciło `"abc"`, a Ty robisz `int(hour)`. Jaki kod odpowiedzi zobaczy użytkownik i czyja to wina według tabeli z materiału 08?
14. W tabeli MTV z rozdziału 11: która warstwa to `views.py`, a która `urls.py`? Dlaczego klasyczne MVC nie ma `urls.py` jako osobnego pliku?
15. `python --version` pokazuje 3.11, a `django-admin --version` pokazuje 5.2. Co się stało i co zrobić?
16. Otwórz swoje `views.py`, wskaż dowolną funkcję i odpowiedz: pod jakim adresem żyje, jakie ma parametry, jaki kod zwróci dla poprawnego wejścia i jaki dla błędnego.