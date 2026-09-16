# Automation of Scientific Information Extraction from Organic Photovoltaic Literature Using LLM

Extraction automatisée, via un LLM (Google Gemini), de données structurées de dispositifs photovoltaïques organiques (OPV) à partir d'articles scientifiques au format PDF.
Automated, LLM-based (Google Gemini) extraction of structured organic photovoltaic (OPV) device data from scientific PDF papers.

---

## 🇫🇷 Français

### Description

Ce projet extrait automatiquement, à partir d'articles scientifiques PDF portant sur les cellules solaires organiques (OPV), les données structurées de chaque dispositif testé : architecture de la cellule, matériaux de la couche active, conditions de fabrication (solvant, additifs, recuit) et métriques de performance (PCE, Voc, Jsc, FF, HOMO/LUMO).

Le pipeline s'appuie sur l'API **Google Gemini** (modèle multimodal, lecture native de PDF) guidée par un prompt d'extraction détaillé, puis met les résultats en forme dans un fichier Excel exploitable pour l'analyse de corpus.

### Contenu du dépôt

| Fichier | Description |
|---|---|
| `demoOPV.ipynb` | Notebook Jupyter contenant tout le pipeline : configuration de l'API, prompt d'extraction, fonctions de traitement, exécution principale. |
| `full_corpus_extracted.xlsx` | Exemple de résultat : données extraites sur un corpus d'articles. |
| `requirements.txt` | Dépendances Python nécessaires. |
| `.env.example` | Modèle de fichier de configuration pour la clé API. |

### Fonctionnement du pipeline

1. **Regroupement des fichiers** : les PDF d'un même article (article principal + informations supplémentaires) sont regroupés automatiquement par numéro détecté dans leur nom de fichier (ex. `Article 6.pdf` + `information support_Article 6.pdf`).
2. **Envoi à Gemini** : les fichiers PDF sont envoyés directement au modèle (lecture native, sans OCR manuel), accompagnés d'un prompt d'extraction détaillé qui impose :
   - la recherche systématique de la section expérimentale/fabrication ;
   - l'application des paramètres globaux (solvant, additif, recuit) à tous les dispositifs, sauf mention contraire ;
   - la séparation de chaque variation (ratio, température de recuit, etc.) en une entrée JSON distincte ;
   - des règles de normalisation strictes (unités, noms de solvants, gestion des données manquantes).
3. **Parsing et nettoyage** : la réponse du modèle est nettoyée puis validée comme JSON, avec une logique de nouvelle tentative (jusqu'à 3 essais) en cas de réponse mal formée.
4. **Mise à plat et export** : chaque dispositif extrait devient une ligne d'un tableau (`Source File`, `Device ID`, `Structure`, `Donor`, `Acceptor`, `Ratio`, `Solvent`, `Additive`, `Annealing Temp`, `PCE (%)`, `Voc (V)`, `Jsc (mA/cm2)`, `FF (%)`, `Notes`), exporté en `.xlsx`.

### Structure JSON extraite par article

```json
{
  "global_fabrication_context": "résumé des conditions globales",
  "devices": [
    {
      "device_id": "...",
      "architecture": { "type": "...", "anode": "...", "hole_transport_layer": "...", "electron_transport_layer": "...", "cathode": "..." },
      "active_layer": { "donor": "...", "acceptor": "...", "ratio_weight": "...", "thickness_nm": "..." },
      "processing": { "solvent": "...", "total_concentration_mg_ml": "...", "additive_name": "...", "additive_volume_percent": "...", "deposition_method": "...", "annealing_temperature_celsius": "...", "annealing_time_min": "..." },
      "metrics": { "pce_percent": "...", "voc_volts": "...", "jsc_ma_cm2": "...", "ff_percent": "...", "HUMO": "...", "LUMO": "..." },
      "notes": "..."
    }
  ]
}
```

### Installation

```bash
git clone https://github.com/mokhmadguiriev/automation-of-scientific-information-extraction-from-organic-photovoltaic-literature-using-llm.git
cd automation-of-scientific-information-extraction-from-organic-photovoltaic-literature-using-llm
pip install -r requirements.txt
```

### Configuration de la clé API

1. Récupérer une clé API Gemini sur [Google AI Studio](https://aistudio.google.com/app/apikey).
2. Copier `.env.example` vers `.env` :
   ```bash
   cp .env.example .env
   ```
3. Renseigner la clé dans `.env` :
   ```
   GOOGLE_API_KEY=votre-clé-api
   ```

⚠️ **Ne commitez jamais votre fichier `.env`** (il est déjà exclu via `.gitignore`). Une clé API exposée publiquement doit être révoquée immédiatement depuis Google AI Studio.

### Utilisation

1. Placer les PDF à traiter dans un dossier `test/` à la racine du projet (l'article principal et, le cas échéant, ses informations supplémentaires, avec un même numéro dans le nom de fichier).
2. Ouvrir `demoOPV.ipynb` et exécuter les cellules dans l'ordre.
3. Le fichier `scanned_pdf_extraction.xlsx` est généré avec les données extraites.

> Note : l'API Gemini impose des limites de requêtes ; il est conseillé de tester d'abord avec un seul article dans `test/` avant de traiter un corpus complet.

### Licence

Ce projet est distribué sous licence [MIT](LICENSE).

---

## 🇬🇧 English

### Description

This project automatically extracts, from scientific PDF papers about organic photovoltaic (OPV) solar cells, structured data for every tested device: cell architecture, active-layer materials, fabrication conditions (solvent, additives, annealing), and performance metrics (PCE, Voc, Jsc, FF, HOMO/LUMO).

The pipeline relies on the **Google Gemini** API (a multimodal model with native PDF reading) guided by a detailed extraction prompt, then formats the results into an Excel file suitable for corpus-level analysis.

### Repository contents

| File | Description |
|---|---|
| `demoOPV.ipynb` | Jupyter notebook containing the full pipeline: API setup, extraction prompt, processing functions, main execution. |
| `full_corpus_extracted.xlsx` | Sample output: data extracted from a corpus of papers. |
| `requirements.txt` | Required Python dependencies. |
| `.env.example` | Template configuration file for the API key. |

### How the pipeline works

1. **File grouping**: PDFs belonging to the same article (main text + supporting information) are automatically grouped by a number detected in the filename (e.g. `Article 6.pdf` + `information support_Article 6.pdf`).
2. **Sending to Gemini**: the PDF files are sent directly to the model (native reading, no manual OCR), along with a detailed extraction prompt that enforces:
   - systematically locating the experimental/fabrication section;
   - applying global parameters (solvent, additive, annealing) to all devices unless explicitly stated otherwise;
   - splitting every variation (ratio, annealing temperature, etc.) into a separate JSON entry;
   - strict normalization rules (units, solvent names, handling of missing data).
3. **Parsing and cleanup**: the model's response is cleaned and validated as JSON, with a retry loop (up to 3 attempts) in case of malformed output.
4. **Flattening and export**: each extracted device becomes one row of a table (`Source File`, `Device ID`, `Structure`, `Donor`, `Acceptor`, `Ratio`, `Solvent`, `Additive`, `Annealing Temp`, `PCE (%)`, `Voc (V)`, `Jsc (mA/cm2)`, `FF (%)`, `Notes`), exported as `.xlsx`.

### Extracted JSON structure (per article)

```json
{
  "global_fabrication_context": "summary of global conditions",
  "devices": [
    {
      "device_id": "...",
      "architecture": { "type": "...", "anode": "...", "hole_transport_layer": "...", "electron_transport_layer": "...", "cathode": "..." },
      "active_layer": { "donor": "...", "acceptor": "...", "ratio_weight": "...", "thickness_nm": "..." },
      "processing": { "solvent": "...", "total_concentration_mg_ml": "...", "additive_name": "...", "additive_volume_percent": "...", "deposition_method": "...", "annealing_temperature_celsius": "...", "annealing_time_min": "..." },
      "metrics": { "pce_percent": "...", "voc_volts": "...", "jsc_ma_cm2": "...", "ff_percent": "...", "HUMO": "...", "LUMO": "..." },
      "notes": "..."
    }
  ]
}
```

### Installation

```bash
git clone https://github.com/mokhmadguiriev/automation-of-scientific-information-extraction-from-organic-photovoltaic-literature-using-llm.git
cd automation-of-scientific-information-extraction-from-organic-photovoltaic-literature-using-llm
pip install -r requirements.txt
```

### API key setup

1. Get a Gemini API key from [Google AI Studio](https://aistudio.google.com/app/apikey).
2. Copy `.env.example` to `.env`:
   ```bash
   cp .env.example .env
   ```
3. Fill in your key in `.env`:
   ```
   GOOGLE_API_KEY=your-api-key
   ```

⚠️ **Never commit your `.env` file** (it is already excluded via `.gitignore`). A publicly exposed API key should be revoked immediately from Google AI Studio.

### Usage

1. Place the PDFs to process in a `test/` folder at the project root (the main article and, if available, its supporting information, sharing the same number in their filename).
2. Open `demoOPV.ipynb` and run the cells in order.
3. The `scanned_pdf_extraction.xlsx` file is generated with the extracted data.

> Note: the Gemini API enforces request limits; it is recommended to first test with a single article in `test/` before processing a full corpus.

### License

This project is distributed under the [MIT](LICENSE) license.
