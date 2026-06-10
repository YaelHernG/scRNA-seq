# Análisis scRNA-seq: Efecto de Idasanutlin y Nilotinib en Leucemia Mieloide Crónica (LMC)

### Información General
* **Integrantes:** Yael Daniel Hernández González y Paola Albarrán Godoy.
* **Programa:** Licenciatura en Ciencias Genómicas, UNAM ENES Juriquilla.
* **Materia:** Transcriptómica Diferencial.
* **Semestre:** 6to Semestre.
* **Fecha:** Junio de 2026.

---

##  Abstract del Trabajo
En la leucemia mieloide crónica (LMC), las células madre leucémicas (LSC) quiescentes persisten en casi todos los pacientes en fase crónica bajo tratamiento con inhibidores de tirosina cinasa (TKI), representando el principal obstáculo para alcanzar la remisión libre de terapia (TFR). En este trabajo se realizó un análisis de scRNA-seq (10x Chromium, Illumina HiSeq4000) sobre células humanas huCD45⁺ y huCD34⁺ aisladas de médula ósea de ratones PDX injertados con células primarias de LMC en fase crónica (GSE218185; Scott et al., 2024). Se compararon tres condiciones: vehículo (VEH), nilotinib (NIL) y la combinación nilotinib+idasanutlin (NIL+IDASA). 
El pipeline bioinformático consistió en lectura de matrices procesadas de GEO con Seurat v5, control de calidad por célula (genes, UMIs, porcentaje mitocondrial), normalización LogNormalize, selección de 2,000 genes altamente variables (VST), reducción de dimensionalidad con PCA (50 componentes), corrección de efecto de lote con Harmony por experimento de captura, proyección UMAP, clustering con Louvain (resolución 0.5), identificación de marcadores con FindAllMarkers (Wilcoxon) y análisis de expresión diferencial entre condiciones. Se identificaron 17 clusters que representan los cuatro linajes hematopoyéticos descritos en el paper original (LSC/progenitores ESC-REG, NMM, ME y EBM). El cluster de LSC quiescentes (ESC-REGlow) presentó 0 células bajo tratamiento NIL+IDASA, consistente con el agotamiento de esta población reportado por Scott et al. (2024). El análisis de expresión diferencial en células CD34⁺ reveló downregulation de genes de supervivencia de LSC (FOS, DUSP1, ZFP36) y upregulation de genes de estrés oxidativo bajo NIL+IDASA. El enriquecimiento funcional GO-BP confirmó activación de vías relacionadas con respuesta a estrés y diferenciación, en concordancia con los resultados del artículo de referencia.

## Enlaces Importantes
* **Reporte Renderizado (Quarto/HTML):** [https://yaelherng.github.io/scRNA-seq/](https://yaelherng.github.io/scRNA-seq/)
* **Objeto Seurat Final (Zenodo):** Debido a restricciones de tamaño de GitHub, el objeto `.rds` con la anotación completa, reducciones dimensionales y metadatos puede descargarse desde el siguiente enlace de acceso restringido en Zenodo: [Descargar GSE218185_CML_anotado.rds](https://zenodo.org/records/20631502?token=eyJhbGciOiJIUzUxMiJ9.eyJpZCI6IjZhMTJkNDdjLTAyYTAtNDJhNC1hYTZlLWMzOGZhYjdiMzA5ZSIsImRhdGEiOnt9LCJyYW5kb20iOiJmZTZhMDRiOTk1ZDA2NDJkMzhhY2FmNGQ4NzI4OWExYiJ9.mRd_Gtf8Deqoj9unEMLkNLwUOenHcBD5PB0sRsrpbkqJqMNot6-3NXpCLZ8ReZ0afW1UJJYxYgAONYMq4_Pa3Q)

---

## Flujo de Trabajo y Estructura del Proyecto

El análisis se estructuró abarcando dos entornos computacionales. El intento de procesamiento primario a partir de FASTQs se diseñó en clúster HPC, mientras que el análisis *downstream* (a partir de las matrices descargadas de GEO por ausencia del *read 1*) se ejecutó en entorno local.


### 1. Entorno de Clúster HPC (Procesamiento primario)
Los *scripts* correspondientes a la obtención de los datos crudos desde SRA y la configuración para el alineamiento con Cell Ranger se estructuraron de la siguiente manera en el clúster:

```text
BioProject_scRNA/
├── data/
│   └── fastq/                         # FASTQs descargados desde SRA (21 SRRs, ~466 GB)
├── reference/
│   ├── cellranger_7.2.0.sif           # Imagen Singularity de Cell Ranger
│   └── refdata-gex-GRCh38-2020-A/     # Genoma de referencia GRCh38 (GENCODE v32)
└── scripts/
    ├── 01_download_fastq.sh           # Descarga de FASTQs con prefetch/fasterq-dump
    ├── 02_download_reference.sh       # Descarga del genoma de referencia
    ├── 03_pull_cellranger.sh          # Descarga de imagen Singularity
    ├── 04_run_cellranger.sh           # Alineamiento y cuantificación con Cell Ranger
    └── logs/                          # Logs de los jobs SLURM

