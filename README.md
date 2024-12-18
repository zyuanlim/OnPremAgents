<p align = "center" draggable=”false” ><img src="https://github.com/AI-Maker-Space/LLM-Dev-101/assets/37101144/d1343317-fa2f-41e1-8af1-1dbb18399719" 
     width="200px"
     height="auto"/>
</p>

# AI Makerspace: On Prem Agents with LangGraph Platform and Ollama ft. Research Rabbit 🐰

From LangChain's [Research Rabbit](https://github.com/langchain-ai/research-rabbit) repository: 

> Research Rabbit is a web research and summarization assistant that autonomously goes down the rabbit-hole of any user-defined topic. It uses an LLM to generate a search query based on the user's topic, gets web search results, and uses an LLM to summarize the results. It then uses an LLM to reflect on the summary, examines knowledge gaps, and generates a new search query to fill the gaps. This repeats for a user-defined number of cycles, updating the summary with new information from web search and provided the user a final markdown summary with all sources used. It is configured to run with fully local LLMs (via [Ollama](https://ollama.com/search))! 

You can read more about LangChain's Research Rabbit [here](https://github.com/langchain-ai/research-rabbit)!

## 🏗️ Using LangGraph Platform Self-Hosted Lite to Deploy LangGraph Application

We'll be using LangChain's Self-Hosted Lite version of LangGraph Platform to deploy our LangGraph application today. 

In order to make things slightly easier - we're going to leverage the `docker compose` method, let's get into what we need to do!

### 🐳 Build a Local Docker Image

The first thing we'll want to do is as follows:

```bash
uv add langgraph-cli
```

> NOTE: If you don't have `uv` installed, you can get it with: ```pip install uv```

Now, we can build the image from our root repository directory with:

```bash
langgraph build -t research-rabbit
```

### 🐋🎵 Spinning Up with Docker Compose

Next, we can create our `docker-compose.yaml` as follows:

```bash
touch docker-compose.yaml
```

Edit the created file with your favourite text editor to reflect the following:

```
volumes:
    langgraph-data:
        driver: local
services:
    langgraph-redis:
        image: redis:6
        healthcheck:
            test: redis-cli ping
            interval: 5s
            timeout: 1s
            retries: 5
    langgraph-postgres:
        image: postgres:16
        ports:
            - "5433:5432"
        environment:
            POSTGRES_DB: postgres
            POSTGRES_USER: postgres
            POSTGRES_PASSWORD: postgres
        volumes:
            - langgraph-data:/var/lib/postgresql/data
        healthcheck:
            test: pg_isready -U postgres
            start_period: 10s
            timeout: 1s
            retries: 5
            interval: 5s
    langgraph-api:
        image: ${IMAGE_NAME}
        ports:
            - "8123:8000"
        depends_on:
            langgraph-redis:
                condition: service_healthy
            langgraph-postgres:
                condition: service_healthy
        env_file:
            - .env
        environment:
            REDIS_URI: redis://langgraph-redis:6379
            LANGSMITH_API_KEY: ${LANGSMITH_API_KEY}
            TAVILY_API_KEY: ${TAVILY_API_KEY}
            POSTGRES_URI: postgres://postgres:postgres@langgraph-postgres:5432/postgres?sslmode=disable
```

Notice that this relies on a `.env` file. Let's fix that now!

Run the following command to copy the sample `.env` file.

```
cp .env.sample .env
```

Use your favourite text editor to include your own LangSmith API key and Tavily API key!

## 🎉 Conclusion

Now you should have a working instance of your LangGraph application! 

You can access your newly deployed system as follows:

- [http://localhost:8123/docs](http://localhost:8123/docs)
- [https://smith.langchain.com/studio/?baseUrl=127.0.0.1:8123][https://smith.langchain.com/studio/?baseUrl=127.0.0.1:8123]
