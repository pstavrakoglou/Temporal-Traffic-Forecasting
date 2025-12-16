# NYC Traffic Knowledge Graph & Forecasting

A semantic web and machine learning project that transforms NYC Automated Traffic Volume data into a **Knowledge Graph (KG)** to perform traffic forecasting using **Graph Neural Networks (GNNs)**.

## 📌 Project Overview

This project explores the intersection of **Knowledge Graphs** and **Time-Series Forecasting**. Instead of treating traffic data as isolated tabular rows, we construct a semantic graph where:
* **Streets** are distinct entities with geospatial properties (WKT).
* **Traffic Observations** are events linked to specific streets and timestamps.

The pipeline ingests raw CSV data, converts it into RDF triples, creates a traversable graph structure using NetworkX, and prepares the data for a PyTorch-based forecasting model.

## 📂 Data Source

* **Dataset:** [NYC Automated Traffic Volume Counts]([https://www.kaggle.com/datasets](https://www.kaggle.com/datasets/aadimator/nyc-automated-traffic-volume-counts)) (Kaggle)
* **Note:** The raw CSV file is not included in this repository due to its large size. You must download it and place it in the appropriate directory (e.g., Google Drive root) before running the notebook.

## 🛠️ Tech Stack

* **Python 3.10+**
* **Data Manipulation:** `pandas`, `gzip`
* **Semantic Web / RDF:** `rdflib` (for constructing Turtle and N-Triples)
* **Graph Processing:** `networkx` (for graph traversal and analysis)
* **Machine Learning:** `torch` (PyTorch) for building the forecasting model
* **Utilities:** `tqdm` (progress tracking)

## ⚙️ Methodology & Pipeline

### 1. Ontology Definition
We defined a custom lightweight ontology to map the CSV data:
* **Prefixes:**
    * `ex`: `http://example.org/traffic/`
    * `geo`: `http://www.opengis.net/ont/geosparql#`
    * `time`: `http://www.w3.org/2006/time#`
* **Entities:**
    * `ex:Street`: Represents a physical street segment.
    * `ex:TrafficObservation`: Represents a single volume count event.

### 2. Knowledge Graph Construction (RDF)
The project initially attempted to serialize the graph into **Turtle (.ttl)** format.
* **Challenge Encountered:** Parsing the massive `.ttl` file resulted in syntax errors due to complex string formatting in the raw data (e.g., nested quotes in street names).
* **Solution:** The pipeline was refactored to generate **N-Triples (.nt)**. This format is line-based and more robust for streaming large datasets, allowing for successful parsing without memory overflows.

### 3. Graph Ingestion (NetworkX)
We implemented a custom **Stream Parser** to load the N-Triples directly into a `networkx.MultiDiGraph`.
* Nodes are created for Streets and Observations.
* Edges represent the `observedAt` relationship.
* This approach avoids loading the entire RDF graph into memory at once, enabling efficient processing of millions of triples.

### 4. Forecasting Model
The graph data is transformed into a time-series format suitable for machine learning. A **PyTorch** model is constructed to perform **One-Step Ahead Forecasting**, predicting the traffic volume $t+1$ based on the historical data $t$ derived from the graph structure.

## 🚀 How to Run

1.  **Environment Setup:**
    Ensure you have the required libraries installed:
    ```bash
    pip install pandas rdflib networkx torch
    ```

2.  **Data Placement:**
    Download the `TrafficCount.csv` from Kaggle and upload it to your Google Drive (if using Colab) or local project folder.

3.  **Execute the Notebook:**
    Run `KGINDIVID.ipynb`. The notebook is structured sequentially:
    * **Step 1:** Mounts drive and installs dependencies.
    * **Step 2:** ETL process converting CSV to `.nt` (N-Triples).
    * **Step 3:** Streaming the graph into NetworkX.
    * **Step 4:** Extracting time-series tensors from the graph.
    * **Step 5:** Training the PyTorch model for forecasting.

## 📝 Documentation & Comments

The code within `KGINDIVID.ipynb` is heavily commented to explain the decision-making process, specifically:
* Why specific RDF serialization formats were chosen.
* How the custom sink for NetworkX works to handle memory constraints.
* The architecture of the forecasting loop.

## ⚠️ Known Issues / Limitations

* **Turtle Parsing:** As noted in the methodology, the `.ttl` parser in `rdflib` may fail on specific dirty data rows. The `.nt` generator provided in the notebook is the stable workaround.
* **Memory Usage:** Creating a NetworkX graph with millions of nodes requires significant RAM. Run this in an environment with at least 12GB+ RAM (e.g., Google Colab Pro or a local machine).

## 📜 License

This project is open-source and available for educational and academic use.
