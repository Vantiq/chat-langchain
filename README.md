# 🦜️🔗 ChatLangChain

This repo is an implementation of a locally hosted chatbot specifically focused on question answering over the [LangChain documentation](https://langchain.readthedocs.io/en/latest/).
Built with [LangChain](https://github.com/hwchase17/langchain/) and [FastAPI](https://fastapi.tiangolo.com/).

The app leverages LangChain's streaming support and async API to update the page in real time for multiple users.

## Vantiq Notes:

This is a fork of the original ChatLangChain project.  It provides a good example of using LangChain to solve a "semantic search" problem over specific content. The following changes have been made:

1. To run any of the code you will want to create a `.env` file at the root of the repo.  This file should have the following contents: `OPENAI_API_KEY=<Vantiq OpenAI key>`.  The location of this key has been sent to the Vantiq AI teams channel.
1. `ingest.py` has been changed to ingest the `rules.md` file from our documentation.  The code assumes that this repo was cloned as peer to our docs repo.  It creates a file called `rules.pkl` that is used by the app.  This file is not checked in, but a pointer to a copy was sent to the Teams channel (generation of the file does cost money, though I'm not sure how much at this point).
1. Some of the code has been changed to address API changes since this was originally published.

We recommend the use of a virtual environment for running this code (or any python project really). There are a few ways to do this, so if you'd like some setup help please reach out to the Vantiq AI team. The instructions below rely on `make`, but you can also just run `main.py` from your IDE if you prefer.

## ✅ Running locally
1. Install dependencies: `pip install -r requirements.txt`
1. Run `ingest.sh` to ingest LangChain docs data into the vectorstore (only needs to be done once).
   1. You can use other [Document Loaders](https://langchain.readthedocs.io/en/latest/modules/document_loaders.html) to load your own data into the vectorstore.
1. Run the app: `make start`
   1. To enable tracing, make sure `langchain-server` is running locally and pass `tracing=True` to `get_chain` in `main.py`. You can find more documentation [here](https://langchain.readthedocs.io/en/latest/tracing.html).
1. Open [localhost:9000](http://localhost:9000) in your browser.

## 🚀 Important Links

Deployed version (to be updated soon): [chat.langchain.dev](https://chat.langchain.dev)

Hugging Face Space (to be updated soon): [huggingface.co/spaces/hwchase17/chat-langchain](https://huggingface.co/spaces/hwchase17/chat-langchain)

Blog Posts: 
* [Initial Launch](https://blog.langchain.dev/langchain-chat/)
* [Streaming Support](https://blog.langchain.dev/streaming-support-in-langchain/)

## 📚 Technical description

There are two components: ingestion and question-answering.

Ingestion has the following steps:

1. Pull html from documentation site
2. Load html with LangChain's [ReadTheDocs Loader](https://langchain.readthedocs.io/en/latest/modules/document_loaders/examples/readthedocs_documentation.html)
3. Split documents with LangChain's [TextSplitter](https://langchain.readthedocs.io/en/latest/reference/modules/text_splitter.html)
4. Create a vectorstore of embeddings, using LangChain's [vectorstore wrapper](https://python.langchain.com/en/latest/modules/indexes/vectorstores.html) (with OpenAI's embeddings and FAISS vectorstore).

Question-Answering has the following steps, all handled by [ChatVectorDBChain](https://langchain.readthedocs.io/en/latest/modules/indexes/chain_examples/chat_vector_db.html):

1. Given the chat history and new user input, determine what a standalone question would be (using GPT-3).
2. Given that standalone question, look up relevant documents from the vectorstore.
3. Pass the standalone question and relevant documents to GPT-3 to generate a final answer.
