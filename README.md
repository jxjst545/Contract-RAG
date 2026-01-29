# Contract-RAG
Retrieval-Augmented Generation (RAG) is an advanced AI framework that enhances large language models by incorporating information retrieved from external sources during a user’s search. Instead of relying solely on the model’s vast knowledge, RAG performs a semantic search across a specific knowledge base such as documents, databases, or vector stores and adds the most relevant excerpts into the model for additional context. This process significantly improves search accuracy, reduces LLM hallucinations, and enables organizations to build and use AI assistants that can work confidently using their own information for search criteria.
Modern LLMs are great at generating fluent text in context, but they are inherently limited by the data they are trained on. Organizations have proprietary information not available to LLMs data sourcing. When asked about internal policies or contractual details, standalone LLMs often hallucinate or default to generic assumptions, which is an unacceptable risk in compliance-driven organizations.
RAG addresses these limitations through two core components:
Retrieval System: Searches a structured knowledge base to identify the most relevant passages for a given query using semantic similarity.
Generative Model: Uses these retrieved passages as authoritative context to produce accurate, document-aware responses.
This combination produces outputs in trusted sources, dramatically improving reliability for precision-critical tasks such as contract analysis, regulatory compliance, and legal review.
To achieve efficient retrieval, RAG systems rely on text embeddings. An embedding model converts each text chunk into a high-dimensional vector representation where semantically similar content clusters together. When a query is issued, it is encoded into a vector and compared against stored vectors using similarity search techniques (typically cosine similarity) ensuring that the most relevant information is found quickly and accurately.

Contract RAG: Semantic Search for Contract Specifications
Organizations increasingly rely on extensive contract databases to manage relationships with customers, suppliers, and partners. At Calgon Carbon, these documents are scattered across folders, shared drives, applications, and cloud repositories. Prior to the COVID pandemic in 2020, these documents were physical copies, saved in filing cabinets. While progress was made to save these files digitally over the last 5 years, and cloud storage is inexpensive, locating specific information within contracts remains costly. Employees typically must resort to manual keyword searches and repetitive scanning to answer questions such as "What indemnification language did we use for this customer?" or "Which contracts require a 100% performance bond?"
As our contract databases grow, this manual approach becomes slow, inconsistent, and prone to human error. Critical information and contract language can be overlooked, and teams may inadvertently renegotiate terms that already exist somewhere else. Traditional keyword searches struggle with synonyms and varied legal phrasing, where identical concepts appear under different wording. This limits us from reusing best-practice clauses and increases the risk of introducing new, non-standard, terms.

Cosine Similarity
This project uses cosine similarity to measure how closely two text embeddings align in meaning. Cosine similarity compares the angle between two vectors rather than their magnitude, making it ideal for semantic search. Scores near 1 indicate strong similarity, while scores near 0 indicate little or no similarity. This approach is well-suited for RAG systems where the goal is to find text chunks that best match a user query.
For two vectors a and b, cosine similarity is:
cos(θ) = (a · b) / (||a|| · ||b||)
where a · b is the dot product and ||a||, ||b|| are Euclidean norms. A cosine similarity near 1 indicates semantically related texts.
After retrieving the top-k contract chunks, they are combined into a context window for the language model. Even without generating new text, returning exact chunks with source attribution is a major improvement over manual search.

System Design and Implementation
Contract RAG is designed as a modular pipeline adaptable from local file storage to enterprise systems like Box or SharePoint. The system comprises:
1.	Document Ingestion: The system recursively scans directories for contract files (PDF and TXT formats in this prototype).
2.	Text Extraction: Each contract is converted to raw text using pypdf or file reading, producing documents with associated file paths.
3.	Chunking: Contracts are broken into overlapping chunks of approximately 800 characters with 200-character overlap. Overlap ensures clauses spanning boundaries appear together in at least one chunk.
4.	Embedding and Indexing: Each chunk passes through a SentenceTransformer embedding model (all-MiniLM-L6-v2) to obtain dense vector representations. The prototype stores these in memory with metadata (chunk ID, file path). Production deployments could use FAISS, Chroma, or managed vector stores.
5.	Retrieval: When users submit natural-language queries (e.g., "Find clauses related to liquidated damages"), the system encodes the query and computes cosine similarity with all chunk embeddings, returning the top-k most relevant passages.
6.	Presentation: The current implementation assembles the question and retrieved chunks into labeled output. Advanced versions would use language models to summarize clauses, highlight key obligations, and compare across contracts.
This modular design facilitates experimentation with alternative chunking strategies, embedding models, or storage backends while preserving the core RAG architecture.

Implementation Details
The prototype is implemented in Python as a Jupyter notebook using:
•	pypdf for PDF text extraction
•	sentence-transformers for text embeddings
•	numpy for vector operations and cosine similarity
•	matplotlib for visualization
The SimpleVectorIndex class encodes all chunks into embeddings and implements cosine similarity retrieval through matrix multiplication. Visualizations include chunk-length histograms and similarity-score bar charts for transparency and stakeholder communication.

Experimental Results
The system was tested on sample public works municipal contracts from New Jersey municipalities containing standard clauses (indemnification, liquidated damages, warranties, bonds). Test queries included:
•	"Find clauses related to liquidated damages or delay penalties"
•	"Show indemnification language"
•	"Which contracts mention contractor registration”

Retrieved chunks consistently included correct clauses or closely related sections. Notably, the liquidated damages query surfaced relevant clauses even when exact phrases were not used, demonstrating semantic understanding beyond keyword matching.
Similarity-score visualizations revealed that high-confidence matches showed significantly higher scores than alternatives, while distributed scores indicated ambiguous queries or repetitive language. These qualitative results demonstrate that embedding-based retrieval substantially outperforms keyword search with paraphrased or varied legal language.

Limitations and Future Work
The current prototype operates on a small, in-memory index optimized for (up to) hundreds of contracts. Scaling to thousands or millions of documents will require robust vector databases with support for efficient indexing, sharding, and low-latency queries to maintain performance at scale.
Access control is another critical gap. The current implementation does not enforce authentication or authorization. A production-ready system must integrate with enterprise identity providers and implement role-based visibility to ensure confidentiality and compliance with organizational security policies. For this example, I only intend to use municipal contracts for simplicity.
Answer generation is minimal in the prototype, the greatest value will come from language model summarization with source citation, which introduces challenges such as controlling hallucinations, designing effective prompts, and implementing audit logging for traceability.
Chunking is currently character-based and ignores document structure. This limits retrieval quality and readability. Future improvements include clause-aware segmentation that respects headings, numbered sections, and logical boundaries within contracts.
Planned extensions include integration with Box and SharePoint APIs for seamless document ingestion, development of web and chat interfaces for user interaction, analytics on frequently searched clauses to identify trends, and full generative capabilities for summarization, comparison, and drafting.

Conclusion
Contract RAG demonstrates the potential of Retrieval-Augmented Generation (RAG) as a practical solution for semantic clause-level search across a large contract repository. Contract RAG combines text embeddings, cosine similarity retrieval, and Python architecture, to create a system that enables users to ask natural-language questions and quickly find relevant clauses from multiple contracts, quickly.
The prototype highlights clear advantages over traditional manual review and keyword-based search. Local embedding models paired with in-memory indexing deliver faster, more accurate results, significantly reducing the time and effort required to locate critical provisions.
Although additional work is needed to achieve secure enterprise deployment, scale indexing, and integrate advanced generative capabilities, the core RAG approach is both technically feasible and strategically valuable. It establishes a foundation for improving clause consistency, accelerating contract analysis, and unlocking greater value from existing contract portfolios, ultimately transforming how my organization can manage and leverage contractual data.
