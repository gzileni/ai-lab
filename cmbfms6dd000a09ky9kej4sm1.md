---
title: "Unlock the Power of AI: Stream, Search, and Log with py_agent_search"
seoTitle: "AI Toolkit: Streamline Tasks with py_agent_search"
seoDescription: "Discover py_agent_search: A ReAct agent for web, Wikipedia, YouTube search, with Redis memory and Grafana logging for seamless AI integration"
datePublished: Mon Jun 02 2025 21:57:54 GMT+0000 (Coordinated Universal Time)
cuid: cmbfms6dd000a09ky9kej4sm1
slug: unlock-ai-py-agent-search
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1748901407247/a999cf52-3d79-4a8b-ba57-55f982d3e892.png
tags: langchain, langgraph, agentic-ai

---

Are you a Python developer seeking a seamless AI agent that can search the web, Wikipedia, and YouTube— streaming results token by token — while maintaining conversation context in Redis and logging everything to Grafana? Discover **py\_agent\_search**, a LangGraph-powered “ReAct” agent designed to do just that. Here's a quick overview of what you need to know to get started and how it can enhance your workflow.

---

## What Is py\_agent\_search?

* **LangGraph + LangChain**: Under the hood, `py_agent_search` uses LangGraph’s `create_react_agent` to orchestrate the agent’s reasoning (ReAct pattern) and LangChain’s `ChatOpenAI` (or your preferred LLM) to generate answers.
    
* **Streaming Interface**: Tokens stream back in real time via a custom `StreamingCallbackHandler`. As each token arrives, it’s logged to Loki—and you can view the entire exchange in Grafana.
    
* **Search Tools Built-In**:
    
    * **DuckDuckGoSearchRun** for generic web queries
        
    * **WikipediaQueryRun** for encyclopedia lookups
        
    * **YouTubeSearchTool** for finding relevant videos
        
* **Persistent Memory**: Conversation history lives in Redis (via `AsyncRedisSaver`), so your agent “remembers” previous chats.
    
* **Centralized Logging**: All events (LLM start, new token, LLM end, tool invocations, errors) go to a Loki instance. Grafana dashboards let you slice and dice those logs however you like.
    

---

## Why You’ll Love It

1. **Zero-Fuss Deployment**
    
    * If you already run Docker, a single `docker-compose up -d` command spins up Redis, Loki, Promtail, and Grafana (see the provided `docker-compose.yml`).
        
    * Grafana’s preconfigured Loki datasource means you can jump straight to querying `{application="agent_search"} |= "" | json` once Grafana is live at [`http://localhost:3000`](http://localhost:3000).
        
2. **Simple Environment Configuration**
    
    * Add a `.env` file (or set ENV vars) for Redis (`REDIS_HOST`, `REDIS_PORT`, `REDIS_DB`), Loki (`LOKI_URL`), and your `OPEN_API_KEY` if you want to use OpenAI.
        
    * Defaults will work out of the box if you stick to [`localhost`](http://localhost) for everything.
        
3. **Plug-and-Play with pip**
    
    ```bash
    pip install py_agent_search
    ```
    
    * You get LangChain, LangGraph, Redis client, Loki logger handler, and all the community search tools—nothing else to install.
        
4. **Developer-Friendly Example**
    
    * Check out [`main.py`](http://main.py) for a minimal async example: instantiate `AgentSearch(...)`, call [`agent.stream`](http://agent.stream)`(input=question, thread_id=uuid4())`, and watch tokens print as they arrive.
        
    * Every callback (start, token, end) invokes `send_log(message, metadata)` so you can debug or audit any part of the flow.
        

---

## Quick Start

1. **Clone or install**:
    
    ```bash
    git clone https://github.com/your-repo/py_agent_search.git
    cd py_agent_search
    pip install py_agent_search
    ```
    
2. **Create a** `.env` at the project root:
    
    ```plaintext
    REDIS_HOST=localhost
    REDIS_PORT=6379
    REDIS_DB=0
    
    LOKI_URL=http://localhost:3100/loki/api/v1/push
    
    APP_ENV=development
    
    # If you plan to use OpenAI:
    OPEN_API_KEY=your_openai_api_key
    ```
    
3. **Start Docker services**:
    
    ```bash
    docker-compose up -d
    ```
    
    * Redis runs on [`localhost:6379`](http://localhost:6379)
        
    * Loki runs on [`localhost:3100`](http://localhost:3100)
        
    * Grafana runs on [`http://localhost:3000`](http://localhost:3000) (look for application logs via `{application="agent_search"} |= "" | json`)
        
4. **Run the example**:
    
    ```python
    import asyncio
    import uuid
    import os
    import argparse
    from py_agent_search import AgentSearch, send_log
    
    redis_conf = {
        "host": os.getenv("REDIS_HOST", "localhost"),
        "port": os.getenv("REDIS_PORT", 6379),
        "db": os.getenv("REDIS_DB", 0)
    }
    
    async def main(question):
        """
        Main function to run the chat interaction.
        It initializes the Chat class and calls the chat method with a question and context.
        """
        try:
            thread_id = str(uuid.uuid4())
            
            if not question:
                question = input("Enter your question: ")
            print("You can type 'exit' to quit the chat.")
            if question.lower() == 'exit':
                return
            # Send the initial log message
            send_log(message="Starting chat interaction", metadata={"question": question})
            print(f"Question: {question}\n Waiting for response...")
            
            agent = AgentSearch(redis_persistence_config=redis_conf)
            async for token in agent.stream(input=question, thread_id=thread_id):
                print(token, end="", flush=True)
        except Exception as e:
            print(f"An error occurred: {e}")
            send_log(message="Error during chat interaction", metadata={"error": str(e)})
        
    if __name__ == "__main__":
        parser = argparse.ArgumentParser(description="Chat with AgentSearch")
        parser.add_argument("--question", type=str, help="Question to ask the agent")
        args = parser.parse_args()
        asyncio.run(main(args.question))
    ```
    
    * Type a question when prompted
        
    * Watch the agent fetch info from the web/Wikipedia/YouTube and stream tokens back
        

The first question I asked the AI agent was, “Find description and video about Machine Learning.” Because the agent has memory and retains its previous responses, my follow-up questions all referred back to that initial answer. Specifically, I then asked, “Give me detailed information,” and finally requested, “Give me the summary of Video 1.”.

```python
python test.py --question="find description and video about Machine Learning"
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748898693372/dba8a689-1291-4bca-b0f4-f8dd6162e045.png align="center")

```python
python test.py --question="give me detailed information"
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748898739892/3e31f52d-235a-49a0-8bf0-28eec62cb083.png align="center")

```python
python test.py --question="give me resume Video 1"
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748898777267/13da2def-1203-4a95-b8f9-5d4fb4691371.png align="center")

### **Inspect Logs in Grafana**:

* * Open [`http://localhost:3000`](http://localhost:3000)
        
        * In Explore → Loki, run:
            
            ```plaintext
            {application="agent_search"} |= "" | json
            ```
            
        * You’ll see structured log events (timestamps, metadata, LLM/tool calls) in real time.
            

---

## How It All Fits Together

* [`agent.py`](https://github.com/gzileni/qi_agent_search/blob/main/py_agent_search/agent.py)  
    Defines `AgentSearch`, ties LangGraph’s ReAct agent to your Redis memory backend and all the search tools. When you call [`agent.stream`](http://agent.stream)`(...)`, it:
    
    1. Checks Redis for conversation history (if any).
        
    2. Uses LangGraph to decide whether to call DuckDuckGo, Wikipedia, YouTube, or the LLM.
        
    3. Streams tokens back to your console (and to Grafana via Loki).
        
* [`stream.py`](https://github.com/gzileni/qi_agent_search/blob/main/py_agent_search/stream.py)  
    Houses `StreamingCallbackHandler` (inherits from `BaseCallbackHandler`). Its job is to intercept every LLM callback:
    
    * `on_llm_start`: log metadata
        
    * `on_llm_new_token`: buffer tokens (and optionally stream them to the console)
        
    * `on_llm_end`: send a final log event
        
* [`memory.py`](https://github.com/gzileni/qi_agent_search/blob/main/py_agent_search/memory.py)  
    Wraps `AsyncRedisSaver` + `CheckpointSaver` from LangGraph. Each time the agent responds, it checkpoints conversation state in Redis. Next time you pick up the same `thread_id`, the agent “remembers” what happened before.
    
* [`tools.py`](https://github.com/gzileni/qi_agent_search/blob/main/py_agent_search/tools.py)  
    Instantiates three search tools:
    
    ```python
    handler = StreamingCallbackHandler()
    
    DuckDuckGoSearchRun( name="duckduckgo_search", callbacks=[handler], … )
    WikipediaQueryRun( name="wikipedia_search", callbacks=[handler], … )
    YouTubeSearchTool( name="youtube_search", callbacks=[handler], … )
    ```
    
    Each tool fetches content and uses the same `StreamingCallbackHandler` for token-level logging.
    
* [`log.py`](https://github.com/gzileni/qi_agent_search/blob/main/py_agent_search/log.py)  
    Centralizes logging logic. Exports a single function:
    
    ```python
    def send_log(message: str, metadata: dict = {}) -> None
    ```
    
    Behind the scenes, it configures `LokiLoggerHandler` (sending JSON to Loki) and a console handler for non-production environments.
    
* [test.py](https://github.com/gzileni/qi_agent_search/blob/main/py_agent_search/test.py)  
    A minimal async example showing how to tie it all together:
    
    1. Read `.env`
        
    2. Build a `redis_conf` dict
        
    3. Instantiate `AgentSearch(redis_persistence_config=redis_conf)`
        
    4. Ask the user for input and `async for token in` [`agent.stream`](http://agent.stream)`(...)` to print tokens
        

---

## In a Nutshell

If you want a Python-native, production-ready AI agent that:

* Automatically chooses when to invoke web/Wikipedia/YouTube searches
    
* Streams LLM tokens back in real time
    
* Persists conversation history in Redis
    
* Ships detailed logs to Loki (viewable in Grafana)
    
* Installs with a single `pip install py_agent_search`
    
* Spins up all dependencies with one `docker-compose up -d`
    

…then **py\_agent\_search** is exactly what you’re looking for. Give it a spin, tweak the tools or LLM, and integrate it into your chatbot, data-research pipeline, or any other Python project where you need an AI agent with built-in search and persistence.

## Conclusion

In conclusion, **py\_agent\_search** offers a robust solution for Python developers seeking an AI agent capable of performing web, Wikipedia, and YouTube searches while maintaining conversation context and logging activities. With its seamless integration of LangGraph, LangChain, Redis, and Grafana, it provides a comprehensive and efficient tool for real-time information retrieval and analysis. Its ease of deployment, simple configuration, and developer-friendly features make it an ideal choice for enhancing workflows in various Python projects. Whether you're building a chatbot, a data-research pipeline, or any application requiring an AI agent with built-in search and persistence, **py\_agent\_search** is a powerful and versatile option.

## Links

* [GitHub Repository - qi\_agent\_search](https://github.com/gzileni/qi_agent_search)