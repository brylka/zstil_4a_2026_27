# Nie rozumiesz kodu – nie oddawaj

Ściąga: jak pytać LLM, żeby się nauczyć, a jak pytać, żeby dostać jedynkę.

---

## Zasada

LLM wolno. Zawsze. Nie pytam, kto napisał kod. Pytam, **co ten kod robi**.

Różnica między piątką a jedynką nie jest w tym, czy użyłeś Gemini / Claude / ChatGPT. Jest w tym, **co mu napisałeś**. Ten sam model, ten sam kod, dwa różne prompty – jeden uczeń wychodzi z zajęć mądrzejszy, drugi z jedynką.

Jedynka nie jest za LLM. Jedynka jest za oddanie czegoś, czego nie rozumiesz. To jest to samo, co oddać spisaną od kolegi pracę i nie umieć jej przeczytać.

---

## Złe prompty

Rozpoznasz je po tym, że po odpowiedzi masz **kod, ale nie masz wiedzy**.

| Prompt | Co dostaniesz | Co się stanie na odpytywaniu |
|---|---|---|
| `Zrób zadanie 3.` | działający kod | „Co robi linia 12?” – cisza |
| `Napisz mi aplikację Flask ze sklepem.` | 300 linii, z czego rozumiesz zero | ja pytam o `get_or_404`, Ty patrzysz w sufit |
| `Nauczyciel pozwolił na LLM, zrób mi to zadanie, będę miał czas na TikToka.` | kod, może z docinką od modelu | TikTok był, wiedzy nie było, jedynka jest |
| `Popraw błąd.` (wklejony traceback) | poprawiony kod | nie wiesz, co było zepsute ani dlaczego działa |
| `Dodaj usuwanie produktu.` | trasa z usuwaniem, pewnie przez GET | „Czemu link, a nie POST?” – nie wiesz nawet, że jest link |
| `Wyjaśnij ten kod.` (bez kontekstu, bez pytania) | ogólny wykład, którego nie przeczytasz | jak wyżej |

Zauważ: te prompty **działają**. Model chętnie zrobi zadanie. Problem nie jest w modelu. Problem jest w tym, że Ty nie zrobiłeś nic.

Czasem model sam Cię pouczy („zamiast robić za Ciebie, wyjaśnię…”). Nie licz na to. Jego zadaniem jest odpowiedzieć na to, o co prosisz. Jak prosisz o gotowca, dostajesz gotowca.

---

## Dobre prompty

Rozpoznasz je po tym, że po odpowiedzi **umiesz coś, czego nie umiałeś przed nią**. Możesz je kopiować dosłownie, humor jest gratis.

### Zanim cokolwiek napiszesz

```
Mam napisać trasę Flask, która dodaje produkt przez formularz POST.
Nie pisz kodu. Powiedz, jakie kroki muszą się wydarzyć i jakich funkcji Flaska użyję.
```

```
Jak działa dekorator @app.route? Wyjaśnij na przykładzie z dwiema trasami,
potem zadaj mi jedno pytanie sprawdzające, czy zrozumiałem.
```

```
Chcę zrobić X. Pokaż mi dwa sposoby i powiedz, który wybrałbyś w projekcie szkolnym i dlaczego.
```

### Po wygenerowaniu kodu (albo po napisaniu własnego)

```
Zaraz zapyta mnie mój upierdliwy nauczyciel o ten kod. Wyjaśnij mi linia po linii,
ale krótko, tak żebym umiał to powtórzyć bez kartki.
```

```
Mój nauczyciel chce mnie zagiąć na tym kodzie. Wyjaśnij mi go tak, żebym to ja zagiął jego.
Jakie podchwytliwe pytania może zadać i co odpowiedzieć?
```

```
W tym kodzie jest get_or_404. Co się stanie, jak zamienię na get()? Pokaż różnicę na przykładzie.
```

```
Co się stanie, jak usunę linię z redirect po POST? Nie mów „będzie źle”, powiedz konkretnie co.
```

```
Przepisz ten kod prościej, bez żadnej magii. Potem powiedz, co straciłem przez uproszczenie.
```

```
Który fragment tego kodu jest modelem, który widokiem, który kontrolerem? Uzasadnij.
```

### Kiedy nie działa

```
Mam ten błąd: [traceback]. Nie naprawiaj. Powiedz, co oznacza komunikat i w której linii szukać.
```

```
Naprawiłeś błąd. Teraz wyjaśnij, co było zepsute, dlaczego to nie działało i jak mam to rozpoznać następnym razem.
```

```
Mój kod działa, ale nie wiem dlaczego. To mnie niepokoi. Wytłumacz.
```

### Przed odpytywaniem

```
Jesteś moim nauczycielem programowania. Masz mój kod (poniżej). Zadaj mi 5 pytań,
na które musiałbym znać odpowiedź, gdybym naprawdę to napisał. Nie podawaj odpowiedzi.
Czekaj na moje.
```

```
Odpowiedziałem na Twoje pytania tak: [odpowiedzi]. Oceń, co jest dobrze, co słabo,
gdzie bym poległ na prawdziwym odpytywaniu.
```

```
Zrób mi 3 wersje mojego kodu z jednym celowym błędem każda. Mam znaleźć, gdzie.
```

### Przy commicie

```
Jakie pliki zmieniłem i co w nich? Zaproponuj komunikat commita w Conventional Commits.
Ale najpierw powiedz, czy ten commit nie jest za duży i nie powinien być podzielony.
```

---

## Test: czy Twój prompt jest dobry?

Zadaj sobie jedno pytanie: **czy po tej odpowiedzi będę wiedział coś, czy tylko będę miał coś?**

- „Zrób X” → będę miał. Zły.
- „Wyjaśnij X” → będę wiedział. Dobry.
- „Zrób X, potem wyjaśnij, co zrobiłeś, potem zapytaj mnie o to” → będę miał **i** wiedział. Najlepszy.

Trzeci wariant to jest to, jak pracuje programista z agentem w 2026. Nie pisze wszystkiego ręcznie – bo po co. Ale nie oddaje niczego, czego nie przeczytał – bo to on odpowiada za kod, nie model.

---

## Jak to wygląda w praktyce (jedna lekcja)

1. Zadanie: dodać edycję produktu.
2. `Nie pisz kodu. Jakie kroki, jakie trasy, jakie metody HTTP?` → czytasz, rozumiesz plan.
3. `OK, zrób to.` → patrzysz na diff, **zanim** zatwierdzisz.
4. `Linia z request.form – co się stanie, jak pole będzie puste?` → poprawka, rozumiesz dlaczego.
5. `Zadaj mi 3 pytania o ten kod.` → odpowiadasz. Jak nie umiesz – wracasz do 4.
6. Commit. Push.
7. Na zajęciach pytam. Odpowiadasz. Piątka.

Czas: może 15 minut dłużej niż „zrób zadanie”. Za to jedynki nie ma, a na następnym zadaniu krok 2 robisz już sam.

---

## Dla agenta

Wklej do `AGENTS.md` (albo `CLAUDE.md` / `GEMINI.md`) w projekcie. Agent będzie Cię pilnował, nawet jak zapomnisz.

```markdown
# Zasady pracy w tym projekcie

- To jest projekt szkolny. Uczę się. Nie rób za mnie – rób ze mną.
- Po każdej zmianie w kodzie: krótko wyjaśnij, co zmieniłeś i dlaczego.
- Jeśli proszę „zrób X” bez „wyjaśnij”, zrób X, ale dopisz 2 pytania sprawdzające.
- Nazwy po angielsku, PEP 8, docstringi. Bez nowych pakietów bez pytania.
- Usuwanie i edycja zawsze przez POST. Po POST redirect.
```

---

*Nie rozumiesz kodu – nie oddawaj. Rozumiesz – oddawaj, nie ma znaczenia, kto pisał.*