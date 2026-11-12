Decision
========

Purpose
-------
Use pretrained decision models for sentiment analysis and, optionally, person name extraction.
Reuse the Web service interfaces from the first and third exercises,
and explore how decision models differ from general-purpose generative
LLMs. 

Task
----
Construct a Docker/Podman container that exposes a FastAPI Web service on port 8000.
The service must score Danish and English course evaluations for sentiment.

Choose one of the following approaches:

- Use the Jev-compatible System One endpoint on CampusAI with the model alias `decision`.
- Run a pretrained model locally, for example a suitable GLiNER2 or GLiNER2.5 checkpoint.

You only need to implement one approach. Sentiment predictions must come from the chosen model.
You may reuse code from your earlier exercises.

The 200 MB container size limit from the first exercise does not apply to this exercise.

Required endpoint: sentiment
---------------------------
Implement `POST /v1/sentiment` with the same interface as in the first exercise.

The input is a JSON object with a string field `text`, for example:

```json
{"text": "Det var en god lærer."}
```

The response must be a JSON object with a numeric field `score`:

```json
{"score": 3}
```

The score must be a float or integer between `-5` and `5`, with the following interpretation:

| Score | Interpretation |
| --- | --- |
| -5 | Extremely negative |
| -3 | Negative |
| 0 | Neutral |
| 3 | Positive |
| 5 | Extremely positive |

A three-category classifier whose predictions are mapped to `-3`, `0`, and `3` is sufficient.
You may use a finer scale or a probability-weighted score instead.
Document how you convert the model's output to your sentiment score.

Use the following reference examples when testing:

| Text | Expected score |
| --- | --- |
| Det var en god lærer. | 3 |
| It was a bad course | -3 |
| It was a very dry course and I did not learn much. | -3 |
| The course takes place on Tuesdays. | 0 |

Optional endpoint: person extraction
-----------------------------------
Implement `POST /v1/extract-persons` with the same interface as in the third exercise.

The input is a JSON object with a string field `text`:

```json
{"text": "Einstein and von Neumann meet each other."}
```

The response must be a JSON object with a field `persons` containing a list of strings:

```json
{"persons": ["Einstein", "von Neumann"]}
```

Each string must be a person name appearing in the input text. Preserve the spelling used in the text.
If no person names are found, return:

```json
{"persons": []}
```

Use a model that supports entity extraction. You may use the same
model as for sentiment analysis or a separate local model. 

The CampusAI `decision` endpoint offers typed decisions such as
`choice`, `score`, and `noul`; these do not directly return arbitrary
person name spans. 
A local GLiNER model with entity extraction support is one option for this extension.
Check the capabilities of the particular checkpoint you select.

Using CampusAI
--------------
The API base URL is:

```text
https://api.campusai.compute.dtu.dk/v1
```

The decision endpoint is `POST /v1/systemone`, with model alias `decision`.
Its request format uses `state` for the input and `questions` for the decisions to make.
See section 13.5 of the course notes and the API documentation listed below.

Keep your API key outside the repository and container image. For example, use a `~/.env` file containing:

```dotenv
CAMPUSAI_API_KEY=sk-...
CAMPUSAI_API_URL=https://api.campusai.compute.dtu.dk/v1
CAMPUSAI_DECISION_MODEL=decision
```

Build and run the container with:

```bash
docker build -t decision:latest .
docker run --rm -p 8000:8000 --env-file ~/.env decision:latest
```

You may use Podman instead of Docker.
Others running your service must supply their own API key.

For a local implementation, a CampusAI API key is not needed.
Document how model weights are obtained and any required model cache or volume.

Implementation requirements
---------------------------
- Expose the service on port 8000 and provide Swagger documentation at `/docs`.
- Support both Danish and English input.
- Initialise local models once per server process, rather than loading them for every request.
- Keep dependencies and setup instructions in the repository.
- If you use a remote inference service, use CampusAI.
- Exclude API keys and other credentials from the hand-in and container image.

Testing
-------
With the service running, inspect and test it through the Swagger UI:

http://127.0.0.1:8000/docs

You should also be able to call the sentiment endpoint from the command line:

```bash
curl -s -X POST http://localhost:8000/v1/sentiment \
  -H 'Content-Type: application/json' \
  -d '{"text":"It was a good course."}'
```

If you implement person extraction, test it similarly:

```bash
curl -s -X POST http://localhost:8000/v1/extract-persons \
  -H 'Content-Type: application/json' \
  -d '{"text":"Einstein and von Neumann meet each other."}'
```

Include automated tests in one or more files prefixed with `test_`.
Test the HTTP response format and representative positive, neutral, and negative texts in both languages.
Add examples involving negation or mixed sentiment.

For the optional person endpoint, include tests with multiple names and with no names.

Questions
---------
Consider the following issues:

- How does your decision model compare with your sentiment implementation from the first exercise and with a general-purpose generative LLM?
- How do the labels, label descriptions, or instructions affect predictions?
- Does the model perform equally well on Danish and English? What do the model card and your own tests tell you?
- What is the difference between choosing the most probable category and returning a probability-weighted score?
- If the backend reports probabilities and confidence, what do these quantities mean? Are the probabilities well calibrated?
- What are the trade-offs between CampusAI and local inference in latency, memory, container size, startup time, and concurrent requests?
- For the optional endpoint, can the same checkpoint perform both classification and entity extraction? How would you assess whether it performs both tasks well?

Requirements & Resources
------------------------
- Python, FastAPI, Git, and Docker/Podman.
- Finn Årup Nielsen, [Natural language processing](https://people.compute.dtu.dk/faan/ps/Nielsen2025Natural.pdf):
  - Chapter 13, “Decision models”, particularly sections 13.3–13.5. In the 4 November 2026 version, the chapter is on printed pages 119–122 (PDF pages 121–124).
  - The chapters and sections on sentiment analysis, Web serving, containerization, and testing.
- Finn Årup Nielsen, *Knowledge graphs*: the section on entity recognition, for the optional endpoint.
- [CampusAI](https://campusai.compute.dtu.dk/).
- [vLLM: Structured Decisions Models and the System One endpoint](https://docs.vllm.ai/en/latest/models/structured_decisions_models/#typesafe-system-one-endpoint).
- [GLiNER2 documentation](https://github.com/fastino-ai/GLiNER2).
- Example local model: [GLiNER2.5-multi-Decide](https://huggingface.co/fastino/GLiNER2.5-multi-Decide).

Hand-in
-------
Hand in a zipped archive of your Git repository, with a `Dockerfile` in the root:

```bash
git archive -o latest.zip HEAD
```

Include the source code, dependency specification, tests, and a
`README.md` describing how to build and run the service, which model
and package versions you used, and how its output is mapped to the
sentiment score. 
State whether the optional person extraction endpoint is implemented.

Commit the files you want to include before creating the archive.

