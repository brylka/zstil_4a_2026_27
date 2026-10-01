# Django – modele, migracje, admin

**Zaawansowane aplikacje webowe | klasa 4 | materiał 12**

Odpowiednik materiału 07 z Flaska, ale z trzema rzeczami, których tam nie było: **migracje** (zamiast „usuń plik .db i uruchom od nowa”), **panel admina** (CRUD bez pisania jednej linii HTML) i **ModelForm** (formularz z materiału 11 skraca się do trzech linii). Po tym materiale masz pełne MTV i pełny CRUD – wszystko, co we Flasku, tylko krócej.

---

## Spis treści

1. Co Django robi za Ciebie tym razem
2. Pierwszy model
3. Migracje: jak zmiany w klasie trafiają do bazy
4. ORM w konsoli: CRUD
5. Zapytania: filter, exclude, order_by, lookupy
6. Relacja: ForeignKey
7. Panel admina
8. Widoki na modelach
9. ModelForm – formularz z modelu
10. Edycja i usuwanie
11. Gdzie teraz jest MTV
12. Najczęstsze problemy
13. Zadania
14. Pytania kontrolne

---

## 1. Co Django robi za Ciebie tym razem

We Flasku (materiał 07) instalowałeś SQLAlchemy, konfigurowałeś `SQLALCHEMY_DATABASE_URI`, pisałeś `db.create_all()`, a gdy dodałeś kolumnę – kasowałeś bazę. Do tego formularz ręcznie, lista ręcznie, edycja ręcznie.

Django: ORM jest wbudowany i **już skonfigurowany**. Zajrzyj do `settings.py`, sekcja `DATABASES` – SQLite, plik `db.sqlite3` obok `manage.py`. Nic nie zmieniasz. Pamiętasz „18 unapplied migrations” przy pierwszym `runserver`? To były tabele wbudowanych aplikacji czekające na utworzenie. Dziś je utworzymy – i Twoje też.

| | Flask + SQLAlchemy | Django |
|---|---|---|
| konfiguracja | `app.config[...]`, `db = SQLAlchemy(app)` | gotowe w `settings.py` |
| model | `class Product(db.Model)` | `class Product(models.Model)` |
| kolumna | `db.Column(db.String(100))` | `models.CharField(max_length=100)` |
| utworzenie tabel | `db.create_all()` | `makemigrations` + `migrate` |
| zmiana kolumny | usuń bazę, od nowa | `makemigrations` + `migrate` – dane zostają |
| zapytania | `Product.query.filter_by(...)` | `Product.objects.filter(...)` |
| 404 | `get_or_404()` | `get_object_or_404()` |
| formularz z modelu | ręcznie | `ModelForm` – 3 linie |
| panel do edycji danych | brak | `/admin/` – 5 linii |

---

## 2. Pierwszy model

`shop/models.py`:

```python
from decimal import Decimal

from django.db import models


class Product(models.Model):
    name = models.CharField("nazwa", max_length=100)
    price = models.DecimalField("cena", max_digits=8, decimal_places=2)
    is_available = models.BooleanField("dostępny", default=True)
    description = models.TextField("opis", blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        verbose_name = "produkt"
        verbose_name_plural = "produkty"
        ordering = ["name"]

    def __str__(self):
        return self.name

    def gross_price(self):
        return (self.price * Decimal("1.23")).quantize(Decimal("0.01"))
```

Linia po linii:

| Element | Co robi |
|---|---|
| `class Product(models.Model)` | klasa = tabela. Nazwa tabeli: `shop_product` (aplikacja_model, małe litery) |
| `models.CharField("nazwa", max_length=100)` | kolumna `VARCHAR(100)`. Pierwszy argument to **etykieta** (verbose name) – pokaże się w adminie i w formularzu. `max_length` obowiązkowy |
| `DecimalField(max_digits=8, decimal_places=2)` | do 8 cyfr, 2 po przecinku: max `999999.99`. Do pieniędzy – nigdy `FloatField` |
| `BooleanField(default=True)` | `default` – wartość, gdy nie podasz |
| `TextField(blank=True)` | tekst bez limitu. `blank=True` = pole może być puste **w formularzu**. (`null=True` = może być `NULL` **w bazie** – do tekstów nie używaj, pusty string wystarczy) |
| `DateTimeField(auto_now_add=True)` | ustawia się raz, przy tworzeniu. `auto_now=True` – przy każdym zapisie. Oba nie pokazują się w formularzu |
| `class Meta` | ustawienia modelu, nie kolumny. `ordering` – domyślne sortowanie każdego zapytania. `verbose_name` – jak admin ma nazywać ten model |
| `__str__` | jak obiekt wyświetla się w adminie, w konsoli, w `{{ product }}` w szablonie. **Zawsze** definiuj |
| `gross_price()` | logika w modelu – dokładnie jak w materiale 07. W szablonie: `{{ p.gross_price }}` bez nawiasów |

Nie ma `id`. Django dodaje go sam: `id = BigAutoField(primary_key=True)`. Dostęp: `product.id` albo `product.pk` (primary key – działa też, gdy ktoś nazwie klucz inaczej; przyzwyczaj się do `pk`).

Najczęstsze typy pól:

| Pole | SQL | Kiedy |
|---|---|---|
| `CharField(max_length=…)` | `VARCHAR` | krótki tekst: nazwa, tytuł, e-mail (`EmailField`) |
| `TextField()` | `TEXT` | długi tekst: opis, treść |
| `IntegerField()`, `PositiveIntegerField()` | `INTEGER` | liczby całkowite, ilość |
| `DecimalField(max_digits, decimal_places)` | `DECIMAL` | pieniądze |
| `BooleanField()` | `BOOL` | tak/nie |
| `DateField()`, `DateTimeField()` | `DATE`, `DATETIME` | daty |
| `SlugField()` | `VARCHAR` | fragment adresu: `laptop-dell-15` |
| `ForeignKey(Model, on_delete=…)` | `INTEGER` + klucz obcy | relacja „należy do” – rozdział 6 |
| `ImageField()`, `FileField()` | `VARCHAR` (ścieżka) | pliki – później |

---

## 3. Migracje: jak zmiany w klasie trafiają do bazy

Napisałeś klasę. Baza o niej nie wie. **Migracja** to plik, który opisuje, co zmienić w bazie, żeby odpowiadała klasom. Dwa kroki, zawsze w tej kolejności:

```powershell
python manage.py makemigrations
```

```text
Migrations for 'shop':
  shop/migrations/0001_initial.py
    + Create model Product
```

Django porównał `models.py` z tym, co wie o bazie, i **wygenerował plik** `shop/migrations/0001_initial.py`. Otwórz go. Zobaczysz Pythona opisującego tabelę: `CreateModel`, nazwa, lista pól. Nie edytujesz go – ale masz wiedzieć, co tam jest.

```powershell
python manage.py migrate
```

```text
Applying admin.0001_initial... OK
Applying auth.0001_initial... OK
...
Applying shop.0001_initial... OK
```

Teraz Django **wykonał** migracje – Twoją i te 18 wbudowanych. Pojawił się plik `db.sqlite3`. Otwórz go w DB Browser for SQLite: tabela `shop_product` z Twoimi kolumnami, plus `auth_user`, `django_session` i inne.

### 3.1 Zmiana modelu

Dodaj pole do klasy:

```python
    stock = models.PositiveIntegerField("stan magazynowy", default=0)
```

```powershell
python manage.py makemigrations
```

```text
Migrations for 'shop':
  shop/migrations/0002_product_stock.py
    + Add field stock to product
```

```powershell
python manage.py migrate
```

Kolumna dodana. **Dane w tabeli zostały.** We Flasku kasowałeś bazę. Tu zmiana bazy jest kodem, ma numer, jest w Gicie, można ją cofnąć (`migrate shop 0001`) i wykonać na serwerze produkcyjnym bez utraty danych. To jest powód, dla którego migracje istnieją.

### 3.2 Zasady

- `makemigrations` po **każdej** zmianie w `models.py`. `migrate` zaraz po.
- Folder `migrations/` **idzie do Gita**. To historia bazy. Bez niego kolega po `git clone` nie odtworzy tabel.
- `db.sqlite3` **nie idzie** do Gita. To dane, nie kod.
- Pole bez `default` dodane do tabeli z danymi: Django zapyta, co wpisać w istniejące wiersze. Odpowiedz `1` i podaj wartość, albo dodaj `default=` w modelu.
- Zobaczyć SQL, który migracja wykona: `python manage.py sqlmigrate shop 0001`. Zobaczysz `CREATE TABLE "shop_product" (...)` – dokładnie to, co pisałeś ręcznie na INF.03.
- Stan migracji: `python manage.py showmigrations shop` – `[X]` wykonana, `[ ]` czeka.

> ⚠️ Nie usuwaj plików z `migrations/`, nie zmieniaj ich numerów, nie edytuj ręcznie. Jak coś się posypie na etapie nauki: usuń `db.sqlite3` **i** wszystkie `000x_*.py` (nie `__init__.py`!), potem `makemigrations` + `migrate` od nowa. Na produkcji tego się nie robi – ale Ty nie masz jeszcze produkcji.

---

## 4. ORM w konsoli: CRUD

```powershell
python manage.py shell
```

To konsola Pythona z załadowanym Django. Odpowiednik `flask shell`.

```python
>>> from shop.models import Product
>>> from decimal import Decimal
```

### Create

```python
>>> p = Product(name="Laptop", price=Decimal("2999.00"))
>>> p.save()
>>> p.pk
1

>>> Product.objects.create(name="Mysz", price=49, is_available=False)
<Product: Mysz>
```

Dwa sposoby: obiekt + `save()`, albo `create()` (robi oba naraz). Nie ma `session.add` / `commit` – `save()` zapisuje od razu.

### Read

```python
>>> Product.objects.all()
<QuerySet [<Product: Laptop>, <Product: Mysz>]>

>>> Product.objects.get(pk=1)
<Product: Laptop>

>>> Product.objects.get(name="Mysz")
<Product: Mysz>

>>> Product.objects.get(pk=99)
DoesNotExist: Product matching query does not exist.

>>> Product.objects.count()
2
```

`objects` to **menedżer** – brama do zapytań. Odpowiednik `query` z SQLAlchemy. `all()` zwraca **QuerySet** – leniwą listę: zapytanie do bazy poleci dopiero, gdy coś z niej weźmiesz (pętla, `list()`, `len()`, szablon). `get()` zwraca jeden obiekt albo rzuca wyjątek – dwa wyjątki: `DoesNotExist` (brak) i `MultipleObjectsReturned` (za dużo). Do `get()` używaj pól unikalnych.

### Update

```python
>>> p = Product.objects.get(pk=1)
>>> p.price = Decimal("2799")
>>> p.save()
```

Jak w SQLAlchemy: zmień atrybut, zapisz. Hurtowo, bez wczytywania obiektów:

```python
>>> Product.objects.filter(is_available=False).update(price=0)
1
```

### Delete

```python
>>> Product.objects.get(pk=2).delete()
(1, {'shop.Product': 1})
```

Zwraca, ile wierszy usunęło i z jakich tabel (przy relacjach może być więcej).

> 💡 Zobacz SQL, który ORM wygenerował: `print(Product.objects.filter(is_available=True).query)`. Tak wygląda mostek między klasą a `SELECT ... FROM shop_product WHERE ...`.

---

## 5. Zapytania: filter, exclude, order_by, lookupy

```python
Product.objects.filter(is_available=True)            # WHERE is_available
Product.objects.exclude(is_available=True)           # WHERE NOT is_available
Product.objects.filter(price__lt=100)                # WHERE price < 100
Product.objects.filter(price__gte=100, price__lte=500)   # AND
Product.objects.filter(name__icontains="lap")        # WHERE name LIKE '%lap%' (bez wielkości liter)
Product.objects.filter(name__startswith="Lap")
Product.objects.filter(pk__in=[1, 2, 3])
Product.objects.order_by("price")                    # ORDER BY price
Product.objects.order_by("-price")                   # DESC
Product.objects.filter(is_available=True).order_by("-price")[:5]   # LIMIT 5
Product.objects.filter(name="Mysz").exists()         # True / False, bez pobierania
Product.objects.filter(name="Mysz").first()          # obiekt albo None – bez wyjątku
Product.objects.values_list("name", "price")         # lista krotek zamiast obiektów
```

**Lookup** to `__` po nazwie pola: `pole__operator`. Najczęstsze: `exact` (domyślny), `iexact`, `contains`, `icontains`, `startswith`, `endswith`, `lt`, `lte`, `gt`, `gte`, `in`, `isnull`, `range`. Litera `i` na początku = ignoruj wielkość liter.

QuerySety można **łańcuchować**: `filter().exclude().order_by()` – każde wywołanie zwraca nowy QuerySet, baza dostaje jedno zapytanie na końcu.

Tabela z podstawy INF.04 „stosuje mechanizm zapytań do bazy” – to jest ten mechanizm. SQL i tak znasz; ORM go pisze za Ciebie i chroni przed SQL injection, bo wartości nigdy nie są sklejane ze stringiem.

---

## 6. Relacja: ForeignKey

Kategoria produktu jako osobna tabela, nie string. `shop/models.py`, **przed** `Product`:

```python
class Category(models.Model):
    name = models.CharField("nazwa", max_length=50, unique=True)
    slug = models.SlugField(unique=True)

    class Meta:
        verbose_name = "kategoria"
        verbose_name_plural = "kategorie"
        ordering = ["name"]

    def __str__(self):
        return self.name
```

W `Product` dodaj pole:

```python
    category = models.ForeignKey(
        Category, on_delete=models.PROTECT, related_name="products", verbose_name="kategoria"
    )
```

`makemigrations` – Django zapyta o wartość domyślną dla istniejących produktów (nie mają kategorii). Na etapie nauki prościej: usuń bazę i migracje, zrób od nowa. Potem `migrate`.

| Argument | Co robi |
|---|---|
| `Category` | do jakiego modelu. Musi być zdefiniowany wyżej, albo podaj jako string `"Category"` |
| `on_delete=models.PROTECT` | co zrobić z produktami, gdy ktoś usuwa kategorię. `PROTECT` – nie pozwól (`ProtectedError`). `CASCADE` – usuń produkty razem z nią. `SET_NULL` – wyzeruj (wymaga `null=True`). **Obowiązkowy**, Django nie zgadnie |
| `related_name="products"` | jak dojść z kategorii do jej produktów: `category.products.all()`. Bez tego byłoby `category.product_set.all()` |

W konsoli:

```python
>>> from shop.models import Category, Product
>>> lap = Category.objects.create(name="Laptopy", slug="laptops")
>>> acc = Category.objects.create(name="Akcesoria", slug="accessories")
>>> Product.objects.create(name="Laptop", price=2999, category=lap)
>>> Product.objects.create(name="Mysz", price=49, category=acc)

>>> p = Product.objects.get(name="Mysz")
>>> p.category                      # w jedną stronę
<Category: Akcesoria>
>>> p.category.name
'Akcesoria'

>>> acc.products.all()              # w drugą stronę (related_name)
<QuerySet [<Product: Mysz>]>

>>> Product.objects.filter(category__slug="laptops")    # lookup przez relację
<QuerySet [<Product: Laptop>]>

>>> lap.delete()
ProtectedError: Cannot delete some instances of model 'Category' because they are referenced...
```

W bazie: kolumna `category_id` w `shop_product` z kluczem obcym do `shop_category.id`. Dokładnie to, co rysowałeś na diagramach ERD.

> ⚠️ Pętla po produktach z `{{ p.category.name }}` w szablonie = jedno zapytanie na produkt (problem **N+1**). Rozwiązanie: `Product.objects.select_related("category")` – jeden `JOIN` zamiast N zapytań. Django 6.1 dodało też `fetch_mode(FETCH_PEERS)` – ale `select_related` wystarczy na start.

---

## 7. Panel admina

Django ma gotowy panel do zarządzania danymi. Włączenie to dwa kroki.

### 7.1 Superużytkownik

```powershell
python manage.py createsuperuser
```

Login, e-mail (może być pusty), hasło dwa razy. Django sprawdzi, czy hasło nie jest za proste – w projekcie szkolnym możesz wymusić `y`.

### 7.2 Rejestracja modeli

`shop/admin.py`:

```python
from django.contrib import admin

from .models import Category, Product


@admin.register(Category)
class CategoryAdmin(admin.ModelAdmin):
    prepopulated_fields = {"slug": ("name",)}


@admin.register(Product)
class ProductAdmin(admin.ModelAdmin):
    list_display = ["name", "category", "price", "is_available", "updated_at"]
    list_filter = ["category", "is_available"]
    search_fields = ["name"]
    list_editable = ["price", "is_available"]
```

Otwórz http://127.0.0.1:8000/admin/ i zaloguj się. Masz:

- listę produktów z kolumnami z `list_display`,
- filtry po prawej z `list_filter`,
- wyszukiwarkę z `search_fields`,
- edycję ceny i dostępności **bezpośrednio na liście** z `list_editable`,
- formularz dodawania/edycji z walidacją – wygenerowany z modelu,
- przy kategorii: slug wypełnia się sam z nazwy (`prepopulated_fields`).

Wszystko po polsku, bo `LANGUAGE_CODE = "pl"` i `verbose_name` w modelach. Zero HTML-a, zero widoków.

Minimum to samo `admin.site.register(Product)` – jedna linia, bez klasy. Ale `list_display` i `search_fields` są warte tych pięciu linii więcej.

> 💡 To jest powód, dla którego Django powstał w redakcji gazety: dziennikarze dostali panel do wpisywania artykułów pierwszego dnia. Ty dostajesz panel do wpisywania danych testowych – **przestań tworzyć produkty w konsoli**, wpisuj je w adminie.

Admin jest dla **Ciebie i zespołu**, nie dla użytkowników sklepu. Użytkownicy dostają Twoje widoki. Admin to zaplecze.

---

## 8. Widoki na modelach

Lista słowników `PRODUCTS` z materiału 11 znika. `shop/views.py`:

```python
from django.shortcuts import get_object_or_404, redirect, render

from .forms import ProductForm, SearchForm
from .models import Product


def product_list(request):
    form = SearchForm(request.GET)
    products = Product.objects.select_related("category")
    if form.is_valid():
        q = form.cleaned_data["q"]
        if q:
            products = products.filter(name__icontains=q)
        if form.cleaned_data["only_available"]:
            products = products.filter(is_available=True)
    return render(request, "shop/product_list.html", {"products": products, "form": form})


def product_detail(request, product_id):
    product = get_object_or_404(Product, pk=product_id)
    return render(request, "shop/product_detail.html", {"product": product})
```

- `Product.objects.select_related("category")` – QuerySet, jeszcze nie zapytanie. Kolejne `filter()` go zawężają. Baza dostaje jeden `SELECT ... JOIN ... WHERE ...` dopiero w szablonie.
- `get_object_or_404(Product, pk=product_id)` – odpowiednik `get_or_404`. Obiekt albo 404, jedna linia.

Szablony **prawie** się nie zmieniają – `{{ p.name }}` działa na obiekcie tak samo jak na słowniku. Tylko `p.id` → `p.pk` (nawyk) i można dopisać `{{ p.category }}` (wyświetli `__str__`) i `{{ p.gross_price }}`.

---

## 9. ModelForm – formularz z modelu

W materiale 11 opisałeś każde pole formularza ręcznie: typ, etykieta, `max_length`, `min_value`. Ale przecież to samo jest już w modelu. `ModelForm` bierze pola z modelu:

`shop/forms.py`:

```python
from django import forms

from .models import Product


class ProductForm(forms.ModelForm):
    class Meta:
        model = Product
        fields = ["name", "price", "category", "is_available", "description"]

    def clean_name(self):
        name = self.cleaned_data["name"].strip()
        if name.lower() in {"test", "asdf"}:
            raise forms.ValidationError("Wpisz prawdziwą nazwę produktu.")
        return name
```

Trzy linie w `Meta` zastąpiły pięć definicji pól. Z modelu Django wie: `name` to `CharField(max_length=100)` → input z `maxlength`; `price` to `DecimalField` → `type="number" step="0.01"`; `category` to `ForeignKey` → **`<select>` z kategoriami z bazy**; `is_available` ma `default=True` → checkbox zaznaczony; `description` ma `blank=True` → niewymagane. Etykiety z `verbose_name`.

`fields` – wypisz jawnie. `fields = "__all__"` działa, ale pewnego dnia dodasz do modelu pole `is_admin` i zapomnisz, że formularz je pokazuje. `created_at`/`updated_at` z `auto_now` i tak się nie pokażą.

Własna walidacja (`clean_<pole>`, `clean()`) działa identycznie jak w `forms.Form`. Wszystko z materiału 11 zostaje.

Widok dodawania:

```python
def product_add(request):
    if request.method == "POST":
        form = ProductForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect("shop:product_list")
    else:
        form = ProductForm()
    return render(request, "shop/product_form.html", {"form": form, "title": "Nowy produkt"})
```

Jedna zmiana względem materiału 11: `form.save()` zamiast `PRODUCTS.append(...)`. `ModelForm` wie, jaki model, więc tworzy obiekt i zapisuje. Zwraca ten obiekt – `product = form.save()` jeśli go potrzebujesz (np. do `redirect` na szczegóły).

---

## 10. Edycja i usuwanie

### 10.1 Edycja

Ten sam formularz, ten sam szablon, jeden nowy argument: `instance`.

```python
def product_edit(request, product_id):
    product = get_object_or_404(Product, pk=product_id)
    if request.method == "POST":
        form = ProductForm(request.POST, instance=product)
        if form.is_valid():
            form.save()
            return redirect("shop:product_detail", product_id=product.pk)
    else:
        form = ProductForm(instance=product)
    return render(request, "shop/product_form.html", {"form": form, "title": f"Edycja: {product.name}"})
```

`ProductForm(instance=product)` – formularz **wypełniony** danymi produktu (GET). `ProductForm(request.POST, instance=product)` – dane z formularza **nadpisują** ten produkt, nie tworzą nowego (POST). Bez `instance` przy POST dostałbyś duplikat.

`shop/urls.py`:

```python
    path("products/<int:product_id>/edit/", views.product_edit, name="product_edit"),
    path("products/<int:product_id>/delete/", views.product_delete, name="product_delete"),
```

Szablon `product_form.html` ma `<h1>{{ title }}</h1>` – jeden szablon na dodawanie i edycję.

### 10.2 Usuwanie

```python
from django.views.decorators.http import require_POST


@require_POST
def product_delete(request, product_id):
    product = get_object_or_404(Product, pk=product_id)
    product.delete()
    return redirect("shop:product_list")
```

`@require_POST` – GET na ten adres dostanie **405 Method Not Allowed**, widok się nie uruchomi. Zasada z materiału 08 wymuszona dekoratorem, nie dobrą wolą. W szablonie listy:

```html
<form method="post" action="{% url 'shop:product_delete' p.pk %}" style="display:inline">
    {% csrf_token %}
    <button>Usuń</button>
</form>
```

Formularz, nie link. Z tokenem, bo POST.

### 10.3 Lista – pełna wersja

`shop/templates/shop/product_list.html`:

```html
{% extends "shop/base.html" %}
{% block title %}Produkty – Sklep 4TP{% endblock %}
{% block content %}
<h1>Produkty ({{ products|length }})</h1>

<form method="get">
    {{ form.as_p }}
    <button>Szukaj</button>
</form>

<table>
    <tr><th>#</th><th>Nazwa</th><th>Kategoria</th><th>Cena</th><th>Brutto</th><th>Status</th><th></th></tr>
    {% for p in products %}
    <tr>
        <td>{{ forloop.counter }}</td>
        <td><a href="{% url 'shop:product_detail' p.pk %}">{{ p.name }}</a></td>
        <td>{{ p.category }}</td>
        <td>{{ p.price }} zł</td>
        <td>{{ p.gross_price }} zł</td>
        <td>{{ p.is_available|yesno:"dostępny,brak" }}</td>
        <td>
            <a href="{% url 'shop:product_edit' p.pk %}">Edytuj</a>
            <form method="post" action="{% url 'shop:product_delete' p.pk %}" style="display:inline">
                {% csrf_token %}
                <button>Usuń</button>
            </form>
        </td>
    </tr>
    {% empty %}
    <tr><td colspan="7">Brak produktów.</td></tr>
    {% endfor %}
</table>
{% endblock %}
```

---

## 11. Gdzie teraz jest MTV

| Warstwa | Plik | Co tam jest |
|---|---|---|
| **Model** | `models.py` | `Product`, `Category`, `gross_price()`, relacja |
| **Template** | `templates/shop/*.html` | wyświetlanie |
| **View** | `views.py` | `objects.filter`, `get_object_or_404`, `form.save()`, `redirect` |
| formularz | `forms.py` | `ModelForm` + własna walidacja |
| routing | `urls.py` | mapa adresów |
| migracje | `migrations/` | historia schematu bazy |
| admin | `admin.py` | zaplecze |

Komplet. Porównaj z tabelą z materiału 07: to samo, tylko każda warstwa ma swój plik, którego nie musiałeś wymyślać. `views.py` ma pięć funkcji po 5–8 linii. Cała logika jest w modelu, cała walidacja w formularzu, cały HTML w szablonach.

Test z materiału 03: „czy dałoby się do tego modelu dopiąć aplikację na Androida zamiast szablonów?” Tak – i to jest następny etap: Django REST Framework, gdzie `views.py` zamiast `render` zwraca JSON, a `forms.py` zamienia się w `serializers.py`. Modele, migracje, admin – zostają bez zmian.

---

## 12. Najczęstsze problemy

| Objaw | Przyczyna i rozwiązanie |
|---|---|
| `no such table: shop_product` | nie zrobiłeś `migrate` (albo `makemigrations` przed nim) |
| `You have N unapplied migration(s)` | `python manage.py migrate` |
| `makemigrations` nic nie wykrywa | aplikacja nie w `INSTALLED_APPS`, albo nie zapisałeś `models.py` |
| `It is impossible to add a non-nullable field ... without specifying a default` | dodajesz pole do tabeli z danymi. Wybierz `1` i podaj wartość, albo dodaj `default=` / `blank=True` |
| `DoesNotExist` w widoku | `Product.objects.get(pk=99)` – użyj `get_object_or_404` |
| `MultipleObjectsReturned` | `get()` na polu, które nie jest unikalne – użyj `filter().first()` |
| `ProtectedError` przy usuwaniu kategorii | `on_delete=PROTECT` działa. Najpierw usuń/przepnij produkty, albo zmień na `CASCADE`, jeśli to zamierzone |
| `<Product: Product object (1)>` w adminie | brak `__str__` |
| `ForeignKey ... requires on_delete` | argument obowiązkowy |
| `select` z kategoriami pusty w formularzu | nie ma kategorii w bazie – dodaj w adminie |
| edycja tworzy nowy produkt zamiast zmieniać | brak `instance=product` w `ProductForm(request.POST, ...)` |
| 405 przy kliknięciu „Usuń” | `@require_POST` działa, a Ty masz link `<a>` zamiast formularza POST |
| lista wolno się ładuje przy 100 produktach | N+1: `{{ p.category }}` bez `select_related("category")` |
| `db.sqlite3` na GitHubie | `.gitignore`, `git rm --cached db.sqlite3` |
| `migrations/` **nie ma** na GitHubie | ktoś dopisał go do `.gitignore`. Usuń ten wpis – migracje to kod |

---

## 13. Zadania

### Zadanie 1: Model

Zastąp listę słowników z materiału 11 klasą `models.Model` z Twojej domeny. Minimum 5 pól, w tym `DecimalField` lub `IntegerField`, `BooleanField` z `default`, `TextField(blank=True)`, `created_at`. `Meta` z `ordering` i `verbose_name`. `__str__`. Jedna metoda z logiką (jak `gross_price`).

### Zadanie 2: Migracje

`makemigrations`, otwórz wygenerowany plik i przeczytaj. `sqlmigrate` – wklej wygenerowany `CREATE TABLE` do README. `migrate`. Otwórz `db.sqlite3` w DB Browser, zrób zrzut ekranu tabeli.

### Zadanie 3: Zmiana modelu

Dodaj pole (z `default`). `makemigrations` → drugi plik. `migrate`. Sprawdź w DB Browser, że dane z zadania 4 (zrób je najpierw) zostały. Potem `showmigrations` – wklej wynik do README.

### Zadanie 4: CRUD w konsoli

W `manage.py shell`: utwórz 3 obiekty (raz przez `save()`, raz przez `create()`), `all()`, `get()`, `filter()` z dwoma lookupami, `order_by("-…")`, `update()` hurtowo, `delete()`. Wypisz `.query` jednego zapytania. Przebieg sesji do README.

### Zadanie 5: Relacja

Drugi model (kategoria / autor / sala / typ ogłoszenia) i `ForeignKey` z `on_delete=PROTECT` i `related_name`. W konsoli: dojście w obie strony, lookup przez relację (`category__slug`), próba usunięcia „rodzica” z dziećmi.

### Zadanie 6: Admin

`createsuperuser`. Zarejestruj oba modele z `list_display`, `list_filter`, `search_fields`, `list_editable`. Wpisz przez admina 5 rekordów. Zrzut ekranu listy do README.

### Zadanie 7: Widoki na modelach

Lista i szczegóły z bazy. `select_related` w liście. `get_object_or_404` w szczegółach. Wyszukiwarka z materiału 11 przerobiona na `filter(name__icontains=q)`.

### Zadanie 8: ModelForm

Przerób `forms.Form` z materiału 11 na `ModelForm`. Policz linie przed i po – wpisz do README. Własna walidacja `clean_<pole>` zostaje. Widok dodawania z `form.save()`.

### Zadanie 9: Edycja i usuwanie

`product_edit` z `instance`, ten sam szablon co dodawanie, `title` w kontekście. `product_delete` z `@require_POST` i formularzem w liście. Sprawdź: GET na `/…/delete/` → 405; edycja bez `instance` → co się dzieje?

### Zadanie 10: Repo

`.gitignore` z `db.sqlite3`, **bez** `migrations/`. Commity: `feat: add Product and Category models`, `feat: register models in admin`, `feat: migrate views to ORM`, `refactor: replace Form with ModelForm`, `feat: add product edit and delete`. README: tabela tras (metoda, adres, nazwa, opis), struktura MTV z rozdziału 11.

### Zadanie 11: Dla chętnych

Trzeci model z relacją do drugiego (np. `Order` → `Product` przez `ManyToManyField`, albo `Review` → `Product` przez `ForeignKey` z `CASCADE`). Inline w adminie: `class ReviewInline(admin.TabularInline)` w `ProductAdmin.inlines`.

---

## 14. Pytania kontrolne

Sprawdź, czy opanowałeś materiał. Odpowiedz sobie sam, bez zaglądania.

1. Jak nazywa się tabela dla modelu `Product` w aplikacji `shop`? Skąd `id`, skoro go nie napisałeś?
2. `blank=True` vs `null=True` – co każde oznacza i którego użyć do `TextField`?
3. `auto_now_add` vs `auto_now` – kiedy każde się ustawia?
4. Po co `__str__`? Co zobaczysz bez niego w adminie i w `{{ product }}`?
5. `makemigrations` i `migrate` – co robi każde? Co powstaje po pierwszym, a co po drugim?
6. Dlaczego `migrations/` idzie do Gita, a `db.sqlite3` nie?
7. Dodałeś pole bez `default` do tabeli z danymi. Co Django zapyta i dlaczego?
8. `Product.objects.all()` – kiedy leci zapytanie do bazy? Co to jest QuerySet?
9. `get()` vs `filter().first()` – jakie wyjątki może rzucić `get()` i kiedy?
10. Napisz lookup: cena od 100 do 500, nazwa zawiera „lap” bez wielkości liter, posortowane malejąco po cenie, pierwsze 5.
11. `on_delete` – trzy opcje i kiedy którą. Dlaczego jest obowiązkowy?
12. `related_name="products"` – co daje? Jak wyglądałoby bez niego?
13. Co to problem N+1 i jak go rozwiązuje `select_related`?
14. Czym różni się `ModelForm` od `forms.Form`? Co dostajesz z modelu, a co musisz dopisać?
15. Dlaczego `fields = [...]` jawnie, a nie `"__all__"`?
16. `ProductForm(request.POST, instance=product)` – co robi `instance`? Co się stanie bez niego przy edycji?
17. `form.save()` – co robi w `ModelForm`? Co zwraca?
18. `@require_POST` – jaki kod dla GET i dlaczego to lepsze niż sprawdzanie `request.method` w widoku?
19. Admin: dla kogo jest, a dla kogo nie? Co dają `list_display`, `list_filter`, `search_fields`, `list_editable`?
20. Wskaż w swoim projekcie plik dla każdej warstwy z tabeli w rozdziale 11. Która warstwa zna SQL? Która HTML? Która obie? (Podpowiedź: żadna nie powinna znać obu.)