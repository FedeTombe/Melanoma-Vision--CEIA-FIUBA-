# Clasificación y segmentación de lesiones cutáneas con deep learning

Trabajo práctico final de **Visión por Computadora II** (Especialización en Inteligencia Artificial, CEIA, UBA).
Prototipo académico que (i) clasifica imágenes dermoscópicas como **benignas o malignas**, (ii) **segmenta la lesión** y (iii) **audita el dataset público** en busca de réplicas entre entrenamiento y prueba.

> **Aviso:** es un prototipo con fines académicos. No es una herramienta diagnóstica ni tiene validación clínica.

## Resultados principales

Partición corregida (sin réplicas entre conjuntos), evaluación única en prueba.

| Tarea | Modelo | Resultado en prueba |
|---|---|---|
| Clasificación (n = 1.152) | ResNet-18, ajuste completo | AUC 0,977 (IC 95 %: 0,969–0,984), sensibilidad 0,927, especificidad 0,914, 3,7 ms/imagen |
| Segmentación (n = 260) | U-Net, encoder ResNet-18 | Dice 0,883 (IC 95 %: 0,863–0,899), IoU 0,813, 9,0 ms/imagen |

Auditoría de datos: SHA-256 no encontró duplicados exactos; un hash perceptual (dHash, Hamming ≤ 2) encontró 6.344 pares candidatos entre particiones, agrupados en 732 grupos que obligaron a reasignar 1.209 imágenes.
El informe completo está en [`paper/VPCII_TpFinal.tex`](paper/VPCII_TpFinal.tex).

## Estructura del repositorio

```
.
├── notebooks/
│   └── VpCII_TP_Final_C.ipynb   # pipeline completo (datos → auditoría → modelos → evaluación)
├── paper/                                          # informe IEEE en LaTeX
├── results/                                        # métricas livianas de la corrida final
├── requirements.txt
└── README.md
```

## Datasets

Los datos **no se incluyen** en el repositorio; el notebook los descarga con `kagglehub`.

| Tarea | Dataset | Notas |
|---|---|---|
| Clasificación | [Melanoma skin cancer dataset: benign vs malignant](https://www.kaggle.com/datasets/ailearner-researchlab/melanoma-skin-cancer-dataset-benign-vs-malignant) | 13.879 imágenes RGB de 224×224. Versión 1. Fuente primaria y licencia **no verificadas**. |
| Segmentación | [ISIC 2018 Challenge Task 1 (espejo de Kaggle)](https://www.kaggle.com/datasets/tschandl/isic2018-challenge-task1-data-segmentation) | 2.594 pares imagen–máscara, CC0. |

Si `kagglehub` pide credenciales, crear un token en Kaggle (*Settings → API*) y definir `KAGGLE_USERNAME` y `KAGGLE_KEY`.

## Cómo reproducir

### Opción A: Google Colab (recomendada)
1. Subir `notebooks/VpCII_TP_Final_C.ipynb` a Colab.
2. Activar GPU: *Entorno de ejecución → Cambiar tipo → GPU*.
3. *Ejecutar todo*. El notebook instala `kagglehub`, `albumentations` y `segmentation-models-pytorch` en sus primeras celdas.

### Opción B: entorno local
```bash
python -m venv .venv && source .venv/bin/activate    # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter notebook notebooks/VpCII_TP_Final_C.ipynb
```
Se recomienda GPU con CUDA; en CPU el entrenamiento tarda varias horas.

La semilla está fija (`SEED = 42`) y se activa el modo determinista de cuDNN. Aun así, los resultados pueden variar levemente entre versiones de PyTorch y tipos de GPU.

## Qué hace el notebook

1. **Entorno y datos:** descarga, conteo y verificación de imágenes.
2. **Auditoría de integridad:** SHA-256 por archivo y dHash de 64 bits; pares con distancia de Hamming ≤ 2.
3. **Partición sin réplicas:** Union-Find sobre los pares y reasignación de cada grupo a su partición mayoritaria (train > val > test en empate).
4. **Clasificación:** EfficientNet-B0 y ResNet-18 con pesos de ImageNet; etapa 1 con la base congelada y etapa 2 con ajuste completo. El modelo se elige por AUC de validación y el umbral por sensibilidad ≥ 0,90 en validación.
5. **Evaluación única en prueba:** métricas con IC 95 % por bootstrap (2.000 remuestreos), matriz de confusión, ROC y valor predictivo positivo según la prevalencia.
6. **Segmentación:** U-Net (`segmentation_models_pytorch`) con pérdida 0,5 Dice + 0,5 BCE; Dice e IoU por imagen, latencia, mejores, medianos y peores casos.

## Limitaciones conocidas

- El grupo de réplicas más grande reúne 1.172 imágenes (8,4 % del dataset): puede reflejar encadenamiento de imágenes similares y no réplicas reales. Además, 22 grupos tienen etiquetas contradictorias.
- Ningún dataset trae identificadores de paciente o lesión, por lo que no se garantiza independencia por paciente.
- La prueba corregida conserva imágenes de la prueba original, usada durante el desarrollo.
- Clasificación y segmentación son modelos independientes sobre datasets distintos.

## Autores

Federico Tombesi 
Tomás Civini
Hernán Ruggeri
Pablo Gorosito

## Licencia

Pendiente de definir por el equipo. Los datasets conservan sus licencias originales.
