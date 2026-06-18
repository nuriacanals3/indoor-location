# Indoor Location

Projecte de localització en interiors (*indoor positioning*) basat en senyals Wi-Fi (RSS, *Received Signal Strength*), centrat en la predicció de la **planta** (*floor*) d'un edifici a partir de les potències de senyal captades pels punts d'accés (APs).

Les dades provenen de les competicions **IPIN** (*International Conference on Indoor Positioning and Indoor Navigation*) dels anys **2018, 2019, 2020 i 2023**, i cada any es treballa de manera independent en el seu propi notebook.

## Estructura del repositori

```
indoor-location/
├── Dades/
│   ├── 2018/
│   ├── 2019/
│   ├── 2020/
│   └── 2023/
├── Notebook_2018.ipynb
├── Notebook_2019.ipynb
├── Notebook_2020.ipynb
└── Notebook_2023.ipynb
```

Cada carpeta `Dades/<any>/` conté sis fitxers CSV (sense capçalera):

| Fitxer | Contingut |
|---|---|
| `*_trainrss.csv` | Potències de senyal (RSS) del conjunt d'entrenament |
| `*_trainflr.csv` | Planta (etiqueta) del conjunt d'entrenament |
| `*_validrss.csv` | Potències de senyal del conjunt de validació |
| `*_validflr.csv` | Planta del conjunt de validació |
| `*_ctestreducedrss.csv` | Potències de senyal del conjunt de test |
| `*_ctestreducedfloor.csv` | Planta del conjunt de test |

Cada fila representa un registre de senyal Wi-Fi i cada columna un punt d'accés (AP). El valor `100` indica que l'AP no ha estat detectat; els valors reals de senyal estan expressats en dBm (rang aproximat de -99 a -33 dBm).

| Any | Mostres (train) | APs (columnes) |
|---|---|---|
| 2018 | 1.641 | 844 |
| 2019 | 3.274 | 708 |
| 2020 | 4.002 | 468 |
| 2023 | 3.275 | 310 |

## Notebooks

Cada notebook (`Notebook_<any>.ipynb`) segueix la mateixa estructura, aplicada al conjunt de dades del seu any corresponent:

1. **Anàlisi exploratòria i reducció manual de la dimensionalitat**
   - Eliminació dels APs sense cap senyal registrat.
   - Anàlisi de la freqüència de detecció dels APs per planta (heatmaps).
   - Anàlisi de la distribució de les potències més fortes per registre i per planta.
   - Eliminació dels APs amb baixa detecció (< 5% dels registres) i senyal feble (> -80 dBm).
   - Anàlisi de correlacions per eliminar APs redundants (possibles APs físicament duplicats, p. ex. bandes 2.4GHz/5GHz d'un mateix router).
   - Filtratge per variabilitat i potència màxima del senyal.

2. **Experiments amb models d'aprenentatge automàtic**
   - Comparativa entre **Random Forest** i **KNN** (amb normalització i mètrica Manhattan).
   - Anàlisi de sensibilitat al valor utilitzat per representar l'absència de senyal (100, -100, -105, -110, -200).
   - Avaluació amb el conjunt reduït manualment vs. reducció automàtica (moderada i agressiva) basada en presència, variància i representativitat per planta.
   - Selecció dels APs més rellevants segons la importància (`feature_importances_`) del Random Forest.

3. **Avaluació final** sobre el conjunt de test amb el millor model/configuració trobada.

### Resultats (accuracy a test)

| Any | Model final | Nº d'APs | Accuracy validació | Accuracy test |
|---|---|---|---|---|
| 2018 | Random Forest | 100 | 0,9968 | 0,9697 |
| 2019 | Random Forest | 64 (reducció moderada) | 0,9799* | 1,0000 |
| 2020 | Random Forest | 80 | 0,9874 | 0,9756 |
| 2023 | Random Forest | 73 | 0,9646* | 0,8485 |

*\*Accuracy de l'experiment indicat al notebook com a base de l'avaluació final.*

## Entorn d'execució

Els notebooks es poden executar tant en **local** com a **Google Colab**, sense necessitat de cap configuració:

- **En local:** cal clonar el repositori i obrir el notebook des de l'arrel del projecte (perquè la ruta relativa `Dades/<any>/` es resolgui correctament).

  ```bash
  git clone https://github.com/nuriacanals3/indoor-location.git
  cd indoor-location
  jupyter notebook
  ```

- **A Google Colab:** la primera cel·la detecta automàticament l'entorn i clona aquest repositori públic, de manera que no cal muntar Google Drive ni cap carpeta personal.

### Dependències principals

- `pandas`, `numpy`
- `matplotlib`, `seaborn`
- `scikit-learn` (`RandomForestClassifier`, `KNeighborsClassifier`, `StandardScaler`, `VarianceThreshold`, mètriques)
