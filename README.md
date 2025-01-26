# Chat with GPT models - FastAPI backend

This repository includes a simple Python FastAPI app that streams responses from Azure OpenAI GPT models.

## Opening the project

1. Create a [Python virtual environment](https://docs.python.org/3/tutorial/venv.html#creating-virtual-environments) and activate it.

2. Install the requirements:

   ```shell
   python3 -m pip install -r requirements-dev.txt
   ```

3. Install the pre-commit hooks:

   ```shell
   pre-commit install
   ```

## Deployment

This repo is set up for deployment on Azure Container Apps using the configuration files in the `infra` folder.

### Deployment from scratch

1. Login to Azure:

   ```shell
   azd auth login
   ```

2. Provision and deploy all the resources:

   ```shell
   azd up
   ```

3. When `azd` has finished deploying, you'll see an endpoint URI in the command output. Visit that URI, and you should see the chat app! 🎉
4. When you've made any changes to the app code, you can just run:

   ```shell
   azd deploy
   ```

### Adding a frontend

You can pair this backend with a frontend of your choice.
The frontend needs to be able to read NDJSON from a ReadableStream,
and send JSON to the backend with an HTTP POST request.

To pair a frontend with this backend, you'll need to:

1. Deploy the backend using the steps above. Make sure to note the endpoint URI.
2. Open the frontend project.
3. Deploy the frontend using the steps in the frontend repo, following their instructions for setting the backend endpoint URI.
4. Open this project again.
5. Run `azd env set ALLOWED_ORIGINS "https://<your-frontend-url>"`. That URL (or list of URLs) will specified in the CORS policy for the backend to allow requests from your frontend.
6. Run `azd up` to deploy the backend with the new CORS policy.

## Local development (without Docker)

Assuming you've run the steps in [Opening the project](#opening-the-project) and have run `azd up`, you can now run the FastAPI app locally using the `uvicorn` server:

```shell
python -m uvicorn "src.api:create_app" --reload --factory
```

### Using a local LLM server

Define `LOCAL_OPENAI_ENDPOINT` in your `.env` file.

For example, to point at a local llamafile server running on its default port:

```shell
LOCAL_OPENAI_ENDPOINT="http://localhost:8080/v1"
```

If you're running inside a dev container, use this local URL instead:

```shell
LOCAL_OPENAI_ENDPOINT="http://host.docker.internal:8080/v1"
```

## Local development with Docker

In addition to the `Dockerfile` that's used in production, this repo includes a `docker-compose.yaml` for
local development which creates a volume for the app code. That allows you to make changes to the code
and see them instantly.

1. Install [Docker Desktop](https://www.docker.com/products/docker-desktop/). If you opened this inside Github Codespaces or a Dev Container in VS Code, installation is not needed. ⚠️ If you're on an Apple M1/M2, you won't be able to run `docker` commands inside a Dev Container; either use Codespaces or do not open the Dev Container.

2. Make sure that the `.env` file exists. The `azd up` deployment step should have created it.

3. Add your Azure OpenAI API key to the `.env` file if not already there. You can find your key in the Azure Portal for the OpenAI resource, under _Keys and Endpoint_ tab.

   ```shell
   AZURE_OPENAI_KEY="<your-key-here>"
   ```

4. Start the services with this command:

   ```shell
   docker-compose up --build
   ```

5. Click 'http://0.0.0.0:3100' in the terminal, which should open a new tab in the browser. You may need to navigate to 'http://localhost:3100' if that URL doesn't work.
