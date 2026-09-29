# Income Detection — Training and Deployment

![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)
![Python](https://img.shields.io/badge/Python-3.9%2B-blue.svg)
![FastAPI](https://img.shields.io/badge/FastAPI-0.95%2B-009688.svg)
![CatBoost](https://img.shields.io/badge/CatBoost-yellow.svg)
![Docker](https://img.shields.io/badge/Docker-2496ED.svg?logo=docker&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5.svg?logo=kubernetes&logoColor=white)

An end-to-end MLOps example: train a CatBoost classifier on the UCI Census Income dataset, serve it behind a FastAPI endpoint, containerize it with Docker, deploy to Kubernetes, and load-test with Locust.

The dataset is [Census Income (KDD)](https://archive.ics.uci.edu/ml/datasets/Census-Income+(KDD)). The target variable is `income` — whether a person earns above or below $50k/yr.

**Stack:** CatBoost · scikit-learn · FastAPI · Pydantic · Docker · Kubernetes · Locust · Terraform

## Directory Explanation

The repo is split into four parts:

```
income_detection_api
├── app/         FastAPI service + trained model artifacts
├── assets/      images used by this README
├── test/        API and load tests
└── train/       training, EDA, evaluation, and the dataset
```

Details:

| Path | Contents |
| --- | --- |
| `app/main.py` | FastAPI app; `GET /` and `POST /predict` |
| `app/person.py` | Pydantic request/response validation model |
| `app/catboost_income_classifier.cbm` | trained CatBoost model |
| `app/sklearn_income_classifier.pkl` | trained scikit-learn pipeline (not wired into the API) |
| `train/catboost_model_training.py` | downloads the data, trains CatBoost, exports the model |
| `train/sklearn_model_training.py` | scikit-learn pipeline variant |
| `train/explore.py`, `train/evaluate.py` | EDA and model evaluation |
| `test/api_test.py` | plain-Python request test |
| `test/perf.py` | Locust load test |
| `Dockerfile`, `deployment.yaml` | container image and Kubernetes deployment |
| `terraform/main.tf` | Terraform for the AWS-side infrastructure |

## Instructions

### 1. Training

Training related files are under the `/train` folder. For training the model, catboost_model_training.py can be used. Helper function download_census_data downloads and saves the data under data folder. Then the scripts builds catboost classification model. Then, we add model columns into the model and save the model to the "app" folder.

Additionally, one can explore and evaluate the data using the explore.py and evaluate.py files.

If you want to fit a sklearn model with a pipeline, you can use sklearn_model_training.py.

### 2. Deployment

Most deployment related files are under the `/app` folder.

if you want to run fast api directly without Docker or Kubernetes, run the following:

```
cd income_detection_api
python -m venv env
source env/bin/activate
pip install -r requirements.txt
cd app
python main.py
```

To containerize it with Docker and serving it, run the following:

```
docker buildx build --platform=linux/amd64 -t myimage .
docker run -d --platform=linux/amd64 --name mycontainer -p 8000:8000 myimage
```

I have also uploaded the image to Docker Hub. The link is here: <https://hub.docker.com/r/jbaktir/income_detection>

To orchestrate with Kubernetes:

```
minikube start
kubectl apply -f deployment.yaml
```

Trouble shooting for Kubernetes:

```
kubectl get services  #to check the status
minikube tunnel       #to open up the port for the load balancer

kubectl run -it --rm --restart=Never busybox --image=gcr.io/google-containers/busybox sh
# to log into a specific pod and doing curl
kubectl run curl-image --image=radial/busyboxplus:curl -i --tty --rm curl 10.100.247.207:6000

# sometimes the relevant port gets blocked by the system
# to get around that I do port-forwarding with this code
kubectl port-forward service/income-detection-service 6600:6000
```

to get around the port already is use problem, you can kill process that uses the port of interest

```
kill -9 $(lsof -ti:6000)
```

### 3. Testing

for load testing:

```
cd test
locust -f perf.py
```

Also, for simple python requests you can use `api_test.py`

## Running on a remote host

The steps above assume a local machine. To run on an EC2 instance or any other host:

```bash
git clone https://github.com/jbaktir/income_detection_api.git
cd income_detection_api
python3 -m venv env
source env/bin/activate
pip install -r requirements.txt
python3 app/main.py
```

`app/main.py` serves on port `80`, so the API is reachable on the instance's port `80` (`http://<host>/docs` for the OpenAPI UI). Keep it behind a reverse proxy and terminate TLS there if you expose it publicly.

## Visuals

When the app is up and running, one should see the following visuals.

### main page

![](assets/markdown-img-paste-20220123182622901.png)

### FastAPI documentation

![](assets/markdown-img-paste-20220123182816587.png)

### FastAPI sample request

![](assets/markdown-img-paste-20220123182956152.png)

### Successful Response

![](assets/markdown-img-paste-20220123192027754.png)

### Validation Control

![](assets/markdown-img-paste-20220123192346994.png)

### Load Test

![](assets/markdown-img-paste-20220123192759809.png)
