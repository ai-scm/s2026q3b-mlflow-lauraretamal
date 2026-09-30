# Taller Hands-On: MLflow + Docker

## Descripción

En este taller se trabajó el ciclo de vida de un modelo de Machine Learning utilizando MLflow y Docker. Se realizó el entrenamiento de un modelo de clasificación, el seguimiento del experimento, el registro de una versión del modelo y su despliegue para realizar predicciones.

## Estructura del proyecto

El proyecto utilizado contiene los siguientes archivos principales:

```text
Dockerfile
docker-compose.yml
requirements.txt
clf-train.py
clf-train-registry.py
runServer.sh
serveModel.sh
predict.sh
```

`docker-compose.yml` define los servicios utilizados durante el ejercicio:

* `server`: servidor de MLflow.
* `trainmodel`: entrenamiento y registro del modelo.
* `servemodel`: despliegue del modelo.

## Configuración

Las dependencias principales utilizadas fueron MLflow, scikit-learn, pandas, NumPy y SciPy.

Durante la ejecución se presentó un problema de compatibilidad relacionado con SQLAlchemy. Se agregó la siguiente versión al archivo `requirements.txt`:

```text
SQLAlchemy==2.0.35
```

Después se reconstruyeron los contenedores:

```bash
docker compose -f docker-compose.yml up --build
```

## MLflow Server

El servidor se inicia mediante `runServer.sh`:

```bash
mlflow server \
  --backend-store-uri sqlite:///mlflow.db \
  --default-artifact-root ./mlflow-artifact-root \
  --host 0.0.0.0
```

En Docker, el puerto `5000` del contenedor se expone como `8000` en el equipo local.

La interfaz de MLflow quedó disponible en:

```text
http://localhost:8000
```

## Entrenamiento

El entrenamiento se realiza mediante `clf-train-registry.py`.

Se utiliza el dataset `Breast Cancer` de `scikit-learn` y se divide en datos de entrenamiento y prueba utilizando una proporción de 80/20.

El modelo utilizado es:

```text
RandomForestClassifier
n_estimators=100
min_samples_leaf=2
class_weight=balanced
random_state=123
```

También se genera `test.csv`, que contiene los datos utilizados posteriormente para probar el modelo desplegado.

## Tracking

El entrenamiento crea el experimento:

```text
my-experiment
```

El run obtenido durante la ejecución fue:

```text
marvelous-stoat-40
```

Las métricas registradas fueron:

| Métrica        |          Resultado |
| -------------- | -----------------: |
| accuracy_train | 0.9978021978021978 |
| accuracy_test  | 0.9473684210526315 |

El modelo también queda registrado como artefacto del run.

## Model Registry

El modelo se registra en MLflow Model Registry con el nombre:

```text
clf-model
```

La ejecución generó la versión:

```text
v1
```

A esta versión se le asignó el alias:

```text
Staging
```

Por lo tanto, el modelo utilizado para el despliegue se referencia como:

```text
models:/clf-model@Staging
```

## Despliegue

El script `serveModel.sh` se utiliza para iniciar el servidor del modelo:

```bash
mlflow models serve -m $1 -p 1234 -h 0.0.0.0 --env-manager local
```

El modelo quedó disponible en el puerto `1234`.

## Inferencia

Las predicciones se realizaron utilizando:

```bash
./predict.sh test.csv
```

El script envía el archivo al endpoint:

```text
http://localhost:1234/invocations
```

mediante una petición HTTP con `Content-Type: text/csv`.

El servidor respondió correctamente con las predicciones en formato JSON.

## Relación con el ciclo de vida

El flujo realizado puede resumirse de la siguiente manera:

```text
Datos
  ↓
Preparación
  ↓
Entrenamiento
  ↓
Evaluación
  ↓
Tracking
  ↓
Registro y versionamiento
  ↓
Despliegue
  ↓
Inferencia
```

En este caso, MLflow se utilizó principalmente para el tracking del entrenamiento y para registrar y versionar el modelo antes de desplegarlo.

## Resultado

Se completó el flujo de entrenamiento, tracking, registro, despliegue e inferencia utilizando Docker y MLflow.

## Limpieza del entorno 

Para detener los servicios de Docker:

```bash
docker compose down
```
