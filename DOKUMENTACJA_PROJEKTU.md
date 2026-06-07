# Dokumentacja projektu: Drzewa Decyzyjne
### Klasyfikacja typów gwiazd z użyciem drzewa decyzyjnego

**Politechnika Bydgoska im. Jana i Jędrzeja Śniadeckich**  
**Wydział Telekomunikacji, Informatyki i Elektrotechniki**  
al. prof. S. Kaliskiego 7, 85-796 Bydgoszcz

**Autorzy:**  
Maciej Kwiatkowski  
Mikołaj Gawroński  
Mateusz Machowski

## 1. Cel projektu

Celem projektu jest zbudowanie i ocena modelu drzewa decyzyjnego służącego do klasyfikacji typów gwiazd na podstawie ich właściwości fizycznych oraz termodynamicznych.

Projekt pokazuje pełny proces pracy z danymi:

- wczytanie i opis zbioru danych,
- eksploracyjną analizę danych,
- przygotowanie danych do modelowania,
- budowę modelu bazowego,
- ocenę jakości klasyfikacji,
- walidację krzyżową,
- strojenie hiperparametrów,
- interpretację drzewa decyzyjnego,
- analizę ważności cech,
- zapis wyników i modelu do plików.

Główny plik projektu: [`DD_PROJ.ipynb`](DD_PROJ.ipynb)

## 2. Dane

### Źródło danych

W projekcie wykorzystano zbiór danych `Star Dataset` dostępny na Kaggle:

https://www.kaggle.com/datasets/deepu1109/star-dataset

Plik danych użyty w projekcie:

[`data/star_dataset_6_classes.csv`](data/star_dataset_6_classes.csv)

### Rozmiar zbioru

Zbiór zawiera:

- 240 próbek,
- 7 oryginalnych kolumn,
- 6 klas gwiazd,
- brak brakujących wartości,
- brak pełnych duplikatów.

Po dodaniu pomocniczej kolumny `Star_name` notebook pracuje na 8 kolumnach.

### Zmienna docelowa

Zmienną docelową jest kolumna `Star type`, która reprezentuje klasę gwiazdy.

Mapowanie klas:

| Wartość | Nazwa klasy |
|---:|---|
| 0 | Brown Dwarf |
| 1 | Red Dwarf |
| 2 | White Dwarf |
| 3 | Main Sequence |
| 4 | Supergiant |
| 5 | Hypergiant |

Każda klasa ma po 40 próbek, więc zbiór jest idealnie zbalansowany.

### Cechy wejściowe

| Kolumna | Typ | Opis |
|---|---|---|
| `Temperature (K)` | numeryczna | Temperatura gwiazdy w kelwinach. |
| `Luminosity(L/Lo)` | numeryczna | Jasność względem jasności Słońca. |
| `Radius(R/Ro)` | numeryczna | Promień względem promienia Słońca. |
| `Absolute magnitude(Mv)` | numeryczna | Magnitudo absolutne gwiazdy. |
| `Star color` | kategoryczna | Kolor gwiazdy. |
| `Spectral Class` | kategoryczna | Klasa spektralna gwiazdy. |

W danych występują drobne niespójności w zapisie kolumny `Star color`, np. `Blue-white`, `Blue White`, `Blue `. W finalnym notebooku pozostawiono oryginalne wartości, aby zachować stabilność wyników i wizualizacji drzewa zgodną z pierwotną wersją projektu.

## 3. Struktura projektu

```text
DD-projekt/
|-- DD_PROJ.ipynb
|-- DOKUMENTACJA_PROJEKTU.md
|-- requirements.txt
|-- data/
|   `-- star_dataset_6_classes.csv
`-- outputs/
    |-- best_decision_tree_model.pkl
    |-- decision_tree_visualization.png
    |-- feature_importance.csv
    `-- model_results.csv
```

### Najważniejsze pliki

| Plik | Znaczenie |
|---|---|
| [`DD_PROJ.ipynb`](DD_PROJ.ipynb) | Główny notebook projektu |
| [`data/star_dataset_6_classes.csv`](data/star_dataset_6_classes.csv) | Dane wejściowe |
| [`outputs/model_results.csv`](outputs/model_results.csv) | Porównanie wyników modeli |
| [`outputs/feature_importance.csv`](outputs/feature_importance.csv) | Ważność cech w finalnym modelu |
| [`outputs/best_decision_tree_model.pkl`](outputs/best_decision_tree_model.pkl) | Zapisany finalny model |
| [`outputs/decision_tree_visualization.png`](outputs/decision_tree_visualization.png) | Zapisana wizualizacja drzewa decyzyjnego |
| [`requirements.txt`](requirements.txt) | Lista bibliotek potrzebnych do uruchomienia projektu |

## 4. Wymagania i uruchomienie

Projekt wymaga Pythona oraz bibliotek wymienionych w pliku [`requirements.txt`](requirements.txt):

```text
pandas
matplotlib
seaborn
scikit-learn
joblib
```

Instalacja zależności:

```bash
pip install -r requirements.txt
```

Uruchomienie projektu:

1. Otwórz [`DD_PROJ.ipynb`](DD_PROJ.ipynb) w PyCharm, Jupyter Notebook, JupyterLab albo Google Colab.
2. Upewnij się, że plik danych znajduje się w katalogu `data/`.
3. Uruchom wszystkie komórki notebooka od góry do dołu.
4. Wyniki zostaną zapisane w katalogu `outputs/`.

Plik `requirements.txt` nie jest wymagany przez sam notebook, ale jest przydatny przy przenoszeniu projektu na inny komputer lub do środowiska Colab/Jupyter.

## 5. Eksploracyjna analiza danych

W notebooku wykonano podstawową eksplorację danych:

- sprawdzenie rozmiaru zbioru,
- sprawdzenie typów kolumn,
- sprawdzenie brakujących wartości,
- sprawdzenie pełnych duplikatów,
- analizę rozkładu klas,
- analizę korelacji cech numerycznych.

### Rozkład klas

Zmienna docelowa `Star type` jest równomiernie rozłożona. Każda z 6 klas ma po 40 próbek.

Jest to korzystne dla klasyfikacji, ponieważ model nie jest faworyzowany w stronę jednej dominującej klasy.

### Korelacje

Najsilniejsze zależności ze zmienną docelową zaobserwowano dla:

| Cecha | Charakter zależności |
|---|---|
| `Absolute magnitude(Mv)` | silna ujemna korelacja z typem gwiazdy |
| `Luminosity(L/Lo)` | dodatnia korelacja |
| `Radius(R/Ro)` | dodatnia korelacja |
| `Temperature (K)` | słabsza dodatnia korelacja |

Interpretację korelacji z `Star type` należy traktować pomocniczo, ponieważ `Star type` jest etykietą klasy zapisaną liczbowo. W tym zbiorze ma to jednak sens interpretacyjny, ponieważ klasy są uporządkowane zgodnie z właściwościami fizycznymi obiektów.

## 6. Przygotowanie danych

Do modelowania przygotowano:

- macierz cech `X`,
- zmienną docelową `y`,
- listę cech numerycznych,
- listę cech kategorycznych.

Z modelowania usunięto:

- `Star type`, ponieważ jest zmienną docelową,
- `Star_name`, ponieważ jest tekstową reprezentacją klasy i mogłaby powodować wyciek informacji.

### Podział danych

Dane podzielono na:

- zbiór treningowy: 75% danych,
- zbiór testowy: 25% danych.

Użyto parametru `stratify=y`, dzięki czemu rozkład klas w zbiorze treningowym i testowym pozostał równomierny.

### Preprocessing

Wykorzystano `ColumnTransformer`:

- cechy kategoryczne zakodowano przez `OneHotEncoder(handle_unknown='ignore')`,
- cechy numeryczne przekazano bez zmian.

Całość została umieszczona w `Pipeline`, dzięki czemu preprocessing i model są trenowane razem w kontrolowany sposób.

## 7. Model bazowy

Pierwszym modelem był `DecisionTreeClassifier` z domyślnymi parametrami i `random_state=42`.

Model bazowy uzyskał:

| Metryka | Wartość |
|---|---:|
| Accuracy na zbiorze testowym | 1.0 |
| Średnia accuracy w 5-krotnej walidacji krzyżowej | 1.0 |
| Odchylenie standardowe CV | 0.0 |

Macierz pomyłek modelu bazowego zawiera wartości wyłącznie na przekątnej, co oznacza brak błędnych klasyfikacji na zbiorze testowym.

## 8. Walidacja krzyżowa

W projekcie zastosowano 5-krotną walidację krzyżową.

Celem walidacji było sprawdzenie, czy wysoki wynik modelu nie wynika wyłącznie z jednego korzystnego podziału train-test.

Model bazowy uzyskał średni wynik walidacji krzyżowej równy 1.0, co wskazuje, że klasy w zbiorze są bardzo łatwe do rozdzielenia za pomocą drzewa decyzyjnego
## 9. Strojenie hiperparametrów

Do strojenia użyto `GridSearchCV`.

Sprawdzano:

- kryterium podziału: `gini`, `entropy`,
- maksymalną głębokość drzewa,
- minimalną liczbę próbek wymaganą do podziału węzła,
- minimalną liczbę próbek w liściu.

Najlepsze parametry:

```text
criterion: gini
max_depth: 4
min_samples_leaf: 1
min_samples_split: 2
```

Model po strojeniu uzyskał:

| Metryka | Wartość |
|---|---:|
| Accuracy na zbiorze testowym | 1.0 |
| Średnia accuracy CV | 0.9944444444444445 |
| Odchylenie standardowe CV | 0.011111111111111117 |

Strojenie nie poprawiło accuracy na zbiorze testowym, ponieważ model bazowy osiągnął już wynik maksymalny.

## 10. Porównanie modeli

Aktualne wyniki zapisane w [`outputs/model_results.csv`](outputs/model_results.csv):

| Model | Accuracy test | CV mean | CV std |
|---|---:|---:|---:|
| Decision Tree - bazowy | 1.0 | 1.0 | 0.0 |
| Decision Tree - po strojeniu | 1.0 | 0.9944444444444445 | 0.011111111111111117 |

Oba modele osiągają idealny wynik na zbiorze testowym. Model bazowy uzyskał również idealną walidację krzyżową.

Wynik ten należy interpretować ostrożnie. Bardziej prawdopodobne jest, że wynika on z charakteru danych niż z uniwersalnej doskonałości modelu.

## 11. Interpretacja drzewa decyzyjnego

Wizualizacja drzewa została zapisana do pliku:

[`outputs/decision_tree_visualization.png`](outputs/decision_tree_visualization.png)

Drzewo jest ograniczone w wizualizacji do `max_depth=3`, aby zachować czytelność.

Finalne drzewo wykorzystuje przede wszystkim progi na cechach:

- `Absolute magnitude(Mv)`,
- `Radius(R/Ro)`,
- `Luminosity(L/Lo)`.

Struktura drzewa pokazuje, że kilka prostych warunków wystarcza do rozdzielenia klas w tym zbiorze.

## 12. Ważność cech

Najważniejsze cechy finalnego modelu:

| Cecha | Ważność |
|---|---:|
| `num__Absolute magnitude(Mv)` | 0.4 |
| `num__Radius(R/Ro)` | 0.4 |
| `num__Luminosity(L/Lo)` | 0.2 |

Pozostałe cechy, w tym:

- `Temperature (K)`,
- `Star color`,
- `Spectral Class`,

mają ważność równą `0.0` w finalnym modelu.

Oznacza to, że model nie potrzebował tych cech do wykonania końcowych podziałów. Nie oznacza to, że są one fizycznie nieistotne, ale że w tym konkretnym zbiorze informacje zawarte w trzech najważniejszych cechach wystarczyły do klasyfikacji.

## 13. Zapisane artefakty

Po wykonaniu notebooka w katalogu `outputs/` zapisywane są:

| Plik | Opis |
|---|---|
| `model_results.csv` | Porównanie modelu bazowego i modelu po strojeniu. |
| `feature_importance.csv` | Ważność cech finalnego modelu. |
| `best_decision_tree_model.pkl` | Zapisany pipeline z preprocessingiem i modelem. |
| `decision_tree_visualization.png` | Wizualizacja drzewa decyzyjnego w wysokiej rozdzielczości. |

## 14. Ograniczenia projektu

Najważniejsze ograniczenia:

- zbiór danych jest mały,
- dane są czyste i kompletne,
- klasy są idealnie zbalansowane,
- klasy są bardzo dobrze separowalne przez kilka cech fizycznych,
- wynik `accuracy = 1.0` nie musi przenosić się na większe lub bardziej zaszumione dane,
- dataset ma charakter dydaktyczny i nie powinien być traktowany jako pełna reprezentacja rzeczywistych danych astronomicznych.

Z tego powodu wynik modelu należy interpretować jako bardzo dobry rezultat dla tego konkretnego zbioru danych, a nie jako gwarancję bezbłędnego działania na dowolnych nowych obserwacjach.

## 15. Wnioski końcowe

Drzewo decyzyjne okazało się skutecznym i interpretowalnym modelem dla tego problemu.

Najważniejsze obserwacje:

- model poprawnie sklasyfikował wszystkie próbki testowe,
- walidacja krzyżowa potwierdziła bardzo wysoką skuteczność,
- najważniejsze cechy to `Absolute magnitude(Mv)`, `Radius(R/Ro)` i `Luminosity(L/Lo)`,
- strojenie hiperparametrów nie poprawiło wyniku testowego, ponieważ model bazowy osiągnął już maksimum,
- perfekcyjny wynik wynika przede wszystkim z charakteru danych.