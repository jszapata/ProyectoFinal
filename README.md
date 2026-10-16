# ChestX-ray14: comparación de cuatro modelos

Proyecto de clasificación multietiqueta de radiografías del conjunto NIH ChestX-ray14. El entregable principal es `ChestXray14_ProyectoFinal.ipynb`; junto con él se conservan los CSV de resultados e historiales y los checkpoints guardados en `outputs/`.

## Contenido

- `ChestXray14_ProyectoFinal.ipynb`: preparación, entrenamiento original, evaluación, ejemplos de test, Grad-CAM exploratorio, comparación de cuatro modelos y auditoría final sin reentrenamiento.
- `outputs/results.csv`: métricas de test de ResNet-18, DenseNet-121 y EfficientNet-B0.
- `outputs/resnet18_transformer_result.csv`: resultado separado del híbrido ResNet-18 + Transformer.
- `outputs/*_history.csv`: métricas y tiempos por época de los cuatro modelos.
- `outputs/*_best.pt`: checkpoint seleccionado por F1 macro de validación.
- `outputs/run_config.json` y `outputs/resnet18_transformer_run_config.json`: configuración de las ejecuciones.
- `outputs/split_manifest.csv`: imagen, etiqueta, paciente y partición asignada.
- `ChestXray14_Articulo_IEEE.tex`: fuente LaTeX del artículo. El PDF se compila por separado.
- `ChestXray14_Presentacion_Final.pptx`: presentación editable del proyecto.

## Entorno y ejecución reproducible

La ejecución registrada se realizó en macOS ARM64 con Python 3.14.7, PyTorch 2.13.0 y dispositivo MPS. Se usaron semilla 42, imágenes de `224 × 224`, batch 8, cinco épocas, AdamW con learning rate `1e-4` y weight decay `1e-4`, `BCEWithLogitsLoss`, pesos iniciales ImageNet y selección del checkpoint por F1 macro de validación. El protocolo y el dispositivo están registrados en los archivos `*_run_config.json` y en la sección de auditoría del notebook.

`requirements.txt` declara versiones mínimas, no un lockfile exacto. Para repetir el experimento, crea un entorno compatible con PyTorch/torchvision y el acelerador disponible, instala esas dependencias, coloca el dataset en `Data/Archive/images_*/images/` y sus metadatos/splits bajo `Data/archive/`, y abre el notebook desde esta carpeta. Las rutas admiten diferencias de mayúsculas entre `Archive` y `archive`. Configura Kaggle y acepta los términos del dataset si necesitas descargar los archivos. El dataset y las credenciales no se distribuyen con el proyecto.

El manifiesto registra 75,556 imágenes de entrenamiento (24,647 pacientes), 10,968 de validación (3,361 pacientes) y 25,596 de test (2,797 pacientes). El split está agrupado por paciente. Las métricas principales usan umbral 0.5; el ajuste secundario de umbrales se hace con validación para los tres modelos CNN y se evalúa en test después de fijarlo. El híbrido no tiene esa evaluación secundaria.

## Resultados guardados

| Modelo | F1 macro test | F1 micro | AUROC macro | Exactitud de subconjunto | Parámetros |
|---|---:|---:|---:|---:|---:|
| ResNet-18 | 0.163 | 0.236 | 0.790 | 0.371 | 11,183,694 |
| DenseNet-121 | 0.187 | 0.270 | **0.804** | 0.365 | 6,968,206 |
| EfficientNet-B0 | **0.196** | **0.280** | 0.799 | 0.366 | 4,025,482 |
| ResNet-18 + Transformer | 0.119 | 0.253 | 0.782 | 0.367 | 12,904,014 |

La exactitud es de subconjunto: exige acertar las 14 etiquetas de una imagen. Los valores son de una sola ejecución, cinco épocas y un único split; no aportan intervalos de confianza ni validación externa. Las etiquetas del dataset son ruidosas y las clases están muy desbalanceadas.

## Costo computacional y memoria

El tiempo total de entrenamiento (suma de las cinco épocas registradas, sin evaluación) fue aproximadamente 141.6 min para ResNet-18, 300.0 min para DenseNet-121, 242.2 min para EfficientNet-B0 y 141.1 min para el híbrido. El tiempo de evaluación del split de test quedó guardado en los CSV. Estas duraciones corresponden a la máquina y carga del sistema de la ejecución; no son una comparación universal de hardware.

La sección final del notebook midió memoria después del entrenamiento durante inferencia de una sola imagen (batch 1, `224 × 224`) en MPS. Los tensores del proceso ocuparon 43.3 MiB (ResNet-18), 27.5 MiB (DenseNet-121), 16.1 MiB (EfficientNet-B0) y 49.8 MiB (híbrido). La lectura de memoria del driver Metal después de inferencia fue 84.9, 88.7, 70.2 y 92.9 MiB, respectivamente. Son mediciones de inferencia específicas de este sistema.

**La memoria pico del entrenamiento no se midió durante las ejecuciones originales.** No puede inferirse a partir del tamaño del checkpoint, los parámetros ni la memoria de inferencia. Medirla exigiría otro entrenamiento instrumentado; por ahora queda documentada como limitación de costo computacional.

## Protección de artefactos y reejecución

El entrenamiento del híbrido ya fue ejecutado y sus archivos están separados. Las celdas de entrenamiento escriben resultados y pueden tomar varias horas. No ejecutes esas celdas para revisar las conclusiones. La celda final de auditoría lee los historiales y mide inferencia, sin reentrenar ni guardar cambios en `outputs/`. Conserva una copia de `outputs/` antes de iniciar cualquier nueva ejecución experimental.

## Alcance

Grad-CAM se generó solo para CNN y de forma exploratoria. Los modelos son prototipos académicos: no son dispositivos médicos ni deben usarse para tomar decisiones clínicas. Para mejorar el estudio harían falta varias semillas, más épocas con criterio de parada, tratamiento explícito del desbalance, ajuste de umbrales para los cuatro modelos, medición de memoria pico de entrenamiento y evaluación externa.
