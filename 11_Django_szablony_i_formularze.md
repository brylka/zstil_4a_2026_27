# Django – szablony i formularze

**Zaawansowane aplikacje webowe | klasa 4 | materiał 11**

Odpowiednik materiału 06 z Flaska: HTML wynosimy z Pythona do szablonów, robimy szablon bazowy, CSS, listy, warunki. A potem coś, czego we Flasku nie było: **formularze z walidacją**, które Django robi za Ciebie – łącznie z komunikatami błędów, ponownym wypełnieniem pól i ochroną przed CSRF.

---

## Spis treści

1. Gdzie leżą szablony
2. render – pierwszy szablon
3. Składnia: to samo co Jinja2, prawie
4. Szablon bazowy
5. Pliki statyczne
6. Lista i szczegóły
7. Formularze po staremu – i dlaczego nie
8. django.forms – formularz jako klasa
9. Widok obsługujący formularz
10. Walidacja: wbudowana, własna, między polami
11. CSRF – dlaczego bez tokena jest 403
12. Formularz wyszukiwania (GET)
13. Gdzie teraz jest MTV
14. Najczęstsze problemy
15. Zadania
16. Pytania kontrolne

---

## 1. Gdzie leżą szablony

We Flasku był jeden folder `templates/` obok `app.py`. W Django każda aplikacja ma swój, a w nim – uwaga – **jeszcze jeden podfolder z nazwą aplikacji**:

```text
shop/
├── templates/
│   └── shop/            <- tak, drugi raz "shop"
│       ├── base.html
│       ├── index.html
│       └── product_list.html
└── static/
    └── shop/            <- i tu też
        └── style.css
```

Wygląda głupio. Ma sens: Django zbiera szablony ze **wszystkich** aplikacji do jednej puli. Jeśli `shop` ma `index.html` i `blog` ma `index.html`, wygra ten, którego aplikacja jest wyżej w `INSTALLED_APPS` – po cichu, bez błędu. Podfolder z nazwą aplikacji robi z tego `shop/index.html` i `blog/index.html`. Koniec konfliktu.

Załóż foldery teraz:

```powershell
mkdir shop\templates\shop
mkdir shop\static\shop
```

Django znajdzie je sam, bo `shop` jest w `INSTALLED_APPS` i w `settings.py` jest `"APP_DIRS": True` (zajrzyj do sekcji `TEMPLATES`).

---

## 2. render – pierwszy szablon

`shop/templates/shop/index.html`:

```html
<!DOCTYPE html>
<html lang="pl">
<head>
    <meta charset="UTF-8">
    <title>{{ project_name }}</title>
</head>
<body>
    <h1>{{ project_name }}</h1>
    <p>Witaj, {{ user_name }}!</p>
</body>
</html>
```

`shop/views.py`:

```python
from django.shortcuts import render


def index(request):
    return render(request, "shop/index.html", {"project_name": "Sklep 4TP", "user_name": "Ania"})
```

Trzy różnice względem Flaska:

| Flask | Django |
|---|---|
| `render_template("index.html", imie="Ania")` | `render(request, "shop/index.html", {"user_name": "Ania"})` |
| zmienne jako argumenty nazwane | zmienne jako **słownik** (nazywa się *context*) |
| nie ma `request` | `request` zawsze pierwszy – Django potrzebuje go m.in. do tokena CSRF |

Ścieżka `"shop/index.html"` to ścieżka **wewnątrz** `templates/`, stąd prefiks `shop/`.

> 💡 Słownik w `render` to jest dokładnie to, co szablon „widzi”. Nic więcej. Jeśli w szablonie użyjesz `{{ price }}`, a w słowniku nie ma `price`, Django wstawi pusty string i **nie zgłosi błędu**. Jinja2 też tak robiła. Jak coś się nie wyświetla – najpierw sprawdź słownik.

---

## 3. Składnia: to samo co Jinja2, prawie

Silnik szablonów Django (DTL) wygląda jak Jinja2, bo Jinja2 powstała jako jego klon. Różnice są małe, ale wpadniesz na każdą z nich.

| Co | Jinja2 (Flask) | DTL (Django) |
|---|---|---|
| zmienna | `{{ name }}` | `{{ name }}` |
| atrybut / klucz | `{{ p.name }}` | `{{ p.name }}` – tak samo |
| element listy | `{{ items[0] }}` | `{{ items.0 }}` – **kropka, nie nawiasy** |
| filtr | `{{ name\|upper }}` | `{{ name\|upper }}` |
| filtr z argumentem | `{{ price\|round(2) }}` | `{{ price\|floatformat:2 }}` – **dwukropek** |
| if / for | `{% if %}…{% endif %}` | tak samo |
| numer iteracji | `loop.index` | `forloop.counter` |
| pierwsza / ostatnia | `loop.first` / `loop.last` | `forloop.first` / `forloop.last` |
| link do trasy | `{{ url_for('produkt', id=5) }}` | `{% url 'shop:product_detail' 5 %}` – **tag, nie funkcja** |
| dziedziczenie | `{% extends %}` / `{% block %}` | tak samo |
| wywołanie metody | `{{ p.gross_price() }}` | `{{ p.gross_price }}` – **bez nawiasów** |
| arytmetyka | `{{ price * 1.23 }}` | **nie ma.** Liczysz w Pythonie, nie w szablonie |
| wartość domyślna | `{{ x\|default("brak") }}` | `{{ x\|default:"brak" }}` |
| tak / nie | `{{ "tak" if ok else "nie" }}` | `{{ ok\|yesno:"tak,nie" }}` |

Ta ostatnia różnica z arytmetyką jest celowa. Twórcy Django uznali, że szablon ma **wyświetlać**, nie **liczyć** – dokładnie zgodnie z MVC: talerz nie gotuje. Jeśli potrzebujesz ceny brutto, dodaj metodę w modelu i wołaj `{{ p.gross_price }}`.

Przydatne filtry: `length`, `upper`, `lower`, `title`, `truncatechars:30`, `date:"d.m.Y"`, `floatformat:2`, `default:"–"`, `yesno:"tak,nie"`, `join:", "`, `pluralize`.

> ⚠️ Auto-escaping działa jak w Jinja2: `{{ name }}` z wartością `<b>x</b>` wyświetli dosłownie `<b>x</b>`. Nie wyłączaj (`|safe`) bez powodu – to jest ochrona przed XSS.

---

## 4. Szablon bazowy

`shop/templates/shop/base.html`:

```html
{% load static %}
<!DOCTYPE html>
<html lang="pl">
<head>
    <meta charset="UTF-8">
    <title>{% block title %}Sklep 4TP{% endblock %}</title>
    <link rel="stylesheet" href="{% static 'shop/style.css' %}">
</head>
<body>
    <nav>
        <a href="{% url 'shop:index' %}">Start</a> |
        <a href="{% url 'shop:product_list' %}">Produkty</a> |
        <a href="{% url 'shop:product_add' %}">Dodaj</a>
    </nav>
    <main>
        {% block content %}{% endblock %}
    </main>
    <footer>4TP {% now "Y" %}</footer>
</body>
</html>
```

Nowe rzeczy:

- `{% load static %}` – **pierwsza linia** pliku, w każdym szablonie, który używa `{% static %}`. Bez tego: `Invalid block tag 'static'`.
- `{% static 'shop/style.css' %}` – buduje adres do pliku statycznego. Odpowiednik `url_for('static', ...)`.
- `{% url 'shop:index' %}` – adres z nazwy trasy. Z parametrem: `{% url 'shop:product_detail' p.id %}` – parametry po spacji, bez nawiasów i bez `args=`.
- `{% now "Y" %}` – bieżący rok. Drobiazg, ale przydatny w stopce.

`shop/templates/shop/index.html` – teraz krótko:

```html
{% extends "shop/base.html" %}

{% block content %}
<h1>{{ project_name }}</h1>
<p>Witaj, {{ user_name }}!</p>
{% endblock %}
```

Zasady te same co w Jinja2: `extends` w pierwszej linii, treść tylko w blokach, co poza blokiem – ignorowane.

---

## 5. Pliki statyczne

`shop/static/shop/style.css`:

```css
body { font-family: Arial, sans-serif; max-width: 900px; margin: 2rem auto; }
nav a { margin-right: 1rem; }
table { border-collapse: collapse; }
td, th { border: 1px solid #ccc; padding: .4rem .8rem; }
.errorlist { color: #c0392b; list-style: none; padding: 0; }
```

Przy `DEBUG=True` serwer deweloperski serwuje `static/` sam. Sprawdź http://127.0.0.1:8000/static/shop/style.css – ma się wyświetlić plik. Jeśli 404: `{% load static %}` jest? Folder to `shop/static/shop/`, nie `shop/static/`? `runserver` zrestartowany po założeniu folderu?

Klasa `.errorlist` to nie przypadek – tak Django nazywa listę błędów formularza. Za chwilę ją zobaczysz.

---

## 6. Lista i szczegóły

Dane na razie w liście słowników – jak w materiale 06. Model przyjdzie w 12.

`shop/views.py`:

```python
from django.http import Http404
from django.shortcuts import render

PRODUCTS = [
    {"id": 1, "name": "Laptop", "price": 2999, "category": "laptops", "is_available": True},
    {"id": 2, "name": "Mysz", "price": 49, "category": "accessories", "is_available": False},
    {"id": 3, "name": "Klawiatura", "price": 199, "category": "accessories", "is_available": True},
]


def product_list(request):
    return render(request, "shop/product_list.html", {"products": PRODUCTS})


def product_detail(request, product_id):
    product = next((p for p in PRODUCTS if p["id"] == product_id), None)
    if product is None:
        raise Http404("Nie ma takiego produktu")
    return render(request, "shop/product_detail.html", {"product": product})
```

`shop/urls.py`:

```python
urlpatterns = [
    path("", views.index, name="index"),
    path("products/", views.product_list, name="product_list"),
    path("products/add/", views.product_add, name="product_add"),
    path("products/<int:product_id>/", views.product_detail, name="product_detail"),
]
```

Kolejność ma znaczenie: `products/add/` **przed** `products/<int:product_id>/`. Gdyby było odwrotnie, `add` nie pasuje do `int`, więc akurat tu by zadziałało – ale przy `<str:slug>` już nie. Nawyk: trasy stałe przed trasami z parametrami.

`shop/templates/shop/product_list.html`:

```html
{% extends "shop/base.html" %}
{% block title %}Produkty – Sklep 4TP{% endblock %}
{% block content %}
<h1>Produkty ({{ products|length }})</h1>

{% if products %}
<table>
    <tr><th>#</th><th>Nazwa</th><th>Cena</th><th>Status</th></tr>
    {% for p in products %}
    <tr>
        <td>{{ forloop.counter }}</td>
        <td><a href="{% url 'shop:product_detail' p.id %}">{{ p.name }}</a></td>
        <td>{{ p.price }} zł</td>
        <td>{% if p.is_available %}dostępny{% else %}<b>brak</b>{% endif %}</td>
    </tr>
    {% endfor %}
</table>
{% else %}
<p>Brak produktów.</p>
{% endif %}
{% endblock %}
```

`shop/templates/shop/product_detail.html`:

```html
{% extends "shop/base.html" %}
{% block content %}
<h1>{{ product.name }}</h1>
<p>Cena: {{ product.price }} zł</p>
<p>Dostępny: {{ product.is_available|yesno:"tak,nie" }}</p>
<a href="{% url 'shop:product_list' %}">Wróć</a>
{% endblock %}
```

Jest też `{% empty %}` – skrót na pustą listę wewnątrz pętli:

```html
{% for p in products %}
    <li>{{ p.name }}</li>
{% empty %}
    <li>Brak produktów.</li>
{% endfor %}
```

---

## 7. Formularze po staremu – i dlaczego nie

We Flasku formularz wyglądał tak: HTML ręcznie, `request.form["nazwa"]`, `float(request.form["cena"])` i modlitwa, żeby użytkownik nie wpisał „abc”. Przypomnij sobie, ile rzeczy trzeba było sprawdzić samemu:

- czy pole w ogóle przyszło (`KeyError`),
- czy nie jest puste,
- czy cena to liczba (`ValueError`),
- czy cena jest dodatnia,
- czy nazwa nie ma 500 znaków,
- a jak coś nie gra – wyświetlić komunikat **i** nie wyczyścić pozostałych pól, bo użytkownik Cię zabije.

Na INF.03 pisało się to w PHP: dziesięć `if`-ów, `isset()`, `empty()`, `is_numeric()`, każdy błąd osobno do zmiennej, potem `echo` w HTML-u. Działało. Kod na 80 linii dla jednego formularza z trzema polami.

Django ma na to klasę.

---

## 8. django.forms – formularz jako klasa

Nowy plik `shop/forms.py`:

```python
from django import forms

CATEGORIES = [
    ("laptops", "Laptopy"),
    ("accessories", "Akcesoria"),
    ("monitors", "Monitory"),
]


class ProductForm(forms.Form):
    name = forms.CharField(label="Nazwa", max_length=100, min_length=3)
    price = forms.DecimalField(label="Cena", min_value=0.01, max_digits=8, decimal_places=2)
    category = forms.ChoiceField(label="Kategoria", choices=CATEGORIES)
    is_available = forms.BooleanField(label="Dostępny", required=False, initial=True)
    description = forms.CharField(label="Opis", widget=forms.Textarea, required=False)
```

Czytaj to jak opis formularza: pięć pól, każde ma typ, etykietę i reguły. Z tej jednej klasy Django wygeneruje:

1. **HTML** – `<input>`, `<select>`, `<textarea>` z odpowiednimi atrybutami (`required`, `maxlength`, `min`, `step`).
2. **Walidację** po stronie serwera – te same reguły, sprawdzone w Pythonie, bo atrybutom HTML nie można ufać (F12 → usuń `required` → wyślij).
3. **Komunikaty błędów** – po polsku, bo w `settings.py` masz `LANGUAGE_CODE = "pl"`.
4. **Konwersję typów** – `price` wraca jako `Decimal`, `is_available` jako `bool`, nie jako string `"on"`.

| Pole | Typ w Pythonie | HTML | Częste parametry |
|---|---|---|---|
| `CharField` | `str` | `<input type="text">` | `max_length`, `min_length`, `strip` |
| `CharField(widget=Textarea)` | `str` | `<textarea>` | – |
| `IntegerField` | `int` | `<input type="number">` | `min_value`, `max_value` |
| `DecimalField` | `Decimal` | `<input type="number" step="0.01">` | `max_digits`, `decimal_places` |
| `BooleanField` | `bool` | `<input type="checkbox">` | `required=False` – **prawie zawsze** |
| `ChoiceField` | `str` | `<select>` | `choices=[(wartość, etykieta), …]` |
| `EmailField` | `str` | `<input type="email">` | sprawdza `@` i domenę |
| `DateField` | `date` | `<input type="text">` | `widget=forms.DateInput(attrs={"type": "date"})` |
| `URLField` | `str` | `<input type="url">` | – |

Każde pole ma `required=True` domyślnie. `BooleanField` z `required=True` oznacza „checkbox **musi** być zaznaczony” – dobre dla „akceptuję regulamin”, złe dla „dostępny”. Dlatego `required=False`.

`DecimalField`, nie `FloatField`, do pieniędzy. `0.1 + 0.2` w `float` to `0.30000000000000004`. W `Decimal` to `0.3`. Księgowość Ci podziękuje.

---

## 9. Widok obsługujący formularz

`shop/views.py`:

```python
from django.shortcuts import redirect, render

from .forms import ProductForm


def product_add(request):
    if request.method == "POST":
        form = ProductForm(request.POST)
        if form.is_valid():
            data = form.cleaned_data
            PRODUCTS.append({"id": len(PRODUCTS) + 1, **data})
            return redirect("shop:product_list")
    else:
        form = ProductForm()
    return render(request, "shop/product_form.html", {"form": form})
```

Ten kształt jest **zawsze taki sam**. Nauczysz się go na pamięć, bo napiszesz go pięćdziesiąt razy:

```text
GET  -> pusty formularz -> render
POST -> formularz z danymi -> is_valid()?
           tak -> zrób coś z cleaned_data -> redirect
           nie -> render tego samego szablonu z tym samym form (błędy są w środku)
```

Linia po linii:

| Linia | Co robi |
|---|---|
| `ProductForm(request.POST)` | formularz **związany** z danymi (bound). `request.POST` to słownik pól z formularza – odpowiednik `request.form` z Flaska |
| `form.is_valid()` | uruchamia całą walidację. Zwraca `True`/`False` **i** wypełnia `form.cleaned_data` albo `form.errors` |
| `form.cleaned_data` | słownik z danymi **po konwersji**: `price` to `Decimal`, `is_available` to `bool`. Używaj tylko po `is_valid()` |
| `ProductForm()` | formularz **niezwiązany** (unbound) – pusty, do wyświetlenia przy GET |
| ostatni `render` | wykonuje się w dwóch przypadkach: GET (pusty form) i POST z błędami (form z danymi i błędami). Jeden szablon obsługuje oba |

`shop/templates/shop/product_form.html`:

```html
{% extends "shop/base.html" %}
{% block content %}
<h1>Nowy produkt</h1>
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button>Zapisz</button>
</form>
{% endblock %}
```

`{{ form.as_p }}` – cały formularz, każde pole w `<p>`, z etykietą, inputem i listą błędów. Odśwież `/products/add/` i zajrzyj w źródło strony. Zobaczysz mniej więcej:

```html
<input type="hidden" name="csrfmiddlewaretoken" value="IZSiW3eQ…">
<p>
  <label for="id_name">Nazwa:</label>
  <input type="text" name="name" maxlength="100" minlength="3" required id="id_name">
</p>
<p>
  <label for="id_price">Cena:</label>
  <input type="number" name="price" min="0.01" step="0.01" required id="id_price">
</p>
<p>
  <label for="id_category">Kategoria:</label>
  <select name="category" id="id_category">
    <option value="laptops">Laptopy</option>
    …
  </select>
</p>
```

Tego HTML-a nie napisałeś. Wygenerowała go klasa z rozdziału 8. Zmienisz `max_length` w Pythonie – zmieni się `maxlength` w HTML-u.

Alternatywy: `{{ form.as_div }}`, `{{ form.as_table }}`, albo ręcznie pole po polu:

```html
<p>
    {{ form.name.label_tag }}
    {{ form.name }}
    {{ form.name.errors }}
</p>
```

Ręcznie robisz wtedy, gdy potrzebujesz własnego układu (Bootstrap, grid). Na start – `as_p`.

---

## 10. Walidacja: wbudowana, własna, między polami

### 10.1 Wbudowana – już ją masz

Wyślij formularz z nazwą „ab” i ceną „-5”. Zobaczysz:

```text
Nazwa: Upewnij się, że ta wartość ma przynajmniej 3 znaki (obecnie ma 2).
Cena:  Upewnij się, że ta wartość jest większa lub równa 0.01.
```

Pola zostały wypełnione tym, co wpisałeś. Nic nie kodowałeś – to `min_length=3` i `min_value=0.01` z klasy. Wyślij pusty formularz: „To pole jest wymagane.” przy każdym polu bez `required=False`.

Teraz F12 → usuń atrybut `required` z inputa nazwy → wyślij pusty. Dalej „To pole jest wymagane.” – bo walidacja jest **po stronie serwera**. HTML tylko pomaga użytkownikowi, nie chroni Ciebie.

### 10.2 Własna – jedno pole: `clean_<nazwa_pola>`

Metoda o nazwie `clean_` + nazwa pola. Django wywołuje ją automatycznie po wbudowanej walidacji tego pola.

```python
class ProductForm(forms.Form):
    # ... pola jak wyżej ...

    def clean_name(self):
        name = self.cleaned_data["name"].strip()
        if name.lower() in {"test", "asdf"}:
            raise forms.ValidationError("Wpisz prawdziwą nazwę produktu.")
        return name
```

Zasady:

- bierzesz wartość z `self.cleaned_data["name"]` (już po wbudowanej walidacji – to na pewno string o długości 3–100),
- jeśli źle: `raise forms.ValidationError("komunikat")`. Django łapie, dopisuje do `form.errors["name"]`, `is_valid()` zwróci `False`,
- jeśli dobrze: **zwracasz wartość** (`return name`). Możesz ją po drodze zmienić – tu `strip()` obcina spacje. Zapomnisz `return` – pole będzie `None`.

### 10.3 Własna – kilka pól naraz: `clean`

Gdy reguła dotyczy relacji między polami („laptop nie może kosztować mniej niż 500”), nadpisujesz `clean()`:

```python
    def clean(self):
        cleaned = super().clean()
        price = cleaned.get("price")
        category = cleaned.get("category")
        if price is not None and category == "laptops" and price < 500:
            self.add_error("price", "Laptop za mniej niż 500 zł? Sprawdź cenę.")
        return cleaned
```

- `super().clean()` – najpierw to, co robi Django, potem Twoje,
- `cleaned.get("price")`, nie `cleaned["price"]` – bo jeśli `price` nie przeszło swojej walidacji, w słowniku go **nie ma** i `[]` da `KeyError`,
- `self.add_error("price", "…")` – błąd przypięty do konkretnego pola. `raise ValidationError` w `clean()` dałoby błąd „ogólny” (`form.non_field_errors`), wyświetlany nad formularzem,
- `return cleaned` – jak wyżej.

### 10.4 Kolejność

```text
is_valid()
  |
  v
dla każdego pola: wbudowana walidacja pola (typ, required, min, max)
  |          -> błąd? pole wypada z cleaned_data, lecimy dalej
  v
dla każdego pola: clean_<pole>()   (tylko jeśli pole przeszło krok wyżej)
  |
  v
clean()                            (widzi wszystko, co przeszło)
  |
  v
errors puste? -> True : False
```

Stąd `cleaned.get()` w `clean()`: niektóre pola mogły odpaść wcześniej.

---

## 11. CSRF – dlaczego bez tokena jest 403

Usuń `{% csrf_token %}` z szablonu i wyślij formularz. Dostaniesz **403 Forbidden** i stronę „CSRF verification failed”. Wróć, dodaj z powrotem.

**CSRF** (Cross-Site Request Forgery) to atak: siedzisz zalogowany w banku w jednej karcie, w drugiej otwierasz stronę z kotkami, a strona z kotkami ma ukryty formularz `<form action="https://bank.pl/przelew" method="post">` wysyłany JavaScriptem. Przeglądarka dołącza Twoje ciasteczka z banku, bank widzi zalogowanego użytkownika, przelew idzie.

Obrona: każdy formularz POST ma ukryte pole z losowym tokenem, który zna tylko Twoja strona (jest w ciasteczku **i** w formularzu, muszą się zgadzać). Strona z kotkami nie zna tokena. Django sprawdza to w middleware przy **każdym** POST, PUT, PATCH, DELETE – zanim widok w ogóle się uruchomi. Dlatego `{% csrf_token %}` jest obowiązkowy w każdym `<form method="post">` i dlatego `render` potrzebuje `request` – token siedzi w nim.

We Flasku tego nie było. Musiałbyś doinstalować Flask-WTF i sam pamiętać. Tu jest z pudełka i nie da się zapomnieć, bo 403 przypomni.

Formularz **GET** (wyszukiwarka) tokena nie potrzebuje – GET nie zmienia danych, więc nie ma czego podrobić. Pamiętasz zasadę z materiału 08? Dokładnie dlatego usuwanie i edycja nie mogą być GET-em.

---

## 12. Formularz wyszukiwania (GET)

Ta sama klasa `forms.Form`, ale związana z `request.GET` i bez tokena:

`shop/forms.py`:

```python
class SearchForm(forms.Form):
    q = forms.CharField(label="Szukaj", required=False, max_length=50)
    only_available = forms.BooleanField(label="Tylko dostępne", required=False)
```

`shop/views.py`:

```python
from .forms import ProductForm, SearchForm


def product_list(request):
    form = SearchForm(request.GET)
    products = PRODUCTS
    if form.is_valid():
        q = form.cleaned_data["q"]
        if q:
            products = [p for p in products if q.lower() in p["name"].lower()]
        if form.cleaned_data["only_available"]:
            products = [p for p in products if p["is_available"]]
    return render(request, "shop/product_list.html", {"products": products, "form": form})
```

W `product_list.html`, nad tabelą:

```html
<form method="get">
    {{ form.as_p }}
    <button>Szukaj</button>
</form>
```

`SearchForm(request.GET)` na pustym `request.GET` daje pusty, ale **poprawny** formularz (wszystkie pola `required=False`), więc `is_valid()` przechodzi i lista jest pełna. Po wysłaniu `?q=lap&only_available=on` – pola w formularzu zostają wypełnione, bo formularz jest związany z tymi danymi. Nie musisz ręcznie robić `value="{{ q }}"` jak we Flasku.

---

## 13. Gdzie teraz jest MTV

| Warstwa | Plik | Co tam jest |
|---|---|---|
| **Model** | `views.py` (tymczasowo: lista `PRODUCTS`) | dane – w materiale 12 przejdą do `models.py` |
| **Template** | `templates/shop/*.html` | wyświetlanie, `{% for %}`, `{% if %}`, `{{ form.as_p }}` |
| **View** (kontroler) | `views.py` | GET/POST, `is_valid()`, `redirect` |
| **Formularz** | `forms.py` | opis pól + walidacja |

`forms.py` nie jest w skrócie MTV, ale jest osobną warstwą i ma osobny plik. Co tam **nie** powinno być: zapis do bazy, wysyłka maila. Co powinno: „czy te dane są poprawne i jak je znormalizować”. Widok nie sprawdza `if len(name) < 3` – to robota formularza. Szablon nie decyduje, które pole jest wymagane – to też formularz.

---

## 14. Najczęstsze problemy

| Objaw | Przyczyna i rozwiązanie |
|---|---|
| `TemplateDoesNotExist: shop/index.html` | brak podfolderu `shop/` w `templates/`, albo aplikacja nie w `INSTALLED_APPS`, albo `runserver` nie zrestartowany po założeniu folderu. Żółta strona wypisze, gdzie szukał |
| `Invalid block tag on line 5: 'static'` | brak `{% load static %}` na górze szablonu |
| `NoReverseMatch` w szablonie | zła nazwa w `{% url %}` albo brak parametru: `{% url 'shop:product_detail' p.id %}` |
| `Could not parse the remainder: '[0]'` | `{{ items[0] }}` – w DTL jest `{{ items.0 }}` |
| `TemplateSyntaxError: Could not parse ... '* 1.23'` | arytmetyka w szablonie. Policz w widoku albo w modelu |
| 403 CSRF verification failed | brak `{% csrf_token %}` w formularzu POST |
| formularz się wysyła, ale nic się nie dzieje | brak `method="post"` – domyślnie `<form>` wysyła GET |
| `KeyError: 'price'` w `clean()` | użyj `cleaned.get("price")` – pole mogło nie przejść własnej walidacji |
| pole ma wartość `None` po walidacji | zapomniałeś `return` w `clean_<pole>` |
| checkbox „wymagany” | `BooleanField` bez `required=False` |
| błędy nie wyświetlają się | `render` dostał nowy `ProductForm()` zamiast tego z `request.POST`. Sprawdź, czy ostatni `render` jest **poza** `else` |
| CSS nie działa | `/static/shop/style.css` → 404? `{% load static %}`? Ctrl + F5 (cache) |
| komunikaty po angielsku | `LANGUAGE_CODE = "pl"` w `settings.py`, restart serwera |

---

## 15. Zadania

### Zadanie 1: Szablony i baza

Przenieś stronę główną do `templates/shop/index.html`. Zrób `base.html` z menu (Start, Lista, Dodaj), stopką z `{% now %}` i podpiętym `style.css`. Sprawdź w źródle strony, że CSS się ładuje.

### Zadanie 2: Lista i szczegóły

Lista słowników z Twojej domeny (5 elementów, pola: `id`, `name`, dwa inne, jedno logiczne). `/items/` z tabelą, `forloop.counter`, link `{% url %}` do szczegółów, wyróżnienie elementów z polem logicznym `False`. `/items/<int:item_id>/` ze szczegółami i `yesno`. Pusta lista → `{% empty %}`.

### Zadanie 3: Przepisz z Jinja2

Weź `product_list.html` ze swojego projektu Flask i przepisz na DTL. Wypisz w README wszystkie miejsca, które musiałeś zmienić (tabela z rozdziału 3 pomoże). Ile ich było?

### Zadanie 4: Formularz

`forms.py` z klasą dla Twojej domeny: minimum 5 pól, w tym `DecimalField` lub `IntegerField`, `ChoiceField`, `BooleanField(required=False)`, jedno `required=False`. Widok GET/POST według rozdziału 9. `{{ form.as_p }}`. Sprawdź w źródle strony, jaki HTML wygenerowało Django dla każdego pola.

### Zadanie 5: Walidacja wbudowana

Wyślij: pusty formularz, za krótką nazwę, ujemną liczbę, tekst w polu liczbowym (przez F12 zmień `type="number"` na `type="text"`). Zrób zrzut ekranu z błędami, wstaw do README. Potem usuń `required` przez F12 i wyślij pusty – co się stało i dlaczego?

### Zadanie 6: Walidacja własna

`clean_<pole>` z jedną regułą z Twojej domeny (np. nazwa nie może być „test”, e-mail musi być z domeny szkoły, data nie może być z przeszłości). `clean()` z jedną regułą między dwoma polami (np. cena promocyjna < cena, data końca > data początku). Przetestuj oba.

### Zadanie 7: CSRF

Usuń `{% csrf_token %}`, wyślij formularz, zrób zrzut ekranu 403. Przywróć. W README opisz własnymi słowami, na czym polega atak CSRF i jak token go blokuje. Trzy zdania, nie więcej.

### Zadanie 8: Wyszukiwarka

`SearchForm` z `q` i jednym checkboxem. Formularz GET w `base.html` albo nad listą. Po wysłaniu pola mają zostać wypełnione. Bez tokena – i wyjaśnij w README, dlaczego bez.

### Zadanie 9: Ręczny układ

Jedno pole formularza wyrenderuj ręcznie (`label_tag`, pole, `errors`) zamiast przez `as_p`. Nadaj mu klasę CSS przez `widget=forms.TextInput(attrs={"class": "wide"})`.

### Zadanie 10: Repo

Commity: `feat: add templates and base layout`, `feat: add product form with validation`, `feat: add search form`. README: struktura folderów `templates/shop/` i `static/shop/` z wyjaśnieniem podwójnego `shop`.

---

## 16. Pytania kontrolne

Sprawdź, czy opanowałeś materiał. Odpowiedz sobie sam, bez zaglądania.

1. Dlaczego szablony leżą w `shop/templates/shop/`, a nie w `shop/templates/`? Co się stanie, gdy dwie aplikacje mają `index.html`?
2. `render(request, "shop/index.html", {...})` – po co `request` jako pierwszy argument?
3. `{{ items[0] }}` w szablonie Django – co się stanie i jak to zapisać poprawnie?
4. Dlaczego DTL nie pozwala na `{{ price * 1.23 }}`? Gdzie to policzyć?
5. `{% url 'shop:product_detail' p.id %}` – co robi każdy element tego tagu?
6. Bez czego `{% static %}` daje błąd i jak brzmi ten błąd?
7. Wymień cztery rzeczy, które Django generuje z jednej klasy `forms.Form`.
8. `BooleanField` bez `required=False` – co to znaczy dla użytkownika?
9. Dlaczego `DecimalField`, a nie `FloatField`, do ceny?
10. Narysuj przepływ widoku z formularzem: GET, POST poprawny, POST z błędami. Który `render` obsługuje dwa z tych trzech przypadków?
11. Czym się różni `ProductForm()` od `ProductForm(request.POST)`? Jak się nazywają te dwa stany?
12. Kiedy wolno czytać `form.cleaned_data`? Co w nim jest dla pola `price`?
13. Użytkownik usunął `required` przez F12 i wysłał puste pole. Co zobaczy i dlaczego?
14. Jak nazwać metodę walidującą pole `email`? Co musi zwrócić? Co się stanie, jeśli nie zwróci nic?
15. W `clean()` używasz `cleaned.get("price")`, nie `cleaned["price"]`. Dlaczego?
16. `self.add_error("price", ...)` vs `raise ValidationError(...)` w `clean()` – gdzie wyświetli się komunikat w każdym przypadku?
17. Na czym polega atak CSRF? Dlaczego token go blokuje? Dlaczego formularz GET go nie potrzebuje?
18. Formularz wyszukiwania po wysłaniu ma wypełnione pola bez `value="{{ q }}"`. Jak to działa?
19. `forms.py` – która warstwa MTV? Co tam nie powinno się znaleźć?
20. Otwórz swoje `forms.py`, wskaż dowolne pole i powiedz: jaki HTML wygeneruje, jaki typ będzie w `cleaned_data`, jakie komunikaty błędów może dać.